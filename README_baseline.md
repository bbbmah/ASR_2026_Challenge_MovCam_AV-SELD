# 베이스라인 사용 설명서 (Colab)

구글 Colab에서 혼자 실험을 돌리는 사람을 위한 설명서입니다. 코드는 `baseline/seld_challenge/`에 있고, 노트북 `colab_run.ipynb`의 설정 칸만 바꾸면 실험을 돌릴 수 있습니다.

> 코드에는 이 챌린지의 공개 데이터에 없는 데이터를 위한 옵션(`ctrl5`, `fix20`, `train_stages`, `internal_zip_dir` 등)이 남아 있습니다. `configs/base.yaml`의 기본값(`dataset: main20`, `train_stages: 0`) 그대로 사용하면 됩니다.

## 1. 준비물 (드라이브 폴더)

드라이브에 폴더를 하나 만들고(노트북 기본값은 `MyDrive/movcam_avseld`) 아래처럼 둡니다.

```
movcam_avseld/
  dataset/main20/   train.zip  val.zip  test_inputs.zip
  baseline/seld_challenge/   (이 저장소의 baseline/seld_challenge 폴더)
```

- zip은 구글 드라이브에서 받습니다.
- 결과, 체크포인트, 특징 캐시는 `movcam_avseld/seld_runs/` 아래에 자동으로 만들어집니다.

## 2. Colab에서 처음부터 끝까지 돌리는 순서

1. 드라이브의 `baseline/seld_challenge/colab_run.ipynb`를 Colab으로 엽니다.
2. 셀을 위에서부터 차례로 실행한다.
   1. 드라이브 연결과 패키지 설치. `DRIVE_ROOT`가 드라이브의 폴더 경로와 같은지 확인한다.
   2. 설정 칸: `preset`, `seed`, `epochs`, `batch_size`, `lr`, `eval_video` 등을 고른다.
   3. `prepare`: zip을 Colab 로컬 디스크(`/content/data`)에 푼다.
   4. `features`: 오디오와 영상 특징을 뽑아 드라이브에 저장한다.
   5. `train`: 학습한다.
   6. `eval`과 `swaptest`: val 점수를 낸다.
   7. `predict`: test 예측을 만들고 제출용 zip으로 묶는다.
   8. `summary`: 모든 실행 결과를 한 표로 본다.
3. 다른 실험을 하려면 설정 칸만 바꾸고 `prepare`부터 다시 실행하면 됩니다. 이미 한 일(zip 풀기, 특징 뽑기)은 건너뛸 수 있습니다.

## 3. 설정의 뜻

| 이름 | 뜻 | 가능한 값 |
|---|---|---|
| `preset` | 모델 종류(아래 표) | `B1`, `B2`, `B2-0`, `B3` |
| `seed` | 난수 시드. 결과를 여러 번 비교하려면 1, 2, 3으로 | 정수 |
| `epochs` | 학습 횟수 | 기본 200 |
| `batch_size` | 한 번에 학습하는 클립 수 | `auto`(64) 또는 숫자 |
| `lr` | 학습률 | 기본 0.001 |
| `eval_video` | 평가 때 영상을 바꾸는 방법 | `real`(그대로), `zeros`(0), `shuffle`(다른 클립 영상) |
| `select_metric` | 가장 좋은 모델을 고르는 기준(`--set`으로) | `F_ext`(기본), `F_basic` |

세부 옵션은 `configs/base.yaml`에 한국어 주석과 함께 설명되어 있습니다. 개별 옵션(`modality`, `video_input`, `video_crop`, `av_time_window`)은 비워 두면 프리셋 값을 사용합니다.

### 프리셋

| 프리셋 | 모델 | 영상 입력 | 쓰는 이유 |
|---|---|---|---|
| `B1` | 소리만 | 없음 | 기준선 |
| `B2` | 영상 + 소리 (원래 모델) | 진짜 영상, 화면 가운데만 | 영상을 쓰는 기준선 |
| `B2-0` | B2와 같은 구조, 영상 특징을 0으로 | 0 | 구조 효과와 영상 효과를 나누기 위한 대조군 |
| `B3` | B2 + 화면 전체 사용 + 시간 창 제한(±2프레임) | 진짜 영상, 화면 전체 | 영상을 더 잘 쓰게 한 모델 |


## 4. 결과 파일 읽는 법

모든 결과는 `seld_runs/` 아래에 있습니다.

| 파일 | 내용 |
|---|---|
| `runs/{실행이름}/train_log.csv` | 에폭별 학습 손실, val 손실, val 점수 |
| `runs/{실행이름}/best_model.pth` | val 점수가 가장 좋은 모델 |
| `runs/{실행이름}/last.pth` | 가장 최근 에폭(이어 하기용) |
| `runs/{실행이름}/config_used.yaml` | 그 실행에 쓴 설정 전체 |
| `runs/{실행이름}/scores_val_{real,shuffle,zeros}.json` | val 점수(전체와 클래스별) |
| `runs/{실행이름}/submission_*.zip` | test 예측 csv 묶음 (제출용) |
| `results/summary.csv` | 모든 실행의 점수 표 |
| `results/gaps.csv` | "영상 덕분에 오른 점수" |

실행 이름은 `{dataset}_{preset}_s{seed}`입니다(예: `main20_B2_s1`). 프리셋과 다르게 바꾼 옵션이 있으면 뒤에 붙습니다.

### 점수 두 종류

- ext (확장 점수): 방위각을 −180°~180° 그대로 두고 원형 차이로 비교합니다. 순위와 모델 고르기에 이 점수를 사용합니다.
- basic (기본 점수): 정답과 예측의 방위각을 모두 −90°~90°(앞쪽)로 옮겨 비교합니다.
- 열 이름: `F`(위치까지 맞힌 탐지 점수, 높을수록 좋음), `DOA`(방향 오차, 도, 낮을수록 좋음), `relDist`(상대 거리 오차), `onscreen`(화면 안/밖 정확도, 영상 모델만). 소리만 쓰는 B1은 onscreen이 `NA`(비어 있음)입니다.
- 점수를 직접 계산하는 방법은 `evaluation/README.md`를 확인하십시오.


## 5. test 제출 파일 만들기

`predict`가 test 입력으로 예측 csv를 만들고 `runs/{실행이름}/submission_*.zip`으로 묶습니다. 형식은 챌린지 페이지 2.4, 2.5절을 확인하십시오. test 정답은 공개하지 않으므로 이 단계에서는 점수가 나오지 않습니다.
