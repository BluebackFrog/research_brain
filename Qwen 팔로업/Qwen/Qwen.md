https://arxiv.org/pdf/2309.16609

### Pretraining
- Input embedding 이랑 마지막 output projection 을 공유하지 않음. 이게 더 성능 좋고 메모리 덜 씀
- 학습 안정성을 높이기 위해서 QKV 레이어를 제외하고는 ~={blue}**bias를 아예 빼버림**=~.
	- 큰 모델들에서는 bias 빼는게 오히려 안정성 높음. 큰 모델에서는 normalization, residual connection 등이 표현력을 충분히 제공 가능.
	- 근데 ~={blue}QKV 레이어에서는 RoPE 때문에 bias 쓰는게 더 좋음=~	  ![[Pasted image 20260629154807.png]]
	  따라서 긴 context 에서도 잘 동작하게 됨 (extrapolation 능력 높아짐)
- Pre-Norm with RMS Norm
	- Post-Norm vs Pre-Norm
	  ![[Pasted image 20260629155628.png]]
	  ![[Pasted image 20260629155640.png]]
	  Post-Norm 은 residual connection 도 LN 안에 포함되므로 gradient가 통째로 흔들림
	  ~={blue}Pre-Norm 에서는 residual connection이 LN 밖에 있으므로 gradient가 유지, 학습이 더 안정적.=~
	  ![[Pasted image 20260629160001.png]]
	  ![[Pasted image 20260629160008.png]]
	- RMS Norm
	  LayerNorm은 평균, 표준편차 계산해야 하는데 RMSNorm 은 아래 값만 구해서 나눠버림 (centering을 안하고 표준편차도 계산 안함)
	  ![[Pasted image 20260629160626.png]]
	- SwiGLU 사용
		- GeLU: smooth한 ReLU
		- GLU: 1) 정보를 2) 얼마나 gate 할지
		  ![[Pasted image 20260629161312.png]]
		- SwiGLU: GLU 변형 - sigmoid 대신 Swish gate 사용
		  ![[Pasted image 20260629161413.png]]
		  ![[Pasted image 20260629161432.png]]
		- 원래 FFN은 아래와 같이 구성됨
		  ![[Pasted image 20260629162636.png]]
		  (4d로 차원 넓히는건 feature를 늘리기 위함)
		  반면, SwiGLU에서는 gate가 있으므로 up projection이 두 개 필요해서 (W_1, W_2) 이렇게 함
		  ![[Pasted image 20260629163623.png]]
- Context Length Extension
	- pretraining은 2048 토큰으로 잘라서 학습 (비용 상 NTP 학습에 효율적이라서)
	- 더 긴 길이를 보았을 때 위치가 낯설지 않도록 Dynamic NTK-Interpolation 적용
		- Position Interpolation: p → p/4 이렇게 같은 비율로 줄임
		- NTK-Interpolation: RoPE의 base를 키워서 세밀한 위치 정보(작은 i)는 최대한 보존, 긴 거리 정보(큰 i)는 부드럽게 확장
		  [[RoPE]]
		  ![[Pasted image 20260629172818.png]]![[Pasted image 20260629175559.png]]
		- Dynamic NTK-Interpolation: 줄이는 scale을 고정하지 않음
	- LogN-Scaling
		![[Pasted image 20260629175907.png]]
		L: 학습 때의 context length, n: 현재 inference context length
	- Window Attention
	  너무 멀리 있는건 어텐션 계산 안해버림
	- lower layer에서 context length extension에 더 sensitive함을 발견해서, lower layer에 shorter window, higher layer에 longer window 적용

### Alignment
- SFT
- RLHF
	- alignment tax: 원래 base 모델이 갖고 있던 능력을 잃는 현상
	  → 이걸 pretrained gradient로 해결 (ppo gradient 에다가 pretraining LM의 gradient를 더함)
- Tool Use, Code Interpreter, and Agent
	- ReAct 방식의 툴콜링 학습할 때 general-purpose SFT sample 도 끼워 넣어서 general한 능력도 잃지 않게끔 함