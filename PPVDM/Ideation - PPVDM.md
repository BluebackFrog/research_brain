

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
- ‘시간’을 이해 못하는 느낌. 거꾸로 동작을 수행하는 경우 꽤 많음
→ “물체 운동을 따라서 그림자를 정밀하게 만드는건 할 줄 아는데 공 하나를 사용자가 원하는 대로 못쏘는게 말이 되나?”
### 가설
매 timestep 마다 text token에 대한 attention이 계속 적용되어서 매 프레임마다 계속 text를 반영하려고 하는게 아닐까? 즉, 텍스트에 대한 이해도 또는 reasoning 능력이 좀 떨어지는 상태이기 때문에 text 에 담긴 시간적 정보를 잘 이해하지 못하는것 아닐까?
→ 실제로 Wan 아키텍쳐를 보면 LLM이 되게 안좋은게 달려 있음. 5B짜리.
- ~={red}프롬프트를 최대한 배제하고 벤치마크를 돌려보면 결과가 어떨지 궁금함.=~
~={red}	- 예를 들어 경사면에 공이 있는 상태를 initial frame으로 두고 텍스트 인풋은 안넣은 상태로 영상을 생성해본다면? 잘 만들까?=~
- LVLM한테만 텍스트 인풋을 넣고 VDM은 LVLM이 주는 visual info만을 바탕으로 생성한다면, (이 때 LVLM의 감독 하에 video를 조금씩 여러 개 생성해서 하나의 큰 비디오를 만드는 식으로 한다면) 더 잘하지 않을까?
	- LVLM이 planner + critic, VDM이 action policy