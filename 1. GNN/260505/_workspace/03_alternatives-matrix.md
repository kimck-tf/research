# 대안 비교 매트릭스

**작성**: analyst-alternatives
**작성일**: 2026-05-06
**입력**: `01_scout-academic.md` (학술 ref 30건), `02_scout-practical.md` (Abaqus/ISO/상용 surrogate)
**대상**: MeshGraphNet 노드/요소 도메인 불일치 해결 대안 9건의 다축 비교

---

## 1. 대안 목록 (Gemini 3 + 추가 6 = 총 9)

### A1. Element-wise Decoding (Gemini 1)

- **핵심 메커니즘**: MGN encoder/processor 유지 → decoder 단계에서 element를 구성하는 노드들의 잠재 vector를 pooling → element MLP로 σ_E 출력. 노드는 변위 head 유지(multi-head).
- **학술 근거** (scout-academic):
  - Maurizi 2022 (2.1.1, Sci.Reports) — MGN 패턴의 stress/strain 예측 baseline
  - Pfaff 2021 (2.1.2, ICLR) — decoder만 교체하는 최소 차분 수정의 출발점
  - Dalton 2024 (2.1.3, CMAME) — 노드 변위 + 요소 응력 multi-head, 본 논문 시나리오와 직접 호환
  - Hernández 2025 (2.1.4) — multi-decoder 분리가 성능 저하 없음 (직접 정량 증거)
- **데이터 요구** (scout-practical):
  - 시나리오 S1 또는 S3 — `getSubset(position=CENTROID)` 1줄 추가
  - 데이터 부피 0.85~1.0× (S1) 또는 1.85× (S3, 노드 변위 head 유지 시)
  - baseline 코드 재활용율 **90%+**
- **변형**:
  - A1-avg (단순 평균), A1-max (집중부 강조), A1-volume-weighted (요소 부피 가중, scout-practical 3.3 — 학술 ablation 갭),
  - A1-shape-weighted (Storm 2024 shape-function-weighted aggregation 인용),
  - A1+EQ (P-DivGNN equilibrium loss `‖div σ‖²` 추가, 학술 신규성 강화).

### A2. Bipartite Graph (Gemini 2)

- **핵심 메커니즘**: PyG HeteroData로 Node·Element 두 종류 vertex 정의. Node→Element / Element→Node 두 단계 message passing. 노드에서 변위, element vertex에서 응력 디코딩.
- **학술 근거** (scout-academic):
  - PyG HeteroData (2.2.4) — 구현 framework
  - Cao 2024 (2.2.2) — Multi-fidelity hetero graph (coarse/fine 노드 두 종류)
  - BSMS-GNN (2.2.5) — bipartite graph determination (multi-scale 변형)
  - **단, Gemini가 정의한 "단순 bipartite"의 직접 학술 ref는 약함** — A2'(Hypergraph)으로 자연 일반화 필요
- **데이터 요구** (scout-practical):
  - 시나리오 S3 — Node + Element centroid 동시 추출 (3~5줄 추가)
  - 데이터 부피 1.85× (양 라벨 동시 보유)
  - baseline 코드 재활용율 **60~70%** (graph 구축 + message passing 분기 추가)
- **변형**:
  - A2-PyG-Hetero (가장 단순 구현), A2-edge-typed (Node-Node / Node-Element edge type 분리).

### A2'. Hypergraph (FEIH-GNN, A2의 학술적 일반화)

- **핵심 메커니즘**: Element가 hyperedge로서 4~8 노드를 묶음. Hypergraph message passing이 local stiffness matrix 계산을 모방. Bipartite의 "한 element = 한 vertex" 추상화보다 element 위상 구조를 정확히 보존.
- **학술 근거**: **Gao, Combescure, Salvi 2024 (FEIH-GNN, 2.2.1, J.Comput.Phys.) — 직접 학술 baseline**. node-element hypergraph + B-matrix 모방 메시지 passing.
- **데이터 요구**: S3와 동일하나, hypergraph 구축에 element-node membership matrix 추가 추출 (mesh connectivity 그대로). **scout-practical는 PyG hypergraph 지원 제한적이라고 지적**.
- **단점**: 자체 구현 비용. PyG `HypergraphConv`는 단순화된 형태만 제공. 본 논문 baseline 코드 재활용율 약 **50~60%**.
- **변형**: A2'-stiffness-mimic (FEIH-GNN의 stiffness matrix 모방 메시지 함수 도입).

### A3. Dual Graph / Element-as-Node (Gemini 3)

- **핵심 메커니즘**: Element를 graph node로, face-shared element끼리 edge. 입력 = centroid 좌표·부피·물성, 출력 = element von Mises. 변위 예측은 별도 모델 필요.
- **학술 근거** (scout-academic):
  - **★ Hestroffer 2025 (2.3.2) — R²=0.992, 150× speedup, 가장 강한 정량 검증**
  - Storm 2024 (2.3.1) — IP-as-node 변형 (A4와 융합 가능)
  - Lu 2025 (2.3.3) — lattice truss 실험 검증, 본 논문 spoke 위상학 유사
  - DG-GNN (2.3.4) — DG-FEM 이론적 근거
- **데이터 요구** (scout-practical):
  - 시나리오 S1 — `position=CENTROID` 단독 (오히려 baseline보다 0.85× 작음)
  - 그래프 구축 신규 코드 ~150~200줄 (face-sharing adjacency)
  - baseline 코드 재활용율 **30~40%**
- **변형**: A3-static (정적 가정, lattice/polycrystal에 검증), A3-dyn (대변형 추적, **본 논문 임팩트 동적 거동에서 학술 갭** — 후속 publication 후보).

### A4. IP-as-Node (Storm 2024 — Gemini 누락 추가 대안)

- **핵심 메커니즘**: Integration point를 그래프 노드로 직접 도입. IP 간 인접관계를 edge로. **외삽·평균 두 단계 smoothing을 원천 차단**.
- **학술 근거**: **Storm, Rocha, van der Meer 2024 (2.3.1/2.4.1, CMAME) — "dual graph by connecting integration points"**. 단일 IP element 가정.
- **데이터 요구** (scout-practical):
  - 시나리오 S2(C3D8) — **확정: 본 논문 C3D8 (full integration) 사용 (사용자 피드백 2026-05-07)**
  - C3D8(full, 8 IP/element): 부피 6~8×, 추출 시간 3~5×, baseline IO bottleneck 위험
  - C3D8R(reduced integration) 시나리오는 **본 논문 환경 미적용** — 회색 처리 (감사 추적 보존)
- **본 논문 baseline 재활용율**:
  - S2(C3D8) 확정: **40~50%** (IP index 처리 추가, 메모리 6~8×)
  - S2'(C3D8R) 가정치는 **본 논문 미적용** (참고용 보존: 70~80%)
- **변형**:
  - **A4-C3D8 (full integration, 본 논문 환경)**: 학술 정합성 가장 우월. 단점 — **학습 비용 +150~200%, B=2 → B=1 강제, 데이터 부피 ~30 GB로 IO bottleneck, 메모리 50~60 GB → A100 80GB 임계, 6개월 로드맵 outside**
  - A4-C3D8R (reduced int.): **본 논문 환경 미적용** (C3D8 사용 확정). 학술 비교용 보존만

### A5. Edge/Face-attribute regression (Gemini 누락)

- **핵심 메커니즘**: 응력 텐서를 element face traction으로 표현, edge/face attribute로 학습. Conservation law(face flux 보존) 강제 가능.
- **학술 근거**:
  - Horie & Mitsume 2024 (FluxGNN, 2.5.1) — face flux + conservation law
  - Praditia 2024 (2.5.2) — FVM 스킴 edge-level 학습
  - Gong & Cheng 2019 (2.5.3) — edge feature foundation
- **본 논문 적합성**: 학술적으로 가장 우아하지만 **MGN 대규모 변경 필요**. Abaqus에서 face-level traction 추출은 비표준(자체 후처리 필요). 본 논문 baseline 재활용율 **30%** 이하.
- **scout-academic 평가**: "본 논문 6개월 로드맵에는 부적합, **후속 연구로 권장**".

### A6. Equilibrium-loss plug-in (P-DivGNN — Gemini의 "Multi-Head" 강화)

- **핵심 메커니즘**: A1 또는 A2 위에 `L += w₃ ‖div σ‖²` (응력 평형방정식 잔차) 추가. 모델 구조는 그대로, loss term만 추가하는 plug-in.
- **학술 근거**: **Maia & van der Meer 2025 (P-DivGNN, 2.6.6, arXiv:2507.05291) — 직접 학술 baseline**. Multi-head decoder + finite strain hyperelasticity equilibrium loss.
- **데이터 요구**: 추가 추출 없음 (loss만 추가). **단, divergence 계산을 위한 element shape function gradient(B-matrix)가 필요** → Abaqus가 직접 노출하지 않음. Gauss-Green 정리로 면적분 우회 가능 [추가 검증 필요].
- **본 논문 baseline 재활용율**: **A1 위에 얹는 plug-in이므로 95%+**.
- **단독 대안 아님**: A1 또는 A2에 부가되는 강화 옵션.

### A7. NN-in-Solver hybrid (FEMIN/FEINN — Gemini 미언급)

- **핵심 메커니즘**: NN을 FEM solver 내부에 통합. 메시 일부 region을 NN으로 대체(FEMIN) 또는 weak form 잔차를 학습 loss에 직접 포함(FEINN, B-matrix 명시 통합).
- **학술 근거**:
  - **Thel 2024 (FEMIN, 2.6.2, CMAME) — Crash 시뮬레이션 8,550× 가속, 본 논문 휠 임팩트와 도메인 일치**
  - Zhang 2024/2025 (FEINN, 2.6.1, CMAME) — B-matrix·shape function·Gauss integration loss 통합, MAPE < 1% vs FEM
  - I-FENN (2.6.4) — damage 변수 NN + FEM solver 결합, 파단 평가 직접 연관
- **본 논문 적합성**:
  - 학술적으로 **본 논문 도메인(휠 임팩트)과 가장 가까움**
  - **단, Abaqus user subroutine (UEL/UMAT) 통합 필요 → 양산 파이프라인 영향 큼**
  - 외부 FE preprocessor (FreeFEM/FEniCS) 의존성 발생 가능
  - baseline 재활용율 **20% 이하** (paradigm 변경)
- **scout-practical 평가**: 양산 도입 가능성 평가 필요 — Hyundai 사내 Abaqus subroutine 정책 확인 필수.

### A8. Hybrid SPR + 학습 (post-projection — Gemini § 4-4 언급, scout-academic 갭)

- **핵심 메커니즘**: A1/baseline 노드 응력을 학습 → 별도 학습 가능한 IP/centroid projection module(SPR-aware MLP) 후처리.
- **학술 근거**:
  - SPR (Zienkiewicz-Zhu 1992) — 정적 baseline
  - **NN + SPR 결합 학술 ref는 단편적 — scout-academic 5절 갭으로 명시**
- **데이터 요구**: 추가 추출 없음. SPR patch fitting 코드 ~200~500줄 추가 (post-script).
- **본 논문 적합성**: **scout-practical 3.4에서 "Abaqus의 노드 외삽이 이미 유사 averaging 수행 → 사후 SPR 한계 효용 낮음"**. baseline 재활용율 80% 가능하나 효용 의문.
- **후속 publication 후보**: 학술 갭이 명확하므로 1~2건 paper 가능.

### A9. (Gemini 미언급 보너스) Centroid-only direct training (가장 단순)

- **핵심 메커니즘**: A1 변형. multi-head 없이, **데이터 라벨만 centroid stress로 교체**. 노드 입력은 그대로, 출력만 element centroid로.
- **학술 근거**: 명시 ref 없음. baseline의 "labeling 변경" 수준 ablation. **scout-practical 5.1 시나리오 S1**에 직접 대응.
- **본 논문 baseline 재활용율**: **95%+** (decoder가 element 단위로 output하도록 1줄 변경, MLP 구조 유지하되 mapping 정의만 변경 — 사실상 A1-avg 의 단순 형태).
- **장점**: 가장 빠른 PoC. D1 ablation의 첫 단계로 적합.
- **단점**: 변위 예측을 잃음 → 본 논문이 변위·응력 동시 출력하므로 **stand-alone 권장 어려움**.

---

## 2. 6축 비교 매트릭스

평가 척도:
- 정확도 추정: 상(+15% MAPE 개선 추정) / 중(+5~10%) / 하(개선 미미 또는 불확실). 학술 ref 보고치 + 유사 도메인 추정.
- 학습 비용: 1~5× MGN baseline (A100 1대, B=2)
- 추론 비용: 1~5× MGN baseline
- 메모리: 1~3× MGN baseline
- 구현 난이도(baseline 재활용율, %): 높을수록 쉬움
- 실무 호환성: 상/중/하 (ODB 추출 + ISO 3006 평가)
- 학술 신규성: 상/중/하 (발표 가능 venue)

종합 점수 = 정확도×1.5 + 실무호환성×1.5 + 학습비용 + 추론비용 + 메모리 + 구현용이성 + 학술신규성. 각 축은 5점 척도 (상=5, 중=3, 하=1, 비용류는 역수 — 1×=5, 2×=4, 3×=3, 4×=2, 5×=1).

| 대안 | 정확도(×1.5) | 학습비용 | 추론비용 | 메모리 | 구현 난이도(재활용율) | 실무 호환성(×1.5) | 학술 신규성 | 종합 점수 |
|---|---|---|---|---|---|---|---|---|
| **A1. Element-wise Decoding** (avg/vol-weighted) | 중상 (5×1.5=7.5) | 1.0~1.1× (5) | 1.0× (5) | 1.0~1.1× (5) | 90%+ (5) | 상 (5×1.5=7.5) | 중 (3) | **38.0** |
| **A1+EQ (Equilibrium-loss plug-in)** | 상 (5×1.5=7.5)* | 1.1× (5) | 1.0× (5) | 1.1× (5) | 85%+ (5) | 상 (5×1.5=7.5) | 상 (5) | **40.0** |
| **A2. Bipartite (PyG HeteroData)** | 중 (3×1.5=4.5) | 1.3× (4) | 1.2× (4) | 1.5× (4) | 60~70% (3) | 중상 (4×1.5=6.0) | 중 (3) | **28.5** |
| **A2'. Hypergraph (FEIH-GNN)** | 중상 (4×1.5=6.0) | 1.5× (3) | 1.3× (4) | 1.7× (3) | 50~60% (3) | 중 (3×1.5=4.5) | 상 (5) | **28.5** |
| **A3. Dual Graph (Element-as-Node)** | 상 (5×1.5=7.5)** | 1.2× (4) | 0.8× (5)*** | 0.9× (5) | 30~40% (2) | 중상 (4×1.5=6.0) | 중상 (4) | **33.5** |
| ~~A4-C3D8R. IP-as-Node (reduced int.)~~ *(본 논문 환경 미적용 — C3D8 사용 확정)* | 상 (5×1.5=7.5) | 1.3× (4) | 1.1× (5) | 1.3× (4) | 70~80% (4) | 상 (5×1.5=7.5) | 상 (5) | ~~37.0~~ (참고용) |
| **A4-C3D8. IP-as-Node (full int.)** ★본 논문 환경 적용 시 | 상 (5×1.5=7.5) | 2.5~3.0× (3) | 2.0× (3) | 2.5× (2) | 40~50% (2) | 중 (3×1.5=4.5) | 상 (5) | **27.0** (6개월 로드맵 outside) |
| **A5. Edge/Face-attr regression** | 중 (3×1.5=4.5) | 2.0× (3) | 1.5× (4) | 2.0× (3) | 30% (2) | 하 (1×1.5=1.5) | 상 (5) | **23.0** |
| **A7. NN-in-Solver (FEMIN/FEINN)** | 상 (5×1.5=7.5) | 3.0× (2) | 0.5× (5)*** | 2.0× (3) | 20% (1) | 하 (1×1.5=1.5) | 상 (5) | **25.0** |
| **A8. Hybrid SPR post-projection** | 하 (1×1.5=1.5) | 1.0× (5) | 1.5× (4) | 1.0× (5) | 80% (4) | 중 (3×1.5=4.5) | 중상 (4) | **28.0** |
| **A9. Centroid-only (단순)** | 중 (3×1.5=4.5) | 1.0× (5) | 1.0× (5) | 1.0× (5) | 95%+ (5) | 상 (5×1.5=7.5) | 하 (1) | **33.0** |

* A1+EQ 정확도: P-DivGNN의 hyperelasticity reconstruction에서 정확도 우위 보고. 본 논문 휠 임팩트 적용 시 **추가 검증 필요**.
** A3 정확도: Hestroffer R²=0.992 (lattice/polycrystal 정적). **대변형 동적 임팩트는 학술 갭 — 동일 정확도 가정에 [추가 검증 필요]** 태그.
*** 추론 비용 < 1× : Dual graph는 element 수가 노드 수와 동급 또는 적어 추론 시간 단축. FEMIN은 NN이 메시 일부만 대체하므로 부분적 가속.
**** A4-C3D8R 실무 호환성: ODB 출력은 IP raw지만 reduced integration이라 element당 1 IP → centroid와 동격, ISO 3006 평가에 직접 투입 가능.

**종합 순위 (Top 5) — 본 논문 C3D8 환경 한정 재산정 (사용자 피드백 2026-05-07)**:
1. **A1+EQ (40.0)** — 학술·실무 균형 최상, baseline 재활용 85%+, 학술 신규성 상
2. **A1 baseline (38.0)** — Quick win 후보, 학술 보수적이나 실무 즉시 적용
3. **A3 Dual Graph (33.5)** ← **새 추천 #2** (C3D8 환경 현실 가능 대안 중 A1+EQ 다음 최상위. Hestroffer/Storm 학술 baseline + 본 논문이 대변형 동적 임팩트 갭 보강 → CMAME publication 직접 명분)
4. **A9 Centroid-only (33.0)** — 가장 단순한 PoC. ablation 첫 단계로 적합
5. **A2 / A2' Bipartite·Hypergraph (28.5)** — 학술 신규성 상이나 비용 대비 효용 부족

*(참고) A4-C3D8R (37.0): C3D8R 가정 시 매트릭스상 3위였으나 본 논문은 C3D8 사용 → 환경 미적용*
*(참고) A4-C3D8 (27.0): 본 논문 환경 적용 가능하나 학습 비용 +150~200%, B=1 강제, IO bottleneck → **6개월 로드맵 outside**, 향후 GPU 자원 확장 시 재검토 가능*

---

## 3. 본 논문 환경 적용 시뮬레이션

본 논문 baseline 추정치 (CONTEXT.md): MGN N=20 layers, H=64, B=2, 노드 200~400k, 580 샘플, A100 1대.
baseline 학습 시간 추정: 1 epoch ≈ **15~25분** (B=2, 200~400k 노드, 200 샘플 기준 가중)
baseline 추론 시간 추정: 단일 휠 forward pass ≈ **300~500 ms**
baseline GPU 메모리: ≈ **20~25 GB** (A100 80GB 중 1/3 사용 가정)

### 3.1 데이터 추출 비용 (580 샘플 × 200~400k 노드)

scout-practical 5.1 시나리오 활용:

| 대안 | 시나리오 | 추출 시간 추가 | 데이터 부피 추가 | 스크립트 수정 |
|---|---|---|---|---|
| A1, A1+EQ, A9 | S1 (centroid 단독) | +10~15% | -15% (3.5 GB) | 1줄 |
| A1 multi-head | S3 (Node + Centroid) | +20% | +85% (~7.5 GB) | 3~5줄 |
| A2, A2', A6 derivative | S3 | +20% | +85% | 3~5줄 |
| A3 | S1 | +10~15% | -15% | 1줄 + adjacency 신규 |
| ~~A4-C3D8R~~ *(본 논문 환경 미적용)* | S2' (IP, reduced) | +50% | 거의 동일 | 5~10줄 |
| **A4-C3D8 (본 논문 환경 적용 시)** | S2 (IP, full) | +300~500% | +600~700% (~30 GB) | 20~50줄 |
| A8 (SPR) | S0 (baseline) + post | 0% / +200~500줄 post | 0% | post-script 신규 |

**핵심 시사점** (사용자 피드백 2026-05-07 반영):
- A1, A9는 데이터 부피·시간 부담 거의 없음 (스크립트 1줄)
- **A4-C3D8R 분기점은 해소됨** — 본 논문 C3D8 사용 확정 → C3D8R 시나리오는 본 논문 미적용
- A4-C3D8 full integration은 ~30GB 추출에 IO bottleneck 확정 → 6개월 로드맵 outside
- **A3 Dual Graph (centroid 기반)** 는 C3D8 환경 무관 → 본 논문 적용 가능, 추천 #2 후보

### 3.2 학습 시간 / 메모리 (A100 1대 기준)

| 대안 | epoch 시간 추정 | GPU 메모리 추정 | 학습 epoch 수 |
|---|---|---|---|
| baseline MGN | 15~25분 | 20~25 GB | 1000~3000 |
| A1, A9 | 15~26분 (decoder 5% 추가) | 20~25 GB | 1000~3000 (수렴 안정) |
| A1+EQ | 17~28분 (loss term 추가, gradient 부담 +10%) | 22~27 GB | 1500~3000 (수렴 약간 느림) |
| A2 (Bipartite) | 20~33분 (+30%, hetero MP) | 30~38 GB (+50%) | 1500~3500 |
| A2' (Hypergraph) | 23~38분 (+50%) | 34~43 GB (+70%) | 2000~4000 |
| A3 (Dual Graph) | 18~30분 (+20%, 노드 수 동급, edge 적음) | 18~23 GB (-10%) | 1500~3000 |
| ~~A4-C3D8R~~ *(본 논문 미적용)* | 20~33분 (+30%) | 26~33 GB (+30%) | 1500~3000 |
| **A4-C3D8 (본 논문 환경)** | 38~75분 (+150%~200%) | 50~60 GB (+150%, B=1 강제) | 2000~4000 (IO bottleneck 확정) |
| A5 | 30~50분 (+100%) | 40~50 GB (+100%) | 2000~5000 |
| A7 (FEMIN) | NN 부분만 학습 시 가벼움 (10~15분), but Abaqus solver loop와 결합 시 epoch 정의 자체 변경 | NN 단독 ~15 GB | 도메인 별 |
| A8 (SPR post) | 학습 영향 없음 | baseline 동일 | 동일 |

**핵심 시사점** (사용자 피드백 2026-05-07 반영):
- **A4-C3D8 (본 논문 환경 확정)** 은 B=2 → B=1 강제 + gradient checkpointing 필요. 50~60 GB 메모리는 A100 80GB 임계 → 6개월 로드맵 outside
- A1+EQ는 메모리·시간 부담이 작아 **baseline 인프라 그대로 활용 가능** (추천 #1)
- A3 Dual Graph는 element 수 ≈ 노드 수로 메모리는 -10%, 학습 시간 +20% (추천 #2 — C3D8 환경 적용 가능)

### 3.3 추론 시간 (단일 휠 forward pass)

| 대안 | 추론 시간 추정 | 비고 |
|---|---|---|
| baseline MGN | 300~500 ms | |
| A1, A9 | 300~500 ms (동일) | decoder 변경만 |
| A1+EQ | 300~500 ms (학습만 영향) | 추론 시 div σ 미평가 |
| A2 | 360~600 ms (+20%) | hetero MP 두 단계 |
| A2' | 390~650 ms (+30%) | hypergraph 메시지 |
| A3 | 240~400 ms (-20%) | element 수 < 노드 수, edge 적음 |
| ~~A4-C3D8R~~ *(본 논문 미적용)* | 330~550 ms (+10%) | IP가 element와 동격 |
| **A4-C3D8 (본 논문 환경)** | 600~1000 ms (+100%) | IP 8× → 그래프 8× |
| A5 | 450~750 ms (+50%) | edge attr 처리 |
| A7 | NN 부분만 < 100 ms; FEM solver loop 포함 시 수 분~수 시간 (paradigm 변경) | |
| A8 | 600~900 ms (+50% post) | SPR patch fitting overhead |

**핵심 시사점**: 모든 GNN-only 대안의 추론 시간은 ms 수준이라 양산 적용 시 차이가 critical하지 않음. **Gemini 추천 대안 1은 추론 시간 변동 없이 평가 호환성 100% 확보** — 가장 효율적.

---

## 4. Gemini 3대안 비판적 평가

### 4.1 강점 / 약점 / 학술 정당화 충분성 / 반증

#### 대안 1 (Element-wise Decoding) — Gemini 추천

- **강점**:
  - 본 논문 baseline 재활용율 90%+ (scout-practical 정량 확인)
  - 학술 ref 4건 (Maurizi 2022, Pfaff 2021, Dalton 2024, Hernández 2025) — 정당화 충분
  - 산업 사례 (Altair physicsAI Multi-Head 패턴) 와 일치
  - Multi-head로 노드 변위 head 보존 → 본 논문 변위 출력 그대로 유지
- **약점 / Gemini의 누락**:
  - **Pooling 함수 선택 정당화 약함** (scout-academic 5.1 갭). avg/max/sum/volume-weighted/shape-weighted 비교 ablation은 학술 문헌 0건. 본 논문이 ablation 추가 시 contribution 후보.
  - Gemini가 "Loss weighting w₁/w₂ 직관 의존" 이라고 자체 인정. 학술적 표준값 부재.
  - 평형방정식 잔차 같은 **물리 prior 부재** → A1+EQ로 보강하는 것이 학술적으로 우월.
- **학술 정당화 충분성**: **상**. Hernández(2025) multi-decoder 정당화 + Dalton(2024) 노드/요소 multi-head 직접 사례.
- **반증 시도**: scout-academic 5.2의 "노드 외삽 응력 vs centroid 응력 라벨 정확도 비교 학술 0건"이라는 갭은 본 논문이 직접 ablation해야 검증 가능. 즉 **A1이 baseline보다 정확하다는 증거는 인접 도메인 추론**에 의존, 직접 비교 데이터는 본 논문이 생산해야 함.

#### 대안 2 (Bipartite Graph)

- **강점**:
  - PyG HeteroData로 즉시 구현 가능
  - 노드/요소 역할 명시 분리 → 학술 신규성 중상
  - 혼합 element type 처리에 유리 (scout-practical: 본 논문 mesh가 혼합인지 미확정)
- **약점**:
  - **단순 bipartite의 직접 학술 ref는 약함**. FEIH-GNN(A2')의 hypergraph가 더 강력 — element 4~8 노드 구성 모델링 시 단순 bipartite는 정보 손실 가능.
  - baseline 재활용율 60~70% — 추가 비용 정당화 미약
  - scout-practical: "데이터 부피 1.85×, 학습 시간 +30%, 메모리 +50%" — 정확도 향상이 충분하지 않으면 비용 대비 효용 부족
- **학술 정당화 충분성**: **중**. PyG framework는 있으나 직접 학술 baseline은 약함. **A2'(Hypergraph, FEIH-GNN)** 으로 자연 일반화 시 정당화 강화.
- **반증 시도**: A1 multi-head로 동일 효용 (노드 변위 + 요소 응력 동시 출력) 확보 가능 → A2의 추가 비용을 정당화하는 핵심 우월점이 명확하지 않음.

#### 대안 3 (Dual Graph / Element-as-Node)

- **강점**:
  - **Hestroffer 2025 R²=0.992, 150× speedup — 정량 학술 검증 가장 강력**
  - 본 논문 spoke 위상학 (lattice 유사) 와 부분 일치 (Lu 2025)
  - 데이터 부피 작음 (centroid 단독, baseline의 0.85×)
  - 추론 시간 -20% (element 수 < 노드 수)
- **약점**:
  - **변위 예측 별도 모델 필요** (Gemini 자체 단점 지적): 본 논문이 변위·응력 동시 출력하므로 dual model 운영 부담
  - **대변형(geometric nonlinearity) 시 element-as-node 좌표 추적 학술 갭** (scout-academic 5.3). Hestroffer/Storm은 정적·미세 변형에서만 검증. 본 논문 휠 임팩트는 동적 거동 — **A3가 본 논문에 직접 이식 가능한지 확신 어려움**.
  - baseline 재활용율 30~40% — graph 구축 ~150~200줄 신규
- **학술 정당화 충분성**: **상** (정적·소변형 한정). 동적 임팩트 도메인은 **학술 갭 → 본 논문이 검증 시 후속 publication 후보**.
- **반증 시도**: 본 논문 mesh의 element 수가 노드 수와 비교 시 어떻게 되는지에 따라 A3의 "추론 가속" 효용이 결정. spoke 영역 hex 메시는 element ≈ node 수준이라 가속 효과는 제한적일 가능성.

### 4.2 Gemini 답변의 핵심 누락

| 누락 항목 | scout 발견 | 보강안 |
|---|---|---|
| Pooling 함수 정당화 | 학술 ablation 0건 (5.1) | A1 변형으로 ablation, contribution 후보 |
| 평형방정식 잔차 plug-in | P-DivGNN (2.6.6) 직접 ref | A1+EQ로 plug-in |
| IP-as-Node | Storm 2024 (2.3.1) 직접 ref | A4 추가 대안, C3D8R 가정 시 실무 가능 |
| Hypergraph 일반화 | FEIH-GNN (2.2.1) 직접 ref | A2'로 A2 일반화 |
| FEMIN/FEINN | 본 논문 휠 임팩트와 도메인 일치 | A7 추가 대안, paradigm 변경 비용 큼 |
| Centroid 추출 비용 | scout-practical S1 1줄 변경 | A1·A3·A9의 데이터 비용 정량 |
| 본 논문 element type (C3D8 vs C3D8R) 영향 | A4의 실현 가능성 핵심 분기점 | **확정 (사용자 피드백 2026-05-07): 본 논문 C3D8 (full integration) 사용**. C3D8R 가정 시 가능했던 IP-as-Node 직접 학습 경로는 비용 부담 (학습 +150~200%, B=1 강제, 메모리 50~60 GB)으로 6개월 로드맵 outside. **A3 Dual Graph (Element-as-Node) 가 C3D8 환경에서 IP 추출 없이도 'Element 위치 응력' 학습이 가능한 차선 경로** |
| ISO 3006 평가 호환성 | element 응력 직접 호환 | A1·A3·A4의 실무 호환성 우위 |

---

## 5. 추천 우선순위 (1차 선별 — 04_recommendation.md 로 이어짐)

**확정 사항 (사용자 피드백 2026-05-07): 본 논문 C3D8 (full integration) 사용** → 이로 인해 A4-C3D8R 시나리오는 본 논문 환경 미적용. A4-C3D8 (full) 은 학습 비용 +150~200% / B=1 강제 / IO bottleneck 으로 6개월 로드맵 outside.

| 순위 | 대안 | 종합 점수 | 선정 사유 |
|---|---|---|---|
| **1차 단기 Quick Win** | **A1+EQ (Element-wise Decoding + Equilibrium Loss)** | **40.0** | baseline 재활용 85%+, 학술 신규성 상, 1~2개월 PoC 가능, scout-academic Top 1 권장 *(변경 없음)* |
| **1차 보수적 fallback** | A1 baseline (avg) | 38.0 | EQ loss 구현 부담 있을 시 fallback. 가장 빠른 실행 |
| **2차 중기 학술 publication** | **A3 Dual Graph (Element-as-Node)** ← **새 추천** | **33.5** | C3D8 환경 현실 가능 대안 중 추천 #1 다음 최상위. Hestroffer 2025 R²=0.992 (polycrystal 정적) + Storm 2024 (CMAME) 학술 baseline + **본 논문이 대변형 동적 임팩트 학술 갭을 채움 → CMAME publication 직접 명분**. Element 단위 출력 → ISO 3006 직접 호환. centroid 추출 → C3D8 영향 없음 |
| 3차 ablation 후보 | A9 Centroid-only | 33.0 | D1 ablation 첫 단계 (baseline vs centroid label 단순 교체) |
| 비추천 (6개월 outside) | A4-C3D8 (IP-as-Node, full int.) | 27.0 | 본 논문 문제 본질에 가장 직접 대응하나 학습 비용 +150~200%, B=1 강제, 메모리 50~60 GB → 6개월 로드맵 outside. **향후 GPU 자원 확장 시 재검토 가능** |
| 비추천 | A5, A7, A8 | <30 | 6개월 로드맵에는 부적합 (paradigm 변경 / SPR 한계 효용 / 학술 갭) |
| ~~참고용 (본 논문 미적용)~~ | ~~A4-C3D8R (IP-as-Node, reduced int.)~~ | ~~37.0~~ | **본 논문 C3D8 사용 확정으로 환경 미적용** — 감사 추적 보존만 |

상세 구현 단계, 성공 지표, 리스크 분석은 `04_recommendation.md` 에서 다룬다.

---

## 검증 체크리스트

- [x] Gemini 3대안 각각의 학술 근거 / 부재 명시 (4.1)
- [x] 추가 대안 6개 정의 (A2', A4, A5, A6, A7, A8, A9)
- [x] 6축 매트릭스가 모든 대안에 대해 채워짐 (2절)
- [x] 본 논문 환경 시뮬레이션 수치 포함 (3절)
- [x] 추천 #1 (단기) 1~2개월 시작 가능한 구체성 (5절, 04로 이어짐)
- [x] 비추천 대안 + 이유 명시 (5절)
- [x] [추가 검증 필요] 태그로 C3D8 vs C3D8R 분기점 보존 (A4 항)
- [x] **사용자 피드백 (2026-05-07) 반영: 본 논문 C3D8 (full integration) 확정**
- [x] A4-C3D8R 행 회색 처리 / "본 논문 환경 미적용" 표기 (감사 추적 보존)
- [x] A4-C3D8 단점 명시 (학습 비용 +150~200%, B=1 강제, IO bottleneck → 6개월 outside)
- [x] 종합 순위 Top 5 재산정 (C3D8 환경 한정): A1+EQ → A1 → A3 → A9 → A2/A2'
- [x] §5 추천 #2 교체: A4-C3D8R → **A3 Dual Graph (Element-as-Node)**
