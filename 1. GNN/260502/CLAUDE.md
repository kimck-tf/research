# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

알루미늄 휠 13도 동적 충격 시험의 응력(stress) 분포를 **Graph Neural Network(GNN)** 으로 예측하는 연구 프로젝트.
사용자(Changgon Kim, 현대자동차)가 저자로 참여한 JSAE 논문(`ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf`)을 출발점으로 하며, 본 디렉토리는 후속 구현/실험 작업을 위한 작업공간이다.

- **본 시리즈 위치**: `0. Research/1. GNN/260502` (GNN 연구 시리즈, 2026-05-02 시작)
- **선행 작업**: APL 2023 논문 (Jin, Zheng, Kim, Gu) — 단순 휠 형상에서 R²=0.965 달성
- **본 연구의 차별점**: 양산(production-spec) 휠의 high-fidelity FE 데이터(노드 200~400k)로 확장, dynamic impact analysis 결과 사용

## 현재 디렉토리 상태

```
260502/
├── CLAUDE.md            ← 본 파일
├── _claude_docs/        ← 상세 문서 (작업 진입 시 트리거 참조)
└── ref_paper/
    ├── [JSAE] Manuscript_ENG_260406(submit).pdf
    └── _pages/          ← PDF 페이지 PNG (시각 정보 분석용 캐시)
```

코드는 아직 없음. 작업이 시작되면 일반적으로 다음 구조가 예상된다:
- `data/` — Abaqus ODB/PPT에서 추출한 raw + 전처리된 graph 데이터
- `src/` — 데이터 전처리 / GNN 모델 / 학습 / 평가 스크립트 (PyTorch, PyTorch Geometric 추정)
- `scripts/` — Abaqus Python 스크립트 (ODB 자동 추출용)
- `experiments/` — 하이퍼파라미터 튜닝 결과, 체크포인트

## 핵심 수치 (논문 참조용 - 변경 시 주의)

| 항목 | 값 |
|---|---|
| Train / Test 휠 분할 | 77 / 8 (휠 형상 중복 없음) |
| Train / Test 샘플 수 | 524 / 56 (총 580, ~2배 augmentation 후) |
| 노드 수 (양산 사양) | 200,000 ~ 400,000 |
| 모델 출력 | 노드당 stress 6 + deformation 3 |
| 최종 성능 (Top 20% region) | MAPE 7.0%, KLD 0.005 |
| 최적 하이퍼파라미터 | N=20 layers, H=64, B=2, AdamW, Edge+Data Aug |

## 작업 트리거 (반드시 먼저 읽을 문서)

- **논문 내용 / 연구 배경 질문 시** → `_claude_docs/PAPER_SUMMARY.md` 먼저 읽기
- **GNN 모델 구조 / MeshGraphNets 관련 작업 시** → `_claude_docs/ARCHITECTURE.md` 먼저 읽기
- **Abaqus ODB → graph 변환 / 데이터 전처리 작업 시** → `_claude_docs/DATA_FORMATS.md` 먼저 읽기
- **학습 / 평가 / 하이퍼파라미터 실험 작업 시** → `_claude_docs/WORKFLOWS.md` 먼저 읽기

## 도메인 용어 (혼동 주의)

- **13-degree impact test** — 13도 각도로 striker(barrier)를 휠에 낙하시키는 표준 강도 시험 (실차 조건 모사)
- **Spoke / Rim** — 본 연구는 응력이 집중되는 spoke 영역만 graph로 변환 (rim 제외 → 데이터 2.4%로 축소)
- **Quadratic element / Mid-node** — Abaqus 2차 요소의 중간 노드는 graph 학습에 미사용 (1차 노드만)
- **Impact zone / Non-Impact zone** — striker 직접 접촉부(boundary) vs 휠 중심부. Edge augmentation에서 두 영역 간 long-range edge 생성
- **Top N% stress region** — 평가 시 응력 상위 N% 노드에 한정 (논문은 top 20% 기준)

## 외부 시스템 / 도구 의존성

- **Abaqus** — 데이터 생성(13도 충격 동적 해석). 결과는 `.odb` 파일에 저장됨
- **Python + Abaqus Scripting** — ODB/PPT 자동 추출 파이프라인 (반복 데이터 추가 가능하도록 재사용 가능하게 작성됨)
- **MeshGraphNets** (Pfaff et al., ICLR 2021) — 베이스 아키텍처
- **GPU** — 학습 자원이 제한 요인 (논문 시점에 미사용 데이터 다수 존재)

## 주의사항

- 본 작업공간은 양산 차량 개발 데이터를 사용하므로, 실제 휠 설계 / ODB 파일은 보안 자산임. 외부 공유 / 커밋 전 항상 확인 요망.
- `ref_paper/_pages/` 는 PDF 분석용 임시 PNG 캐시. 필요 시 재생성 가능 (PyMuPDF 사용).
- 사용자는 논문 저자 본인이므로, 논문 내용 관련 질문에는 단순 요약보다 **구현/확장 관점**의 답변이 유용하다.
