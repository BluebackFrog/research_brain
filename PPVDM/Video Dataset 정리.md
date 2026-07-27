# GT / Reference Video Dataset 정리

## 분류 기준

- **직접 paired reference rollout**: 생성 영상과 동일한 초기 조건을 공유하며, 시뮬레이터가 계산한 물리적으로 올바른 미래 영상을 직접 비교 대상으로 제공
- **physics-labeled GT video**: 영상 자체는 제공하지만 GT가 정답 영상이 아니라 물성값, 물리 사건, 궤적, 가능/불가능 라벨 등으로 주어짐
- **unpaired real-world reference video**: 실제 물리 현상을 담은 영상은 제공하지만, 생성 샘플과 초기 조건이 일치하는 1:1 정답 영상은 아님

> 생성 모델의 출력과 GT 영상을 직접 비교하려는 목적이라면 **PhysInOne**과 **CRONOS**가 가장 명확하다. 나머지는 물리 이해 학습이나 property/event-level 평가에 적합하다.

## 한눈에 보기

| Dataset | 영상 출처 | GT/reference의 역할 | 규모 | 반영하는 핵심 physics |
|---|---|---|---:|---|
| PhysInOne | Synthetic / simulator | **직접 paired reference rollout** | 153,810개 3D scene, 약 200만 video | Mechanics, optics, fluid dynamics, magnetism의 71개 현상 |
| CRONOS | Synthetic / Unreal Engine | **직접 paired reference rollout** | 675개 video | 낙하, 충돌, 가림 뒤 운동의 연속성 |
| IntPhys 2 | Synthetic / Unreal Engine | 가능·불가능 GT가 있는 violation-of-expectation video | 1,416개 video | Permanence, immutability, continuity, solidity |
| PhysVid | Synthetic / Genesis + real-world | 탄성·점도·동마찰계수의 연속값 GT | 총 36,300개 video | 반발, 액체 퍼짐, 마찰 감속 |
| DynSuperCLEVR | Synthetic / PyBullet + Blender | 완전한 4D 상태와 사건 GT | 1,200개 video | 3D 속도·가속도, 중력·마찰, 충돌 |
| Physion++ | Synthetic / TDW | 숨은 물성값과 접촉 결과 GT | 총 9,536개 trial | 질량, 마찰, 탄성, 변형성 |
| ComPhy | Synthetic / Bullet + Blender | target당 물성 추론용 reference video 4개 | 12,000개 set, 60,000개 clip | 질량·전하, 충돌·인력·척력 |
| WISA-32K / WISA-80K | **Real-world Internet videos** | 물리 현상 학습용 unpaired reference corpus | 버전에 따라 32K 또는 약 80K clip | Dynamics, thermodynamics, optics의 17개 현상 |

---

## 1. 직접 paired reference rollout

### PhysInOne

- **데이터 유형**: 전부 synthetic / simulator 기반
	- 일상적인 현상 대부분은 Unreal Engine 5의 Chaos Physics로 시뮬레이션
	- 변형체·입자성 물질은 Taichi 기반 MPM, 액체는 DoriFlow 기반 SPH를 사용
- **규모와 영상 구성**
	- 153,810개 dynamic 3D scene에서 약 200만 개 영상 생성
	- scene당 고정 카메라 12개와 이동 카메라 1개
	- $1120 \times 1120$, 기본 30 FPS, 평균 약 5.2초
- **GT/reference**
	- 동일한 initial frame(s)와 text prompt로 생성한 영상에 대해 simulator video를 $V_{\text{ref}}$로 사용할 수 있음
	- RGB 외에도 depth, object mask, 3D mesh, 3D trajectory, material/physical property, text description을 제공
	- 생성 비디오와 물리적으로 올바른 simulator rollout을 직접 비교하는 평가에 가장 적합
- **반영하는 physics**
	- 71개 기본 현상과 이들을 조합한 3,284개 multi-physics activity
	- Mechanics: 중력, 가속·회전, 충돌, 운동량 보존, Hooke's law, 부력, 변형·파괴, granular motion 등
	- Optics: 반사·굴절, 거울·렌즈·레이저의 광선 거동 등
	- Fluid dynamics: Newtonian/non-Newtonian liquid, 흐름·비산·부력 등
	- Magnetism: 동극 척력, 이극 인력, 자성체의 운동 등
- **주의점**
	- 물리 상태와 카메라를 정밀하게 통제할 수 있지만, 모든 영상은 시뮬레이터 도메인에 속함

### CRONOS

- **데이터 유형**: 전부 synthetic / photorealistic Unreal Engine 기반
- **규모와 영상 구성**
	- Fall 300개, Collision 300개, Occlusion 75개로 총 675개 reference video
	- 3개 사건에 대해 scene 5종, object 5종, viewpoint 최대 4종, appearance 3종을 full-factorial 방식으로 조합
	- Occlusion은 가림 구조를 유지하기 위해 viewpoint를 고정
- **GT/reference**
	- 표준화된 initial condition에서 시뮬레이터가 올바른 rollout을 렌더링
	- 입력 조건과 대응하는 reference video, 사건·intervention metadata, 원본 object mask를 제공
	- 동일 사건에서 scene/object/view/appearance 중 하나만 바꾼 matched counterfactual reference들을 만들 수 있음
- **반영하는 physics**
	- **Fall**: 물체가 표면을 구르다가 가장자리를 벗어나 중력에 의해 자유낙하하는 운동
	- **Collision**: 두 강체의 접촉과 충돌 후 반응, 방향·속도 변화, 강체성 유지
	- **Occlusion**: 물체가 가림막 뒤에서도 연속적인 경로를 따라 움직이고 반대편에서 다시 나타나는 object permanence와 spatio-temporal continuity
- **주의점**
	- 정밀한 counterfactual 비교에는 강하지만 물리 사건이 낙하·충돌·가림의 세 종류로 제한됨

---

## 2. Physics-labeled GT video

### IntPhys 2

- **데이터 유형**: 전부 synthetic / Unreal Engine 기반
- **규모와 영상 구성**
	- Debug 60개, Main 1,012개, Held-out 344개로 총 1,416개, 영상당 약 10초
	- 각 scene은 **물리적으로 가능한 영상 2개 + 불가능한 영상 2개**의 quadruplet으로 구성
	- 정적·이동 카메라와 easy/medium/hard 난이도를 포함
- **GT/reference**
	- GT는 정답 미래 영상이 아니라 각 영상의 **physically possible / impossible** 라벨
	- 같은 scene의 possible/impossible 영상을 비교할 수 있어 violation-of-expectation 평가에 적합
- **반영하는 physics**
	- **Permanence**: 가려져도 물체가 소멸하지 않음
	- **Immutability**: 물체의 형태와 구조가 이유 없이 변하지 않음
	- **Spatio-temporal continuity**: 위치·운동이 순간이동 없이 연속적으로 변함
	- **Solidity**: 두 물체가 같은 공간을 점유하거나 서로를 관통하지 않음

### PhysVid

- **데이터 유형**: synthetic와 real-world를 모두 제공
	- Synthetic: Genesis simulator
	- Real: YouTube 영상 또는 iPhone slow-motion 촬영
- **규모**
	- 각 물성마다 train 10,000개, synthetic test-1 1,000개, synthetic OOD test-2 1,000개, real test-3 100개
	- 세 물성을 합쳐 synthetic 36,000개 + real 300개 = 총 36,300개
- **GT/reference**
	- 정답 rollout이 아니라 각 영상의 연속적인 물성값을 GT로 제공
	- Synthetic GT는 simulator parameter에서 직접 취득
	- Real GT는 아래와 같이 관찰·측정하여 구축
		- 탄성: 수동 표시한 낙하·반발 높이로 $e \approx \sqrt{h_{\text{bounce}} / h_{\text{drop}}}$ 추정
		- 점도: 통제된 촬영 조건에서 사용한 액체의 표준 물성표 값을 사용
		- 동마찰계수: spring dynamometer로 잰 마찰력 $F$와 무게 $G$로 $\mu_k = F/G$ 계산
- **반영하는 physics**
	- **Elasticity**: 낙하한 공의 충돌 전후 속도와 반발 높이
	- **Viscosity**: 떨어진 액체가 바닥에서 퍼지는 면적의 증가율
	- **Dynamic friction**: 초기 속도를 가진 물체가 표면 위에서 마찰로 감속하는 정도
- **주의점**
	- Real split까지 물성 GT를 제공한다는 장점이 있지만, real GT의 획득 방식이 물성마다 달라 정밀도가 균일하지 않음

### DynSuperCLEVR

- **데이터 유형**: 전부 synthetic
	- PyBullet로 3D dynamics를 계산하고 Blender로 렌더링
	- 실제 촬영 HDRI environment map을 배경으로 사용하지만 물체와 운동은 시뮬레이션
- **규모와 영상 구성**
	- Train 1,000개, validation 100개, test 100개로 총 1,200개
	- 영상당 2초, 60 FPS, 120 frame
- **GT/reference**
	- 매 frame의 object identity, 3D position/pose, velocity, acceleration과 collision event를 포함한 완전한 4D scene state 제공
	- 물체의 속도나 가속도를 바꾼 뒤 physics engine으로 다시 계산한 counterfactual scene도 제공
- **반영하는 physics**
	- 3D 속도 상태와 물체 간 상대 속도
	- 마찰에 의한 감속, 중력에 의한 가속, floating 상태
	- 물체 간 충돌과 충돌 이후 궤적 변화
- **주의점**
	- GT가 매우 정밀하지만 생성 모델과 동일 initial condition을 공유하는 범용 reference rollout보다는 4D VideoQA/상태 추론용 데이터셋임

### Physion++

- **데이터 유형**: 전부 synthetic / ThreeDWorld(TDW), Unity3D 기반
- **규모와 영상 구성**
	- 물성당 dynamics pretraining 2,000개, readout 192개, test 192개
	- 4개 물성 합계 9,536개 trial
	- 하나의 영상 안에 물성을 추론하는 inference period와 이후 접촉 결과를 예측하는 prediction period가 들어감
- **GT/reference**
	- 물체별 latent mechanical property 값과 red/yellow object의 향후 접촉 여부 GT 제공
	- 같은 첫 장면이지만 숨은 물성값이 달라 이후 결과가 달라지는 paired trial 포함
- **반영하는 physics**
	- **Mass**: domino 연쇄 충돌, 유체가 물체를 미는 장면, 강체 충돌
	- **Elasticity**: 벽·바닥과 충돌한 물체의 반발과 궤적
	- **Friction**: 경사면/바닥 위 강체와 천의 미끄러짐·정지
	- **Deformability**: 물체가 천을 누르거나 천 위에서 구를 때의 변형과 운동
- **주의점**
	- paired trial은 “같은 조건의 생성 영상 대 GT 영상”이 아니라 latent property가 결과를 바꾸는지 평가하기 위한 쌍임

### ComPhy

- **데이터 유형**: 전부 synthetic
	- Bullet physics engine으로 운동을 계산하고 Blender로 렌더링
- **규모와 영상 구성**
	- Train 8,000 set, validation 2,000 set, test 2,000 set
	- 각 set은 5초 target video 1개와 2초 reference video 4개로 구성되어 총 60,000개 clip
	- 한 set 안에서는 물체의 외형과 숨은 mass/charge가 모든 영상에서 유지되며 초기 위치·속도만 달라짐
- **GT/reference**
	- 여기서 **reference video는 target의 정답 미래가 아니라**, 같은 물체가 다른 조건에서 상호작용하는 관찰 예시
	- 이 reference들에서 충돌·인력·척력을 보고 target 물체의 mass와 charge를 추론하도록 설계
	- object trajectory, event, mass, charge와 미래·counterfactual event GT 제공
- **반영하는 physics**
	- 질량은 light $=1$, heavy $=5$로 설정하며, 충돌 시 관성 차이가 운동 변화에 반영됨
	- 전하는 positive/negative/neutral이며, 동일 전하는 밀어내고 반대 전하는 끌어당김
	- Bullet이 전하를 직접 지원하지 않아 거리의 제곱에 반비례하는 외력을 추가해 Coulomb force를 모사
- **주의점**
	- “reference video”라는 명칭은 쓰지만 생성 비디오의 1:1 GT rollout으로 사용하면 안 됨

---

## 3. Unpaired real-world reference corpus

### WISA-32K / WISA-80K

- **데이터 유형**: 전부 현실 세계를 촬영한 Internet video
	- 시뮬레이터 영상이 아니라 물리 현상이 명확하게 보이는 실제 영상을 수동 검색·수집
	- 자막이 덧씌워졌거나 심하게 흐린 영상을 제외하고 scene detection과 aesthetic filtering 수행
- **버전과 규모**
	- arXiv v1은 **WISA-32K**, 32,000개 영상으로 발표
	- 현재 workspace에 저장된 개정 manuscript는 **WISA-80K**로 확장되어 약 80,000개 clip을 사용
- **GT/reference**
	- 특정 생성 영상과 초기 조건이 일치하는 paired reference는 아님
	- 각 영상에 caption, textual physical description, 29개 qualitative physics category, density/time/temperature 범위를 부착
	- 단, 상세 물리 설명과 정량 범위는 GPT-4o mini가 caption을 바탕으로 생성한 annotation이므로 엄밀한 측정 GT가 아니라 **약한 물리 supervision**에 가까움
- **반영하는 physics**
	- **Dynamics 6종**: collision, rigid-body motion, elastic motion, liquid motion, gas motion, deformation
	- **Thermodynamics 6종**: melting, solidification, vaporization, liquefaction, explosion, combustion
	- **Optics 5종**: reflection, refraction, scattering, interference/diffraction, unnatural light source
- **장점과 한계**
	- 실제 세계의 복잡한 배경, 재질, 조명, 다중 물리 현상을 포함해 sim-to-real gap이 없음
	- 반면 물리 parameter와 초기 조건이 통제되지 않고, 생성 결과에 대응하는 정답 궤적·정답 미래 영상은 없음

---

## 제외한 논문

- **VideoPhy-2**
	- 200개 action과 prompt, 생성 모델 출력 및 human/VLM 평가 라벨을 제공하는 평가 benchmark
	- 물리적으로 올바른 GT/reference video rollout을 별도로 제공하지 않으므로 위 목록에서 제외
- **MotiBench**
	- interaction-centric initial image와 motion-control 조건으로 생성 모델을 평가하는 benchmark
	- 물리적으로 올바른 target video가 있는 paired dataset이 아니므로 제외

## 선택 가이드

- **생성 영상과 물리적으로 올바른 영상을 직접 비교**: PhysInOne, CRONOS
- **실제 영상에서 연속 물성값을 평가**: PhysVid real split
- **객체 존재·연속성·강체성 위반을 평가**: IntPhys 2
- **정밀한 3D 상태·궤적·counterfactual GT가 필요**: DynSuperCLEVR
- **숨은 물성 추론이 필요**: Physion++, ComPhy
- **현실 세계의 다양한 물리 현상으로 학습**: WISA

## 검토한 논문

- [WISA](https://arxiv.org/abs/2503.08153)
- [VideoPhy-2](https://arxiv.org/abs/2503.06800)
- [PhysInOne](https://arxiv.org/abs/2604.09415)
- [MotiMotion / MotiBench](https://arxiv.org/abs/2605.22818)
- [CRONOS](https://arxiv.org/abs/2605.23699)
- [IntPhys 2](https://arxiv.org/abs/2506.09849)
- [PhysVid](https://arxiv.org/abs/2510.02311)
- [DynSuperCLEVR](https://arxiv.org/abs/2406.00622)
- [Physion++](https://arxiv.org/abs/2306.15668)
- [ComPhy](https://arxiv.org/abs/2205.01089)
