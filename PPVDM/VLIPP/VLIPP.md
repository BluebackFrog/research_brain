## 3. Method

- 목표: 첫 프레임 이미지 `I`와 텍스트 설명 `d`로부터 물리 법칙을 따르는 I2V 결과 생성.
- 핵심 전략: VLM이 물리 기반 coarse motion을 계획하고, VDM이 fine-level video를 합성함.

### Overall Pipeline

- VLM이 이미지와 prompt를 보고 물리 현상에 관련된 객체를 찾음.
- Grounded-SAM2로 객체 bounding box를 얻음.
- LLM/VLM이 적용될 물리 법칙을 판단함.
	- gravity, momentum conservation, optics, thermodynamics, magnetism, fluid mechanics
- VLM이 객체들의 coarse trajectory를 bounding box 변화로 예측함.
- 예측 trajectory로 synthetic motion video를 만들고 optical flow를 추출함.
- optical flow를 structured noise로 바꿔 motion-controllable I2V diffusion model에 넣음.

### 3.1. VLM as a Coarse-Level Motion Planner

- VLM은 비디오를 직접 만들지 않고, 객체의 미래 bounding box를 예측함.
	- ~={red}근데 이거 VLM이 정확히 bounding box 위치를 설정한다는 세팅이 좀 이상한 것 같음=~
- box 형식: `b_i^t = [x_i^t, y_i^t, w_i^t, h_i^t]`
- CoT reasoning:
	- caption에서 물리 법칙 파악
	- 객체 간 상호작용/움직임 분석
	- 시간에 따른 box 위치/크기 변화 예측
- token length 제한 때문에 약 12 frame만 예측함.
- 이후 선형 보간으로 49 frame짜리 dense trajectory로 확장함.
	- 중간 frame을 RAFT가 만드는 것이 아니라, bounding box 좌표를 먼저 보간함.

### 3.2. VDM Serves as a Fine-Level Motion Synthesizer

- VLM trajectory는 coarse하고 부정확할 수 있음.
- VDM은 global planning은 약하지만 세부 motion 합성은 강함.
- 방법:
	- bounding box trajectory로 synthetic motion video 생성
	- [[RAFT]]로 인접 frame pair마다 optical flow 추출
		- 49 frame synthetic video라면 `frame 1 -> 2`, `2 -> 3`, ... 식의 flow sequence를 얻음.
		- 즉 RAFT가 긴 미래를 직접 예측하는 것이 아니라, 이미 만들어진 synthetic video의 frame 간 이동을 추정함.
	- optical flow를 structured noise `Q`로 변환
		- `Q`는 영상 픽셀이 아니라, motion 방향성이 들어간 diffusion noise representation
	- [[Go-with-the-Flow]] 기반 I2V diffusion model에 `Q`를 condition으로 입력
- noise injection은 training이 아니라 inference-time에 적용됨.
	- 최종 영상에 noise를 더하는 것이 아니라, condition인 `Q`를 Gaussian noise `ζ`와 섞는 것
	- 목적: VLM trajectory를 너무 강하게 따르지 않게 해서 VDM이 세부 motion을 보정할 여지를 줌
	- normalization은 noise scale이 diffusion model이 기대하는 분포에서 크게 벗어나지 않게 맞추는 역할
	- 짝수 frame: `gamma = 0.4`, 홀수 frame: `gamma = 0.6`
	- 홀수/짝수 `gamma`를 다르게 두는 이유는 논문에서 명확히 설명하지 않으며, empirical heuristic으로 보임.

### 핵심 요약

- VLIPP = VLM의 물리 기반 global planning + VDM의 local motion synthesis.
- 물리 법칙은 bounding box trajectory -> optical flow -> structured noise 형태로 VDM에 주입됨.
- structured noise는 motion guide이고, noise injection은 그 guide의 강도를 완화하는 장치.
