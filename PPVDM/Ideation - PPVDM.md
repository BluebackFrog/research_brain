

Papers: [[Paper List - PPVDM]]


## Old Idea
- Wan technical report 보니까 강화학습 이런거 아예 안해놨음. 그냥 ‘데이터를 잘 선택했음’이 끝.
- 특히 비디오 데이터는 12개의 major category를 정해놓고 visual/motion quality classifier로 필터링했는데, 아무리 많고 다양한 데이터를 모아 놓았다고 해도 physics 관련 빈약한 카테고리 (누락된? 카테고리) 생길 수밖에 없을듯
  → 카테고리: technology, animals, arts, humans, vehicles, and etc


- RLHF는 어떨까…

- Superpixel / Object Segmentation 을 이용하면 어떨까?


- Blurry image에 속도 정보가 있음을 이용하면 뭔가 할 수 있지 않을까?
	- 아닌가 이건 그냥 모델이 자연스럽게 이해하고 있으려나…
	- 실제 motion 영상 보고 blurred image 를 통해 다음 프레임에 object가 어디로 움직일지 예측하도록 fine-tuning? ~={red}(아마 선행연구 있을듯. 찾아보기)=~
	- + 무게중심을 point로 나타낸다?

- Physical (Natural) motion이랑 intended motion(?)이랑 구분되는 개념이지 않을까
	- ex) 어떤 남자가 왼쪽 방향으로 걸어간다 → 사용자가 프롬프트에 반드시 기재해야 나타낼 수 있는 모션임
	- ex’) 접시가 시멘트 바닥에 떨어진다 → 접시가 깨져서 흩어질텐데 이건 프롬프트에 나타나지 않아도 추론해서 비디오에 반영하는 것이 가능해야 함
	- → high-level planning 은 세세한 것에 대해 적용되면 안됨 (접시 파편이 어떻게 움직이는지는 X, ‘접시가 떨어져 바닥에 깨진다’ 정도의 플래닝)

1. real-world 비디오를 통해서 사전학습되었기 때문에 video generation model 은 natural physical interaction 에 대해 어느 정도의 선행 지식이 분명히 있을 것임.
	- 예를 들어 접시 깨지는거 어디선가 봤을거임.
2. (가설) 근데 물리학적인 모션에 대해 실수를 하는 이유는 프롬프트/모션에 biased 되어 있기 때문일 것 같음.
   → 2-1. 프롬프트를 최소화했을 때 비디오 vs 구체적인 프롬프트가 있을 때의 비디오 이렇게 비교해본다면?
   → 2-2. 기존의 motion-controlled video gen 메소드를 적용할 때 motion 한 개가 아니라 여러 개 있는 상황을 만든다면?
	   - 예를 들어 접시가 떨어질 때 도미노가 쓰러지고 있는 상황, 이 때 프롬프트를 접시에 대해서만 준다면 뒤에 도미노 쓰러지는건 많이 무너질 것 같음
	→ 즉, ‘physically plausible’-ness 에 대한 pretrained knowledge를 측정할 필요가 있음
3. method
	1. 기존 method + bias 끊어버리기
	2.  VLM의 정보와 함께 high level planning 에 해당하는 주요 이미지를 몇 프레임 정도 먼저 만들고 나머지 프레임들을 high level planning 을 이어주도록 만듦.

### VideoGen 모델을 world model / VLA로 사용하는 논문들 추가로 찾아볼 것!!!
- Cosmos?


## 0706

### 현재까지 계획된 pipeline
1. VLM이 object info 생성
2. 1 바탕으로 bounding box 생성
3. 2 바탕으로 object segmentation
4. 3 바탕으로 point trajectory 뽑아내기
   ======= 여기까지 구현 완료 =======
5. point trajectory 를 어떠한 벡터로 변환
6. VLM이 벡터값을 예측하도록 학습시키기
7. inference에서는 VDM이 motion vector 값을 활용
### pipeline에 대한 아이디어들…
1. point trajectory → vector 변환
   - point trajectory는 너무 복잡한데, 이걸 더 간략하게 만들 수는 없을까?
	   - 물체의 ‘운동’은 생각보다 몇 종류 없을지도…
	     ~={red}→ 운동의 분류에 대한 reference 찾아볼 것=~
	       ~={cyan}: 이거 WISA에서 17가지 카테고리로 분류함=~
   - 간략하게 나타내기가 힘들다면… 그냥 Embedding 하나로 때우기?
     → point trajectory 에 정보가 충분한지 고민할 것
2. point trajectory 나타내는 방법
   - 3D point trajectory: Depth 정보를 섞어서 저장
   - Crop & resizing: point trajectory 바깥에 있는 쓸데없는 정보들을 배제
     - 운동은 길이, 속도 등이 중요한데 resizing 해도 되나? 심지어 중력은 어캄?
       → 속도는 point 간의 간격이 오밀조밀한 정도에 따라 자연스럽게 표기될 것.
       → 운동은 어차피 상대적인 것이고 프롬프트에 따라 달라지기 쉬움. ~={red}~={red}속도나 길이에 대한 부분에서 채점이 어떻게 되는지 확인해보기 (ex. 공이 맞는 방향으로 굴러가는데 GT와 비교해서 공 속도가 너무 느리다면? 그런데 프롬프트에 속도에 대한 내용이 없다면? 이건 정답일까 오답일까? 합리적인 pre-defined rubric이 있는 것일까?)=~=~
     - LVLM이 reference image를 보고 anchor object를 생성하도록 하면 어떨까? 크기 비교를 위해 물체 옆에 동전을 놓고 사진을 찍는 것처럼?
       → static anchor object 말고 dynamic anchor object를 만들도록 한다면? 예를 들어 5cm짜리 물체가 1초에 10cm 이동하도록 항상 표기하는 것임 (실현 가능성 엄청 떨어지긴 함 하하)
### video data 50개 검수해본 결과
- 은근히 공간적으로 ‘visual’한 부분은 되게 잘한다.
	- 물체가 지나가도 바닥 무늬가 유지됨
	- 그림자 품질이 굉장히 좋음
	- 풍선 팽창시키는거 되게 잘함 (37 38 39)
	- ‘한 눈에 봐도 어색하다’ 싶은건 거의 없음
- 텍스트 프롬프트를 엄청 이상하게 이해하는 느낌 (특정 단어에 꽂히는 느낌)
	- ex1) 공을 굴린다 (roll) 라는 표현 → 공 하나가 나머지 공 두 개를 빙글빙글 돌아버림
	- ex2) 왼쪽, 오른쪽 이런 방향에 대한 단어 → 기재한 방향대로 object들이 날뜀
	- ex3) “The platform rotates clockwise and the wooden stick hits the first block as it rotates” → 이쑤시개가 아니라 검정색 platform이 돌아버림 ㅋㅋㅋㅋㅋ 근데 이렇게 이해하는게 말은 됨….
	- ~={red}+ “text input 과 관련되지 않은 부분에서 physically corrupted 되는 경우를 못봤음”=~
- ‘시간’을 이해 못하는 느낌. 거꾸로 동작을 수행하는 경우 꽤 많음
→ “물체 운동을 따라서 그림자를 정밀하게 만드는건 할 줄 아는데 공 하나를 사용자가 원하는 대로 못쏘는게 말이 되나?”
### 가설
매 timestep 마다 text token에 대한 attention이 계속 적용되어서 매 프레임마다 계속 text를 반영하려고 하는게 아닐까? 즉, 텍스트에 대한 이해도 또는 reasoning 능력이 좀 떨어지는 상태이기 때문에 text 에 담긴 시간적 정보를 잘 이해하지 못하는것 아닐까?
→ 실제로 Wan 아키텍쳐를 보면 LLM이 되게 안좋은게 달려 있음. 5B짜리.
- ~={red}프롬프트를 최대한 배제하고 벤치마크를 돌려보면 결과가 어떨지 궁금함.=~
~={red}	- 예를 들어 경사면에 공이 있는 상태를 initial frame으로 두고 텍스트 인풋은 안넣은 상태로 영상을 생성해본다면? 잘 만들까?=~
- LVLM한테만 텍스트 인풋을 넣고 VDM은 LVLM이 주는 visual info만을 바탕으로 생성한다면, (이 때 LVLM의 감독 하에 video를 조금씩 여러 개 생성해서 하나의 큰 비디오를 만드는 식으로 한다면) 더 잘하지 않을까?
	- LVLM이 planner + critic, VDM이 action policy



포괄하는 텍스트 + chunk 단위 텍스트 두 개 사용하는 연구 있음

LVLM을 VDM에 아예 통합시키는 연구

텍스트 안쓰는 벤치마크 찾아보기
+ I2V 연구 중에서 텍스트 안넣는 연구도 있음



## 0728

- Interaction을 어떻게 정의해야 할까?
- GT-Generated 간의 event graph 비교에 대한 생각들…
	- GT video 라고 해서 모든 가능한 interaction을 담고 있는 것이 아님.
	  ![[Pasted image 20260728143322.png]]
		→ A = GT, B = Generated 일 때, “non-deterministic”함을 나타내는 $B - A$ 에 대해서 어떻게 reward를 줄 수 있는가? 결국 world knowledge가 필요하지 않나?
		→ $B-A$ 비율이 많지 않을 것으로 기대하고 그냥 무시하면 되려나? GT event graph의 내용이 사용되는 경우만 시그널이 들어가도록?
- Event graph를 써먹는 구체적인 방법?
	- ex) 공이랑 rubber ducky 랑 부딪치는 상황 → “rubber ducky는 잘 안밀려난다”는 knowledge가 event graph에 저장됨. 어디로 공이 튀든 간에 잘 안밀리게 생성하기만 하면 physically plausible.
	- 문제점 1: knowledge를 어떻게 뽑을 것인가?
		- rule-based (현재): event 는 구분할 수 있어도 이걸로부터 general한 팩트를 담기가 어려울 것 같음
		- LVLM??
	- 문제점 2: knowledge를 어떤 형태로 저장할 것인가?
	- 문제점 3: 저장된 knowledge를 어떻게 reward 로 만들 것인가?
- 핵심 IDEA: “현실적인 physical plausibility는 deterministic하지 않다”


## 0730

I2V 관련 쓸만해보이는 insights
- MotiF: VDM이 학습할 때는 배경만 똑같이 만들어도 loss가 많이 줄어든다.
- AlignVid: 이미지를 블러처리하면 이미지 조건이 약해지고 텍스트에 어텐션이 비교적 더 많이 걸려져서 객체 수준 지시를 더 잘 따른다. (ex. 없는 object 만들라고 하는 지시를 더 잘 따름)
- Focal Guidance: 텍스트 조건을 잘 따르지 않는 semantic-weak layers 가 존재한다.

I2V object 관련 벤치마크
- UI2V-Bench: object를 정확히 찾아서 모션을 부여해야 하는 벤치마크
- Omit2V: 객체 수준 지시를 평가 (텍스트 프롬프트를 얼마나 잘 따르는가)





## 0806

- seed를 여러 개 돌리고 가장 모션이 비슷한 것들의 군집을 보고 거기에서 한 개를 뽑으면 이게 가장 plausible 하지 않을까?
	- method: 
	  1) confidence(ANSE 참고) & latent lev similarity(GEOPHYS 참고 또는 PCA)
	  2) DisMo motion embedding
	- 전제가 되는 idea: physically implausible 하면 모션의 variance가 매우 클 것임. 즉 implausible할수록 비슷한 motion이 모일 가능성이 낮아진다.
	- seed를 고르는게 중요하다는건 Related work 에서 보일 수 있으므로 정당성 확보되지 않나?

- + 추가적인 생각: 이걸 reward로 주고 강화학습을 돌리면 안좋은 seed여도 좋게 생성하는 능력이 생길 수도 있을까?
	- metric: ~~seed들간의 variance~~ 다양한 시드로 실험 여러 번 돌렸을 때 벤치마크 점수 평균

- 고려해야 할 점: VDM의 장점인 ‘다양성’은 해치지 않아야 할텐데…
	- 예를 들어 접시가 깨지는 영상을 만든다고 하면 다양한 방향으로 깨져도 다 Plausible 하다고 봐야 하는데 이걸 비슷한 motion으로 감지할 수 있을까? 물론 완전히 이상한 영상은 구분할 수 있겠지만… 어떤건 깨지고 어떤건 안깨진다고 할 때 안깨지는 영상이 (또는 좀 적게 깨지는 영상이) 무조건 implausible 할까?
	- DisMo가 이런거 잡아주지 않을까? 어찌 되었든 physically plausible하면 비슷한 동작이 나오는거잖아

- instruction following 능력을 어떻게든 집어넣을 수는 없을까?
	- 예를 들어 cross-attention 에서의 confidence?


## 0821

- 초반, 중반, 후반 레이어의 역할은 모든 denoising step 에서 동일할 것임.
	- 어느 정도 퀄리티의 noise 가 input으로 주어졌느냐가 다르고 어쨌든 간에 마지막에는 velocity 를 내뱉음
	- 그렇다면 아무 노이즈나 줘도 후반 레이어 태우면 visible한 정보가 나오지 않을까?

생각해보니까 velocity가 바뀐다고 무조건 모션 정보가 들어 있는 것은 아닌데… 예를 들어 모션이 너무 confident 하면 velocity가 확 바뀔 이유가 없음


temporal 한 디테일이 physically plausible prompt 의 가장 큰 요소인가?
→ 만약 그렇다면 CFG를 temporal 하게 적용하는 등의 시도가 가능하다


CFG good vs very good

그냥 비디오 / 프레임 샘플링 최적 으로 줄 때 GPT vs Qwen
→ 프롬프트를 잘 깎는지랑 실패 detect 잘 하는지

## 0824
### Insights
- qwen vs GPT 실험: 전체적인 움직임이 이상한 것들은 둘 다 잘 detect 함. 공 속도가 자연스러운지 이상한지 (운동량 관련 추론)도 잘 함. 그런데 미세하거나 급격한 움직임의 경우에는 GPT도 잘 판별하지 못함. 
	- Qwen: 공과 오리가 가만히 멈췄는데도 공이 계속 앞으로 간다고 함
	- blur 처리, 공의 흰색 문양 등을 공의 형태가 바뀌거나 겹친걸로 오인함

### To-Do
- baseline vs 3-way CFG physics-IQ & videophy (gpt 사용) 스코어 뽑아서 비교
	- 시간 남으면 수식 깎아보기
- Adaptive Sampling 해줄 때 phy plau 잘 detect?: energy state가 높은 (부딪히는 순간) 부분은 구체적으로 주기, 대신 프레임 넘버 정보도 같이 줘야 안헷갈릴듯. energy state 가 낮은 부분은 띄엄띄엄 주기
	- gt video 안주고 할 것
	- energy state 높/낮: 직접 구간 정보를 주기
## 0825

LVLM Eval 결과: 물체의 물리학적 특성에 대한 이해도가 되게 많이 떨어짐.
	- ex) 도미노에 대한 bias: “중간에 물체가 있어도 무조건 도미노는 다 쓰러져야 함”
	- ex2) “고무 오리 쪽으로 도미노가 쓰러지면 오리는 무조건 조금이라도 움직여야 함” (무게와 마찰력에 대한 센스가 전혀 없음)
	- ex3) 중간에 물체가 새로 생겨버려도 이게 implausible하다는걸 잘 detect 못함

결국 인간의 지식을 어떻게든 distill 해줘야 할 것 같음.
- LVLM 단독으로는 정확한 evaluation이 사실상 불가능한 느낌.
	- physical rule에 대한 센스가 전혀 없어서 이 physical rule이 자연어로 주어진다고 한들 제대로 eval이 안될 것 같음
	- 이상한 Bias들이 너무 많음
- I2V 데이터셋을 기반으로, 가능할만한 물체 간 ‘상호작용’에 대한 캡션을 LVLM으로 최대한 많이 뽑고 인간이 걍 annotate 해서 데이터셋 만든 다음에 그거 바탕으로 LVLM이 생성된 영상에 리워드를 준다면? 그리고 그걸로 학습을 한다면 더 좋아질까?
	- Motivation: ‘LVLM이 원인과 결과를 모두 예측하는건 부정확하나, likely한 상호작용 정도는 쉽게 뽑아낼 것이다’
	- ex) 오리 도미노 샘플:
	  1) LVLM이 ‘도미노가 넘어지고 오리와 부딪힌다’는 상호작용 캡션 뽑음
	  2) 인간이 그 캡션 보고 말 되는거 아무거나 annotate - “도미노가 넘어져도 오리는 안움직인다”
	  3) VDM이 생성한 영상을 평가할 때 LVLM은 2)를 보고 채점 + 강화학습
### To-Do
- 샘플링, 프롬프트 최적화 방법 찾아보고 고안해보기
- 실패 영상도 캡셔닝 시켜볼 것
- LVLM Crop tool 줘보기



## 0826

- GPT에 넣을 프롬프트 수정 방안
	- object에 대한 묘사나 쓸데없는 디테일들을 포함하지 않도록 수정
	- implausible 한 
- physics가 틀린 부분을 잘 해결하도록 프롬프트를 agentic하게 깎는 방식 구축


## 0831

### To Do
- PhyT2V 적용해서 프롬프트 여러 번 refine 하는 방식 해보기
	- 이런 류의 연구들이 더 있는지 확인해보기
- https://arxiv.org/pdf/2507.13428 PhyWorldBench 확인해보기

### To Do
- VLIPP 참고해서 T2V 세팅 → image generation 한 장 + I2V” 구현해보기
	- videophy
	- ~~phygenbench~~
	- 구현하고 phyt2v 등등 해보기
- 논문에서 사용된 벤치마크로 기존 메소드 reproducible 한지 확인
- ~~videotpo 구현~~


## 1007

- 괜찮은 프레임들을 LVLM이 앞에서부터 고르는걸 추가?
	- 보통 실수는 뒤쪽에서 심해지기 때문에 앞쪽부터 괜찮다고 판단이 되면 고정시키고 그걸 그냥 condition으로 줘버린다는 아이디어

- 괜찮은 region을 LVLM이 고르는걸 추가?
	- 이렇게 되면 LVLM->ROI 가 맞을까 ROI->LVLM이 맞을까

