---
name: node-element-orchestrator
description: "MeshGraphNet의 노드/요소 도메인 불일치 해결 방안 보고서 작성 오케스트레이터. 4명 서브 에이전트(scout-academic, scout-practical, analyst-alternatives, report-author)를 순차·병렬 호출하여 학술 동향+실무 검토+Gemini 3대안 검증+추가 대안 발굴+권장안+구현 로드맵을 통합한 한국어 보고서를 산출. 사용자가 '노드 요소 불일치', 'element decoding', 'node element domain mismatch', 'IP/centroid 응력', 'GNN 평가 데이터' 등을 요청하면 반드시 이 스킬을 사용한다."
---

# Node-Element Orchestrator — 도메인 불일치 해결 보고서 자동화

본 논문(JSAE 2026)의 노드/요소 도메인 불일치 문제 해결 방안을 4명 에이전트가 협업하여 산출하는 오케스트레이터.

## 실행 모드: 서브 에이전트 (Agent 도구 직접 호출)

본 하네스는 Windows 네이티브 환경 호환을 위해 **서브 에이전트 모드**로 운영된다.

- 오케스트레이터(메인 세션)가 `Agent` 도구로 4명을 순차·병렬 호출
- 에이전트 간 실시간 SendMessage 불가 → **모든 협업은 파일 기반** + 오케스트레이터 중계
- 두 scout 산출물의 발견 공유는 산출물 파일 마지막 섹션 `## 다음 에이전트에게 전달` 에 메모로 기록
- 산출물은 `_workspace/` 하위 약속 경로에 저장

(에이전트 팀 모드 — TeamCreate + SendMessage 실시간 협업 — 는 Windows 네이티브에서 작동하지 않는 Bun SFE 버그(GitHub Issue #26244)로 제외됨. WSL+tmux 환경에서는 별도 스킬 변형으로 전환 가능)

## 에이전트 구성

| 에이전트 | 빌트인 타입 | 역할 | 사용 스킬 | 출력 |
|---|---|---|---|---|
| scout-academic | general-purpose | FEM-aware GNN 학술 동향 | fem-aware-academic-scouting | `_workspace/01_scout-academic.md` |
| scout-practical | general-purpose | Abaqus ODB / 실무 파이프라인 | fem-practical-scouting | `_workspace/02_scout-practical.md` |
| analyst-alternatives | general-purpose | 매트릭스 + 권장안 | alternatives-verification | `_workspace/03_alternatives-matrix.md`, `_workspace/04_recommendation.md` |
| report-author | general-purpose | 통합 보고서 | solution-proposal-report | `노드_요소_도메인불일치_해결방안_보고서.md` |

각 Agent 호출 시 `model: "opus"` 명시.

## 워크플로우

### Phase 1: 사전 점검

1. 입력 자료 3개 존재 확인:
   - `_workspace/00_input/PROBLEM.md`
   - `_workspace/00_input/GEMINI_ANSWER.md`
   - `_workspace/00_input/CONTEXT.md`
2. 에이전트 정의 4개: `.claude/agents/{name}.md`
3. 스킬 4개: `.claude/skills/{name}/SKILL.md`
4. 본 논문 PDF: `../260502/ref_paper/`
5. 도구 가용성: `WebSearch`, `WebFetch` (없으면 ToolSearch로 로드)

5개 점검 통과 시 Phase 2 진행.

### Phase 2: 진행 계획 확정

서브 에이전트 모드는 TeamCreate가 불필요하므로, Phase 2는 단순화된다:

1. 메인 세션의 진행 추적용 task 5개를 TaskCreate로 등록 (T1~T5)
2. 사용자에게 다음을 보고:
   - 4개 에이전트 호출 순서 + 예상 시간
   - 입력/출력 파일 경로
   - Phase 3 시작 승인 요청

사용자 승인 후 Phase 3 진행.

### Phase 3: 학술 + 실무 병렬 조사 (~1~2시간)

`scout-academic` 과 `scout-practical` 을 동시에 호출:

```
Agent(
  subagent_type: "general-purpose",
  model: "opus",
  description: "FEM-aware GNN 학술 동향 조사",
  prompt: """당신은 .claude/agents/scout-academic.md 에 정의된 scout-academic 입니다.

[작업 디렉토리]: C:\_PYTHON\0_CK_Project\0. Research\1. GNN\260505

수행 절차:
1. 본인 정의 읽기: .claude/agents/scout-academic.md
2. 사용 스킬 읽기: .claude/skills/fem-aware-academic-scouting/SKILL.md
3. 입력 자료 3개 읽기: _workspace/00_input/{PROBLEM,GEMINI_ANSWER,CONTEXT}.md
4. 스킬 절차에 따라 학술 reference 발굴 (WebSearch/WebFetch 활용)
5. 산출물 작성: _workspace/01_scout-academic.md
6. 산출물 마지막에 '## 다음 에이전트에게 전달' 섹션 추가:
   - scout-practical 에게 공유할 학술/실무 경계 사례
   - analyst-alternatives 에게 공유할 Gemini 3대안 매핑 결과 + 추가 대안 후보
7. 완료 후 메인 세션에 200단어 이내 핵심 발견 요약 보고""",
  run_in_background: true
)

Agent(
  ... scout-practical (동일 패턴) ...,
  run_in_background: true
)
```

두 에이전트의 완료 통지를 받으면 산출물 파일 존재 + 내용 확인.

### Phase 4: 대안 분석 (~1.5시간)

두 scout 완료 후 `analyst-alternatives` 호출:

```
Agent(
  subagent_type: "general-purpose",
  model: "opus",
  description: "대안 매트릭스 + 권장안 작성",
  prompt: """당신은 .claude/agents/analyst-alternatives.md 에 정의된 analyst 입니다.

[작업 디렉토리]: C:\_PYTHON\0_CK_Project\0. Research\1. GNN\260505

수행 절차:
1. 본인 정의 읽기: .claude/agents/analyst-alternatives.md
2. 사용 스킬 읽기: .claude/skills/alternatives-verification/SKILL.md
3. 입력 자료 5개 읽기:
   - _workspace/00_input/{PROBLEM,GEMINI_ANSWER,CONTEXT}.md
   - _workspace/01_scout-academic.md (특히 '## 다음 에이전트에게 전달' 섹션)
   - _workspace/02_scout-practical.md (동일)
4. 스킬 절차에 따라 매트릭스 + 권장안 작성
5. 산출물 2개 작성:
   - _workspace/03_alternatives-matrix.md
   - _workspace/04_recommendation.md
6. 04_recommendation.md 마지막에 '## report-author 에게 전달' 섹션:
   - Top 추천 1+1 의 핵심 메시지
   - 보고서 5장(추천)에 강조할 포인트
7. 메인 세션에 200단어 이내 핵심 발견 요약 보고"""
)
```

데이터 부족 시 메인 세션이 해당 scout를 재호출하여 보강 (1회 한정).

### Phase 5: 통합 보고서 작성 (~1시간)

analyst 완료 후 `report-author` 호출:

```
Agent(
  subagent_type: "general-purpose",
  model: "opus",
  description: "통합 보고서 작성",
  prompt: """당신은 .claude/agents/report-author.md 에 정의된 저자 입니다.

[작업 디렉토리]: C:\_PYTHON\0_CK_Project\0. Research\1. GNN\260505

수행 절차:
1. 본인 정의 읽기: .claude/agents/report-author.md
2. 사용 스킬 읽기: .claude/skills/solution-proposal-report/SKILL.md
3. 입력 자료 8개 읽기:
   - _workspace/00_input/ 의 3개
   - _workspace/01_scout-academic.md
   - _workspace/02_scout-practical.md
   - _workspace/03_alternatives-matrix.md
   - _workspace/04_recommendation.md (특히 '## report-author 에게 전달')
   - ../260502/CLAUDE.md (본 논문 핵심 수치 검증용)
4. 스킬 절차에 따라 한국어 보고서 작성
5. 산출물: 노드_요소_도메인불일치_해결방안_보고서.md (작업 디렉토리 루트)
6. 분량 15~25페이지, 한국어 학술·실무 혼합 톤
7. 자체 검증 체크리스트 8개 통과
8. 메인 세션에 위치 + 분량 + Executive Summary 핵심 5~7건 보고"""
)
```

### Phase 6: 사용자 검토 및 정리

1. 메인 세션이 1차 검토:
   - 본 논문 수치 정확성 (580 / 200~400k / MAPE 7.0%)
   - Gemini 3대안 모두 다뤄졌는가
   - 추천 #1 즉시 시작 가능한 구체성
   - 한국어 톤 일관성
2. 사용자에게 보고:
   - 보고서 위치
   - Executive Summary 핵심 5~7건 + 추천 1+1 요약
3. 사용자 피드백 시 해당 에이전트 재호출하여 수정 요청 전달
4. 최종 정리:
   - `_workspace/` 보존 (사후 검증용)
   - 메인 세션 task 모두 completed로 마감

## 데이터 흐름

```
사용자 입력 (이미 _workspace/00_input/ 에 저장됨)
     ↓
[메인 세션 = 오케스트레이터]
     ↓ (Agent 호출 × 2, run_in_background=true)
     ├──> [scout-academic]  ──> 01_scout-academic.md
     │                              └─ '## 다음 에이전트에게 전달'
     └──> [scout-practical] ──> 02_scout-practical.md
                                    └─ '## 다음 에이전트에게 전달'
     ↓ (두 scout 완료 확인 후)
     ↓ (Agent 호출 × 1)
     [analyst-alternatives] ──> 03_alternatives-matrix.md
                            ──> 04_recommendation.md
                                    └─ '## report-author 에게 전달'
     ↓ (analyst 완료 확인 후)
     ↓ (Agent 호출 × 1)
     [report-author] ──> 노드_요소_도메인불일치_해결방안_보고서.md
     ↓
사용자 검토
```

**핵심 차이 (에이전트 팀 모드 대비)**:
- SendMessage 실시간 협업 없음 → 모든 발견은 산출물 파일에 명시 후 다음 에이전트가 Read로 흡수
- 메인 세션이 모든 호출 순서·결과 검증을 직접 책임
- analyst의 추가 조사 요청은 메인 세션이 scout를 재호출하는 형태로 구현

## 에러 핸들링

| 상황 | 전략 |
|---|---|
| 입력 자료(`_workspace/00_input/`) 누락 | Phase 1에서 차단. 사용자에게 하네스 재구성 안내 |
| WebSearch/WebFetch 미로드 | ToolSearch로 사전 로드 안내 후 재시작 |
| Scout 1명 실패 (3회 재시도) | 가용 결과로 진행, 보고서에 "{영역} 일부 데이터 미수집" 명시 |
| Scout 두 명 모두 동일 영역 누락 | analyst가 발견 시 메인 세션이 scout 재호출 |
| analyst 매트릭스 부실 | 1회 재호출 (보강 지시 prompt). 미해결 시 report-author에게 부실 명시 |
| report-author 분량 초과 (25페이지+) | 학술/실무 장 압축, 디테일은 부록으로 |
| report-author 분량 미달 (10페이지 미만) | 5장(추천) 상세도 강화하여 재호출 |
| 사용자 중도 변경 요청 | 현재 Phase 완료 후 적용 |

## 비용 / 시간 추정

- 총 토큰: 150k~350k (4명 에이전트 순차·병렬)
- 총 시간: 3~5시간 (Phase 3 병렬 조사가 가장 큰 비중)
- 산출물: 보고서 1개 + 중간 산출물 4개

## 테스트 시나리오

### 정상 흐름
1. 사용자가 보고서 작성 요청 → 본 스킬 트리거
2. Phase 1에서 입력 자료 / 에이전트 / 스킬 / 도구 모두 확인 ✓
3. Phase 2에서 사용자에게 진행 계획 보고 + 승인
4. Phase 3에서 2 scout 병렬 (~1~2시간)
5. Phase 4에서 analyst 매트릭스 + 권장 (~1.5시간)
6. Phase 5에서 report-author 통합 (~1시간)
7. Phase 6에서 사용자 검토 후 확정
8. 결과: `노드_요소_도메인불일치_해결방안_보고서.md` 생성

### 에러 흐름
1. Phase 3에서 scout-academic이 WebFetch 반복 실패
2. 메인 세션이 산출물 파일 부재 또는 핵심 발견 < 3건 감지
3. 사용자에게 보고: "scout-academic 데이터 수집 제한적. 부분 결과 또는 재시도?"
4. 부분 결과 선택 시 analyst prompt에 "{학술 영역} 데이터 부족" 명시 후 진행
5. 재시도 선택 시 사용자가 도구 로드 또는 검색 채널 변경 후 메인 세션이 재호출

## 종료 후 사용자에게 전달할 정보

1. 보고서 위치
2. Executive Summary 핵심 5~7건 + 추천 1+1
3. `_workspace/` 보존 안내
4. 추가 조사 / 수정 요청 방법 (해당 에이전트 재호출 prompt 안내)
5. 후속 publication 후보 1~2건 (260502 D1~D8 로드맵과의 연계)
