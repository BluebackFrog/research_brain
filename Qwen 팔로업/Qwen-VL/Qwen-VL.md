## Architecture
- LLM (Qwen-7B) + Visual Encoder + Vision-Language Adapter
- Vision-Language Adapter 추가 정리
	- cross-attention layer 한 개 끼워넣는거임
	- 이 단계에서 시각 정보가 tokenize 됨 (grid로 나눠지고 각 patch 가 하나의 토큰)
	- 2D absolute positional encoding → q, k 에 각각 position 정보 추가
	  ~={red}Q. RoPE를 왜 안썼을까?=~
	  ~={red}GPT A. 이미지에서는 절대적인 위치가 중요하기 때문. 나중 세대 VLM에서 사용하는 경우도 있음.=~

## Inputs and Outputs
- Image input vs Text input 구분하기 위해 <img> ‘\<img>’ 스페셜 토큰 사용
- Bounding Box 에 대한 스페셜 토큰 사용
	- \<box>, \<ref> 이렇게 두 개 사용됨
	- \<ref>...\</ref>: 어떤 텍스트 표현/대상을 가리키는지 표시
	- \<box>...\</box>: 해당 대상의 bounding box 좌표 표시
	- 예시
	  ![[Pasted image 20260630143912.png]]

## Training
- Pretraining: LM 고정, vision encoder & VL adapter 학습
	- 이미지 넣고 text token 에 대해서 cross-entropy minimize. (next token prediction)
	  ~={red}Q. 이러면 진짜 LVLM이 이미지를 잘 참조하는건지 보장이 안되는거 아님?=~
- Multi-task pretraining: LM, vision encoder, VL adapter 다 학습
	- Tex-oriented task 를 향상시키기 위해 pdf, HTML 데이터와 synthetic OCR 데이터를 활용함
- Supervised Fine-tuning: vision encoder 고정, LM & VL adapter 학습
	- Instruction following 능력을 학습시킴.