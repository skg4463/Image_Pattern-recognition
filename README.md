# pattern_midterm

1. **이미지 분할 / 분류** — 수원 지역 항공(위성) 영상(`suwon.jpg`)의 픽셀을 RGB 색상 기준으로 4개 내외 영역으로 나누고, **K-Means**, **Mean Shift**, **MLE(가우시안 혼합 모델)** 결과를 비교합니다.
2. **텍스처 분석** — **Laws' Texture Energy Measures**의 5×5 필터 25개를 이미지와 동영상 프레임에 적용해 텍스처 에너지 맵을 만듭니다.

## 결과

| K-Means | Mean Shift | MLE (GMM) |
|:---:|:---:|:---:|
| ![K-Means](result/Figure_1.png) | ![Mean Shift](result/Figure_2.png) | ![MLE](result/Figure_3.png) |

- **K-Means**: 클러스터 4개. 산림(노랑), 도로·하천, 시가지가 비교적 고르게 나뉩니다.
- **Mean Shift**: 클러스터 수를 자동으로 정하며, 이 영상에서는 크게 3개(산림 / 시가지·하천 / 밝은 지붕 등)로 묶였습니다.
- **MLE (GMM)**: 가우시안 4개의 혼합 모델로 분류. K-Means보다 영역 경계가 더 덩어리지게 나타납니다.

## 파일 구성

```
.
├── k-means.py                        # K-Means 이미지 분할
├── mean-shift.py                     # Mean Shift 이미지 분할
├── Maximum Likelihood Estimation.py  # K-Means 사전확률 + GMM(MLE) 분류
├── texture.py                        # Laws 텍스처 에너지: 단일 이미지 → 9개 맵 시각화
├── Laws Texture Energy Measures.py   # Laws 텍스처 에너지: 동영상 전 프레임 → energy_maps/에 저장
├── Laws Texture Energy Measures2.py  # Laws 텍스처 에너지: 동영상 전 프레임 → 한 화면에 시각화
├── suwon.jpg                         # 분할/분류 입력: 수원 항공 영상 (489×365)
├── 104591554.4.jpg                   # 텍스처 입력 이미지 (510×340)
├── 123.webm                          # 텍스처 입력 동영상
├── energy_maps/                      # 동영상 텍스처 에너지 맵 결과 (172프레임 × 25맵 = 4,300장)
└── result/                           # 분할/분류 결과 그림 (Figure_1~3)
```

## 1. 이미지 분할 / 분류

세 스크립트 모두 같은 전처리를 합니다.

1. `suwon.jpg`를 읽어 0~1 범위로 정규화
2. `(H×W, 3)` 형태로 펼쳐 **각 픽셀의 RGB 값을 특징 벡터**로 사용
3. 클러스터 라벨을 원래 이미지 크기로 되돌려 `viridis` 컬러맵으로 표시

| 스크립트 | 방법 | 주요 설정 |
|---|---|---|
| `k-means.py` | `sklearn.cluster.KMeans` | `n_clusters=4`, `random_state=0` |
| `mean-shift.py` | `sklearn.cluster.MeanShift` | 대역폭은 `estimate_bandwidth(quantile=0.1, n_samples=500)`로 추정, `bin_seeding=True` |
| `Maximum Likelihood Estimation.py` | K-Means → `GaussianMixture` | K-Means(4개) 결과로 클래스별 **사전확률**을 계산해 출력하고, `covariance_type='full'`인 GMM을 EM으로 학습(최대우도 추정)해 각 픽셀을 분류 |

```bash
python k-means.py
```

```bash
python mean-shift.py
```

```bash
python "Maximum Likelihood Estimation.py"
```

## 2. Laws' Texture Energy Measures

1차원 5탭 벡터 5개의 외적으로 5×5 필터 **25개**를 만듭니다.

| 벡터 | 값 | 감지하는 특성 |
|---|---|---|
| L5 (Level) | `[1, 4, 6, 4, 1]` | 평균 밝기 |
| E5 (Edge) | `[-1, -2, 0, 2, 1]` | 에지 |
| S5 (Spot) | `[-1, 0, 2, 0, -1]` | 점 |
| W5 (Wave) | `[-1, 2, 0, -2, 1]` | 물결 |
| R5 (Ripple) | `[1, -4, 6, -4, 1]` | 잔물결 |

필터 순서는 `L5L5, L5E5, …, R5R5`이며, 입력을 흑백으로 바꾼 뒤 각 필터로 `cv2.filter2D`를 적용하고 절댓값을 에너지 맵으로 사용합니다.

| 스크립트 | 입력 | 출력 |
|---|---|---|
| `texture.py` | `104591554.4.jpg` | 25개 중 앞의 **9개** 맵(`L5L5`~`E5W5`)을 3×3 격자로 표시 |
| `Laws Texture Energy Measures.py` | `123.webm` 전체 프레임 | `energy_maps/frame_{프레임번호}_energy_map_{필터번호}.png` 로 저장 |
| `Laws Texture Energy Measures2.py` | `123.webm` 전체 프레임 | 프레임 × 25 격자로 한 화면에 표시 (프레임이 많으면 화면이 매우 큼) |

`energy_maps/`의 필터 번호(0~24)는 위 벡터 순서대로 `행 벡터 × 5 + 열 벡터` 입니다. 예: `energy_map_6` = E5E5, `energy_map_12` = S5S5.

```bash
python texture.py
```

```bash
python "Laws Texture Energy Measures.py"
```

## 실행 환경

```bash
pip install numpy matplotlib opencv-python scikit-image scikit-learn
```

모든 스크립트는 입력 파일을 상대 경로로 읽으므로 **레포 루트에서 실행**해야 합니다.
