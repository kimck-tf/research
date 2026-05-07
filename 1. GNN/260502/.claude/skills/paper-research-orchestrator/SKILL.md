---
name: paper-research-orchestrator
description: "기존 논문(특히 ref_paper/ 내의 논문)을 발전시키기 위한 최신 연구 동향 조사 + 갭 분석 + 향후 연구 방향 제안 보고서를 작성하는 오케스트레이터. 4명의 에이전트 팀(scout 2명 + gap-strategist + report-author)을 자동 구성·조율하여 30~50페이지 한국어 보고서를 산출. 사용자가 '논문 동향 조사', '향후 연구 방향', '연구 발전 보고서', '관련 연구 분석', '연구 로드맵' 등을 요청하면 반드시 이 스킬을 사용한다."
---

# Paper Research Orchestrator — 동향 조사 및 향후 방향 보고서 자동화

본 논문(`ref_paper/` 내)을 발전시키기 위한 최신 동향 조사와 향후 연구 방향 제안 보고서를 4명의 에이전트 팀이 협업하여 작성하는 오케스트레이터.

## 실행 모드: 에이전트 팀 (TeamCreate 사용)

조사자 간 발견 공유, 갭 분석가의 추가 조사 요청 등 실시간 협업이 품질을 결정하므로 에이전트 팀 모드로 운영한다.

## 에이전트 구성

| 팀원 | 에이전트 타입 | 역할 | 사용 스킬 | 출력 |
|---|---|---|---|---|
| scout-gnn-mesh | general-purpose (커스텀 정의) | GNN/Neural Operator 동향 | academic-research-scouting | `_workspace/01_scout-gnn-mesh.md` |
| scout-domain-app | general-purpose (커스텀 정의) | 자동차/CAE 응용 동향 | academic-research-scouting | `_workspace/02_scout-domain-app.md` |
| gap-strategist | general-purpose (커스텀 정의) | 갭 분석 + 방향 제안 | research-gap-analysis, future-research-proposal | `_workspace/03_gap-analysis.md`, `_workspace/04_future-directions.md` |
| report-author | general-purpose (커스텀 정의) | 통합 보고서 작성 | technical-research-report | `최종_연구동향_및_향후방향_보고서.md` |

리더 = 오케스트레이터 (사용자와 직접 대화하는 본 세션) — Phase 진행 관리, 사용자 검토 대응.

## 워크플로우

### Phase 1: 준비

1. 사용자 입력 분석:
   - 대상 논문 위치 확인 (기본: `ref_paper/`)
   - `_claude_docs/PAPER_SUMMARY.md` 존재 여부 확인 (없으면 사용자에게 `/init` 먼저 권장)
   - 사용자가 강조한 영역이 있다면 (예: "특히 시계열 부분") 메모

2. `_workspace/` 디렉토리 생성:
   ```
   _workspace/
   ├── 00_input/
   │   └── PAPER_SUMMARY.md   (복사본)
   ├── 01_scout-gnn-mesh.md   (실행 시 생성)
   ├── 02_scout-domain-app.md
   ├── 03_gap-analysis.md
   ├── 04_future-directions.md
   └── _drafts/
   ```

3. 입력 자료 복사: `_claude_docs/PAPER_SUMMARY.md` → `_workspace/00_input/PAPER_SUMMARY.md`

### Phase 2: 팀 구성 및 작업 등록

```
TeamCreate(
  team_name: "paper-research-team",
  members: [
    {
      name: "scout-gnn-mesh",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 .claude/agents/scout-gnn-mesh.md에 정의된 에이전트입니다.
               해당 파일을 먼저 읽고 역할을 숙지하세요.
               .claude/skills/academic-research-scouting/SKILL.md의 절차를 따라
               GNN/Neural Operator/Mesh-based simulation 최신 연구를 조사하고
               _workspace/01_scout-gnn-mesh.md에 저장하세요.
               시작하기 전 _workspace/00_input/PAPER_SUMMARY.md를 반드시 읽으세요."
    },
    {
      name: "scout-domain-app",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 .claude/agents/scout-domain-app.md에 정의된 에이전트입니다.
               해당 파일을 먼저 읽고 역할을 숙지하세요.
               .claude/skills/academic-research-scouting/SKILL.md의 절차를 따라
               자동차/구조해석/CAE surrogate 응용 사례를 조사하고
               _workspace/02_scout-domain-app.md에 저장하세요.
               시작하기 전 _workspace/00_input/PAPER_SUMMARY.md를 반드시 읽으세요."
    },
    {
      name: "gap-strategist",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 .claude/agents/gap-strategist.md에 정의된 에이전트입니다.
               두 scout(scout-gnn-mesh, scout-domain-app)의 산출물이 완료된 후
               작업을 시작합니다 (TaskCreate의 depends_on으로 보장).
               .claude/skills/research-gap-analysis/SKILL.md와
               .claude/skills/future-research-proposal/SKILL.md를 사용하여
               _workspace/03_gap-analysis.md와 _workspace/04_future-directions.md를 생성하세요."
    },
    {
      name: "report-author",
      agent_type: "general-purpose",
      model: "opus",
      prompt: "당신은 .claude/agents/report-author.md에 정의된 에이전트입니다.
               gap-strategist 작업 완료 후 시작합니다.
               .claude/skills/technical-research-report/SKILL.md의 절차를 따라
               4개 산출물을 통합한 보고서를
               '최종_연구동향_및_향후방향_보고서.md'로 작성하세요."
    }
  ]
)
```

작업 등록 (의존성 명시):
```
TaskCreate(tasks: [
  { id: "T1", title: "GNN/Neural Operator 동향 조사", assignee: "scout-gnn-mesh" },
  { id: "T2", title: "자동차/CAE 응용 동향 조사", assignee: "scout-domain-app" },
  { id: "T3", title: "갭 분석", assignee: "gap-strategist", depends_on: ["T1", "T2"] },
  { id: "T4", title: "향후 방향 로드맵", assignee: "gap-strategist", depends_on: ["T3"] },
  { id: "T5", title: "통합 보고서 작성", assignee: "report-author", depends_on: ["T3", "T4"] }
])
```

### Phase 3: 조사 (병렬)

scout-gnn-mesh와 scout-domain-app이 병렬 실행:

- 두 scout는 SendMessage로 발견을 공유 (영역 경계 사례)
- 리더는 진행 상황 모니터링, 막힌 경우 개입
- 약 30분~1시간 소요 예상

### Phase 4: 갭 분석 + 방향 제안 (T3, T4)

gap-strategist가 두 scout 산출물을 Read한 후:
- T3 (갭 분석) 완료 → `_workspace/03_gap-analysis.md`
- T4 (방향 제안) 완료 → `_workspace/04_future-directions.md`

gap-strategist는 분석 중 데이터 부족 시 scout에게 SendMessage로 추가 조사 요청 가능.

### Phase 5: 통합 보고서 작성 (T5)

report-author가 4개 산출물을 통합:
- 한국어 학술 톤
- Executive Summary 1페이지
- 본문 6개 장
- 부록 A/B/C
- 30-50페이지 분량

작성 완료 시 리더에게 알림.

### Phase 6: 사용자 검토 및 정리

1. 리더가 보고서 1차 검토 (체크리스트):
   - 인용 정합성 (본문 ↔ 부록 A)
   - 한국어 톤 일관성
   - 본 논문 수치 정확성
   - Executive Summary 간결성

2. 리더가 사용자에게 보고:
   - 보고서 위치 안내
   - 핵심 발견 5-7개 요약
   - Top 3 추천 방향

3. 사용자 피드백 수신:
   - 수정 요청 시 report-author에게 SendMessage로 전달
   - 추가 조사 요청 시 해당 scout 재가동

4. 최종 정리:
   - `_workspace/` 보존 (사후 검증·감사 추적용)
   - 팀 정리 (TeamDelete)

## 데이터 흐름

```
사용자 입력
    ↓
_claude_docs/PAPER_SUMMARY.md → _workspace/00_input/
    ↓
[scout-gnn-mesh] ←─SendMessage─→ [scout-domain-app]
    ↓                                ↓
01_scout-gnn-mesh.md          02_scout-domain-app.md
    └──────────────┬──────────────────┘
                   ↓
          [gap-strategist]
                   ↓
03_gap-analysis.md, 04_future-directions.md
                   ↓
            [report-author]
                   ↓
   최종_연구동향_및_향후방향_보고서.md
                   ↓
              사용자 검토
```

## 에러 핸들링

| 상황 | 전략 |
|---|---|
| `_claude_docs/PAPER_SUMMARY.md` 없음 | 사용자에게 `/init` 먼저 실행 권장. 사용자가 진행 강행 시 ref_paper PDF 직접 분석 후 진행 |
| WebSearch/WebFetch 도구 미로드 | 사용자에게 ToolSearch로 로드 필요 안내 |
| Scout 1명 실패 | 1회 재시작. 재실패 시 가용 결과로 진행, 보고서에 "{영역} 일부 데이터 미수집" 명시 |
| 두 Scout 모두 동일 영역 발견 누락 | gap-strategist 단계에서 발견 시, 리더에게 알림 → scout 재가동 또는 진행 |
| Gap-strategist 산출물 부실 | 1회 재작성 요청. 미해결 시 report-author에게 부실 명시 후 진행 |
| Report-author 분량 초과 (50페이지+) | 5장(향후 방향) 외 영역 압축 요청, 디테일은 부록으로 |
| 사용자 중도 변경 요청 | 현재 Phase 완료 후 적용. 진행 중 Phase는 완료 |

## 테스트 시나리오

### 정상 흐름
1. 사용자가 `/harness:harness 동향 조사 보고서 만들어줘` 입력
2. Phase 1에서 `_claude_docs/PAPER_SUMMARY.md` 존재 확인 ✓
3. Phase 2에서 4명 팀 구성, 5개 작업 등록
4. Phase 3에서 2명 scout 병렬 조사 (~1시간)
5. Phase 4에서 gap-strategist 분석 (~30분)
6. Phase 5에서 report-author 통합 보고서 작성 (~30분)
7. Phase 6에서 사용자 검토 후 확정
8. 결과: `최종_연구동향_및_향후방향_보고서.md` 생성

### 에러 흐름
1. Phase 3에서 scout-gnn-mesh가 WebFetch 실패 반복 (3회)
2. 리더가 알림 수신
3. 사용자에게 보고: "scout-gnn-mesh 데이터 수집 제한적. 부분 결과로 진행 또는 재시도?"
4. 사용자 선택에 따라:
   - 부분 결과: gap-strategist에 "{영역} 데이터 부족" 명시 후 진행
   - 재시도: 사용자가 ToolSearch로 도구 로드 또는 다른 검색 도구 활성화 후 재시작

## 사전 점검 (Phase 1 시작 시)

스킬 시작 시 다음 사전 점검 수행:

1. **WebSearch / WebFetch 도구 가용성**: ToolSearch로 로드되어 있는지 확인. 없으면 사용자에게 안내
2. **본 논문 PDF 존재**: `ref_paper/` 디렉토리 확인
3. **PAPER_SUMMARY.md 존재**: 없으면 사용자에게 `/init` 권장
4. **에이전트 정의 4개 파일 존재**: `.claude/agents/{4개}.md`
5. **스킬 4개 파일 존재**: `.claude/skills/{4개}/SKILL.md`

5개 점검 모두 통과 시 Phase 2 진행.

## 비용 / 시간 추정

- 총 토큰: 200k~500k (4명 에이전트 × 동시 작업)
- 총 시간: 1.5~3시간 (Phase 3 병렬 조사가 가장 큰 비중)
- 산출물: 보고서 1개 (~30-50 페이지) + 중간 산출물 5개

대규모 작업이므로 사용자에게 시작 전 비용 안내 고려.

## 종료 후 사용자에게 전달할 정보

1. 보고서 위치
2. Executive Summary의 핵심 발견 5-7개
3. Top 3 추천 방향 요약
4. `_workspace/` 보존 안내 (감사 추적)
5. 추가 조사 / 수정 요청 방법 안내
