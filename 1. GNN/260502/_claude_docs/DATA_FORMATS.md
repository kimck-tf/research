# DATA_FORMATS.md

데이터 파이프라인: Abaqus 분석 결과 → graph 학습 데이터.

## 1. 원본 데이터 (raw)

### 1.1 Abaqus ODB (Output Database)

- Abaqus 13도 충격 동적 분석 결과
- 휠당 2~4개 impact load direction
- 추출 대상: **maximum stress 발생 시점**의 모든 노드 데이터
  - Stress (6 components)
  - Displacement (3 components)
- Abaqus Python API로 접근 (Abaqus 내부에서 실행되는 스크립트)

### 1.2 Analysis Report (PPT)

- 분석 결과 요약 문서
- 추출 대상:
  - Wheel specification (디자인 정보, 재질 등)
  - Striker mass (충격 시 사용된 striker 무게)
  - Maximum stress 값

### 1.3 데이터 규모 (논문 시점)

| 단위 | 수량 |
|---|---|
| Total wheels | 238 |
| Total analysis results (모든 충격방향 합) | 745 |
| 본 연구 사용 wheels | 85 |
| 본 연구 사용 analysis | 290 |
| Augmented samples (~2배) | 580 |

## 2. 추출 파이프라인 (Fig. 7)

```
┌──────────────┐     ┌──────────────────────────┐
│ ODB files    │────→│ Abaqus Python script     │──→ stress, displacement
└──────────────┘     │ (Abaqus 내부 실행)       │
                     └──────────────────────────┘

┌──────────────┐     ┌──────────────────────────┐
│ PPT files    │────→│ Python script (외부)     │──→ wheel spec, striker mass
└──────────────┘     │ (python-pptx 등)         │
                     └──────────────────────────┘
```

**자동화 원칙** (논문 명시):
- 향후 데이터 추가 시 재사용 가능하도록 설계
- 논문 시점 이후 신규 양산 휠 데이터를 누적 학습할 계획

## 3. 전처리 (Section 2.3)

### 3.1 Mid-node 제거

- Abaqus mesh = quadratic element (2차 요소) 기반
  - 각 폴리곤의 vertex 사이에 mid-node가 추가되어 있음
  - 형상 정확도 ↑ (FE 분석용)
- **Mid-node는 graph 학습에 기여 없음** → 제거
- 1차 vertex 노드만 graph 노드로 사용

### 3.2 Spoke 영역 추출 (Rim 제외)

- Rim: 휠 바깥 림 부분, 형상 변화 적고 강도 영향 작음
- **Spoke**: 강도 분석의 주 관심 영역 (실제 파괴 발생부)
- Rim 노드를 학습 대상에서 제외 → 데이터 크기 평균 **2.4%** 수준으로 축소

### 3.3 Random Edge Connection (~2배 augmentation)

- 290개 분석 결과 → ~580개 학습 샘플
- 본 단계에서 random edge connection 적용 (이후 모델 학습 시 추가 edge augmentation과 별개)

### 3.4 Train/Test 분할 (Fig. 8)

**원칙**: 같은 휠 형상이 train과 test 양쪽에 들어가지 않음.

```
                 Wheel  | Impact load | Loading direction
        ┌──────  1      |             |  θ₀
        │        2      | A           |  θ₁
Train   │        3      |             |  θ₂
data    │        4      |             |  θ₃
        │        5      | B           |  θ₄
        └──────  6      |             |  θ₅
        ┌──────  ...    |  ...        |  ...
        │        578    |             |  θ₆
Test    │        579    | Z           |  θ₇
data    └──────  580    |             |  θ₈
```

- Train: 524 samples (77 wheels)
- Test: 56 samples (8 wheels)

## 4. Graph 변환

### 4.1 Mesh → Graph 매핑

| Mesh 요소 | Graph 요소 |
|---|---|
| Vertex (1차 노드) | Node |
| Polygon edge | Edge |
| Vertex 좌표 | Node feature (x, y, z) |
| Vertex별 BC, material | Node feature |
| Edge 길이, 방향 벡터 | Edge feature |

### 4.2 Augmented Edge (Impact ↔ Non-Impact)

- 학습 시점에 동적으로 생성 (또는 사전 생성 후 캐싱)
- Mask 정보 필요:
  - `impact_mask`: striker 직접 접촉부 노드
  - `non_impact_mask`: 중심부 노드
- 일부 노드만 랜덤 샘플링 → edge 생성 (자세한 알고리즘은 ARCHITECTURE.md 참조)

## 5. 추천 저장 포맷 (구현 시)

논문에는 명시 없지만, 일반 GNN 프로젝트 관행 기준:

```
data/
├── raw/                    # ODB, PPT 원본 (보안 자산 - .gitignore 필수)
├── extracted/              # ODB → numpy/HDF5 추출본 (노드별 stress/disp)
│   └── wheel_{id}_dir_{theta}.h5
├── processed/              # graph 변환 후 (PyTorch Geometric Data 객체)
│   └── wheel_{id}_dir_{theta}.pt
└── splits/
    ├── train.txt           # 위 파일 ID 리스트
    └── test.txt
```

**보안 주의**: 
- `data/raw/` 내 ODB와 PPT는 양산 휠 자산. **절대 커밋 금지**.
- `.gitignore`에 `data/raw/` 명시 필수.
- 프로세스 후 `data/processed/`도 internal 자료로 취급 권장.

## 6. 자주 마주칠 이슈

- **노드 수 가변**: 휠마다 200~400k 노드 → batch 처리 시 padding/masking 필요
- **메모리**: 대규모 graph는 GPU 메모리 한계 — 논문 batch=2 사용 이유
- **좌표계**: Abaqus 좌표계가 휠마다 다를 수 있음 — 정규화 절차 필요 (휠 중심 정렬, 회전 정합)
- **단위**: stress는 Pa 또는 MPa — 학습 시 정규화 권장 (분포 차이 큼)
