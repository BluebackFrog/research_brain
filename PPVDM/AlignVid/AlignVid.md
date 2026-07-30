## 한 줄 정리

- Reference image가 text instruction을 압도하는 **visual dominance**를 attention entropy 문제로 해석하고, 추론 중 $Q/K$ scaling과 block·step scheduling만으로 prompt-specified addition·deletion·modification을 강화하는 training-free 방법.

## Motivation

- **큰 도메인:** text prompt와 reference image로부터 영상을 만드는 text-guided image-to-video generation.
- **기존 문제:** 기존 TI2V 모델은 reference image를 지나치게 보존해 prompt가 요구한 object 추가·삭제·변형을 무시하며, image attention의 우세가 text signal과 motion을 억제한다.
- **해결 방식:** training-free attention scaling + block/denoising-step guidance scheduling.

## Main Method

### 1. Pilot observation: visual dominance

![[AlignVid_visual_dominance.png]]

- 원본 image를 그대로 조건으로 주면 image prior가 text보다 강하게 작용해 attention이 여러 visual token으로 분산되고, 모델이 입력 장면을 거의 복사하는 **semantic negligence**가 발생한다.
- 같은 image에 약한 Gaussian blur를 적용하면 30개 sample 전반에서 image·video·text attention score가 증가하고 entropy는 감소했으며, 특히 text attention 증가가 가장 컸다.
	- Blur가 불필요한 low-level visual detail을 제거해 text와 temporal cue가 다시 경쟁력을 얻은 것으로 해석한다.
	- 실제 생성에서는 prompt adherence와 motion이 좋아지지만 입력 자체를 훼손하므로 aesthetic quality가 떨어질 수 있다.
- AlignVid의 목표는 blur를 실제로 적용하는 대신, 모델 내부 attention만 직접 sharpen하여 같은 효과를 얻는 것이다.

### 2. Energy/temperature 관점의 attention 분석

- 한 attention head에서 video query와 text·image·video key를 각각 $I_{\text{text}},I_{\text{img}},I_{\text{vid}}$로 나누고, 기본 attention을 다음처럼 쓴다.

$$
A(Q,K,V)=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V,
\qquad
Q\in\mathbb{R}^{n_q\times d_k},\;
K\in\mathbb{R}^{n_k\times d_k},\;
V\in\mathbb{R}^{n_k\times d_v}
$$

- $Q$ 또는 $K$에 $\gamma>1$을 곱하면 logit 전체에 $\gamma$가 곱해지므로 softmax의 **inverse temperature**를 높이는 것과 같다.
- 특정 key subset $S$ 안의 attention entropy는 scaling이 커질수록 단조 감소한다.

$$
\frac{d}{d\gamma}H_{i,S}(\gamma)
=-\gamma\,\operatorname{Var}_{p_{i,S}(\gamma)}[z_S^{(i)}]\le 0
$$

- 즉 $\gamma$는 attention의 “초점 조절 손잡이”다. 값을 높이면 낮은 logit의 확률 질량이 소수의 high-logit semantic token으로 이동해 text instruction을 더 결정적으로 선택하지만, 지나치게 높이면 한 token에 과집중해 visual quality가 손상될 수 있다.

### 3. Attention Scaling Modulation (ASM)

- 별도 mask나 학습 module 없이 각 transformer attention 안에서 $Q$ 또는 $K$만 rescale한다.

$$
A_{\mathrm{ASM}}(Q,K,V)
=\operatorname{softmax}\left(\frac{Q'(K')^\top}{\sqrt{d_k}}\right)V
$$

- **Scalar scaling:** 가장 단순하고 최종 실험에서 사용하는 방식이다.

$$
Q'=\gamma_sQ
\quad\text{or}\quad
K'=\gamma_sK,\qquad \gamma_s>1
$$

	- 기본값은 backbone별 탐색 없이 전이 가능한 $\gamma_s=1.35$이다.
	- FramePack처럼 image·text·video token을 한 self-attention에서 처리하면 $Q/K$ scaling이 사실상 대칭이며, 실험에서는 image key와 text key를 함께 scale한다.
	- Wan2.1처럼 image/text가 cross-attention condition인 구조에서는 위치가 비대칭이고, image query + text key 조합이 addition/deletion과 전체 trade-off에서 가장 좋았다.
- **Energy-based modulation:** 평균 attention logit의 sharpness에 따라 scaling 강도를 $f(\cdot)$로 적응적으로 정해 diffuse한 attention에 더 강하게 개입한다.

$$
\gamma_e
=f\left(
\frac{1}{n_qn_k}
\sum_{i,j}\frac{Q_iK_j^\top}{\sqrt{d_k}}
\right)
$$

	- Aesthetic 손실은 scalar보다 작지만 semantic gain도 더 작고 계산이 추가되어, 논문의 기본 설정은 scalar scaling이다.

### 4. Guidance Scheduling (GS)

- ASM을 모든 layer와 denoising step에 적용하면 prompt fidelity는 더 커지지만 texture/detail까지 과도하게 바뀐다. GS는 **어디에**, **언제** ASM을 켤지 제한한다.
- **Block-level Guidance Scheduling (BGS)**
	- 50개 calibration prompt에서 block별 attention을 수집하고 PCA 기반 pseudo-RGB projection과 SAM2 foreground mask를 이용해 high-attention token이 foreground에 놓이는 비율 $r^{(l)}$을 구한다.
	- $r^{(l)}>\tau$인 foreground-sensitive block만 미리 선택하며, 논문은 $\tau=0.5$를 사용한다.
	- 추론 중 선택된 block에는 $\gamma$를, 나머지 block에는 1을 적용한다. 대부분의 foreground-sensitive block은 network 전반부에 있어 “first half blocks”를 쓰는 단순 heuristic도 가능하다.
- **Step-level Guidance Scheduling (SGS)**
	- 초기 denoising step은 global semantics와 구조, 중간은 coarse structure, 후기는 detail을 결정한다.
	- early $[0,0.30]$, middle $[0.35,0.65]$, late $[0.70,1.00]$ schedule을 비교하며, 기본값은 semantic gain이 크고 aesthetic 손실이 작은 **초기 30% step**이다.
- Block $l$, step $t$의 gate를 $b(l),m(t)\in\{0,1\}$이라 하면 실제 scaling은 다음과 같다.

$$
g^{(l,t)}=m(t)b(l)(\gamma-1)
$$

$$
Q'^{(l,t)}=\left(1+s_Qg^{(l,t)}\right)Q^{(l)},\qquad
K'^{(l,t)}=\left(1+s_Kg^{(l,t)}\right)K^{(l)}
$$

- 전체 추론 흐름은 `(reference image, prompt, noisy video latent)` → `선택 block·초기 step에서 ASM` → `sharpened attention` → `기존 denoising update` → `video`이며, weight update나 추가 conditioning input은 없다.

### 5. Conflict-Aware Scaling (CAS, adaptive variant)

- 고정 $\gamma$ 대신 각 token에서 image attention이 text attention보다 강하고 entropy도 높을 때만 scaling하는 search-free variant이다.

$$
C=
\frac{\max(A_{\mathrm{img}}-A_{\mathrm{text}},0)}
{A_{\mathrm{img}}+A_{\mathrm{text}}+\epsilon}H,
\qquad
\gamma=1+C,\quad 0\le C\le0.35
$$

- $\gamma\in[1,1.35]$로 제한해 visual dominance가 실제로 발생한 token에만 개입하며, fixed scalar와 비슷한 semantic fidelity를 내면서 aesthetic quality를 더 잘 보존한다.

### 6. OmitI2V benchmark 구축

- **Task:** 입력은 `(reference image, natural-language edit instruction)`이고, 출력 video는 image의 identity·구조·물리적 일관성을 가능한 한 보존하면서 지시된 **Modification / Addition / Deletion**을 실제 시간 변화로 수행해야 한다.
- 총 367개의 사람이 검수한 image-instruction pair로 구성한다.
	- Edit 비율은 Modification 34.19%, Addition 33.16%, Deletion 32.65%로 거의 균등하다.
	- Real photograph를 중심으로 animation frame과 GPT-4o synthetic image를 섞고, 생물·예술/엔터테인먼트·자연·구조물·사물·기술/가상 요소·음식/생필품·텍스트/커뮤니케이션 등 다양한 domain을 포함한다.
	- Rare/extreme scenario는 GPT-4o로 보강하되 instruction의 의도와 평가 질문은 사람이 설계·검수한다.
- 각 prompt를 “해당 object가 추가됐는가?”, “기존 object가 사라졌는가?”, “요구한 appearance/action으로 변했는가?” 같은 yes/no VQA question으로 바꾸므로, 단순 text-video similarity가 아니라 **edit가 실제로 실행됐는지**를 직접 평가한다.

## 실험

- **OmitI2V Semantic Alignment**
	- 모델은 image와 edit instruction을 받아 video를 생성하고, Qwen2.5-VL-32B가 edit별 yes/no question에 답한 accuracy를 Modification·Addition·Deletion으로 나누어 측정한다.
	- ViCLIP은 prompt와 video embedding의 유사도를 재는 보조 metric으로 ablation에서 사용한다.
- **Visual Quality**
	- **Dynamic Degree:** 영상에 실제 움직임과 시간 변화가 얼마나 발생하는지 측정한다. 정적인 입력 복사로 semantic score를 회피하는지를 함께 확인할 수 있다.
	- **Aesthetic Quality:** frame의 선명도·구도·시각적 매력도를 평가해 attention sharpening이 perceptual quality를 훼손하는지 본다.
- **핵심 결과**
	- FramePack은 `(Modification, Addition, Deletion)` accuracy가 $(64.99,68.55,58.14)\rightarrow(68.22,73.13,60.21)$, Dynamic Degree가 $20.05\rightarrow28.53$으로 개선되며 Aesthetic Quality는 $63.94\rightarrow63.57$로 거의 유지된다.
	- Wan2.1은 accuracy가 $(72.35,71.75,63.13)\rightarrow(77.20,79.54,69.47)$로 모든 edit에서 개선되고, Dynamic Degree는 $46.02\rightarrow47.04$, Aesthetic Quality는 $63.12\rightarrow61.63$이다.
	- 60개 sample·5명 평가자의 1-7점 human study에서도 FramePack 대비 semantic fidelity가 Addition $3.05\rightarrow5.72$, Deletion $3.20\rightarrow5.80$, Modification $3.16\rightarrow5.82$로 상승했고 aesthetic 점수는 거의 동일했다.
- **Generalization**
	- 동일한 scaling을 T2I의 GenEval, T2V의 VBench, image editing의 ImgEdit에 적용해 다수의 semantic dimension이 개선됐다.
	- 다만 T2V에서는 stronger motion과 temporal change가 motion blur를 유발해 imaging/aesthetic quality 및 일부 motion metric이 하락할 수 있다.

## Ablation / Analysis

- **Scalar vs. energy-based modulation:** 두 방식 모두 semantic fidelity를 높이지만 scalar가 더 큰 gain을 내고, energy-based 방식은 gain이 작은 대신 aesthetic 하락도 작다.
- **Scaling 위치:** FramePack은 image+text key를 함께 scale할 때 가장 안정적이고, Wan2.1은 cross-attention의 $Q/K$ 역할이 달라 image query + text key 조합이 addition/deletion에서 가장 좋다.
- **BGS:** Foreground-focused block만 사용하면 background-focused·last-half block보다 semantic fidelity가 높고 all-block scaling의 aesthetic 손실을 줄인다.
- **SGS:** Early-step scaling이 mid/late보다 semantic gain이 크며, all-step scaling은 가장 높은 일부 semantic score를 내지만 FramePack aesthetic score가 $63.94\rightarrow61.56$으로 떨어진다.
- **Scaling coefficient:** $\gamma$가 커질수록 attention entropy가 단조 감소하고 semantic fidelity·motion은 증가하지만 aesthetic quality는 서서히 감소한다. $\gamma=1.35$가 세 backbone에서 공통으로 균형이 좋았다.
- **CAS:** FramePack Addition에서 fixed $\gamma=1.35$의 $73.44$에 근접한 $72.37$을 얻으면서 Aesthetic Quality는 $63.41\rightarrow63.77$로 더 잘 보존한다.
- **Attention analysis:** ASM 이후 video query의 text token·인접 frame attention은 증가하고 static image region attention과 entropy는 감소해, visual dominance 완화라는 초기 가설과 실제 내부 변화가 일치한다.
- **VQA evaluator 신뢰도:** 사람이 재검수한 Qwen2.5-VL-32B 판정의 전체 error는 3.38%이며, 주된 오류는 작거나 가려진 object를 놓치는 false negative와 미세한 변화를 과장하는 false positive다.
- **효율:** H100 기준 video당 overhead는 FramePack +0.09%, FramePack-F1 +0.03%, Wan2.1 +0.01%로 사실상 negligible하다.
- **실질적 한계:** Prompt adherence와 motion을 강하게 만들수록 blur·texture 손상 가능성이 커지는 semantic-aesthetic trade-off가 남으며, task/backbone에 따라 scaling 위치와 강도를 잘못 선택하면 오히려 품질이 크게 떨어질 수 있다.
