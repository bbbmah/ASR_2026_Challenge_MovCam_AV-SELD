# Moving-Camera AV-SELD Challenge

카메라가 회전하는 영상과 스테레오 음향을 입력으로 받아 소리 이벤트의 종류, 방위각, 거리를 추정하는 챌린지입니다.

DCASE 2025 Task 3(스테레오 AV-SELD)에서는 음향에 영상을 추가해도 성능이 크게 오르지 않음을 확인할 수 있었습니다. 카메라가 비디오 전반에서 고정되어 있고, 길이도 5초로 짧아 영상을 활용할 기회가 적다고 판단하여 같은 STARSS23 원본으로 카메라가 움직이는 20초 클립과 데이터셋과 평가 방법을 만들었습니다. 태스크, 데이터, 평가 지표, 규칙, 리더보드는 챌린지 페이지(`index.html`)에 정리되어 있습니다.

## 저장소 구성

| 위치 | 내용 |
|---|---|
| `index.html` | 챌린지 소개 페이지 |
| `dataset/` | 데이터셋 설명(`README_dataset.md`)과 데이터셋 다운로드를 위한 구글 드라이브 링크 |
| `baseline/seld_challenge/` | 베이스라인 학습, 추론 코드와 Colab 노트북(`colab_run.ipynb`) |
| `README_baseline.md` | 베이스라인 사용 설명서 |
| `evaluation/` | 평가 코드(`score.py`, `metrics.py`, `seld_label_utils.py`)와 사용 설명서 |

## 구글 드라이브 링크
링크: (추가 예정)

## 빠른 시작

1. 데이터 받기. 구글 드라이브에서 `val.zip`, `test_inputs.zip`, `train.zip`를 받습니다.
2. 베이스라인 학습. `baseline/seld_challenge/colab_run.ipynb`를 Colab에서 열고 위에서부터 실행합니다. 폴더 배치와 설정은 `README_baseline.md`를 확인하십시오.
3. val 채점. 예측 csv 폴더를 `evaluation/score.py`로 채점합니다. 사용법은 `evaluation/README.md`를 확인하십시오.
4. test 제출. test 입력에 대한 예측 csv를 `submission.zip`으로 묶어 제출합니다. 형식은 챌린지 페이지 2.4, 2.5절에서 확인하십시오. test 정답은 공개하지 않으며 test 채점은 주최 측이 같은 코드로 수행합니다.

## 출처

- 데이터는 STARSS23의 개발용(dev) 녹음에서 만들었습니다. 원본의 이용 조건을 따릅니다. Shimada et al., *STARSS23: An audio-visual dataset of spatial recordings of real scenes with spatiotemporal annotations of sound events*, NeurIPS 2023 Datasets and Benchmarks.
- 베이스라인 코드는 DCASE 2025 Task 3 공식 베이스라인 [partha2409/DCASE2025_seld_baseline](https://github.com/partha2409/DCASE2025_seld_baseline)(커밋 `42a48b6456b73be35ad0e1a9ffeb6ceef83ae0bd`)을 복사하여 이 챌린지에 맞게 고친 것입니다. 평가 코드(`metrics.py`)도 같은 저장소의 것을 고쳤습니다.
- 스테레오 변환식은 [SonyResearch/dcase2025_stereo_seld_data_generator](https://github.com/SonyResearch/dcase2025_stereo_seld_data_generator)를 참고하여 카메라가 움직이도록 다시 구현했습니다.

