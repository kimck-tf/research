# PAPER_SUMMARY.md

논문 `ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf` 의 핵심 정리. 시각 정보(그림/표/수식) 포함.

## 서지 정보

- **제목**: Developing of an Geometry-aware AI model for Predicting Stress Distribution in Aluminum Wheels under Impact
- **저자**: Changgon Kim¹ · Zeqing Jin² · Bowen Zheng²
  - ¹ Hyundai Motor Company
  - ² University of California, Berkeley
- **저널**: JSAE (일본자동차공학회) — 2026-04-06 submit
- **키워드**: vehicle development, computer aided engineering, deep learning, stress prediction (B2)

## 1. 연구 배경 (Introduction)

- 휠 강도는 차량 안전과 직결, 13도 충격 강도 시험으로 평가 (Fig. 1: 실차 시험 + 휠 형상 사진)
- 기존 CAE/FE 분석은 시간·계산비용 大 → 다양한 신규 디자인 대응 어려움
- 선행 데이터 표현 방식의 한계 (Fig. 2):
  - (a) 2D image — 강성·충격강도 예측 실패 [Ref 1, 2020 현대 학술대회]
  - (b) 3D Voxel — geometric detail 부족 [Ref 2, 2021 현대 학술대회]
  - (c) 3D Graph (mesh 기반) — 본 연구가 채택
- 선행 GNN 연구 [Ref 3, APL 2023, Jin·Zheng·Kim·Gu] (Fig. 3):
  - 단순 휠 형상에서 von Mises stress R² = 0.965
  - novel mesh connectivity reconstruction 제안
  - 한계: 작은 데이터(3~5k 노드), 단순 형상 → 실제 개발 적용 불가

## 2. 학습 데이터

### 2.1 Graph ↔ Mesh 구조적 유사성 (Fig. 4)

- Graph: nodes + edges (관계 표현)
- Mesh: vertices + 폴리곤 경계 (3D 형상 표현)
- 둘 다 구조적으로 동일 → mesh를 graph로 변환 가능

### 2.2 휠 충격 분석 데이터 (Table 1, Fig. 5, Fig. 6)

| 항목 | Preliminary Study | Current (본 연구) |
|---|---|---|
| Analysis Method | Static Strength | **Dynamic Strength** |
| Analysis Conditions | Simple | Complex |
| Analysis Time | Small | Large |
| Wheel Geometry | Simplified structure | **Production Geometry** |
| Data Size (No. of Nodes) | 3 ~ 5k | **200 ~ 400k** |

- Fig. 5 — (a) 단순 연구용 휠 vs (b) 양산 휠 형상
- Fig. 6 — (a) Static analysis (Static load) vs (b) Dynamic analysis (Barrier dropping & impacting the wheel)
- 실제 개발 업무 표준에 맞춰 dynamic impact 결과를 학습 데이터로 사용

### 2.3 데이터 전처리 (Fig. 7, Fig. 8)

**자동 추출** (Fig. 7):
- 입력원: Analysis Report(PPT) + Result files(ODB)
- 추출 도구: Python + Abaqus scripts (재사용 가능하게 구현)
- 추출 항목:
  - PPT → wheel specification, maximum stress
  - ODB → stress distribution, displacement (max stress 시점의 모든 노드)

**전처리**:
- 238 휠 × 2~4 충격 방향 = 745 strength 데이터셋
- Quadratic element의 mid-node 제외 (graph에 기여 안함)
- Rim 영역 제외, **spoke 영역만 사용** → 데이터 평균 2.4%로 축소
- 본 연구 사용분: 85 휠 → 290 분석 결과 → Random Edge Connection으로 ~2배 증강 → **580 샘플**

**Train/Test 분할** (Fig. 8):
- 휠 형상 중복 없음 (train과 test에 같은 휠 안 들어감)
- Train: **524 샘플 (77 휠)**
- Test: **56 샘플 (8 휠)**
- 각 휠마다 여러 impact load direction (P_A, θ₀~θ₈ 등)이 한 행씩 차지

## 3. 예측 모델 (Section 3)

### 3.1 GNN 아키텍처 (Fig. 9)

**MeshGraphNets [Ref 6, Pfaff et al., ICLR 2021] 기반**:

```
Input                          Encoder        Message Passing    Decoder           Output
-----                          -------        ---------------    -------           ------
Geometric information   ┐
Material properties     ├──→  Latent space → GNN model ────→  Physical fields ──→ 6 stresses
Boundary conditions     ┘                                                        + 3 displacements
                                                                                 (per node)
```

3단계 구조:
- **Encoder** — geometry/BC 특징을 latent space로 매핑
- **Message Passing** — 인접 노드 간 정보 교환으로 representation 업데이트
- **Decoder** — latent → physical quantity 복원

### 3.2 성능 개선 기법

#### (1) Modified Loss Function (Fig. 10)

```
New loss = L₂(prediction, ground truth)_full + γ · L₂(prediction, ground truth)_region
```

- Stress concentration region(spoke junction, notch 등)에 추가 가중치 γ 부여
- 실제 휠 파괴는 국소 지역에서 발생 → 해당 영역 학습 강화
- **결과: 10% 성능 향상**

#### (2) Geometry-aware Edge Reconstruction & Augmentation (Fig. 11)

**문제**: Mesh 밀도 ↑ → message passing 범위 한정 → long-range 정보 캡처 어려움

**선행 방법** (a) Node Connectivity Reconstruction:
- Epsilon ball 안의 모든 노드 사이에 reconstructed connectivity 추가

**본 연구** (b) Improved Node Connectivity Reconstruction:
- Impact area (boundary, striker 직접 접촉부) ↔ Non-impact area (중심부) 구분
- 두 영역에서 **랜덤하게 선택된 노드 사이**에만 추가 엣지 생성
- 효과:
  - boundary의 critical 정보를 long distance로 효율 전달
  - 모든 노드에 적용 안 하므로 계산비용 ↓ → 학습 시간 ↓
  - 랜덤 시드 + 선택 비율 = 하이퍼파라미터 → 다양한 시드로 augmented 데이터 생성 → 일반화 성능 ↑
- **결과: 추가 3% 성능 향상**

### 3.3 최종 예측 성능

- **Top 20% stress region**에서 평가 (Fig. 12: Top 20%/50%/70%/100% 비교 — 20%가 실제 임계 영역)
- **MAPE = 7.0%** (von Mises stress, 연성 재료 파괴 기준에 적합한 지표)
- **KLD = 0.005** (예측 vs 실제 stress 분포 차이, 0에 가까울수록 유사) [Ref 7]
- 8개 미관측 휠 × 충격조건 = 56 테스트 샘플로 검증 (Fig. 13: GT/Prediction/Error contour)

### 하이퍼파라미터 튜닝 (Table 2)

**N=Layers / H=Hidden / B=Batch**

| | N/H/B | R² | MAPE | MAPE Top 20% | KL div |
|---|---|---|---|---|---|
| Baseline | 15/64/2 | 0.700±0.001 | 0.328±0.038 | 0.085±0.001 | 0.105±0.009 |
| Trial 1 | 20/64/2 | 0.698±0.004 | 0.285±0.042 | 0.076±0.013 | 0.078±0.010 |
| Trial 2 | 10/64/2 | 0.696±0.002 | 0.341±0.018 | 0.082±0.005 | 0.082±0.015 |
| Trial 3 | 15/128/2 | 0.702±0.004 | 0.329±0.071 | 0.072±0.008 | 0.078±0.020 |
| Trial 4 | 15/32/2 | 0.685±0.001 | 0.368±0.040 | 0.085±0.004 | 0.122±0.034 |
| Trial 5 | 15/64/4 | 0.684±0.003 | 0.316±0.027 | 0.076±0.002 | 0.110±0.011 |
| Edge Aug 1 | 15/64/2 | 0.691±0.004 | 0.313±0.045 | 0.078±0.002 | 0.078±0.010 |
| Edge Aug 2 | 20/64/2 | 0.693±0.002 | 0.291±0.021 | 0.077±0.001 | 0.072±0.015 |
| **Edge+Data Aug** | **20/64/2** | **0.712±0.001** | **0.270±0.025** | **0.070±0.004** | **0.050±0.009** |

**Optimizer 비교**:

| | Optimizer | LR | R² | MAPE |
|---|---|---|---|---|
| Trial 6 | Adam | 0.001 | 0.676±0.010 | 0.354±0.076 |
| Trial 7 | SGD | 0.001 | 0.660±0.001 | 0.330±0.002 |
| Trial 8 | AdamW | 0.0001 | 0.667±0.001 | 0.353±0.045 |
| Trial 9 | AdamW | 0.01 | 0.674±0.001 | 0.302±0.033 |

**관찰**:
- Layer 수↑, Embedding↑ → 성능 개선 (Trial 1 vs 2, 3 vs 4)
- Batch size 영향 미미 (샘플 수 부족 때문)
- AdamW가 우세 (large-scale 학습에서 일반화 우수)
- Edge augmentation으로 ~3% MAPE 개선
- **Edge + Data 누적 augmentation이 최고 성능** (다양한 시드로 multiple batch 누적)

## 4. 결론 및 향후 계획

**기여**:
- Geometry-aware GNN으로 알루미늄 휠 동적 충격 응력 분포 신뢰성 있게 예측
- FE 분석 대비 계산 시간 대폭 단축, 동시에 high geometric fidelity 유지
- Critical stress concentration 영역 정확 식별
- weighted loss + edge reconstruction/augmentation으로 production-level 정밀 3D 데이터 효과적 처리
- 초기 설계 단계 성능 예측 활용 가능

**향후 계획**:
- GPU 자원 한계로 미사용 데이터 추가 학습 (점진적)
- 향후 양산 휠 분석 데이터로 순차 학습
- 다른 휠 성능 항목, suspension arm 등 chassis 부품으로 framework 확장
- **시계열 GNN**으로 충격 전체 과정 추적 (현재는 max load 시점만)
- 실험 데이터 활용한 GNN 신뢰성 향상

## 참고문헌

1. Kim, C. et al. (2020) — DB-Based Aluminum Wheel Stiffness Performance Prediction Model. Hyundai Motor Group Academic Conf., INT-VN-2020-192
2. Kim, C. et al. (2021) — Data-Driven Wheel Impact Strength Performance Prediction Model. Hyundai Motor Group Academic Conf., INT-DU-2021-039
3. Z. Jin, B. Zheng, C. Kim, G. Gu (2023) — Leveraging GNN and Neural Operator techniques for High-fidelity Mesh-based Physics simulations. APL Mar. 2023
4. Z. Wu et al. (2021) — A Comprehensive Survey on Graph Neural Networks. IEEE TNNLS 32(1), 4-24
5. Zhao, Y. et al. (2024) — A review of GNN Application in Mechanics-related domains. Artif Intell Rev 57, 315
6. Pfaff, T., Fortunato, M., Sanchez-Gonzalez, A., Battaglia, P. (2021) — Learning Mesh-Based Simulation with Graph Networks. ICLR 2021 (**MeshGraphNets**)
7. Shuyi Ji, Zizhao Zhang, Shihui Ying (2022) — Kullback-Leibler Divergence Metric Learning. IEEE Cybernetics 52
