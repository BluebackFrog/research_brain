## Method 간략 정리

### 핵심 아이디어

- MotiMotion은 **I2V motion control** 모델이다. 즉, 텍스트만으로 비디오를 만드는 것이 아니라 **첫 프레임 이미지 + 유저가 그린 trajectory + optional prompt**를 입력으로 받는다.
- 기존 motion control은 trajectory를 그대로 따르는 **literal executioner**처럼 작동한다.
- MotiMotion은 trajectory를 정답이 아니라 **유저의 대략적인 의도**로 보고, VLM이 물리/상식적으로 자연스러운 움직임을 보완한다.

```text
Reference image
+ User-drawn trajectory
+ Optional prompt
        |
        v
VLM reasoning
        |
        v
Refined primary trajectory + secondary trajectory + detailed prompt
        |
        v
Motion-controlled video generator
        |
        v
Physically plausible video
```

### Motion Representation

$$
T \in \mathbb{R}^{N \times L \times 2}
$$

- `N`: trajectory point 개수
- `L`: 비디오 프레임 수
- `T(n, l)`: `n`번째 point가 `l`번째 프레임에서 가져야 할 `(x, y)` 좌표
- 각 좌표는 한 픽셀로 찍는 것이 아니라 **Gaussian heatmap**으로 변환한다.

$$
M \in \mathbb{R}^{L \times H \times W \times 3}
$$

- `M`: motion volume
- 실제 RGB 영상이 아니라, 각 프레임에 “motion signal이 어디 있는지” 표시한 **heatmap video**에 가깝다.

### Motion Condition

$$
M \rightarrow \text{VAE Encoder} \rightarrow z_m
$$

- DiT에 들어가기 전, 다음 latent들을 channel dimension으로 concat한다.

$$
[z_t,\ z_{\text{ref}},\ z_m]
$$

- `z_t`: 현재 생성 중인 noisy video latent
- `z_ref`: 첫 프레임의 장면/외형 정보를 담은 reference image latent
- `z_m`: trajectory 정보를 담은 motion latent
- `z_m`은 DiT 중간에 주입되는 것이 아니라, **DiT 입력 전에 channel-wise concat**된다.
- concat으로 입력 채널 수가 늘어나므로 DiT의 **initial tokenization layer**를 확장한다.
- LoRA처럼 기존 모델을 최대한 보존하려는 성격은 있지만, 실제 구현은 **input channel expansion + 일부 fine-tuning**에 가깝다.

### Confidence-Aware Control

- 각 trajectory에는 confidence score가 붙는다.

$$
s \in [0, 1], \quad
G' = sG
$$

- `s`가 높으면 trajectory를 강하게 따른다.
- `s`가 낮으면 trajectory를 rough guidance로 보고 video prior에 더 의존한다.
- 학습 중 trajectory를 일부러 perturb/smoothing해서, 부정확한 입력에도 자연스럽게 보정하도록 만든다.

> 한 줄 요약: MotiMotion은 유저가 그린 sparse trajectory를 VLM reasoning과 confidence-aware control로 보완하여, 물리적으로 더 그럴듯한 I2V 영상을 생성하는 방법이다.
