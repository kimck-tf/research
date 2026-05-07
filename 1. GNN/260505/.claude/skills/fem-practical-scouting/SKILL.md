---
name: fem-practical-scouting
description: "Abaqus ODB의 응력 데이터 추출 방식(Node/Centroid/IP/Unaveraged), FEM stress recovery 알고리즘(SPR, L2-projection), 상용 surrogate 플랫폼의 응력 처리, ISO 3006/KMVSS 평가 metric을 조사하는 스킬. scout-practical 에이전트가 실무 파이프라인 영향 평가를 위해 사용한다."
---

# FEM Practical Scouting — 실무 파이프라인 조사 절차

본 논문의 자동화된 ODB 추출 파이프라인을 변경할 경우의 영향을 정량 평가하기 위한 실무 자료 수집 절차.

## 사용 시점

scout-practical 에이전트가 `_workspace/02_scout-practical.md` 작성을 시작할 때.

## 사전 준비

1. `_workspace/00_input/{PROBLEM,GEMINI_ANSWER,CONTEXT}.md` 숙지
2. 본 논문 PDF 위치 (`../260502/ref_paper/`) 확인 — Section 2 (Materials & Methods) 의 데이터 추출 절차 부분이 가장 중요
3. WebSearch / WebFetch 가용성 확인

## 조사 절차

### Step 1. Abaqus ODB 응력 추출 방식 4종 비교 (45분)

각 방식별로 다음 정보 수집:

| 방식 | 설명 | Python API | 데이터 부피 (상대) | 시간 손실 (상대) | 정보 충실도 |
|---|---|---|---|---|---|
| Node extrapolated (averaged) | 외삽 + 인접 요소 평균 | `odbAccess.fieldOutputs['S'].getSubset(position=NODAL)` | 1× | 1× | 낮음 (왜곡) |
| Element centroid | 요소 중심 단일값 | `position=CENTROID` | ~M_n/E_c × | 중 | 중간 |
| Integration point (raw) | 적분점 8개 (C3D8 기준) | `position=INTEGRATION_POINT` | ~8× | 높음 | 최고 |
| Unaveraged nodal | 외삽 후 평균 X | `position=ELEMENT_NODAL` | ~Conn × | 중 | 중상 |

소스:
- Abaqus User Manual — "Output requests" / "field output"
- Simulia Knowledge Base
- iMechanica posts

### Step 2. FEM Stress Recovery / Mapping 알고리즘 (40분)

- **SPR (Superconvergent Patch Recovery)**: Zienkiewicz-Zhu 1992 원본, IJNME
- **Improved SPR / SPR with bubble**: Wiberg & Abdulwahab 1993
- **L2-projection**: 변분원리 기반 IP→Node 사상
- **Lumping vs consistent mass**: 단순화 옵션
- 본 논문 noise injection 또는 데이터 증강과 결합 가능성

### Step 3. 상용 / 산업 surrogate 플랫폼의 응력 처리 (30분)

- **Neural Concept**: Geodesic CNN / surface-based, mesh node 기반인지 element 기반인지
- **Altair physicsAI**: SAE 2025-01-8241 — element 응력 직접 처리 여부
- **NVIDIA PhysicsNeMo**: X-MeshGraphNet 의 출력 형식
- **Ansys Discovery AI**: 어떤 도메인에 어떤 형식
- **Siemens Simcenter**: 동등 비교

### Step 4. 본 논문 파이프라인 영향 정량 (30분)

본 논문이 명시한 기존 파이프라인 가정:
- 580 샘플 / 200~400k 노드
- spoke 영역만 (전체 평균 2.4%)
- Python + Abaqus 자동 ODB / PPT 추출

각 변경 시나리오의 추가 비용:
- (S1) Element centroid 추가 추출 → 데이터 부피 +X%, 추출 시간 +Y%
- (S2) IP 추출로 전환 → 데이터 부피 +Z%, 추출 시간 +W%
- (S3) 노드 + centroid 동시 추출 → +V%
- (S4) 기존 노드 응력 + post-recovery (SPR) → 무추가 추출 + 후처리 비용

### Step 5. ISO 3006 / KMVSS / OEM 평가 metric (30분)

- ISO 3006 13도 충격 시험에서 element 응력을 어떻게 사용하는가 (피크값 / 누적 / 영역 평균)
- KMVSS 자동차 안전기준의 휠 강도 항목
- 일반 OEM 사내 기준 (Toyota / GM / 일본 OEM 공개 사례)
- GNN 출력이 평가 로직에 직접 투입 가능한 형태

## 인용 품질 기준

- Abaqus 공식 문서 우선 (버전 명시: 2022, 2023 등)
- 실측 비교 가능한 출처 우선
- 산업 사례는 OEM 공식 발표 / 학회 논문 우선
- `[Abaqus 환경 의존]` 태그 — 다른 솔버에서는 다를 수 있는 정보

## 산출물 작성 규칙

- 형식: `_workspace/02_scout-practical.md`
- 분량: 200~350줄 권장
- 비교 표를 본문 핵심에 배치
- 본 논문 환경(580 샘플, 200~400k 노드) 기준 정량 추정 포함
- Top 1~2 권장 데이터 형식 명시

## 협업 트리거

- **scout-academic에게**: 실무 제약이 학술 가정을 무효화하는 사례 발견 시
- **analyst-alternatives에게**: Gemini 각 대안의 데이터 추출 비용 추정치 제공

## 종료 조건

1. Abaqus ODB 4종 형식 비교 완료
2. 본 논문 파이프라인 변경 시나리오 (S1~S4) 정량 추정 완료
3. ISO 3006 / OEM 평가 metric 정합성 분석
4. Top 1~2 권장 데이터 형식 선정

타임아웃(~2시간) 시 ODB 형식 비교 + 파이프라인 영향만이라도 우선 완수.
