---
name: alternatives-verification
description: "Gemini가 제안한 3개 대안(Element-wise Decoding, Bipartite Graph, Dual Graph)을 학술·실무 측면에서 검증하고 추가 대안을 발굴하여, 6축 비교 매트릭스 + 본 논문 환경 시뮬레이션 + 권장 대안 1+1을 산출하는 스킬. analyst-alternatives 에이전트가 사용한다."
---

# Alternatives Verification — 대안 검증 및 권장안 산출 절차

두 scout 산출물을 종합하여 노드/요소 도메인 불일치 해결 대안을 체계적으로 비교·평가하고, 본 논문(JSAE 2026) 환경에 가장 적합한 대안을 권장하는 절차.

## 사용 시점

analyst-alternatives 에이전트가 두 scout 산출물 완료 후 작업을 시작할 때.

## 사전 준비

1. `_workspace/01_scout-academic.md` 읽기 (학술 동향)
2. `_workspace/02_scout-practical.md` 읽기 (실무 파이프라인)
3. `_workspace/00_input/{PROBLEM,GEMINI_ANSWER,CONTEXT}.md` 재확인

## 작업 절차

### Step 1. 대안 목록 통합 (30분)

다음 대안 세트를 정의하고 각각 학술 ref + 실무 데이터를 매핑:

**Gemini 3대안 (검증 대상)**
- A1. Element-wise Decoding (노드 잠재→요소 pooling→요소 응력)
- A2. Bipartite Graph (Node + Element 두 종류 vertex)
- A3. Dual Graph (Element-as-node)

**추가 대안 (scout-academic 발견 + 본 절차에서 정의)**
- A4. IP-as-node (적분점을 별도 그래프 노드로 도입)
- A5. Edge/Face-attribute regression (응력 텐서를 edge/face attr로)
- A6. Hybrid post-projection (노드 학습 → 학습 가능한 IP/centroid 사상 모듈)
- A7. B-matrix integrated decoding (FEM shape function gradient를 GNN에 명시 통합)
- A8. Multi-output multi-head (변위→노드 / 응력→요소 동시 출력) — Gemini 추천 워크플로우와 동일하나 별도 분류
- (필요 시) A9. Continuous-resolution / mesh-free 변형

각 대안에 대해:
- 핵심 메커니즘 1단락
- 학술 근거 (scout-academic ref 인용 / 부재 시 명시)
- 데이터 요구 (scout-practical ref 인용)
- 변형 (예: A1-avg / A1-volume-weighted)

### Step 2. 6축 비교 매트릭스 작성 (45분)

각 대안을 다음 6축으로 평가:

| 축 | 평가 방법 | 척도 |
|---|---|---|
| **정확도 추정** | 학술 ref가 보고한 정확도 / 유사 도메인 추정 | 상/중/하 + 구체값 |
| **학습 비용** | 학습 시간·메모리 (A100 1대 기준) | 1~5× MGN baseline |
| **추론 비용** | 단일 휠 inference 시간 | 1~5× MGN baseline |
| **메모리** | 학습 시 GPU 메모리 | 1~3× MGN baseline |
| **구현 난이도** | 본 논문 baseline 코드 재활용율 | %, 또는 상/중/하 |
| **실무 호환성** | 본 논문 데이터 추출 + 평가 metric 호환 | 상/중/하 + 사유 |
| **학술 신규성** | 발표 가능 venue / impact | 상/중/하 |

**종합 점수**: 6축 가중합 (실무 호환성 1.5×, 정확도 1.5×, 나머지 1× 권장)

### Step 3. 본 논문 환경 시뮬레이션 (30분)

각 대안을 본 논문 데이터(580 샘플 / 200~400k 노드 / spoke 2.4%)에 적용 시 추정:

- **데이터 추출 비용**: scout-practical 의 S1~S4 시나리오 활용
- **학습 시간**: A100 1대 / B=2 가정으로 epoch당 시간
- **추론 시간**: 단일 휠 forward pass (밀리초 단위 추정)
- **메모리 peak**: GPU 메모리 사용량 추정

### Step 4. Gemini 3대안 비판적 평가 (30분)

각 대안의:
- **강점** (Gemini 답변 + 학술 근거 종합)
- **약점** (학술/실무 양 측면)
- **학술 정당화 충분성** — 직접 ref 존재 / 간접 ref / 부재
- **반증** — 대안의 가정이 본 논문 환경에서 무너지는 사례

### Step 5. Top 추천 1+1 선정 (45분)

- **단기 Quick Win (1~2개월)** — 본 논문 baseline 재활용율 ≥70%, 1차 ablation 가능, JSAE/SAE workshop 가능
- **중기 학술 publication (3~6개월)** — 학술 신규성 상, 후속 publication 후보, D1~D8 로드맵과 연계

각 추천은:
- 대안 명칭 + 변형 (예: A1-volume-weighted)
- 선정 근거 (6축 매트릭스 점수 + 시뮬레이션 결과)
- 구현 단계 (Step 1~N) — 의사코드 또는 모듈 구조 수준
- 성공 지표 (정량) — Top-20% MAPE, KLD, inference time 등
- 리스크 + 완화책

### Step 6. 비추천 대안 명시 (15분)

평가에서 제외된 대안과 그 이유 명시 (독자가 "왜 이건 안 되는가" 판단 가능하도록).

### Step 7. 단계적 마이그레이션 경로 (15분)

추천 #1 (단기) → 추천 #2 (중기) 의 자연스러운 진화 경로.

## 산출물 분리

- `_workspace/03_alternatives-matrix.md` — Step 1~4 결과
- `_workspace/04_recommendation.md` — Step 5~7 결과

## 협업 트리거

- 데이터 부족 영역 발견 시 해당 scout에게 SendMessage로 추가 조사 요청 (1회)
- 권장안 확정 시 report-author에게 핵심 발견 5~7개 SendMessage

## 검증 체크리스트

- [ ] Gemini 3대안 각각의 학술 근거 / 부재가 명시되었는가
- [ ] 추가 대안 ≥ 2개 정의되었는가
- [ ] 6축 매트릭스가 모든 대안에 대해 채워졌는가
- [ ] 본 논문 환경 시뮬레이션 수치가 포함되었는가
- [ ] 추천 #1 이 실제 1~2개월 내 시작 가능한 구체성을 갖는가
- [ ] 비추천 대안 + 이유가 명시되었는가

## 종료 조건

위 체크리스트 6개 모두 충족 시 산출물 마무리. 타임아웃(~2시간) 도달 시 매트릭스 + 추천 #1 만이라도 완성.
