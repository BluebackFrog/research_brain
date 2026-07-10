******Qwen2.5-VL MRoPE 찾아볼 것******

## Architecture
- LLM
- Vision Encoder
	- 2D-RoPE & interpolate absolute position embeddings based on input size
	  [[MRoPE-I]]
	- textual token-based time encoding
		- 원래 MRoPE에서 temporal position 정보를 absolute time 형태로 넣었음
		  근데 이러면 fps 변화에 적응하기 어렵고 position encoding 숫자가 커짐
		  그래서 그냥 텍스트 토큰으로 <00:00:00> 이렇게 넣어버림
- MLP-based Vision-Language Merger
	- [[DeepStack]] 메커니즘 사용
	- 기존 DeepStack을 변형: visual token을 ViT 중간 레이어에서 가져와버림

## Pretraining
- 생각보다 일반적인 LLM 학습에 쓰이는 reasoning 관련 데이터를 많이 씀. 난 vision 데이터만 많이 쓸 줄 알았는데 ‘V’LM이 아니라 V’LM’인 느낌

## Post-training
- SFT
- Strong-to-Weak Distillation
	- LLM backbone을 distillation 함
- Reinforcement Learning
	- SAPO(Smooth and Adaptive Policy-gradient Method) 사용