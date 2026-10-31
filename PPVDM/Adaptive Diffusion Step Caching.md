# Adaptive Diffusion Step Caching

분류 확인일: **2026-10-31**. [전체 목록](<Efficient Video Diffusion Generation.md>)의 50편 중 **현재 생성에서 관측한 변화에 맞춰 계산을 생략하는 7편**을 추렸다. **MagCache 1편**은 같은 목적을 사전 통계로 달성하는 비교군으로 함께 둔다. 새로운 전체 분야 검색이 아니라 기존 목록의 재분류이며, 아래 소분류 이름은 이 문서의 비교를 위한 구분이다.

논문 표기는 기존의 **accept · 짧은 제목 · 최초 공개 연월(YYMM)**을 유지했다. 날짜·accept는 전체 목록의 **2026-10-26 확인 정보**를 옮겼으며 이번 작업에서 전부 다시 조사한 것은 아니다. (x)는 accept 미확인이고, 거절 확정을 뜻하지 않는다.

## 핵심 아이디어 먼저

같은 영상을 denoising하는 인접 step에서는 신경망의 중간 feature나 출력이 비슷할 수 있다. 이때 이전 결과를 재사용하면 attention·MLP 또는 denoiser 전체를 다시 계산하는 비용을 줄인다. 그러나 **현재 출력을 전부 계산해서 이전 출력과 비교하면 이미 비용을 지불한 뒤**이므로, 아래 방법들은 입력 변화·얕은 층의 출력·최근 관측 이력 같은 더 저렴한 신호로 재사용 여부를 판단한다. 이 문서의 ‘스킵’은 주로 **sampling step 안의 신경망 계산 생략**을 뜻하며, sampler의 latent 갱신은 계속한다. [TeaCache §3](https://arxiv.org/html/2411.19108v2#S3) · [EasyCache §III-C](https://arxiv.org/html/2507.02860v1#S3.SS3)

## 1. 범위를 어디까지 좁혔는가

| 묶음 | 포함 논문 | 구분 기준 |
|---|---|---|
| **입력 변화로 큰 계산 구간을 스킵** | TeaCache, EasyCache, NaviCache | 현재 입력의 변화로 출력 변화를 추정한다. 입력을 가공하는 방식과 입력→출력 변화의 변환식이 다르다. |
| **얕은 층을 먼저 계산하고 깊은 층을 스킵** | DiCache | 현재 step에서 작은 probe를 실제로 계산해 판단 신호를 얻는다. |
| **관측한 residual 변화로 다음 재계산 간격 결정** | AdaCache / AdaCache-MoReg | 실제 계산한 step들의 변화를 보고 앞으로 몇 step을 재사용할지 정한다. |
| **latent·timestep 변화에 대한 민감도로 스킵** | SenCache | 두 입력의 이동량을 각각 사전 측정한 민감도로 가중한다. |
| **변화·중요도에 따라 일부 token만 갱신** | Astraea | 한 step의 큰 구간을 일괄 생략하는 대신, 갱신할 token을 고른다. |
| **비교군: 사전 통계로 스킵 오차 추정** | MagCache | 현재 영상의 변화 신호 대신 사전 측정한 step별 residual 크기 비율을 사용한다. |

**NaviCache에 가장 가까운 비교 묶음은 TeaCache·EasyCache·NaviCache**다. DiCache는 판단을 위한 신경망 계산이 추가되고, AdaCache는 다음 재사용 길이를 결정하며, SenCache는 출력 자체를 복사한다. Astraea는 계산을 생략하는 단위가 token으로 작아진다. 각 차이의 원문 근거는 아래 표·설명에 연결했다.

## 2. 핵심 논문: 무엇을 계산하지 않는가

Training-free는 추가 신경망 학습 여부다. 다항식 fitting·통계 측정·설정 검색은 가능하므로 **training-free와 calibration-free는 별개**다. 외부 모델은 기본 생성기·text encoder·VAE 이외에 방법 적용을 위해 별도 실행하는 모델을 뜻한다. 비디오 모델 학습은 원래 사전학습이 아니라 해당 가속 방법을 적용하기 위한 추가 학습이다.

| 논문 | Training-free | 외부 모델 사용 | 비디오 모델 학습 | 효율화 위치 | 효율화 방법 |
|---|---|---|---|---|---|
| [CVPR'25](https://openaccess.thecvf.com/content/CVPR2025/papers/Liu_Timestep_Embedding_Tells_Its_Time_to_Cache_for_Video_Diffusion_CVPR_2025_paper.pdf) **[TeaCache (2411)](https://arxiv.org/abs/2411.19108)** | O (다항식 보정) | X | X | DiT 블록 묶음 | 입력 변화로 재사용을 판정하고, 현재 embedded input에 저장한 블록 residual을 더함 |
| (x) **[EasyCache (2507)](https://arxiv.org/abs/2507.02860)** | O | X | X | denoiser 전체 forward | 입력·출력 변화 비율로 재사용을 판정하고, 현재 latent에 저장한 출력−입력 변환량을 더함 |
| [ICML'26](https://proceedings.mlr.press/v306/lv26m.html) **[NaviCache (2606)](https://arxiv.org/abs/2606.26795)** | O | X | X | denoiser 전체 forward | 변화 비율을 칼만 필터 방식으로 보정하여 재사용을 판정하고, 현재 latent에 저장한 residual을 더함 |
| [ICLR'26](https://proceedings.iclr.cc/paper_files/paper/2026/hash/78288ef33b18a351c3cd679dc9a15c8d-Abstract-Conference.html) **[DiCache (2508)](https://arxiv.org/abs/2508.17356)** | O | X | X | 얕은 probe 이후 깊은 층 | 앞쪽 몇 층은 실행하고, 그 변화에 맞춰 과거 전체 모델 residual들을 조합해 뒤쪽 계산을 대체 |
| [ICCV'25](https://openaccess.thecvf.com/content/ICCV2025/html/Kahatapitiya_Adaptive_Caching_for_Faster_Video_Generation_with_Diffusion_Transformers_ICCV_2025_paper.html) **[AdaCache (2411)](https://arxiv.org/abs/2411.02397)** | O | X | X | 각 블록의 attention·MLP residual 가지 | 관측한 residual 변화로 재사용 길이를 정하고, 각 가지의 과거 출력을 현재 hidden state에 더함 |
| [CVPR'26](https://openaccess.thecvf.com/content/CVPR2026/html/Haghighi_SenCache_Accelerating_Diffusion_Model_Inference_via_Sensitivity-Aware_Caching_CVPR_2026_paper.html) **[SenCache (2602)](https://arxiv.org/abs/2602.24208)** | O (민감도 보정) | X | X | denoiser 전체 forward | latent·시간 이동의 민감도 점수가 작으면 마지막 noise/velocity 출력 자체를 재사용 |
| [ICLR'26](https://proceedings.iclr.cc/paper_files/paper/2026/hash/921ac785fa9edc73cacaf2664f43d234-Abstract-Conference.html) **[Astraea (2506)](https://arxiv.org/abs/2506.05096)** | O (검색) | X | X | attention·MLP의 일부 token 출력 | 변화·중요도가 높은 token만 새로 계산하고, 나머지 token 출력은 cache에서 가져옴 |

TeaCache의 경계는 공식 HunyuanVideo 구현 기준이다. 입력 embedding·판정용 modulation·최종 head는 계속 실행한다. NaviCache의 전체 forward residual 역시 공식 HunyuanVideo 구현 기준이며, 모델별 이식에서 cache 경계가 같다고 일반화하지 않는다. [TeaCache 코드](https://github.com/ali-vilab/TeaCache/blob/main/TeaCache4HunyuanVideo/teacache_sample_video.py) · [NaviCache 코드](https://github.com/HelloZicky/NaviCache/blob/main/NaviCache4HunyuanVideo/navicache_sample_video.py)

## 3. 핵심 차이: 무엇을 보고, 언제 스킵을 결정하는가

### 3.1 입력 변화로 출력 변화를 추정: TeaCache / EasyCache / NaviCache

| 논문 | 현재 step에서 보는 값 | 출력 변화 추정·갱신 방식 | 스킵 판정 | 사전 calibration |
|---|---|---|---|---|
| **TeaCache** | timestep embedding으로 변조한 입력의 상대 L1 변화 | 사전에 fitting한 다항식으로 입력 변화의 척도를 출력 변화에 맞춤 | 추정 변화량을 누적해 threshold 이하면 블록 재사용; 초과하면 실제 계산·cache 갱신·누적값 초기화 | **있음:** 다항식 fitting |
| **EasyCache** | 현재 latent의 변화 | 최근 실제 계산에서 관측한 출력 변화÷입력 변화 비율을 현재 입력 변화에 곱함 | 추정한 상대 출력 변화의 누적값이 threshold보다 작으면 재사용 | **없음:** 초기 warm-up과 실행 중 실제 계산으로 비율 갱신 |
| **NaviCache** | 현재 latent의 변화 | 출력 변화÷입력 변화 비율의 추정값과 불확실성을 유지; 새 관측이 생기면 이전 추정과 가중 결합 | 예측 출력 변화의 누적값 E_acc가 threshold보다 작으면 재사용 | **없음:** 초기 정렬과 실행 중 관측으로 보정 |

- **차이를 한 문장씩:** TeaCache는 ‘미리 구한 변환 함수’, EasyCache는 ‘마지막 관측 비율’, NaviCache는 ‘여러 관측을 반영해 보정한 비율’을 사용한다.
- **판정값과 cache tensor는 별개:** 출력 변화 추정값은 스킵 여부를 정하는 작은 수치다. 이를 신경망 출력 tensor로 쓰는 것이 아니라, 재사용이 허용되면 저장해 둔 residual로 출력을 구성한다.
- **NaviCache의 P와 E_acc도 별개:** P는 비율 추정의 불확실성으로 새 관측을 얼마나 반영할지에 관여한다. §3.2.3의 직접적인 skip gate는 E_acc다. E_acc를 실제 측정한 누적 생성 오차나 최종 비디오 품질 오차로 해석하지 않는다.

근거: [TeaCache §3.2–3.4](https://arxiv.org/html/2411.19108v2#S3.SS2) · [EasyCache §III-B–C](https://arxiv.org/html/2507.02860v1#S3.SS2) · [NaviCache §3.2](https://arxiv.org/html/2606.26795v2#S3.SS2)

### 3.2 얕은 층의 실제 변화로 판단: DiCache

- **판정 흐름:** 현재 입력 → 앞쪽 m개 층 실행 → shallow output의 step 간 상대 L1 변화 측정 → 누적값이 threshold를 넘는지 판정.
- **작으면:** 뒤쪽 층을 생략한다. 최근 두 실제 계산의 전체 모델 residual을 저장하고, 현재 shallow residual의 이동에 맞춰 두 residual을 보간·외삽한다.
- **크면:** 이미 계산한 shallow feature부터 나머지 층을 이어 실행하고 cache를 갱신한다.
- **구분점:** offline calibration은 없지만 매 step probe 비용이 든다. ‘입력만 보고 전체 denoiser를 스킵’하는 방식과 계산 경계가 다르다. [§3.2, Algorithm 1, Eqs. 5–8](https://arxiv.org/html/2508.17356v2#S3.SS2)

### 3.3 관측한 변화로 다음 재사용 길이를 결정: AdaCache / MoReg

- **판정 흐름:** 최근 두 실제 계산의 residual 차이를 경과 step 수로 나눔 → 미리 정한 ‘변화량 구간→재사용 길이’ codebook에 매핑 → 변화가 작으면 더 오래 재사용.
- **갱신 시점:** 정한 재사용 구간이 끝나면 실제 계산하고 다음 길이를 다시 정한다. 따라서 TeaCache처럼 스킵 중인 매 step에서 현재 입력 변화를 검사하는 방식과 구분한다.
- **재사용 대상:** 각 블록의 attention·cross-attention·MLP residual 가지 출력. 실제 구성은 여러 층에 공통인 재계산 schedule을 사용한다.
- **MoReg:** 프레임 간 residual 차이와 그 차이의 step 간 변화를 추가해, 움직임이 큰 영상에는 더 많은 계산을 배정한다. codebook·threshold 설정은 필요하다. [§4.2–4.3, Eqs. 4–9](https://arxiv.org/html/2411.02397v2#S4.SS2)

### 3.4 latent와 timestep의 이동을 따로 평가: SenCache

- **판정 흐름:** 마지막 실제 계산 시점 대비 latent 이동과 timestep 이동을 구함 → 각각 latent 민감도와 시간 민감도로 가중 → 합친 점수가 tolerance 이하이면 재사용.
- **사전 준비:** 민감도는 작은 입력 perturbation에 대한 출력 변화로 미리 추정한다. 논문은 모델별 8개 calibration 영상을 사용하며, 매 step 정확한 Jacobian을 구하지 않는다.
- **재사용 대상:** 마지막 denoiser 출력 자체다. 현재 입력에 과거 residual을 더하는 EasyCache·NaviCache와 대체 출력이 다르다.
- **갱신:** 허용 점수나 최대 연속 재사용 길이를 넘으면 다시 계산한다. 온라인 입력 변화로 결정하지만 calibration-free는 아니다. [§4.3–4.5, Algorithm 1](https://arxiv.org/html/2602.24208v2#S4.SS3)

### 3.5 변화가 작은 token의 계산만 생략: Astraea

- **판정 흐름:** token별 현재 입력 변화에 이전 attention의 중요도 정보(LSE)를 반영하고, 오랫동안 갱신하지 않은 token에는 추가 점수를 줌 → 점수가 높은 token부터 갱신.
- **재사용 대상:** 선택하지 않은 token의 attention·MLP 출력. self-attention에서는 선택한 query의 출력만 갱신하지만 전체 key/value 정보는 사용한다.
- **사전 준비:** step별로 몇 %의 token을 계산할지는 evolutionary search로 정한다. 그 예산 안에서 어떤 token을 계산할지는 온라인 점수로 고른다.
- **구분점:** ‘이번 step의 denoiser를 통째로 실행할지’라는 이진 결정이 아니다. **현재 변화량을 활용하는 token 단위 선택적 재계산**으로 분리한다. [§3.2–3.3, Eqs. 4–7](https://arxiv.org/html/2506.05096v4)

## 4. 별도 비교군: 사전 통계로 예상 오차를 계산하는 MagCache

| 논문 | Training-free | 외부 모델 사용 | 비디오 모델 학습 | 효율화 위치 | 효율화 방법 |
|---|---|---|---|---|---|
| [NeurIPS'25](https://proceedings.neurips.cc/paper_files/paper/2025/hash/311207bb626e36a8f1d3eb92aa67af22-Abstract-Conference.html) **[MagCache (2506)](https://arxiv.org/abs/2506.09045)** | O (통계 보정) | X | X | denoiser 전체 residual | 사전 측정한 step별 residual 크기 비율로 재사용 오차를 추정하고, 현재 입력에 과거 residual을 더함 |

- **사전 준비:** 하나의 calibration 생성에서 timestep별 residual 크기 비율을 구한다.
- **판정 흐름:** 저장된 비율들을 조합해 cache 기준점 이후의 residual 변화와 누적 오차를 추정 → threshold와 최대 연속 skip 길이 안에서는 재사용 → 넘으면 실제 계산·cache 갱신·누적값 초기화.
- **분리 이유:** ‘변화가 작을 것으로 예상되면 재사용’한다는 목적은 같지만, 그 예상은 **현재 샘플의 입력/probe 변화가 아니라 사전 통계**에 근거한다. 동일한 통계·step schedule·설정이면 스킵 일정도 입력 내용에 맞춰 새로 정해지는 구조가 아니다. [§3.2–3.3, Eqs. 3–8](https://arxiv.org/html/2506.09045v2#S3.SS2)

## 5. 나머지 step 재사용 연구는 별도 소분류로 유지

아래 연구도 step 간 중복을 활용한다. 다만 위의 **온라인 변화 기반 재계산 결정**과 구분해서 읽는다. 각 논문의 상세 설명과 학습·외부 모델 표는 [전체 목록](<Efficient Video Diffusion Generation.md>)에 유지했다.

| 소분류 | 논문 | 재사용 대상과 스킵 방식 | 위 핵심군과의 차이 |
|---|---|---|---|
| **attention별 예정된 주기** | [ICLR'25](https://proceedings.iclr.cc/paper_files/paper/2025/file/092c2d45005ea2db40fc24c470663416-Paper-Conference.pdf) **[PAB (2408)](https://arxiv.org/abs/2408.12588)** | attention 종류별로 정한 간격 동안 출력 재사용. [§2.3, 부록 A.2](https://arxiv.org/html/2408.12588v3#S2.SS3) | 현재 영상의 현재 변화량을 매번 재어 판정하지 않음 |
| **블록 선별 + 예정된 주기** | [ICCV'25](https://openaccess.thecvf.com/content/ICCV2025/papers/Ma_Model_Reveals_What_to_Cache_Profiling-Based_Feature_Reuse_for_Video_ICCV_2025_paper.pdf) **[ProfilingDiT (2504)](https://arxiv.org/abs/2504.03140)** | 배경에 주로 집중하는 블록을 profiling으로 골라 residual 재사용. [§3.3](https://arxiv.org/html/2504.03140v2#S3.SS3) | 무엇을 재사용할지 선별하며, 간격은 timestep에 따른 schedule |
| **사전 검색한 attention 재사용 계획** | [NeurIPS'24](https://proceedings.neurips.cc/paper_files/paper/2024/file/0267925e3c276e79189251585b4100bf-Paper-Conference.pdf) **[DiTFastAttn (2406)](https://arxiv.org/abs/2406.08552)** | layer·step별 검색으로 attention 출력 재사용(AST) 등을 선택. [§3.3–3.5](https://arxiv.org/html/2406.08552v2#S3.SS3) | inference 시점의 온라인 변화량 gate가 아님; CFG 분기 공유는 다른 중복 축 |
| **feature의 선형 외삽** | [ICLR'25](https://proceedings.iclr.cc/paper_files/paper/2025/file/518046d86bbc41a0707727c38301ad8e-Paper-Conference.pdf) **[FasterCache (2410)](https://arxiv.org/abs/2410.19355)** | 최근 attention 출력 두 개의 차분으로 중간 step 출력 예측; 기본적으로 교대 계산. [§2.4](https://arxiv.org/html/2410.19355v2#S2.SS4) | 변화 신호는 다음 값을 만드는 데 사용; 현재 오차로 refresh 여부를 고르는 gate와 구분. CFG-Cache는 추가 구성 |
| **feature의 고차 외삽** | [ICCV'25](https://openaccess.thecvf.com/content/ICCV2025/papers/Liu_From_Reusing_to_Forecasting_Accelerating_Diffusion_Models_with_TaylorSeers_ICCV_2025_paper.pdf) **[TaylorSeer (2503)](https://arxiv.org/abs/2503.06923)** | 주기적으로 계산한 feature의 유한차분으로 Taylor 예측. [§3.2](https://arxiv.org/html/2503.06923v2#S3.SS2) | 핵심은 생략한 feature를 어떻게 예측하는가 |
| **feature 예측식의 교체** | [ICLR'26](https://proceedings.iclr.cc/paper_files/paper/2026/hash/b67c77f8db8b991d73d6f2e16f491840-Abstract-Conference.html) **[HiCache (2508)](https://arxiv.org/abs/2508.16984)** | Taylor-style 예측을 scaled Hermite 다항식으로 교체. [§3.3, Algorithm 1](https://arxiv.org/html/2508.16984v2#S3.SS3) | 지정한 refresh 간격 사이의 feature를 예측; 독립적인 온라인 변화량 gate가 아님 |
| **이미지 선행: U-Net 깊은 feature 재사용** | [CVPR'24](https://openaccess.thecvf.com/content/CVPR2024/html/Ma_DeepCache_Accelerating_Diffusion_Models_for_Free_CVPR_2024_paper.html) **[DeepCache (2312)](https://arxiv.org/abs/2312.00858)** | 정한 cache 깊이·일정에 따라 깊은 경로를 생략하고 얕은 경로는 계산. [§3.3](https://arxiv.org/html/2312.00858v2#S3.SS3) | 예정된 partial forward; DiT나 비디오 성능으로 그대로 일반화하지 않음 |
| **이미지 선행: 생성 단계별 attention 고정** | [TMLR'25](https://openreview.net/pdf?id=xXs2GKXPnH) **[T-GATE (2404)](https://arxiv.org/abs/2404.02747)** | 전반부 self-attention 주기 재사용, 후반부 cross-attention 고정. [§4](https://arxiv.org/html/2404.02747v3#S4) | 정한 phase·gate step을 사용 |

### 인접하지만 cache 대상이나 최적화 범위가 다른 연구

| 논문 | 별도로 두는 이유 |
|---|---|
| (x) **[LiteAttention (2511)](https://arxiv.org/abs/2511.11062)** | 이전 step에서 기여도가 작았던 attention tile의 **skip-mask**를 재사용한다. 출력 feature를 복사하는 방법과 구분한다. [§4.1–4.4](https://arxiv.org/html/2511.11062#S4.SS1) |
| [NeurIPS'26](https://neurips.cc/virtual/2026/poster/153122) **[HVA (2607)](https://arxiv.org/abs/2607.03012)** | **token→cluster 배정과 일부 token의 재배정 계획**을 재사용해 sparse attention 준비 비용을 줄인다. 현재 attention 출력 자체는 계산한다. [§4.2](https://arxiv.org/html/2607.03012#S4.SS2) |
| (x) **[CachedSearch (2607)](https://arxiv.org/abs/2607.23159)** | 후보 내부에는 입력 변화 기반 cache를 쓰지만, 기여의 중심은 **여러 후보를 저렴하게 탐색하고 선택한 seed를 다시 생성하는 전체 예산 배분**이다. 새 단일 생성용 skip 판정기와 구분한다. [§3](https://arxiv.org/html/2607.23159v2#S3) |

## 6. Related Work에 사용할 분류명

- **상위 범주:** denoising step 사이의 계산 재사용 — cross-step computation reuse.
- **이번에 좁힌 핵심 범주:** 현재 생성의 변화에 적응하는 캐싱 — runtime-adaptive caching. 큰 구간의 재사용을 정하는 **6편**과 token 단위로 갱신을 정하는 **Astraea 1편**을 구분한다.
- **바로 옆 비교 범주:** 사전 통계 기반 error-aware caching — MagCache.
- **그다음 비교 범주:** 정해진 schedule의 feature reuse / 생략한 feature를 외삽하는 feature forecasting.

‘재계산 시점 선택’과 ‘생략한 출력의 구성’은 서로 다른 축이다. 예를 들어 DiCache는 현재 probe로 시점을 선택하면서 residual 외삽도 하고, NaviCache는 변화 비율을 보정하면서 출력은 과거 residual로 구성한다. 따라서 **예측을 쓴다는 이유만으로 NaviCache와 TaylorSeer를 같은 소분류로 묶지 않는다.** 하나는 skip 판단용 변화량을 예측하고, 다른 하나는 생략한 feature tensor 자체를 예측한다.

양자화·일반적인 sparse attention·step distillation·자기회귀 문맥 KV cache는 이 문서의 핵심 분류에서 제외한다. 각각 숫자 정밀도, 한 step 안의 연결 수, 학습을 통한 sampling 횟수, 이전 프레임/청크 문맥 재처리라는 다른 비용을 줄이기 때문이다.
