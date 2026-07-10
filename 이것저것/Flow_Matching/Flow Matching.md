# Flow Matching 핵심 정리

참고 논문:
- `Flow_Matching_22.pdf`: Flow Matching for Generative Modeling
- `Flow_Matching_24.pdf`: Scaling Rectified Flow Transformers for High-Resolution Image Synthesis

## Diffusion이 등장한 배경
- GAN은 선명한 이미지를 만들지만 학습 불안정, mode collapse 문제가 있음.
- VAE는 안정적이지만 결과가 흐릿한 경향이 있음.
- Normalizing Flow는 likelihood 계산은 좋지만 구조 제약과 학습 비용 문제가 있음.
- Diffusion은 복잡한 생성 문제를 **여러 단계의 denoising 문제**로 쪼개 안정적으로 학습함.

핵심 아이디어:

```text
clean data -> 점점 noise 추가 -> pure noise
pure noise -> 점점 denoise -> clean data 생성
```

## 기존 Diffusion 메커니즘
Forward process:

$$
x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon,
\quad \epsilon \sim \mathcal{N}(0, I)
$$

- $x_0$: clean data
- $x_t$: noise가 섞인 중간 상태
- $\epsilon$: 추가된 Gaussian noise
- $t$가 커질수록 noise에 가까워짐

학습:

$$
\mathcal{L}
=
\mathbb{E}
\left[
\|\epsilon_\theta(x_t,t,c)-\epsilon\|^2
\right]
$$

- 모델은 noisy sample $x_t$에서 실제로 섞인 noise $\epsilon$을 예측함.
- 생성 시에는 pure noise에서 시작해 noise를 여러 step에 걸쳐 제거함.

## Flow Matching이 나온 이유
- 기존 diffusion은 안정적이지만 sampling step이 많아 느릴 수 있음.
- Continuous Normalizing Flow(CNF)는 ODE로 noise distribution을 data distribution으로 직접 이동시킬 수 있음.

$$
\frac{dy_t}{dt}=v_\theta(y_t,t)
$$

- 하지만 기존 CNF는 학습 중 ODE simulation이 비싸서 scale이 어려웠음.
- Flow Matching은 **CNF처럼 velocity field를 배우되, diffusion처럼 simulation-free로 학습**할 수 있게 만든 방법.

## Flow Matching 핵심 개념
Flow Matching은 noise와 data 사이의 probability path를 정하고, 그 path를 따라 움직이는 **velocity**를 학습함.

가장 단순한 Rectified Flow 형태:

$$
x_t = (1-t)x_0 + t x_1
$$

- $x_0$: noise
- $x_1$: data
- $t=0$: noise
- $t=1$: data

이때 target velocity:

$$
v_t = \frac{dx_t}{dt}=x_1-x_0
$$

즉 모델은 다음을 학습함:

```text
현재 noisy latent x_t를 clean data 쪽으로 보내려면
어느 방향으로 움직여야 하는가?
```

학습 loss (MSE):

$$
\mathcal{L}
=
\mathbb{E}
\left[
\|v_\theta(x_t,t,c)-v_t\|^2
\right]
$$

## 학습 알고리즘
```text
1. real data x_1을 샘플링
2. noise x_0 ~ N(0, I)를 샘플링
3. timestep t를 샘플링
4. x_t = (1-t)x_0 + t x_1 생성
5. target velocity v_t = x_1 - x_0 계산
6. model이 v_t를 예측하도록 MSE 학습
```

## Conditional Flow Matching
- 전체 data distribution의 vector field는 직접 계산하기 어려움.
- 대신 각 data sample $x_1$에 대해 conditional path를 만들고 학습함.

```text
noise sample 하나와 data sample 하나를 연결하는 경로를 계속 샘플링
```

- 22년 논문은 이 conditional objective가 원래 Flow Matching objective와 같은 gradient를 준다고 보임.
- 덕분에 고차원 이미지에서도 scalable하게 학습 가능.

## Rectified Flow / OT Path
- 22년 논문은 diffusion path뿐 아니라 Optimal Transport(OT) path도 Flow Matching 안에서 다룸.
- OT / Rectified Flow path는 data와 noise를 거의 직선으로 연결함.

```text
Diffusion path: curved path, 여러 step 필요
Rectified / OT path: straight path, 더 적은 step에 유리
```

- 직선 경로는 모델이 배울 velocity가 단순하고 sampling error가 덜 누적될 수 있음.

## 22년 논문 vs 24년 논문
| 항목 | 22년 Flow Matching | 24년 Rectified Flow Transformer |
|---|---|---|
| 성격 | 이론/프레임워크 제안 | 대규모 text-to-image 적용 |
| 핵심 | Conditional Flow Matching, CNF, OT path | Rectified Flow + Transformer scaling |
| 모델 | 주로 U-Net 기반 이미지 실험 | MM-DiT 기반 text-to-image |
| 관심 | FM이 왜 가능한지, diffusion/OT path 비교 | 고해상도 생성에서 RF가 잘 scale되는지 |
| timestep | 기본 FM 설정 중심 | logit-normal 등 timestep sampling 개선 |

## 24년 논문의 핵심 포인트
- Rectified Flow를 고해상도 text-to-image 모델에 적용.
- uniform timestep sampling보다 중간 noise scale을 더 자주 학습하는 방식이 효과적이라고 봄.
- 대표적으로 **logit-normal timestep sampling**을 사용.
- Transformer에서 image token과 text token을 양방향으로 섞는 MM-DiT 구조를 제안.

## [[Wan]]과의 연결
Wan도 Flow Matching / Rectified Flow 계열을 사용함.

Wan의 학습 식:

$$
x_t = t x_1 + (1-t)x_0
$$

$$
v_t = x_1 - x_0
$$

- $x_0$: random noise
- $x_1$: clean video latent
- $x_t$: noise와 clean latent가 섞인 중간 latent
- DiT는 $x_t$, text condition, timestep $t$를 입력받아 velocity를 예측함.

즉 Wan의 DiT는:

```text
현재 noisy video latent를 clean video latent 쪽으로 이동시키는 방향
```

을 예측하도록 학습됨.

## 표기 주의
- 논문마다 $x_0$, $x_1$, $t$의 방향 표기가 다를 수 있음.
- 중요한 것은 항상 동일함:

```text
noise와 data 사이의 path를 만들고,
그 path 위에서 data 쪽으로 움직이는 velocity를 학습한다.
```
