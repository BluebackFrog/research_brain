# Physical Plausibility Metric 정리

## 범위

- 생성 영상 자체의 물리적 타당성을 평가하는 metric만 정리한다.
- **계산식 자체가 물리 현상을 측정하지 않는 일반적인 task 성능 지표**는 정리 범위에서 제외한다.
- 계산 기반 metric은 다음 두 종류를 구분한다.
	- 순수 계산 기반: 영상이나 reference에서 얻은 motion·shape·appearance 값을 고정된 수식으로 비교한다.
	- Feature 기반 계산: DINOv2, DisMo, SAM3D 등 학습된 encoder를 사용하지만, 최종 판정은 VLM의 자연어 판단이 아니라 고정된 거리·유사도 계산으로 수행한다.

## 한눈에 보기

| 논문/Benchmark | 핵심 metric | 판정 방식 | Reference 필요 여부 |
|---|---|---|---|
| PhysInOne | PMF | Fourier motion energy 비교 | 필요 |
| CRONOS | Object/Background/Shape Stability, Motion Similarity | Feature·거리 기반 계산 | 일부 필요 |
| WISA | VideoCon-Physics PC | VLM-as-a-Judge | 불필요 |
| VideoPhy-2 | PC, Physical Rule, Joint Performance | Human/VLM-as-a-Judge + threshold 집계 | 불필요 |
| MotiMotion | Physical Realism, 2AFC Preference | VLM/Human pairwise judge | 불필요 |
| CRONOS | Physical Plausibility | VLM binary-question judge + 위반 수 집계 | 불필요 |

---

## 1. 계산 기반 Metric

### ~={cyan}PMF (Physical Motion Fidelity) — PhysInOne=~

- 목적: 생성 영상과 물리 simulator reference 영상의 **motion dynamics가 얼마나 유사한지** 측정한다.
- 계산:
	1. 생성 영상 $V_{\mathrm{gen}}$과 reference 영상 $V_{\mathrm{ref}}$에 공간·시간축 3D DFT를 적용한다.
	2. Fourier coefficient의 제곱 진폭을 전체 주파수 에너지로 정규화하여 $E^{\mathrm{gen}}$, $E^{\mathrm{ref}}$를 얻는다.
	3. 두 energy distribution의 Total Variation distance를 계산한다.

$$
d_{\mathrm{TV}}(E^{\mathrm{gen}},E^{\mathrm{ref}})
=
\frac{1}{2}\sum_{u,v,s}
\left|E^{\mathrm{gen}}_{u,v,s}-E^{\mathrm{ref}}_{u,v,s}\right|
$$

$$
\mathrm{PMF}(V_{\mathrm{gen}},V_{\mathrm{ref}})
=
-\ln d_{\mathrm{TV}}(E^{\mathrm{gen}},E^{\mathrm{ref}})
$$

- 해석:
	- 높을수록 reference와 motion spectrum이 유사하다.
	- 초기 위치·시점 offset과 밝기 변화에는 비교적 둔감하다.
	- 물리 법칙을 직접 검사하는 것이 아니라, **물리적으로 정확한 reference와 움직임 패턴이 비슷한지** 측정한다.
	- 하나의 reference rollout만 정답으로 간주할 수 있다는 한계가 있다.

### Object Stability — CRONOS

- 목적: 생성 도중 객체의 identity와 appearance가 변형되는지 측정한다.
- 계산:
	1. 각 프레임에서 객체 영역을 segmentation한다.
	2. 배경을 제거한 객체 crop의 DINOv2 CLS feature를 추출한다.
	3. 첫 프레임과 이후 프레임의 cosine similarity를 계산하고, 객체·시간축에서 robust minimum을 취한다.

$$
\mathrm{ObjectStability}
=
\operatorname{RobustMin}_{i,t}
\cos\left(f_i^t,f_i^0\right)
$$

- 해석:
	- 높을수록 객체의 appearance와 identity가 안정적이다.
	- 평균 대신 robust minimum을 사용해 특정 객체나 짧은 구간에서만 발생한 심각한 변형도 반영한다.
	- DINOv2를 사용하므로 완전한 hand-crafted metric은 아니지만, VLM judge처럼 자연어로 판정하지는 않는다.

### Background Stability — CRONOS

- 목적: 배경 morphing, 카메라 이동, 조명 변화, 새로운 배경 객체 출현 등을 측정한다.
- 계산:
	1. 첫 프레임과 이후 프레임에서 공통으로 background인 pixel만 선택한다.
	2. 공통 background 영역의 MSE를 계산하고 시간축 robust maximum을 취한다.
	3. 큰 오차가 낮은 score가 되도록 exponential decay를 적용한다.

$$
e_{\mathrm{BG}}
=
\operatorname{RobustMax}_t
\mathrm{MSE}(B^t,B^0)
$$

$$
\mathrm{BackgroundStability}
=
\exp(-50e_{\mathrm{BG}})
$$

- 해석: 높을수록 배경과 카메라가 안정적이다.

### Shape Stability — CRONOS

- 목적: rigid object가 생성 과정에서 찌그러지거나 다른 형태로 morphing되는 현상을 측정한다.
- 계산:
	1. ~={cyan}SAM3D로 프레임별 객체 mesh $M_i^t$를 복원한다.=~
	2. scale과 rotation을 첫 프레임 mesh에 정렬한다.
	3. 첫 프레임과 이후 프레임 mesh 사이 Chamfer distance의 robust maximum을 계산한다.
	4. 거리를 exponential decay로 변환하여 0–1 score로 만든다.

$$
e_{\mathrm{shape}}
=
\operatorname{RobustMax}_{i,t}
\mathrm{CD}(M_i^t,M_i^0)
$$

- 해석: 높을수록 객체의 3D shape이 안정적이다.

### ~={cyan}Motion Similarity — CRONOS=~

- 목적: 생성 영상의 움직임이 simulator reference의 움직임과 유사한지 측정한다.
- 계산:
	1. ~={cyan}생성 영상과 reference 영상을 appearance-invariant motion encoder인 DisMo에 입력한다.=~
	2. 대응 시점 motion embedding의 cosine similarity를 계산한다.
	3. 시간축에서 robust minimum을 취한다.

$$
\mathrm{MotionSimilarity}
=
\operatorname{RobustMin}_t
\cos\left(g(I^t),g(\tilde I^t)\right)
$$

- 해석:
	- 높을수록 reference와 동작 패턴이 유사하다.
	- 객체 색상이나 category보다 motion 차이에 집중하도록 설계되었다.

### Success Rate — CRONOS

- 목적: 한 metric의 심각한 실패가 다른 metric과 평균되면서 가려지는 것을 방지한다.
- 계산:
	- Object, Background, Shape, Motion, Physical Plausibility가 각각 human-calibrated threshold를 모두 통과해야 성공이다.
	- 객체 disappearance가 감지된 영상은 실패로 처리한다.

$$
\mathrm{Success}(V)
=
\mathbf{1}
\left[
\bigwedge_m S_m(V)\ge\tau_m
\;\land\;
\text{no disappearance}
\right]
$$

$$
\mathrm{SuccessRate}
=
\frac{1}{N}\sum_{i=1}^{N}\mathrm{Success}(V_i)
$$

### Intervention Sensitivity — CRONOS

- 목적: 동일한 물리 사건이 viewpoint, scene, appearance, object category 변화에도 일관되게 생성되는지 평가한다.
- 계산:
	- 한 intervention group 안에서 metric의 최고값과 최저값 차이를 구해 group·metric에 대해 평균한다.

$$
\mathrm{Sensitivity}
=
\mathbb{E}_{g,m}
\left[
\max_{v\in g}S_m(v)-\min_{v\in g}S_m(v)
\right]
$$

- 해석:
	- 낮을수록 조건 변화에 대한 counterfactual physical consistency가 높다.
	- 절대적인 물리성보다는 **조건 변화에 따른 성능 변동**을 측정한다.

---

## 2. VLM-as-a-Judge Metric

### VideoCon-Physics PC — WISA

- 입력:
	- PC 평가: 생성 영상
	- SA 평가: 생성 영상과 conditioning caption
- VLM rubric:
	- PC: 영상 전체가 real-world physical commonsense를 따르는가?
	- SA: caption에 명시된 객체, 행동, 관계가 영상에 나타나는가?
- 출력과 집계:
	- VideoCon-Physics가 Yes/No token probability를 출력한다.

$$
s
=
\frac{p(\mathrm{Yes})}
{p(\mathrm{Yes})+p(\mathrm{No})}
$$

	- WISA는 $s\ge0.5$이면 해당 항목을 통과한 것으로 처리하고 전체 영상의 통과 비율을 보고한다.
- 특징과 한계:
	- PC rubric은 “전체적으로 물리적으로 가능한가?”에 가까운 holistic 판정이다.
	- 어떤 객체가 어떤 법칙을 얼마나 위반했는지를 세부적으로 분해하지 않는다.
	- SA는 physics metric이 아니며 PC가 prompt와 무관한 엉뚱한 영상을 높게 평가하는 문제를 방지하기 위한 보조 지표다.

### PC와 Physical Rule — VideoPhy-2

#### Physical Commonsense (PC)

- 입력: 생성 영상. PC 판정은 원칙적으로 conditioning prompt 일치 여부와 독립적으로 수행한다.
- VLM/Human rubric:
	- 중력과 관성에 맞게 움직이는가?
	- 충돌 후 반응이 자연스러운가?
	- 질량, 운동량, 마찰, 탄성, 부력 등의 물리 상식과 충돌하는가?
	- 객체가 이유 없이 생성·소멸·복제되는가?
	- 물체의 형태나 재질 특성이 비정상적으로 변하는가?
- 출력: 1–5점 Likert score
	- 1점: 다수의 근본적인 물리 법칙 위반이 관찰된다.
	- 5점: 명확한 물리 법칙 위반 없이 자연스럽다.

#### Physical Rule (PR)

- 입력: 생성 영상과 영상별 candidate physical rule
- rubric:
	- 해당 rule이 영상에서 실제로 지켜졌는가?
	- 명백히 위반되었는가?
	- 영상에서 확인할 수 없어 판단 불가능한가?
- 출력:
	- 0: violated
	- 1: followed
	- 2: cannot be determined
- 예시:
	- “돌은 경사면 아래로 굴러야 한다.” — Gravity
	- “수건에서 수건이 보유할 수 있는 양보다 많은 물이 생성되면 안 된다.” — Conservation of Mass
	- “충돌 후 운동 방향과 속도 변화가 물체의 질량 관계에 부합해야 한다.” — Conservation of Momentum

#### Joint Performance

- PC 판정에 Semantic Adherence를 gate로 결합한 hybrid metric이다.

$$
\mathrm{Joint}
=
\frac{1}{N}
\sum_i
\mathbf{1}[SA_i\ge4\land PC_i\ge4]
$$

- Joint 자체는 threshold 집계식이지만, 입력인 SA와 PC가 Human/VLM 판단이므로 rule-based physics metric으로 보기는 어렵다.

### Physical Realism과 2AFC — MotiMotion

- VLM 입력:
	- 초기 context image
	- 사용자 trajectory 및 trajectory overlay
	- 사용자 prompt
	- 비교할 두 생성 영상
- rubric:
	- Object Property
		- 객체의 질량감과 중력 반응이 자연스러운가?
		- 재질에 맞는 강성·탄성·변형을 보이는가?
		- rigid object와 soft object가 구분되는가?
	- Interaction
		- 충돌과 접촉력이 올바른 결과를 만드는가?
		- 지지점 제거, 외력 발생·소멸, 압력 해제 등이 적절한 secondary motion을 유발하는가?
		- 자석, 기류, 유체, 기어, 레버, 래치 등의 interaction과 mechanism이 논리적으로 작동하는가?
	- Overall
		- 객체 특성과 상호작용을 종합했을 때 어느 영상이 더 물리적으로 자연스러운가?
- 출력:
	- verdict: A 승리, B 승리, Tie
	- 차이 강도: Slight, Moderate, Strong
- 집계:
	- Slight/Moderate/Strong에 각각 1/2/3점을 부여하고 Tie는 0점으로 처리한다.

$$
S_M=\sum_{k\in W_M}W(s_k)
$$

$$
\mathrm{WinRate}(A)
=
\frac{S_A}{S_A+S_B}\times100
$$

- 최종 win rate는 계산식이지만, 승패와 강도는 VLM 또는 사람이 결정하는 hybrid metric이다.

### Physical Plausibility — CRONOS

- VLM: Qwen3-VL-32B
- 입력:
	- 생성 영상
	- 영상의 사건 description
	- 공통 질문 5개와 event-specific 질문 5개
- 출력:
	- 각 질문의 True/False
	- 판정 근거
	- 0–1 confidence
- 공통 rubric:
	- 배경이 전체 영상에서 정적인가?
	- 객체의 색상과 형태가 유지되는가?
	- 새로운 객체가 갑자기 등장하는가?
	- 객체가 jump·teleport 없이 부드럽게 움직이는가?
	- 외력이 없는데 객체가 스스로 움직이는가?
- Fall rubric:
	- 객체가 edge에 도달했을 때 실제로 떨어지는가?
	- 바닥에 충돌하는가?
	- 낙하 중 이유 없이 방향이 바뀌거나 비정상적인 arc를 그리는가?
	- 낙하하면서 적절히 가속하는가?
- Collision rubric:
	- 두 객체가 실제로 접촉하는가?
	- 접촉 전에 이유 없이 멈추는가?
	- 충돌이 객체의 운동에 영향을 주는가?
	- 물체의 크기와 질량을 고려했을 때 충돌 반응이 현실적인가?
	- 예상하지 않은 파손이나 변형이 발생하는가?
- Occlusion rubric:
	- 객체가 occluder 뒤로 이동하는가?
	- 반대편에서 다시 나타나는가?
	- 가려지는 동안 trajectory가 일관적인가?
	- 객체가 사라지거나 외형이 달라지는가?
- 집계:
	- VLM 답변이 video-specific ideal answer와 다른 문항의 수를 센다.

$$
\mathrm{PhysicalPlausibility}
=
\left(
1+
\sum_b
\mathbf{1}[\hat b_{\mathrm{VLM}}\ne b_{\mathrm{ideal}}]
\right)^{-1}
$$

- 해석:
	- 위반 0개면 1, 1개면 0.5, 2개면 약 0.33이다.
	- 단순 평균보다 소수의 명확한 물리 위반에 강한 penalty를 준다.
	- 질문별 expected answer가 미리 정의되어 있으므로 open-ended holistic judge보다 진단성이 높다.

---

## 3. Human Judge Rubric

### WISA

- Semantic Consistency와 Physical Alignment를 기준으로 후보 모델을 순위화한다.
- 1위 3점, 2위 2점, 3위 0점으로 집계한다.
- Physical Alignment의 세부 법칙별 rubric은 제공하지 않아 holistic preference에 가깝다.

### VideoPhy-2

- VLM AutoEval의 학습 및 검증에 사용하는 gold judgment다.
- 세 명의 annotator가 SA와 PC를 각각 1–5점으로 평가하고 평균 후 반올림한다.
- candidate physical rule은 다수결로 followed/violated/cannot determine를 결정한다.

### PhysInOne

- reference 영상을 먼저 본 뒤 여러 생성 영상을 물리적 자연스러움 순으로 정렬한다.
- rubric:
	- Trajectory Fidelity: timing, direction, acceleration, interaction이 reference와 유사한가?
	- Prompt-Aligned Physics: prompt가 요구한 물리 조건을 따르는가?
	- Trajectory Smoothness: jitter나 불가능한 운동 전환이 없는가?
	- Shape Consistency: 객체 geometry가 안정적인가?
	- Semantic Consistency: 객체 identity와 category가 유지되는가?
	- Object Persistence: 객체가 생성·소멸·복제되지 않는가?

### MotiMotion

- VLM과 동일한 Object Property, Interaction, Overall 2AFC rubric을 사용한다.
- 승패와 Slight/Moderate/Strong 차이 강도를 함께 선택한다.

---

## 4. PPVDM에서의 활용 관점

- 물리 simulator reference가 있는 경우:
	- PMF로 전체 motion spectrum 유사도를 평가한다.
	- CRONOS Motion Similarity와 Shape Stability로 motion과 geometry failure를 분리한다.
- reference가 없는 open-domain 생성 평가:
	- VideoPhy-2 PC로 전체적인 physical commonsense를 평가한다.
	- Physical Rule 또는 CRONOS처럼 질문을 세분화해 어떤 법칙이 위반되었는지 기록한다.
- 권장 보고 방식:
	- 하나의 총점만 사용하지 않고 `motion / shape / object persistence / interaction / law violation`을 분리한다.
	- Semantic Adherence는 physical score와 별도로 보고하여, 물리적으로 자연스럽지만 prompt와 무관한 영상이 높은 평가를 받는 것을 방지한다.
	- VLM judge는 소규모 human evaluation과의 agreement를 함께 검증한다.

## 핵심 한계 및 연구 공백

- 위 논문들의 계산 기반 metric은 대부분 reference similarity 또는 visual stability를 측정한다.
- 운동량 보존 오차, 에너지 보존 오차, 중력가속도 오차, coefficient of restitution 오차처럼 **명시적인 물리 방정식의 residual을 영상에서 직접 계산하는 metric은 거의 사용되지 않는다.**
- 따라서 PPVDM에서는 다음과 같은 equation-based metric을 별도로 설계할 여지가 있다.
	- 추정된 물체 궤적으로부터 충돌 전후 momentum residual 계산
	- 낙하 구간의 추정 가속도와 중력가속도 비교
	- 충돌 전후 속도로 restitution consistency 계산
	- 폐쇄계에서 kinetic/potential energy 변화의 허용 범위 검사
