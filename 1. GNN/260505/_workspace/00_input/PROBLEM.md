# 문제 정의: MeshGraphNet의 Node-Element Domain Mismatch

**작성일**: 2026-05-05
**문제 제기자**: Changgon Kim (현대자동차)
**연관 논문**: `../260502/ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf`

## 1. 문제 본질

**FEM 해석의 메커니즘과 GNN(MeshGraphNet) 학습 데이터 구조 사이의 도메인 불일치.**

| 측면 | FEM (Abaqus 등 CAE) | GNN (MeshGraphNet) |
|---|---|---|
| 변위(Displacement) | Node에서 정의 | Node(=vertex)에서 학습 |
| 응력(Stress) / 변형률(Strain) | **Element의 Integration Point**에서 계산 | Node 위치에서 학습 (외삽 결과) |
| 파단 평가 기준 | **Element Von Mises 응력** | (현재) Node 응력 → 평가 부적합 |

## 2. 노드 응력의 왜곡 메커니즘

1. CAE 솔버는 적분점(IP, integration point)에서 응력·변형률을 계산
2. 후처리 시 IP → Node로 외삽(extrapolation) + 인접 요소 평균
3. **응력 구배가 큰 영역(특이점, 응력 집중부)** 에서 외삽 오차 + smoothing error 발생
4. 노드 응력은 동일 노드를 공유하는 인접 요소들의 평균값이므로, 요소별 실제 응력 정보가 손실됨
5. 결과적으로 노드 응력에는 종종 **비현실적으로 큰 값**(또는 평균화로 인한 작은 값)이 나타남

## 3. 실무 평가 요건과의 괴리

- **파단 평가**: ISO 3006, KMVSS, OEM 사내 기준 모두 Element Von Mises 응력 기반
- **현 GNN 예측**: 노드 위치 응력 → 외삽 왜곡 그대로 학습됨
- **결과**: GNN 예측값을 평가 로직에 직접 투입할 수 없음 (변환·후처리 필요)

## 4. 대치 상황 (The Dilemma)

```
  [학습 데이터 요구]                [평가 로직 요구]
        ↓                                ↓
  Node 위치 응력 (왜곡됨)         Element 위치 응력 (실값)
        ↓                                ↓
  ────────────────  Domain Mismatch ────────────────
```

GNN은 노드 그래프에서 학습하는 것이 자연스럽지만, 학습 라벨로 사용하는 노드 응력은 외삽으로 왜곡되어 있고, 실제 평가에 사용하는 응력은 요소 위치의 값이다.

## 5. 해결해야 할 핵심 질문

1. **데이터 측면**: GNN 학습 라벨을 노드/요소 어느 쪽으로 잡을 것인가? Abaqus ODB에서 어느 데이터를 추출해야 하는가?
2. **모델 측면**: GNN 아키텍처를 어떻게 수정해야 노드 기반 학습 + 요소 기반 출력이 양립하는가?
3. **평가 측면**: 어떤 평가 metric이 노드/요소 도메인 차이를 공정하게 반영하는가?
4. **실무 적용**: 기존 MGN 베이스라인 대비 추가 비용/복잡도가 양산 도입 가능 수준인가?

## 6. 현재 보유 자원

- 본 논문 데이터 (580 샘플, 200~400k 노드)
- Abaqus ODB / PPT 자동 추출 파이프라인 (재사용 가능)
- MGN baseline 코드
- 260502 보고서 (60건 reference, 8개 연구방향 — D1 SOTA Baseline ablation 진행 시 본 문제 자연스럽게 포함됨)

## 7. 출력 요구사항

본 하네스의 산출물은 다음을 포함해야 한다:

- 학술 동향 (Element-aware GNN, Bipartite/Dual graph, FEM-aware 모델 사례)
- 실무 검토 (Abaqus ODB IP/centroid 추출 실용성, 데이터 파이프라인 영향)
- Gemini 제안 3개 대안의 검증·확장
- 추가 발굴 가능한 대안
- 최종 추천 방법론 1개 + 단계적 구현 로드맵 (1~6개월)
