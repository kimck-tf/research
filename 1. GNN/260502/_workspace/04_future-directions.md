# 향후 연구 방향 로드맵

> **대상**: Kim·Jin·Zheng (JSAE 2026) 후속 1~3년 연구 방향
> **저자 환경**: 현대자동차 Changgon Kim (현업 엔지니어, 사내 양산 데이터 접근 가능, Hyundai-NVIDIA AI Factory 50K GPU 활용 가능, UC Berkeley 학계 협업)
> **작성**: gap-strategist, 2026-05-04
> **입력**: `03_gap-analysis.md` (5축 갭, SWOT, 우선순위 갭 Top 3) + scout 2건
> **SMART 기준**: 모든 방향에 Specific(무엇을), Measurable(성공 지표), Achievable(현실성), Relevant(왜 지금), Time-bound(예상 기간) 명시

---

## 1. 로드맵 개요

본 로드맵은 본 논문의 5축 갭(데이터/모델/기법/응용/평가) 중 **impact × feasibility 우선순위 Top 3 갭**(시계열 부재 / multi-scale 부재 / UQ 부재)을 단계적으로 해소하면서, 본 논문의 강점(양산 데이터, dynamic impact, domain-aware edge augmentation)을 강화하는 방향으로 설계되었다.

### 단계별 목표

| 단계 | 기간 | 핵심 목표 | 정량 KPI |
|---|---|---|---|
| **단기** | 6~12개월 | 기존 코드베이스 확장 + 즉시 실행 가능 개선 | Top-20% MAPE 7.0% → **5.0% 이하** |
| **중기** | 1~2년 | 아키텍처 수준 개선 (시계열 + multi-scale + UQ) | 시계열 rollout 가능, ECE ≤0.05, MAPE ≤4% |
| **장기** | 2~3년+ | Foundation model + 다부품 + 양산 통합 | 초기 설계 단계 AI-driven design loop, 신차 1건 양산 적용 |

### 단계 간 의존성

```
단기 (T+0~12M)
├── 방향 1: SOTA Baseline 비교 ablation (즉시) ────┐
├── 방향 2: Spectral edge augmentation 비교        ├──→ 중기 (T+12~24M)
├── 방향 3: Foundation model fine-tuning           │      ├── 방향 4: 시계열 GNN 확장
└── 방향 4(중기)·7(장기) 데이터 추출 작업 ──┘     │      ├── 방향 5: Multi-scale GNN
                                                          │      └── 방향 6: UQ + Bayesian GNN
                                                          │
                                                          └──→ 장기 (T+24M+)
                                                                ├── 방향 7: chassis 다부품 확장
                                                                └── 방향 8: 양산 통합 + 설계 최적화
```

---

## 2. 단기 연구 방향 (6~12개월)

### 방향 1: SOTA Baseline 비교 ablation 및 학술적 포지셔닝 강화

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문의 MGN+Edge Aug baseline을 2024-2026 SOTA(Transolver, BSMS-GNN, X-MeshGraphNet, Multiscale-AMR)와 동일 데이터(580 샘플)에서 정량 비교. Random edge augmentation의 spectral baseline (JDR, spectrum-preserving sparsification) 대비 ablation. |
| **Why (왜 지금)** | ① 본 논문이 5년 전 백본(MGN 2021)만 비교하여 SOTA 비교 부재 (`03_gap-analysis.md` §2.5 갭). ② Random edge augmentation의 학술적 정당화 부족 (Tortorella·Micheli 2024 survey가 분류한 spatial rewiring → spectral 정량 기준 미비교). ③ 후속 논문 publication에서 reviewer가 가장 먼저 지적할 항목 — 선제 대응 필요. |
| **How (어떻게)** | (1) Transolver (ICML 2024) GitHub 코드 fork, MGN과 동일 데이터로 학습. (2) BSMS-GNN (ICML 2023) 코드 (Eydcao/BSMS-GNN) 적용 — BFS pooling 휠 spoke 자연 분할. (3) Spectrum-preserving sparsification (ICML 2025) + JDR 코드 적용, 본 논문 random rewiring vs spectral 정량 비교. (4) ablation 차원: (i) Random vs Spectral edge, (ii) Domain-aware vs random, (iii) MGN vs Transolver vs BSMS. |
| **Action Items** | 1. M+0~M+1: Transolver/BSMS-GNN/X-MGN/JDR 코드 환경 구축. 2. M+1~M+3: 본 논문 데이터셋(580 샘플)으로 baseline 학습 + 동일 hyperparameter sweep. 3. M+3~M+5: ablation 결과 정리, MAPE/KLD/inference time/메모리 비교 표 작성. 4. M+5~M+6: 학술 venue (CMAME, JCISE, AIAA SciTech) 워크샵 short paper 투고. |
| **데이터/리소스** | 기존 580 샘플 그대로. GPU: A100 1대 × 1주 (각 baseline 학습). 인력: 1명 fulltime 또는 인턴 1명. |
| **성공 지표 (Measurable)** | (a) 4개 SOTA baseline + 본 논문 method의 MAPE/KLD/inference time 정량 비교 표 완성. (b) 본 논문의 Domain-aware random edge가 spectral 기법 대비 동등 또는 우월 입증 (또는 어떤 시나리오에서 우월한지 명확화). (c) Workshop short paper 1건 투고. |
| **예상 성과** | 본 논문 contribution의 학술적 위치 명확화. 후속 논문 reviewer 대응 데이터 확보. |
| **리스크** | (1) SOTA가 본 논문 method보다 우월할 경우 → 그 경우에도 ablation 자체가 가치 있음, 하이브리드 (SOTA 백본 + 본 논문 augmentation) 새 contribution 가능. (2) 코드 호환성 문제 — Hyundai-NVIDIA 협력으로 PhysicsNeMo 통합 코드 활용 가능. |
| **우선순위** | **최고 (Quick Win)** |
| **기간** | 6개월 |

---

### 방향 2: 데이터 추가 학습 + Foundation Model Fine-tuning

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문이 향후 계획에 명시한 "GPU 자원 한계로 미사용 데이터" — 238 휠 × 745 strength dataset 중 활용한 85 휠 외 미사용분을 추가 학습. 또한 SGUNET (Apple 2025) parameter mapping 또는 Poseidon (NeurIPS 2024) fine-tuning으로 small-data 한계 우회. |
| **Why (왜 지금)** | ① 본 논문 §4 향후 계획 첫 항목 직접 실행 — 가장 우선순위. ② Hyundai-NVIDIA AI Factory 50K GPU 가용성 확보 (2025). ③ SGUNET이 1/16 데이터로 RMSE 11.05% 개선 입증 → 본 논문 580 샘플의 한계 직접 해결. |
| **How (어떻게)** | (1) Hyundai 사내 ODB·PPT 추가 추출 — 기존 Python+Abaqus 스크립트 재사용. (2) 데이터 정제 — 노드 수 다양성, 충격 조건 다양성 유지. (3) SGUNET pretraining: ABC dataset 20,000 시뮬에서 사전학습 → Hyundai 580 + 추가 데이터로 fine-tune. (4) 또는 Poseidon backbone에서 fine-tune — Multiscale operator transformer pretraining 활용. |
| **Action Items** | 1. M+0~M+2: Hyundai 사내 미사용 데이터 추출 (target: 추가 +500 샘플 → 총 1,080 샘플). 2. M+2~M+4: SGUNET/Poseidon 사전학습 모델 다운로드 + Hyundai 데이터 fine-tune 환경 구축. 3. M+4~M+8: fine-tuning 학습 + 비교 (from scratch vs SGUNET ft vs Poseidon ft). 4. M+8~M+12: 결과 정리, JSAE 또는 SAE 후속 paper 투고. |
| **데이터/리소스** | 추가 데이터 +500 샘플 (Hyundai 사내 추출). GPU: A100 8대 × 2주 또는 H100 4대 × 1주. 인력: 1명 fulltime + Hyundai 사내 CAE 엔지니어 협업. |
| **성공 지표 (Measurable)** | (a) 학습 데이터 580 → 1,000+ 샘플 확대. (b) Top-20% MAPE 7.0% → **5.0% 이하** 달성. (c) Fine-tuning이 from scratch 대비 동등 성능에 도달하는 데 필요한 데이터량 정량 (예: 1/8 데이터로 동등). |
| **예상 성과** | 양산 적용 임계값 (≤2-3%)에 한 단계 근접. 본 논문 §4 향후 계획 첫 항목 완수. |
| **리스크** | (1) 추가 데이터 추출에 시간 소요 — Hyundai 사내 우선순위 의존. (2) Foundation model이 solid mechanics 도메인에 적합하지 않을 가능성 — Poseidon은 PDE foundation 일반, solid 특화 부족. → 위험 시 SGUNET 우선 (ABC dataset이 더 mechanics 친화적). |
| **우선순위** | **최고** |
| **기간** | 6~12개월 |

---

### 방향 3: 추론 속도·산업 KPI 정량화 및 Industrial Benchmark 구축

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문이 정성 언급한 "FE 대비 계산 시간 단축"을 정량화 (× 가속 ratio). MAPE 7%가 ISO 3006 인증 시험 합격률과 어떻게 연결되는지 industrial KPI 연계. 본 논문 데이터(보안 해제 가능 부분)를 mini-benchmark로 공개 또는 Hyundai 사내 표준 benchmark로 정착. |
| **Why (왜 지금)** | ① surrogate의 핵심 가치는 가속 ratio — Polycrystal GNN 150×, GINO 26,000×, Neural Concept 1,000× 보고 (`03_gap-analysis.md` §2.5). ② 본 논문 7% MAPE가 양산 임계값 (≤2-3%) 미달이지만 "초기 설계 탐색용" 가치 입증 필요. ③ Crash impact 분야 공개 벤치마크 자체가 부재 — 본 논문이 selective 공개 시 학술 contribution 큼. |
| **How (어떻게)** | (1) 동일 휠에 대해 FE 해석 시간 vs GNN 추론 시간 정밀 측정 (CPU·GPU 환경별). (2) ISO 3006 합격/불합격 분류 정확도 → MAPE → Top-20% MAPE 환산 — confusion matrix 작성. (3) DOE 시나리오 시뮬레이션: surrogate로 100 design variant 평가 → top 5 만 FE 검증 → 전체 process 단축 시간 측정. (4) 보안 검토 후 mini-benchmark 정의 — 익명화된 휠 형상 + 분포 통계만 공개. |
| **Action Items** | 1. M+0~M+2: FE/GNN 시간 측정 protocol 수립, 동일 환경에서 5 휠 × 3 충격 조건 측정. 2. M+2~M+4: ISO 3006 합격 기준과 MAPE의 연계 분석 — Hyundai 사내 합격/불합격 실제 데이터 활용. 3. M+4~M+6: DOE 가속 시나리오 시뮬레이션, Pareto efficiency 평가. 4. M+6~M+10: 보안 검토 + selective benchmark 공개 안 마련. 5. M+10~M+12: SAE 또는 industrial venue paper 투고. |
| **데이터/리소스** | 기존 데이터 + Hyundai 사내 ISO 3006 합격/불합격 결과. 인력: 1명 + Hyundai CAE 엔지니어 협업. |
| **성공 지표 (Measurable)** | (a) GNN 추론이 FE 해석 대비 ≥**100× 가속** 정량 보고 (목표 1,000×). (b) ISO 3006 합격/불합격 분류 정확도 ≥**90%** (Top-20% MAPE 7%로 산출). (c) DOE 100 design 시나리오에서 전체 process 시간 ≥**60% 단축**. (d) Selective public mini-benchmark 1건 공개 (또는 Hyundai 사내 benchmark 정착). |
| **예상 성과** | 본 논문의 산업 가치 정량 입증. 후속 양산 통합 단계의 ROI 근거. 학술 community에 데이터 contribution. |
| **리스크** | (1) Hyundai 데이터 보안 — selective 공개도 사내 승인 필요. (2) 합격/불합격 실제 데이터 부족 — 누적 데이터 부족 시 Monte Carlo 시뮬레이션으로 보완. |
| **우선순위** | **상** |
| **기간** | 9~12개월 |

---

## 3. 중기 연구 방향 (1~2년)

### 방향 4: 시계열 GNN 확장 (Temporal Rollout)

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문의 단일 max-stress 시점 예측을 충격 전체 시계열(t=0~10ms)로 확장. ReGUNet (Pfaff 공저, 2025) 또는 NVIDIA crash dynamics ML(arXiv:2510.15201)의 transient strategy 3종(time-conditional, autoregressive, stability-enhanced AR with rollout-based training)을 본 데이터에 적용. |
| **Why (왜 지금)** | ① 우선순위 갭 #1 (`03_gap-analysis.md` §5) — 시계열 부재가 가장 큰 갭. ② 본 논문 §4 향후 계획에 명시. ③ 동적 충격 surrogate의 본질적 가치 (fatigue, residual stress, impact toughness) 회복. ④ ReGUNet B-pillar 0.74% intrusion 오차로 feasibility 입증. |
| **How (어떻게)** | (1) Hyundai 사내 ODB time history 추출 — 기존 max-stress 시점만이 아닌 t=0~T 전체 frame 저장. (2) ReGUNet (Graph U-Net + Recurrence) 본 데이터에 적용 — spoke 영역 mesh를 temporal sequence로 변환. (3) NVIDIA의 3가지 transient strategy 비교 — time-conditional vs autoregressive vs stability-enhanced AR. (4) noise injection (MGN 원논문 기법) 적용으로 long rollout 안정성 확보. (5) Dynami-CAL momentum conservation hard constraint 통합 (선택적). |
| **Action Items** | 1. M+12~M+15: 시계열 데이터 추출 — ODB 전체 frame 추출 스크립트 개발. 2. M+15~M+18: ReGUNet 코드 fork, 본 데이터에 적용. 3. M+18~M+20: NVIDIA 3가지 strategy 비교 ablation. 4. M+20~M+22: Dynami-CAL momentum constraint 통합 실험. 5. M+22~M+24: NeurIPS workshop 또는 ICLR workshop 투고. |
| **데이터/리소스** | 기존 580 샘플의 시계열 확장 (각 샘플당 ~100 frame → 58,000 frame). GPU: H100 4대 × 4주. 인력: 1명 fulltime + 인턴 1명. |
| **성공 지표 (Measurable)** | (a) t=0~10ms (100 frame) 시계열 예측 가능. (b) Pure rollout (test 시 ground truth feedback 없음) MAPE Top-20% ≤**8%** at t=peak, ≤**12%** at t=10ms. (c) Long-term rollout error 누적 증가율 ≤**0.5% per ms**. (d) FE 대비 시계열 전체 해석 시간 ≥**500× 가속**. |
| **예상 성과** | 본 논문 단일 시점 → 시계열 surrogate로 발전. fatigue·residual stress 분석 가능 → 양산 가치 격상. 학술 인용 가치 큼 (Pfaff 그룹의 ReGUNet 후속에 본 논문이 양산 데이터로 응답). |
| **리스크** | (1) ODB 시계열 추출 비용 — Hyundai 사내 disk 공간 부족 가능. (2) Pure rollout long-term 안정성 — noise injection + AR rollout training 필수. (3) 데이터 양 추가 증가 (×100) — GPU 자원 의존. |
| **우선순위** | **최고 (P1 갭 직접 대응)** |
| **기간** | 12~24개월 (단기 방향 2의 데이터 추출 작업 일부 병행 가능) |

---

### 방향 5: Multi-scale GNN 도입 (Architecture)

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문의 single-resolution flat MGN(N=20)을 multi-scale 구조로 교체. BSMS-GNN (ICML 2023) BFS pooling 또는 Multiscale-AMR (CMAME 2024) phase-field fracture 적용형, X-MeshGraphNet (NVIDIA 2024) partition+halo 중 본 논문 spoke geometry에 가장 적합한 구조 선정. |
| **Why (왜 지금)** | ① 우선순위 갭 #2 (`03_gap-analysis.md` §5) — multi-scale 부재. ② 본 논문 N=20에서 한계 도달, 추가 layer 시 over-smoothing 위험. ③ 200~400k 노드 학습 효율 (BSMS 보고 inference 2.8× 가속, 메모리 절반). ④ 응력 집중 영역(spoke junction)에 fine resolution 적용으로 weighted loss 효과 강화. |
| **How (어떻게)** | (1) BSMS-GNN BFS pooling을 spoke mesh에 적용 — 자연 분할 vs 세분화 비교. (2) Multiscale-AMR (Perera·Agrawal 2024) — 응력 집중 영역에 mesh refine, 비집중 영역 coarsen. transfer learning 까지. (3) X-MeshGraphNet partition+halo — Hyundai-NVIDIA PhysicsNeMo 통합 활용. (4) M4GN 자동 segment learning — 본 논문 spoke 수동 분할의 자동화 가능성 탐색. |
| **Action Items** | 1. M+15~M+18: BSMS-GNN, Multiscale-AMR, X-MGN 3개 구조 본 데이터에 적용 비교. 2. M+18~M+21: 가장 우월한 구조 선정 + weighted loss + edge augmentation 결합 실험. 3. M+21~M+24: M4GN 자동 segment learning 시도 (선택적). 4. M+24: CMAME 또는 JCISE 본 논문 발전형 투고. |
| **데이터/리소스** | 기존 + 추가 데이터 (방향 2 결과). GPU: A100 4대 × 3주. 인력: 1명 fulltime. |
| **성공 지표 (Measurable)** | (a) Multi-scale 구조에서 Top-20% MAPE 7.0% → **5.0% 이하** (단기 방향 2와 시너지). (b) Inference 시간 50% 감소 (BSMS 보고치 수준). (c) GPU 메모리 사용량 50% 감소 → 200~400k 노드 단일 GPU 학습 가능. (d) Layer 수 증가 (N=30, N=50)에서 over-smoothing 회피 입증. |
| **예상 성과** | 본 논문 baseline의 효율성·정확도 동시 개선. 양산 휠 처리 시 industrial scalability 확보. |
| **리스크** | (1) Multi-scale 분할 전략이 spoke geometry에 부적합할 가능성 — M4GN 자동 학습으로 보완. (2) BSMS BFS pooling이 비대칭 spoke에서 안정성 저하 가능. (3) 코드 통합 비용. |
| **우선순위** | **상 (P2)** |
| **기간** | 12~24개월 (방향 1·4와 병행) |

---

### 방향 6: Uncertainty Quantification + Bayesian GNN

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문의 점추정에 UQ 추가. (1) MC Dropout, Deep Ensemble (단순) → (2) Variational Bayesian MGN (FEMIN with VBF, TUM 2024 적용) → (3) Conformal prediction (small-data finite-sample guarantee) 단계적 도입. Calibration 평가 (ECE, sharpness, prediction interval coverage). |
| **Why (왜 지금)** | ① 우선순위 갭 #3 (`03_gap-analysis.md` §5) — UQ 부재. ② 양산 안전 critical 도메인 필수 요건. ③ 비교 매트릭스 7건 모두 UQ 미적용 (`03_gap-analysis.md` §4) → first-mover 가능. ④ FEMIN VBF (TUM 2024)가 동일 motivation으로 검증된 사례. |
| **How (어떻게)** | (1) M+12~M+15: MC Dropout 적용 — 학습 후 dropout 활성화 inference, T=50 sample. Deep Ensemble (5 model)과 비교. (2) M+15~M+18: Variational Bayesian MGN — FEMIN VBF 적용. ELBO loss + KL regularization. (3) M+18~M+21: Conformal prediction — 580 샘플 calibration set, finite-sample coverage guarantee. (4) M+21~M+24: hybrid 운영 시뮬레이션 — 신뢰도 90%↑ 영역만 surrogate, 그 외 FE 재계산. |
| **Action Items** | 1. M+12~M+14: MC Dropout + Deep Ensemble baseline 구현. 2. M+14~M+18: Variational Bayesian MGN 구현 (FEMIN VBF 코드 참조). 3. M+18~M+20: Conformal prediction 적용. 4. M+20~M+22: hybrid (NN+FE) 시뮬레이션 — 양산 시나리오. 5. M+22~M+24: 안전 critical AI venue (ICRA, NeurIPS Safety workshop) 투고. |
| **데이터/리소스** | 기존 + 추가 데이터. GPU: A100 4대 × 4주. 인력: 1명 fulltime + Hyundai 안전 부서 협업. |
| **성공 지표 (Measurable)** | (a) ECE (Expected Calibration Error) ≤**0.05**. (b) 90% Prediction Interval Coverage ≥**85%** on test set. (c) 신뢰도 90%↑ 영역에서 MAPE ≤**3%** (양산 임계값 도달). (d) Hybrid 운영 시 FE 재계산 비율 ≤30% (전체 시간 단축 ≥70%). |
| **예상 성과** | 양산 채택 결정적 차별화. UQ 적용 사례 0건 시장에서 first-mover. ROI 가시화 (hybrid 운영). |
| **리스크** | (1) 580 샘플로 신뢰성 있는 posterior 학습 도전적 — small-data UQ는 conformal prediction 우선. (2) Variational Bayesian 학습 비용 2~5배 — Hyundai-NVIDIA AI Factory 자원 활용. (3) Calibration 평가에 56 test 샘플은 marginal — 추가 데이터 (방향 2) 필수. |
| **우선순위** | **상 (P2, 양산 적용 결정적)** |
| **기간** | 12~24개월 |

---

## 4. 장기 연구 방향 (2~3년+)

### 방향 7: Chassis 다부품 확장 + Multi-physics Foundation

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문의 single-component (wheel) framework을 chassis 다부품 (suspension arm, B-pillar, sub-frame, body-in-white)으로 확장. 동시에 multi-physics (구조 + NVH + crash) foundation model 구축. Hyundai 차종 다양성 활용. |
| **Why (왜 지금)** | ① 본 논문 §4 향후 계획에 명시 — "다른 휠 성능 항목, suspension arm 등 chassis 부품으로 framework 확장". ② Subaru·Mubea·BMW 등 OEM의 multi-component AI 적용 사례 (`02_scout-domain-app.md` §2.4, §2.6). ③ Hyundai-NVIDIA AI Factory 50K GPU로 large-scale pretraining 가능. ④ Foundation model 시대 — Hyundai chassis 도메인 특화 foundation은 학술·산업 양면 가치 큼. |
| **How (어떻게)** | (1) 부품별 데이터 추출 — wheel + suspension arm + B-pillar + sub-frame + body, 각 1,000 샘플 → 총 5,000+ 샘플. (2) Multi-task GNN — shared backbone + component-specific head. (3) Poseidon-style multi-physics pretraining — Hyundai 사내 다도메인 데이터로 fine-tune. (4) Industrial benchmark "Hyundai-Chassis-Bench" 정의. |
| **Action Items** | 1. T+24~T+30: 부품별 데이터 추출 인프라 (단기 방향 2의 wheel 추출 자동화 응용). 2. T+30~T+36: Multi-task GNN 학습 — 부품별 transfer learning 효과 정량. 3. T+36~T+42: Foundation pretraining — Poseidon backbone에서 fine-tune. 4. T+42~T+48: Hyundai-Chassis-Bench 정의 + 사내 standard 정착. 5. NeurIPS / ICLR 본 논문 발전형 투고. |
| **데이터/리소스** | 다부품 데이터 5,000+ 샘플. GPU: H100 16대 × 8주 (foundation pretraining). 인력: 3명 fulltime + Hyundai 다부서 협업. |
| **성공 지표 (Measurable)** | (a) wheel 외 ≥**3개 chassis 부품**에서 surrogate 성능 입증 (각 Top-20% MAPE ≤6%). (b) Pretraining 효과 — fine-tune 시 from scratch 대비 데이터 1/8로 동등 성능. (c) Hyundai 사내 표준 benchmark 1건 정착. (d) 학술 venue 1건 (NeurIPS/ICLR/CMAME) accept. |
| **예상 성과** | Hyundai chassis 도메인 AI 표준화. 본 논문 framework의 industrial impact 극대화. |
| **리스크** | (1) 부품별 데이터 추출 인프라 비용 — Hyundai 사내 다부서 협업 필수. (2) Multi-task GNN의 부품 간 negative transfer 가능 — careful task balancing. (3) 3년 timescale은 Hyundai 사내 우선순위 변동 가능. |
| **우선순위** | **중-상 (장기 핵심)** |
| **기간** | 24~36개월 |

---

### 방향 8: 양산 통합 + AI-driven Wheel Design Loop

| 항목 | 내용 |
|---|---|
| **What (무엇을)** | 본 논문 surrogate를 Hyundai 양산 wheel 개발 프로세스에 직접 통합. (1) FEMIN-style solver-internal integration (Abaqus 내부 NN 호출), (2) Generative design loop (RL Topology Optim, PPO + surrogate reward), (3) AI-driven Wheel Design 1건 양산 적용. |
| **Why (왜 지금)** | ① 본 논문 결론 마지막 — "초기 설계 단계 성능 예측 활용 가능" 실제 실현. ② DLR Crash Box 사례 — surrogate를 design loop에 통합하여 10% 성능 개선 + 12% 무게 감소. ③ Subaru forming 75% 시간 단축 사례 — Hyundai에서도 가능. ④ Hyundai 사내 SDF 비전 (2024) 부합. |
| **How (어떻게)** | (1) FEMIN-style integration — Abaqus subroutine으로 spoke 영역만 NN 대체 (선택 부품). (2) RL Topology Optim (PPO) + 본 논문 surrogate를 reward — 무게 vs 강도 Pareto 탐색. (3) Generative design — diffusion model로 wheel topology 후보 생성 + surrogate로 평가. (4) Hyundai 신차 1건 wheel design에 실제 적용 (PoC → 양산). |
| **Action Items** | 1. T+30~T+36: FEMIN integration 코드 개발 (TUM 협업 가능성 검토). 2. T+36~T+42: RL Topology Optim (PPO) + surrogate reward 환경 구축. 3. T+42~T+48: Generative wheel design 후보 생성. 4. T+48~T+54: Hyundai 신차 1건 wheel 적용 PoC. 5. T+54~T+60: SAE / SAE-Korea 양산 사례 paper 투고. |
| **데이터/리소스** | 모든 누적 데이터 + 신차 wheel 시제품. GPU: H100 32대 (RL training, generative). 인력: 3명 fulltime + Hyundai R&D 본부 + 외부 합작 (TUM, Berkeley). |
| **성공 지표 (Measurable)** | (a) FEMIN integration: Abaqus 호출 시 spoke 영역 NN 자동 호출 가능, FE 대비 ≥**5,000× 가속**. (b) RL Topology Optim: 동일 강도 조건에서 무게 ≥**10% 감소** (DLR Crash Box 수준). (c) Hyundai 신차 wheel 1건 양산 적용 PoC 완료. (d) ROI 분석 — surrogate 도입으로 wheel 개발 사이클 ≥**30% 단축**. |
| **예상 성과** | 본 논문의 산업 가치 최종 입증. Hyundai 사내 AI-driven design culture 정착. 학술적으로 industrial deployment paper로 차별화. |
| **리스크** | (1) Abaqus 라이선스·코드 access 제약 — TUM FEMIN 협업으로 우회. (2) 양산 적용은 인증·법적 절차 (ISO 3006) 필요 — 시간 소요 큼. (3) Hyundai 사내 우선순위 의존 — SDF 비전 일관성 유지 가정. |
| **우선순위** | **장기 핵심 (산업 가치 극대화)** |
| **기간** | 30~60개월 (장기 방향 7과 병행 가능) |

---

## 5. 실행 로드맵 타임라인 (Gantt-style 표)

| 방향 | 0M | 3M | 6M | 9M | 12M | 15M | 18M | 21M | 24M | 30M | 36M | 42M | 48M | 54M | 60M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **D1 SOTA Baseline ablation** | ████ | ████ | ████ | ████ | | | | | | | | | | | |
| **D2 데이터+Foundation FT** | ████ | ████ | ████ | ████ | ████ | | | | | | | | | | |
| **D3 추론·KPI·Benchmark** | | ████ | ████ | ████ | ████ | | | | | | | | | | |
| **D4 시계열 GNN** | | | | ████ | ████ | ████ | ████ | ████ | ████ | | | | | | |
| **D5 Multi-scale GNN** | | | | | ████ | ████ | ████ | ████ | ████ | | | | | | |
| **D6 UQ + Bayesian GNN** | | | | | ████ | ████ | ████ | ████ | ████ | | | | | | |
| **D7 Chassis 다부품** | | | | | | | | | | ████ | ████ | ████ | ████ | | |
| **D8 양산 통합·Design loop** | | | | | | | | | | | ████ | ████ | ████ | ████ | ████ |

**범례**:
- ████ = 활성 작업 기간
- 단기 (D1-D3) → 중기 (D4-D6) → 장기 (D7-D8) 단계적 진행
- D2 (단기) 데이터 추출은 D4 (중기) 시계열 데이터 추출과 부분 병행
- D5 multi-scale은 D4 시계열과 병행 가능

---

## 6. 리소스 / 기술 전제 조건

### 6.1 GPU 리소스

| 단계 | 필요 GPU | Hyundai-NVIDIA AI Factory 활용 가능성 |
|---|---|---|
| 단기 (D1~D3) | A100 1~8대 × 4주 | 충분 (50K GPU 중 일부 할당 가능) |
| 중기 (D4~D6) | H100 4~8대 × 4~8주 | 충분 |
| 장기 (D7~D8) | H100 16~32대 × 8주 | Hyundai 본부 우선순위 결정 필요 |

### 6.2 인력

| 단계 | 인력 구성 |
|---|---|
| 단기 | Changgon Kim (저자) + 인턴/RA 1명 + Berkeley 협업 (Z. Jin, B. Zheng) |
| 중기 | 위 + Hyundai 사내 ML 엔지니어 1명 + 안전 부서 1명 (UQ) |
| 장기 | 위 + 풀타임 PhD 1~2명 + Hyundai R&D 다부서 + TUM 협업 (FEMIN) |

### 6.3 데이터

| 단계 | 데이터 요구 |
|---|---|
| 단기 | 기존 580 샘플 + Hyundai 사내 미사용 데이터 추가 추출 (+500 샘플) |
| 중기 | wheel 시계열 (frame 100×) + 기존 데이터 |
| 장기 | chassis 다부품 (5,000+ 샘플) + 신차 wheel 시제품 |

### 6.4 외부 협업

| 협업처 | 협업 내용 |
|---|---|
| UC Berkeley (Z. Jin, B. Zheng) | 학계 ML 전문성, 본 논문 공저 협업 지속 |
| NVIDIA (PhysicsNeMo) | X-MGN 통합, AI Factory 활용 |
| TUM (FEMIN) | Solver-internal integration 협업 (장기 D8) |
| ETH Zürich (Poseidon CAMLab) | Foundation model fine-tuning 협업 (단기 D2 가능) |
| Apple ML Research (SGUNET) | parameter mapping 기법 (단기 D2) |

### 6.5 학술 Venue 전략

| 단계 | Target Venue |
|---|---|
| 단기 D1 | CMAME, JCISE workshop short paper |
| 단기 D2 | JSAE, SAE Mobilus |
| 단기 D3 | SAE Mobilus, industrial venue |
| 중기 D4 | NeurIPS workshop, ICLR workshop |
| 중기 D5 | CMAME, JCISE |
| 중기 D6 | NeurIPS Safety workshop, ICRA |
| 장기 D7 | NeurIPS / ICLR full paper |
| 장기 D8 | SAE / SAE-Korea industrial paper |

### 6.6 기술 의존성

- **MGN baseline 재현 환경** (PyTorch Geometric) — 이미 확보 가정
- **NVIDIA PhysicsNeMo** (X-MGN, Transolver 통합) — Hyundai-NVIDIA 협력 채널
- **Abaqus 라이선스** — Hyundai 사내 보유
- **Bayesian deep learning frameworks** (Pyro, BlackJAX) — 중기 D6에서 채택
- **FEMIN 코드 access** — TUM 협업 또는 공개 코드 활용

---

## 7. 핵심 우선순위 추천 (즉시 착수)

다음 4건을 **2026년 5월~12월 사이** 동시 착수 권장:

1. **D1 SOTA Baseline ablation** — 후속 논문 reviewer 대응 + 학술 포지셔닝 (Quick Win)
2. **D2 데이터 추가 + Foundation FT** — 본 논문 §4 향후 계획 첫 항목 + Top-20% MAPE 5% 이하 달성
3. **D3 추론·KPI 정량화** — 양산 ROI 가시화
4. **D4 시계열 데이터 추출 (선행 작업)** — 중기 D4 시계열 GNN을 위한 데이터 인프라 우선 구축

이후 12~24개월에 D4(시계열 GNN), D5(Multi-scale), D6(UQ)를 병행 진행하여 우선순위 갭 Top 3 모두 해소.

---

## 8. 후속 논문 후보 주제 (5건)

| # | 제목 (가제) | 타겟 venue | 기여 |
|---|---|---|---|
| 1 | "Spectral Edge Augmentation for Mesh-Based GNN Surrogates: A Comparative Study on Aluminum Wheel Impact" | CMAME 2027 | D1 결과 활용 — random vs spectral edge rewiring |
| 2 | "Foundation Model Fine-tuning for Industrial Solid Mechanics: 1/16 Data Sufficiency on Production Wheel Data" | NeurIPS 2026 workshop | D2 결과 — SGUNET/Poseidon 적용 |
| 3 | "Temporal Rollout for Dynamic Impact Stress Prediction with Recurrent Graph U-Nets" | ICLR 2027 workshop | D4 결과 — ReGUNet 적용 |
| 4 | "Bayesian MeshGraphNets for Safety-Critical Wheel Stress Prediction with Uncertainty Quantification" | NeurIPS 2027 Safety workshop / ICRA | D6 결과 — UQ 적용 |
| 5 | "AI-Driven Wheel Design: Integration of GNN Surrogate, RL Topology Optimization, and Production Deployment" | SAE / NeurIPS Industrial track | D8 결과 — 양산 통합 사례 |

---

**작업 완료** — 본 산출물(`04_future-directions.md`)은 단기 3건 + 중기 3건 + 장기 2건 = 총 8건의 SMART 기준 연구 방향 + 60개월 timeline + 리소스/협업 전제 + 후속 논문 5건을 포함하여 완성. 
두 파일 모두 저장 완료: `_workspace/03_gap-analysis.md`, `_workspace/04_future-directions.md`.
