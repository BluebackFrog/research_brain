## 정독 우선순위

- **선정 기준:** 우리 설계와의 직접 관련성 → 각 논문에서 배울 수 있는 서로 다른 수정 메커니즘 → 학회 채택 여부. 아래 순서는 성능 순위가 아니라 정독 추천 순서다.
- **구성:** 전반적 iterative refinement 5편 + spatial local 수정 메커니즘 3편, 중복 없이 총 8편. **필수 4편은 PhyT2V·VideoTPO·RAPO++·VideoRepair**이며, VideoRepair는 local 그룹에 배치했다.
- **전체 읽기 순서:** **VideoRepair → PhyT2V → VideoTPO → RAPO++ → P-Flow → Self-Refining Video Sampling → VideoGrain → GenMAC**. 처음 4편에서 직접 비교 기준을 잡고, 이후 history·noise·ROI 선택·region별 prompt 적용을 보완한다.

### 전반적 iterative refinement · 5편

| 그룹 내 우선순위 | 논문 | 최초 공개 / 발표처 | 정독 목적 |
|:---:|---|---|---|
| 1 | **[PhyT2V](https://arxiv.org/html/2412.00596v2#S3) · 필수** | 2024 / CVPR 2025 | 물리적 오류를 어떤 피드백으로 표현하고 다음 prompt에 반영하는지 파악한다. |
| 2 | **[VideoTPO / TiViBench](https://arxiv.org/html/2511.13704v2#S4) · 필수** | 2025 / CVPR 2026 | 두 생성 후보의 비교를 textual loss·gradient와 prompt 갱신으로 연결하는 방식을 파악한다. |
| 3 | **[RAPO++](https://arxiv.org/html/2510.20206v2#S4) · 필수** | 2025 / 학회·저널 채택 미확인 | SSPO의 다중 평가 피드백과 Feedback Memory가 다음 수정에 어떻게 쓰이는지 파악한다. |
| 4 | **[P-Flow](https://arxiv.org/html/2603.22091v1#S3.SS3)** | 2026 / CVPR 2026 | 누적 텍스트 이력·최근 생성 영상 비교·초기 noise 변경의 관계를 파악한다. |
| 5 | **[GenMAC](https://arxiv.org/html/2412.04440v1#S3)** | 2024 / AAAI 2026 | 오류 진단을 prompt뿐 아니라 frame별 layout·guidance 수정으로 연결하는 방식을 파악한다. |

- **PhyT2V**
	- **우선 읽을 부분:** §3.1–3.3, Fig. 6, Appendix B.2의 refinement 사례.
	- **확인할 질문:** 영상이 caption으로 바뀌면서 어떤 정보가 남는가? 물리 규칙·불일치·평가 점수는 각각 어느 단계에 들어가며, 언제 수정을 중단하는가?
- **VideoTPO / TiViBench**
	- **우선 읽을 부분:** §4 Eq. 1–3, Appendix D.1과 공식 구현의 refinement loop. 벤치마크 전체보다 VideoTPO method를 먼저 읽는다.
	- **확인할 질문:** 같은 round의 두 후보 비교에서 무엇을 개선 방향으로 뽑는가? 두 iteration 사이 변화로 ROI를 찾으려는 우리 설계와 비교 목적이 어떻게 다른가?
- **RAPO++**
	- **우선 읽을 부분:** §IV-A SSPO, Table IV의 history 입력 템플릿, §IV-B의 Stage 3와의 연결. Stage 1·3은 맥락을 잡은 뒤 읽는다.
	- **확인할 질문:** history에 저장하는 항목과 실제 다음 LLM 입력에 넣는 항목은 무엇인가? 추론 시 반복 개선과 rewriter 학습용 pair 수집은 어떻게 구분되는가?
- **P-Flow**
	- **우선 읽을 부분:** §3.3–3.5, Appendix A.1 Algorithm 1과 VLM prompt.
	- **확인할 질문:** 모든 텍스트 history를 유지하면서 시각 입력을 reference·직전·현재 영상으로 제한하는 이유는 무엇인가? reference noise prior와 새 random noise가 비교에 어떤 영향을 주는가?
	- **설정 차이:** 외부 목표 효과 영상을 사용하는 방법이다. reference 없는 우리 설계와 입력 조건을 구분해서 읽는다.
- **GenMAC**
	- **우선 읽을 부분:** §3.1–3.3, Fig. 2–3, Appendix A, §4.4–4.5.
	- **확인할 질문:** 오류 확인·수정 제안·수정 실행을 어떻게 나누고, 진단 결과에 따라 correction agent를 어떻게 선택하는가? 이는 우리 ROI 타당성 판단과 수정 범위 선택을 설계할 때 참고할 수 있다.
	- **설정 차이:** frame별 box를 수정해도 기존 영상의 해당 ROI만 다시 만드는 것은 아니다. 전반적 refinement 그룹에 둔다.

### Spatial local 수정 메커니즘 · 3편

| 그룹 내 우선순위 | 논문 | 최초 공개 / 발표처 | 정독 목적 |
|:---:|---|---|---|
| 1 | **[VideoRepair](https://aclanthology.org/2026.findings-acl.1817/) · 필수** | 2024 / Findings of ACL 2026 | 오류 평가 → 보존 객체·mask 선택 → region별 noise·prompt 적용까지 연결된 직접 비교 대상. |
| 2 | **[Self-Refining Video Sampling](https://proceedings.mlr.press/v306/jang26d.html)** | 2026 / ICML 2026 | 연속 clean latent 예측의 차이로 수정할 시공간 위치를 고르는 방식을 확인한다. |
| 3 | **[VideoGrain](https://openreview.net/pdf?id=SSslAtcPB6)** | 2025 / ICLR 2025 | 여러 region의 prompt가 서로 간섭하지 않도록 attention과 latent를 제어하는 방식을 확인한다. |

- **그룹의 범위:** 3편이 모두 완성 영상에 대한 자동 피드백 loop를 제시하는 것은 아니다. **VideoRepair는 생성 후 오류 보정**, **Self-Refining Video Sampling은 sampling 내부 보정**, **VideoGrain은 지정한 영역·prompt에 따른 영상 편집**이다. 후자의 두 편은 국소 수정 메커니즘을 배우기 위한 인접 연구로 선정했으며, 아래 20편 비교표에 동일한 종류의 outer-loop 방법으로 추가하지 않는다.
- **VideoRepair — 전체 최우선**
	- **우선 읽을 부분:** §2.1–2.3, Fig. 2, 초기 noise 구성과 region별 denoising을 설명하는 수식, 관련 ablation.
	- **확인할 질문:** 왜 오류 영역을 직접 찾기보다 잘된 객체를 선택해 보존 mask를 만드는가? mask는 시간축으로 어떻게 적용되고, 보존 영역의 noise·latent·최종 픽셀 중 실제로 무엇이 유지되는가?
	- **우리 방법과의 비교:** 객체 기반 mask와 iteration 차이 기반 ROI, ROI 타당성 재검증, frame별 accept/freeze의 차이를 먼저 정리한다.
- **Self-Refining Video Sampling**
	- **우선 읽을 부분:** §4.2–4.3, Eq. 10–11, Algorithm 1.
	- **확인할 질문:** 같은 noise level에서 연속 clean 예측의 차이를 어떻게 uncertainty mask로 바꾸는가? mask 밖에서 재사용하는 상태는 무엇인가?
	- **우리 방법과의 비교:** 차이 기반 ROI의 직접적인 참고 대상이다. 다만 비교하는 것은 완성 영상 V0·V1이 아니라 sampling 내부 clean latent 예측이며, 최종 영상의 성공 프레임을 영구 freeze하는 방법은 아니다. uncertainty가 높다는 사실이 실제 오류 위치의 정답을 뜻하지도 않는다.
- **VideoGrain**
	- **우선 읽을 부분:** §3.2–3.4의 space-time cross-attention·self-attention modulation, Appendix I Eq. 6–7의 latent blending.
	- **확인할 질문:** region과 local prompt를 어떻게 연결하고, region 간 속성 누출을 어떻게 줄이는가? 원본 background latent는 어느 단계에 다시 넣는가?
	- **우리 방법과의 비교:** “선택한 ROI에만 refined prompt를 적용”하는 실행 부분의 참고 대상이다. 오류 진단·ROI 타당성 판단·완성 영상의 반복 재평가를 자동화하는 방법으로 해석하지 않는다. SD1.5 기반 attention 제어가 Wan DiT에 그대로 적용된다고 가정하지 않는다.

- **우선순위를 낮춘 이유:** CineCrew는 clip 제작·재시도, VISTA는 후보 선택과 critique, PQSG는 평가 방법이 중심이라 위 8편 다음에 읽는다. FreeInit·NoisEasier는 초기 noise를 바꾸는 원리까지 깊이 비교할 때 후속으로 읽는다.
- **공통 정독 메모:** 각 논문에서 입력·출력, 평가기가 보는 정보, 반복 사이 넘기는 상태, 수정 mask의 의미, mask 밖 보존 방식, 종료 조건을 같은 형식으로 기록한다. 논문과 코드의 동작이 다르면 각각 근거를 구분한다.

## 분류 요약

- **조사 기준일:** 2026-10-14. 총 20편.
- **시드·초기 noise 추가 확인일:** 2026-10-15. 동일 사례의 refinement 반복 사이를 비교하며, 논문별 근거는 아래 **시드·초기 noise** 항목에 표시했다.
- **Reference video 추가 확인일:** 2026-10-15. 아래 표는 현재 생성 목표를 위한 외부 참조 영상 사용 여부를 기준으로 한다.
- **비교 범위:** 영상·shot·segment를 생성한 뒤 피드백이나 생성 결과를 활용해 다시 생성하는 추론 시 반복 개선. prompt 수정뿐 아니라 NoisEasier의 reward 기반 noise 최적화와 FreeInit의 clean latent 기반 재초기화를 포함한다. 생성 전 계획 수정, 학습 데이터 구축, 단일 sampling 내부에서만 수행하는 보정·탐색은 별도 범위로 남겼다.
- **연도 기준:** ‘최초 공개’는 arXiv v1 연도이며, 학회 발표 연도와 구분했다. ‘채택 미확인’은 확인한 1차 출처에서 채택 근거를 확보하지 못했다는 뜻이다.
- **기호:** O = 해당함, X = 해당하지 않음, △ = 조건부·선택적 지원 또는 비교 범위가 다른 이력, — = 확인한 원문·공식 구현에서 정책이나 범위 미확인. 우리 방법은 구현 검증 결과가 아닌 설계안이다.

| 논문 | 최초 공개 | 발표처·연도 | 공간 국소 수정 | 시간 선택 수정 | 누적 텍스트 이력 | 누적 시각 이력 | 시드 변경 | 초기 noise 변경 | Reference video 사용 |
|---|:---:|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **PhyT2V** | 2024 | CVPR 2025 | X | X | — | — | X | X | X |
| **VideoTPO / TiViBench** | 2025 | CVPR 2026 | X | X | X | X | X | X | X |
| **VideoRepair** | 2024 | Findings of ACL 2026 | O | X | — | — | △ | O | X |
| GenMAC | 2024 | AAAI 2026 | X | X | X | X | X | X | X |
| VISTA | 2025 | CVPR 2026 | X | X | X | X | — | — | X |
| PQSG refinement | 2026 | ECCV 2026 | X | X | — | — | — | — | X |
| MotionAgent · rethinking | 2025 | ICCV 2025 | X | X | — | — | X | X | X |
| P-Flow | 2026 | CVPR 2026 | X | X | O | X | — | O | O |
| CineCrew | 2026 | ECCV 2026 | X | O | O | △ | △ | — | X |
| NoisEasier | 2026 | ECCV 2026 | X | X | X | X | — | O | X |
| FreeInit | 2023 | ECCV 2024 | X | X | X | X | X | O | X |
| VideoAgent | 2024 | RLBRew @ RLC 2025 워크숍 | X | X | X | X | X | O | X |
| Bootstrapping Physics-Grounded… | 2025 | ICCV 2025 워크숍 챌린지 보고 | X | X | — | — | — | — | O |
| AIGVE-MACS refinement | 2025 | CVPR 2026 VGBE 워크숍 | X | X | — | — | — | — | X |
| **RAPO++** | 2025 | 채택 미확인 | X | X | O | — | X | X | X |
| NEWTON | 2026 | 채택 미확인 | X | X | O | X | — | — | X |
| SPIRAL | 2026 | 채택 미확인 | X | O | △ | △ | — | — | X |
| CoAgent | 2025 | 채택 미확인 | X | O | △ | △ | — | — | X |
| A²RD · 기본형 | 2026 | 채택 미확인 | X | O | △ | △ | — | — | X |
| VQQA | 2026 | 채택 미확인 | X | X | △ | X | O | O | X |
| **우리 방법 · 설계안** | — | — | O | O | O | O | — | — | X |

- **공간 국소 수정:** 생성된 영상의 특정 영역에 mask 등으로 수정 제어를 적용하는지. ROI 밖 픽셀이 정확히 고정된다는 뜻은 아니다. 입력 객체를 검출하거나 생성 layout을 지정하는 것만으로는 O로 보지 않는다.
- **시간 선택 수정:** 실패한 프레임·shot·segment를 골라 다시 생성하는지. 평가 시 일부 프레임만 관찰하거나 I2V 입력 첫 프레임을 유지하는 것은 포함하지 않는다. SPIRAL·CoAgent·A²RD·CineCrew의 O는 segment·shot 단위이다.
- **누적 이력:** 다음 수정에 여러 과거 결과·피드백을 참조하는지. 직전 결과 하나 또는 best-so-far 하나만 이어 쓰면 X이다. O도 모든 iteration의 모든 정보를 빠짐없이 입력한다는 뜻은 아니며, 실제 범위는 각 논문의 History 항목에 적었다.
- **시각 이력:** 과거 생성 영상·프레임의 참조를 기준으로 한다. 다른 shot의 상태·시각 embedding이나 관련 사례 검색은 △로 구분한다. 최종 후보 선택을 위해 영상을 모아 두는 것과 다음 refinement에 다시 입력하는 것도 구분한다. 갱신된 noise·latent·optimizer state가 이전 연산의 영향을 계승하는 것만으로 누적 시각 이력 O로 보지 않는다.
- **시드 변경:** 동일 영상·shot·segment를 다시 개선할 때 **영상 생성기의 RNG seed 번호**를 바꾸는지. 서로 다른 평가 샘플이나 한 round 안의 후보들에 다른 seed를 쓰는 것만으로는 O가 아니다.
- **초기 noise 변경:** 다음 생성의 시작 noise tensor를 바꾸는지. 전체·부분 재추출, prior와의 혼합, 이전 결과의 re-noising, gradient 갱신을 포함한다. 한 번의 sampling 안에서 timestep마다 latent가 바뀌는 것은 이 축의 대상이 아니다.
- **두 축의 차이:** seed 번호를 바꾸지 않아도 RNG에서 계속 새 난수를 뽑거나 tensor를 직접 수정하면 초기 noise는 달라진다. 따라서 **seed 고정 ≠ 초기 noise 재사용**이다. 전체 영상을 재생성한다는 사실만으로 noise 변경 O로 판정하지 않았다.
- **판정 근거의 범위:** 원문에 명시된 정책을 우선하고, 미명시된 경우 공식 구현으로 보완했다. PhyT2V·VideoTPO·RAPO++·GenMAC·VideoAgent·MotionAgent·FreeInit의 seed/noise 판정은 아래에 명시한 구현 경로 기준이다. 특히 GenMAC의 X/X는 기본 단일-prompt demo, VQQA의 O/O는 CogVideoX 실험 기준이다. 다른 backbone·API까지 같은 정책이라고 일반화하지 않는다.
- **Reference video 사용:** 추론 시 **현재 생성 목표의 내용·동작·시간적 조건을 지정하는 외부 영상·clip**을 입력하거나 비교 기준으로 쓰는지. 목표 효과를 보여 주는 예시 영상과 continuation의 입력 prefix를 모두 포함하되, 역할은 논문별로 구분했다.
- **Reference에서 제외하는 것:** 현재 수정 대상·과거에 자체 생성한 영상은 생성 결과/history, 단일 I2V 이미지·keyframe은 이미지 조건으로 구분한다. 학습용 영상, 평가에만 쓰는 정답 영상, VLM에 판정 방법을 알려 주는 고정 few-shot 예시는 이 축의 reference video로 세지 않는다.

## 본회의·Findings 채택

### PhyT2V

- **발표:** 최초 공개 **2024** → **CVPR 2025**. [arXiv 이력](https://arxiv.org/abs/2412.00596), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2025/html/Xue_PhyT2V_LLM-Guided_Iterative_Self-Refinement_for_Physics-Grounded_Text-to-Video_Generation_CVPR_2025_paper.html).
- **핵심 방법:** 현재 영상의 caption과 prompt 간 불일치·물리 법칙·평가 점수를 LLM에 제공 → 단계적 추론과 상위 물리 원칙 재검토 → prompt 수정 → 영상 재생성.
- **Reference video:** X. prompt로 자체 생성한 영상의 caption·물리 오류를 바탕으로 수정하며, 외부 목표 영상은 입력받지 않는다. [§3](https://arxiv.org/html/2412.00596v2#S3).
- **공간·시간 범위:** 영상 전체를 다시 생성한다. 공간 ROI 선택이나 성공 프레임 freeze는 없다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 CogVideoX T2V 구현 기준). 매 round 생성 호출마다 기본 seed 42로 Generator를 다시 만들어 동일한 초기 Gaussian noise를 재현한다. 논문 자체에는 이 정책이 미명시이다. [반복 호출](https://github.com/pittisl/PhyT2V/blob/7416e18da2c516caf6efb1e60dc142b6745e947b/inference.py#L337-L373), [생성 시 seed 재설정](https://github.com/pittisl/PhyT2V/blob/7416e18da2c516caf6efb1e60dc142b6745e947b/inference.py#L234-L243).
- **History:** 수정 prompt는 다음 라운드로 이어지지만, 과거 라운드의 prompt·평가·영상을 얼마나 누적 입력하는지는 미명시이다.
- **비교 포인트:** 물리 피드백 기반 prompt 개선의 직접 비교 대상. 오류 진단을 공간·프레임별 수정 제어로 연결하는지는 별도 차이이다.
- **근거:** [§3, Fig. 6, §3.2](https://arxiv.org/abs/2412.00596v2).

### VideoTPO / TiViBench

- **발표:** 최초 공개 **2025** → **CVPR 2026**. VideoTPO는 *TiViBench: Benchmarking Think-in-Video Reasoning for Video Generation*의 방법명이다. [arXiv 이력](https://arxiv.org/abs/2511.13704), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Chen_TiViBench_Benchmarking_Think-in-Video_Reasoning_for_Video_Generation_CVPR_2026_paper.html).
- **핵심 방법:** 같은 입력 이미지·prompt로 만든 두 영상 후보를 비교 → 장단점을 textual loss로 표현 → prompt의 수정 방향인 textual gradient 계산 → 공통 prompt 갱신.
- **Reference video:** X. 입력은 단일 I2V 이미지와 prompt이고, 비교하는 두 영상은 그 round에서 자체 생성한 후보이다. [§4 Eq. 1–3](https://arxiv.org/html/2511.13704v2#S4).
- **공간·시간 범위:** 생성 구간 전체를 다시 만든다. 공개 Wan 구현의 첫·마지막 프레임 추출은 평가용 관찰이며, 그 프레임만 수정하는 기능이 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 Wan 구현 기준). 두 후보의 seed 42·123은 서로 다르지만, 다음 round에도 같은 두 seed를 재사용하고 생성마다 Generator를 재설정한다. 따라서 후보별 시작 noise는 유지된다. [round별 후보 생성](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L352-L403), [Generator 재설정](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L130-L144).
- **History:** 현재 후보 쌍·prompt·loss·gradient를 사용한다. 공개 기본 설정은 gradient memory가 0이며, 여러 과거 시도를 누적 입력하지 않는다.
- **비교 포인트:** 두 후보의 비교를 prompt 최적화에 쓰지만, 두 iteration의 영상 차이로 수정 ROI를 계산하는 방식은 아니다.
- **근거:** [§4, Eq. 1–3, Appendix D.1](https://arxiv.org/abs/2511.13704v2), [공식 Wan 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L345).

### VideoRepair

- **발표:** 최초 공개 **2024** → **Findings of ACL 2026**. 최종 제목은 *Self-Correcting Text-to-Video Generation with Misalignment Detection and Localized Refinement*. [arXiv 이력](https://arxiv.org/abs/2411.15115), [공식 proceedings](https://aclanthology.org/2026.findings-acl.1817/).
- **핵심 방법:** 객체별 QA로 정합성 평가 → pointing·segmentation으로 잘된 객체의 보존 mask 구성 → 나머지 영역에 새 noise와 refinement prompt 적용.
- **Reference video:** X. 원본 V0는 prompt·noise로 자체 생성한 수정 대상이며, 별도의 외부 목표 clip이 아니다. [§2 Problem statement](https://arxiv.org/html/2411.15115v3#S2).
- **공간·시간 범위:** 공간 국소 수정 O, 시간 선택 수정 X. 보존 영역도 함께 denoising하므로 영역 밖 픽셀의 hard freeze는 아니다.
- **시드·초기 noise:** 시드 △ / 초기 noise O(원문 기준). score=0이면 새 seed로 전체를 재생성한다. 국소 수정에서는 보존 영역의 기존 초기 noise와 수정 영역에 새로 뽑은 Gaussian noise를 혼합한다. [§2.1, §2.3 Eq. 2](https://arxiv.org/html/2411.15115v3#S2.SS3). 공식 T2V-Turbo 구현은 국소 수정용 seed도 round별로 바꾼다. [local seed·초기 noise 구성](https://github.com/daeunni/VideoRepair/blob/a0a261f55e771403de44accded15c7c5eb0b1370/t2vturbo/pipeline/multidiffusion.py#L349-L368).
- **History:** 현재 영상·QA·직전 선택 결과를 사용하며, 전체 수정 이력의 누적 입력 범위는 미명시이다.
- **비교 포인트:** 영상 점수가 0이면 전체 재생성, 1이면 중단하는 분기가 이미 있다. 우리 방법과는 **프레임별 ROI 타당성 검증으로 region/whole-frame을 선택한다**는 판단 기준을 비교해야 한다.
- **근거:** [§2.1–2.3, Appendix E.1](https://aclanthology.org/2026.findings-acl.1817.pdf). 전역 재생성·중단 분기는 §2.1.

### GenMAC

- **발표:** 최초 공개 **2024** → **AAAI 2026**. [arXiv 이력](https://arxiv.org/abs/2412.04440), [공식 proceedings](https://ojs.aaai.org/index.php/AAAI/article/view/37418).
- **핵심 방법:** prompt를 프레임별 객체 box로 설계 → 영상 생성 → 검증·제안·수정·구조화 agent가 prompt·layout·guidance를 다시 설계.
- **Reference video:** X. 텍스트에서 구성한 layout·prompt·guidance로 영상을 생성하고 검증하며, 외부 reference video는 사용하지 않는다. [§3.1–3.3](https://arxiv.org/html/2412.04440v1#S3.SS1).
- **공간·시간 범위:** 객체 box로 생성을 유도하지만 기존 영상의 ROI나 성공 프레임을 보존하는 수정은 아니다. 전체 재생성으로 분류한다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 기본 단일-prompt demo 한정). 매 round 새 생성 프로세스에 동일 seed 12345678을 전달해 RNG를 재설정하고 같은 시작 noise를 재현한다. 여러 prompt를 batch로 처리하면 앞선 사례의 skip·종료로 RNG 소비 순서가 달라질 수 있어 일반화하지 않는다. [launcher](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/scripts/run_t2v.py#L36-L74), [seed 초기화](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/scripts/evaluation/inference_pipeline_iter.py#L560-L570), [시작 noise 생성](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/lvdm/models/samplers/ddim.py#L130-L138).
- **History:** 현재 영상과 직전 iteration의 구조화된 설계를 사용하며, 모든 과거 수정본을 누적 입력하는 방식은 아니다.
- **비교 포인트:** 수정 대상이 prompt뿐 아니라 layout·guidance까지 확장된다. 생성 위치 제어와 기존 결과의 국소 수정을 구분하는 데 유용하다.
- **근거:** [§3.1–3.3, Appendix A Eq. A3–A6](https://arxiv.org/html/2412.04440v1).

### VISTA

- **발표:** 최초 공개 **2025** → **CVPR 2026**. [arXiv 이력](https://arxiv.org/abs/2510.15831), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Long_VISTA_A_Test-Time_Self-Improving_Video_Generation_Agent_CVPR_2026_paper.html).
- **핵심 방법:** 여러 후보를 tournament로 비교해 champion 선택 → 시각·오디오·맥락 critique → prompt 개선 → 새 후보 생성·비교.
- **Reference video:** X. 비교 대상은 자체 생성 후보와 이전 winner이며, 외부 목표 영상 없이 사용자 prompt와 평가 기준으로 개선한다. [§2 Algorithm 1](https://arxiv.org/html/2510.15831v1#S2).
- **공간·시간 범위:** 영상 전체를 재생성하며, 공간 ROI나 특정 실패 프레임만 수정하지 않는다.
- **시드·초기 noise:** 시드 — / 초기 noise —. Veo 후보 생성·선택과 prompt 수정은 설명하지만, refinement round 사이 seed나 시작 latent의 유지·변경 정책은 명시하지 않는다. [§2 Algorithm 1](https://arxiv.org/html/2510.15831v1).
- **History:** 현재 champion·prompt·critique와 최선 후보의 유지가 핵심이다. 과거 여러 시도의 이력을 다음 수정에 누적 입력하는 방식은 아니다.
- **비교 포인트:** best-so-far 후보 보관과 전체 수정 history 활용의 차이를 보여 준다.
- **근거:** [§2.1–2.2, Algorithm 1](https://arxiv.org/html/2510.15831v1).

### PQSG의 refinement 실험

- **발표:** 최초 공개 **2026** → **ECCV 2026**. [arXiv 이력](https://arxiv.org/abs/2606.25306), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37035-8_17).
- **핵심 방법:** 객체→행동→물리 질문 그래프로 오류 진단 → 질문별 피드백으로 GPT가 prompt 수정 → Wan으로 새 영상 생성·재평가.
- **Reference video:** X(§5.5 기본 refinement 기준). 자체 생성 영상의 PQSG 피드백으로 prompt를 수정한다. reference video를 QG에 추가하는 실험은 별도 ablation이며 기본 refinement 입력과 구분한다. [§5.5–5.6, Appendix B Table 11](https://arxiv.org/html/2606.25306v1#S5.SS5).
- **공간·시간 범위:** 질문별 진단을 mask나 프레임 선택으로 연결하지 않으므로 전체 재생성이다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 평가 피드백으로 prompt를 수정하고 Wan2.2로 재생성하지만, round별 seed·초기 noise 정책은 미명시이다. 공개 평가 코드나 별도의 재현 구현 설정으로 이를 대신 판정하지 않았다. [§5.5](https://arxiv.org/html/2606.25306v1#S5.SS5).
- **History:** 누적 prompt·피드백·시각 입력의 범위는 미명시이다.
- **비교 포인트:** 주목적은 평가이며, refinement는 §5.5의 응용 실험이다. 세밀한 오류 진단과 세밀한 수정 범위가 동일한 기능은 아니다.
- **근거:** [§3, §5.5, Fig. 4](https://arxiv.org/html/2606.25306v1).

### MotionAgent · rethinking 옵션

- **발표:** 최초 공개 **2025** → **ICCV 2025**. [arXiv 이력](https://arxiv.org/abs/2502.03207), [공식 proceedings PDF](https://openaccess.thecvf.com/content/ICCV2025/papers/Liao_MotionAgent_Fine-grained_Controllable_Video_Generation_via_Motion_Field_Agent_ICCV_2025_paper.pdf).
- **핵심 방법:** 입력 이미지의 객체를 Grounded-SAM으로 찾음 → 객체 궤적·카메라 움직임 계획 → optical flow로 변환해 I2V 생성 → **선택적 rethinking**에서 생성 영상과 이전 action을 검토하고 제어 조건 수정.
- **Reference video:** X. 입력은 이미지와 motion text이며, rethinking에서 검토하는 영상은 직전 생성 결과이다. [§3.1.2–3.1.4](https://arxiv.org/html/2502.03207v2#S3.SS1).
- **공간·시간 범위:** 전체 I2V를 다시 생성한다. SAM은 입력 객체의 grounding·운동 제어에 쓰이며, 기존 출력의 ROI만 보정하는 mask는 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 rethinking demo 기준). 최초 생성과 rethinking 모두 생성 직전에 seed 42로 재설정하고 같은 shape의 시작 noise를 재현한다. 수정하는 것은 trajectory·camera 조건이다. [공통 생성 함수](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/run_agent.py#L629-L652), [noise 생성](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/pipeline/pipeline.py#L224-L255).
- **History:** 이전 영상·action을 검토하지만, 여러 refinement 라운드의 모든 이력을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 피드백으로 고치는 대상이 prompt뿐 아니라 객체 궤적·카메라 제어이다. 반복 개선은 선택적 단계이므로 기본 생성 전체의 필수 절차로 해석하지 않는다.
- **근거:** [§3.1.2, §3.1.4, §3.2, §4.3.4, Appendix 8](https://arxiv.org/html/2502.03207v2).

### P-Flow

- **발표:** 최초 공개 **2026** → **CVPR 2026 본회의**. *P-Flow: Prompting Visual Effects Generation*. [arXiv 이력](https://arxiv.org/abs/2603.22091), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Zhao_P-Flow_Prompting_Visual_Effects_Generation_CVPR_2026_paper.html).
- **핵심 방법:** reference 영상의 폭발·변형 같은 시각 효과와 현재 생성 결과를 VLM이 비교 → 효과를 설명하는 prompt 수정 → reference에서 얻은 시간적 noise prior와 random noise를 혼합해 다시 생성.
- **Reference video:** O. 원하는 동적 효과를 보여 주는 **외부 목표 영상**을 사용한다. 해당 영상에서 temporal noise prior를 추출하고, VLM이 생성 결과와 비교해 prompt를 수정한다. [§3.1, §3.3–3.5](https://arxiv.org/html/2603.22091v1#S3.SS1).
- **공간·시간 범위:** 영상 전체를 재생성한다. 효과 관련 단어만 고치는 것은 해당 객체의 spatial ROI만 수정하는 기능이 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise O. 매 iteration마다 새 Gaussian noise를 뽑아 reference의 temporal noise prior와 혼합한다. seed 번호를 변경하는지는 미명시이며, 새 noise 추출만으로 seed 변경을 단정하지 않는다. [Appendix A.1 Algorithm 1, §3.3](https://arxiv.org/html/2603.22091v1#A1.SS1).
- **History:** **모든 과거 prompt·VLM 분석은 누적**하므로 텍스트 O. 직접 보는 영상은 reference·직전 결과·현재 결과로 제한되어, 다회 시각 archive 입력은 X이다.
- **비교 포인트:** 텍스트와 시각 history의 보존 범위를 명시적으로 나눈 직접 비교군이다. 우리 설계는 ROI가 표시된 과거 프레임을 어떻게 선택·활용하는지, 그 이력이 수정 범위 결정에 어떤 역할을 하는지 대비할 수 있다.
- **근거:** [§3.3–3.5, Appendix A.1 Algorithm 1](https://arxiv.org/html/2603.22091v1), [공식 supplement](https://openaccess.thecvf.com/content/CVPR2026/supplemental/Zhao_P-Flow_Prompting_Visual_CVPR_2026_supplemental.pdf).

### CineCrew

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *Better Call CineCrew: Consistent Ultra-Long Narrative-to-Film Generation*. [arXiv 이력](https://arxiv.org/abs/2609.07720), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37023-5_19).
- **핵심 방법:** 서사를 shot·asset·camera 제약으로 구조화 → keyframe과 clip 생성 → Reviewer가 실패 원인을 진단 → 제약·keyframe prompt·video prompt를 수정해 해당 clip 재생성.
- **Reference video:** X. narrative와 character/set 이미지·생성 keyframe을 조건으로 clip을 만들며, Dailies Reviewer가 검토하는 영상도 자체 생성 clip이다. [§3.1, §3.3–3.4](https://arxiv.org/html/2609.07720v1#S3.SS1).
- **공간·시간 범위:** 실패 clip을 재시도하고 통과한 clip을 모아 장편 영상을 구성한다. 공간 X, 시간 O이며 clip 내부의 개별 프레임 freeze는 아니다.
- **시드·초기 noise:** 시드 △ / 초기 noise —(공식 구현의 API 요청값 기준). seed를 지정한 경우 검수 실패 후 retry마다 1을 더하고, 기본값 None이면 seed를 보내지 않는다. 외부 생성 API 내부의 초기 noise는 확인할 수 없다. [검수·재시도 loop](https://github.com/Ironieser/CineCrew/blob/3eac773704088f65ae2e24054c48fccbd4172f2d/src/agents/production_operator/production_operator_agent.py#L291-L334).
- **History:** Production Rulebook에 실패 tag·확정 prompt·실행 피드백을 누적하고 역할에 맞는 제약을 가져오므로 텍스트 O. 시각 △는 asset·accepted keyframe·continuity anchor의 재사용이며, 모든 실패 영상의 누적 관찰을 뜻하지 않는다.
- **비교 포인트:** 성공 구간 유지와 누적 memory를 함께 다루는 학회 채택 비교군이다. 우리 방법의 프레임 단위 accept/freeze 및 ROI 검증과 판단 단위를 구분해야 한다.
- **근거:** [§3.3–3.4, Algorithm 1](https://arxiv.org/html/2609.07720v1), [공식 구현의 Production loop 설명](https://github.com/Ironieser/CineCrew).

### NoisEasier

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *NoisEasier: Test-Time Noise Optimization for Text-to-Video Generation*. [arXiv 이력](https://arxiv.org/abs/2608.30194), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37098-3_3).
- **핵심 방법:** 짧은 sampling으로 영상 생성 → 정합성·시각 품질·motion reward 계산 → gradient로 초기 noise와 sampling 단계의 stochastic noise를 갱신 → 수정된 noise로 영상 재생성.
- **Reference video:** X. 텍스트·noise로 생성한 영상을 reward로 평가하며, 외부 reference clip에 맞추는 목적함수는 사용하지 않는다. [§3.1–3.3](https://arxiv.org/html/2608.30194v1#S3).
- **공간·시간 범위:** 매번 영상 전체를 생성한다. 평가에 일부 프레임·clip·optical flow를 쓰더라도 ROI나 성공 프레임을 선택해 고정하는 기능은 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise O. 초기 noise와 중간 stochastic noise tensor를 reward gradient로 직접 갱신한다. seed 번호의 변경 정책은 미명시이며, 모델 간 평가에 같은 seed를 쓴다는 설명을 refinement 간 고정 근거로 쓰지 않았다. [§3.1 Eq. 4, §3.3 Eq. 7, §4.4](https://arxiv.org/html/2608.30194v1).
- **History:** 현재 생성 결과의 reward로 noise를 갱신하며, 과거 prompt·critique·영상 archive를 누적 재입력하지 않는다. optimizer state와 noise의 계승은 표의 누적 history와 구분한다.
- **비교 포인트:** 모델 가중치와 생성 prompt를 유지하면서 **샘플별 noise 변수**를 최적화한다. prompt를 통한 언어적 수정과 reward gradient를 통한 수치적 수정의 직접 대비가 가능하다.
- **근거:** [§3.1 Eq. 4, §3.3 Eq. 7, Fig. 2](https://arxiv.org/html/2608.30194v1).

### FreeInit

- **발표:** 최초 공개 **2023** → **ECCV 2024 본회의**. *FreeInit: Bridging Initialization Gap in Video Diffusion Models*. [arXiv 이력](https://arxiv.org/abs/2312.07537), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-031-72646-0_22).
- **핵심 방법:** 전체 DDIM sampling으로 clean video latent 생성 → 다시 noise 추가 → 시공간 저주파 성분은 유지하고 고주파는 새 random noise로 교체 → 이 noise에서 전체 sampling을 반복.
- **Reference video:** X. 재초기화에 사용하는 clean latent는 직전 sampling에서 자체 생성한 결과이며, 외부 참조 영상이 아니다. [§4 Eq. 8–11](https://arxiv.org/html/2312.07537v2#S4).
- **공간·시간 범위:** 모든 공간·시간 위치를 다시 생성한다. 3D Fourier filter의 mask는 **주파수 선택**이며 pixel ROI나 특정 프레임 선택이 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise O(공식 AnimateDiff 예제 기준). 영상별 seed는 시작에 한 번 설정하고, 반복 중에는 새 난수를 계속 추출한다. 이전 결과를 re-noise한 저주파와 새 난수의 고주파를 섞으므로 시작 tensor는 달라진다. [seed 설정](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/examples/AnimateDiff/scripts/animate_with_freeinit.py#L94-L114), [반복 구현](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/examples/AnimateDiff/animatediff/pipelines/pipeline_animation.py#L568-L590).
- **History:** 직전 생성 latent의 구조를 다음 초기 noise에 전달한다. 과거 여러 수정본을 명시적으로 보관·재입력하는 텍스트·시각 history는 없다.
- **비교 포인트:** 외부 critic 없이 생성 결과의 저주파 구조를 재사용하는 반복 개선이다. prompt는 고정하며, VLM이 오류를 진단하고 수정 범위를 선택하는 방법과 구분한다.
- **근거:** [§4, Eq. 8–11, Fig. 7, §5.1](https://arxiv.org/html/2312.07537v2). 통상적인 한 번의 denoising 내부 반복과 달리, clean latent를 만든 뒤 sampling을 다시 시작한다.

## 워크숍

### VideoAgent

- **발표:** 최초 공개 **2024** → **RLBRew @ RLC 2025 워크숍** 채택 확인. 비정규(non-archival) 워크숍이며 RLC 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2410.10076), [공식 accepted papers](https://rlbrew2-workshop.github.io/papers.html), [워크숍 성격](https://rlbrew2-workshop.github.io/submit.html).
- **핵심 방법:** 로봇 행동을 위한 video plan을 생성 → 학습된 self-conditioning 모델이 직전 비디오를 입력받아 보정 → VLM이 수용 여부 판단. Suggestive 변형은 언어 피드백도 사용한다.
- **Reference video:** X. 초기 frame·task text에서 시작하고 자체 생성 영상으로 self-conditioning한다. Descriptive-feedback 변형의 고정 few-shot 영상 예시는 critic의 판정 방법을 알려 주는 용도이므로 이 축에서 제외한다. [§3.1–3.2, Appendix B.2·C/Fig. 6](https://arxiv.org/html/2410.10076v3#A2.SS2).
- **공간·시간 범위:** video plan 전체를 수정하며, 공간 ROI나 실패 프레임만 선택하지 않는다.
- **시드·초기 noise:** 시드 X / 초기 noise O(공식 Meta-World 구현 기준). 시작에 seed를 설정한 뒤 refinement 사이에 재설정하지 않으며, 매 생성 시작에서 새 Gaussian noise를 추출한다. [seed 설정](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/experiment/benchmark_mw.py#L21-L23), [refinement loop](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/flowdiffusion/flowdiffusion/inference_utils.py#L401-L440), [시작 noise 생성](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/flowdiffusion/flowdiffusion/goal_diffusion.py#L539-L565).
- **History:** 직전 비디오 하나를 이어 쓰는 구조이므로, 여러 라운드의 누적 텍스트·시각 이력은 X로 분류한다.
- **비교 포인트:** prompt만 고치는 방법과 달리 기존 비디오 자체가 보정 모델의 입력이다. 동명의 video understanding 논문과 구분한다.
- **근거:** [§3.1–3.2, Eq. 12–13, §4.3.1](https://arxiv.org/html/2410.10076v3).

### Bootstrapping Physics-Grounded Video Generation

- **발표:** 최초 공개 **2025**. **ICCV 2025 Perception Test Workshop의 Physics-IQ 챌린지 3위 솔루션 보고·발표**이며, 정규 proceedings 논문 채택은 미확인이다. [arXiv 이력·보고서 설명](https://arxiv.org/abs/2511.20280), [공식 챌린지 안내](https://physics-iq.github.io/workshop/physics-iq-challenge.html), [저자 발표 기록](https://yafeng19.github.io/).
- **핵심 방법:** VLM이 입력 prefix의 물리 단서에서 미래 서술 생성 → LLM이 생성 prompt로 정리 → continuation 생성 → 물리 오류 피드백으로 prompt 수정.
- **Reference video:** O. **외부 3초 prefix 영상**을 VLM의 물리적 미래 예측과 다음 5초 continuation 생성의 조건으로 사용한다. P-Flow의 목표 효과 예시와 달리, 이어 만들 장면의 시작 상태·동역학을 제공한다. [§2](https://arxiv.org/html/2511.20280v1#S2).
- **공간·시간 범위:** 입력 3초 뒤의 5초 continuation 전체를 재생성한다. 주어진 prefix 유지와 성공한 출력 프레임 선택 freeze는 다르다.
- **시드·초기 noise:** 시드 — / 초기 noise —. prompt 수정 후 continuation을 재생성하지만, 반복 사이 seed·시작 noise를 재사용하는지 새로 뽑는지는 명시하지 않는다. [§2 Step 3](https://arxiv.org/html/2511.20280v1#S2).
- **History:** 현재 결과에서 다음 prompt를 만드는 과정은 설명하지만, 누적 history 입력 범위는 미명시이다.
- **비교 포인트:** 영상 continuation이라는 설정 차이를 반영해 비교해야 한다. ICCV 본회의 채택 논문으로 표기하지 않는다.
- **근거:** [§2, Step 1–3](https://arxiv.org/html/2511.20280v1).

### AIGVE-MACS의 refinement 실험

- **발표:** 최초 공개 **2025** → **CVPR 2026 VGBE 워크숍**. *AIGVE-MACS: Unified Multi-Aspect Commenting and Scoring Model for AI-Generated Video Evaluation*. 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2507.01255), [공식 workshop proceedings](https://openaccess.thecvf.com/content/CVPR2026W/VGBE/html/Liu_AIGVE-MACS_Unified_Multi-Aspect_Commenting_and_Scoring_Model_for_AI-Generated_Video_CVPRW_2026_paper.html).
- **핵심 방법:** 평가 VLM이 생성 영상에 대해 여러 측면의 score·comment 출력 → GPT-4.1 Instruction Revisor가 prompt 수정 → HunyuanVideo로 재생성·재평가.
- **Reference video:** X. 현재 instruction과 자체 생성 영상의 score·comment로 개선하며, 별도의 외부 목표 영상은 사용하지 않는다. [§6 Fig. 5](https://arxiv.org/html/2507.01255v1#S6).
- **공간·시간 범위:** 영상 전체를 다시 생성한다. 평가기의 dynamic frame sampling은 관찰 전략이며, 선택된 프레임만 수정하는 방법은 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. HunyuanVideo 재생성은 명시하지만, refinement 사이 seed 및 시작 noise의 재사용·재추출 방식은 명시하지 않는다. [§6](https://arxiv.org/html/2507.01255v1#S6).
- **History:** 현재 instruction·영상의 평가 피드백을 사용하는 것은 명시되지만, 여러 과거 피드백·영상을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 주 기여는 평가 모델이고, 반복 생성은 적용 실험이다. PQSG처럼 평가기의 세밀한 진단이 실제 refinement에 어떻게 쓰이는지를 비교할 수 있다.
- **근거:** [arXiv §6, Fig. 5](https://arxiv.org/html/2507.01255v1#S6).

## 채택 미확인

### RAPO++

- **발표:** 최초 공개 **2025**. **학회 채택 미확인**. 원래 RAPO의 CVPR 2025와 구분한다. [arXiv 이력](https://arxiv.org/abs/2510.20206), [공식 저장소](https://github.com/Vchitect/RAPO).
- **핵심 방법:** 검색·문장 정리로 초기 prompt를 구성 → SSPO 단계에서 영상의 불일치·점수·선택적 optical-flow 피드백으로 반복 수정 → 수집한 prompt pair로 rewriter 학습.
- **Reference video:** X. SSPO는 자체 생성 영상과 평가 피드백으로 prompt를 수정한다. retrieval은 caption 기반 표현을 보완하며, 외부 reference clip을 생성 목표로 사용하지 않는다. [§III–IV](https://arxiv.org/html/2510.20206v2#S4).
- **공간·시간 범위:** prompt를 고친 뒤 영상 전체를 재생성한다. ROI나 성공 프레임 선택 보존은 없다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 Stage2 SSPO Wan2.1 구현 기준). 각 round에서 기본 seed 1423으로 Generator를 새로 초기화하고 같은 shape의 noise에서 생성한다. 논문 자체에는 round별 seed/noise 정책이 미명시이다. [생성 함수](https://github.com/Vchitect/RAPO/blob/c7c2cb9b3e3f5fa073d05528232ed0e28d1a7912/examples/Stage2_SSPO/phyaware_wan2.1.py#L96-L118), [반복 호출](https://github.com/Vchitect/RAPO/blob/c7c2cb9b3e3f5fa073d05528232ed0e28d1a7912/examples/Stage2_SSPO/phyaware_wan2.1.py#L304-L344).
- **History:** Feedback Memory가 이전 optimized prompt와 평가·수정 결과를 참조하므로 누적 텍스트 O. 과거 영상 프레임까지 누적 재입력하는 범위는 미명시이다.
- **비교 포인트:** history 사용 자체의 선행 연구. 우리 방법은 ROI가 표시된 시각 이력과 프레임별 수정 결정을 어떻게 연결하는지 구체화해야 한다.
- **근거:** [§4.1, Table 4, §4.2](https://arxiv.org/abs/2510.20206v2). 표는 논문의 전체 설계 기준이다.

### NEWTON

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. [arXiv 이력](https://arxiv.org/abs/2605.18396), [공식 프로젝트](https://newton026.github.io/newton/).
- **핵심 방법:** Planner가 물리 계산·keyframe 생성·prompt refinement 도구 선택 → Executor 실행 → Verifier가 의미·물리 점수 반환 → 여러 cycle 후 최고 점수 영상 선택.
- **Reference video:** X. 입력 query를 바탕으로 prompt·keyframe·물리 계산 결과를 구성하며, 주요 pipeline에 외부 reference clip은 없다. keyframe은 생성한 이미지 조건이다. [§4.1.1–4.1.3](https://arxiv.org/html/2605.18396v2#S4.SS1).
- **공간·시간 범위:** 생성 조건을 고쳐 영상 전체를 재생성한다. keyframe 생성은 기존 성공 프레임을 freeze하는 기능이 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 재계획으로 prompt·keyframe 등의 조건을 바꾸지만, 논문의 생성기에서 round별 seed·초기 noise를 어떻게 관리하는지는 미명시이다. [§4.1.2, §5.1](https://arxiv.org/html/2605.18396v2).
- **History:** reasoning·도구 입력/출력·점수를 누적하지만, **생성 비디오는 memory에서 명시적으로 제외**한다. 텍스트 O, 시각 X.
- **비교 포인트:** Generator는 고정하고 Planner를 학습한다. 도구 실행 이력과 실제 생성 영상의 시각 이력을 구분할 수 있다.
- **근거:** [§4.1.1–4.1.3, §4.2](https://arxiv.org/html/2605.18396v2).

### SPIRAL

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. 현재 제목은 *SPIRAL: Self-Evolving Action-Conditioned Video Generation via Reflective Planning Agents*. [arXiv 이력](https://arxiv.org/abs/2603.08403), [공식 프로젝트](https://yuyang-cloud.github.io/spiral/).
- **핵심 방법:** 목표를 action step으로 분해 → segment 생성·평가 → 실패한 지시문 수정·재생성 → 반복 실패 시 해당 step부터 이후 계획 변경.
- **Reference video:** X. 초기 visual state와 goal에서 시작하며, 이후 영상 문맥은 성공 판정을 받은 자체 생성 segment의 memory이다. 외부 목표 clip은 입력하지 않는다. [§3.1–3.3, Appendix B.4](https://arxiv.org/html/2603.08403v3#S3.SS1).
- **공간·시간 범위:** 실패 segment의 공간 전체를 다시 만들고 성공한 앞 구간은 유지한다. 공간 X, 시간 O이며 같은 clip 내부의 프레임별 freeze와 구분한다.
- **시드·초기 noise:** 시드 — / 초기 noise —. inference retry의 정책은 미명시이다. shared initial noise 조건은 GRPO 학습의 후보 group에 관한 것이므로, 추론 시 refinement에도 noise를 고정한다고 해석하지 않았다. [Appendix B.4–B.5](https://arxiv.org/html/2603.08403v3#A2.SS4).
- **History:** Planner는 성공한 과거 segment의 시각·의미 이력과 실패 시도·진단을 사용한다. Critic은 현재 segment를 평가한다. 시간적으로 앞선 실행 이력이 중심이므로 두 history 축은 △.
- **비교 포인트:** 시간 선택 수정·시각 이력의 중요한 선행 연구. 동일 clip의 수정본 비교 및 ROI 타당성 판단과 대비해야 한다. 별도의 agent/generator 학습도 다룬다.
- **근거:** [§3.1–3.5, Appendix B.1/B.3/B.4, F.2](https://arxiv.org/html/2603.08403v3).

### CoAgent

- **발표:** 최초 공개 **2025**. **학회 채택 미확인**. *CoAgent: Collaborative Planning and Consistency Agent for Coherent Video Generation*. [arXiv 이력](https://arxiv.org/abs/2512.22536).
- **핵심 방법:** storyboard를 shot으로 분해 → 객체 상태 memory를 참조해 생성 → VLM 검증 → 의미 오류이면 storyboard 수정, 외관 오류이면 첫 프레임·양끝 프레임 조건 모드로 변경 → 해당 shot 재생성.
- **Reference video:** X. concept·storyboard로 시작하며, 직전 shot과 appearance memory는 자체 생성 결과에서 갱신된다. 양끝 frame 조건도 외부 reference video와 구분한다. [§3.1, §3.3–3.5](https://arxiv.org/html/2512.22536v1#S3.SS1).
- **공간·시간 범위:** 선택한 shot 전체를 다시 생성한다. 공간 X, 시간 O이며 shot 내부의 프레임별 freeze는 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 실패 shot의 description이나 생성 모드를 변경하지만, 재생성 사이 seed·시작 noise 처리 방식은 명시하지 않는다. [§3.5 Algorithm 2](https://arxiv.org/html/2512.22536v1#S3.SS5).
- **History:** 객체 속성과 appearance embedding을 shot 간 memory로 유지한다. 이는 같은 shot의 모든 실패 prompt·영상 이력이 아닌 **장면 간 상태 기억**이므로 두 축은 △.
- **비교 포인트:** 검증 결과가 prompt뿐 아니라 생성 조건 모드 선택으로 이어진다. 객체 시각 memory와 ROI가 표시된 수정 history의 역할 차이를 비교할 수 있다.
- **근거:** [§3.3–3.5, Algorithm 1–2](https://arxiv.org/html/2512.22536v1). 수정 단위는 Algorithm 2의 shot 재생성 호출을 기준으로 판정했다.

### A²RD · 기본형

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *A²RD: Agentic Autoregressive Diffusion for Long Video Consistency*. [arXiv 이력](https://arxiv.org/abs/2605.06924), [Google Research 공식 목록](https://research.google/pubs/a2rd-agentic-autoregressive-diffusion-for-long-video-consistency-2/).
- **핵심 방법:** 관련 장면 상태·시각 자료 검색 → segment 생성 → 일관성·품질 평가 → 성공·실패 사례를 검색해 prompt 개선(MAPO) → 필요하면 ending frame도 편집한 뒤 segment 재생성.
- **Reference video:** X. 사용자 제공 reference frame은 허용하지만 이미지 조건이며, video memory에는 자체 합성한 이전 segment를 저장한다. 외부 reference clip을 입력받는 설정과 구분한다. [§3.1–3.2](https://arxiv.org/html/2605.06924v1#S3.SS1).
- **공간·시간 범위:** video 수정은 segment 전체를 재생성하므로 공간 X, 시간 O. boundary image 편집은 video ROI만 수정하거나 성공 프레임을 freeze하는 절차와 구분한다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 기본형의 segment self-improvement에서 prompt·ending frame 수정은 설명하지만, 재생성 사이 seed·시작 noise 정책은 확인되지 않는다. [§3.3, Appendix C](https://arxiv.org/html/2605.06924v1#S3.SS3).
- **History:** 텍스트는 사전·온라인 사례 DB에서 관련 성공·실패 사례를 검색한다. 시각은 관련 과거 segment 영상·keyframe을 참조한다. 동일 segment의 전체 수정 archive 입력과 달라 두 축은 △.
- **비교 포인트:** **기본 A²RD만** 이 표의 대상이다. A²RD-Parallel은 video self-improvement를 생략한다. 전체 history 입력과 관련 사례 검색의 차이도 비교할 수 있다.
- **근거:** [§3.1–3.4, Eq. 9–10, MAPO 문단](https://arxiv.org/html/2605.06924v1).

### VQQA

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *VQQA: An Agentic Approach for Video Evaluation and Quality Improvement*. [arXiv 이력](https://arxiv.org/abs/2603.12310), [공식 프로젝트](https://yiwen-song.github.io/vqqa/).
- **핵심 방법:** 입력에 맞는 평가 질문 생성 → 영상의 답변·점수·이유 수집 → 낮은 점수의 QA로 prompt 수정 → 영상 재생성. 최종 후보는 원래 prompt에 대한 평가로 선택한다.
- **Reference video:** X(원문의 T2V/I2V refinement 기준). 원래 prompt·reference image를 조건으로 자체 생성 영상을 개선하며, 외부 reference clip을 사용한 설정은 제시하지 않는다. [§3.1–3.2, Appendix C.1.2](https://arxiv.org/html/2603.12310v1#S3.SS1).
- **공간·시간 범위:** 세부 질문으로 오류를 찾지만 생성은 영상 전체에 적용한다. 국소 QA는 공간 mask나 프레임 선택 수정이 아니다.
- **시드·초기 noise:** 시드 O / 초기 noise O(CogVideoX-5B·I2V 실험 기준). 첫 생성은 고정 seed, 이후 모든 refinement는 새 seed를 균등 추출한다. 초기 noise O는 이 seed 정책과 사용 라이브러리의 Gaussian latent 생성 동작을 대조한 판정이다. seed를 받지 않는 논문의 Veo 3.1 API 실험에는 일반화하지 않는다. [Appendix C.1–C.2](https://arxiv.org/html/2603.12310v1#A3.SS2), [공식 CogVideoX sampler](https://github.com/huggingface/diffusers/blob/0f252be0ed42006c125ef4429156cb13ae6c1d60/src/diffusers/pipelines/cogvideo/pipeline_cogvideox.py#L325-L349).
- **History:** prompt 템플릿은 과거 prompt·VQA·점수의 **선택적 텍스트 history**를 지원하므로 △. 보존 길이는 미명시이며, 이전 영상들을 다음 refinement에 누적 재입력하는 기능은 정의하지 않아 시각 X.
- **비교 포인트:** 질문 기반 피드백과 선택적 history를 결합한다. 최종 후보 pool의 보관·선정은 다음 refinement의 시각 history 입력과 구분한다.
- **근거:** [§3.1.2 Eq. 5, §3.2](https://arxiv.org/html/2603.12310v1), [PDF Appendix F.3, pp. 42–48의 prompt 템플릿](https://arxiv.org/pdf/2603.12310v1). HTML에 빠진 history 세부 내용은 PDF로 확인했다.

## 우리 아이디어와의 비교

- **핵심 흐름:** P0·V0 → prompt 수정 후 P1·V1 → 두 영상으로 ROI 계산 → ROI가 표시된 프레임들과 prompt history를 GPT에 입력.
- **Reference video:** X(현재 설계안 기준). V0·V1은 자체 생성한 수정 대상과 history이며, 별도의 외부 목표 영상은 입력으로 두지 않는다.
- **공간·시간 범위:** 잘된 프레임은 통과·freeze. 실패 프레임은 ROI가 적절하면 region만, 부적절하면 해당 프레임 전체에 refined prompt를 적용.
- **시드·초기 noise:** 두 축 모두 아직 미정(—/—). 두 영상의 차이를 ROI로 해석하려면 prompt 변경과 noise 변경의 영향을 구분할 수 있도록, round 간 seed 정책과 실제 초기 noise 재사용 여부를 각각 명시해야 한다.
- **History:** 같은 clip의 prompt와 ROI 표시 영상 이력을 함께 사용한다는 설계. 모든 iteration을 넣을지, 대표 이력만 선택할지는 명시해야 한다.
- **비교 포인트:** **두 iteration 기반 ROI + 동일 clip의 시각적 수정 이력 + 프레임별 accept/freeze + ROI 검증에 따른 수정 범위 전환**의 결합.
- **선행 연구와의 관계:** RAPO++·NEWTON·P-Flow는 누적 텍스트 이력, VQQA는 선택적 텍스트 history, SPIRAL·CoAgent·A²RD·CineCrew는 segment·shot 선택 수정이나 장면 간 기억을 사용한다. 특히 P-Flow는 누적 텍스트와 최근 두 생성 영상의 비교를, CineCrew는 실패 clip 재시도와 Rulebook 누적을 결합한다. history나 시간 선택 자체만으로 차별점을 주장하기는 어렵다.
- **구체화할 사항:** freeze가 픽셀·latent·noise 중 무엇을 고정하는지 정의해야 한다. 두 영상 간 변화가 실제 오류 위치와 항상 같지는 않으므로 ROI 제안과 타당성 검증을 분리해야 한다.

## 추가로 둘 만한 축

- **수정 위치의 선택·검증 방식 — 우선 추천**
	- **구분:** 현재 영상의 객체 검출·분할 / iteration 간 차이 기반 ROI / 제안 ROI의 오류 포함 여부 재검증.
	- **비교 효과:** VideoRepair의 객체 mask와 우리 방법의 변화 기반 ROI·타당성 판단을 직접 비교한다.
- **성공한 결과의 보존 방식 — 우선 추천**
	- **구분:** noise·prompt로 보존 유도 / 성공 shot·segment 재사용 / 프레임·latent의 명시적 freeze.
	- **비교 효과:** 공간·시간 선택 여부만으로 드러나지 않는 보존 강도를 구분한다.
- **반복이 이루어지는 단계 — 우선 추천**
	- **구분:** 완성 영상의 피드백 기반 재생성 / 생성 결과를 이용한 초기 noise 재최적화 / sampling 내부 latent 보정·후보 탐색 / 학습용 데이터 구축.
	- **비교 효과:** 모두 iterative refinement라고 불려도 수정하는 상태와 반복 시점이 다르다. 현재 표는 앞의 두 범위를 중심으로 비교한다.
- **수정 대상**
	- **구분:** prompt / layout·guidance / 궤적·카메라 / noise·latent / 기존 video / boundary image.
	- **비교 효과:** VideoTPO, GenMAC, MotionAgent, VideoAgent, A²RD의 실제 개입 지점을 비교한다.
- **피드백의 판단 단위**
	- **구분:** 영상 전체 점수 / 질문·객체별 진단 / shot·segment 판정 / frame·ROI별 판정.
	- **비교 효과:** 오류를 세밀하게 진단하는 것과 그 위치만 수정하는 것을 구분한다.
- **History의 범위·사용 방식**
	- **구분:** 직전 상태 / 같은 clip의 누적 수정본 / 장면 간 상태 기억 / 관련 성공·실패 사례 검색.
	- **비교 효과:** 현재 표의 O·△를 보완한다. 같은 ‘memory’라도 무엇을 저장하고 다음 수정의 어느 단계에 넣는지가 다르다.
