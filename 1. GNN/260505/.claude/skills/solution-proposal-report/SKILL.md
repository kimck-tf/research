---
name: solution-proposal-report
description: "노드/요소 도메인 불일치 해결 방안 보고서를 4개 산출물(scout 학술/실무, analyst 매트릭스/권장)을 통합하여 한국어 학술·실무 혼합 톤으로 작성하는 스킬. report-author 에이전트가 사용한다. 분량 15~25페이지, 본 논문 저자가 즉시 활용 가능한 구체성 요구."
---

# Solution Proposal Report — 통합 보고서 작성 절차

4개 산출물(`_workspace/01~04`)을 한국어 단일 보고서로 통합하는 절차.

## 사용 시점

report-author 에이전트가 4개 산출물 모두 완료된 후 작업을 시작할 때.

## 사전 준비

1. 4개 산출물 모두 Read:
   - `_workspace/01_scout-academic.md`
   - `_workspace/02_scout-practical.md`
   - `_workspace/03_alternatives-matrix.md`
   - `_workspace/04_recommendation.md`
2. `_workspace/00_input/` 3개 파일 재확인
3. 본 논문 핵심 수치 확인 (`../260502/CLAUDE.md` 또는 `_claude_docs/PAPER_SUMMARY.md`)

## 작성 절차

### Step 1. Executive Summary 우선 작성 (30분)

원칙: 본 논문 저자가 1페이지만 읽어도 행동 가능해야 함.

포함 요소:
- 문제 본질 3줄 (FEM-GNN 도메인 차이)
- 검토한 대안 N개 (Gemini 3 + 추가 N) 한 줄씩
- **최우선 추천 1+1** (단기 quick win + 중기 publication)
- 예상 효과 정량 (정확도 향상 / 시간 단축 / 데이터 비용 증가)
- 다음 행동 항목 3건 (이번 주 / 이번 달 / 이번 분기)

### Step 2. 1장 문제 정의 (1~2페이지)

`_workspace/00_input/PROBLEM.md` 기반으로 학술 톤 재작성:
- FEM 메커니즘과 GNN 학습 데이터 차이를 도식 + 문장으로
- 노드 응력 외삽 왜곡 메커니즘 (단계별)
- 본 논문 적용 시 발생하는 구체적 문제 (Top-20% MAPE 7%가 element 응력 기준이면 어떤가)
- 본 보고서 범위 (학술 + 실무 통합)

### Step 3. 2장 학술 동향 (3~5페이지)

`_workspace/01_scout-academic.md` 통합. 다음 6개 절로 구성:
- 2.1 Element-aware GNN 발전사
- 2.2 Bipartite / Heterogeneous Graph 사례
- 2.3 Dual Graph 사례
- 2.4 IP / Gauss point 직접 예측
- 2.5 Edge / Face attribute regression
- 2.6 FEM 물리 명시 통합 (B-matrix, shape function)

각 절 마무리에 "본 논문 적용 가능성" 한 단락.

### Step 4. 3장 실무 파이프라인 검토 (3~5페이지)

`_workspace/02_scout-practical.md` 통합. 다음 5개 절:
- 3.1 Abaqus ODB 응력 데이터 형식 비교 (4종 표)
- 3.2 FEM Stress Recovery 알고리즘
- 3.3 상용 surrogate 플랫폼 비교
- 3.4 본 논문 파이프라인 변경 영향 (S1~S4 시나리오)
- 3.5 ISO 3006 / KMVSS 평가 metric 정합성

### Step 5. 4장 대안 비교 (3~5페이지)

`_workspace/03_alternatives-matrix.md` 통합:
- 4.1 검토 대안 목록 (각 대안 1~2단락 + 학술/실무 ref)
- 4.2 6축 비교 매트릭스 (전체 표를 부록 B로 이동, 본문은 핵심 추출 표)
- 4.3 Gemini 3대안 비판적 평가
- 4.4 본 논문 환경 시뮬레이션 결과 (정량 표)

### Step 6. 5장 추천 및 로드맵 (3~5페이지)

`_workspace/04_recommendation.md` 통합:
- 5.1 단기 Quick Win — 추천 #1 상세
  - 대안 + 변형, 근거, 구현 단계 (의사코드 부록 C로 이동), 성공 지표, 리스크
- 5.2 중기 학술 publication — 추천 #2 상세
- 5.3 단계별 마이그레이션 경로
- 5.4 1/3/6개월 마일스톤 표
- 5.5 리소스 / 외부 협업 (UC Berkeley, NVIDIA PhysicsNeMo 등)

### Step 7. 6장 결론 (1~2페이지)

- 6.1 종합 평가 (3줄)
- 6.2 즉시 착수 사항 (4건, 우선순위 순)
- 6.3 후속 publication 후보 1~2건
- 6.4 260502 D1~D8 로드맵과의 연계
  - 본 작업이 D1 (SOTA Baseline ablation)의 ablation 항목으로 자연스럽게 흡수
  - D5 (Multi-scale GNN)와 결합 시 시너지

### Step 8. 부록 작성 (4종)

- 부록 A. 참고 문헌 (학술 + 실무)
- 부록 B. 6축 비교 매트릭스 전문
- 부록 C. 추천 #1 의 상세 의사코드 / 모듈 구조
- 부록 D. 약어 정리

## 산출물 위치

`노드_요소_도메인불일치_해결방안_보고서.md` (작업 디렉토리 루트)

## 작성 원칙 (재확인)

- 한국어 학술·실무 혼합 톤
- 본 논문 핵심 수치 정확 인용 (580 샘플, 200~400k 노드, MAPE 7.0%)
- 영어 약어는 첫 등장 시 한국어 보충
- 표 적극 활용 (각 장에 1개 이상)
- 인용 형식: 본문 `[저자, 연도]` / 부록 A에 전체 출처

## 자체 검증 체크리스트

작성 완료 후 자체 검증 (수정 1회 한정):

- [ ] Executive Summary가 1페이지를 넘지 않음
- [ ] 본 논문 핵심 수치 정확
- [ ] Gemini 3대안 모두 명시적으로 다뤄짐 (검증 결과 포함)
- [ ] 추가 대안 ≥1개 제시
- [ ] 추천 #1이 1~2개월 내 시작 가능한 구체성
- [ ] 부록 A의 reference가 본문 인용과 1:1 매칭
- [ ] 약어가 부록 D에 정리
- [ ] 분량 15~25페이지 (대략 7,500~12,500단어)

## 협업

- 작성 중 데이터 부족 시 analyst-alternatives에게 SendMessage로 명확화 요청 (1회 한정)
- 작성 완료 시 리더(오케스트레이터)에게 위치 + 분량 + 핵심 발견 알림

## 종료 조건

체크리스트 8개 통과 + 자체 검증 완료. 타임아웃 시 본문 6장만이라도 완성하고 부록은 다음 단계로.
