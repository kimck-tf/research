# 최종 추천 대안

**작성**: analyst-alternatives
**작성일**: 2026-05-06 / **갱신일**: 2026-05-07 (사용자 피드백 반영)
**입력**: `03_alternatives-matrix.md` 6축 매트릭스 + 본 논문 환경 시뮬레이션
**출력 사용처**: report-author 가 보고서 5장(추천)·6장(로드맵)에 직접 활용

**핵심 갱신 사항 (2026-05-07)**:
- **확정**: 본 논문 C3D8 (full integration) 사용 (사용자 피드백)
- **추천 #2 교체**: A4-C3D8R (IP-as-Node, reduced int., 37.0) → **A3 Dual Graph (Element-as-Node, 33.5)**
- 사유: C3D8 환경에서 A4-C3D8 (full int.) 는 학습 비용 +150~200% / B=1 강제 / IO bottleneck → 6개월 로드맵 outside. A3 는 centroid 사용으로 C3D8 영향 없음 + Hestroffer/Storm 학술 baseline + 본 논문이 대변형 동적 임팩트 갭 보강 = CMAME publication 직접 명분
- **추천 #1 (A1+EQ) 변경 없음** — 사용자 옵션 A 선택은 추천 #2 한정

---

## 1. Top 추천 1: 단기 Quick Win (1~2개월)

### 1.1 대안 명칭 및 근거

**대안명**: **A1+EQ — Element-wise Decoding (Multi-Head, Volume-weighted) + Equilibrium-loss plug-in**

**근거 요약**:
- 6축 매트릭스 종합 점수 **40.0 (1위)**
- baseline 코드 재활용율 **85%+** — 1~2개월 내 PoC 가능 (scout-practical 정량 확인)
- 학술 정당화 4건 직접 ref (Maurizi 2022, Pfaff 2021, Dalton 2024, Hernández 2025) + plug-in 1건 (P-DivGNN 2025) — scout-academic 정당화 최상
- 데이터 추출 비용 +10~15% (시나리오 S1 또는 S3, 스크립트 1~5줄)
- ISO 3006 / KMVSS 평가 metric 직접 호환 (element von Mises 출력)
- 산업 사례 (Altair physicsAI Multi-Head 패턴) 와 일치 → 양산 도입 정당성

**선정에서 결정적 차별점**:
- A1 baseline 대비 **학술 신규성 상승 (중→상)**: P-DivGNN equilibrium loss 추가는 plug-in 수준 (5% 추가 코드)이지만 후속 publication 가능
- A4-C3D8R 대비 **본 논문 element type 미확정 분기점 회피**: A1+EQ는 element type 무관. C3D8R 확정 시 A4-C3D8R로 마이그레이션 가능 (추천 2)

### 1.2 구현 단계 (Step 1~6, 8~10주 추정)

```
Week 1~2: 데이터 추출 변경
  Step 1.1 [scout-practical S3 시나리오]
    - Abaqus odb 스크립트에 `getSubset(position=CENTROID)` 추가 (~3~5줄)
    - 580 샘플 × element centroid stress 6 components 추가 추출
    - element label - element node mapping 테이블 산출 (mesh connectivity)
  Step 1.2 [데이터 검증]
    - 한 샘플에서 노드 averaged stress vs centroid stress 분포 비교
    - 응력 집중부(spoke)에서 차이가 큼을 확인 — 본 논문 문제의 직접 증거

Week 3~4: Multi-Head Decoder 구현
  Step 2.1 [모델 구조 변경]
    - MGN baseline의 final decoder MLP를 두 head로 분리:
        head_node (변위 3) — 기존 baseline 그대로 보존
        head_element (응력 6 또는 von Mises 1) — 신규
    - Pooling 함수: A1-volume-weighted 우선 채택 (Storm 2024 shape-function-weighted ref + scout-practical 3.3 volume-weighted 권장)
        h_E = Σ_i (V_i / V_E) · h_node_i   (V_i: i번째 노드 인접 element 부분 부피)
    - 코드 추가 ~30~50줄
  Step 2.2 [Loss 정의]
    L = w₁ ‖u_node_pred - u_node_gt‖² + w₂ ‖σ_element_pred - σ_centroid_gt‖²
    초기 w₁=1.0, w₂=1.0 설정. ablation에서 w 조정.

Week 5~6: Equilibrium-loss plug-in
  Step 3.1 [div σ 잔차 계산 모듈 추가, P-DivGNN 차용]
    - element 별 ∂σ/∂x 계산 — Gauss-Green 정리로 면적분 우회 (Abaqus B-matrix 우회)
        div σ_E = (1/V_E) ∮_∂E σ · n dA  → 인접 element σ 차이로 근사
    - Loss 추가: w₃ ‖div σ‖² (단, 외력 0인 spoke 영역 한정)
    - 코드 ~80~120줄
  Step 3.2 [학습 안정화]
    - w₃ warm-up scheduling: epoch 0~500은 w₃=0, 500~1000은 선형 증가
    - 초기에는 main loss 수렴 후 EQ loss 활성화

Week 7~8: Pooling ablation + 평가 metric 확장
  Step 4.1 [A1 변형 ablation]
    - A1-avg / A1-max / A1-volume-weighted / A1-shape-weighted 4종 비교
    - 본 논문 Top-20% MAPE / KLD / R² (element-level) 측정
  Step 4.2 [평가 metric 추가]
    - element von Mises 직접 출력 → ISO 3006 평가 로직에 직접 투입 (변환 0)
    - 노드 응력 baseline과 비교 시 평가 단계 변환 오차 누적 vs A1+EQ 직접 호환의 정확도 차이 정량화

Week 9~10: 보고서 / 후속 paper draft
  Step 5.1 [본 논문 D1 ablation에 통합]
    - 260502 D1 (SOTA Baseline ablation) 의 한 항목으로 자연 포함
  Step 5.2 [후속 paper outline]
    - "Pooling function ablation for element-wise decoding in MGN" — JSAE / SAE workshop 후보
    - "Equilibrium-loss plug-in for FEM-aware MGN" — CMAME 또는 Comp.Mech.

  Step 6 [모델 저장 / 양산 파이프라인 통합]
    - 추론 시간 baseline 동일 (~300~500 ms) — 양산 영향 없음
    - 평가 metric 직접 호환 → CAE 평가 자동화 파이프라인 단순화
```

### 1.3 성공 지표 (정량)

| 지표 | baseline (본 논문 보고치) | 목표 (A1+EQ) | 보고서 수치 |
|---|---|---|---|
| Top-20% MAPE | 7.0% | **≤6.5% (개선 -7%)** | scout-academic Hernández 2025 multi-decoder 향상 추정 |
| KLD (Top-20%) | 0.005 | ≤0.005 (유지) | |
| R² (Top 100%, element) | 0.712 (node-level) | **≥0.75 (element-level)** | 라벨 정합성 향상 |
| 평가 metric 호환성 | 변환 필요 | **변환 0** | ISO 3006 직접 투입 |
| 추론 시간 / 단일 휠 | 300~500 ms | 300~500 ms (동일) | scout-practical 추론 비용 매트릭스 |
| baseline 코드 재활용율 | — | **85%+** | scout-practical 5.1 정량 |
| 학습 epoch 시간 | 15~25분 | 17~28분 (+10%) | EQ loss gradient 추가 |
| GPU 메모리 | 20~25 GB | 22~27 GB (+10%) | A100 단일 GPU 운영 가능 |

### 1.4 리스크 / 완화책

| 리스크 | 완화책 |
|---|---|
| **R1. Pooling 함수 선택의 학술적 미정** (avg/max/volume) | Step 4.1 에서 4종 ablation. volume-weighted를 default, avg를 fallback. 학술 contribution으로 전환 |
| **R2. Loss weighting w₁/w₂/w₃ tuning 비용** | grid search 9 조합 (3×3) 자동화. baseline w₁=w₂=1, w₃=0.1 시작. P-DivGNN ref 의 warm-up scheduling 차용 |
| **R3. EQ loss 계산 시 div σ 근사 정확도** | Gauss-Green 면적분 우회로 B-matrix 회피. 실패 시 FEINN 의 명시 B-matrix 통합으로 fallback (단, 비용 증가) |
| **R4. centroid stress 라벨이 IP raw 대비 정보 손실 1단계** | scout-practical 3.4: "Abaqus 노드 외삽이 이미 averaging 수행 → centroid 1단계 외삽이 더 양호". 추가 검증 필요 시 추천 2 (A4-C3D8R) 로 마이그레이션 |
| **R5. 본 논문 mesh 혼합 element type 인 경우 pooling 일관성** | element type 별 pooling weight 분리 (C3D8: 8 노드, C3D4: 4 노드 등). PyG `scatter` 기반 일반 구현 |
| **R6. 580 샘플 부족으로 multi-head overfitting** | Modified L₂ loss(+10%) + Random edge augmentation(+3%) 본 논문 기법 그대로 유지. dropout 추가 가능 |

### 1.5 본 논문 baseline 코드 재활용 비율

- **데이터 추출 스크립트**: 95%+ 재활용 (1~5줄 추가)
- **MGN encoder/processor**: 100% 재활용 (변경 없음)
- **MGN decoder**: ~50% 재활용 (head 분리)
- **Loss 함수**: ~70% 재활용 (term 추가)
- **학습 루프 / optimizer / scheduling**: 100% 재활용
- **평가 / 시각화 코드**: ~80% 재활용 (element-level metric 추가)
- **종합 재활용율**: **85~90%**

---

## 2. Top 추천 2: 중기 학술 Publication (3~6개월) — A3: Dual Graph (Element-as-Node)

*(2026-05-07 갱신: 사용자 피드백으로 본 논문 C3D8 사용 확정 → A4-C3D8R 환경 미적용. A4-C3D8 (full) 은 학습 +150~200% / B=1 강제 / IO bottleneck → 6개월 outside. A3 가 C3D8 환경 적용 가능 차선 경로로 채택)*

### 2.1 대안 명칭 및 근거

**대안명**: **A3 — Dual Graph (Element-as-Node Graph Neural Network)**

**근거 요약**:
- 6축 매트릭스 종합 점수 **33.5 (C3D8 환경 현실 가능 대안 중 추천 #1 다음 2위)**
- 학술 baseline: **Hestroffer 2025** (Comp.Mech, R²=0.992 polycrystal **정적**), **Storm 2024** (CMAME), Lu 2025 (lattice truss)
- **본 논문 기여 = 학술 갭 보강**: Hestroffer/Storm 의 정적·소변형 학술 baseline의 명확한 미답 영역 = **대변형 동적 임팩트 + 양산 휠 (200~400k 노드)** 을 채움 → CMAME publication 직접 명분
- 핵심 수식: 그래프 G' = (V', E'), V' = elements, E' = face-shared 인접 element 쌍
- 입력: element centroid 좌표, element 부피, 재료 물성, 인접 element 정보
- 출력: element von Mises stress (직접 평가 호환)
- **C3D8 영향 없음**: centroid 추출 (시나리오 S1) 사용 → IP 추출 부담 회피
- Element 단위 출력 → ISO 3006 평가 직접 호환 (변환 0)
- 단점: baseline 재활용율 30~40% (그래프 구조 차이)

**선정 결정점**:
- C3D8 환경에서 A4-C3D8 (full int.) 비용 부담 (학습 +150~200%, B=1 강제, 메모리 50~60 GB) → 6개월 outside
- C3D8 환경 적용 가능 대안 중 학술 신규성·정량 근거가 가장 강력한 차선 경로
- 추천 1 (A1+EQ) 로부터 자연스러운 진화 경로: 노드 그래프 → 요소 그래프 (encoder/processor 골격 재사용)

### 2.2 구현 단계 (Step 1~6, 12~24주)

```
Month 1: ODB centroid 추출 + Element graph 변환 모듈
  Step 1.1 [데이터 추출]
    - scout-practical S1 시나리오 — `getSubset(position=CENTROID)` (스크립트 +1줄)
    - element centroid 좌표 + 부피 + element type + 재료 물성 산출
  Step 1.2 [Mesh → element graph 변환 모듈 신규 작성]
    - ~150~200줄
    - 인접 정보: face sharing 기반 (Tri/Quad/Tet/Hex 통일 처리)
    - element label - element node mapping 테이블 활용

Month 2~3: MGN 백본 element graph 위에서 재학습
  Step 2.1 [백본 적응]
    - Encoder/Processor 골격 유지 (baseline 코드 재활용 30~40%)
    - Edge 정의만 face-shared adjacency로 교체
    - Decoder는 element 단위 출력 (von Mises stress + 6 components)
  Step 2.2 [입력 feature 설계]
    - element centroid 좌표, element 부피, element type categorical, 재료 물성
    - 변위 출력은 별도 head 또는 별도 노드 graph 유지 (선택)

Month 4: Ablation 4종
  Step 3.1 [Pooling/aggregation 정당화 갭 (Hestroffer는 정적) 직접 공략]
    - face-only vs face+vertex 인접
    - centroid coordinate vs centroid+volume vs centroid+anisotropic-Jacobian vs centroid+면적-가중
    - 본 논문 Top-20% MAPE / KLD / R² (element-level) 측정
    - **목표: MAPE 7.0% → ≤6.5% (-7%), R²(element) ≥0.78**

Month 5: 대변형 동적 임팩트 검증 (학술 갭 직접 공략)
  Step 4.1 [face sharing 정의 안정화]
    - 시간 시점 분리 학습 또는 reference configuration 기준 face 고정
    - Hestroffer/Storm 정적 baseline 대비 본 논문이 동적에서 검증 시 핵심 contribution
  Step 4.2 [Ablation: A3 vs A1+EQ vs baseline]
    - 동일 학습 데이터 / 동일 epoch 3종 비교

Month 6: CMAME publication draft
  Step 5.1 [paper outline]
    - Title: "Element-Centric Graph Neural Networks for Dynamic Impact Stress Prediction in Production Aluminum Wheels"
    - Target venue: CMAME (high impact, FEM-aware GNN 성숙 venue)
  Step 5.2 [본 논문 / 260502 / 본 추천 1 결과 통합]
    - baseline → 추천 1 (A1+EQ) → 추천 2 (A3 Dual Graph) 의 점진 개선 spectrum 제시
    - Hestroffer/Storm 정적 baseline의 미답 영역 (대변형 동적) 보강이 핵심 contribution

  Step 6 [모델 저장 / 양산 파이프라인 통합]
    - 추론 시간 240~400 ms (-20% vs baseline) — 양산 적용 영향 긍정
    - 평가 metric 직접 호환 → CAE 평가 자동화 파이프라인 단순화
```

### 2.3 성공 지표 (정량)

| 지표 | baseline | 추천 1 (A1+EQ) | 추천 2 (A3 Dual Graph) |
|---|---|---|---|
| Top-20% MAPE | 7.0% | ≤6.5% | **≤6.5% (element-level, baseline 대비 -7%)** |
| KLD | 0.005 | ≤0.005 | **≤0.005 (element 분포 기준)** |
| R² (Top 100%, element-level) | 0.712 (node) | 0.75+ | **≥0.78 (element-level)** |
| 추출 시간 | 1× | 1.1× | 1.1× (S1, centroid) |
| 데이터 부피 | 4 GB | 7.5 GB | ~3.5 GB (centroid-only, 오히려 -15%) |
| 학습 epoch 시간 | 15~25분 | 17~28분 (+10%) | **18~30분 (+20%)** |
| 추론 시간 | 300~500 ms | 300~500 ms | **240~400 ms (-20%, element 수가 노드 수보다 적음)** |
| baseline 재활용율 | — | 85%+ | **30~40%** |

### 2.4 리스크 / 완화책

| 리스크 | 완화책 |
|---|---|
| **R1. baseline 재활용율 낮음 (30~40%, 그래프 구조 차이)** | 모듈화로 처리. Encoder/Processor 골격 유지, Edge 정의만 교체. 학습 루프 / optimizer / scheduling 재활용 |
| **R2. 대변형에서 face sharing 정의 불안정** | 시간 시점 분리 학습 또는 reference configuration 기준 face 고정. 실패 시 정적 가정(impact 초기 단계만) 후퇴 |
| **R3. Pooling/aggregation 정당화 갭 (Hestroffer는 정적)** | Step 3.1 ablation 4종 (centroid only / centroid+volume / centroid+anisotropic-Jacobian / centroid+면적-가중) 으로 본 논문 contribution 확보 |
| **R4. Element 4~8 노드 → element vertex 수 차이 처리 (Tri vs Quad 등)** | element type을 categorical feature로 추가. PyG `scatter` 기반 일반 구현. 본 논문 mesh 혼합 element type 인 경우에도 대응 가능 |
| **R5. 변위 예측 별도 모델 필요 (Gemini 자체 단점)** | 추천 1의 multi-head 구조와 융합 가능 — element graph + node head 동시 운영. 또는 변위는 추천 1 결과 재사용 |
| **R6. 학술 publication review에서 "정적 baseline 대비 동적 검증의 정확도 차이 정량화 부족" 지적** | Step 4.2 의 3종 ablation (A3 vs A1+EQ vs baseline) 으로 동적에서의 정량 차이 명시 |

### 2.5 본 논문 baseline 코드 재활용 비율

- **데이터 추출 스크립트**: 95%+ 재활용 (S1, 1줄 추가)
- **MGN encoder**: 60% (입력 feature 변경 — element centroid 좌표/부피)
- **MGN processor**: 70% (element graph 위에서 동작, edge 정의만 교체)
- **MGN decoder**: 30% (element 단위 출력, MessagePassing layer 일부 신규)
- **Loss / 학습 루프**: 80% (학습 루프 / optimizer / scheduling 재활용)
- **평가 코드**: 60% (element-level metric)
- **그래프 구축 모듈**: **신규** (~150~200줄)
- **종합 재활용율**: **30~40%** (Encoder/Processor 골격 + 학습 루프 재활용. 그래프 구성, MessagePassing layer 일부, Decoder는 신규)

---

## 3. 비추천 대안 (이유 명시)

| 대안 | 점수 | 비추천 사유 |
|---|---|---|
| **A2 Bipartite (단순)** | 28.5 | 직접 학술 baseline 약함 (PyG framework만). A2'(Hypergraph)로 일반화하지 않으면 비용 대비 효용 불충분. 본 논문 baseline 재활용율 60~70% 인데 정확도 우위가 명확하지 않음 |
| **A2' Hypergraph (FEIH-GNN)** | 28.5 | 학술 신규성은 상이지만 **PyG hypergraph 지원 제한 + 자체 구현 비용 + 학습 시간 +50%, 메모리 +70%**. 6개월 로드맵에 무리. 후속 연구 (12개월+) 후보 |
| **A4-C3D8 (IP-as-Node, full integration) ★본 논문 환경 적용 시** | 27.0 | **이론적으로 본 논문 문제 본질("IP→Node 외삽 왜곡")에 가장 직접 대응**이지만, 학습 비용 +150~200%, B=2 → B=1 강제, 메모리 50~60 GB (A100 80GB 임계), 데이터 ~30 GB IO bottleneck — **6개월 로드맵 outside**. **향후 GPU 자원 확장 시 재검토 가능** (장기 후속 연구) |
| ~~A4-C3D8R (IP-as-Node, reduced int.)~~ | ~~37.0~~ | **본 논문 C3D8 사용 확정 (사용자 피드백 2026-05-07) → 환경 미적용**. 감사 추적 보존만 |
| **A5 Edge/Face-attr regression** | 23.0 | MGN paradigm 변경 필요, baseline 재활용율 30%, 실무 호환성 하 (Abaqus 표준 미지원). **후속 연구 (24개월+) 후보** |
| **A7 NN-in-Solver (FEMIN/FEINN)** | 25.0 | 학술적으로 가장 가까운 도메인 (휠 crash) 이지만 **Abaqus user subroutine (UEL/UMAT) 통합 필요 → 양산 도입 부담 큼**. baseline 재활용율 20% 미만. Hyundai 사내 Abaqus subroutine 정책 확인 필요. 단독 추진은 **별도 프로젝트 규모** |
| **A8 Hybrid SPR post-projection** | 28.0 | scout-practical 3.4: "Abaqus 노드 외삽이 이미 유사 averaging 수행 → 사후 SPR 한계 효용 낮음". A1+EQ가 동일 효용을 paradigm 변경 없이 달성 |

---

## 4. 단계적 마이그레이션 경로 (추천 1 → 추천 2 → (선택) 장기)

*(2026-05-07 갱신: 본 논문 C3D8 확정 → A4-C3D8R 분기 제거. A4-C3D8 부분 sampling 학술 검증을 장기 옵션으로 추가)*

```
[Month 0] 본 논문 baseline (Node averaged stress, 변위·응력 노드 출력, MAPE 7.0%, C3D8 full integration 확정)
                |
                | Step A. 데이터: getSubset(CENTROID) 추가 (1줄)
                | Step B. Decoder: Multi-Head 분리 (변위 head + element head)
                | Step C. Pooling: volume-weighted 채택 (ablation 후 확정)
                v
[Month 1~2] A1 (Element-wise Decoding, multi-head, MAPE ≤6.5%)
                |
                | Step D. Equilibrium-loss plug-in (P-DivGNN), w₃ warm-up
                | Step E. Pooling ablation 4종 (avg/max/vol/shape) — contribution 확보
                v
[Month 2~3] **추천 1 완성: A1+EQ (MAPE ≤6.5%, KLD ≤0.005, ISO 3006 직접 호환, baseline 재활용 85~90%)**
                |
                | Step F. ODB centroid 추출 (S1 시나리오, 스크립트 +1줄)
                | Step G. Mesh → element graph 변환 모듈 신규 작성 (~150~200줄, face-shared adjacency)
                | Step H. MGN 백본을 element graph 위에서 재학습 (Encoder/Processor 골격 유지)
                | Step I. 4종 ablation (face-only vs face+vertex / centroid 가중 4종)
                v
[Month 4~6] **추천 2 완성: A3 Dual Graph (Element-as-Node, MAPE ≤6.5% element-level, R²(elem) ≥0.78, 추론 -20%, baseline 재활용 30~40%)**
                |
                | Step J. 대변형 face sharing 정의 안정화 (학술 갭 직접 공략)
                | Step K. CMAME paper draft "Element-Centric GNN for Dynamic Impact Stress Prediction in Production Aluminum Wheels"
                v
[Month 6+] 후속 publication 1~2건 + 본 논문 D1 ablation 통합 + 양산 파이프라인 적용

(선택, 장기 - GPU 자원 확장 시) Month 12+:
  Step L. A4-C3D8 부분 sampling 학술 검증 (전체 데이터셋 X, 일부 샘플로 IP raw 학습)
  Step M. Polycrystal-style ref value paper (정적 baseline 정합성 검증용)
```

**핵심 마이그레이션 원칙** (2026-05-07 갱신):
- 추천 1 → 추천 2 마이그레이션: **encoder/processor 골격 + 학습 루프 재활용**. 그래프 구성·decoder 신규 (재활용 30~40%)
- 추천 1의 multi-head 구조는 A3 Dual Graph 와 융합 가능 (element graph + node head)
- 추천 1이 실패해도 baseline 보존, 추천 2가 실패해도 추천 1 결과는 보존
- **A4-C3D8 (full int.) 은 6개월 로드맵 outside** — 장기 GPU 자원 확장 시 부분 sampling 학술 검증 옵션

---

## 5. 후속 publication 후보 1~2건

### Publication 1: Quick Win 결과 (추천 1 완성 직후)

- **Title**: *Pooling Strategy Matters: A Volume-Weighted Element-Wise Decoder with Equilibrium Loss for MeshGraphNet Stress Prediction on Wheel Impact*
- **Target venue**: JSAE Annual Congress 2026 또는 SAE WCX 2027 (industry-aligned)
- **Contribution**:
  - Pooling 함수 ablation (avg / max / volume-weighted / shape-weighted) — 학술 갭 직접 공략 (scout-academic 5.1)
  - Equilibrium loss plug-in 효과 정량화
  - ISO 3006 평가 호환 element-level 출력 입증
- **추정 분량**: 8~12 pages

### Publication 2: 학술 깊이 (추천 2 완성 직후) — *(2026-05-07 갱신: A4-C3D8R → A3 Dual Graph 로 교체)*

- **Title**: *Element-Centric Graph Neural Networks for Dynamic Impact Stress Prediction in Production Aluminum Wheels*
- **Target venue**: Computer Methods in Applied Mechanics and Engineering (CMAME) 또는 Computational Mechanics
- **Contribution**:
  - Hestroffer 2025 (R²=0.992 polycrystal **정적**) / Storm 2024 (CMAME) 의 정적·소변형 학술 baseline의 명확한 미답 영역인 **대변형 동적 임팩트 + 양산 휠 (200~400k 노드)** 을 직접 공략
  - Element-as-Node 그래프의 face sharing 정의 안정화 알고리즘 (대변형 추적, 학술 갭, scout-academic 5.3)
  - 노드 외삽 vs centroid 라벨 학습 정확도 비교 — 학술 갭 (scout-academic 5.2)
  - Pooling/aggregation 정당화 ablation 4종 (centroid only / +volume / +anisotropic-Jacobian / +면적-가중) — Hestroffer 정적 한계 보강
  - Hyundai 양산 휠 580 샘플 데이터셋 vs Hestroffer/Storm lattice/polycrystal 의 도메인 전이 검증
- **추정 분량**: 20~30 pages, ablation 풍부

### (Optional) Publication 3: 장기 후속 연구 (12~24개월)

- **A. A4-C3D8 부분 sanity check** — Polycrystal-style ref value paper
  - **근거**: 본 논문 C3D8 환경에서 IP raw 학습은 비용 부담 크나 (학습 +150~200%), 일부 샘플로 부분 sampling 학술 검증 가능. 정적 baseline 정합성 검증용
  - **Target venue**: Computational Mechanics
- **B. Hypergraph Stiffness-Mimicking Networks for Mixed-Element FEM Surrogates** (A2'/FEIH-GNN 변형)
  - **Target venue**: Journal of Computational Physics (FEIH-GNN 와 동일 venue)
  - **근거**: 6개월 로드맵 outside

---

## 6. report-author 에게 전달 — *(2026-05-07 갱신)*

### Top 추천 1+1 의 핵심 메시지 (보고서 5장에 강조할 포인트)

**추천 #1: A1+EQ — Element-wise Decoding (Multi-Head, Volume-weighted Pooling) + Equilibrium-loss plug-in**

- 핵심 메시지: *"Gemini 대안 1을 학술적으로 강화한 형태. baseline 코드 85~90% 재활용, ISO 3006 평가 직접 호환, 1~2개월 PoC, 후속 publication 1건 (JSAE/SAE workshop) 가능. 학술 ref 5건 직접 매핑."*

**추천 #2: A3 — Dual Graph (Element-as-Node Graph Neural Network)** *(A4-C3D8R 에서 교체)*

- 핵심 메시지: *"본 논문 C3D8 (full integration) 확정으로 IP-as-Node 직접 학습은 6개월 outside. **A3 Dual Graph 가 C3D8 환경에서 IP 추출 없이도 'Element 위치 응력' 학습이 가능한 차선 경로**. Hestroffer 2025 (R²=0.992 polycrystal 정적) / Storm 2024 (CMAME) 학술 baseline + 본 논문이 대변형 동적 임팩트 + 양산 휠 학술 갭 보강 → CMAME publication 직접 명분. baseline 재활용 30~40%, 추론 시간 -20%."*

### 보고서 Executive Summary 에 반드시 포함할 정량 수치 5건

1. **본 논문 element type 확정: C3D8 (full integration)** — 이로 인해 A4-C3D8R 환경 미적용, A4-C3D8 (full) 6개월 outside
2. **추천 #1 (A1+EQ) 효과**: MAPE 7.0% → ≤6.5% (-7%), baseline 재활용 85~90%, 학습 epoch +10%
3. **추천 #2 (A3 Dual Graph) 효과**: MAPE 7.0% → ≤6.5% (element-level), R²(element) ≥0.78, 추론 시간 -20% (240~400 ms), baseline 재활용 30~40%
4. **학습 시간 영향**: A1+EQ +10% (17~28분), A3 +20% (18~30분) — 모두 A100 단일 GPU 운영 가능
5. **비추천 A4-C3D8 (IP-as-Node, full int.)**: 학습 비용 +150~200%, B=2 → B=1 강제, 메모리 50~60 GB → 6개월 로드맵 outside (장기 GPU 자원 확장 시 재검토)

### 본 논문 baseline 코드 재활용율 (각 추천별)

- **추천 #1 (A1+EQ)**: 데이터 추출 95%+ / encoder 100% / processor 100% / decoder ~50% / loss ~70% / 평가 ~80% → **종합 85~90%**
- **추천 #2 (A3 Dual Graph)**: 데이터 추출 95%+ / encoder 60% / processor 70% / decoder 30% / loss 80% / 평가 60% / 그래프 구축 신규 (~150~200줄) → **종합 30~40%**

### Gemini 답변과의 핵심 차이점 (보고서 4장에 명시할 것)

1. **Gemini 대안 1 → A1+EQ (추천 #1)**: P-DivGNN(Maia 2025) equilibrium loss plug-in 추가, Pooling 함수 학술 ablation 추가. Gemini 본문은 "Multi-Head Decoder만으로 충분" 가정하나 학술적 정당성·신규성 강화
2. **Gemini 대안 3 (Dual Graph) → A3 (추천 #2)**: 정량 학술 근거 강력 (Hestroffer R²=0.992 polycrystal **정적**) 하나 **대변형 동적 거동 학술 갭** — 본 논문이 양산 휠 임팩트 (동적) 에서 검증 시 CMAME publication 직접 명분
3. **Gemini 누락 IP-as-Node (A4)**: 본 논문 C3D8 (full integration) 확정 (사용자 피드백 2026-05-07) → 학습 비용 +150~200%, B=1 강제, IO bottleneck 으로 **6개월 로드맵 outside**. 장기 GPU 자원 확장 시 재검토 가능
4. **Gemini 대안 2 (Bipartite) 는 학술 ref 약함 → A2' (Hypergraph, FEIH-GNN) 로 일반화 권장 (장기 후속 연구)**. 6개월 로드맵 outside
5. **Gemini가 누락한 추가 대안 6건 발굴**: A2' Hypergraph, A4 IP-as-Node (Storm 2024), A6 Equilibrium-loss plug-in (P-DivGNN), A7 NN-in-Solver (FEMIN/FEINN), A8 SPR post-projection, A9 Centroid-only
6. **Gemini가 누락한 실무 정량**: scout-practical 의 Abaqus ODB 추출 시간·부피 정량 (시나리오 S1~S4) — 추천 1의 비용 부담이 "1줄 변경" 임을 정량 입증. Gemini는 정성 비교에 그침

---

## 검증 체크리스트

- [x] 추천 #1 (A1+EQ) 의 1~2개월 시작 가능한 구체적 Step 1~6 정의 *(변경 없음)*
- [x] **추천 #2 (A3 Dual Graph) 의 3~6개월 학술 publication 경로 정의 — A4-C3D8R 에서 교체 (2026-05-07)**
- [x] 두 추천의 baseline 재활용율 정량 (85~90% / **30~40%**)
- [x] 비추천 대안 6건 + 사유 명시 (**A4-C3D8 추가**, A4-C3D8R 환경 미적용 표기)
- [x] 단계적 마이그레이션 경로 (추천 1 → 추천 2 → (선택) 장기 A4-C3D8) 의 자연 진화 정의
- [x] 후속 publication 후보 2건 + venue 명시 (**Pub 2 제목 갱신: Element-Centric GNN for Dynamic Impact**)
- [x] report-author 인계 섹션 (Executive Summary 정량 5건, Gemini 차이 6건) — **C3D8 확정 사실 명시**
- [x] **사용자 피드백 (2026-05-07) 반영: 본 논문 C3D8 (full integration) 확정**
- [x] **A4-C3D8R 환경 미적용 / A4-C3D8 6개월 outside 명시 (감사 추적 보존)**
