# Efficient Video Diffusion Generation — Related Work 목록

확인일: **2026-10-26**. 비디오 diffusion·flow-matching 모델의 **생성 추론 비용을 낮추는 연구**를 모은 1차 읽기 목록이다. [비디오 추론 벤치마크 조사](thread://01a16659-7fb8-7632-bf41-6019ce6358a3?hostId=local)와 같이 핵심 내용·최초 공개일·학회 채택 근거를 표로 정리했다. 아래 묶음은 찾아 읽기 위한 임시 분류이며, 논문의 Related Works 구성은 추후 조정할 수 있다.

사수님이 말씀하신 **“한 번의 video generation 내부에서 발생하는 redundancy 감소”**와 직접 연결되는 연구는 우선 **1절의 캐싱·재사용**과 **2절의 희소 어텐션·토큰 축소**에서 볼 수 있다. 그 밖에 증류·스트리밍·양자화·모델 압축·시스템 최적화도 넓게 포함했다.

**수록: 50편** — 캐싱·재사용 12편, 희소 어텐션·토큰 축소 17편, 증류 6편, 자기회귀·스트리밍 4편, 양자화·압축·시스템 8편, 이미지 생성 선행 연구 2편, 탐색 비용 인접 연구 1편. 각 표 안에서는 최초 공개일순으로 정렬했다.

## 표를 읽는 기준

- **최초 공개일:** 비교 기준을 통일하기 위해 각 논문의 **arXiv v1 제출일(UTC)**을 사용했다. 논문명 링크의 Submission history에서 확인할 수 있다. 더 이른 OpenReview 공개·코드 공개가 있을 수 있으므로 인터넷 전체에서의 절대 최초 공개일을 뜻하지 않는다.
- **학회 accept:** 공식 proceedings·학회 페이지·저자의 공식 프로젝트/논문에서 확인한 채택만 기록했다. 워크숍·Technical Communications·저널은 본회의와 구분한다. **‘accept 확인 안 됨’은 거절을 뜻하지 않는다.**
- **핵심 내용:** 현재 확인 가능한 논문 버전의 방법을 요약한다. 같은 arXiv ID에서 제목이 바뀌어도 최초 날짜는 v1 기준이다. 모든 논문의 핵심 내용은 해당 행의 원문 링크, 채택 정보는 학회 열의 출처에 근거한다.
- **‘시간’의 두 의미:** 비디오의 프레임 시간축과 diffusion의 denoising timestep은 별개다. 스텝 간 캐싱, 프레임 간 유사성, 과거 청크의 KV 재사용을 구분한다.
- **속도 수치:** 모델·해상도·프레임 수·GPU·품질 허용치가 달라 이번 목록에는 통일된 speedup 순위를 만들지 않았다. `training-free`도 calibration·profiling·임계값 조정이 없다는 뜻은 아니다.

## 1. 캐싱·feature 재사용·예측

이전 denoising step의 계산 결과를 그대로 재사용하거나 다음 feature를 예측한다. 모델 계산을 생략해도 sampler의 상태 업데이트는 계속할 수 있으므로, 전체 sampling step 수를 줄이는 증류와 구분한다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[PAB — Real-Time Video Generation with Pyramid Attention Broadcast](https://arxiv.org/abs/2408.12588)** | 2024-08-22 | **[ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/file/092c2d45005ea2db40fc24c470663416-Paper-Conference.pdf)** | 스텝 간 attention 재계산 | Diffusion 중간 구간에서 attention 출력 변화가 작다는 관찰을 이용한다. 공간·시간·cross attention의 안정성 차이에 맞춰 재사용 간격을 달리하고, 계산한 출력을 후속 denoising step에 전달한다. **스텝 간 attention 재계산**을 줄이는 training-free 방법이다. |
| **[FasterCache](https://arxiv.org/abs/2410.19355)** | 2024-10-25 | **[ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/file/518046d86bbc41a0707727c38301ad8e-Paper-Conference.pdf)** | 스텝 간 feature·CFG 분기 중복 | 단순히 이전 feature를 복사할 때 손실되는 변화를 고려해 동적으로 feature를 재사용한다. 여기에 CFG-Cache를 더해 **인접 스텝 간 중복과 같은 스텝의 conditional/unconditional branch 간 중복**을 함께 줄인다. |
| **[AdaCache — Adaptive Caching for Faster Video Generation with Diffusion Transformers](https://arxiv.org/abs/2411.02397)** | 2024-11-04 | **[ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/html/Kahatapitiya_Adaptive_Caching_for_Faster_Video_Generation_with_Diffusion_Transformers_ICCV_2025_paper.html)** | 영상별 불필요한 residual 재계산 | DiT residual feature의 변화량을 이용해 **각 영상별 캐싱 간격**을 정한다. Motion Regularization으로 움직임이 큰 영상에 더 많은 계산을 배정한다. denoising state는 계속 갱신하면서 attention·MLP의 residual 계산을 재사용한다. |
| **[TeaCache — Timestep Embedding Tells: It's Time to Cache for Video Diffusion Model](https://arxiv.org/abs/2411.19108)** | 2024-11-28 | **[CVPR 2025](https://openaccess.thecvf.com/content/CVPR2025/papers/Liu_Timestep_Embedding_Tells_Its_Time_to_Cache_for_Video_Diffusion_CVPR_2025_paper.pdf)** | 변화가 작은 스텝의 model forward | Timestep embedding으로 조절한 입력의 차이를 rescaling해 출력 변화량을 저렴하게 추정한다. 변화가 작을 것으로 예상되는 구간에는 캐시를 사용해 **비슷한 출력을 내는 denoising model의 반복 계산**을 생략한다. |
| **[TaylorSeer — From Reusing to Forecasting: Accelerating Diffusion Models with TaylorSeers](https://arxiv.org/abs/2503.06923)** | 2025-03-10 | **[ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/papers/Liu_From_Reusing_to_Forecasting_Accelerating_Diffusion_Models_with_TaylorSeers_ICCV_2025_paper.pdf)** | 생략한 스텝의 feature 계산 | 과거 feature의 유한차분으로 고차 미분을 근사하고 Taylor 전개로 **생략한 미래 스텝의 feature를 예측**한다. 캐시 값을 그대로 복사할 때 생기는 오차 누적을 줄인다. 이미지·비디오 모두 평가하며 HunyuanVideo 실험이 있다. |
| **[ProfilingDiT — Model Reveals What to Cache: Profiling-Based Feature Reuse for Video Diffusion Models](https://arxiv.org/abs/2504.03140)** | 2025-04-04 | **[ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/papers/Ma_Model_Reveals_What_to_Cache_Profiling-Based_Feature_Reuse_for_Video_ICCV_2025_paper.pdf)** | 배경 중심 블록의 반복 feature 계산 | Attention 분석으로 전경·배경을 주로 처리하는 블록을 구분하고 스텝별 noise prediction 변화도 고려한다. **동적 전경에 필요한 계산을 보존하면서 배경 중심 블록의 반복 feature 계산을 재사용**하는 선택적 캐싱이다. |
| **[MagCache — Fast Video Generation with Magnitude-Aware Cache](https://arxiv.org/abs/2506.09045)** | 2025-06-10 | **[NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/311207bb626e36a8f1d3eb92aa67af22-Abstract-Conference.html)** | 스텝 간 residual 재계산 | 연속 residual output의 **크기 비율**에 나타나는 규칙성을 활용한다. 단일 샘플 calibration으로 얻은 정보와 오차 모델을 이용해 캐싱 구간을 적응적으로 선택하고 중요한 스텝은 다시 계산한다. |
| **[EasyCache — Less is Enough: Training-Free Video Diffusion Acceleration via Runtime-Adaptive Caching](https://arxiv.org/abs/2507.02860)** | 2025-07-03 | **accept 확인 안 됨** — [공식 프로젝트](https://h-embodvis.github.io/EasyCache/)·[저장소](https://github.com/H-EmbodVis/EasyCache) 확인 | 스텝 간 transformation 재계산 | 실행 중 변화 지표를 이용해 이전에 계산한 **transformation vector를 동적으로 재사용**한다. 별도 offline profiling이나 사전 계산 없이 캐싱을 결정하며 OpenSora·Wan2.1·HunyuanVideo를 평가한다. |
| **[HiCache — A Plug-in Scaled-Hermite Upgrade for Taylor-Style Cache-then-Forecast Diffusion Acceleration](https://arxiv.org/abs/2508.16984)** | 2025-08-23 | **[ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/hash/b67c77f8db8b991d73d6f2e16f491840-Abstract-Conference.html)** | 캐시 기반 feature 예측 오차 | Feature 미분 근사의 통계적 특성을 바탕으로 Hermite 다항식과 dual scaling을 사용해 **캐시 기반 feature 예측의 정확성·안정성**을 높인다. 이미지·비디오·초해상도에 적용한다. v1 제목은 *Training-free Acceleration of Diffusion Models via Hermite Polynomial-based Feature Caching*이다. |
| **[DiCache — Let Diffusion Model Determine Its Own Cache](https://arxiv.org/abs/2508.17356)** | 2025-08-24 | **[ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/hash/78288ef33b18a351c3cd679dc9a15c8d-Abstract-Conference.html)** | 스텝 간 feature 재계산 | 얕은 층의 online probe로 **언제 캐시를 쓸지** 판단하고, 얕은·깊은 층의 feature trajectory 관계를 이용해 여러 과거 캐시를 정렬하여 **캐시를 어떻게 쓸지**도 결정한다. Wan2.1·HunyuanVideo·FLUX에서 검증한다. |
| **[SenCache — Accelerating Diffusion Model Inference via Sensitivity-Aware Caching](https://arxiv.org/abs/2602.24208)** | 2026-02-27 | **[CVPR 2026](https://openaccess.thecvf.com/content/CVPR2026/html/Haghighi_SenCache_Accelerating_Diffusion_Model_Inference_via_Sensitivity-Aware_Caching_CVPR_2026_paper.html)** | 민감도가 낮은 구간의 재계산 | Noisy latent와 timestep의 변화에 대한 **모델 출력의 민감도**로 캐싱 오차를 분석하고 샘플별 재계산 시점을 선택한다. 고정 heuristic 대신 sensitivity를 기준으로 중복 denoising 계산을 줄이며 Wan2.1·CogVideoX·LTX-Video를 평가한다. |
| **[NaviCache — Test-Time Self-Calibration Caching for Video Generation](https://arxiv.org/abs/2606.26795)** | 2026-06-25 | **[ICML 2026](https://proceedings.mlr.press/v306/lv26m.html)** | 스텝 간 block 재계산 | 입력·출력 변화의 비율과 drift를 두 상태로 추적하고 불확실성에 따라 측정값을 갱신한다. 이 test-time self-calibration으로 **캐싱 오차를 추정해 재계산을 생략**하며 HunyuanVideo·Wan·Open-Sora에서 평가한다. |

## 2. 희소 어텐션·토큰 축소

각 forward에서 처리하는 token 또는 attention 연결을 줄인다. DiTFastAttn·LiteAttention·HVA처럼 스텝 간 재사용까지 결합한 연구도 있으며, 가중치 학습 없이 적용하는 방식과 학습·증류가 필요한 방식을 각 행에서 구분했다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[DiTFastAttn: Attention Compression for Diffusion Transformer Models](https://arxiv.org/abs/2406.08552)** | 2024-06-12 | [NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/0267925e3c276e79189251585b4100bf-Paper-Conference.pdf) | 공간·denoising step·CFG 분기의 attention 중복 | window attention과 residual sharing, timestep 사이 attention 공유, conditional/unconditional CFG branch 사이 공유를 결합한다. 이미지뿐 아니라 OpenSora 영상 생성도 검증해 본 목록에 포함할 수 있다. caching 표와 중복될 수 있다. |
| **[Importance-Based Token Merging for Efficient Image and Video Generation](https://arxiv.org/abs/2411.16720)** | 2024-11-23 | [ICCV 2025](https://openaccess.thecvf.com/content/ICCV2025/papers/Wu_Importance-Based_Token_Merging_for_Efficient_Image_and_Video_Generation_ICCV_2025_paper.pdf) | 유사 토큰의 반복 처리 | CFG 크기 등 이미 계산된 중요도 신호로 중요한 토큰을 보존하면서 유사 토큰을 병합한다. 이미지·다중 시점·AnimateDiff 영상 생성에서 검증한 범용 방법이다. |
| **[Sparse VideoGen (SVG)](https://arxiv.org/abs/2502.01776)** | 2025-02-03 | [ICML 2025](https://proceedings.mlr.press/v267/xi25c.html) | 3D full attention의 낮은 기여도 연결 | attention head를 공간형·시간형으로 온라인 분류하고 각 유형에 맞는 sparse pattern을 적용한다. tensor layout과 전용 kernel을 함께 설계한 training-free 비디오 DiT 가속이다. |
| **[Fast Video Generation with Sliding Tile Attention (STA)](https://arxiv.org/abs/2502.04507)** | 2025-02-06 | [ICML 2025](https://proceedings.mlr.press/v267/zhang25m.html) | 국소 3D 영역 밖의 attention | 토큰별 sliding window 대신 GPU tile에 맞는 시공간 sliding window를 사용한다. training-free 적용과 fine-tuning을 통한 추가 가속을 모두 다룬다. |
| **[Efficient-vDiT: Efficient Video Diffusion Transformers With Attention Tile](https://arxiv.org/abs/2502.06155)** | 2025-02-10 | [ICML 2025 ES-FoMo **Workshop**](https://haozhang.ai/publication/) | 반복 attention tile + denoising 단계 수 | 비디오의 반복 attention tile 패턴으로 frame 수에 선형적인 sparse attention을 구성하고, multi-step consistency distillation을 결합한다. 순수 training-free 방법이 아니라 3단계 학습을 사용하는 혼합 방식이다. |
| **[SpargeAttention / SpargeAttn](https://arxiv.org/abs/2502.18137)** | 2025-02-25 | [ICML 2025](https://proceedings.mlr.press/v267/zhang25ch.html) | attention의 불필요한 행렬곱 | attention map을 빠르게 예측한 뒤 online softmax 정보를 활용하는 2단계 필터로 계산을 건너뛴다. 비디오를 포함한 여러 모델에 적용하는 training-free sparse attention이며 저정밀 연산도 함께 사용한다. |
| **[VSA: Faster Video Diffusion with Trainable Sparse Attention](https://arxiv.org/abs/2505.13389)** | 2025-05-19 | [NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/dfc310e81992d2e4cedc09ac47eff13e-Abstract-Conference.html) | 중요도가 낮은 attention tile | coarse 단계에서 tile을 pooling해 중요한 영역을 선택하고 fine 단계에서 해당 tile 내부만 정밀 계산한다. 학습과 추론 모두에 쓰는 미분 가능한 sparse attention이며 step distillation과 결합한 Sparse-Distill도 제시한다. |
| **[Sparse VideoGen2 (SVG2)](https://arxiv.org/abs/2505.18875)** | 2025-05-24 | [NeurIPS 2025](https://papers.neurips.cc/paper_files/paper/2025/hash/8bd5148caced2d73cea7b6961a874a49-Abstract-Conference.html) · [Spotlight](https://svg-project.github.io/v2/) | 부정확한 중요 토큰 선택·흩어진 토큰의 계산 낭비 | 의미 유사도에 따른 k-means clustering과 token permutation으로 중요한 토큰을 모은다. top-p budget과 가변 크기 sparse attention kernel을 결합한 training-free 방식이다. |
| **[VORTA: Efficient Video Diffusion via Routing Sparse Attention](https://arxiv.org/abs/2505.18809)** | 2025-05-24 | [NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/file/0badcb4e95306df76a719409155e46e8-Paper-Conference.pdf) | full 3D attention의 국소·장거리 계산 | 장거리 의존성을 처리하는 sparse attention과 여러 sparse attention 변형을 적응적으로 선택하는 routing을 결합한다. 캐싱·step distillation과 함께 쓸 수도 있다. |
| **[Astraea: A Token-wise Acceleration Framework for Video Diffusion Transformers](https://arxiv.org/abs/2506.05096)** | 2025-06-05 | [ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/hash/921ac785fa9edc73cacaf2664f43d234-Abstract-Conference.html) | 모든 step에서 모든 토큰을 동일하게 처리하는 비용 | 경량 token selection과 GPU 친화적 sparse attention을 결합하고, 진화 탐색으로 denoising timestep별 token budget을 정한다. |
| **[Radial Attention](https://arxiv.org/abs/2506.19852)** | 2025-06-24 | [NeurIPS 2025](https://papers.neurips.cc/paper_files/paper/2025/hash/18a367688479c1b08b001584218a4443-Abstract-Conference.html) | 시공간 거리가 먼 약한 attention 연결 | 시간 거리가 멀어질수록 공간 attention window를 좁히는 정적 sparse mask를 사용해 attention 복잡도를 $O(n\log n)$으로 줄인다. 사전학습 모델 가속과 LoRA 기반 길이 확장을 다룬다. |
| **[VMoBA: Mixture-of-Block Attention for Video Diffusion Models](https://arxiv.org/abs/2506.23858)** | 2025-06-30 | [ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/hash/d6c4014ff8d95025aa35d831c0f81faa-Abstract-Conference.html) | 중요도가 낮은 query–key block 상호작용 | 층별 1D/2D/3D block 분할, head 전체의 중요 block 선택, 누적 유사도 기반 threshold를 결합한다. 비디오 특성에 맞춘 native sparse 학습과 training-free 추론을 모두 평가한다. |
| **[LiteAttention: A Temporal Sparse Attention for Diffusion Transformers](https://arxiv.org/abs/2511.11062)** | 2025-11-14 | accept 확인 안 됨 ([논문](https://arxiv.org/abs/2511.11062)·[공식 저장소](https://github.com/moonmath-ai/LiteAttention)) | step별 반복 sparse-pattern 탐색·attention 계산 | denoising step 사이에서 중요하지 않은 attention tile이 계속 중요하지 않다는 성질을 이용한다. skip 결정을 이후 step으로 전파해 매번 profiling하는 비용과 해당 tile 계산을 함께 줄인다. 여기서 temporal은 **영상 프레임 간 시간보다 denoising step 간 변화**를 뜻한다. |
| **[CalibAtt: Accelerating Text-to-Video Generation with Calibrated Sparse Attention](https://arxiv.org/abs/2603.05503)** | 2026-03-05 | [ECCV 2026 (공식 코드의 accept 명시)](https://github.com/apple-aiml-research/ml-calibatt) | 입력에 걸쳐 안정적인 sparse 연결·반복 query 패턴 | offline calibration으로 layer·head·timestep별 희소성과 반복 패턴을 찾아 미리 실행 형태로 구성한다. 추론에서는 선택된 연결만 계산하며, 가중치 재학습은 없지만 calibration 비용은 있다. |
| **[SVG-EAR: Parameter-Free Linear Compensation for Sparse Video Generation via Error-aware Routing](https://arxiv.org/abs/2603.08982)** | 2026-03-09 | [ECCV 2026](https://eccv.ecva.net/Conferences/2026/AcceptedPapers) | 생략 가능한 attention block·희소화 복원 오차 | 생략한 block의 기여를 cluster centroid로 보상하고, 작은 probe로 보상 오차가 큰 block을 골라 정확 계산한다. 추가 학습 없이 sparse attention의 품질–비용 균형을 개선한다. |
| **[Veda: Scalable Video Diffusion via Distilled Sparse Attention](https://arxiv.org/abs/2605.30325)** | 2026-05-28 | [ICML 2026](https://icml.cc/virtual/2026/poster/66331) | full attention의 불필요한 tile | full attention의 tile 중요도를 경량 predictor에 증류한다. 통계 기반 tile scoring·head별 tiling과 tile-skipping kernel을 결합한다. denoising step을 줄이는 증류가 아니라 **어떤 attention tile을 계산할지 학습하는 증류**다. |
| **[HyperVAttention (HVA)](https://arxiv.org/abs/2607.03012)** | 2026-07-03 | [NeurIPS 2026](https://neurips.cc/virtual/2026/poster/153122) | clustering 반복·불규칙 sparse block의 GPU 낭비 | 3D local-window clustering, anchor step에서의 전체 clustering과 중간 step의 부분 갱신, GPU tile에 맞춘 cluster 병합을 결합한다. sparse attention 자체뿐 아니라 sparse pattern을 만드는 반복 작업도 줄인다. |

## 3. 증류·소수 스텝 생성

다단계 생성 과정을 소수 스텝 모델로 학습한다. 기존 모델의 중간 결과를 추론 시 재사용하는 캐싱과는 구분한다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[VideoLCM: Video Latent Consistency Model](https://arxiv.org/abs/2312.09109)** | 2023-12-14 | accept 확인 안 됨 | Denoising 스텝 수 | 기존 latent video diffusion model을 consistency distillation하여 소수 스텝 생성기로 만든다. 영상 품질과 시간적 일관성을 유지하는 4-step 생성의 초기 연구다. |
| **[AnimateLCM](https://arxiv.org/abs/2402.00769)** | 2024-02-01 | [SIGGRAPH Asia 2024 **Technical Communications**](https://github.com/G-U-N/AnimateLCM) | Denoising 스텝 수 | 이미지 생성 prior와 motion prior의 consistency learning을 분리해 학습하고, 개인화 모델·adapter와 결합할 수 있는 소수 스텝 비디오 생성을 구현한다. 본회의 Technical Papers와 다른 트랙이다. |
| **[T2V-Turbo](https://arxiv.org/abs/2405.18750)** | 2024-05-29 | [NeurIPS 2024](https://papers.nips.cc/paper/2024/file/8a57aa8e8b57e64a42e95f7dceb0adb9-Paper-Conference.pdf) | Denoising 스텝 수 | Consistency distillation에 이미지·비디오 reward model의 피드백을 결합해 소수 스텝에서의 품질 저하를 줄인다. 대표적으로 4-step 생성을 평가한다. |
| **[SF-V: Single Forward Video Generation Model](https://arxiv.org/abs/2406.04324)** | 2024-06-06 | [NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/bb80772833b5e7ce85321a4fb50a66e7-Paper-Conference.pdf) | 반복 denoising 전체 | Stable Video Diffusion을 시공간 discriminator를 이용한 adversarial training으로 미세조정해 단일 forward의 image-to-video 생성기로 만든다. |
| **[T2V-Turbo-v2](https://arxiv.org/abs/2410.05677)** | 2024-10-08 | [ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/file/e68af7d8a44bc1964f6be4de464e38f9-Paper-Conference.pdf) | Denoising 스텝 수 | Consistency distillation에서 학습 데이터·reward·teacher의 conditional guidance를 함께 설계한다. 특히 motion guidance를 teacher ODE solver에 넣어 소수 스텝 영상의 움직임을 개선한다. |
| **[rCM: Large Scale Diffusion Distillation via Score-Regularized Continuous-Time Consistency](https://arxiv.org/abs/2510.08431)** | 2025-10-09 | [ICLR 2026](https://proceedings.iclr.cc/paper_files/paper/2026/hash/0534abc9e6db91683d82186ef0d68202-Abstract-Conference.html) | Denoising 스텝 수 | Continuous-time consistency에 score distillation 정규화를 결합하고 대규모 비디오 모델용 JVP 계산을 지원한다. Cosmos·Wan 계열을 1–4-step 생성으로 증류한다. |

## 4. 자기회귀·스트리밍 생성

한 영상의 프레임·청크를 순차 생성하면서 과거 문맥의 KV를 재사용한다. 출력 프레임의 시간축과 denoising timestep은 서로 다른 축이다. 첫 프레임 지연시간, 생성 throughput, 전체 영상 완료시간도 구분해서 읽어야 한다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[CausVid: From Slow Bidirectional to Fast Autoregressive Video Diffusion Models](https://arxiv.org/abs/2412.07772)** | 2024-12-10 | [CVPR 2025](https://openaccess.thecvf.com/content/CVPR2025/papers/Yin_From_Slow_Bidirectional_to_Fast_Autoregressive_Video_Diffusion_Models_CVPR_2025_paper.pdf) | Denoising 스텝·과거 문맥 재계산 | Bidirectional teacher를 causal autoregressive student로 증류한다. 4-step 생성과 KV cache를 결합하여 영상을 청크 단위로 빠르게 출력한다. |
| **[Self Forcing](https://arxiv.org/abs/2506.08009)** | 2025-06-09 | [NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/hash/f4823f831af67a3ef15e41a85434422a-Abstract-Conference.html) | 소수 스텝 생성·과거 문맥 재계산 | 학습 시에도 자신의 이전 생성 결과를 조건으로 autoregressive rollout하여 train–test gap을 줄인다. Few-step 모델과 rolling KV cache로 효율적인 스트리밍 생성을 구현한다. |
| **[AAPT: Autoregressive Adversarial Post-Training for Real-Time Interactive Video Generation](https://arxiv.org/abs/2506.09350)** | 2025-06-11 | [NeurIPS 2025](https://proceedings.neurips.cc/paper_files/paper/2025/file/3a9468a918fc65dc9ce7b7bd99f4f0ef-Paper-Conference.pdf) | 프레임별 반복 denoising·문맥 재계산 | Pretrained video diffusion model을 adversarial post-training하여 latent frame마다 1회 함수 평가로 생성한다. KV cache와 student-forcing 학습을 활용하여 장시간 생성의 누적 오류를 줄인다. |
| **[Causal-rCM](https://arxiv.org/abs/2606.25473)** | 2026-06-24 | accept 확인 안 됨 | Causal 생성의 denoising 스텝 수 | Teacher-forcing consistency distillation으로 초기화한 뒤 self-forcing DMD를 결합하는 causal distillation 방법이다. rCM을 스트리밍·상호작용형 비디오 생성으로 확장한다. [공식 설명](https://github.com/NVlabs/rcm/blob/main/Causal_rCM.md) |

## 5. 양자화·모델 압축·통합 가속·시스템

연산의 정밀도, 모델과 latent의 크기, 하드웨어 실행 효율을 바꾼다. 단일 생성의 지연시간을 줄이더라도 모두 동일한 종류의 redundancy 제거를 의미하지는 않는다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[ViDiT-Q: Efficient and Accurate Quantization of Diffusion Transformers for Image and Video Generation](https://arxiv.org/abs/2406.02540)** | 2024-06-04 | **[ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/hash/a4a1ee071ce0fe63b83bce507c9dc4d7-Abstract-Conference.html)** | 선형층 정밀도·가중치/activation 메모리 | 토큰 단위 동적 양자화, static-dynamic 채널 균형 조정, 지표별 민감도를 고려한 mixed precision으로 이미지·비디오 DiT의 저비트 선형층을 구현한다. 가중치·activation 저장량과 연산 정밀도. **PTQ**이며 캐싱이나 토큰 제거와 별개다. |
| **[QVD: Post-training Quantization for Video Diffusion Models](https://arxiv.org/abs/2407.11585)** | 2024-07-16 | **ACM MM 2024** — [저자 arXiv accept 명시](https://arxiv.org/abs/2407.11585) | 연산 정밀도·메모리 | denoising timestep 임베딩의 구별 능력을 유지하는 HTDQ와 채널별 activation 범위를 정돈하는 SCRI로 비디오 diffusion의 양자화 오차를 줄인다. 연산 정밀도와 메모리. **PTQ·캘리브레이션**을 사용한다. 여기서 temporal feature는 프레임 간 캐시가 아니라 timestep에서 유도된 특징이다. |
| **[xDiT: an Inference Engine for Diffusion Transformers (DiTs) with Massive Parallelism](https://arxiv.org/abs/2411.01738)** | 2024-11-04 | **accept 확인 안 됨** — [공식 저장소](https://github.com/xdit-project/xDiT) | 병렬 실행 지연·통신 비용 | sequence parallelism, patch pipeline, CFG parallelism을 조합하는 DiT 분산 추론 엔진. 현재 공식 구현은 여러 비디오 DiT도 지원한다. **여러 GPU의 병렬 실행·통신 효율**을 개선한다. 총 계산량 절감과 지연 시간 단축을 구분해야 하며, 지원 병렬 방식은 모델마다 다르다. |
| **[SageAttention2: Efficient Attention with Thorough Outlier Smoothing and Per-thread INT4 Quantization](https://arxiv.org/abs/2411.10958)** | 2024-11-17 | **[ICML 2025](https://proceedings.mlr.press/v267/zhang25ae.html)** | attention 행렬곱 정밀도·실행 비용 | Q/K를 thread 단위 INT4로 양자화하고 smoothing을 적용하며, P/V의 FP8 연산과 두 단계 accumulation으로 attention 오차를 억제한다. 이미지·비디오·언어 모델에서 검증한 범용 커널이다. **attention 행렬곱의 정밀도·커널 실행 비용**. 모델을 다시 학습하지 않는 교체형 가속이며, attention 연결을 희소화하는 방식과 구분한다. |
| **[VDMini: Individual Content and Motion Dynamics Preserved Pruning for Video Diffusion Models](https://arxiv.org/abs/2411.18375)** | 2024-11-27 | **[ACM MM 2025](https://sites.google.com/site/yimingwu0/home)** — 저자 공식 홈페이지 | 블록 수·각 forward 비용 | 모션 보존에 중요한 깊은 층을 더 유지하면서 얕은 층의 중복 블록을 제거한다. 프레임별 content distillation과 여러 프레임의 adversarial loss로 생성 품질을 복구한다. **모델 블록 수·파라미터·각 forward 비용**. pruning 뒤 **학습·증류가 필요**하다. |
| **[H3AE: High Compression, High Speed, and High Quality AutoEncoder for Video Diffusion Models](https://arxiv.org/abs/2504.10567)** | 2025-04-14 | **accept 확인 안 됨** — [저자 홈페이지의 Preprint 표기](https://wuyushuwys.github.io/) | latent 크기·VAE decoding 비용 | 고압축·고속 비디오 autoencoder를 설계하고, latent consistency loss와 통합 학습으로 재구성 품질을 높인다. 압축된 latent 공간에서 DiT를 학습해 생성 효율을 검증한다. **latent 시공간 크기와 VAE decoding 비용**. 새 AE와 그 latent 공간에 맞춘 생성 모델 **학습이 필요**하며 기존 모델의 캐시를 바로 교체하는 기법은 아니다. |
| **[Q-VDiT: Towards Accurate Quantization and Distillation of Video-Generation Diffusion Transformers](https://arxiv.org/abs/2505.22167)** | 2025-05-28 | **[ICML 2025](https://proceedings.mlr.press/v267/feng25q.html)** | 저비트 DiT의 연산·메모리 | Token-aware Quantization Estimator로 토큰·특징 차원의 양자화 오차를 보정하고, Temporal Maintenance Distillation으로 프레임 간 관계를 유지한다. **저비트 video DiT의 연산·메모리 비용**. 양자화 파라미터와 TQE를 최적화하는 **post-training calibration**이 필요하다. 스텝 수를 줄이는 증류와는 목적이 다르다. |
| **[TurboDiffusion: Accelerating Video Diffusion Models by 100–200 Times](https://arxiv.org/abs/2512.16093)** | 2025-12-18 | **accept 확인 안 됨** — [공식 저장소](https://github.com/thu-ml/TurboDiffusion) | attention·스텝 수·정밀도·실행 비용 | SageAttention·학습 가능한 Sparse-Linear Attention, rCM 스텝 증류, W8A8 선형층 양자화와 엔지니어링 최적화를 결합한 비디오 가속 프레임워크다. **attention·스텝 수·정밀도·실행 비용을 함께 절감**. 증류·sparse attention **학습이 포함**되므로 단일 training-free 캐싱 기법으로 분류하면 안 된다. |

## 6. 이미지 diffusion에서 출발한 선행 연구

비디오 캐싱 연구의 배경을 읽기 위한 항목이다. 비디오 전용 방법으로 세지 않는다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[DeepCache — Accelerating Diffusion Models for Free](https://arxiv.org/abs/2312.00858)** | 2023-12-01 | **[CVPR 2024](https://openaccess.thecvf.com/content/CVPR2024/html/Ma_DeepCache_Accelerating_Diffusion_Models_for_Free_CVPR_2024_paper.html)** | 스텝 간 U-Net 고수준 feature | U-Net의 고수준 feature를 인접 denoising step에서 재사용하고 저수준 feature만 갱신한다. **스텝 간 feature 중복 활용의 이미지 생성 선행 연구**로 포함한다. 원 논문을 video-specific 방법으로 분류하지 않는다. |
| **[TGATE / T-GATE — Faster Diffusion via Temporal Attention Decomposition](https://arxiv.org/abs/2404.02747)** | 2024-04-03 | **[TMLR 2025](https://openreview.net/pdf?id=xXs2GKXPnH)** | 단계별 self/cross-attention 재계산 | 초기 semantic planning과 후기 fidelity improvement에서 self/cross-attention의 역할이 다르다는 관찰을 이용해 attention 출력을 단계별로 재사용한다. **이미지 text-conditional diffusion 출발**이며 이후 비디오 가속 비교에 사용된다. v1 제목은 *Cross-Attention Makes Inference Cumbersome in Text-to-Image Diffusion Models*이다. |

## 7. 여러 후보의 탐색 비용을 다루는 인접 연구

생성 내부의 가속을 여러 후보의 선택 과정에 활용한 경우다. 단일 영상 생성기 가속과 최적화 범위를 구분해 함께 읽을 필요가 있다.

| 논문 | 최초 공개일 | 학회·저널 accept | 줄이는 비용·중복 | 핵심 내용 |
|---|---|---|---|---|
| **[CachedSearch — Training-Free Cached Exploration for Test-Time Search in Video Diffusion](https://arxiv.org/abs/2607.23159)** | 2026-07-25 | **accept 확인 안 됨** — [저자 프로젝트](https://shreshthsaini.github.io/CachedSearch/)·[저장소](https://github.com/shreshthsaini/CachedSearch) 확인 | 여러 후보의 탐색 총비용 | 여러 후보를 캐싱으로 저렴하게 생성·평가한 뒤 **선택된 후보만 full compute로 다시 생성**한다. 캐싱이 후보 순위를 얼마나 유지하는지도 분석한다. 기본 캐싱은 각 rollout 내부에 적용되지만 목적은 **best-of-N/test-time search 전체 비용 감소**이므로 위 12종과 구별한다. |

## Related Works를 읽으며 유지할 구분

**현재 목록에서 캐싱과 희소 어텐션 연구는, 한 번의 생성 과정에서 반복되는 feature·attention 계산이나 불필요한 token interaction을 줄이는 방향으로 설명할 수 있다.** 예를 들어 [PAB](https://arxiv.org/abs/2408.12588)·[TeaCache](https://arxiv.org/abs/2411.19108)는 denoising 과정의 재계산을 줄이고, [STA](https://arxiv.org/abs/2502.04507)는 각 attention 연산의 범위를 제한한다.

다만 efficient video generation 전체를 이 설명으로 묶지는 않는다. **증류는 생성 스텝 수, 양자화는 정밀도·메모리, 시스템 연구는 병렬 실행·통신 비용**을 줄인다. 또 [CachedSearch](https://arxiv.org/abs/2607.23159)는 캐싱을 여러 후보의 탐색 비용 감소에 사용하므로, ‘기존 연구는 모두 단일 생성 내부만 최적화한다’는 식의 novelty 주장은 피해야 한다. 이 문서는 문헌 목록이며, 우리 방법과의 차별점이나 기존 연구 전체의 비중을 확정한 결과는 아니다.

### 서지·범위상 주의할 항목

- **HiCache·TGATE:** 초기 arXiv 제목과 출판 제목이 달라 최신 제목을 사용했다. 동일 논문을 별도 항목으로 중복 집계하지 않았다.
- **QVD:** 논문의 temporal feature는 denoising timestep에서 유도되는 embedding을 가리킨다. 프레임 사이 feature 재사용으로 해석하지 않는다. [정의, §3.3](https://arxiv.org/html/2407.11585v2#S3.SS3)
- **Q-VDiT:** 이름에 distillation이 들어가지만 이 표에서는 저비트 모델의 품질을 보전하는 압축 연구다. 양자화 파라미터와 보정 모듈을 최적화하는 calibration을 사용한다. [Appendix B](https://arxiv.org/html/2505.22167v1#A2)
- **xDiT:** 현재 저장소의 비디오 모델 지원과 원 논문의 범위를 구분한다. 저장소에 적힌 별도 PipeFusion 논문의 NeurIPS 채택을 xDiT의 accept로 옮기지 않았다. [공식 저장소](https://github.com/xdit-project/xDiT)
- **TurboDiffusion:** 여러 가속 요소의 통합 연구다. 구성 요소인 SageAttention 계열의 학회 정보를 TurboDiffusion 자체의 accept로 기록하지 않았다. [공식 저장소](https://github.com/thu-ml/TurboDiffusion)

- **Efficient-vDiT:** 확인한 채택은 ICML 2025 ES-FoMo 워크숍이다. 익명 ICLR 제출본이나 홈페이지의 축약 표기를 본회의 accept로 해석하지 않았다. [저자 publication list](https://haozhang.ai/publication/)
