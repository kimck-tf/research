---
name: fem-aware-academic-scouting
description: "FEM-aware GNN, Element-wise decoding, Bipartite/Dual graph, IP-level prediction 등 노드/요소 도메인 불일치 해결 관련 학술 reference를 발굴·정리하는 스킬. scout-academic 에이전트가 Gemini 3대안 검증 + 추가 대안 발굴을 위해 사용한다."
---

# FEM-Aware Academic Scouting — 학술 동향 조사 절차

본 논문(JSAE 2026, MeshGraphNets 기반)의 노드/요소 도메인 불일치 해결을 위한 학술 reference를 발굴하는 절차.

## 사용 시점

scout-academic 에이전트가 `_workspace/01_scout-academic.md` 작성을 시작할 때.

## 사전 준비

1. `_workspace/00_input/PROBLEM.md` — 문제 정의 숙지
2. `_workspace/00_input/GEMINI_ANSWER.md` — 검증 대상 3대안 숙지
3. `_workspace/00_input/CONTEXT.md` — 본 논문 컨텍스트 숙지
4. WebSearch / WebFetch 도구 가용성 확인 (없으면 ToolSearch로 로드 요청)

## 검색 절차

### Step 1. Gemini 3대안 직접 매핑 검색 (각 대안당 30분)

각 대안에 대응하는 학술 reference를 우선 발굴.

**대안 1 (Element-wise Decoding) 키워드**:
- "element decoder graph network mesh"
- "MeshGraphNet element-wise output"
- "node aggregation element prediction GNN"
- "element pooling stress prediction"
- "shape-function-weighted aggregation"

**대안 2 (Bipartite Graph) 키워드**:
- "bipartite graph FEM neural network"
- "node element heterogeneous graph mesh"
- "two-type graph mesh simulation"
- "PyTorch Geometric HeteroData mesh"

**대안 3 (Dual Graph) 키워드**:
- "dual graph mesh element"
- "face-graph stress prediction"
- "element-as-node GNN"
- "primal-dual mesh graph"

### Step 2. 추가 대안 탐색 (45분)

Gemini가 누락한 대안을 발굴.

- **IP-level prediction**: "Gauss point neural network", "integration point GNN", "quadrature point prediction"
- **Edge/face attribute regression**: "edge attribute mesh GNN stress tensor", "face flux GNN PDE"
- **Hybrid post-projection**: "neural surrogate followed by FE recovery", "patch recovery neural network"
- **B-matrix / shape function 통합**: "FEM shape function neural network", "B-matrix aware GNN", "differentiable FEM"
- **Multi-resolution node-element**: "multi-scale GNN element refinement"

### Step 3. 베이스라인 / Foundation 연구 확인 (30분)

도메인 불일치를 우회하지 않고 명시적으로 다룬 핵심 reference.

- Battaglia et al. 2018 (Graph Networks 원본) — node/edge/global attribute의 일반화 개념
- Pfaff et al. 2021 (MeshGraphNets) — 본 논문 baseline, 출력이 노드인 이유
- 최근(2024~2026) FEM-AI hybrid 사례 — FEMIN, FEM-PINN, PI-MGN 등이 응력을 어떻게 다루는가

### Step 4. 차원별 정리 (30분)

발견한 reference를 다음 차원으로 표 정리:
- **분야**: Element-wise / Bipartite / Dual / IP / Edge attr / Hybrid / Other
- **응용 도메인**: 유체 / 고체 / impact / fracture / 분자 등
- **데이터 구조**: 출력이 노드 / 요소 / IP / edge 중 무엇인가
- **본 논문 적용 가능성**: 직접 적용 / 부분 차용 / 참고만 / 부적합

## 인용 품질 기준

각 reference는 다음을 포함:
- 저자(연도)
- 제목 전체
- 출처 (arXiv ID / DOI / 학회·저널명)
- URL (arxiv.org/abs/XXXX 또는 DOI link)
- 한 줄 요약 (한국어)
- 본 논문 적용 시 가치 (Gemini 어느 대안 대응 / 추가 대안 정의)
- `[추가 검증 필요]` 태그 (불확실 시)

## 산출물 작성 규칙

- 형식: `_workspace/01_scout-academic.md`
- 분량: 200~350줄 권장
- 핵심 발견 5건 우선 노출 (앞쪽 § 1~2)
- 본 논문에 직접 적용 가능한 Top 3 모델 명시 (마지막 §)
- Gemini 3대안 매핑 표 포함 (각 대안 ↔ 학술 ref ↔ 정당화 충분성)

## 협업 트리거

다음 발견 시 즉시 SendMessage:

- **scout-practical에게**:
  - 학술 모델이 실무 데이터 형식과 충돌하는 경우 (예: IP raw stress 직접 사용 가정)
  - 학술 모델이 사용한 데이터 추출 절차가 본 논문 파이프라인과 다른 경우

- **analyst-alternatives에게**:
  - "Gemini 대안 X에 직접 대응 학술 ref 발견" / "Gemini 누락 대안 발견"

## 종료 조건

다음 중 하나 충족 시 산출물 마무리:
1. Gemini 3대안 모두에 대해 학술 근거 / 부재 명시 완료
2. 추가 대안 최소 2개 이상 정의
3. Top 3 권장 모델 선정
4. 핵심 발견 5건 이상

타임아웃 (~2시간) 도달 시 부분 결과로 마무리하고 누락 영역을 명시.
