# Wan 핵심 정리

## Overall
- Wan은 **이미지 생성과 비디오 생성 모두 지원**하지만, 핵심 타깃은 비디오 생성.
- 생성 core는 대략 다음 구조:

```text
Text / Image condition
 -> text encoder(umT5) / condition encoder
 -> DiT
 -> Wan-VAE Decoder
 -> image or video
```

- 4.5의 LLM prompt rewriting은 Wan 내부에 통합된 LLM이 아니라, **user prompt를 training caption 스타일로 바꾸는 전처리 단계**.

## Wan-VAE
![[Pasted image 20260701174631.png]]
- 2D image VAE를 먼저 학습한 뒤 3D causal VAE로 확장해 spatial compression prior를 활용함.
- 입력 video를 latent space로 압축:

$$
V \in \mathbb{R}^{(1+T)\times H\times W\times 3}
\xrightarrow{\text{Wan-VAE}}
z \in \mathbb{R}^{(1+\frac{T}{4})\times \frac{H}{8}\times \frac{W}{8}\times C},
\quad C=16
$$

- 첫 1 frame은 temporal compression 없이 spatial compression만 적용하고, 나머지 $T$ frames는 $4\times$ temporal compression 적용.

### Feature Cache
- Feature cache는 **DiT가 아니라 Wan-VAE 내부 causal Conv3D**에 적용됨.
- 긴 비디오를 chunk 단위로 encode/decode하면서, 이전 chunk의 마지막 frame-level feature를 다음 chunk에 넘겨 temporal continuity를 유지함.
- 기본 causal conv는 kernel size가 3이므로 이전 chunk의 마지막 **2개 feature**를 cache로 유지.

## Wan Architecture
![[Pasted image 20260702132257.png]]
- Text-to-video든 image-to-video든, 비디오 생성 모드에서는 내부적으로 **time 축을 가진 video latent 전체**를 생성함.
- 영상 길이는 모델이 자동으로 정하는 것이 아니라 `num_frames`, `duration`, `fps` 같은 설정으로 미리 정함.
- 예: 81 frames라면

$$
1+T=81,\quad 1+\frac{T}{4}=21
$$

- 즉 DiT는 21개의 latent time step을 가진 video latent를 처리.

## Diffusion Transformer
![[Pasted image 20260702132912.png]]
- Patchifying module은 VAE latent를 transformer가 처리할 수 있는 **V-token sequence**로 변환함.
- Unpatchifying module은 DiT 출력 token sequence를 다시 latent grid 형태로 되돌림.
- V-token은 video latent에서 온 token, T-token은 text encoder(umT5)에서 온 text token.
- T-token은 V-token을 쪼갠 것이 아니라 별도 text input에서 생성되며, cross-attention으로 V-token에 조건을 주입함.

Sequence length:

$$
L=
\left(1+\frac{T}{4}\right)
\times
\frac{H}{16}
\times
\frac{W}{16}
$$

- $H/16, W/16$은 Wan-VAE의 $8\times$ spatial compression 이후 patchifying에서 다시 $2\times2$로 묶기 때문.
- 따라서 max context length는 **frame 수와 해상도를 함께 고려**해야 함.

## Flow Matching in Wan
- Wan은 diffusion/flow matching 계열로 학습됨.
- Clean video latent $x_1$, random noise $x_0$에 대해:

$$
x_t = t x_1 + (1-t)x_0
$$

$$
v_t = x_1 - x_0
$$

- DiT는 $x_t$, text condition, timestep $t$를 입력받아 velocity $v_t$를 예측함.
- Velocity는 **각 Transformer block마다 나오는 것이 아니라**, N개의 DiT block을 모두 통과한 뒤 timestep마다 한 번 예측됨.

```text
noisy video latent x_t
 -> patchifying
 -> N x DiT blocks
 -> unpatchifying
 -> velocity prediction
```

- Sampling step이 30번이면 DiT 전체 forward가 30번 수행되고, velocity도 30번 예측됨.
- 비디오 frame들은 순차적으로 한 장씩 생성되는 것이 아니라, **전체 video latent의 모든 time/spatial token이 병렬적으로 함께 denoise**됨.
