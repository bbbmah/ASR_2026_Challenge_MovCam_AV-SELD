# 평가 (Evaluation)

예측 csv 폴더를 정답 csv 폴더로 채점하는 코드입니다. val 채점과 test 채점(주최 측)에 같은 코드를 씁니다. 지표의 정의와 순위 규칙은 챌린지 페이지 4절에 있고, 여기서는 사용법과 출력을 설명합니다.

| 파일 | 내용 |
|---|---|
| `score.py` | 명령줄 채점 스크립트 |
| `metrics.py` | 지표 계산 |
| `seld_label_utils.py` | csv 읽기, 방위각 차이, 짝짓기(헝가리안) |

`baseline/seld_challenge/`에도 같은 세 파일이 있습니다(`run.py`가 같은 폴더에서 불러 씁니다). 두 곳의 내용은 같습니다.

## 준비

```bash
pip install -r requirements.txt   # numpy, scipy
```

## 입력

- 예측: 클립마다 csv 하나. 파일 이름은 정답 라벨과 같고(`val_00000.csv`, test는 `test_00000.csv`), 열은 `frame,class,source,azimuth,distance,onscreen`입니다. 열의 뜻과 제출 형식은 챌린지 페이지 2.3, 2.4절을 확인하십시오.
- 정답: csv가 바로 들어 있는 폴더. val은 `val.zip`을 풀면 나오는 `val/labels`입니다.
- 예측 csv가 없는 클립은 아무것도 예측하지 않은 것으로 채점합니다.
- frame이 0 ~ (`--clip_len` − 1) 밖인 행은 무시합니다.

## 실행

```bash
python score.py --pred 예측_csv_폴더 --ref val/labels --clip_len 200 \
                --modality audio_visual --out scores.json
```

| 옵션 | 뜻 |
|---|---|
| `--pred` | 예측 csv 폴더 (필수) |
| `--ref` | 정답 csv 폴더 (필수) |
| `--clip_len` | 클립의 라벨 프레임 수 (필수). main20은 200 |
| `--modality` | `audio_visual`(기본) 또는 `audio`. 소리만 쓰는 모델은 `audio`로 두며 onscreen 정확도는 `NA`가 된다 |
| `--exclude_list` | 채점에서 뺄 클립 이름 목록 파일(한 줄에 하나) |
| `--keep_absent_classes` | 정답이 없는 클래스도 평균 F에 넣는다(기본은 뺀다) |
| `--out` | 결과를 json으로 저장할 경로 |

```
clips=123 [ext] F=..% DOA=.. RelDist=.. Onscreen=..% [basic] F=..% DOA=.. RelDist=.. Onscreen=..%
```

파이썬에서 부를 때는 `from score import score_dirs`를 사용하십시오.

## 출력 (json)

최상위에 `ext`, `basic`, `n_clips`, `n_missing_pred`가 있습니다. `n_missing_pred`는 예측 csv가 없는 클립 수입니다.

- `ext`: 방위각을 −180°~180° 그대로 두고 원형 차이로 비교한 점수. 순위에는 이 점수를 사용합니다.
- `basic`: 정답과 예측의 방위각을 모두 −90°~90°(앞쪽)로 옮겨 비교한 점수.

`ext`와 `basic` 안에는 같은 항목이 있습니다.

| json 키 | 챌린지 페이지의 이름 | 뜻 |
|---|---|---|
| `F` | F<sub>ext</sub>, F<sub>basic</sub> | 위치까지 맞힌 검출 점수 (0~1) |
| `DOA_err` | DOAE | 짝지어진 검출의 평균 방위각 오차 (도) |
| `rel_dist_err` | RDE | 짝지어진 검출의 평균 상대 거리 오차 (0.35는 35%) |
| `onscreen_acc` | OSA | 짝지어진 검출 중 onscreen을 맞힌 비율 (0~1). 소리만 쓰면 `null` |
| `dist_err` | | 짝지어진 검출의 평균 거리 오차. 라벨과 같은 단위(cm) |
| `classwise` | | 위 다섯 값의 클래스별 결과(클래스 0~12 순서). 값이 없는 클래스는 `null` |

## 채점 방식 요약

- 0.1초 프레임마다, 클래스마다 정답과 예측을 방위각 차이를 기준으로 짝짓습니다(헝가리안 매칭). 방위각이 아닌 `source` 번호는 짝짓기에 쓰지 않습니다. 같은 프레임, 같은 클래스에서 최대 10개까지 읽습니다.
- 짝이 같은 클래스이고 방위각 오차가 20° 이하이며 상대 거리 오차 `|d_pred − d_ref| / d_ref`가 1.0 이하이면 True Positive입니다. 절대 거리 기준은 쓰지 않고, onscreen 일치도 TP 조건에 넣지 않습니다.
- 조건을 만족하지 못한 짝은 FP<sub>spatial</sub>, 예측이 정답보다 많은 초과분은 FP, 적은 부족분은 FN입니다.
- 클래스별로 `F_c = TP / (TP + FP_spatial + 0.5 × (FP + FN))`를 구하고, 정답이 하나라도 있는 클래스의 평균을 `F`로 합니다.
- 모든 프레임 수는 `--clip_len`으로 고정합니다. 마지막 소리 이후의 오검출이나 정답이 없는 클립에서의 오검출도 FP로 셉니다.
- 점수는 split 전체 클립을 한꺼번에 계산한 값이며, 클립별 점수의 평균이 아닙니다.

## test 채점

test 정답은 공개하지 않습니다. 참가 팀이 제출한 csv는 주최 측이 같은 코드로 채점합니다.
