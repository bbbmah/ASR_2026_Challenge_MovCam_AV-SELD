# 데이터셋

STARSS23 개발용(dev) 녹음(FOA 음향 + 360도 영상)으로 만든, 스테레오 음향 + 투시 영상 데이터입니다. 한 클립은 20초이고 그 동안 카메라가 회전합니다.

## 구성

| split | 클립 수 | 시간 | 용량 | 정답 |
|---|---|---|---|---|
| train | 2,999 | 16.66 h | 10.67 GB | 공개 |
| val | 123 | 0.68 h | 0.43 GB | 공개 |
| test | 526 | 2.92 h | 1.88 GB | 비공개 (입력만 공개) |

- 녹화가 이루어지는 방은 split끼리 섞이지 않습니다. (train, val, test가 모두 다른 방에서의 녹화 파일을 사용합니다.)
- train 클립은 같은 녹음에서 겹치게 뽑았습니다. val과 test는 녹음마다 겹치지 않는 창을 앞에서부터 잘라 사용했습니다.
- 라벨된 프레임이 30개(3초) 미만인 창과 음향 피크가 0.9999를 넘는 창은 제외했습니다.

## 내려받기

구글 드라이브에 아래 파일이 있습니다.
링크: (추가 예정)

| 파일 | 내용 |
|---|---|
| `train.zip`| train 영상, 음향, 라벨, meta |
| `val.zip` | val 영상, 음향, 라벨, meta |
| `test_inputs.zip` | test 영상, 음향 (정답 없음) |


## 폴더 구조

```
main20/
  train/  video/*.mp4  audio/*.wav  labels/*.csv  meta/*.csv   # 공개
  val/    (train과 같은 구조)                                    # 공개
  test/   video/*.mp4  audio/*.wav                              # 입력만 공개
```

- 클립 이름은 `{split}_{번호 5자리}`(예: `train_00012`)이며, video, audio, labels, meta가 같은 이름으로 대응합니다.
- `train.zip`과 `val.zip`의 맨 위에는 `manifest_public.csv`와 `split_info.json`이 있습니다. `test_inputs.zip`의 맨 위에는 클립 목록 `test_manifest_inputs.csv`(열: `clip, split, n_frames`)가 있습니다.

## 형식

- 음향: 스테레오(L, R) wav, 24 kHz, 16 bit, 20초. FOA(W, Y, Z, X)에서 `newY = sinθ·X + cosθ·Y`, `L = W + newY`, `R = W − newY`로 만들고, 카메라 방향 ψ(t)에 대해 θ = −ψ(t)를 샘플마다 바꿔 사용합니다.
- 영상: mp4(OpenCV `mp4v`), 640×360, 10 fps, 200프레임. 수평 시야 100°, 수직 시야 약 67.7°.
- 프레임 시각 규칙: 프레임 k(0~199)의 기준 시각은 클립 시작 후 `0.1k + 0.05`초(그 100 ms 구간의 가운데)입니다. 영상 프레임 k, 라벨 프레임 k, 카메라 자세가 같은 시각을 사용합니다.
- 라벨 `labels/{clip}.csv`: 헤더가 있는 csv, 열은 `frame,class,source,azimuth,distance,onscreen`.
  - `frame`: 0~199. 한 프레임에 여러 행이 있을 수 있고, 소리가 없는 프레임에는 행이 없습니다.
  - `class`: 0 Female speech, 1 Male speech, 2 Clapping, 3 Telephone, 4 Laughter, 5 Domestic sounds, 6 Walk/footsteps, 7 Door open/close, 8 Music, 9 Musical instrument, 10 Water tap, 11 Bell, 12 Knock.
  - `source`: 소리를 내는 사람의 번호(원 녹음 기준). 0은 사람이 아닌 소리입니다.
  - `azimuth`: 카메라 기준 방위각(도, 정수). 정면이 0이고 반시계 방향(왼쪽)이 양수이며 범위는 −180~180입니다. 같은 소리라도 카메라가 돌면 값이 바뀝니다.
  - `distance`: 소리까지의 거리(cm, 정수).
  - `onscreen`: 소리 방향이 그 프레임의 카메라 시야 안이면 1, 밖이면 0.
- 고도(elevation)는 공개 라벨에 없습니다.

## 좌표 규약과 onscreen

- 세계 방향 벡터는 `(cos el·cos az, cos el·sin az, sin el)`이고 x가 앞, y가 왼쪽, z가 위 입니다.
- 카메라는 세계 방위각 ψ, 고도 φ를 봅니다(φ < 0이 아래). 라벨의 방위각은 `wrap(az_world − ψ)`입니다.
- 시야 안 조건: 방향 벡터를 카메라 좌표 (x, y, z)로 돌렸을 때 `x > 0`, `|y| ≤ x·tan 50°`, `|z| ≤ x·tan 33.84°`이면 `onscreen = 1`입니다.
- 카메라의 움직임은 소리의 위치와 무관하게 결정했습니다.

## meta (train, val에만 있음)

분석용이며 추론 입력으로 쓸 수 없습니다(챌린지 규칙 5.2절).

- `meta/{clip}_ext.csv`: 공개 라벨의 열에 `az_world, el_world, cam_yaw, cam_pitch`를 더한 것입니다. 세계 기준 방위각과 고도, 프레임별 카메라 자세입니다.
- `meta/{clip}_traj.csv`: 프레임별 카메라 자세 `psi, phi`.
