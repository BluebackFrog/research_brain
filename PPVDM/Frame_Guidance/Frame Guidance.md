# Frame Guidance

## Method 핵심 정리

Frame Guidance는 pretrained VDM을 fine-tuning하지 않고, sampling 중인 video latent $z_t$를 frame-level condition에 맞게 직접 업데이트하는 training-free guidance 방법이다.

keyframe, style image, depth map, sketch 등은 모델 입력으로 직접 들어가지 않고, 현재 예측 frame과 비교되는 guidance loss의 target으로 사용된다. 이 loss의 gradient를 $z_t$까지 역전파해 latent를 수정한다.

전체 흐름:

- $z_t$에서 predicted clean latent $z_{0|t}$를 예측한다.
- guided frame index $I$에 해당하는 latent만 slicing해서 frame $x^I_{0|t}$를 decode한다.
- condition $c_{\text{frames}}$와 비교해 loss $\mathcal{L}_e$를 계산한다.
- $g_t = \nabla_{z_t}\mathcal{L}_e(x^I_{0|t}, c_{\text{frames}})$로 $z_t$를 업데이트한다.

여기서 $i$는 video frame index, $t$는 denoising timestep이다. loss는 선택된 frame $I$에서만 계산되지만, gradient는 denoising network를 통해 전체 video latent에 영향을 줄 수 있다.

### 4.1 Latent Slicing

CausalVAE는 한 frame만 필요해도 전체 temporal latent sequence를 decode하려 해 guidance memory가 크게 증가한다. 논문은 실제 latent 영향이 local하다는 점을 이용해, target frame 주변의 작은 latent window만 decode한다.

역할: guidance loss 계산용 frame만 decode해 GPU memory를 줄인다.

### 4.2 Video Latent Optimization (VLO)

VLO는 denoising 단계에 따라 $z_t$ 업데이트 방식을 다르게 쓰는 전략이다.

- 초기 단계: layout과 motion 방향이 결정되므로 deterministic update를 사용한다. $z_t \leftarrow z_t - \eta g_t$
- 후반 단계: artifact와 oversaturation을 줄이기 위해 noise를 섞는 stochastic update를 사용한다.

즉 초반에는 condition을 강하게 반영하고, 후반에는 자연스러움과 디테일을 보정한다.

### 4.3 Frame Guidance Algorithm

전체 알고리즘은 latent slicing으로 memory를 줄이고, VLO로 $z_t$를 업데이트하는 방식이다. ControlNet처럼 condition을 network에 주입하는 것이 아니라, sampling 중간 결과가 condition과 맞도록 latent를 gradient로 조정한다.

### 4.4 Loss Design

task별로 guidance loss만 바꾸면 여러 frame-level control에 적용할 수 있다.

- Keyframe: keyframe $x_i^*$와 generated frame $x^i_{0|t}$ 사이의 L2 loss.
- Style: differentiable style encoder $\Psi$의 feature similarity.
- Loop: 첫 frame과 마지막 frame 사이의 similarity loss.
- Depth / sketch: differentiable encoder feature space에서 condition과 비교.

여기서 differentiable encoder가 필요한 이유는 loss gradient가 generated frame을 거쳐 $z_t$까지 흘러가야 하기 때문이다.

### Keyframe의 의미

Keyframe은 사용자가 추가로 제공한 reference frame이다. latent를 직접 keyframe에 맞추는 것이 아니라, decoded frame이 keyframe과 비슷해지도록 loss를 걸고 그 gradient로 latent를 수정한다.
