# ARCHITECTURE.md

GNN 모델 구조와 학습 알고리즘 상세. 구현 시 본 문서를 가이드로 참조.

## 1. Backbone: MeshGraphNets

[Pfaff et al., ICLR 2021] 논문의 MeshGraphNets를 backbone으로 채택.

### 1.1 Encoder-Process-Decode 3단계

```
                     [Node features]                    [Per-node]
geometry           ┌────────────┐    ┌────────────┐    ┌────────────┐
material props ───→│  Encoder   │───→│  Message   │───→│  Decoder   │───→ stress (6)
boundary cond.     │            │    │  Passing   │    │            │     deformation (3)
                   │  MLP_n     │    │  ×N steps  │    │  MLP_out   │
                   │  MLP_e     │    │            │    │            │
                   └────────────┘    └────────────┘    └────────────┘
                     [Edge features]
```

### 1.2 입력 특징 (Node / Edge feature)

**Node features** (각 노드마다):
- 공간 좌표 (x, y, z) — geometric information
- Material properties (알루미늄 휠 → 동일하지만 input feature로 보존)
- Boundary condition flag (Impact zone 여부 등)

**Edge features**:
- 원시 mesh edge: 인접 vertex 사이 상대 위치 벡터 (Δx, Δy, Δz), 길이
- 재구성된 augmented edge: 동일하지만 별도 edge type 가능

### 1.3 출력 (per-node)

총 9개 채널 / 노드:
- **Stress** (6 components) — von Mises 계산용 6개 stress tensor 성분 (σ_xx, σ_yy, σ_zz, σ_xy, σ_yz, σ_zx)
- **Deformation** (3 components) — (u_x, u_y, u_z)

평가 시 6개 stress 성분으로 von Mises stress를 계산하여 비교.

## 2. 핵심 개선 기법

### 2.1 Weighted Loss Function

```python
# 의사 코드
def weighted_loss(pred, gt, region_mask, gamma):
    full_loss   = L2(pred, gt)                            # 모든 노드
    region_loss = L2(pred[region_mask], gt[region_mask])  # stress concentration 영역
    return full_loss + gamma * region_loss
```

- `region_mask`: stress 상위 N% 노드 (논문은 top 20% 기준 평가, 학습 시 region 정의는 별도 검토 필요)
- `gamma`: stress 집중부 가중치 (하이퍼파라미터)
- **효과: 10% 성능 향상**

### 2.2 Geometry-aware Edge Augmentation

기본 mesh edge에 추가하여, **Impact zone ↔ Non-Impact zone** 사이 long-range edge를 생성:

```python
# 의사 코드
def augment_edges(node_coords, impact_mask, non_impact_mask, ratio, seed):
    rng = np.random.default_rng(seed)
    impact_nodes     = np.where(impact_mask)[0]
    non_impact_nodes = np.where(non_impact_mask)[0]

    n_aug = int(len(impact_nodes) * ratio)
    src = rng.choice(impact_nodes,     size=n_aug)
    dst = rng.choice(non_impact_nodes, size=n_aug)
    aug_edges = np.stack([src, dst], axis=1)
    return aug_edges
```

- 모든 노드가 아니라 양 영역에서 **랜덤 샘플링**한 노드 사이만 연결
- 하이퍼파라미터: `seed`, `ratio` (선택 비율)
- 다양한 seed로 multiple augmented graph 생성 → train data 다양성 ↑

**Epsilon ball 방식**(선행 연구)과 차이:
- Epsilon ball: 거리 기준, 모든 노드 적용 → 계산 비용 큼
- 본 방식: 의미 있는 두 영역 간만, 일부 노드만 → 비용 ↓ + 효과 ↑

### 2.3 Data Augmentation (누적 학습)

- `Edge Augmented`: 한 가지 augmented graph로 학습
- `Edge + Data Augmented`: 여러 seed의 augmented graph를 누적해서 학습
- Table 2 결과상 **누적 학습이 최고 성능** (Top 20% MAPE 0.070, KLD 0.050)

## 3. 하이퍼파라미터 (논문 검증치)

| 항목 | 권장값 (논문 최적) | 비고 |
|---|---|---|
| Message passing layers (N) | **20** | 15에서 20 증가 시 성능↑ |
| Hidden dimension (H) | **64** | 128로 키우면 미세 개선, 32는 저하 |
| Batch size (B) | **2** | 4로 키워도 큰 차이 없음 (샘플 수 부족) |
| Optimizer | **AdamW** | Adam, SGD 대비 우세 |
| Learning rate | 0.01 (AdamW) | Trial 9 기준; 0.0001은 저하 |
| Weighted loss γ | (논문 미공개) | 별도 튜닝 필요 |
| Edge aug ratio / seed | (논문 미공개) | 별도 튜닝 필요 |

## 4. 평가 지표

### 4.1 MAPE (Mean Absolute Percentage Error)

```
MAPE = (1/N) Σ |y_pred_i - y_gt_i| / |y_gt_i|
```

- von Mises stress에 적용
- **Top 20% region** 한정 평가 (전체보다 stress 집중부 정확도가 실용적 의미)

### 4.2 KLD (Kullback-Leibler Divergence)

- 예측 stress 분포 vs 실제 stress 분포의 차이 측정
- 0에 가까울수록 좋음
- 분포 단위 평가 (점별이 아닌 전체 분포 형태)

### 4.3 R²

- 전체 노드 단위 예측 정확도
- 단순 회귀 지표, 본 연구의 주 지표는 아님 (Top 20% MAPE / KLD가 우선)

## 5. 구현 시 권장 라이브러리

- **PyTorch** + **PyTorch Geometric** (또는 DGL) — GNN 표준
- MeshGraphNets 공식 구현 참고: DeepMind 원저자 코드 기반 다수 PyTorch 포팅 존재
- Mesh I/O: `meshio`, `trimesh`
- Abaqus ODB 파싱: Abaqus 자체 Python API (Abaqus 내부에서 실행)

## 6. 향후 확장 (논문 4장)

구현 시 고려할 확장 방향:
- **시계열 GNN** — 현재는 max stress 시점만, 향후 충격 전체 과정 (multiple time steps) 학습
- **다른 chassis 부품** — suspension arm 등 동일 framework 적용
- **실험 데이터 통합** — physics-based loss 또는 fine-tuning으로 신뢰성 향상
- **추가 학습 데이터** — 미사용 분(745 - 290) + 향후 신규 양산 휠
