# VLM을 사용하지 않는 Video Generation Metric

## 일반 자동 계산 Metric

| 연구·벤치마크          | 자동 계산 Metric                     | 평가 대상                                   | 비교 기준                           | GT Target Video 필요 | Motion Change 평가 적합성         |
| ---------------- | -------------------------------- | --------------------------------------- | ------------------------------- | -----------------: | ---------------------------- |
| **Goku-Bench**   | FVD                              | 생성 영상 집합의 시각 품질 및 temporal distribution | 생성 영상 집합 ↔ 실제·GT 영상 집합          |                 필요 | 중간                           |
| **Goku-Bench**   | Background Consistency, BC       | 프레임 간 배경 일관성                            | 출력 영상 내부 프레임                    |                불필요 | 보조 지표                        |
| **Goku-Bench**   | Temporal Consistency, TC         | 프레임 간 시간적 안정성                           | 출력 영상 내부 프레임                    |                불필요 | 높음                           |
| **Goku-Bench**   | Motion Smoothness, MS            | 움직임의 부드러움                               | 출력 영상의 연속 프레임                   |                불필요 | 높음                           |
| **Goku-Bench**   | CLIP Similarity                  | 텍스트 지시와 출력 영상의 의미적 일치                   | 출력 영상 ↔ 편집 프롬프트                 |                불필요 | 중간                           |
| **Goku-Bench**   | DINO Style Similarity            | 목표 스타일과 출력의 유사도                         | 출력 영상 ↔ 스타일 참조                  |                조건부 | 낮음                           |
| **Goku-Bench**   | Optical-flow Camera Motion Score | 요구한 카메라 움직임의 수행 여부                      | 출력 optical flow ↔ 목표 카메라 motion |                조건부 | 객체 motion에는 제한적              |
| **FiVE-Bench**   | PSNR                             | 비편집 영역의 픽셀 보존                           | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **FiVE-Bench**   | MSE                              | 비편집 영역의 픽셀 오차                           | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **FiVE-Bench**   | SSIM                             | 비편집 영역의 구조 보존                           | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **FiVE-Bench**   | LPIPS                            | 비편집 영역의 perceptual 변화                   | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **FiVE-Bench**   | Structure Distance               | 전체 또는 비편집 영역의 구조 변화                     | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **FiVE-Bench**   | Motion Fidelity Score            | 원본 영상의 움직임 보존                           | 출력 motion ↔ 원본 motion           |                불필요 | **Motion 유지에는 적합, 변경에는 부적합** |
| **FiVE-Bench**   | CLIP Score                       | 목표 객체·텍스트와의 의미적 일치                      | 출력 ↔ 프롬프트                       |                불필요 | 중간                           |
| **FiVE-Bench**   | NIQE                             | 비참조 영상 품질                               | 출력 영상 단독                        |                불필요 | 보조 지표                        |
| **IVEBench**     | DINO Subject Consistency         | 객체 identity의 프레임 간 유지                   | 출력 영상 내부 프레임                    |                불필요 | 높음                           |
| **IVEBench**     | CLIP Background Consistency      | 배경의 프레임 간 일관성                           | 출력 영상 내부 프레임                    |                불필요 | 보조 지표                        |
| **IVEBench**     | Frame Difference                 | flicker 및 시간적 불안정성                      | 출력의 인접 프레임                      |                불필요 | 높음                           |
| **IVEBench**     | VideoCLIP Similarity             | 원본 영상과 출력의 의미적 유사도                      | 출력 ↔ 원본 영상                      |                불필요 | motion 변경에서는 주의              |
| **IVEBench**     | Grounding-DINO Object Count      | 요구한 객체 수의 정확성                           | 출력 검출 결과 ↔ 목표 개수                |                불필요 | 객체 추가·삭제에 적합                 |
| **IVEBench**     | CoTracker Trajectory Similarity  | 객체의 위치·속도 trajectory 보존                 | 출력 trajectory ↔ 원본 trajectory   |                불필요 | **Motion 유지에는 적합, 변경에는 부적합** |
| **V2V-Bench**    | DINO + SSIM Frame Correspondence | 원본과 출력의 프레임별 구조 대응                      | 출력 ↔ 원본 영상                      |                불필요 | motion 변경에서는 제한적             |
| **V2V-Bench**    | Optical-flow Relative EPE        | 원본 motion의 보존 정도                        | 출력 flow ↔ 원본 flow               |                불필요 | **Motion 변경에는 부적합**          |
| **V2V-Bench**    | Motion Smoothness                | optical-flow 변화의 부드러움                   | 출력 영상 내부 프레임                    |                불필요 | 높음                           |
| **V2V-Bench**    | Canny Edge F1                    | 구조 및 윤곽 보존                              | 출력 edge ↔ 원본 edge               |                불필요 | 객체 교체에서는 제한적                 |
| **V2V-Bench**    | Layout SSIM                      | 장면 layout 보존                            | 출력 ↔ 원본 영상                      |                불필요 | 객체 교체에 적합                    |
| **V2V-Bench**    | CLIP Edit Faithfulness           | 텍스트 편집 지시 수행 정도                         | 출력 ↔ 프롬프트                       |                불필요 | 중간                           |
| **V2V-Bench**    | VGG Gram Distance                | 스타일 유사성                                 | 출력 ↔ 스타일 참조                     |                조건부 | 낮음                           |
| **V2V-Bench**    | BRISQUE                          | 비참조 영상 품질                               | 출력 영상 단독                        |                불필요 | 보조 지표                        |
| **V2V-Bench**    | RGB Histogram Similarity         | 전체적인 색상·콘텐츠 보존                          | 출력 ↔ 원본 영상                      |                불필요 | 보조 지표                        |
| **RefVIE-Bench** | 공식적인 주요 수식 기반 metric 거의 없음       | 주로 reference fidelity 및 편집 품질           | VLM judge 중심                    |                  — | —                            |
| **TIDE-Bench**   | 공식적인 주요 수식 기반 metric 거의 없음       | multi-reference object editing          | VLM judge 중심                    |                  — | —                            |
| **OpenVE-Bench** | 공식적인 주요 수식 기반 metric 거의 없음       | instruction compliance 및 안정성            | VLM judge 중심                    |                  — | —                            |
| **VEFX-Bench**   | VEFX-Reward                      | 편집 결과의 종합 품질                            | 학습된 reward model                |                불필요 | 수학적 고정 metric은 아님            |

## Mask-aware Metric

Mask-aware metric은 segmentation mask 또는 motion mask를 이용해 객체와 배경을 분리한 뒤, 특정 영역만 비교하거나 마스크에서 trajectory·형상·물리량을 추출하는 자동 계산 지표를 의미한다.

### 마스크 픽셀 또는 마스크 영역을 직접 점수에 사용하는 Metric

| 연구·벤치마크 | Metric | 계산 방식 | 주로 평가하는 것 | GT/reference video 필요 |
| --- | --- | --- | --- | ---: |
| [WorldBench](https://arxiv.org/abs/2601.21282) | **Foreground mIoU** | 생성 영상에서 SAM2로 얻은 객체 마스크와 GT 객체 마스크의 IoU를 객체·프레임·영상에 걸쳐 평균 | 객체 위치, 형상, 출현 여부 및 reference dynamics 재현 | 필요 |
| WorldBench | **Background RMSE** | GT background mask가 1인 픽셀에서만 생성 영상과 GT 영상의 RMSE 계산 | 배경 변형, 카메라 이동, 새로운 객체 출현 | 필요 |
| [PISA Experiments](https://arxiv.org/abs/2503.09595) | **Mask IoU** | 프레임별 낙하 물체의 생성 마스크와 GT 마스크 사이 $\lvert M_t^{gen}\cap M_t^{gt}\rvert/\lvert M_t^{gen}\cup M_t^{gt}\rvert$를 평균 | 객체 permanence와 공간적 일치 | 필요 |
| PISA Experiments | **Mask Chamfer Distance** | 생성·GT 마스크 영역을 점 집합으로 간주하고 양방향 최근접 거리 평균 계산 | 객체 위치와 형상 차이 | 필요 |
| [Physics-IQ](https://arxiv.org/abs/2501.09038) | **Spatial IoU** | 프레임 간 픽셀 변화량을 thresholding해 $H\times W\times T$ motion mask를 만들고, 시간축 max pooling 후 GT motion map과 IoU 계산 | 영상 전체에서 움직임이 발생한 위치 | 필요 |
| Physics-IQ | **Spatiotemporal IoU** | 생성·GT motion-mask volume의 IoU 계산 | 움직임이 발생한 위치와 시점 | 필요 |
| Physics-IQ | **Weighted Spatial IoU** | 각 픽셀에서 움직임이 발생한 프레임 비율을 누적한 activity map을 만들고, pixel-wise minimum의 합을 maximum의 합으로 나눔 | 움직임의 위치와 반복량 | 필요 |
| [NewtonBench-60K](https://arxiv.org/abs/2512.00425) | **Mask IoU / Chamfer Distance** | Renderer instance mask와 SAM2 생성 마스크의 프레임별 overlap 및 형상 거리 계산 | 자유낙하·포물선·경사면 운동의 공간적 재현 | 필요 |
| [MagicBench](https://arxiv.org/abs/2503.16421) | **Mask IoU** | GT 첫 프레임 마스크로 SAM2를 초기화해 생성 영상의 객체 마스크를 추적하고, 주어진 mask trajectory와 프레임별 IoU 계산 | 단일·다중 객체 trajectory control 정확도 | 필요 |
| [Cosmos-Transfer1 / TransferBench](https://arxiv.org/abs/2503.14492) | **Mask mIoU** | 조건 영상과 생성 영상에서 GroundingDINO+SAM2 마스크를 얻고, 같은 caption phrase에 해당하는 마스크를 합친 후 IoU 기반 matching | Segmentation condition 준수 | 필요 |
| Cosmos-Transfer1 / TransferBench | **FG Mask mIoU** | 전체 Mask mIoU와 동일한 계산을 salient foreground 객체에만 적용 | 배경·도메인이 바뀔 때 핵심 객체의 형태와 위치 보존 | 필요 |
| Cosmos-Transfer1 / TransferBench | **Lane mIoU** | 생성 영상에서 추출한 차선 영역과 HD-map 조건의 차선 영역 사이 mIoU 계산 | 자율주행 장면의 차선 layout 준수 | 필요 |
| [Lumen](https://arxiv.org/abs/2508.12945) | **Intrinsic Consistency: Masked PSNR / SSIM / LPIPS** | $sim(U(V_{src})\odot M_{fg}, U(V_{gen})\odot M_{fg})$를 계산. $U$는 조명 효과를 제거하는 uniform-lit restorer | Relighting·배경 교체 후 foreground albedo와 texture 보존 | 불필요 |
| [Object-WIPER / WIPER-Bench](https://arxiv.org/abs/2601.06391) | **BG-PSNR** | 제거 대상 마스크 밖의 배경 영역에서만 input/output PSNR 계산 | 객체 제거 과정에서 비편집 배경 보존 | 불필요 |
| Object-WIPER / WIPER-Bench | **FG-Flicker** | 인접 프레임 객체 마스크의 합집합 영역에서만 프레임 간 $L_1$ 차이 계산 | 제거·인페인팅 영역의 시간적 깜빡임 | 불필요 |
| Object-WIPER / WIPER-Bench | **TokSim** | Masked DINOv3 token에 대해 $100\operatorname{mean}[\lambda(1-\eta)\tau]$ 계산. $\lambda$는 인접 프레임 일관성, $\eta$는 원본 객체와의 유사도, $\tau$는 주변 배경과의 유사도 | 객체가 실제로 제거되고 빈 영역이 배경과 자연스럽게 연결되는지 | 불필요 |
| [CRONOS](https://arxiv.org/abs/2605.23699) | **Object/Appearance Stability** | SAM3 객체 마스크로 배경을 제거한 뒤, 각 객체의 DINOv2 특징과 첫 프레임 특징 사이 cosine similarity의 robust minimum 계산 | 객체 identity·외형의 일시적 붕괴 | 불필요 |
| CRONOS | **Background Stability** | 현재·첫 프레임 background mask의 교집합에서 MSE를 계산하고 시간축 robust maximum으로 집계한 뒤 지수 함수로 $[0,1]$ 범위에 scaling | 배경 morphing, lighting drift, 카메라 이동, 새로운 객체 출현 | 불필요 |

### 마스크에서 Trajectory·형상·물리량을 추출하는 Metric

| 연구·벤치마크 | Metric | 마스크의 역할 및 계산 방식 | 주로 평가하는 것 | GT/reference video 필요 |
| --- | --- | --- | --- | ---: |
| [PISA Experiments](https://arxiv.org/abs/2503.09595) | **Trajectory $L_2$** | 각 마스크의 centroid $c_t$를 구한 뒤 $\frac{1}{T}\sum_t\lVert c_t^{gen}-c_t^{gt}\rVert_2$ 계산 | 낙하 trajectory의 위치 오차 | 필요 |
| [NewtonBench-60K](https://arxiv.org/abs/2512.00425) | **Trajectory Position Error** | SAM2 생성 마스크와 renderer GT 마스크에서 centroid trajectory를 추출하고 평균 $L_2$ 거리 계산 | 생성 운동의 위치 오차 | 필요 |
| NewtonBench-60K | **Velocity RMSE** | $v_t=(c_{t+1}-c_t)/\Delta t$를 계산해 생성/GT 속도 사이 RMSE 측정 | 운동 방향과 속도 변화 | 필요 |
| NewtonBench-60K | **Acceleration RMSE** | $a_t=(c_{t+2}-2c_{t+1}+c_t)/\Delta t^2$를 계산해 생성/GT 가속도 사이 RMSE 측정 | 일정 가속도 운동 및 중력 운동 재현 | 필요 |
| [Morpheus](https://arxiv.org/abs/2504.02918) | **Dynamical Score** | SAM2 mask centroid trajectory를 운동방정식이 loss에 포함된 PINN에 fitting하고 trajectory NMSE를 $0$–$1$ 점수로 변환 | GT trajectory와 정확히 겹치지 않아도 해당 운동방정식을 따르는지 | 불필요 |
| Morpheus | **Physical Invariance Score** | Mask trajectory에서 속도·가속도와 에너지·운동량·각운동량 등을 계산하고, 보존량 시계열의 표준편차가 작을수록 높은 점수 부여 | 에너지·운동량 등 물리적 보존 법칙 | 불필요 |
| [VAMP](https://arxiv.org/abs/2411.13609) | **Masked Appearance Score** | SAM2 객체 마스크 내부에서 RGB histogram EMD, contour Hausdorff Distance, GLCM texture cosine similarity 계산 | 객체의 색상·형상·질감 일관성 | 불필요 |
| VAMP | **Velocity / Acceleration Consistency** | Mask centroid로 $v_t=\lVert c_{t+1}-c_t\rVert$와 $a_t=v_{t+1}-v_t$를 계산하고 $S_{vel}=\exp[-\operatorname{std}(v)/\operatorname{mean}(v)]$, $S_{acc}=\exp[-\operatorname{Var}(a)]$ 계산 | 움직임의 매끄러움과 시간적 일관성 | 불필요 |
| [CRONOS](https://arxiv.org/abs/2605.23699) | **3D Shape Stability** | 객체 마스크로 SAM3D reconstruction을 수행하고, 정렬된 프레임별 mesh와 첫 프레임 mesh 사이 Chamfer Distance 계산 | 객체의 3D morphing 및 구조 붕괴 | 불필요 |

### 적용 시 주의점

| 문제 | 영향을 받는 Metric | 설명 |
| --- | --- | --- |
| Physically valid alternative | GT Mask IoU, Trajectory $L_2$, Spatiotemporal IoU | Reference와 다른 결과가 물리적으로 타당해도 pixel/trajectory 불일치로 낮은 점수를 받을 수 있음 |
| 카메라 이동·원근 변화 | Motion mask, centroid trajectory, velocity·acceleration | 객체 운동과 카메라 운동이 섞이며, image-plane 가속도가 실제 물리 가속도와 달라질 수 있음 |
| Segmentation 오류 | 모든 mask-aware metric | SAM2·SAM3 등이 객체를 놓치거나 다른 객체를 포함하면 metric 오차가 물리 오차로 해석됨 |
| 그림자·조명·유체 변화 | Physics-IQ motion-mask metric | 단순 픽셀 변화 mask는 객체 운동과 그림자·반사·조명 변화를 구분하지 못함 |
| Smoothness와 physics의 불일치 | VAMP Velocity / Acceleration Consistency | 일정하고 부드럽지만 물리적으로 틀린 운동이 높은 점수를 받을 수 있고, 충돌처럼 정당한 급격한 변화가 불리할 수 있음 |

### 권장 조합

| 평가 조건 | 권장 Metric 조합 | 이유 |
| --- | --- | --- |
| Paired GT physics video 존재 | Foreground Mask IoU + Background RMSE + Trajectory $L_2$ + Velocity/Acceleration RMSE + Mask Chamfer Distance | 객체·배경·궤적·운동학·형상을 분리해 진단 가능 |
| GT 없이 객체/배경 안정성 평가 | CRONOS Object/Background Stability | Reference trajectory 없이 객체와 배경의 붕괴 탐지 가능 |
| GT 없이 물리 법칙 평가 | Morpheus Dynamical Score + Physical Invariance Score | 특정 reference trajectory보다 운동방정식과 보존 법칙을 직접 평가 |
| Trajectory-controlled generation | Mask IoU + centroid trajectory error | 목표 mask의 위치뿐 아니라 객체 실루엣과 이동 경로를 함께 평가 |
| Video object removal | TokSim + BG-PSNR + FG-Flicker | 제거 성공, 배경 보존, 인페인팅 영역의 시간 안정성을 분리 평가 |
