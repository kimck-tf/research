---
name: academic-research-scouting
description: "학술 논문/학회/preprint 사이트(arXiv, Google Scholar, IEEE Xplore, Springer, Semantic Scholar, OpenReview)에서 특정 기술 영역의 최신 연구 동향을 체계적으로 조사하는 워크플로우. 본 논문과 비교 가능한 인용·메타데이터·간단 요약을 표준 형식으로 수집한다. 학술 자료 조사, 동향 분석, 인용 수집, 관련 연구 파악이 필요한 모든 작업에 사용. WebSearch/WebFetch 사용을 적극 활용해야 한다."
---

# Academic Research Scouting

학술 동향 조사를 효율적이고 신뢰성 있게 수행하기 위한 절차. scout-gnn-mesh와 scout-domain-app 두 에이전트가 공유한다.

## 1. 조사 시작 전 준비

조사 시작 전 다음 항목을 명확히 한다:

1. **타깃 영역** — 5-10개의 핵심 키워드 (영문)
2. **시점 컷오프** — 보통 본 논문 제출 전 12~24개월
3. **최소 수집 목표** — 영역당 5건 이상, 전체 30건 이상
4. **출처 우선순위** — arXiv > 학회 > 저널 > 미디어 (도메인에 따라 조정)

## 2. 검색 전략

### 2.1 키워드 조합 (boolean)

```
("graph neural network" OR "GNN") AND ("mesh" OR "FEM") AND ("simulation" OR "surrogate")
("MeshGraphNets" OR "GNS") AND (year:2024..2026)
"impact" AND ("crash" OR "drop") AND ("deep learning" OR "neural")
```

### 2.2 검색 사이트별 진입점

| 사이트 | URL 패턴 | 강점 |
|---|---|---|
| arXiv | `https://arxiv.org/list/cs.LG/recent` 또는 `https://arxiv.org/abs/{id}` | preprint, 최신성 |
| Google Scholar | `https://scholar.google.com/scholar?q=` | 인용 추적 |
| Semantic Scholar | `https://www.semanticscholar.org/search?q=` | 영향력, related papers |
| OpenReview | `https://openreview.net/search?query=` | NeurIPS/ICLR/ICML 리뷰 |
| IEEE Xplore | `https://ieeexplore.ieee.org/search/searchresult.jsp?queryText=` | IEEE 학회/저널 |
| ACM DL | `https://dl.acm.org/action/doSearch?AllField=` | ACM 학회 |

### 2.3 인용 추적 (Snowballing)

핵심 논문 1개를 발견하면:
- **Backward**: 해당 논문이 인용한 논문들 (특히 최근 인용)
- **Forward**: 해당 논문을 인용한 논문들 (Semantic Scholar, Google Scholar의 "cited by")

본 작업에서는 **MeshGraphNets (Pfaff et al., 2021)** 와 본 논문 저자의 선행 연구 **(Jin, Zheng, Kim, Gu, APL 2023)** 를 시작점으로 forward citation 추적이 효율적이다.

## 3. 도구 사용

### 3.1 WebSearch (1차 검색)

```
WebSearch(query: "MeshGraphNets follow-up 2024 graph neural simulator")
```

- 결과 5-10개 중 관련성 높은 것만 선별
- 동일 키워드 변형으로 2-3회 반복 검색

### 3.2 WebFetch (상세 확인)

```
WebFetch(url: "https://arxiv.org/abs/2410.XXXXX",
         prompt: "이 논문의 주요 기여, 사용 데이터셋, 평가 지표, baseline 정리")
```

- arXiv 페이지에서 abstract, BibTeX, related papers 수집
- 학회 페이지에서 발표 비디오 / poster 링크 확보 가능

### 3.3 이미 알려진 정보 활용

본 논문 저자가 보유한 외부 컨텍스트(예: MCP context7, 이미 알려진 표준 논문)를 우선 활용. 동일 정보를 web에서 재검색하지 않는다.

## 4. 수집 정보 표준 형식

각 인용에 대해 다음 7개 항목을 수집:

```yaml
- title: "정확한 논문 제목"
  authors: "First Author et al." (대표 1-3인)
  year: 2024
  venue: "ICLR 2024" 또는 "arXiv:2410.XXXXX"
  url: "https://arxiv.org/abs/2410.XXXXX"
  contribution: "한 문장 핵심 기여"
  relevance_to_paper: "본 논문과의 관련성 (왜 인용 가치 있는가)"
```

## 5. 출력 형식

각 scout 에이전트의 SKILL 정의에 명시된 섹션 구조를 따르되, 다음 4가지를 반드시 포함:

1. **핵심 트렌드 요약** (5줄 이내) — 가장 가치 있는 정보
2. **분야별 주요 연구** (영역당 3-5건, 위 yaml 형식)
3. **본 논문 직접 비교 가능 연구** (반드시 본 논문의 데이터/모델/기법과 1:1 비교 가능한 것)
4. **관찰된 갭 / 미해결 과제** (다른 팀원이 활용할 수 있도록)

## 6. 신뢰도 관리

| 상황 | 표기 |
|---|---|
| 직접 abstract 확인 | (표기 없음 — 기본) |
| 검색 결과 스니펫만 확인 | `[abstract 미확인]` |
| 2차 출처(블로그/리뷰)에서만 정보 | `[2차 출처]` |
| 정보 신뢰도 낮음 | `[추가 검증 필요]` |
| 유료 저널 미확보 | `[전문 미확보]` |

태그가 붙은 인용은 후속 작업자가 검증할 수 있도록 한다.

## 7. 중복 방지 및 품질 관리

- 동일 논문이 여러 영역에 해당하면 가장 핵심 영역에 1회만 (다른 영역에선 짧은 cross-reference)
- preprint와 학회 출판 동일 논문이면 학회 버전 우선 (arXiv ID는 부기)
- "최신성"보다 "관련성" 우선 — 2020년 논문이라도 본 논문에 더 직접적이면 포함

## 8. 협업 신호

조사 중 다른 scout 에이전트의 영역에 해당하는 발견 시:
- `SendMessage(to: "{other-scout}", message: "{영역}에 해당하는 자료 발견: {요약 + 링크}")`
- 단, 메시지 빈도는 영역당 2-3회로 제한 (과도한 통신은 비용)

## 9. 작업 종료 조건

- 영역별 5건 이상 수집 완료
- 핵심 트렌드 요약 작성 완료
- 본 논문 비교 가능 연구 3건 이상 확보
- 산출물 파일을 `_workspace/0X_{scout-name}.md`로 저장

미달 시 리더에게 SendMessage로 추가 시간 요청 또는 영역 축소 협의.
