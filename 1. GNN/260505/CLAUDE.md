# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

본 작업공간은 **MeshGraphNet의 Node-Element 도메인 불일치 문제** 의 학술·실무 통합 해결 방안을 도출하기 위한 하네스 기반 보고서 작성 워크스페이스이다.

- **본 시리즈 위치**: `0. Research/1. GNN/260505` (선행: `260502`)
- **대상 논문**: `../260502/ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf` (Changgon Kim et al., JSAE 2026)
- **핵심 문제**: FEM은 Integration Point에서 응력 계산 → 노드 응력은 외삽 왜곡됨. 파단 평가는 Element Von Mises 기반. GNN(MGN) 학습은 노드 위치 → 도메인 불일치
- **목표 산출물**: `노드_요소_도메인불일치_해결방안_보고서.md` (15~25페이지, 한국어, 추천 1+1 포함)

## 디렉토리 구조

```
260505/
├── CLAUDE.md                    ← 본 파일
├── .claude/
│   ├── agents/                  ← 에이전트 정의 4개
│   │   ├── scout-academic.md
│   │   ├── scout-practical.md
│   │   ├── analyst-alternatives.md
│   │   └── report-author.md
│   └── skills/                  ← 스킬 5개 (작업 4 + 오케스트레이터 1)
│       ├── fem-aware-academic-scouting/SKILL.md
│       ├── fem-practical-scouting/SKILL.md
│       ├── alternatives-verification/SKILL.md
│       ├── solution-proposal-report/SKILL.md
│       └── node-element-orchestrator/SKILL.md
├── _workspace/
│   ├── 00_input/
│   │   ├── PROBLEM.md           ← 문제 정의
│   │   ├── GEMINI_ANSWER.md     ← Gemini 사전 답변 (검증 대상)
│   │   └── CONTEXT.md           ← 본 논문 + 260502 컨텍스트
│   ├── 01_scout-academic.md     ← scout-academic 산출물 (실행 시)
│   ├── 02_scout-practical.md    ← scout-practical 산출물
│   ├── 03_alternatives-matrix.md ← analyst 산출물 1
│   └── 04_recommendation.md     ← analyst 산출물 2
└── 노드_요소_도메인불일치_해결방안_보고서.md  ← 최종 산출물 (실행 시)
```

## 하네스 실행 방법

본 하네스는 4명 서브 에이전트(scout-academic, scout-practical, analyst-alternatives, report-author)로 구성된다.

### 실행 모드: 서브 에이전트 (Agent 도구 직접 호출)

Windows 네이티브 환경 호환을 위해 **서브 에이전트 모드**를 채택. 오케스트레이터(메인 세션)가 `Agent` 도구로 4명을 순차·병렬 호출하며, 에이전트 간 협업은 산출물 파일의 `## 다음 에이전트에게 전달` 섹션을 통해 이루어진다.

(에이전트 팀 모드 — TeamCreate + SendMessage 실시간 협업 — 는 Windows 네이티브에서 작동하지 않는 Bun SFE 버그(GitHub Issue #26244)로 제외됨. WSL+tmux 환경에서는 별도 변형으로 전환 가능)

### 트리거

1. **자동**: 사용자가 "보고서 작성", "노드 요소 불일치 분석", "Element-wise decoding 검토" 등 입력 시 `node-element-orchestrator` 스킬이 자동 호출됨
2. **명시**: `Skill(node-element-orchestrator)` 직접 실행

총 소요 시간: 3~5시간 (Phase 3 병렬 조사 ~1~2시간, Phase 4 분석 ~1.5시간, Phase 5 통합 ~1시간)

## 작업 트리거 (반드시 먼저 읽을 문서)

- **문제 정의 / 작업 시작 시** → `_workspace/00_input/PROBLEM.md` 먼저 읽기
- **Gemini 답변 검토 / 비판 시** → `_workspace/00_input/GEMINI_ANSWER.md` 먼저 읽기
- **본 논문 컨텍스트 / 260502 연계 시** → `_workspace/00_input/CONTEXT.md` + `../260502/CLAUDE.md` 읽기
- **에이전트 역할 확인 시** → `.claude/agents/{name}.md`
- **스킬 절차 확인 시** → `.claude/skills/{name}/SKILL.md`

## 본 논문 핵심 수치 (참조용 - 변경 시 주의)

| 항목 | 값 |
|---|---|
| Train / Test 휠 분할 | 77 / 8 (휠 형상 중복 없음) |
| Train / Test 샘플 수 | 524 / 56 (총 580) |
| 노드 수 (양산 사양) | 200,000 ~ 400,000 |
| 모델 출력 | 노드당 stress 6 + deformation 3 |
| 최종 성능 (Top 20% region) | MAPE 7.0%, KLD 0.005 |
| 백본 | MeshGraphNets, N=20 layers, H=64, B=2 |

## 도메인 용어 (혼동 주의)

- **Integration Point (IP)**: FEM 요소 내부의 적분점 (Gauss point). 응력·변형률은 IP에서 계산
- **Node extrapolated stress**: IP 응력을 노드로 외삽 + 인접 요소 평균. 본 논문이 학습에 사용
- **Element centroid stress**: 요소 중심점의 응력 (단일 값)
- **Unaveraged nodal stress**: 외삽 후 평균화 X (요소별 분리)
- **SPR (Superconvergent Patch Recovery)**: Zienkiewicz-Zhu 1992. IP→Node 사상의 표준 알고리즘
- **B-matrix**: FEM 변형률-변위 관계 행렬 (shape function gradient)
- **Bipartite graph**: 두 종류 vertex (Node + Element)가 공존하는 그래프

## 외부 시스템 / 도구 의존성

- **Abaqus** — 데이터 생성 및 응력 추출 (`odbAccess` Python API)
- **WebSearch / WebFetch** — 학술 reference 발굴 (필요 시 ToolSearch로 사전 로드)
- **TeamCreate / TaskCreate / SendMessage** — 에이전트 팀 운영 (필요 시 ToolSearch로 사전 로드)

## 주의사항

- 본 작업공간은 양산 차량 개발 데이터 컨텍스트를 사용하므로, 외부 공유 / 커밋 전 항상 확인 요망
- `_workspace/` 디렉토리는 사후 검증·감사 추적을 위해 작업 완료 후에도 보존
- Gemini 답변에 무비판적 동의 금지 — 학술 정당화 부족한 부분은 반드시 명시적으로 표시
- 본 논문 baseline 코드 재활용율은 추천 우선순위의 핵심 평가 기준
