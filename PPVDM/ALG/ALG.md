## 한 줄 정리

- I2V 모델이 reference image의 고주파 detail에 너무 일찍 고정되어 motion을 잃는 문제를, denoising 초기에만 image latent를 low-pass filtering하고 후반에는 원본 latent로 전환하는 training-free guidance로 완화한다.

## Motivation

- **큰 도메인:** reference image와 text prompt로부터 자연스럽고 동적인 영상을 생성하는 image-to-video generation.
- **기존 문제:** T2V 모델을 I2V로 fine-tuning하면 입력 image 보존이 지나치게 강해져, prompt가 큰 motion을 요구해도 거의 정적인 video가 생성된다.
- **해결 방식:** training-free adaptive low-pass guidance.

## Main Method

### 1. Suppressed motion의 원인 진단

![[ALG_overview.png]]

- 같은 Wan 2.1 backbone의 T2V 결과에서 첫 frame을 뽑아 I2V 입력으로 다시 사용하면, 다른 quality metric은 비슷하지만 Dynamic Degree가 $39.4\rightarrow32.1$로 18.6% 감소한다. 즉 motion 감소는 주로 image conditioning의 도입에서 발생한다.
- 논문의 가설은 reference image의 고주파 detail이 coarse-to-fine generation 초기에 너무 강하게 노출되어 **shortcut trajectory**를 만든다는 것이다.
	- Wan 2.1 DiT의 중간 feature를 PCA로 RGB 시각화하면, 기본 I2V는 50-step 중 첫 update인 $t=0.02$부터 input의 세부 구조를 재현한다.
	- 이렇게 appearance가 조기에 확정되면 이후 trajectory가 움직일 자유도가 줄어들어 static video로 수렴한다.
- Input image에 강한 low-pass filter를 적용할수록 Dynamic Degree는 단조 증가하지만, 흐려진 조건 때문에 aesthetic quality와 input fidelity가 감소한다.
- 따라서 필요한 것은 고주파 정보를 완전히 제거하는 것이 아니라, motion의 coarse structure가 정해지는 **초기 step에서만 잠시 감추는 것**이다.

### 2. Adaptive low-pass condition

- RGB conditioning image $w_{\mathrm{init}}$를 기존 VAE encoder로 latent $x_{\mathrm{init}}=E(w_{\mathrm{init}})$로 변환한다.
- Timestep $t$에서 denoiser에 제공할 condition을 다음처럼 정의한다.

$$
x_{\mathrm{init}}^{(t)}
=\mathcal{F}_{\mathrm{LP}}\!\left(x_{\mathrm{init}},\kappa(t)\right)
$$

- Main experiment의 $\mathcal{F}_{\mathrm{LP}}$는 latent를 bilinear downsampling한 뒤 원래 크기로 upsampling하는 단순 low-pass filter이다.
	- $\kappa(t)$는 downsampling factor이며, $\kappa=0$이면 filter를 적용하지 않은 원본 latent를 뜻한다.
	- Raw image가 아니라 denoiser가 실제로 받는 **VAE latent**를 filtering한다.
- 기본 schedule은 두 구간으로 나뉘는 step function이다.

$$
\kappa(t)=
\begin{cases}
\kappa_* & t<t_{\mathrm{trans}}\\
0 & t\ge t_{\mathrm{trans}}
\end{cases}
$$

	- **Early denoising:** $\kappa_*>0$인 저주파 latent만 보여 주어 fine detail shortcut을 막고 large motion과 coarse structure가 형성될 공간을 남긴다.
	- **Late denoising:** 원본 latent로 전환해 identity, texture, edge 같은 고주파 detail을 복원한다.
	- 기본값은 대체로 $t_{\mathrm{trans}}=0.1$, $\kappa_*=2.5$이며 backbone별로 조정한다.

### 3. ALG guidance

- 기존 I2V velocity model을 $v_\theta(x_t,x_{\mathrm{init}},t,c)$라고 할 때 ALG prediction은 다음과 같다.

$$
v_{\mathrm{ALG}}(x_t,t)
=v_\theta(x_t,x_{\mathrm{init}},t,\varnothing)
+w\left[
v_\theta(x_t,x_{\mathrm{init}}^{(t)},t,c)
-v_\theta(x_t,x_{\mathrm{init}}^{(t)},t,\varnothing)
\right]
$$

- **Motion-enhancing branch:** CFG difference의 conditional/unconditional prediction에는 같은 filtered latent $x_{\mathrm{init}}^{(t)}$를 넣어, 초기 trajectory가 세부 appearance에 고정되는 것을 막는다.
- **Fidelity anchor:** 첫 unconditional term에는 항상 원본 $x_{\mathrm{init}}$를 넣어, filtering 때문에 사라질 수 있는 identity와 고주파 정보를 유지한다.
	- 세 prediction 모두 filtered latent를 사용하면 distortion, spatial incoherence, 갑작스러운 scene transition이 자주 발생한다.
- 각 step에서 $v_{\mathrm{ALG}}$로 기존 ODE solver update를 수행할 뿐이며 model weight를 수정하지 않는다. $\kappa(t)=0$인 후반부에는 식이 원래 CFG와 같아진다.

### 4. Backbone별 condition 주입

- Video latent $x$는 일반적으로 $B\times F\times C\times W\times H$의 5D tensor이고, 한 frame짜리 $x_{\mathrm{init}}$는 temporal zero-padding으로 길이를 맞춘다.
- **Wan 2.1 / 2.2:** filtered image latent를 zero-pad한 뒤 noisy video latent와 channel-wise concatenate한다.
	- Wan 2.1의 CLIP image embedding은 fine detail보다 high-level semantics를 담으므로 원본 image를 그대로 사용한다.
	- Wan 2.2는 high-noise/low-noise용 two-stage denoiser와 explicit first-frame mask를 갖지만 ALG 적용 위치는 동일하다.
- **LTX-Video:** 매 step noisy video latent의 첫 frame을 conditioning latent로 치환하므로, 초기에는 filtered latent를 넣고 $t_{\mathrm{trans}}$ 이후 원본 latent로 교체한다.

### 5. 전체 sampling pipeline

- `(input image, text prompt)` $\rightarrow$ `VAE encoding` $\rightarrow$ `초기 step에서 latent low-pass filtering` $\rightarrow$ `ALG의 3개 denoiser prediction 결합` $\rightarrow$ `ODE solver update 반복` $\rightarrow$ `후기 step에서 원본 condition으로 detail 복원` $\rightarrow$ `VAE decoding` $\rightarrow$ `dynamic video`.
- 품질 안정화를 위해 Wan 계열에서는 다음 보조 기법을 사용한다.
	- Denoising 종료 후 final latent의 첫 frame을 clean input latent로 덮어써 첫 frame fidelity를 높인다.
	- 일부 설정에서는 처음 1-2 step을 clean latent로 진행한 뒤 filtered latent 노출을 시작한다. Filtering 기간 자체는 유지하고 시작점만 약간 늦춘다.
- Main setting은 Wan 2.2에서 $(t_{\mathrm{trans}},\kappa_*)=(0.1,2.5)$, Wan 2.1에서 $(0.2,2.5)$, LTX-Video에서 $(0.1,4.0)$이다.

## 실험

- **Benchmark**
	- **VBench-I2V:** `(reference image, prompt)`를 받아 5초 video를 생성하는 I2V 평가다. Camera motion/background 전용 항목을 제외한 246개 pair를 사용한다.
	- **PVD:** 실제 video-caption pair 100개에서 첫 frame을 reference image로 추출하고 expert caption을 prompt로 사용해, 실제 장면에서의 continuation을 평가한다.
	- **VidProM:** 무작위 prompt 750개를 선택하고 FLUX.1-dev로 reference image를 만든 뒤, synthetic input에서도 motion enhancement가 유지되는지 평가한다.
- **Metric**
	- **Dynamic Degree:** RAFT optical flow의 상위 5% magnitude로 frame interval을 dynamic/static으로 판정하고, dynamic interval 비율이 threshold를 넘는 video의 비율을 측정한다. 해상도 threshold와 FPS를 보정한다.
	- **VBench-QS:** subject consistency, temporal flicker, motion smoothness, aesthetic quality, imaging quality의 평균으로, motion 증가가 일반 video quality를 훼손하는지 확인한다.
	- **VBench-I2V:** input image와 각 generated frame 및 연속 frame 사이의 DINO similarity를 결합해 subject/input fidelity를 측정한다.
	- **DOVER / VisionReward:** 각각 human judgment에 맞춘 aesthetic+technical video quality와 VLM 기반 다차원 human preference를 평가한다.
- **핵심 결과**
	- VBench에서 Dynamic Degree는 Wan 2.2 $31.7\rightarrow39.0$, Wan 2.1 $28.9\rightarrow39.4$, LTX-Video $15.5\rightarrow21.5$로, 모델 평균 약 33% 향상된다.
	- 같은 실험에서 VBench-QS, VBench-I2V, DOVER, VisionReward는 거의 유지되고 VBench-Avg.는 세 모델 모두 상승해, motion gain이 단순한 품질 희생에서 나온 것이 아님을 보인다.
	- Wan 2.2 기준 PVD Dynamic Degree는 $65.0\rightarrow69.0$, VidProM은 $27.3\rightarrow30.5$로 올라 input source가 real/synthetic인지와 무관하게 개선된다.
	- 추가 계산 비용은 H200 한 장 기준 Wan 2.2 $475\rightarrow494$s, Wan 2.1 $476\rightarrow527$s, LTX-Video $58\rightarrow59$s로 최대 약 11%다.

## Ablation / Analysis

- **Transition time $t_{\mathrm{trans}}$:** $0.06$만 되어도 Dynamic Degree가 32% 증가해, motion을 결정하는 매우 초기 구간에서의 filtering이 핵심임을 보여 준다. 너무 길게 유지하면 input fidelity가 감소한다.
- **Filter strength $\kappa_*$:** 강도를 높일수록 motion gain은 커지지만 점차 포화되고 quality가 조금 감소한다. $\kappa_*=1.6$에서는 Dynamic Degree가 29% 증가하면서 VBench-QS는 0.5%만 감소한다.
- **Filter type:** Gaussian blur도 motion을 늘려 frequency-removal 가설을 지지하지만, downsampling+upsampling보다 고주파 제거가 약해 gain이 작다.
- **Motion-augmented prompt:** Gemini 2.5로 prompt의 motion 표현을 강화하면 CFG와 ALG가 모두 좋아지고, ALG+augmentation이 가장 높다. ALG 단독도 augmented CFG보다 Dynamic Degree가 높아 prompt engineering과 상호 보완적이다.
- **Fidelity correction design:** Original latent를 쓰는 unconditional anchor를 제거하고 모든 term에 filtered latent를 넣으면 distortion과 scene instability가 증가해, motion branch와 fidelity anchor의 분리가 필요하다.
- **Shortcut visualization:** Default I2V는 첫 denoising update부터 input detail이 feature map에 나타나지만, low-pass condition에서는 detail이 점진적으로 형성되며 최종 motion이 커진다.
- **실질적 한계:** $t_{\mathrm{trans}}$와 $\kappa_*$를 backbone/resolution별로 선택해야 하고, 과도한 filtering은 fidelity를 떨어뜨린다. 초기 step에서 denoiser forward가 하나 더 필요해 완전히 비용 없는 guidance는 아니다.
