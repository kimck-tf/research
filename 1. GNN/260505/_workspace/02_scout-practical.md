# FEM 실무 파이프라인 동향

작성: scout-practical (node-element-team)
일자: 2026-05-06
대상 환경: Abaqus 2022/2023, Python 2.7 odbAccess, 본 논문 580 샘플 / 200~400k 노드

---

## 1. 핵심 발견 요약 (5줄 이내)

1. Abaqus ODB의 응력 추출 옵션은 사실상 4종(`NODAL` / `CENTROID` / `INTEGRATION_POINT` / `ELEMENT_NODAL`)이며, **본 논문이 사용한 averaged nodal은 IP→node 외삽 후 인접 요소 평균이 한 번 더 적용된, 정보 손실이 가장 큰 형식**이다.
2. **Element centroid 추출은 본 논문 스크립트에서 인자 1개 변경(`position=NODAL` → `position=CENTROID`) 수준으로, 추가 비용이 가장 낮으면서도 ISO 3006/사내 평가 metric과 직접 호환된다.**
3. IP raw 추출은 데이터 부피 6~8배 증가(C3D8 기준 8 IP × 6 stress components), 추출 시간 3~5배 증가가 예상되며, 학습 라벨로 직접 활용하기에는 후처리 부담이 크다.
4. SPR(Zienkiewicz-Zhu 1992)은 학술적으로 우아하나 Abaqus가 이미 노드 외삽에서 비슷한 averaging을 자동 수행하므로 사후 적용의 한계 효용이 작다 — 차라리 **요소 centroid 응력을 학습 라벨로 사용**하는 편이 단순/명료하다.
5. 상용 surrogate(Neural Concept, Altair physicsAI, NVIDIA PhysicsNeMo)는 **노드 출력이 표준**이지만, Altair는 explicit dynamics(crash) 사례에서 element 변수도 출력 가능하며 BMO-GNN 등 학술 사례는 element-level 출력의 산업 채택을 시사한다.

---

## 2. Abaqus ODB 응력 데이터 형식 비교

### 2.1 Node extrapolated, averaged (현재 본 논문 사용 — `position=NODAL`)

- **계산 절차** (Abaqus User Manual 24.5.1, "Understanding how results are computed"):
  1. 솔버는 element의 IP에서 stress tensor 계산
  2. 후처리 시 element shape function의 inverse를 사용해 IP → element nodes 외삽
  3. 한 노드를 공유하는 인접 요소들의 외삽 stress를 산술 평균 (averaging)
- **Python API**:
  ```python
  stress_field = odb.steps['Step-1'].frames[-1].fieldOutputs['S']
  nodal_stress = stress_field.getSubset(position=NODAL).values  # 자동 averaging
  ```
- **데이터 부피 (basis)**: 노드당 stress 6 components (S11, S22, S33, S12, S13, S23) → 노드 수 N_n × 6 = **1×**
- **정보 충실도**: **낮음**. 외삽 + averaging 두 단계 smoothing으로 응력 집중부 왜곡, 본 논문 PROBLEM.md 의 핵심 문제.

### 2.2 Element centroid (`position=CENTROID`)

- **계산 절차**: IP 응력을 element 중심 1점으로 외삽 (단방향, averaging 없음)
- **Python API**:
  ```python
  centroid_stress = stress_field.getSubset(position=CENTROID).values
  # 또는 elementType 지정: getSubset(region=instance.elements, position=CENTROID, elementType='C3D8')
  ```
- **데이터 부피**: element당 stress 6 → element 수 N_e × 6
  - 본 논문 200~400k 노드 메시(spoke 영역). 솔리드 hex 메시일 때 N_e ≈ N_n × (0.85~1.0)으로 가정 [추정] → **0.85~1.0×**
- **정보 충실도**: **중간**. averaging 없으나 IP→centroid 외삽은 여전히 1단계 smoothing
- **실무 호환성**: **ISO 3006 / 사내 휠 평가가 element von Mises 기반이므로 평가 metric과 직접 일치**

### 2.3 Integration point raw (`position=INTEGRATION_POINT`)

- **계산 절차**: 솔버 결과 그대로, 외삽·평균 모두 없음
- **Python API**:
  ```python
  ip_stress = stress_field.getSubset(position=INTEGRATION_POINT).values
  # 각 value의 .integrationPoint 속성으로 IP index (1~8 for C3D8) 식별
  ```
- **데이터 부피**: element당 8 IP × 6 components (C3D8 기준)
  - C3D8R(reduced integration)은 1 IP/element이므로 부피 1× 수준이지만, 본 논문 휠 모델이 어떤 element인지 확정 필요 `[추가 검증 필요]`
  - C3D8 full integration 기준: **6~8×** baseline 대비
- **정보 충실도**: **최고**. 솔버 출력 그대로, 정보 손실 0
- **실무 부담**: ODB 파일 크기 폭증, 추출 시간 3~5× `[추정]`, 후처리 시 IP 좌표·element shape function 재구성 필요

### 2.4 Unaveraged nodal (`position=ELEMENT_NODAL`)

- **계산 절차**: IP → element nodes 외삽까지만, 인접 요소 averaging 없음 → 동일 mesh node가 element별로 여러 stress 값을 보유
- **Python API**:
  ```python
  en_stress = stress_field.getSubset(position=ELEMENT_NODAL).values
  # 각 value는 (elementLabel, nodeLabel, stress) tuple
  ```
- **데이터 부피**: element당 8 nodes × 6 = element 수 × 8 × 6
  - C3D8 hex 기준 노드 1개당 평균 8개 element가 공유 → **약 8×** (사실상 IP raw와 동급)
- **정보 충실도**: **중상**. averaging은 없으나 외삽은 1단계 적용
- **실무 활용**: ParaView 등에서 mesh quality 진단 (averaged vs unaveraged 차이가 클수록 mesh 부족)

### 2.5 데이터 부피 / 추출 시간 / 정보 손실 비교 표

본 논문 환경 (1 sample = 약 300k nodes, hex element 가정, 580 samples) 기준 정량 추정:

| 형식 | 단위/element | 부피 (relative) | 1 sample 크기 [추정] | 580 samples 총량 [추정] | 추출 시간 (relative) | smoothing 단계 | 정보 손실 |
|---|---|---|---|---|---|---|---|
| `NODAL` (averaged) | 6 / node | 1.0× | ~7 MB (float32) | ~4 GB | 1.0× | extrapolation + averaging | **High** |
| `CENTROID` | 6 / elem | 0.85~1.0× | ~6 MB | ~3.5 GB | 1.1× | extrapolation only | **Medium** |
| `INTEGRATION_POINT` (C3D8) | 6 × 8 / elem | 6~8× | ~50 MB | ~30 GB | 3~5× | none | **None** |
| `INTEGRATION_POINT` (C3D8R) | 6 × 1 / elem | 0.85~1.0× | ~6 MB | ~3.5 GB | 1.5× | none | **None** |
| `ELEMENT_NODAL` (unavg) | 6 × 8 / elem | ~8× | ~50 MB | ~30 GB | 3× | extrapolation only | **Low** |

`[추정]` 절대값은 본 논문 mesh 구체 사양(요소 종류, IP 수)에 따라 ±50% 변동.
`[Abaqus 환경 의존]` C3D8R(reduced integration, 1 IP)을 사용한다면 IP 추출 비용은 centroid와 동급.

**핵심 시사점**: 본 논문이 C3D8R을 쓴다면 IP/centroid/NODAL 부피 차이가 작아져 **IP 추출 → 후속 element-level 학습이 실무적으로 충분히 가능**. C3D8 full integration이라면 centroid가 가장 비용 효율적.

---

## 3. FEM Stress Recovery / Mapping 알고리즘

### 3.1 SPR (Superconvergent Patch Recovery)

- **출처**: Zienkiewicz, Zhu, IJNME 1992 (Vol 33, Issue 7-8) "The superconvergent patch recovery and a posteriori error estimates" Part 1 & 2
- **개념**:
  - 노드를 둘러싼 element patch에서 IP의 stress를 polynomial(보통 quadratic)에 least-squares로 fit
  - 동일 polynomial을 노드 좌표에 evaluate하여 superconvergent O(h^(p+1)) 정확도의 노드 응력 획득
- **장점**: 단순 averaging보다 응력 집중부 정확도가 높고, FEM error estimator의 표준
- **단점**: patch 정의가 boundary node에서 불안정 → SPR-C (constraint version, Diez et al.)로 보완
- **Abaqus 통합**: 표준 후처리 옵션에 미포함 (averaging 방식 사용). SPR을 별도 적용하려면 IP raw 데이터 추출 후 외부 코드 작성 필요

### 3.2 L2-projection

- **개념**: IP에서 평가된 stress field를 globally smooth 한 함수공간으로 변분원리(L2 minimization) 사상
- **수식**: ∫(σ_smooth - σ_IP)² dΩ → minimize w.r.t. nodal stress unknowns
- **비교**: SPR은 patch-local, L2-projection은 global. SPR이 일반적으로 quadratic element에서 우월
- **본 논문 적용성**: GNN 학습 후 출력 단계에서 적용 가능하나 추가 mass matrix 풀이가 필요해 복잡도 증가

### 3.3 Lumping / averaging 방식

- **Abaqus의 default**: 단순 산술 averaging (가중치 없음)
- **Volume-weighted averaging**: 각 element 부피로 가중 평균 → spoke 같이 element 크기 편차 큰 영역에서 더 합리적
- **본 논문 baseline의 averaging**은 사실상 가장 단순한 선택이며, volume-weighted 변경만으로도 노드 응력의 신뢰도가 일부 개선 가능 [추정]

### 3.4 본 논문 데이터에 적용 가능성

| 알고리즘 | 적용 위치 | 추가 비용 | 본 논문 효용 |
|---|---|---|---|
| SPR (post-prediction) | GNN 출력 후처리 | IP raw 추출 + patch fitting 코드 | 낮음 — element centroid 직접 학습이 더 단순 |
| L2-projection | GNN 출력 후처리 | mass matrix solve | 낮음 — 후처리 변환 비용 |
| Volume-weighted averaging | ODB 추출 시 | element 부피 추가 추출 | **중간** — 기존 노드 라벨 품질 향상 가능 |
| Element centroid 직접 학습 | 학습 단계 | 스크립트 1줄 변경 | **높음** — 본 논문 도메인 불일치를 근본 해결 |

**권장**: SPR/L2-projection은 GNN 학습 paradigm과 결이 달라 우선순위 낮음. **데이터 추출 단계에서 centroid 또는 IP를 사용**하는 것이 보다 직접적.

---

## 4. 상용 Surrogate 플랫폼의 응력 처리 방식

### 4.1 Neural Concept / Altair physicsAI / NVIDIA PhysicsNeMo

| 플랫폼 | 응력 출력 위치 | element 응력 직접 출력 | 비고 |
|---|---|---|---|
| **Neural Concept Shape** | 노드 (mesh vertex) | 미공개 / 노드 출력 표준 | Geodesic CNN 기반, 표면 메시 위주, 노드별 von Mises 직접 회귀 |
| **Altair physicsAI** | 노드 + element 모두 | **가능** (변위/응력/변형률 별도 모델) | GDL 기반. Cyient 사례에서 crash explicit dynamics에 element 변수 직접 학습 |
| **NVIDIA PhysicsNeMo MeshGraphNet** | 노드 (decoder MLP per node) | 표준 미지원 | LatticeGraphNet (additive manufacturing) 에서 displacement + stress invariant를 노드 단위로 출력 |
| **Ansys Discovery AI** | 미공개 / 노드 추정 | 불명 | `[추가 검증 필요]` |
| **Siemens Simcenter HEEDS / Reduced-order** | 미공개 | 불명 | `[추가 검증 필요]` |

**핵심 패턴**: 학술/오픈소스 플랫폼은 노드 출력이 사실상 표준. 상용은 **유연한 출력 위치 지정이 가능**하지만 자세한 아키텍처는 비공개.

### 4.2 OEM 채택 사례에서의 후처리 방식

- **Altair PhysicsAI 자동차 crash 사례** (Cyient white paper, Altair Taiwan rail crash): 450 FEA 학습 + 50 검증, 1000× 가속. 변위/응력/변형률을 **별도 모델**로 학습 → element 응력을 노드 응력과 분리해서 다룸
- **BMO-GNN (J. Comput. Design Eng. 2024)**: rim stiffness 예측에 GNN 사용, 3D CNN/SubdivNet/GCN 대비 우수. 노드 입력 + 그래프 단위 출력 (질량/강성)
- **Hyundai 사내 방식**: 본 논문 pipeline 외 공개 정보 없음 `[추가 검증 필요]`. 본 논문이 노드 응력 → 평가 metric 변환을 어떻게 처리하는지 향후 확인 필요

**시사점**: 산업 사례는 **응력과 변위에 다른 모델/다른 출력 형식**을 쓰는 경향이 있다 → Gemini 대안 1 (Multi-Head Decoder)의 산업적 정당성 확인됨.

---

## 5. 본 논문 파이프라인 변경 영향 평가

### 5.1 IP 응력 추출로 변경 시 비용 (시간/용량/스크립트 수정)

본 논문 baseline은 580 샘플 / 200~400k 노드 / 자동 ODB 추출 스크립트로 가정.

| 시나리오 | 데이터 부피 | 추출 시간 | 스크립트 수정 [추정] | 후처리 부담 |
|---|---|---|---|---|
| **S0**. baseline (averaged nodal) | 1× (~4 GB) | 1× | — | 없음 |
| **S1**. Element centroid 추가 추출 | +0.85× (~7.5 GB total) | +10~15% | **1줄 추가** (`position=CENTROID`) | 낮음 — element label로 mapping만 |
| **S2**. IP raw 추출로 전환 (C3D8) | 6~8× (~30 GB) | 3~5× | **20~50줄 수정** (IP index 처리, element type 분기) | 높음 — IP→학습용 형식 변환 필요 |
| **S2'**. IP raw 추출 (C3D8R 가정) | ~1× (~4 GB) | 1.5× | **5~10줄 수정** | 중간 |
| **S3**. Node + Centroid 동시 추출 | ~1.85× (~8 GB) | +20% | **3~5줄 추가** | 낮음 |
| **S4**. Node 응력 + 후처리 SPR | 1× | 1× | 추출 변경 없음, **post-script 200~500줄 신규** | 매우 높음 — patch fitting 구현 |

`[추정]` 스크립트 줄 수는 본 논문 baseline 추출 스크립트의 평균적 구조 가정.

**중요 발견**: **S3 (Node + Centroid 동시 추출)이 baseline 호환성과 신정보 획득의 균형 측면에서 최적**. 기존 노드 라벨을 유지하면서 element 라벨을 추가 확보 → Gemini 대안 1 (Multi-Head Decoder, Node head + Element head) 의 학습에 그대로 사용 가능.

### 5.2 Element centroid 추출이 가장 실무적인지

**Yes. 다음 4가지 이유로 centroid 추출이 가장 실무적:**

1. **Abaqus API 1줄 변경**으로 추출 가능 (`position=NODAL` → `position=CENTROID`)
2. **데이터 부피 거의 동일** (1× ↔ 0.85~1.0×) → 디스크/메모리/네트워크 비용 영향 거의 없음
3. **ISO 3006 / 사내 평가 metric과 직접 호환** — element von Mises 평가 로직에 변환 없이 투입 가능
4. **Gemini 대안 1·2·3 모두에서 라벨로 활용 가능** — 대안별 GNN 출력 정의 변경에 무관하게 centroid 데이터가 ground truth 역할

**유일한 단점**: IP → centroid 외삽이 1단계 smoothing이라 응력 집중부에서 미세한 왜곡은 잔존. 이를 완전히 제거하려면 S2 (IP raw)가 필요하나, 본 논문 환경에서는 비용 대비 효용이 낮음.

---

## 6. ISO 3006 / KMVSS / Hyundai 사내 평가 metric

### 6.1 평가 시 element 응력 사용 방식

- **ISO 3006:2015** (Road vehicles — Passenger car wheels — Test methods):
  - 라디알 피로, 다이나믹 코너링 피로, 13° 충격 시험 3종
  - FEA 기반 평가는 **element peak von Mises stress vs material yield** 비교가 표준 (J. Mater. Sci. Mater. Eng. 2026, "Geometrical optimization of aluminum alloy wheels"; ResearchGate "Radial Fatigue Analysis ISO 3006")
  - Modified Goodman 접근으로 피로 수명 예측 — element 단위 응력의 mean/amplitude 사용
- **KMVSS (한국 자동차 안전기준)**: 휠 강도 항목은 ISO 3006 동등 적용 `[추가 검증 필요]`
- **Hyundai 사내**: 공개 자료 없음. 본 논문 PROBLEM.md 가 "element von Mises 기반"으로 명시 → 사내 기준도 element 응력 사용 추정

### 6.2 GNN 출력이 평가 로직에 직접 투입 가능한 형태

| GNN 출력 형식 | 평가 직접 투입 | 변환 필요 |
|---|---|---|
| Node averaged stress (현재) | **불가** | element averaging 추가 변환 필요 (정보 손실 누적) |
| Node unaveraged stress (per element) | 가능 (element 별 평균 후) | element 평균 1단계 |
| **Element centroid stress** | **가능 (직접)** | 없음 |
| **Element von Mises (직접 출력)** | **가능 (즉시)** | 없음 |
| IP raw stress | 가능 | element 대표값 산출 (max/avg) |

**결론**: 평가 호환성은 element 위치 출력이 절대적으로 우월. **본 논문 baseline의 노드 응력 출력은 평가 단계에서 한 번 더 변환이 필요**, 이는 GNN 예측 오차 + 변환 오차 누적 위험을 의미.

---

## 7. 실무 관점 권장 데이터 형식 (Top 1~2)

### Top 1: Node + Element Centroid 동시 추출 + Multi-Head 학습 (시나리오 S3)

- **추출**: `getSubset(position=NODAL)` + `getSubset(position=CENTROID)` 동시 실행 → ~1.85× 부피
- **학습**: Gemini 대안 1 (Multi-Head Decoder)
  - Node head: 변위 (기존 baseline 유지)
  - Element head: centroid stress (신규 평가 호환)
- **장점**:
  - 본 논문 baseline 코드 90% 이상 재활용
  - ISO 3006 평가에 직접 투입 가능한 element 응력 획득
  - 변위 예측 능력은 그대로 보존
- **추가 비용**: 추출 스크립트 ~5줄, 학습 코드 ~30~50줄 (decoder head 추가) `[추정]`

### Top 2: Element Centroid 단독 추출 + Dual Graph 학습 (시나리오 S1 + Gemini 대안 3)

- **추출**: `getSubset(position=CENTROID)` 만 → ~0.85× 부피 (오히려 baseline보다 작음)
- **학습**: Gemini 대안 3 (Dual Graph) — element를 node로, face-sharing element를 edge로
- **장점**:
  - 데이터 형식이 평가 metric과 1:1 일치
  - 노드 → element 매핑 후처리 불필요
- **단점**:
  - 변위(node 정의)를 다루기 어려움 → 변위 예측 별도 모델 필요
  - 그래프 구축 코드 신규 작성 (~100~200줄) `[추정]`

**추천 우선순위**: **Top 1 (S3 + Multi-Head)** — 본 논문 baseline 보존 + 평가 호환 + 학술적 신규성 균형. Top 2는 응력 예측 단일 목적 후속 연구에 적합.

---

## 다음 에이전트에게 전달

### scout-academic 에게 (학술 모델 적합성 재평가용)

- **데이터 형식 제약**:
  - IP raw stress(C3D8 full): 6 components × 8 IPs/element → 노드 응력 대비 6~8배 부피, 580 샘플 기준 ~30 GB (관리 가능 수준이나 학습 시 IO bottleneck 위험)
  - C3D8R(reduced integration) 사용 시 IP 부피는 centroid와 동급 → IP-level 학술 모델(예: IP를 별도 graph node로 도입)도 본 논문 환경에서 실현 가능. **본 논문이 사용한 element type 확인이 핵심 의사결정 변수**
- **본 논문 환경에서 추출 비용이 과도한 학술 모델 후보**:
  - 모든 IP를 별도 graph node로 도입하는 hyper-graph 모델 (메모리 8× 증가 + edge 수 폭증) → C3D8 full integration 사용 시 비현실적
  - 반면, dual graph (element-as-node)는 부피 1× 수준이라 실무 적용 가능
- **학술 가정 vs 실무 제약 미스매치**:
  - SPR/L2-projection은 학술적으로 우아하나 Abaqus의 노드 외삽이 이미 유사 averaging을 수행하므로 사후 적용의 한계 효용 낮음 → "데이터 추출 단계 변경"이 더 직접적
  - 상용 surrogate(Altair physicsAI)가 변위/응력/변형률 **별도 모델** 패턴을 쓴다는 점은 **단일 GNN으로 모두 통합** 시도하는 학술 모델 가정과 충돌. Multi-Head 또는 Dual-Model이 산업적으로 더 자연스러움

### analyst-alternatives 에게 (대안별 데이터 비용 검토용)

- **Gemini 대안 1 (Element-wise Decoding)**:
  - 데이터: Element centroid 추출 — Abaqus 스크립트 **1줄 추가** (`position=CENTROID`)
  - 학습 코드: Decoder head 추가 ~30~50줄
  - 데이터 부피: 0.85~1.0× (baseline 대비 거의 동일)
  - **본 논문 baseline 재활용율: 90%+**
- **Gemini 대안 2 (Bipartite Graph)**:
  - 데이터: Node + Element centroid 동시 추출 — 스크립트 **3~5줄 추가**
  - 그래프 구축: element-node bipartite 연결 정의 ~100줄, MGN message passing 분기 ~50~100줄
  - 데이터 부피: 1.85× (양 라벨 보유)
  - **본 논문 baseline 재활용율: 60~70%**
- **Gemini 대안 3 (Dual Graph)**:
  - 데이터: Element centroid 단독 추출 — 스크립트 **1줄 변경**
  - 그래프 구축: face-sharing 인접 관계 산출 ~150~200줄 (Abaqus mesh connectivity 파싱), edge attribute 정의
  - 데이터 부피: 0.85~1.0× (오히려 baseline보다 약간 작음)
  - **본 논문 baseline 재활용율: 30~40%** (그래프 구조 자체가 변경)
  - 변위 예측 별도 모델 필요 → 실무 부담 추가
- **본 논문 baseline 스크립트 수정 분량 추정**:
  - 대안 1: ODB 1줄 + 학습 30~50줄 → **총 ~50줄**
  - 대안 2: ODB 5줄 + 학습 150~200줄 → **총 ~200줄**
  - 대안 3: ODB 1줄 + 그래프 150~200줄 + 학습 100줄 + 변위 모델 별도 → **총 ~400줄+**
- **ISO 3006 호환성**:
  - **대안 1, 3 우수** (element 단위 출력 → 평가에 직접 투입)
  - 대안 2는 element vertex 디코딩 결과를 사용 시 동등하게 호환
  - **현재 baseline (averaged nodal)은 호환 불가** — 평가에 추가 변환 필요
- **권장**: 비용/효용/baseline 호환성 종합 시 **대안 1 (Element-wise Decoding) 이 최선**, 구현 난이도와 학술 신규성을 동시 고려 시 대안 2 (Bipartite) 가 차선. 대안 3은 응력 단일 목적 후속 연구에 적합.

---

## 출처

- Abaqus User Manual v6.6, "31.4 FieldOutput object" (Washington Univ. mirror): https://classes.engineering.wustl.edu/2009/spring/mase5513/abaqus/docs/v6.6/books/ker/pt01ch31pyo04.html
- Abaqus CAE User Manual, "24.5.1 Understanding how results are computed", "24.5.2 Result value averaging"
- Abaqus 2017 Three-dimensional solid element library (C3D8): https://abaqus-docs.mit.edu/2017/English/SIMACAEELMRefMap/simaelm-r-3delem.htm
- Zienkiewicz O.C., Zhu J.Z. (1992) "The superconvergent patch recovery and a posteriori error estimates", IJNME 33(7-8). https://onlinelibrary.wiley.com/doi/10.1002/nme.1620330702
- ParaView Discourse "Averaged to unaveraged stresses": https://discourse.paraview.org/t/averaged-to-unaveraged-stresses/11719
- 4RealSim "Result Averaging in Abaqus": https://www.4realsim.com/post/result-averaging-in-abaqus
- ISO 3006:2015 — Road vehicles — Passenger car wheels for road use — Test methods. https://www.iso.org/standard/60208.html
- "Geometrical optimization of aluminum alloy wheels for high fatigue and impact strength", J. Mater. Sci. Mater. Eng. 2026: https://link.springer.com/article/10.1186/s40712-026-00418-9
- Altair PhysicsAI 공식: https://altair.com/physicsai
- Cyient white paper "Altair PhysicsAI - Revolutionizing Explicit Dynamic Simulations": https://www.cyient.com/whitepaper/altair-physics-ai-revolutionizing-explicit-dynamic-simulations-cyient
- BMO-GNN, J. Comput. Design Eng. 2024: https://academic.oup.com/jcde/article/11/6/260/7896417
- NVIDIA PhysicsNeMo MeshGraphNet docs: https://docs.nvidia.com/physicsnemo/latest/user-guide/model_architecture/meshgraphnet.html
- X-MeshGraphNet (arxiv 2411.17164): https://arxiv.org/html/2411.17164v2
