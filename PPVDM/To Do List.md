
### 0908
- ~~VideoTPO reasoning benchmark (VBVR bench) 일부 돌려보기 (https://arxiv.org/pdf/2602.20159)
	- 실험 결과 refine 되는지 확인
- ~~T2V to I2V 전환: 5-step method + refine ~~
	- 실험 돌리기
- ~~PhyT2V 코드 공유드리기~~


### 0910
- ~~5B 세팅하고 early exit 으로 5-step 에서 모션 정해지는지 보기
	- ~~괜찮으면 5B로 실험 진행
	- ~~실행 시간 남기도록 하기
- ~~VideoTPO 시드 1개 세팅 먼저 해보기
	- ~~시간 남으면 프레임 4장 세팅 해보기
		- 소희님께 결과 공유드리기



### 0929
- HunyuanVideo, Wan 해상도 이슈 최적화
- '점수 높으면 stop 해야한다' -> 기존 결과들도 필터링해서 통계 뽑아보기
- T2V -> I2V 작업한 데이터셋: 줌아웃시킨 사진 10장 정도만 정제해서 드리기 (ROI 줄이기)
- RAPO++ 구현

- baseline 실험 돌리기
	- Wan2.2-I2V-A14B, Wan2.1-I2V-14B, Wan2.2-TI2V-5B, HunyuanVideo-I2V
	- PhyT2V, VideoTPO, RAPO++, GPT Refinement 가 최종 refinement method
		- 데이터셋: physics-IQ, VBVR, (다른 I2V?)
			- videophy & phygenbench
	+ 실험 돌리면서 공유해주신 코드 이용해서 ROI 같이 뽑아보기

- early-stop filtering metric 다르게 해서 통계 다시 뽑아보기
- rapo++ 구현 더블체크
- rapo++ physics law 필요 여부 관련해서 논문에 기재된 내용 있는지 체크




## 1012
- ~~physics-iq verified 인지 체크
- phyground 세팅 및 실험
- ~~videotpo 재확인 (마지막에 비디오 선택하는 부분)
- evaluation 퀄리티 확인을 위해 좋은거 안좋은거 pair 5개 만들어보기
- 위에꺼 다 하면 VBVR 풀로 다 돌리기


## 1013
** 우리 아이디어
1. P0&V0 → P1&V1
2. V0와 V1으로 ROI 계산
3. “ROI 영역이 표시된 비디오 프레임들 + 프롬프트” history를 GPT input으로. 
	1. 잘된 프레임들은 통과 & freeze.
	2. 잘 안된 프레임들의 경우 우선 Region이 선택이 잘 되었는지 확인
	   ① 잘 되었다면 그 부분에만 refined 된 프롬프트를 적용
	   ② 잘 선택이 안되었다면 해당 프레임 전체에 refined 된 프롬프트를 적용

 
- phyground에 있는 physics-iq 샘플들만 evaluator 돌려보기
- iterative refinement 우리랑 비슷한거 찾아보기 (~목)
	- 현재 iteration만 본다
	- localize → VLM이 coordinate 내뱉도록 / SAM 등으로


## 1015
- iterative refinement 5 / local refine 3 추려서 10개 이내로 디테일하게 읽기
	- 관련된 논문의 related works 참고
- PhyGround 세팅 + PhyT2V&RAPO++ 실험

## 1019
- videorepair 를 VBVR 에 적용해서 안되는거 확인
	- iteration도 돌려볼 것
	- 100개 id 소희님이 알려주시면 그거 대상으로 돌릴 것.
	- cogvideox baseilne 실험 돌릴 것