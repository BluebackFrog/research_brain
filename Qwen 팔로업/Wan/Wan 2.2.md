https://github.com/Wan-Video/Wan2.2

![[Pasted image 20260720150044.png]]
![[Pasted image 20260720150101.png]]
![[Pasted image 20260720150113.png]]

## 한 줄 정리

- Wan2.1을 기반으로, denoising **step**별 두 expert를 교체하는 MoE와 고압축 VAE를 도입해 품질과 효율을 함께 높인 모델.

## Step-wise MoE

- 기존 token-level MoE와 달리, **diffusion step의 noise level**에 따라 DiT 전체 expert 하나를 선택하는 구조.
	- video timestep/frame별 expert를 고르는 것이 아님. 한 step에서는 모든 시간·공간 V-token이 같은 expert를 통과함.
- High-noise expert: denoising 초반의 높은 노이즈 구간에서 전체 구도와 큰 구조를 정함.
- Low-noise expert: 후반의 낮은 노이즈 구간에서 질감·세부 사항을 다듬음.
- Denoising은 $x_T$의 high-noise 상태에서 시작해 $x_0$의 clean 상태로 진행하며, SNR threshold $t_{\mathrm{moe}}$에서 high-noise expert에서 low-noise expert로 전환함.
	- 높은 노이즈일수록 SNR이 낮으므로 high-noise expert가 선택됨.
- 각 expert는 약 14B parameter이며, 전체는 약 27B parameter이지만 step마다 하나만 활성화되어 추론 시 활성 parameter는 약 14B.

### Analysis

- Wan2.2의 두 expert를 모두 사용한 설정이 validation loss가 가장 낮음.
- 한쪽 expert만 Wan2.2 것으로 바꾼 혼합 설정보다도 성능이 좋아, 초반의 구조 생성과 후반의 detail refinement가 서로 보완적임을 보임.

## Efficient High-Definition Hybrid T2V

- 별도로 5B dense T2V 모델을 제공하며, 고압축 Wan2.2-VAE를 사용함.
- VAE 압축: 시간 $4\times$, 공간 $16\times16$; latent channel은 48.
	- 정보 압축률은 64. Wan2.1-VAE의 $4\times8\times8$, channel 16, 정보 압축률 48보다 높음.
- patchifying까지 포함한 T2V-5B의 전체 압축은 $4\times32\times32$.
- T2V와 I2V를 하나의 모델로 지원하며, 별도 최적화 없이도 consumer GPU에서 5초 720P 영상을 9분 이내에 생성한다고 보고함.
- 높은 압축률에도 reconstruction 품질을 유지함.
	- 비교표에서 SSIM은 최고 수준(0.922), LPIPS는 최저(0.022)이며, PSNR도 경쟁력 있는 수준(33.223).
