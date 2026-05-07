# WORKFLOWS.md

학습/평가/실험 워크플로우. 코드 구현 시작 시 본 문서를 출발점으로 사용.

## 1. 일반적인 GNN 프로젝트 단계

본 프로젝트가 코드 구현 단계로 진입할 때 예상되는 단계:

1. **데이터 추출 자동화** — Abaqus ODB → 노드별 stress/disp 추출 (Abaqus Python)
2. **데이터 전처리** — mid-node 제거, spoke 영역 추출, graph 변환
3. **데이터 증강** — Random Edge Connection으로 ~2배, Edge augmentation으로 추가
4. **모델 구현** — MeshGraphNets 기반 (PyTorch + PyTorch Geometric 권장)
5. **학습** — Weighted loss, AdamW, 권장 하이퍼파라미터 (ARCHITECTURE.md 참조)
6. **평가** — Top 20% region MAPE + KLD
7. **시각화** — Ground Truth / Prediction / Error contour (Fig. 13 형식)

## 2. 평가 프로토콜 (논문 기준)

### 2.1 Top N% region 평가

```python
# 의사 코드
def topn_mape(pred_stress, gt_stress, percentile=20):
    threshold = np.percentile(gt_stress, 100 - percentile)
    mask = gt_stress >= threshold
    mape = np.mean(np.abs(pred_stress[mask] - gt_stress[mask]) / np.abs(gt_stress[mask]))
    return mape
```

- 논문 기준: top 20% (가장 의미 있는 stress 집중부)
- Top 5%, 10%, 50%, 70%, 100%도 비교 가능 (Fig. 12)

### 2.2 KLD 계산

```python
# stress 분포를 히스토그램으로 추정 후 KLD 계산
def kld(pred_stress, gt_stress, n_bins=100):
    p, edges = np.histogram(pred_stress, bins=n_bins, density=True)
    q, _     = np.histogram(gt_stress,   bins=edges,  density=True)
    eps = 1e-12
    return np.sum(p * np.log((p + eps) / (q + eps)))
```

- 논문 KLD = 0.005 수준 목표

### 2.3 시각화 (Fig. 13)

각 test 샘플에 대해 3개 contour 생성:
- Ground Truth (FE 결과)
- Prediction (GNN 결과)
- Error (절대 차이 또는 percentage error)

스케일은 0~250 (논문 Fig. 13의 colorbar 기준, 단위 추정 MPa).

## 3. 하이퍼파라미터 실험 매트릭스 (논문 Table 2 재현용)

논문 결과를 재현하려면 다음 조합 실험:

| Trial 종류 | N | H | B | Aug | Optimizer |
|---|---|---|---|---|---|
| Baseline | 15 | 64 | 2 | 없음 | AdamW |
| Trial 1 | 20 | 64 | 2 | 없음 | AdamW |
| Trial 2 | 10 | 64 | 2 | 없음 | AdamW |
| Trial 3 | 15 | 128 | 2 | 없음 | AdamW |
| Trial 4 | 15 | 32 | 2 | 없음 | AdamW |
| Trial 5 | 15 | 64 | 4 | 없음 | AdamW |
| Edge Aug 1 | 15 | 64 | 2 | Edge | AdamW |
| Edge Aug 2 | 20 | 64 | 2 | Edge | AdamW |
| **Edge+Data** | **20** | **64** | **2** | **Edge+Data** | **AdamW** |
| Trial 6 | 15 | 64 | 2 | 없음 | Adam (lr=1e-3) |
| Trial 7 | 15 | 64 | 2 | 없음 | SGD (lr=1e-3) |
| Trial 8 | 15 | 64 | 2 | 없음 | AdamW (lr=1e-4) |
| Trial 9 | 15 | 64 | 2 | 없음 | AdamW (lr=1e-2) |

각 Trial을 여러 random seed로 반복 → 평균 ± 표준편차로 보고 (논문 형식).

## 4. 작업 우선순위 (논문 미해결 영역 = 향후 연구)

본 프로젝트에서 사용자가 다음 단계로 진행할 가능성이 높은 작업:

### 4.1 단기 (즉시 가능)
- [ ] 미사용 데이터(745 - 290) 추가 학습으로 정확도 향상
- [ ] Top 20% 외에 더 좁은 region (top 5%, top 10%) 정확도 비교

### 4.2 중기
- [ ] **시계열 GNN** — max stress 시점만이 아닌 전체 충격 시계열 예측
  - MeshGraphNets는 원래 시계열 학습 가능 (rollout)
  - 각 timestep의 stress, displacement를 학습
- [ ] 다른 chassis 부품 적용 (suspension arm 등)
- [ ] 다른 wheel 성능 항목 (강성, NVH 등)

### 4.3 장기
- [ ] 실험 데이터(실제 13도 충격 시험) 통합 — physics-based loss 또는 fine-tuning
- [ ] 다른 backbone 비교 (Equivariant GNN, Transformer 기반 등)
- [ ] Surrogate model로 design optimization 연동

## 5. 환경 설정 (코드 구현 시)

코드 구현 단계에서 권장 환경:

```bash
# Python 3.10+
# PyTorch 2.x (CUDA 지원)
# PyTorch Geometric 2.x

# 필수
pip install torch torchvision torchaudio
pip install torch-geometric
pip install meshio trimesh

# 선택
pip install h5py pyyaml tensorboard wandb
pip install python-pptx  # PPT 추출용
```

**Abaqus Python**:
- Abaqus 내장 Python 사용 (Abaqus 2022+ 권장, Python 3.x)
- `abaqus python script.py` 또는 Abaqus CAE GUI에서 실행

## 6. PDF/논문 재분석이 필요한 경우

본 디렉토리는 시각 분석 캐시를 가지고 있음:
- `ref_paper/_pages/page_XX.png` — 200 DPI 전체 페이지
- `ref_paper/_pages/p{N}_*.png` — 특정 그림/표 고해상도 추출

추가 영역 추출이 필요할 때:
```python
import fitz
doc = fitz.open("ref_paper/[JSAE] Manuscript_ENG_260406(submit).pdf")
mat = fitz.Matrix(5.0, 5.0)  # 5배 확대
clip = fitz.Rect(x0, y0, x1, y1)  # 페이지 좌표계 (A4: 595 x 842)
pix = doc[page_idx].get_pixmap(matrix=mat, clip=clip)
pix.save("ref_paper/_pages/extract.png")
doc.close()
```
