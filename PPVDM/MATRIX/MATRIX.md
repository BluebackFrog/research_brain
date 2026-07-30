## 한 줄 정리

- Video DiT의 interaction 표현을 **noun/verb의 spatial grounding**과 **instance binding의 temporal propagation**으로 분해해 분석하고, 성공·실패를 가르는 소수 attention layer만 mask track에 정렬하도록 학습하는 방법.

## Motivation

- **큰 도메인:** multi-instance 또는 subject-object interaction을 정확히 수행하는 image-to-video generation.
- **기존 문제:** Video DiT는 prompt의 “누가 무엇을 누구에게 하는지”를 잘못 bind하거나, 올바른 binding도 시간축에서 유지하지 못해 instance drift·복제·소멸·hallucination을 일으킨다.
- **해결 방식:** interaction-aware dataset 구축 + attention 분석 + mask-track-based attention alignment를 이용한 selective LoRA fine-tuning.

## Main Method

### 1. MATRIX-11K 구축

- 각 sample은 video $V$, interaction-aware prompt $P$, persistent instance ID별 mask track $\{M_k\}$로 구성되며, 한 clip에서 최대 5개 instance track을 사용한다.
- **Interaction-aware captioning**
	- LLM이 caption에서 interaction을 찾아 $\langle k_{\mathrm{sub}}, \mathrm{verb}, k_{\mathrm{obj}}\rangle$ triplet으로 변환하고, 참여 noun마다 고유 ID와 base class를 부여한다.
	- interaction의 **Contactness**와 **Dynamism**을 각각 1-5점으로 평가해 기준 이하 interaction을 제거한다.
	- 같은 class의 여러 instance를 구분할 수 있도록 각 ID의 appearance description도 추출한다.
- **Multi-instance mask track 생성**
	- 균일하게 sampling한 frame에 GroundingDINO를 적용해 ID별 bounding-box 후보를 confidence 순으로 얻는다.
	- VLM이 `class + appearance description + candidate crop`을 보고 실제 대상과 일치하는지 검증하며, 처음 통과한 box를 anchor로 선택한다.
	- anchor box에서 SAM2를 시작해 전체 clip으로 mask를 propagation하고, drift·누락·clip 불일치가 남은 sample은 사람이 최종 제거한다.
- 최종 데이터는 HOIGen 78.6%와 PE-Video 21.4%에서 구성되며, dynamic하고 contact-rich한 interaction을 중심으로 약 11K clip을 포함한다.

### 2. Video DiT 내부의 interaction 표현 분석

- MM-DiT의 3D full-attention은 video/text token을 하나의 sequence로 연결하므로 attention matrix를 $A^{v2v}$, $A^{v2t}$, $A^{t2v}$, $A^{t2t}$ 네 영역으로 나눌 수 있다.
- **Semantic Grounding - $A^{v2t}$**
	- subject/object의 head noun과 modifier token attention을 평균해 $A^{v2t}_{\mathrm{sub}}, A^{v2t}_{\mathrm{obj}}\in\mathbb{R}^{F\times H\times W}$를 만든다.
	- verb와 auxiliary/particle token을 평균해 $A^{v2t}_{\mathrm{verb}}$를 만들며, target은 subject/object mask의 union인 $M_{\mathrm{verb}}=M_{\mathrm{sub}}\cup M_{\mathrm{obj}}$이다.
	- 즉 noun은 각 instance 영역, verb는 두 instance가 함께 만드는 interaction 영역에 attention이 모이는지를 본다.
- **Semantic Propagation - $A^{v2v}$**
	- 첫 frame의 subject/object mask가 1인 latent 위치를 query set $Q_{\mathrm{sub}},Q_{\mathrm{obj}}$로 사용하고, $Q_{\mathrm{verb}}=Q_{\mathrm{sub}}\cup Q_{\mathrm{obj}}$로 정의한다.
	- 각 query에서 모든 spatiotemporal video token으로 가는 $A^{v2v}$를 평균해 instance별 propagation heatmap을 만든다.
	- 이 heatmap이 이후 frame에서도 같은 mask track을 따라가면 identity와 interaction binding이 유지된 것으로 해석한다.
- **Attention Alignment Score (AAS)**
	- attention이 GT mask 안에 얼마나 집중되는지를 다음처럼 계산한다.

$$
\mathrm{AAS}^{v2t}_e=\sum_{f,h,w}(A^{v2t}_e\odot m_e)_{f,h,w},\qquad
\mathrm{AAS}^{v2v}_e=\sum_{f,h,w}(A^{v2v}_e\odot m_e)_{f,h,w}
$$

	- CogVideoX-5B-I2V의 42개 layer와 50개 denoising step에서 AAS를 측정한다.
	- video별 AAS 상위 10개 layer의 **등장 빈도 + 평균 크기**로 influential layer 후보를 고르고, human-labeled 성공/실패 영상 사이 AAS gap이 큰 layer를 interaction-dominant layer로 선택한다.
	- 최종적으로 grounding에는 layer 7·11, propagation에는 layer 12를 사용한다.

### 3. MATRIX 학습 구조

![[MATRIX_framework.png]]

- **입력**
	- noisy video latent $z_t$
	- 첫 RGB frame $x_0$
	- 첫 frame의 instance ID map $I_0$: instance별 binary mask에 고정 palette ID를 부여해 합친 map
	- subject·verb·object token이 식별된 prompt $P$
- $x_0$와 $I_0$를 latent와 channel-wise concatenate하도록 input projection을 확장한다. 기존 channel weight는 pretrained weight를 복사하고 새 channel은 zero initialization해 학습 시작 시 base model의 동작을 보존한다.
- 선정된 attention layer에만 LoRA를 부착하고 나머지 transformer·3D VAE는 freeze한다.
	- LoRA rank는 128, $\alpha=64$.
	- trainable component는 layer 7·11·12의 LoRA, 확장된 input projection, grounding/propagation decoder이다.
- **Attention readout**
	- layer 7·11의 $A^{v2t}$에서 subject·verb·object grounding map을, layer 12의 $A^{v2v}$에서 subject·object propagation map을 추출한다.
	- attention head를 평균하고 DiT latent grid로 reshape하면 시간축 길이가 13인 map이 나온다.
	- lightweight causal decoder $\mathcal{D}_\phi$가 CogVideoX 3D VAE의 temporal upsampling schedule을 따라 이를 49-frame pixel-grid mask로 복원한다.
	- 4개 frame을 단순 OR해 latent supervision으로 만드는 대신 $13\rightarrow49$ causal upsampling을 사용하므로 motion 중 mask가 팽창하거나 시간 순서가 사라지는 문제를 피한다.

### 4. Attention alignment loss

- decoder가 복원한 attention map $\hat A_e=\mathcal{D}_\phi(A_e)$와 GT mask track $M_e$를 BCE, soft Dice, $L_2$의 조합으로 정렬한다.

$$
\ell(X,Y)=\beta_{\mathrm{bce}}\mathrm{BCE}(X,Y)
+\beta_{\mathrm{dice}}\left(1-\mathrm{Dice}(X,Y)\right)
+\beta_2\lVert X-Y\rVert_2^2
$$

$$
\mathcal{L}_{\mathrm{SGA}}
=\sum_{e\in\{\mathrm{sub,obj,verb}\}}\ell(\hat A^{v2t}_e,M_e),\qquad
\mathcal{L}_{\mathrm{SPA}}
=\sum_{e\in\{\mathrm{sub,obj}\}}\ell(\hat A^{v2v}_e,M_e)
$$

$$
\mathcal{L}_{\mathrm{total}}
=\mathcal{L}_{\mathrm{DM}}
+\lambda_{\mathrm{SGA}}\mathcal{L}_{\mathrm{SGA}}
+\lambda_{\mathrm{SPA}}\mathcal{L}_{\mathrm{SPA}}
$$

- **SGA (Semantic Grounding Alignment):** video query가 올바른 noun/verb text key를 보도록 만들어 subject·object·interaction region을 정확히 bind한다.
- **SPA (Semantic Propagation Alignment):** 첫 frame에서 형성된 instance binding이 전체 frame의 같은 instance track으로 이어지게 한다.
- CogVideoX-5B-I2V를 $480\times720$, 49 frame으로 4K step 학습하며, 단일 NVIDIA A6000에서 약 32시간이 걸린다.

### 5. 추론

- 사용자는 첫 frame과 prompt, 첫 frame의 instance ID mask만 제공한다. ID mask는 SAM2 같은 off-the-shelf segmentor로 얻을 수 있다.
- full mask track은 **학습 supervision에만** 필요하고, grounding/propagation decoder도 학습 후 제거되므로 추론 시 추가 frame-level control이나 decoder overhead가 없다.
- 따라서 base I2V generator의 출력 형식은 그대로 유지하면서 prompt가 지정한 instance와 interaction을 더 안정적으로 생성한다.

## 실험

- **Benchmark**
	- 60개 synthetic pair와 58개 real pair로 총 118개의 `(first image, interaction prompt)`를 구성하며, prompt에는 2-4개 instance의 appearance와 subject-object interaction이 포함된다.
	- 모델은 첫 image와 prompt를 받아 interaction video를 생성해야 하며, contact/dynamism이 낮은 단순 motion부터 contact-rich interaction까지 평가한다.
- **InterGenEval**
	- SAM2로 instance를 추적해 고유 색 bounding box가 표시된 frame sequence를 만들고, GPT-5가 interaction마다 10개 yes/no 질문을 평가한다.
	- **KISA:** pre 1개, during 4개, post 1개 질문으로 action이 예상 단계와 결과를 실제로 수행했는지 측정한다.
	- **SGI:** subject가 actor인지, 올바른 object에 action을 수행하는지 등 4개 질문으로 “누가 무엇을 누구에게”의 grounding을 측정한다.
	- **SPI:** 첫 frame을 anchor로 instance의 갑작스러운 등장·소멸 frame 비율을 구해 temporal consistency penalty를 만든다. 논문은 penalty weight $\lambda=5$를 사용한다.
	- $\mathrm{KISA}=\mathrm{KISA}_{raw}\times\mathrm{SPI}$, $\mathrm{SGI}=\mathrm{SGI}_{raw}\times\mathrm{SPI}$로 보정하고 $\mathrm{IF}=(\mathrm{KISA}+\mathrm{SGI})/2$를 최종 interaction fidelity로 보고한다.
- **추가 metric**
	- **HA:** frame마다 사람·손·얼굴의 merge, split, deformation이 없는 비율로 human anatomy fidelity를 측정한다.
	- **MS:** video interpolation motion prior를 이용해 motion의 연속성과 smoothness를 측정한다.
	- **IQ:** MUSIQ로 blur·noise·과노출 같은 frame-level distortion을 측정한다.
- **핵심 결과**
	- CogVideoX-5B-I2V 대비 KISA는 $0.406\rightarrow0.546$, SGI는 $0.491\rightarrow0.641$, IF는 $0.449\rightarrow0.593$으로 상승했다.
	- HA도 $0.936\rightarrow0.954$로 개선됐고 MS $0.994$, IQ $69.73$을 유지해, interaction fidelity 향상이 일반 영상 품질 저하에서 나온 것이 아님을 보였다.
	- Wan2.1-14B-I2V에도 같은 SGA/SPA를 적용했을 때 interaction motion과 binding이 개선되어 3D full-attention 기반 Video DiT에 plug-and-play로 적용 가능함을 보였다.

## Ablation / Analysis

- **MATRIX-11K만 사용한 naive LoRA:** IF가 $0.449\rightarrow0.486$으로 올라 curated interaction data 자체의 효과는 있지만, attention을 직접 정렬하지 않아 개선 폭이 제한적이다.
- **SPA만 추가:** IF $0.496$으로 propagation과 smoothness는 개선되지만 명시적 noun/verb grounding이 없어 interaction 정확도 향상은 작다.
- **SGA의 attention 방향:** $A^{t2v}$를 감독한 경우 IF $0.531$, $A^{v2t}$를 감독한 경우 $0.550$이다. 하나의 text query가 여러 spatial location에 대응하는 $A^{t2v}$보다, 생성 중인 각 video region이 올바른 text token을 조회하게 하는 $A^{v2t}$가 안정적이다.
- **SGA + SPA:** IF $0.593$으로 가장 높다. 먼저 subject·verb·object를 올바르게 ground하고 그 binding을 시간축으로 propagate하는 두 loss가 상호 보완적이다.
- **Interaction-dominant layer 분석:** 성공 영상은 선택 layer의 AAS가 전체 평균보다 높고 실패 영상은 낮아, interaction 정보가 모든 layer에 균등하게 퍼진 것이 아니라 소수 layer에 집중됨을 확인했다.
- **효율:** full MATRIX는 LoRA-only 대비 학습 시간이 약 24.43시간에서 31.09시간으로 늘지만, inference에서는 decoder head를 제거하므로 추가 비용이 거의 없다.
- **한계:** 최대 5개 instance track만 지원하며, 화면에서 매우 작은 instance는 mask와 grounding signal이 약해져 정확도가 떨어질 수 있다.
