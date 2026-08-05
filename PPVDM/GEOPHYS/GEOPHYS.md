## 한 줄 정리

- Frozen image encoder가 각 video frame에서 만든 feature trajectory의 속도·곡률·가속도·선형 예측 오차를 측정해, 별도 학습 없이 물리 위반을 검출하고 best-of-$N$ video generation의 verifier로 사용하는 방법이다.

## Motivation

- **큰 도메인:** video의 physical plausibility 평가와 physics-aligned video generation.
- **기존 문제:** 기존 verifier는 MLLM judge, video-pretrained world model, physics-specific training 또는 simulator에 의존해 느리고 계산량이 크다.
- **해결 방식:** frozen image encoder의 temporal feature geometry를 이용한 training-free scoring.

## Main Method

### 1. 핵심 가설

- 자연 영상에서 일어나는 정상적인 변화는 image encoder의 feature space에서 **부드럽고 국소적으로 선형인 trajectory**로 나타난다.
	- 가능한 영상: frame 간 feature 이동의 크기와 방향이 비교적 일정하며 다음 feature를 이전 feature들로 예측하기 쉽다.
	- 물리 위반 영상: 물체의 순간 이동·소멸·비정상 가속처럼 trajectory의 step size, 방향 또는 local subspace가 갑자기 깨진다.
- 이는 backbone이 물리 법칙을 명시적으로 표현하거나 simulation한다는 주장이 아니라, 자연 영상 통계로부터 학습된 feature geometry와 물리적 개연성 사이의 **통계적 상관관계**를 이용하는 것이다.

### 2. Frame에서 feature trajectory로 변환

![[attachments/GEOPHYS_pipeline.png]]

- 입력 video는 $V=(x_1,\ldots,x_T)$이고, backbone $f_\theta$의 weight는 고정하며 video training이나 fine-tuning을 수행하지 않는다.
- 각 frame에서 선택한 layer의 spatial token을 추출한다.

$$
Z_t=f_\theta^{(\ell^*)}(x_t)=\{z_{t,n}\}_{n=1}^{N}\in\mathbb{R}^{N\times d}
$$

- Spatial average pooling으로 frame 전체를 하나의 vector로 만든다.

$$
\bar z_t=\frac{1}{N}\sum_{n=1}^{N}z_{t,n}\in\mathbb{R}^{d}
$$

	- DINOv2/v3 ViT-L의 $d=1024$, CORnet-S와 VOneNet의 $d=512$이다.
	- 시간순으로 쌓은 $\Gamma(V)=(\bar z_1,\ldots,\bar z_T)$가 $d$차원 feature space의 discrete trajectory가 된다.
- Readout layer $\ell^*$는 held-out validation split에서 plausible/violated video의 평균 curvature 차이가 최대가 되는 layer로 정한다.
	- LikePhys와 generation에 쓰는 대표 조합은 DINOv2 L12, DINOv3 L18, CORnet-S IT, VOneNet V1이다.
	- IntPhys2의 국소적인 소멸·관통 위반은 더 낮은 level의 feature가 유리해 DINOv3 L12와 CORnet-S V1이 선택된다.

### 3. 다섯 geometric signal

- Frame 간 displacement와 speed를 다음처럼 정의한다.

$$
v_t=\bar z_{t+1}-\bar z_t,\qquad s_t=\lVert v_t\rVert_2
$$

- **Speed variation:** 갑작스러운 이동량 변화를 $\phi_{\mathrm{speed}}(V)=\operatorname{std}(\{s_t\})$로 측정한다.
	- 물체의 순간 이동·소멸 또는 일관되지 않은 움직임에서 커진다.
- **Curvature:** 연속 displacement 사이의 turning angle을 계산하고 그 평균을 사용한다.

$$
\theta_t=\arccos\frac{\langle v_t,v_{t+1}\rangle}{\lVert v_t\rVert_2\lVert v_{t+1}\rVert_2},\qquad
\phi_{\mathrm{curv}}(V)=\frac{1}{T-2}\sum_{t=1}^{T-2}\theta_t
$$

	- 국소적으로 직선에 가까운 trajectory는 작고, wall-pass처럼 예상 trajectory에서 급격히 벗어나면 커진다.
- **Angle consistency:** turning angle의 시간적 변동 $\phi_{\mathrm{ang}}(V)=\operatorname{std}(\{\theta_t\})$를 측정해, 방향 변화가 얼마나 불규칙한지 본다.
- **Acceleration:** displacement의 2차 차분을 계산한다.

$$
a_t=v_{t+1}-v_t=\bar z_{t+2}-2\bar z_{t+1}+\bar z_t,\qquad
\phi_{\mathrm{accel}}(V)=\frac{1}{T-2}\sum_{t=1}^{T-2}\lVert a_t\rVert_2^2
$$

	- Curvature가 방향 변화만 분리한다면 acceleration은 방향과 step magnitude의 급변을 함께 포착한다.
- **Local linear prediction error:** 직전 $H$개 feature를 concatenate해 linear autoregressive predictor로 다음 feature를 예측한다.

$$
\hat{\bar z}_{t+1}=\hat P_H[\bar z_{t-H+1};\ldots;\bar z_t],\qquad
\epsilon_t=\bar z_{t+1}-\hat{\bar z}_{t+1}
$$

$$
\phi_{\mathrm{perr}}(V)=\frac{1}{T-H}\sum_{t=H}^{T-1}\lVert\epsilon_t\rVert_2
$$

	- 다음 feature가 과거 $H$-step의 local linear span에서 벗어날수록 커지며, 비정상적인 fluid reversal이나 spontaneous acceleration을 포착한다.
- 최종 temporal descriptor는 다음 다섯 scalar로 구성되며, 모든 값은 클수록 trajectory가 불규칙하고 물리적으로 덜 그럴듯하다는 뜻이다.

$$
\Phi_{\mathrm{temp}}(V)=
(\phi_{\mathrm{curv}},\phi_{\mathrm{ang}},\phi_{\mathrm{speed}},\phi_{\mathrm{accel}},\phi_{\mathrm{perr}})
$$

### 4. Backbone별 scalar score와 ensemble

- Held-out LikePhys split에서 backbone마다 가장 잘 작동하는 signal 하나를 고정한다.
	- DINOv2 L12 / DINOv3 L18: angle consistency.
	- CORnet-S IT: speed variation.
	- VOneNet V1: acceleration.
- Backbone $b$의 선택된 signal을 표준화한 scalar $z_b$를 사용하며, 두 counterfactual video 중 $z_b$가 큰 쪽을 violated로 판정한다.
- Backbone이 서로 다른 위반 유형에 강하므로 두 ensemble을 사용한다.
	- **Majority:** 네 backbone 중 3개 이상이 동의한 판정을 선택한다.
	- **OR:** $b^*=\arg\max_b|z_b|$로 가장 확신이 큰 backbone의 판정을 사용한다.
- Best-of-$N$ generation에서는 같은 score와 OR 규칙으로 $N$개 완성 후보를 ranking하고 가장 plausible한 후보를 고른다. PhysicsIQ에 맞춘 reward model 학습이나 signal 재탐색은 없다.

### 5. 전체 pipeline

- `video frames` $\rightarrow$ `frozen backbone의 layer feature` $\rightarrow$ `spatial average pooling` $\rightarrow$ `$d$차원 temporal trajectory` $\rightarrow$ `다섯 geometric statistic` $\rightarrow$ `backbone별 standardized implausibility score` $\rightarrow$ `single/majority/OR decision 또는 best-of-$N$ selection`.

## 실험

### 1. Object-permanence와 human EEG alignment

- **Benchmark:** object가 가림막 뒤에서 하나 늘어나는 Create와 하나 사라지는 Vanish의 valid/invalid paired video를 입력한다. Model의 frame-wise signal이 위반 직후 사람의 visual working-memory EEG 지표인 CDA와 같은 방향·시간대에 변하는지 본다.
- **Metric:** occlusion 전 차이를 0으로 보정한 뒤 post-occlusion의 valid$-$invalid signal과 paired $t$-test를 측정하고, 추가 VOE dataset에서는 pairwise detection accuracy를 사용한다.
- **핵심 결과:** CORnet-S IT의 speed·acceleration·prediction error가 Create와 Vanish 모두에서 object 수 변화에 맞춰 반응했으며, Create $p=0.0041$, Vanish $p=0.0264$로 유의했다. 다만 object가 완전히 가려진 동안에는 CDA가 유지되는 것과 달리 score가 거의 0이 되어, abstract object tracking 자체를 구현하는 것은 아님을 보여 준다.

### 2. Physics violation detection

- **LikePhys:** Blender로 만든 650개 plausible/violated matched pair와 12개 scenario를 사용한다. 두 video는 appearance가 같고 gravity reversal, penetration, energy non-conservation, temporal disorder 등 하나의 물리 위반만 다르며, 모델은 어느 쪽이 violated인지 골라야 한다.
- **IntPhys2:** Unreal Engine의 506개 photorealistic pair에서 permanence, solidity, continuity, immutability를 평가한다. 같은 initial condition을 공유하는 possible/impossible video와 fixed/moving camera가 주어지며 violated video를 선택한다.
- **Metric:** pairwise accuracy는 각 matched pair 안에서 올바른 video를 violated로 ranking한 비율이다. ROC-AUC는 pair를 넘어서 하나의 global threshold로 implausibility score를 구분할 수 있는지도 측정한다.
- **핵심 결과:** OR ensemble은 LikePhys $98.3\%$, IntPhys2 $93.3\%$로 각각 기존 diffusion model, GPT-4o/Gemini, V-JEPA 2 등을 크게 앞섰다. IntPhys2에서는 human $96.4\%$와 3.1%p 차이였으며, ROC-AUC도 LikePhys $90\%$, IntPhys2 $82\%$로 pair 내부 shortcut만 사용한 결과가 아님을 확인했다.

### 3. PhysicsIQ best-of-$N$ generation

- **Benchmark:** 66개 real-world physical scenario를 solid mechanics, fluid, optics, thermodynamics, magnetism의 5개 category로 촬영하고 3개 view를 사용해 198개 평가 scenario를 구성한다. 8초 video를 3초 conditioning과 5초 ground-truth continuation으로 나눈다.
	- I2V는 사건 직전의 마지막 frame, V2V는 전체 3초 conditioning clip을 입력받아 5초 continuation을 생성한다.
	- 각 generator가 scenario마다 $N=16$개 후보를 생성하면 verifier가 하나를 선택한다.
- **Metric:** generated/real continuation에서 frame 변화량을 thresholding해 motion mask를 만든 뒤 네 값을 합쳐 PhysicsIQ score를 계산한다.
	- **Spatial IoU:** 시간축을 max로 접어, motion이 발생한 공간 위치가 ground truth와 겹치는지 측정한다.
	- **Spatiotemporal IoU:** frame별 motion mask IoU를 평균해 motion의 위치와 timing을 함께 평가한다.
	- **Weighted Spatial IoU:** pixel별 motion 양까지 반영해 반복 motion과 한 번 지나가는 motion을 구분한다.
	- **MSE:** generated frame과 real frame의 pixel-level appearance 차이이며 낮을수록 좋다.
	- 세 IoU를 더하고 MSE 방향을 뒤집어 합친 뒤, 동일 조건의 실제 두 번째 촬영이 $100\%$가 되도록 정규화한다.
- **핵심 결과:** MAGI-1 24B V2V에서 no-verifier $50.01\%\rightarrow64.50\%$로 상승해 WMReward의 $62.29\%$를 넘었고 oracle $72.9\%$와의 gap을 가장 많이 줄였다. I2V의 MAGI-1 4.5B, Wan2.1 14B, CogVideoX-5B, MAGI-1 24B에서도 모든 비교 verifier보다 높았다.
- **비용:** 4-backbone OR ensemble은 H100에서 video당 1.0초, 2.0 GB VRAM으로, V-JEPA 2 기반 WMReward의 1.5초, 9.3 GB보다 wall-clock은 $1.5\times$, memory는 $4.65\times$ 적게 사용한다.
- **보조 품질 평가:** PhysicsIQ가 높아질수록 FVD와 LPIPS도 개선됐지만 VBench-Q는 거의 변하지 않았다. 따라서 gain은 단순히 일반적인 visual quality가 높은 후보를 고른 결과라기보다 physics fidelity와 과도한 spurious motion 억제에 집중된다.

## Ablation / Analysis

- **Perceptual straightening:** 4개 backbone의 총 57개 layer 조합 모두에서 violated video의 turning angle이 plausible video보다 컸고 $p<10^{-8}$이었다. 물리 위반이 특정 layer 하나의 우연한 artifact가 아니라 전반적인 trajectory bending으로 나타난다.
- **Signal별 역할:** LikePhys에서 DINOv2/v3는 angle consistency $79/81\%$, CORnet-S는 speed variation $78\%$, VOneNet은 acceleration $78\%$가 최고였다. Prediction error는 모든 backbone에서 chance 이상이지만 단독 최고 signal은 아니었다.
- **Layer sweep:** LikePhys는 ViT의 deep-but-not-final layer와 CORnet-S IT가 유리하지만, IntPhys2의 국소적인 object disappearance·wall penetration은 mid-level/V1-like feature가 더 잘 포착했다. 즉 최적 layer는 violation의 spatial/semantic scale에 따라 달라진다.
- **Backbone complementarity:** LikePhys의 rigid/optical은 VOneNet, fluid는 DINOv3, continuum은 DINOv2/CORnet-S가 강했다. 이 낮은 정답 집합 overlap 덕분에 OR ensemble이 single backbone의 약 $78$-$81\%$에서 $98.3\%$까지 상승했다.
- **Difficulty/camera analysis:** IntPhys2 OR은 Easy/Medium/Hard에서 $85.3/92.0/89.9\%$, fixed/moving camera에서 $91.3/91.1\%$로 안정적이었다. Majority는 fixed camera가 조금 유리했으며 retinotopic CORnet-S가 spatial stability의 도움을 받았다.
- **Best-of-$N$ scaling:** $N$이 커질수록 Qwen verifier는 빠르게 plateau했지만 이 score는 oracle과 가장 비슷하게 계속 상승했다. 개별 backbone도 baseline보다 대체로 5-11%p 개선하고 OR이 추가로 약 4-5%p를 얻어, detection에서 관찰한 complementarity가 generation에도 이어졌다.
- **핵심 한계:** 현재 signal은 시간 대칭적이라 video를 역재생해도 거의 같은 trajectory statistic을 얻으며, causality나 genuine physics simulation을 검증하지 못한다. 또한 best-of-$N$은 후보 pool에 plausible continuation이 있을 때만 효과가 있어 generator의 candidate diversity와 oracle ceiling에 제한된다.
