## Optical flow 개념

- 정의
	- 연속된 두 이미지 사이에서 각 픽셀이 어디로 이동했는지를 나타내는 2D vector field
	- 각 픽셀마다 `(x 방향 이동, y 방향 이동)` 값을 가짐

- 다음 프레임 예측과의 차이
	- optical flow는 다음 프레임 이미지를 직접 생성하는 것이 아님
	- 현재 픽셀이 다음 프레임에서 어디로 이동했는지를 예측하는 것
	- 다음 프레임 예측은 이동 정보뿐 아니라 새로 보이는 영역, 가려짐, appearance 변화까지 생성해야 함

- 관계
	- optical flow는 다음 프레임 예측에 사용할 수 있는 중요한 motion 정보
	- 하지만 optical flow 자체는 이미지가 아니라 pixel-wise displacement map

## Method 핵심

- 문제 설정
	- 두 이미지 사이의 dense optical flow 예측
	- coarse-to-fine 방식 대신 하나의 flow field를 반복적으로 refinement

- 핵심 구조
	- 두 이미지에서 1/8 해상도 feature 추출
	- 모든 pixel pair 간 correlation을 계산해 4D correlation volume 생성
	- correlation volume을 pooling해 multi-scale correlation pyramid 구성

- Flow update
	- 초기 flow는 0으로 설정
	- 현재 flow가 가리키는 위치 주변의 correlation 값을 lookup
	- correlation feature, 현재 flow, context feature를 ConvGRU에 입력
	- ConvGRU가 `delta flow`를 예측하고 현재 flow에 반복적으로 더함

- Output / training
	- 1/8 해상도 flow를 learned upsampling으로 원본 해상도로 복원
	- 각 iteration의 flow prediction에 L1 loss 적용
	- 뒤쪽 iteration의 prediction에 더 큰 loss weight 부여

- 핵심 아이디어
	- 모든 matching 후보를 미리 만들어두고, recurrent update block이 optimizer처럼 flow를 점진적으로 개선
