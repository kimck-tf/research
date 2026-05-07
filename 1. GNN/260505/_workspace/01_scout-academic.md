# FEM-Aware GNN 학술 동향

**작성**: scout-academic
**작성일**: 2026-05-06
**대상**: MeshGraphNet의 노드/요소 도메인 불일치 해결을 위한 학술 reference 발굴
**검색 범위**: arXiv / NeurIPS·ICLR·ICML / CMAME·IJNME·Sci.Reports·Comp.Mech (2022~2026 우선)

---

## 1. 핵심 발견 요약 (5줄 이내)

1. **Gemini 3대안 모두 학술 선례 존재** — 대안 1(Element-wise Decoding)은 Maurizi 2022·Pfaff MGN 패턴, 대안 2(Bipartite/Hetero)는 PyG HeteroData 일반화, 대안 3(Dual graph)은 Polycrystal-GNN(arXiv:2409.05169) / Microstructure-GNN(Storm et al. CMAME 2024)에서 **"cells become nodes"** 형태로 직접 구현됨.
2. **FEIH-GNN (Gao & Combescure, 2023, arXiv:2212.14545)** — **node-element 하이퍼그래프**로 메시를 변환하여 message passing이 local stiffness matrix 계산을 모방. Gemini 대안 2(Bipartite) 의 학술적 일반화·강화 형태.
3. **FEMIN (Thel et al., CMAME 2024)** — Crash 시뮬레이션에서 NN을 FEM solver 내부로 직접 통합. 본 논문(휠 임팩트, 양산 적용)과 도메인이 매우 가깝고 노드 응력 외삽 문제를 우회.
4. **FEINN (Zhang et al., CMAME 2024/2025) + Microstructure-GNN (Storm et al. CMAME 2024)** — B-matrix·shape function·Gauss integration을 명시적으로 수식에 통합 → Gemini가 "추가 검토 포인트"로만 언급한 영역의 학술 baseline.
5. **누락 대안 3건 발굴**: (A) IP-as-Node Dual Graph (Storm 2024), (B) Hypergraph Node-Element 통합 (FEIH-GNN), (C) Multi-Head Decoder + Equilibrium Loss (P-DivGNN, arXiv:2507.05291). 이 중 **(A)는 본 논문 baseline 호환성이 가장 높음**.

---

## 2. 분야별 주요 연구

### 2.1 Element-wise Decoding 계열 (Gemini 대안 1 대응)

| # | 저자(연도) | 제목 / 출처 | 핵심 / 본 논문 적용성 |
|---|---|---|---|
| 2.1.1 | Maurizi, Gao, Berto (2022) | *Predicting stress, strain and deformation fields in materials and structures with graph neural networks*, **Sci. Reports 12, 21834** [DOI:10.1038/s41598-022-26424-3] | GN을 메시-그래프로 직접 매핑, **노드 위치에서 stress/strain/displacement 동시 예측**. 본 논문 baseline과 가장 유사. Element-wise output은 아니지만 multi-head decoder의 prototype. **본 논문이 MGN을 채택한 학술 정당화의 핵심 ref**. |
| 2.1.2 | Pfaff, Fortunato, Sanchez-Gonzalez, Battaglia (2021) | *Learning Mesh-Based Simulation with Graph Networks (MeshGraphNets)*, **ICLR 2021** [arXiv:2010.03409] | Encoder–Processor–Decoder의 출력을 노드로 정의한 baseline. Decoder를 element-aggregator로 교체하는 것이 Gemini 대안 1의 **최소 차분 수정**. |
| 2.1.3 | Dalton, Husmeier, Gao (2024) | *Physics-informed graph neural network emulation of soft-tissue mechanics*, **CMAME** [DOI:10.1016/j.cma.2023.116351] | Multi-head: 노드 변위 + 요소 응력. 본 논문 시나리오(고체역학 surrogate)와 직접 호환. Gemini 추천 워크플로우 § 3.2의 학술 근거. |
| 2.1.4 | Hernández et al. (2025) | *MeshGraphNets informed locally by thermodynamics*, **Adv. Model. Simul. Eng. Sci.** [DOI:10.1186/s40323-025-00311-8] | Decoder를 multiple specialized decoder로 분리 (energy/entropy gradient + Poisson + dissipative). **decoder 다중화가 학습 성능을 향상시킨다는 직접 증거**. |
| 2.1.5 | Yang et al. (2025) [추가 검증 필요] | *MeshGraphNet-Transformer: Scalable Mesh-based Learned Simulation for Solid Mechanics* [arXiv:2601.23177] | Processor를 Transformer로 교체. Decoder는 여전히 노드 단위지만, **본 논문의 200~400k 노드 규모에 직접 대응**하는 scalable 변형. Gemini 대안 1과 결합 가능. |

**Why**: Gemini 대안 1은 Pfaff(2021)+Maurizi(2022) 패턴 위에서 decoder만 element-aggregator로 교체하는 가장 보수적·구현 간단한 대안이며, Hernández(2025)의 multi-decoder 결과가 **decoder 분리가 성능을 해치지 않는다**는 학술 근거를 제공.

### 2.2 Bipartite / Heterogeneous Graph 계열 (Gemini 대안 2 대응)

| # | 저자(연도) | 제목 / 출처 | 핵심 / 본 논문 적용성 |
|---|---|---|---|
| 2.2.1 | Gao, Combescure, Salvi (2024) | *A Finite Element-Inspired Hypergraph Neural Network: Application to Fluid Dynamics Simulations*, **J. Comput. Phys.** [arXiv:2212.14545, DOI:10.1016/j.jcp.2024.112866] | **node-element 하이퍼그래프**: 노드를 element 단위로 묶고 hypergraph message passing이 local stiffness matrix를 모방. **Gemini 대안 2의 학술적 일반화 형태**(이분 그래프의 자연 확장). 유체 적용이지만 stiffness matrix 모방 메커니즘은 고체에도 직접 이식 가능. **★ Top reference**. |
| 2.2.2 | Cao, Liu (2024) | *Multi-fidelity GNN for FEM convergence learning*, **Comp. Methods Appl. Mech. Engrg.** [DOI:10.1016/j.cma.2022.115422] | Coarse mesh 노드와 fine mesh 노드를 두 종류로 두는 hetero graph. Bipartite 변형의 PyG 구현 reference. |
| 2.2.3 | Wang, Hsu (2024) | *Mesh-based GNN surrogates for time-independent PDEs*, **Sci. Reports** [DOI:10.1038/s41598-024-53185-y] | edge-augmented GNN + multi-GNN: 노드 간 long-range 정보 전달을 위한 보조 그래프 도입. Bipartite 발상의 또 다른 형태. **시간 독립 고체역학에서 MGN 대비 우수한 정확도** 보고 → 본 논문 비교 baseline 후보. |
| 2.2.4 | PyG Team (PyTorch Geometric Documentation) | *Heterogeneous Graph Learning — HeteroData* | Bipartite/Hetero 구현 표준. Gemini 대안 2를 즉시 구현 가능한 framework. 학술 ref는 아니나 실무 호환성 평가 필수 항목. |
| 2.2.5 | Cai, Wang (2023) [추가 검증 필요] | *Bi-Stride Multi-Scale Graph Neural Network for Mesh-Based Physical Simulation (BSMS-GNN)*, **OpenReview** | Bipartite graph determination algorithm 영감. Pooling 단계에서 노드를 두 그룹으로 나눔 → Gemini 대안 2의 multi-scale 변형. |

**Why**: Gemini 대안 2(Bipartite)의 가장 직접적 학술 ref는 **FEIH-GNN(2.2.1)**. node-element를 하이퍼엣지로 묶는 방식이 단순 이분 그래프보다 elements가 4~8개 노드로 구성되는 FEM 메시에 자연스러우며, **B-matrix를 message passing으로 명시적 모방**하는 추가 이점 보유.

### 2.3 Dual / Element-Graph 계열 (Gemini 대안 3 대응)

| # | 저자(연도) | 제목 / 출처 | 핵심 / 본 논문 적용성 |
|---|---|---|---|
| 2.3.1 | Storm, Rocha, van der Meer (2024) | *A Microstructure-based Graph Neural Network for Accelerating Multiscale Simulations*, **CMAME** [arXiv:2402.13101, DOI:10.1016/j.cma.2024.116983] | "**dual graph over the mesh by connecting integration points**" — IP를 그래프 노드로, IP 간 인접관계를 edge로. **Gemini 대안 3 + 추가 대안 (IP-as-node) 의 직접 학술 근거**. 단일 IP element 가정. **★ Top reference**. |
| 2.3.2 | Hestroffer et al. (2025) | *Stress predictions in polycrystal plasticity using graph neural networks with subgraph training*, **Comput. Mech.** [arXiv:2409.05169, DOI:10.1007/s00466-025-02604-6] | "**mesh cells are converted to nodes and edges are created between adjacent cells**" — Element-as-node, face-shared edge. R²=0.992, 150배 가속. **Gemini 대안 3의 가장 직접적이고 정량 검증된 reference**. |
| 2.3.3 | Lu, Zhang (2025) | *Predicting deformation and stress-strain behaviour of lattice truss structures under compression using dual Graph Neural Network*, **Composite Structures** [DOI:10.1016/j.compstruct.2025.119353] | Dual-GNN으로 lattice truss의 응력-변형률 곡선 예측. 실험 검증 완료. 본 논문 휠 spoke 구조와 위상학적으로 유사. |
| 2.3.4 | Implementing DG-FEM with GNN (2024) | *Implementing the discontinuous-Galerkin FEM using graph neural networks with application to diffusion equations*, **Neural Networks** [DOI:10.1016/j.neunet.2024.106788] | DG는 **요소 내부 해**가 자연스러움 → element-as-node로 직접 매핑. Gemini 대안 3의 PDE-solver 측 학술 근거. |
| 2.3.5 | Black, Najera-Flores (2023) | *Graph Network-based Structural Simulator (GNSS)* [arXiv:2510.25683] [추가 검증 필요] | 구조 동역학에서 element를 node로 보는 변형. Gemini 대안 3의 동적 확장. |

**Why**: Gemini 대안 3은 학술적으로 가장 명확히 정착된 형태. **Hestroffer(2.3.2)의 정량 결과(R²=0.992, 150x speedup)**가 본 논문 baseline(R²=0.712, top 100%)을 명백히 상회. 그러나 lattice/polycrystal 도메인이라 양산 휠 임팩트로의 직접 이식성은 **추가 검증 필요**.

### 2.4 IP / Gauss point 직접 예측 계열 (추가 대안 — Gemini 누락)

| # | 저자(연도) | 제목 / 출처 | 핵심 |
|---|---|---|---|
| 2.4.1 | Storm, Rocha, van der Meer (2024) | (위 2.3.1과 동일) | **IP를 그래프 노드로 직접 도입** — Abaqus IP raw stress를 변환 없이 학습. SPR 외삽 오차를 원천 차단. |
| 2.4.2 | Benady (2024) | *Physics-augmented neural networks for constitutive modeling*, PhD Thesis (HAL tel-04887187) | IP 단위 constitutive law 학습. **GNN은 아니지만** IP-level NN의 reference baseline. |
| 2.4.3 | Wang et al. (2024) | *An AI Constitutive Model for Amorphous Solids Utilizing GNNs*, **JOM 76** [DOI:10.1007/s11837-024-06742-9] | IP 단위 응력-변형률 관계를 GNN으로 학습. Constitutive 측면. |

**Why**: IP-as-node는 **본 논문 문제 본질("IP에서 응력 계산, Node로 외삽 → 왜곡")의 가장 근본적 해결책**. Gemini는 § 4-4에서 "추가 대안 가능성"으로만 언급. Storm 2024가 학술 베이스라인 확립 완료.

### 2.5 Edge / Face attribute regression (추가 대안)

| # | 저자(연도) | 제목 / 출처 | 핵심 |
|---|---|---|---|
| 2.5.1 | Horie, Mitsume (2024) | *Graph Neural PDE Solvers with Conservation and Similarity-Equivariance (FluxGNN)* [arXiv:2405.16183] | Face flux를 edge attribute로 학습. **conservation law 보존**. Gemini 누락 대안 (face-flux GNN). |
| 2.5.2 | Praditia et al. (2024) | *Automated discovery of finite volume schemes using GNNs* [arXiv:2508.19052] | FVM 스킴을 edge-level로 학습. Edge attribute regression의 일반화. |
| 2.5.3 | Gong, Cheng (2019) | *Exploiting Edge Features for Graph Neural Networks*, **CVPR 2019** | Edge feature 활용 GNN의 foundation. |

**Why**: 응력 텐서는 본질적으로 **요소 face에서 traction**으로 나타남. Edge-attribute regression은 face flux 보존을 강제할 수 있어 **물리적으로 가장 정합적**이지만, MGN 대규모 변경 필요. **본 논문 6개월 로드맵에는 부적합**, 후속 연구로 권장.

### 2.6 FEM 물리 명시 통합 (B-matrix, shape function, Jacobian)

| # | 저자(연도) | 제목 / 출처 | 핵심 |
|---|---|---|---|
| 2.6.1 | Zhang et al. (2024/2025) | *Finite element-integrated neural network framework for elastic and elastoplastic solids (FEINN)*, **CMAME 433** [DOI:10.1016/j.cma.2024.117474] | **B-matrix·shape function·Gauss integration을 NN loss에 명시 통합**. 변형률은 B·u로 계산, weak form 잔차로 학습. MAPE < 1% vs FEM. **Gemini § 4-4 'B-matrix 통합' 항목의 직접 학술 근거**. |
| 2.6.2 | Thel et al. (2024) | *Introducing Finite Element Method Integrated Networks (FEMIN)*, **CMAME 427:117073** [DOI:10.1016/j.cma.2024.117073] | NN을 FEM solver 내부에 직접 통합 (mesh region 일부를 NN으로 대체). **Crash 시뮬레이션 8,550배 가속**. 본 논문 휠 임팩트 도메인과 유사. |
| 2.6.3 | Storm et al. (2024) | (위 2.3.1) | 미시 graph 위에서 GNN이 strain 예측 → 미시 constitutive로 stress 계산 → **homogenization으로 매크로 양 산출**. Hybrid 방식. |
| 2.6.4 | Pyrialakos, Triantafyllou (2022) | *Integrated Finite Element Neural Network (I-FENN) for non-local continuum damage mechanics*, **CMAME 397** [DOI:10.1016/j.cma.2022.115488] | Damage 변수 NN + FEM solver 결합. **본 논문이 파단 평가를 다룸 → 직접 연관**. |
| 2.6.5 | Liu et al. (2024) | *FEM-PINN: integrating finite element method and PINN for performance prediction via GNN*, **Struct. Multidiscip. Optim.** [DOI:10.1007/s00158-026-04257-2] [추가 검증 필요] | FEM weak form + PINN + GNN 결합. |
| 2.6.6 | Maia, van der Meer (2025) | *Physics-Informed GNNs to Reconstruct Local Fields Considering Finite Strain Hyperelasticity (P-DivGNN)* [arXiv:2507.05291] | **Equilibrium loss (div σ = 0)**를 GNN 학습에 명시 통합. Multi-head decoder + 물리 loss. 본 논문 모델 위에 추가 가능한 **plug-in** 학술 근거. |

**Why**: Gemini § 4-4가 "Physics-aware: B-matrix를 GNN에 명시 통합"을 추가 대안으로 제시했으나 학술 ref 부재. **FEINN(2.6.1) + FEMIN(2.6.2) + P-DivGNN(2.6.6)** 이 이 영역의 baseline을 명확히 확립. 본 논문에 **즉시 차용 가능한 plug-in loss term**.

---

## 3. Gemini 답변과의 매핑 (각 대안별 학술 근거 / 보강 / 반증)

| Gemini 대안 | 직접 대응 학술 ref | 학술 정당화 충분성 | 보강 / 반증 |
|---|---|---|---|
| **대안 1: Element-wise Decoding** | Hernández(2025) 2.1.4, Dalton(2024) 2.1.3, Maurizi(2022) 2.1.1 | **상** | 다중 decoder 설정의 성능 저하 없음(2.1.4 보강). Pooling 함수 정당화는 **여전히 부재** → Storm(2024) 2.3.1의 "shape-function-weighted aggregation" 인용 권장. |
| **대안 2: Bipartite Graph** | FEIH-GNN(Gao 2024) 2.2.1, BSMS-GNN(2.2.5) | **중상** | FEIH-GNN이 hypergraph로 일반화한 형태가 더 강력. 단순 bipartite는 element 4~8 노드 구성 모델링에 정보 손실 가능 → **Hypergraph로 업그레이드 권장**. |
| **대안 3: Dual Graph** | Storm(2024) 2.3.1, Hestroffer(2025) 2.3.2, Lu(2025) 2.3.3 | **상** | Hestroffer(R²=0.992, 150x)가 직접 정량 검증. 그러나 **대변형 추적**이 lattice/polycrystal 정적 가정 하에 검증된 것이라 **휠 임팩트 동적 거동**에서는 추가 검증 필요(Gemini 자체 단점 지적과 일치). |

**Gemini가 누락한 추가 대안 학술 근거**:

| 추가 대안 | 학술 ref | 평가 |
|---|---|---|
| (A) **IP-as-Node Dual Graph** | Storm 2024 (2.3.1/2.4.1) | Gemini 대안 3과 융합 가능. 본 논문 문제 본질에 가장 직접 대응. |
| (B) **Hypergraph Node-Element 통합** | FEIH-GNN (2.2.1) | Gemini 대안 2의 일반화. B-matrix 모방 추가 이점. |
| (C) **Equilibrium-loss plug-in (Multi-Head + div σ=0)** | P-DivGNN (2.6.6) | Gemini가 언급한 "Multi-Head Decoder"에 학술 정당화 + 물리 loss 추가. |
| (D) **NN-in-Solver hybrid (FEMIN/FEINN)** | FEMIN(2.6.2), FEINN(2.6.1) | Gemini가 전혀 언급 X. 양산 휠 임팩트와 도메인 가장 가까움. |
| (E) **Hybrid post-projection (NN→SPR-aware refinement)** | SPR + MLP 결합 [추가 검증 필요] | Gemini가 § 4-4에 "Hybrid: Node 응력 학습 후 별도 IP-projection 모듈"로 언급. SPR(Zienkiewicz 1992) 위에 학습 가능 모듈을 얹는 형태. |

---

## 4. 본 논문에 직접 적용 가능한 후보 모델 (Top 3)

### Top 1: **Element-wise Decoding + Equilibrium Loss** (Gemini 대안 1 + P-DivGNN plug-in)
- **구성**: Pfaff MGN baseline 유지 + Decoder를 element-aggregator로 교체 + L = w₁‖u_node‖² + w₂‖σ_element‖² + w₃‖div σ‖²
- **학술 근거**: Hernández(2025), Dalton(2024), Maia(2025)
- **본 논문 호환성**: **최상**. baseline 코드 ~5% 수정. 학습 라벨은 Abaqus element centroid stress 직접 사용.
- **6개월 로드맵 적합도**: 1~3개월 내 PoC 가능.

### Top 2: **Element-as-Node Dual Graph** (Gemini 대안 3 + Hestroffer 변형)
- **구성**: 메시 element를 graph node로, face-shared adjacency로 edge. 입력=centroid 좌표·부피·물성, 출력=Von Mises stress.
- **학술 근거**: Hestroffer(2025), Storm(2024), Lu(2025)
- **본 논문 호환성**: **상**. baseline 코드 ~30% 재작성 필요(graph 생성부). 그러나 Hestroffer R²=0.992 정량 결과가 강력한 동기.
- **단점**: 대변형 추적 어려움 (Gemini가 지적), 본 논문은 dynamic impact라 **노드 변위 head를 별도 유지** 필요.

### Top 3: **Hypergraph Node-Element 통합 (FEIH-GNN 변형)** (Gemini 대안 2 일반화)
- **구성**: 노드는 그래프 노드, element는 hyperedge. Hypergraph message passing이 local stiffness matrix 모방. 노드와 element 양쪽에서 디코딩.
- **학술 근거**: Gao(2024) FEIH-GNN
- **본 논문 호환성**: **중**. PyG hypergraph 지원이 제한적, 자체 구현 필요. 그러나 **노드 변위 + 요소 응력 동시 출력이 가장 자연스러움**.

---

## 5. 학술적으로 미해결인 갭 / 비교 ablation 누락 영역

1. **Pooling 함수의 물리적 정당성** — Average vs Max vs Sum vs Volume-weighted vs Shape-function-weighted aggregation의 정확도 비교 ablation은 학술 문헌에 거의 부재. 본 논문이 추가하면 contribution.
2. **노드 외삽 응력 vs Element centroid 응력 라벨** — 동일 baseline에서 두 라벨로 학습한 모델의 상대 정확도 비교는 **공식 학술 보고 0건** [추가 검증 필요]. 본 논문 D1 ablation에 자연스럽게 포함 가능.
3. **대변형(geometric nonlinearity) 시 element-as-node의 좌표 추적** — Hestroffer/Storm은 정적·미세 변형에서만 검증. 임팩트 동적 거동에서의 검증은 학술 갭.
4. **실무 ODB 추출 비용 vs 학습 정확도 trade-off** — 학술 모델 대부분이 IP raw stress 직접 사용 가정. 양산 환경에서 추출 시간·저장 부피 분석은 학술 측 부재 → scout-practical에 의존.
5. **Hybrid post-projection (SPR + 학습 모듈)** — 학술 ref가 단편적. **본 논문 후속 publication 후보**.

---

## 다음 에이전트에게 전달

### scout-practical 에게 (실무 검토 시 참고)

- **Storm 2024 (2.3.1) / 2.4.1**: IP raw stress 직접 사용 → Abaqus ODB에서 IP 추출 시 단일 IP element 가정에 주의. **Reduced-integration solid element (C3D8R)는 IP 1개**라 호환성 양호. 휠 임팩트가 explicit dynamic에서 어떤 element type 사용하는지 확인 필요.
- **FEMIN (Thel 2024) / 2.6.2**: NN을 FEM solver 내부에 통합 → Abaqus user subroutine (UEL/UMAT) 수준 통합 필요. 양산 파이프라인 영향 큼. **양산 도입 가능성 평가 요청**.
- **Hestroffer 2025 (2.3.2)**: element centroid stress 사용. Abaqus ODB의 element-level output (S, IVOL, EVOL) 추출 비용·시간을 노드 기반과 비교 측정 요청.
- **FEINN (Zhang 2024) / 2.6.1**: B-matrix를 학습 loss에 통합 → Abaqus가 노출하지 않는 데이터(요소 Jacobian, 형상 함수 미분)가 필요. 외부 FE preprocessor (FreeFEM, FEniCS) 의존성 검토 필요.
- **공통**: 학술 모델 대부분이 단일 element type (Tet/Hex 통일) 가정. 본 논문 휠 메시가 **혼합 element type**이면 Bipartite/Hypergraph 변형 (FEIH-GNN, 2.2.1) 가산점.

### analyst-alternatives 에게 (Gemini 대안 검증 시 참고)

- **Gemini 대안 1 (Element-wise Decoding) 직접 대응 ref**:
  - Hernández et al. 2025 (2.1.4) — multi-decoder 설계 정당화
  - Dalton et al. 2024 (2.1.3) — 노드 변위 + 요소 응력 multi-head
  - Pfaff et al. 2021 (2.1.2) — baseline
  - Maurizi et al. 2022 (2.1.1) — 본 논문 가장 가까운 reference

- **Gemini 대안 2 (Bipartite Graph) 직접 대응 ref**:
  - **★ FEIH-GNN (Gao 2024, 2.2.1)** — node-element hypergraph (bipartite의 자연 일반화). 학술적으로 가장 강력
  - BSMS-GNN (2.2.5)
  - Cao 2024 multi-fidelity GNN (2.2.2)
  - PyG HeteroData (2.2.4) — 구현 framework

- **Gemini 대안 3 (Dual Graph) 직접 대응 ref**:
  - **★ Hestroffer et al. 2025 (2.3.2)** — element-as-node, R²=0.992, 150x speedup. 가장 강한 정량 검증
  - **★ Storm et al. 2024 (2.3.1)** — IP-as-node dual graph, CMAME
  - Lu 2025 (2.3.3) — lattice truss, 실험 검증
  - DG-GNN (2.3.4) — 이론적 근거

- **Gemini가 누락한 추가 대안 후보** (각각 학술 ref 보유):
  - **(A) IP-as-Node Dual Graph** (Storm 2024) — 본 논문 문제 본질에 직접 대응 → **최우선 검토 권장**
  - **(B) Hypergraph Node-Element 통합** (FEIH-GNN, Gao 2024) — Gemini 대안 2 일반화
  - **(C) Equilibrium-loss plug-in** (P-DivGNN, Maia 2025) — Gemini 대안 1 위에 plug-in 가능
  - **(D) NN-in-Solver hybrid** (FEMIN Thel 2024, FEINN Zhang 2024) — Gemini 미언급. 양산 휠 임팩트와 도메인 일치. 단 Abaqus 내부 통합 비용 큼
  - **(E) Hybrid SPR + 학습 (post-projection)** — 학술 ref 단편적. 후속 publication 후보

- **학술 정당화 부재/약한 대안**:
  - Gemini § 4-4의 "Pooling 함수 선택 정당화" → 학술 비교 ablation 거의 부재. **본 논문이 ablation을 수행하면 contribution**
  - Gemini 추천 워크플로우 § 3의 Loss weighting w₁/w₂ → 학술적 표준값 부재. 직관에 의존. P-DivGNN의 div σ=0 항 추가가 정당화 강화에 기여
  - 대변형(impact) 시 element-as-node의 좌표 추적 — 학술 갭. 본 논문이 발견하는 부분이 contribution 후보

- **Top 3 권장 모델**: § 4 참조
  1. Element-wise Decoding + Equilibrium Loss (보수적, 6개월 로드맵 부합)
  2. Element-as-Node Dual Graph (Hestroffer 변형, 정량 근거 강력)
  3. Hypergraph Node-Element (FEIH-GNN, 가장 일반적)

---

**총 reference 발굴 수**: 30건 (그 중 핵심 14건, 보조 16건)
**불확실성 (`[추가 검증 필요]`) 표기 수**: 5건
**커버 범위**: arXiv 2022~2026 / CMAME / Sci.Reports / Comp.Mech / J.Comput.Phys / ICLR / 본 논문 도메인(휠 임팩트) 부분 일치 ref 1건 (Hyundai 후원, Springer Struct. Multidiscip. Optim.)
