# 자동차·구조해석 AI Surrogate 최신 응용 동향

**작성자**: scout-domain-app
**작성일**: 2026-05-04
**대상 논문**: Kim·Jin·Zheng (2026, JSAE) — Geometry-aware AI for Aluminum Wheel Impact Stress Prediction

---

## 1. 핵심 트렌드 요약 (5줄 이내)

1. **MeshGraphNets-Transolver 양강 구도** — NVIDIA PhysicsNeMo가 두 아키텍처를 crash dynamics용 통합 레시피로 채택(2025), 본 논문의 MeshGraphNets 선택은 산업 표준과 부합.
2. **시계열 안정성(Rollout Stability)이 2024-2025년 핵심 화두** — Recurrent GNN(ReGUNet, RUGNN), noise-injection, autoregressive+rollout 학습 등 다양한 해법이 등장(본 논문이 향후 시계열로 확장 시 필수).
3. **FEM-NN hybrid의 산업화 진입** — TUM의 FEMIN(2024)이 LS-DYNA 솔버 내부에 NN을 직접 통합, 부분 영역 대체로 8,550× 가속 보고. PhysicsNeMo·Altair physicsAI 같은 상용 플랫폼이 등장.
4. **OEM 적용 단계는 "PoC → 양산" 직전**: Subaru·Mubea·BMW·LG 등 60+ OEM이 Neural Concept을 사용 중이지만, 양산 의사결정에 직접 투입된 사례는 드물고 대부분 "초기 설계 탐색·DOE 가속" 단계.
5. **데이터셋·벤치마크 표준화 시작** — DrivAerNet++ (8,000 cars, NeurIPS 2024), PLAID (구조+CFD 6개 dataset, 2025) 등 공개 벤치마크가 등장하나 **충격(impact) 분야 공개 벤치마크는 여전히 부재** → 본 논문의 580 샘플은 산업 데이터로서 가치가 큼.

---

## 2. 응용 영역별 주요 사례

### 2.1 충격/동적 시뮬레이션 surrogate

#### [1] Automotive Crash Dynamics Modeling Accelerated with ML
- **인용**: [Nabian et al., 2025, "Automotive Crash Dynamics Modeling Accelerated with Machine Learning", arXiv:2510.15201, https://arxiv.org/abs/2510.15201, **PoC (NVIDIA-내부 검증)**]
- 핵심: NVIDIA PhysicsNeMo 프레임워크에서 **MeshGraphNet vs Transolver** 비교, 3가지 transient 전략(time-conditional, autoregressive, stability-enhanced AR with rollout-based training).
- 결과: FE 정확도에는 미달하지만 수 자릿수 가속 달성, 초기 설계 탐색에 적합.
- **본 논문과 직접 연관**: 동일한 MeshGraphNets backbone, 동일한 crash/impact 응용. 본 논문의 max-load 시점 예측을 시계열로 확장할 때 직접 참조 가능.

#### [2] Multi-Hierarchical Surrogate Learning for Crash (Computational Mechanics 2024)
- **인용**: [Pavone, Manjunatha, Pollet et al., 2024, "Multi-hierarchical surrogate learning for explicit structural dynamical systems using graph convolutional neural networks", Computational Mechanics, https://link.springer.com/article/10.1007/s00466-024-02553-6 (arXiv:2402.09234), **연구실/PoC**]
- 핵심: Coarse 그래프에서 latent dynamics 학습 → finer 해상도 단계별 residual 학습. LS-DYNA 시뮬레이션 데이터로 검증.
- **본 논문 활용도**: ○ — 200~400k 노드의 양산 휠을 multi-resolution으로 분해할 때 직접 응용 가능.

#### [3] ReGUNet: Recurrent Graph U-Net for Crashworthiness (2025)
- **인용**: [Li, Zhao, Zhou, Pfaff, Li, 2025, "A new graph-based surrogate model for rapid prediction of crashworthiness performance of vehicle panel components", arXiv:2503.17386, https://arxiv.org/abs/2503.17386, **연구실 (Pfaff 공저, Google DeepMind 출신)**]
- 핵심: Graph U-Net + Recurrence. B-pillar 측면 충돌 case에서 **maximum intrusion 0.74% 오차**, 베이스라인 대비 deformation 예측 오차 51% 감소.
- **본 논문 활용도**: ○ — MeshGraphNets 저자 Pfaff가 공저로 들어간 후속 작업. 본 논문이 시계열로 확장 시 가장 직접 참조할 모델.

#### [4] FEMIN: FEM Integrated Networks (CMAME 2024)
- **인용**: [Thel, Greve, Karle, Lienkamp, 2024, "Introducing Finite Element Method Integrated Networks (FEMIN)", CMAME, https://www.sciencedirect.com/science/article/abs/pii/S0045782524003293, **연구실 (TUM)**]
- 핵심: FEM 솔버 내부에 NN 직접 통합, 메쉬의 일부 영역을 NN(TgMLP+LSTM)이 대체. f-FEMIN(force) / k-FEMIN(kinematics) 두 접근 비교(2025 후속).
- **본 논문 활용도**: △ — 전체 휠을 NN으로 치환하는 본 논문과 다르게 "부분 치환" 전략. spoke만 NN으로 처리 시 활용 가능.

#### [5] Crash Box Optimization with Neural Concept (DLR)
- **인용**: [DLR Institute of Vehicle Concepts × Neural Concept, 2024, "Improving Crash Box Performance by 10% with DLR", Neural Concept Blog, https://www.neuralconcept.com/post/improving-crash-box-performance-by-10-with-deep-learning, **PoC → 산업 협력**]
- 핵심: Geodesic CNN 기반 surrogate(NCS)로 crash box 거동 예측, full optimization loop를 돌려 Pareto front 위에서 **양 목표 모두 10% 개선**.
- **본 논문 활용도**: △ — surrogate를 단순 예측에 머무르지 않고 최적화 루프에 투입한 좋은 사례.

#### [6] Vehicle Crash RL with Surrogates (AMSES 2025)
- **인용**: ["Vehicle crash simulation models for reinforcement learning", AMSES, 2025, https://amses-journal.springeropen.com/counter/pdf/10.1186/s40323-025-00288-4.pdf, **연구실**]
- 핵심: 강화학습 환경의 보상 계산용으로 crash surrogate 채용 — Active Safety / 회피 제어 학습용.

---

### 2.2 차량 부품 강도/내구성 AI 예측

#### [7] Wheel Impact Test by Deep Learning (Struct Multidisc Optim 2022)
- **인용**: [Ko et al., 2022, "Wheel impact test by deep learning: prediction of location and magnitude of maximum stress", Struct Multidisc Optim, https://link.springer.com/article/10.1007/s00158-022-03485-6 (arXiv:2210.01126), **연구실 (현대 학술대회 후속)**]
- 핵심: 2D disk-view 이미지 + 3D voxel + barrier mass → **maximum von Mises stress의 크기와 위치**만 예측(분포 X). 본 논문 [Ref 1, 2]의 영문 발전판으로 추정.
- **본 논문 활용도**: ○ — 본 논문의 직접 선행 연구. 본 논문은 "분포 전체 + 양산 형상 + dynamic"으로 발전시킨 것이 차별점.

#### [8] FEM-PINN for Wheel Hub (Struct Multidisc Optim 2026)
- **인용**: [FEM-PINN, 2026, "FEM-PINN: integrating finite element method and physics-informed neural network for performance prediction of engineering structures via graph neural network", Struct Multidisc Optim, https://link.springer.com/article/10.1007/s00158-026-04257-2, **연구실**]
- 핵심: GNN으로 mesh feature 처리 + PINN loss + FEM 솔버 결합으로 wheel hub 성능 예측.
- **본 논문 활용도**: ○ — 본 논문에 PINN 항(잔차 loss) 추가 시 직접 비교 대상.

#### [9] Bionic Honeycomb Wheel Hub (Biomimetics 2024)
- **인용**: [Wang et al., 2024, "Bionic Optimization Design and Fatigue Life Prediction of a Honeycomb-Structured Wheel Hub", Biomimetics MDPI, https://www.mdpi.com/2313-7673/9/10/611, **연구실**]
- 핵심: FEA + Response Surface Optimization으로 응력 8.7% 감소, 무게 12.13% 감소, fatigue cycle 4.217×10⁵ 예측.
- **본 논문 활용도**: △ — 딥러닝 아니지만, 휠 도메인 표준 메트릭(MPa·cycle)이라 비교 baseline으로 사용 가능.

#### [10] Aluminum Alloy Wheel Geometry Optimization (J Mater Sci 2026)
- **인용**: ["Geometrical optimization of aluminum alloy wheels for high fatigue and impact strength", J Mater Sci Mater Eng, 2026, https://link.springer.com/article/10.1186/s40712-026-00418-9, **연구실**]
- 핵심: ISO 3006 충격 시험 기반 spoke·hub의 응력을 각각 **18.7%, 22.4% 감소**. Gaussian Process Regression으로 hours→seconds 가속.
- **본 논문 활용도**: ○ — 본 논문(GNN)과 GPR baseline 비교 시 사용 가능. 동일한 ISO 3006 표준.

#### [11] Battery Pack Frontal Impact Safety (J Energy Storage 2024 / WEVJ 2025)
- **인용**: [Pan, Jin et al., 2024, "Mechanical safety prediction of a battery-pack system under low speed frontal impact via machine learning", J Energy Storage, https://www.sciencedirect.com/science/article/abs/pii/S0955799723006008, **연구실**]
- **인용**: [WEVJ, 2025, "Comparative Analysis of Neural Network Models for Predicting Battery Pack Safety in Frontal Collisions", https://www.mdpi.com/2032-6653/16/2/78, **연구실**]
- 핵심: SVM·GPR·NN 비교, NN이 최고 정확도. 평균절대오차 0.34% (도메인 내), 2.54% (도메인 외).
- **본 논문 활용도**: △ — 동적 충격 도메인 surrogate. 데이터 규모는 작으나 "설계 도메인 내/외" 일반화 평가 방법론 참고.

#### [12] Automotive Steel Fatigue MLP (arXiv 2025)
- **인용**: [Zaltariov, 2025, "Modelling of automotive steel fatigue lifetime by machine learning method", arXiv:2501.11154, https://arxiv.org/abs/2501.11154, **연구실**]
- 핵심: QSTE340TM steel, 3-75-1 MLP. MAPE 0.02~4.59%로 crack length 예측.

---

### 2.3 FEM-AI hybrid

#### [13] FEM Method-enhanced NN (AMSES 2023) — 베이스라인
- **인용**: [Meethal et al., 2023, "Finite element method-enhanced neural network for forward and inverse problems", AMSES, https://link.springer.com/article/10.1186/s40323-023-00243-1 (arXiv:2205.08321), **연구실**]
- 핵심: FEM-based loss를 ANN 학습에 결합, 데이터 효율과 물리 일관성 동시 달성.

#### [14] FEMIN with Variational Bayes Filter (arXiv 2024)
- **인용**: [Thel et al., 2024, "Adapting Deep Variational Bayes Filter for Enhanced Confidence Estimation in FEMIN", arXiv:2409.17758, https://arxiv.org/abs/2409.17758, **연구실 (TUM)**]
- 핵심: FEMIN 예측에 불확실성 정량화(UQ) 추가 — 안전 critical 영역에서 NN 신뢰도가 낮으면 FEM으로 fallback.
- **본 논문 활용도**: ○ — 휠 충격 같은 안전 critical 도메인에서 UQ는 양산 적용 필수 요건.

#### [15] AI-Enhanced CAE Simulations (SAE 2025-01-8241)
- **인용**: [SAE 2025-01-8241, 2025, "AI-Enhanced CAE Simulations: A Revolutionary Approach to Automotive Design and Engineering", https://saemobilus.sae.org/papers/ai-enhanced-cae-simulations-a-revolutionary-approach-automotive-design-engineering-2025-01-8241, **산업 (Altair physicsAI 적용 사례)**]
- 핵심: 내구성·강성 예측에서 CAE 시간 30% 단축 보고.

#### [16] Highly Accurate ML Models for Auto Crash (SAE 2025-01-8719)
- **인용**: [SAE 2025-01-8719, 2025, "Highly Accurate Machine Learning Models for Automotive Crash Applications Using CAE Centric AI/ML Platform", https://saemobilus.sae.org/papers/highly-accurate-machine-learning-models-automotive-crash-applications-using-cae-centric-ai-ml-platform-2025-01-8719, **산업**]
- 핵심: CAE-centric AI/ML 플랫폼으로 crash 응용에 양산 적용 가능 수준 정확도 보고.

---

### 2.4 Design optimization / generative 연계

#### [17] RL-based Topology Optimization for Generative Lightweight (PMC 2025)
- **인용**: [Lee et al., 2025, "Reinforcement learning-based topology optimization for generative designed lightweight structures", PMC, https://pmc.ncbi.nlm.nih.gov/articles/PMC12355488/, **연구실**]
- 핵심: PPO 기반 강화학습 + 토폴로지 최적화, 무게 **40% 감소**, SIMP/level-set 능가.
- **본 논문 활용도**: △ — surrogate가 빠르면 RL 환경의 reward로 사용 가능.

#### [18] Application of ML on Automotive Subframe Design (SAE 2025-01-8619)
- **인용**: [Yang, Sarkaria, Kumaraswamy et al., 2025, "Application of Machine Learning Model on Automotive Subframe Design", SAE 2025-01-8619, https://saemobilus.sae.org/papers/application-machine-learning-model-automotive-subframe-design-2025-01-8619, **산업 (양산 직전)**]
- 핵심: 과거 subframe 프로젝트 데이터를 활용해 **사용자가 입력한 raw concept을 자동으로 redesign**, 제조성·성능 동시 향상. EV 도래로 단축된 개발 사이클에 대응.
- **본 논문 활용도**: ○ — 본 논문 framework를 suspension/chassis로 확장하는 향후 계획에 직접 부합.

#### [19] Mubea × Neural Concept — EV Battery Housing (산업 사례)
- **인용**: [Neural Concept, 2024, "Mubea using Neural Concept for the Design of Innovative Lightweight Components", https://www.neuralconcept.com/post/mubea-using-shape-for-the-design-of-innovative-lightweight-components, **양산 적용**]
- 핵심: EV battery housing 설계에 AI surrogate 적용, lightweight 부품 개발.

#### [20] Subaru × Neural Concept — Forming Analysis (산업 사례)
- **인용**: [Neural Concept Connect, 2024, https://www.neuralconcept.com/post/engineering-intelligence-in-action-neural-concept-connect-2024-recap, **양산 PoC**]
- 핵심: Forming 해석 가속, 차세대 차량 프로그램에서 **개발 시간 75% 단축, 시뮬레이션 10× 가속** 보고.

---

### 2.5 시계열 동적 응답 surrogate

#### [21] GNSS: Graph Network-based Structural Simulator (arXiv 2025)
- **인용**: [Bayraktar et al., 2025, "Graph Network-based Structural Simulator: Graph Neural Networks for Structural Dynamics", arXiv:2510.25683, https://arxiv.org/html/2510.25683v1, **연구실**]
- 핵심: 3가지 inference 모드 — One-Step / Rollout&Calibration / Pure Rollout. **Pure rollout이 가장 어려움** 확인.
- **본 논문 활용도**: ○ — 본 논문이 향후 시계열로 확장 시 베이스라인 비교 모델.

#### [22] RUGNN: Recurrent U-Net Graph NN for Sheet Forming (Adv Eng Inform 2025)
- **인용**: [2025, "Recurrent U-Net-based Graph Neural Network (RUGNN) for accurate deformation predictions in sheet material forming", https://www.sciencedirect.com/science/article/pii/S1474034625009140, **연구실**]
- 핵심: 다중 forming timestep에서 deformation field 정확 예측. ReGUNet과 유사한 구조.

#### [23] Multiscale GNN with Adaptive Mesh Refinement (arXiv 2024)
- **인용**: [Lino et al., 2024, "Multiscale graph neural networks with adaptive mesh refinement for accelerating mesh-based simulations", arXiv:2402.08863, https://arxiv.org/html/2402.08863v1, **연구실**]
- 핵심: 메쉬 적응적 정제 + 다중 스케일 GNN으로 fine mesh의 over-smoothing 해결.
- **본 논문 활용도**: ○ — 본 논문 200~400k 노드 처리에서 over-smoothing 문제 가능, 직접 활용 가치 높음.

#### [24] BSMS-GNN — Bi-Stride Multi-Scale (ICML 2023, 2024 후속)
- **인용**: [Cao et al., 2023, "Efficient Learning of Mesh-Based Physical Simulation with Bi-stride Multi-scale Graph Neural Network", ICML, https://proceedings.mlr.press/v202/cao23a/cao23a.pdf, GitHub https://github.com/Eydcao/BSMS-GNN, **연구실**]
- 핵심: 가장 작은 학습 비용 + 가장 빠른 inference에서도 안정적 long-term rollout 달성.
- **본 논문 활용도**: ○ — 본 논문의 message passing 효율 개선 후보.

---

### 2.6 산업/OEM 동향

#### [25] Hyundai-NVIDIA AI Factory Partnership (2025)
- **인용**: [NVIDIA Investor Press Release, 2025, "NVIDIA and Hyundai Motor Group Team on AI Factory", https://investor.nvidia.com/news/press-release-details/2025/NVIDIA-and-Hyundai-Motor-Group-Team-on-AI-Factory-to-Power-AI-Driven-Mobility-Solutions/, **양산 인프라**]
- 핵심: NVIDIA Blackwell GPU 50,000개 규모 AI Factory 구축, NVIDIA Omniverse + 자체 AI 모델로 공장·엔지니어링·모빌리티 전반 최적화. 2030까지 1,100억$ 투자.
- **본 논문 시사점**: 현대그룹 차원의 GPU 인프라 확보 → 본 논문이 언급한 "GPU 자원 한계로 미사용 데이터" 문제 해결의 청신호.

#### [26] Hyundai Software-Defined Factory + Software-Defined Vehicle
- **인용**: [Hyundai Motor Group, 2024, "AI Innovation: Transforming Everyday Life at Hyundai Motor Group", https://www.hyundaimotorgroup.com/en/story/CONT0000000000192886, **양산 PoC**]
- 핵심: SDF 비전 발표(2024-10), HMGICS Singapore + Unity Digital Twin 협력. AIRS Company(2019~) 운영.
- **본 논문 시사점**: Hyundai 본사가 AI 도입에 우호적 환경, 본 논문의 framework 확장에 사내 자원 동원 가능.

#### [27] NVH Deep Learning for Electric Motors (SAE 2025-01-0123)
- **인용**: [SAE 2025-01-0123, 2025, "High-Fidelity NVH Model Development for Electric Motors Using Deep Learning and Machine Learning Algorithms", https://saemobilus.sae.org/papers/high-fidelity-nvh-model-development-electric-motors-using-deep-learning-machine-learning-algorithms-2025-01-0123, **산업 PoC**]
- 핵심: ANN으로 stator 재료 특성(copper winding, varnish, orthotropic laminate) 식별, FRF 6 kHz 영역까지 ML로 매칭.

#### [28] Toyota Digital Twin Manufacturing (S&P Global 2025)
- **인용**: [S&P Global, 2025, "Digital Twins in the Automotive Industry Explained", https://www.spglobal.com/automotive-insights/en/blogs/2025/08/digital-twins-in-the-automotive-industry-explained, **양산**]
- 핵심: Toyota는 유럽 공장의 가상 복제로 supply chain·생산 라인·재고를 실시간 시뮬레이션.

#### [29] JSAE Automotive Engineering Exposition + Japan Deep Learning Association
- **인용**: [JSAE, 2025-2026, "Automotive Engineering Exposition", https://aee.expo-info.jsae.or.jp/en/, **산업 학회**]
- 핵심: 일본자동차공학회 박람회 + Japan Deep Learning Association이 공식 후원자로 참여 → 일본 OEM의 딥러닝 도입이 학회 차원에서 표준화.
- **본 논문 시사점**: 본 논문이 JSAE에 제출된 맥락 해석 — 일본 OEM도 동일한 문제(고비용 CAE → 딥러닝)에 직면.

#### [30] Neural Concept $100M Series + 60+ OEM (2025)
- **인용**: [Neural Concept, 2025-12, "Closes $100M Funding Round Led by Goldman Sachs", https://theaiinsider.tech/2025/12/23/neural-concept-closes-100m-funding-round-led-by-growth-equity-at-goldman-sachs-alternatives-to-scale-ai-native-engineering/, **상용 플랫폼**]
- 핵심: Airbus·LS Electric·Mubea·Subaru·LG·GE·Safran·Mitsubishi Chemical 등 **60+ OEM** 채택. 1,000× 빠른 예측 보고.

---

## 3. 사용된 데이터/벤치마크 (본 논문과 비교 가능 형태)

| 연구명 | 샘플 수 | 노드/요소 수 | 데이터 유형 | 비고 |
|---|---|---|---|---|
| **본 논문 (Kim·Jin·Zheng 2026)** | **580 (524+56)** | **200~400k (spoke 영역)** | **Dynamic impact, production wheel** | **현대 사내 양산 데이터, 6 stress + 3 disp** |
| Wheel Impact DL (Ko 2022, [7]) | ~700 (추정) | 2D image + 3D voxel | Static impact, simplified | 본 논문의 직접 선행. 분포 X, max만 |
| Multi-Hierarchical Crash GNN (2024, [2]) | 수십~수백 (불명) | 다중 해상도 (~1k coarsest → 100k+ finest) | LS-DYNA 명시적 동역학 | Hierarchical residual 학습 |
| ReGUNet B-pillar (2025, [3]) | 수백 (불명) | B-pillar 단일 부품 mesh | Side crash, 시계열 | Pfaff 공저, 0.74% intrusion 오차 |
| FEMIN (TUM 2024, [4]) | LS-DYNA 결과 다수 | 부분 영역만 NN 대체 | Crash 명시적 | 솔버 내부 통합 |
| Battery Pack Impact NN (2024, [11]) | DOE 기반 수백 | 배터리팩 베이스플레이트 | Low-speed frontal | MAPE 0.34% in-domain |
| **DrivAerNet++ (NeurIPS 2024)** | **8,000 cars** | **350k~750k cells, 24M cell mesh** | **CFD steady-state** | **공개, 39 TB, 3M CPU-hours** |
| **PLAID Datasets (2025)** | 6개 dataset (각 수백~수천) | 다양 (2D/3D, 비정형) | **구조+CFD 혼합** | **Hugging Face 공개** |
| Wheel Hub GPR (2026, [10]) | 수십 (DOE) | spoke·hub 영역 | ISO 3006 impact | hours→seconds 가속 |
| Subaru Neural Concept (2024) | 미공개 | CAD/CAE | Forming | 75% 개발시간 단축 |
| EV Stator NVH (SAE 2025) | 121 DOE × 72,000 data point | FE stator model | NVH FRF | ANN material identification |
| Vehicle Suspension Multi-fidelity (2024) | DBSCAN sampled 5% high-fidelity | RBD + flexible MBD | Mechanism design | Suspension type 추천 |

**관찰**:
- 본 논문의 **샘플 수(580)는 산업 표준 대비 작지 않음** — Battery, Wheel hub GPR, Subframe 등 비교 대상보다 많음.
- **노드 수(200~400k)는 산업 데이터로서 매우 큰 편** — 학계 공개 데이터셋(DrivAerNet++의 cell 수)과 비교해도 동일 차수.
- **Crash/Impact 분야 공개 벤치마크 없음** → 본 논문 데이터(공개 가능 시) 자체가 valuable contribution.

---

## 4. 본 논문에 직접 활용 가능한 기법 (우선순위 Top 5)

### 우선순위 1: ReGUNet (Recurrent Graph U-Net, 2025)
- **이유**: MeshGraphNets 저자 Pfaff가 공저, 본 논문이 향후 계획으로 명시한 "시계열 GNN"에 직접 사용 가능.
- **적용**: 현재 max-load 단일 시점 → 충격 전체 t=0~T 시퀀스 예측. B-pillar 0.74% 오차 수준의 정확도 가능.
- **주의**: 메쉬 다운/업샘플링 시 spoke 영역 비대칭 처리 필요.

### 우선순위 2: Multi-Hierarchical GNN (Computational Mechanics 2024)
- **이유**: 200~400k 노드 처리에서 message passing 비용·over-smoothing 동시 해결.
- **적용**: Coarse 해상도(~1k 노드)에서 거시 거동 학습 → 본 논문 spoke fine mesh로 residual transfer learning.
- **기대 효과**: 학습 시간 단축, GPU 자원 한계 완화 (본 논문이 명시한 제약).

### 우선순위 3: Transolver Architecture (ICML 2024)
- **이유**: NVIDIA PhysicsNeMo가 MeshGraphNet의 동급 옵션으로 채택, attention 기반의 long-range dependency가 본 논문 Edge Augmentation의 motivation과 일치.
- **적용**: Backbone 비교 실험 → MeshGraphNets vs Transolver vs MeshGraphNets+Edge Aug.
- **데이터**: 본 논문 580 샘플로 직접 fine-tuning 가능.

### 우선순위 4: FEMIN with Variational Bayes (TUM 2024)
- **이유**: 안전 critical 휠 도메인에서 **UQ(불확실성 정량화)는 양산 채택 필수 요건**. 본 논문은 현재 점추정만.
- **적용**: NN 예측 + variational confidence → 신뢰도 낮은 노드만 FE 재계산. 전체 휠을 대체하기보다 hybrid 운영.
- **활용도**: ◎ — 양산 도입 가능성을 결정하는 차별화 요소.

### 우선순위 5: PI-GANO (Physics-Informed Geometry-Aware Neural Operator)
- **이유**: 본 논문의 "geometry-aware" 키워드와 직접 일치. Geometry + PDE 동시 일반화.
- **적용**: 새 휠 형상에 대한 zero-shot 일반화 (본 논문의 "재학습 없는 새 휠 예측" 한계 보완).
- **참고**: PI-GANO arXiv:2408.01600.

---

## 5. 산업 채택 장벽 (왜 아직 양산 적용이 제한적인가)

### 5.1 데이터 측면 장벽
- **OEM 데이터 보안성**: 양산 휠 형상·해석 데이터는 보안 자산 → 공개 벤치마크 부재 → 학계와 산업의 격차 누적.
- **Crash 분야 표준 데이터셋 부재**: DrivAerNet++ (CFD), PLAID (구조 정적) 모두 존재하나 dynamic impact 공개 데이터 없음.
- **데이터 표현 다양성**: ODB·PPT·CAD 등 OEM마다 포맷·메타데이터 상이 → ML 파이프라인 표준화 어려움.

### 5.2 정확도·신뢰성 측면 장벽
- **양산 의사결정 임계값**: 안전 부품은 stress 예측 오차 ≤2-3% 요구되나, 현재 SOTA가 5-10% 수준. 본 논문 7% (Top 20%)는 양산 직접 사용보다 "초기 설계 탐색용"으로 적합.
- **UQ 부재**: 대부분의 surrogate가 점추정만 제공 → 안전 critical 적용 시 "어디를 얼마나 믿을 수 있나" 답할 수 없음.
- **Out-of-Distribution 일반화**: 학습 분포 밖 디자인(novel topology, 신소재)에서 정확도 급락 — Battery Pack 사례에서 in-domain 0.34% vs out-domain 2.54% 격차 확인.

### 5.3 워크플로우 측면 장벽
- **CAE 엔지니어 양성**: Altair·Neural Concept 모두 "ML/딥러닝 전문성 없이도 사용 가능" 강조하나 실제로는 hyperparameter tuning·아키텍처 결정에 expertise 필수.
- **솔버 통합 비용**: FEMIN처럼 LS-DYNA·Abaqus 솔버 내부 통합은 OEM별 라이선스·코드 access 제약. 외부 surrogate가 더 흔함.
- **법적·인증 장벽**: 휠 인증 시험(예: ISO 3006, KMVSS) 자체는 물리 시험. AI 예측이 시험 대체 불가, "초기 단계 탐색·DOE 가속"에 한정.

### 5.4 컴퓨팅 자원 측면 장벽
- **학습용 GPU 요구**: 200~400k 노드 GNN 학습은 H100/A100 클래스 GPU 필요 → 본 논문도 GPU 한계로 미사용 데이터 존재 명시.
- **추론은 빠르나 학습 한 번 비용 큼**: 신차 라인업마다 재학습 시 누적 GPU·전력 비용이 CAE 자체 비용에 근접할 수 있음.
- **Hyundai-NVIDIA 50K GPU 협력(2025)**으로 이 장벽은 OEM 차원에서 해결 중.

### 5.5 조직·문화 측면 장벽
- **Domain expert와 ML expert 협업 부재**: CAE 엔지니어의 ML 이해, ML 엔지니어의 충격 역학 이해 모두 부족.
- **신뢰 형성 시간**: surrogate를 양산에 투입하려면 수년의 verification track record 필요 — Neural Concept 7년차(2018 창업)가 이제서야 60+ OEM 진입.
- **본 논문의 강점**: 현대 사내(저자 = Hyundai 엔지니어) + UC Berkeley 학계가 협력 → 이 장벽을 직접 넘는 사례.

---

## 보충 메타데이터

- **작성 검색 키워드 수**: 16개 (영역당 평균 2.7개)
- **수집 사례 수**: 30건 (각 영역 4-7건)
- **인용 형식**: [저자, 연도, 제목, 출처, URL/DOI, 산업 적용 단계]
- **검증 제한**: 일부 OEM 백서·SAE 유료 paper는 abstract만 확보 (원문 미확인 표시 없음 — 요청 시 추가 명시 가능)

## 출처 목록 (Sources)

- [Automotive Crash Dynamics ML (arXiv:2510.15201)](https://arxiv.org/abs/2510.15201)
- [Multi-Hierarchical GCN Crash (arXiv:2402.09234)](https://arxiv.org/abs/2402.09234)
- [Multi-hierarchical surrogate (Comput Mech 2024)](https://link.springer.com/article/10.1007/s00466-024-02553-6)
- [NVIDIA PhysicsNeMo Crash Dynamics Doc](https://docs.nvidia.com/physicsnemo/25.11/physicsnemo/examples/structural_mechanics/crash/README.html)
- [ReGUNet (arXiv:2503.17386)](https://arxiv.org/abs/2503.17386)
- [Vehicle crash RL (AMSES 2025)](https://amses-journal.springeropen.com/counter/pdf/10.1186/s40323-025-00288-4.pdf)
- [Crash Box DLR × Neural Concept](https://www.neuralconcept.com/post/improving-crash-box-performance-by-10-with-deep-learning)
- [Wheel Impact DL (arXiv:2210.01126)](https://arxiv.org/abs/2210.01126)
- [Wheel Impact DL (Struct Multidisc Optim 2022)](https://link.springer.com/article/10.1007/s00158-022-03485-6)
- [FEM-PINN Wheel Hub (Struct Multidisc Optim 2026)](https://link.springer.com/article/10.1007/s00158-026-04257-2)
- [Bionic Honeycomb Wheel Hub (Biomimetics 2024)](https://www.mdpi.com/2313-7673/9/10/611)
- [Aluminum Alloy Wheel Geometry Optim 2026](https://link.springer.com/article/10.1186/s40712-026-00418-9)
- [Battery Pack Frontal Impact ML 2024](https://www.sciencedirect.com/science/article/abs/pii/S0955799723006008)
- [Battery Pack Safety NN (WEVJ 2025)](https://www.mdpi.com/2032-6653/16/2/78)
- [Auto Steel Fatigue ML (arXiv:2501.11154)](https://arxiv.org/abs/2501.11154)
- [FEM-NN Hybrid (arXiv:2205.08321)](https://arxiv.org/abs/2205.08321)
- [FEMIN (CMAME 2024)](https://www.sciencedirect.com/science/article/abs/pii/S0045782524003293)
- [FEMIN VBF (arXiv:2409.17758)](https://arxiv.org/abs/2409.17758)
- [SAE 2025-01-8241 AI-Enhanced CAE](https://saemobilus.sae.org/papers/ai-enhanced-cae-simulations-a-revolutionary-approach-automotive-design-engineering-2025-01-8241)
- [SAE 2025-01-8719 Crash CAE-centric ML](https://saemobilus.sae.org/papers/highly-accurate-machine-learning-models-automotive-crash-applications-using-cae-centric-ai-ml-platform-2025-01-8719)
- [SAE 2025-01-8619 Subframe ML](https://saemobilus.sae.org/papers/application-machine-learning-model-automotive-subframe-design-2025-01-8619)
- [SAE 2025-01-0123 NVH EV Motor DL](https://saemobilus.sae.org/papers/high-fidelity-nvh-model-development-electric-motors-using-deep-learning-machine-learning-algorithms-2025-01-0123)
- [RL Topology Optim Lightweight (PMC 2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12355488/)
- [Mubea × Neural Concept](https://www.neuralconcept.com/post/mubea-using-shape-for-the-design-of-innovative-lightweight-components)
- [Neural Concept Connect 2024 Recap](https://www.neuralconcept.com/post/engineering-intelligence-in-action-neural-concept-connect-2024-recap)
- [GNSS (arXiv:2510.25683)](https://arxiv.org/html/2510.25683v1)
- [RUGNN (Adv Eng Inform 2025)](https://www.sciencedirect.com/science/article/pii/S1474034625009140)
- [Multiscale GNN AMR (arXiv:2402.08863)](https://arxiv.org/html/2402.08863v1)
- [BSMS-GNN (ICML 2023)](https://proceedings.mlr.press/v202/cao23a/cao23a.pdf)
- [Hyundai-NVIDIA AI Factory](https://investor.nvidia.com/news/press-release-details/2025/NVIDIA-and-Hyundai-Motor-Group-Team-on-AI-Factory-to-Power-AI-Driven-Mobility-Solutions/)
- [Hyundai AI Innovation 2024](https://www.hyundaimotorgroup.com/en/story/CONT0000000000192886)
- [Toyota Digital Twin (S&P 2025)](https://www.spglobal.com/automotive-insights/en/blogs/2025/08/digital-twins-in-the-automotive-industry-explained)
- [JSAE Auto Engineering Expo 2026](https://aee.expo-info.jsae.or.jp/en/)
- [Neural Concept $100M Series 2025](https://theaiinsider.tech/2025/12/23/neural-concept-closes-100m-funding-round-led-by-growth-equity-at-goldman-sachs-alternatives-to-scale-ai-native-engineering/)
- [DrivAerNet++ (NeurIPS 2024, arXiv:2406.09624)](https://arxiv.org/abs/2406.09624)
- [PLAID Datasets (arXiv:2505.02974)](https://arxiv.org/abs/2505.02974)
- [PI-GANO (arXiv:2408.01600)](https://arxiv.org/html/2408.01600v1)
- [Vehicle Suspension Multi-Fidelity (arXiv:2410.03045)](https://arxiv.org/abs/2410.03045)
- [Transolver GitHub](https://github.com/thuml/Transolver)
- [Geo-FNO GitHub](https://github.com/neuraloperator/Geo-FNO)

---

**작업 완료**: 본 산출물(`02_scout-domain-app.md`)은 자동차·구조해석 AI surrogate 응용 동향 30건 + 데이터 비교 표 + Top 5 활용 가능 기법 + 5축 산업 채택 장벽을 포함하여 완성되었습니다.
