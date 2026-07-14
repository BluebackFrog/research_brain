## Metrics for Physical Understanding

- 정적 카메라 영상에서 연속 frame 간 픽셀 변화가 threshold를 넘는 위치를 움직임으로 간주해, binary motion mask $M \in \{0, 1\}^{h \times w \times t}$를 만든다.
- 아래 네 metric은 생성 영상과 실제 test 영상의 움직임 및 외관을 서로 다른 관점에서 비교한다.

### Spatial IoU - Where does action happen?

- 시간축 $t$에 대해 max를 취해, 한 번이라도 움직인 pixel을 1로 둔 $h \times w$ binary motion map을 만든다.
- 생성 영상과 실제 영상에서 움직임이 발생한 위치의 overlap을 측정한다.
	- $\mathrm{Spatial\text{-}IoU} = \frac{|M_{\mathrm{real}}^{\mathrm{spatial}} \cap M_{\mathrm{gen}}^{\mathrm{spatial}}|}{|M_{\mathrm{real}}^{\mathrm{spatial}} \cup M_{\mathrm{gen}}^{\mathrm{spatial}}|}$
	- 도미노가 쓰러져야 하는 구간, 공이 지나가야 하는 경로처럼 action의 공간적 위치가 맞는지를 본다.
	- 언제 움직였는지는 무시하므로, 올바른 위치에서 너무 이르거나 늦게 움직여도 높은 점수를 받을 수 있다.

### Spatiotemporal IoU - Where and when does action happen?

- 시간축을 collapse하지 않고 $h \times w \times t$ motion mask 전체를 비교한다.
	- $\mathrm{Spatiotemporal\text{-}IoU} = \frac{|M_{\mathrm{real}} \cap M_{\mathrm{gen}}|}{|M_{\mathrm{real}} \cup M_{\mathrm{gen}}|}$
	- action이 올바른 위치에서 일어났는지뿐 아니라, 올바른 frame/시점에 일어났는지도 평가한다.
- Spatial IoU는 높지만 이 점수가 낮다면, 움직일 장소는 맞췄지만 타이밍을 틀린 경우다.

### Weighted Spatial IoU - Where and how much action happens?

- binary motion mask를 시간축에서 평균내어, 각 pixel이 전체 영상 중 얼마나 자주 움직였는지를 담은 weighted $h \times w$ motion map을 만든다.
	- $\mathrm{Weighted\text{-}spatial\text{-}IoU} = \frac{\sum_i \min(M_{\mathrm{real}, i}^{\mathrm{weighted, spatial}}, M_{\mathrm{gen}, i}^{\mathrm{weighted, spatial}})}{\sum_i \max(M_{\mathrm{real}, i}^{\mathrm{weighted, spatial}}, M_{\mathrm{gen}, i}^{\mathrm{weighted, spatial}})}$
	- 단순히 움직임의 위치만 보는 Spatial IoU와 달리, 그 위치에서 얼마나 많은 움직임이 있었는지도 본다.
	- 같은 영역을 반복해서 지나는 진자와 한 번만 지나가는 공을 구분할 수 있다.

### MSE - How does an action happen?

- 대응되는 실제/생성 frame의 모든 pixel 값 차이를 제곱해 평균낸다.
	- $\mathrm{MSE}(f_{\mathrm{real}}, f_{\mathrm{gen}}) = \frac{1}{n}\sum_{i=1}^{n}(f_{\mathrm{real}, i} - f_{\mathrm{gen}, i})^2$
	- 물체의 외관, 색, 세부 상호작용까지 pixel 수준으로 맞는지를 보는 엄격한 metric이며, 낮을수록 좋다.
- 앞선 motion 기반 세 metric이 놓칠 수 있는 시각적/물리적 불일치를 보완한다.

### Aggregation and Physical Variance

- 네 metric을 합쳐 Physics-IQ score를 만든다. MSE는 낮을수록 좋으므로 음의 부호로 더한다.
- `physical variance`는 동일한 scenario를 같은 조건에서 두 번 촬영한 실제 영상(`take 1`, `take 2`) 사이의 차이이다.
	- 마찰, 힘의 방향, 충돌 시점, chaotic motion 등의 미세한 변화로 실제 재촬영본도 픽셀 및 움직임이 완전히 일치하지 않는다.
- 이 real-vs-real 비교의 종합 점수가 `100%`가 되도록 정규화한다.
	- `100%`는 ground-truth와 완전히 같은 영상을 만들었다는 뜻이 아니라, 독립적으로 다시 일어난 실제 물리 과정과 같은 수준으로 일치한다는 기준선이다.
	- 따라서 최고 모델의 `29.5%`는 픽셀의 29.5%를 맞췄다는 의미가 아니라, real-vs-real 기준선에 한참 못 미친다는 뜻이다.
