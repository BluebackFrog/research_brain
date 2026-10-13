- 조사 기준: **2026-10-13**. 아래는 생성 결과를 평가한 뒤 다시 수정하는 **inference-time iterative refinement** 중심의 선별 목록이다. 단일 생성 과정 안의 diffusion timestep 반복이나 학습 시의 반복만을 뜻하는 연구는 중심 비교에서 제외했다.
- 분류는 **논문에 제시된 방법**을 기준으로 한다. 아래의 ‘우리 방법’은 제안된 설계이며, 구현·실험으로 검증된 결과를 뜻하지 않는다.

## 분류 축의 정의

| 축 | 분류 기준 | 구분할 점 |
|---|---|---|
| **① 공간적 수정 범위** | 전체 화면 / 특정 객체·영역 / 상황에 따라 영역↔전체 프레임 전환 | 객체별 prompt나 bounding box가 있어도, 영상 전체를 다시 만들면 전체 재생성이다. 영역을 중심으로 수정하더라도 영역 밖이 정확히 고정되는지는 별도 확인한다. |
| **② 시간적 수정 범위** | 생성 구간 전체 / 선택한 프레임 / 선택한 segment | VLM이 일부 프레임만 **관찰**하는 것과 그 프레임만 **수정**하는 것은 다르다. 고정된 입력 첫 프레임·prefix도 성공한 출력 프레임을 판정해 freeze하는 것과 구분한다. |
| **③ Refinement history** | 현재·직전 상태 / best-so-far / 누적 텍스트·점수 / 누적 시각 정보 | 저장 여부보다 **다음 수정 결정을 할 때 실제로 무엇을 입력하는가**를 본다. ‘history 사용’만으로 모든 과거 비디오를 다시 본다고 해석하지 않는다. |

- History에는 **동일 clip을 수정해 온 iteration 이력**과 **긴 영상의 시간적으로 앞선 segment 이력**이 있다. 두 종류를 구분하고, Planner·Critic·Generator가 서로 다른 입력을 받으면 역할별로 적었다.
- **미명시**는 조사한 원문에 전체 history의 입력 범위가 충분히 설명되지 않았다는 뜻이다. history가 없다는 단정이 아니다.

## 포함 예정인 핵심 4편

| 논문 | 내용과 핵심 method | ① 공간 범위 | ② 시간 범위 | ③ History 사용 |
|---|---|---|---|---|
| **[PhyT2V](https://arxiv.org/abs/2412.00596v2)** (2024; CVPR 2025) | 현재 영상의 caption과 prompt 사이의 불일치, 관련 물리 법칙, 평가 점수를 LLM에 제공한다. 단계적 추론과 상위 물리 원칙 재검토로 prompt를 고쳐 영상을 다시 생성한다. | **전체 재생성**. 전역 prompt를 수정하며 ROI mask는 사용하지 않는다. | **전체**. 성공 프레임을 선별하여 고정하는 절차는 없다. | **현재 라운드 중심**: 현재 caption·불일치·점수·prompt. 수정 prompt는 이어 쓰지만, 과거 라운드별 영상·피드백을 모두 재입력하는 과정은 명시하지 않는다. |
| **[VideoTPO / TiViBench](https://arxiv.org/abs/2511.13704v2)** (2025; CVPR 2026) | 같은 입력 이미지·prompt로 생성한 두 후보를 VLM이 비교해 textual loss를 만든다. 이를 prompt의 수정 방향인 textual gradient로 바꿔 하나의 공통 prompt를 갱신하고 반복 생성한다. | **전체 재생성**. 전역 prompt 최적화이며 ROI mask는 없다. | **생성 구간 전체**. I2V 입력 이미지를 유지하는 것과 성공한 출력 프레임의 freeze는 다르다. | **현재 후보 쌍 + 현재 prompt·loss·gradient**. §4의 갱신식은 누적 history를 입력하지 않으며 공식 기본 구현도 gradient memory를 켜지 않는다. |
| **[RAPO++](https://arxiv.org/abs/2510.20206v2)** (2025; 확인본 v2 2026) | 검색·문장 정리로 prompt를 만든 뒤 SSPO가 생성 영상의 불일치 평가·점수·선택적 optical-flow 피드백으로 prompt를 반복 개선한다. 누적 feedback memory를 참조하며, 별도 Stage 3에서는 수집한 prompt pair로 rewriter를 학습한다. | **전체 재생성**. SSPO는 전역 prompt를 재작성한다. | **전체**. 시간적 일관성을 평가하지만 오류 프레임만 선택해 재생성하는 것은 아니다. | **누적 텍스트·평가 history**. 이전 optimized prompt, 평가·수정 결과, 현재·과거 feedback을 참조한다. **모든 과거 영상 프레임의 직접 재입력은 명시하지 않는다.** |
| **[VideoRepair](https://aclanthology.org/2026.findings-acl.1817/)** (2024; ACL Findings 2026) | 객체별 질문으로 정합성을 평가하고, 잘 생성된 객체를 pointing model과 segmentation으로 찾아 보존 mask를 만든다. 나머지 영역의 noise를 다시 뽑고 지역별 prompt를 적용해 보존·수정 영역을 함께 생성한 뒤 좋은 후보를 다음 iteration에 사용한다. | **국소 mask 중심 + 전역 재생성 분기**. 다만 보존 영역도 공동 denoising하므로 **영역 밖 hard freeze는 아니다**. | **전 시간축 처리**. 프레임에 대응하는 mask는 있으나 성공한 프레임 전체를 판정해 freeze하는 절차는 아니다. | **현재 영상·QA·원문 prompt 중심**, 직전 선택 결과의 상태를 계승한다. 전체 과거 prompt·feedback·visual history의 누적 입력은 **미명시**. |

- **VideoRepair의 전역 분기:** 현재 영상의 정합성 점수가 0이면 새 seed로 전체 재생성하고, 1이면 중단한다. 따라서 ‘국소 수정이 안 되면 더 넓게 재생성한다’는 발상만으로 차별화하기보다, 우리 설계의 **프레임별 ROI 타당성 검증과 region/whole-frame 선택**을 대조해야 한다. [§2.1](https://aclanthology.org/2026.findings-acl.1817.pdf#page=3)
- **관찰과 수정의 구분:** VideoTPO 공개 Wan 구현에서 첫·마지막 프레임을 뽑아 평가하는 것은 관찰량을 줄이는 과정이다. 생성 범위는 여전히 전체 구간이다. [공식 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L345)

## 함께 비교할 추가 후보

| 논문 | 내용과 핵심 method | ① 공간 범위 | ② 시간 범위 | ③ History 사용 |
|---|---|---|---|---|
| **[GenMAC](https://arxiv.org/html/2412.04440v1)** (2024) | prompt를 프레임별 객체 배치로 바꿔 생성한 뒤, 검증·제안·수정·구조화 agent가 **prompt, bounding box, guidance scale**을 다시 설계한다. 오류 유형에 맞는 correction agent를 선택한다. | **전체 재생성**. 객체 box로 attention을 유도하지만 기존 box 밖 픽셀을 고정하는 ROI repair는 아니다. | **전체**. 프레임별 layout 제어이며 성공 프레임을 선별 freeze하지 않는다. | **현재 영상 + 직전 iteration의 구조화된 설계**. Appendix A의 Eq. A5는 직전 출력을 명시하며, 전체 과거 영상 archive 입력은 제시하지 않는다. |
| **[VISTA](https://arxiv.org/html/2510.15831v1)** (2025) | 여러 생성 후보를 tournament로 비교해 champion을 고른다. champion에 대한 시각·오디오·맥락 critique로 prompt를 고쳐 새 후보를 만들고 다시 경쟁시킨다. | **전체 재생성**. | **전체**. 시간순 장면 prompt가 있어도 오류 프레임만 편집하는 것은 아니다. | **현재 champion·prompt·critique + best-so-far 유지**. Algorithm 1은 모든 과거 시도의 직접 입력 대신 최선 후보를 다음 라운드에 남긴다. |
| **[PQSG의 refinement 실험](https://arxiv.org/html/2606.25306v1#S5.SS5)** (2026) | 객체→행동→물리 질문 그래프로 실패 원인을 평가한다. §5.5에서는 그 피드백으로 GPT가 prompt를 수정하고 Wan으로 새 영상을 생성·재평가한다. | **전체 재생성**. 세밀한 질문별 진단이 공간 mask를 뜻하지는 않는다. | **전체**. | **현재 영상의 질문별 피드백을 사용**. 전체 과거 prompt·진단·영상을 누적 입력하는지는 **미명시**. 평가 논문의 응용 실험이라는 점을 구분한다. |
| **[NEWTON](https://arxiv.org/html/2605.18396v2)** (2026) | Planner가 물리 계산·keyframe 생성·prompt refinement 도구를 골라 조건을 구성한다. Verifier의 의미 정합성·물리 점수에 따라 여러 cycle을 수행하고 최고 점수 영상을 반환한다. | **전체 재생성**. | **전체**. 특정 시간의 keyframe은 생성 조건이며 기존 성공 프레임 freeze가 아니다. | **누적 reasoning·도구 입력/출력·평가 점수**. §4.1.2는 **생성 비디오를 memory에서 제외**한다고 명시한다. 영상 평가 결과는 점수로 전달된다. |
| **[VideoAgent](https://arxiv.org/html/2410.10076v3)** (2024; v3 2025) | 로봇 행동을 위한 video plan을 **직전 생성 비디오에 conditioning하는 보정 모델**로 반복 개선한다. VLM이 수용 여부를 판단하며 Suggestive 변형은 언어 피드백도 보정에 사용한다. | **생성 plan의 공간 전체**. ROI 제한 없음. | **생성 plan 전체**. 첫 관찰 프레임은 입력 조건이다. | **직전 비디오 + 현재 평가**. Eq. 12–13의 반복 보정은 모든 과거 비디오를 동시에 입력하는 방식이 아니다. 학습된 보정 모델을 사용한다. |
| **[Bootstrapping Physics-Grounded Video Generation…](https://arxiv.org/html/2511.20280v1)** (2025) | 입력 video prefix를 보고 VLM이 물리 단서를 포함한 미래 서술을 만들고 LLM이 생성 prompt로 정리한다. continuation의 물리 오류를 다시 진단해 prompt를 반복 수정한다. | **continuation의 공간 전체**. | **생성 continuation 전체**. 입력 3초 prefix가 고정되고 5초 continuation을 생성하는 설정으로, 출력 내 오류 프레임만 고르는 것은 아니다. | **현재 생성 결과 중심의 피드백**. §2의 점화식은 현재 결과로 다음 prompt를 만들며, 전체 대화·시각 history 보존 여부는 **미명시**. |
| **[SPIRAL](https://arxiv.org/html/2603.08403v3)** (2026) | 목표를 action step으로 나누고 실패한 step의 지시문을 수정해 해당 segment를 다시 생성한다. 반복 실패하면 그 step부터 계획을 바꾸고 성공한 prefix는 보존한다. | **선택 segment 내 공간 전체**. 여기서 국소 수정은 공간 ROI가 아니라 action step 단위다. | **선택 segment 재생성 + 성공 prefix 보존**. 긴 영상을 순차 구성하는 설정이며, 완성된 clip의 임의 프레임별 freeze와는 다르다. | **역할별 혼합**. Planner는 샘플링한 실행 history·실패 시도·진단을, Critic은 현재 segment를 사용한다. 성공 segment의 시각 이력은 누적하지만 동일 segment의 모든 실패 revision을 다시 보는지는 미명시다. |

- **가까운 비교 대상:** GenMAC은 수정 제어의 종류, PQSG는 물리 오류의 세부 진단, NEWTON은 memory의 내용, SPIRAL은 시간 선택과 성공 구간 보존을 비교하는 데 특히 유용하다. VISTA는 prompt 탐색·후보 보존, VideoAgent는 직전 비디오를 직접 입력하는 보정 방식, Bootstrapping은 물리 피드백 기반 continuation 개선을 대표한다.
- 학습 여부는 별도 메타데이터로 남기는 편이 좋다. 예를 들어 VideoAgent는 보정 모델을 학습하고, NEWTON은 Planner를 학습하며, SPIRAL은 inference loop와 별도로 agent/generator 학습 및 GRPO를 다룬다. RAPO++도 Stage 3의 rewriter 학습을 포함한다. 위 표는 이들의 **추론 시 수정 과정**을 비교한다.

## 우리 아이디어를 같은 축에 놓으면

| 내용과 핵심 method | ① 공간 범위 | ② 시간 범위 | ③ History 사용 |
|---|---|---|---|
| 초기 prompt·video와 수정 후 prompt·video를 얻고, 두 영상으로 ROI를 계산한다. GPT가 ROI 표시 프레임과 prompt 이력을 보고 성공 프레임의 통과 여부와 실패 프레임의 수정 범위를 정한다. | **적응적 region/whole-frame 수정**. ROI가 적절하면 해당 영역에 refined prompt를 적용하고, 부적절하면 해당 프레임 전체에 적용한다. | **실패 프레임 선택 수정 + 성공 프레임 freeze**를 목표로 한다. | **동일 clip의 여러 iteration에 걸친 ROI 표시 시각 정보 + prompt 이력**을 명시적으로 사용하려는 설계다. |

- 우리 방법의 history는 ‘과거 프레임 일부를 샘플링해 모든 iteration을 포함’하는 것인지, ‘선택한 iteration만 포함’하는 것인지도 기록해야 한다. 여러 iteration의 시각 정보를 본다는 것과 모든 원본 프레임을 입력한다는 것은 다르다.
- `freeze`는 구현 시 **픽셀, latent, noise 또는 조건 영상 중 무엇을 고정하는지** 명시해야 한다. 이를 정하기 전에는 ‘성공 프레임의 픽셀이 정확히 유지된다’고 단정하지 않는다.

## 추가로 권하는 분류 축

| 추가 축 | 분류 값 예시 | 우리 아이디어에서의 의미 | 비교에 도움이 되는 이유 |
|---|---|---|---|
| **④ 무엇을 수정하는가?** | prompt / layout·guidance / noise·latent·기존 video / 여러 도구가 만든 조건 | refined prompt를 선택 영역·프레임에 적용. 정확한 생성기 개입 방식은 구현에서 구체화할 항목이다. | 같은 전체 재생성이라도 VideoTPO의 prompt 최적화, GenMAC의 layout 제어, VideoAgent의 video conditioning은 다르다. |
| **⑤ 수정 위치를 어떻게 정하고, 그 선택을 검증하는가?** | 의미적 오류→객체 segmentation / iteration 간 변화 / 프레임·구간 평가; 선택 검증 유무와 범위 전환 규칙 | **두 iteration으로 ROI 제안 → ROI 타당성 재검토 → region 또는 whole-frame 선택** | VideoRepair의 객체 중심 분해와 대비할 수 있다. 특히 ‘오류가 있는가’와 ‘제안한 ROI가 오류를 제대로 포함하는가’를 별도 판단하는 점을 드러낸다. |
| **⑥ 성공한 내용을 어떻게 보존하는가?** | 재생성에 맡김 / 원본 조건·noise로 보존 유도 / mask·latent·합성으로 고정 / 성공 segment 재사용 | **성공 프레임 freeze**와 실패 프레임 내 수정 범위 제한 | VideoRepair의 보존 유도, SPIRAL의 성공 prefix 재사용, 프레임 단위 고정을 구분한다. 좁은 ROI를 쓰는 것만으로 영역 밖이 불변이라고 볼 수 없다. |
| **⑦ 피드백은 무엇을 어느 단위로 판단하는가?** | 의미 정합성·물리·시각 품질; 영상 전체 점수 / 질문·객체별 진단 / segment / frame·ROI별 판정 | **프레임 성공 여부 + ROI 선택 타당성**을 수정 제어에 연결 | 물리 목적이 같아도 PQSG의 질문별 진단, NEWTON의 영상 점수, 우리 설계의 수정 위치 판정은 다르다. |

- 최종 논문 표의 폭을 줄여야 한다면, 기존 3축에 **⑤ 위치 선택·검증**과 **⑥ 성공 결과 보존**을 우선 추가하는 것을 권한다. ④ 수정 대상과 ⑦ 피드백 단위는 method 설명에 넣어도 된다.
- 세 번째 축은 단순한 history 사용 O/X보다 **범위(직전·선별·누적) + 내용(텍스트·점수·시각) + 사용 주체**로 적는 편이 정확하다.

## Related works에서 강조할 비교 논리

- **전체 재생성 기반 refinement:** PhyT2V·VideoTPO·RAPO++ 등과 비교해, feedback으로 prompt를 개선하는 것에서 더 나아가 수정할 공간·시간 범위와 보존할 결과를 함께 결정한다는 방향으로 설명할 수 있다.
- **공간적 국소 수정과의 비교:** VideoRepair와는 ROI를 얻는 근거, ROI 자체의 타당성 검증, 실패 시 whole-frame으로 전환하는 규칙, 성공 프레임 보존 여부를 대조하는 것이 핵심이다.
- **시간 선택·memory와의 비교:** RAPO++의 누적 피드백과 SPIRAL의 성공 segment·시각 memory 때문에, ‘history 사용’이나 ‘시간적으로 일부만 다시 생성’ 자체를 새로운 기여라고 주장하기는 어렵다. 우리 설계의 비교 초점은 **동일 clip의 수정 이력 + 두 iteration 기반 ROI + 프레임별 accept/freeze + ROI 검증에 따른 수정 범위 전환**의 결합이다. 이는 현재 조사에 따른 positioning 제안이며 최초성의 확정은 아니다.
- 두 iteration 사이의 변화가 곧 오류 위치라는 보장은 없다. 따라서 ROI의 변화 검출 능력과 실제 오류를 포함하는 정도를 구분하는 것이 ⑤의 설득력을 높인다.

## 원문과 분류 근거

1. **PhyT2V: LLM-Guided Iterative Self-Refinement for Physics-Grounded Text-to-Video Generation.** [arXiv v2](https://arxiv.org/abs/2412.00596v2). PDF p.4–5, §3·Fig. 6과 §3.2는 prompt 갱신의 입력을 제시한다. 고정된 few-shot 예시는 같은 샘플의 refinement history가 아니다. [공식 prompt 구성](https://github.com/pittisl/PhyT2V/blob/7416e18da2c516caf6efb1e60dc142b6745e947b/prompts.py#L65-L101)도 현재 입력·물리 법칙·불일치·점수 중심이다.
2. **TiViBench: Benchmarking Think-in-Video Reasoning for Video Generative Models.** [arXiv v2](https://arxiv.org/abs/2511.13704v2). **VideoTPO는 이 논문의 §4에서 제안된 방법명**이다. PDF p.7 Eq. 1–3 및 Appendix D.1(p.22)이 현재 후보 쌍→loss→gradient→prompt update를 정의한다. [공식 optimizer](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/textgrad/optimizer/optimizer.py#L82-L123)의 기본 gradient memory는 0이다.
3. **RAPO++: Cross-Stage Prompt Optimization for Text-to-Video Generation via Data Alignment and Test-Time Scaling.** [arXiv v2](https://arxiv.org/abs/2510.20206v2). PDF p.6–7의 §4.1과 **Table 4**는 Feedback Memory와 이전 prompt·평가 기록을 입력하는 지침을 명시한다. §4.2는 rewriter 학습을 설명한다. 원래 RAPO와 구분하며, 표에서는 간소화된 공개 데모가 아닌 논문의 SSPO 설계를 기준으로 분류했다.
4. **Self-Correcting Text-to-Video Generation with Misalignment Detection and Localized Refinement.** [ACL Findings 2026 최종 원문](https://aclanthology.org/2026.findings-acl.1817.pdf). **VideoRepair의 최종 학회 제목**이며, 이전 preprint 제목은 *VideoRepair: Improving Text-to-Video Generation via Misalignment Evaluation and Localized Refinement*이다. §2.1–2.3(PDF p.3–5)은 평가·mask·noise 재사용과 공동 생성, Appendix E.1(p.17)은 반복 적용의 근거다. §2.3은 모든 픽셀의 재생성을 명시하므로 hard freeze로 분류하지 않았다.

5. **GenMAC: Compositional Text-to-Video Generation with Multi-Agent Collaboration.** [arXiv v1](https://arxiv.org/html/2412.04440v1). §3.1–3.3, Fig. 2–3은 생성·재설계 loop와 layout guidance, Appendix A의 Eq. A3–A6 및 설명은 agent별 입력과 직전 structured output의 사용 근거다.
6. **VISTA: A Test-Time Self-Improving Video Generation Agent.** [arXiv v1](https://arxiv.org/html/2510.15831v1). §2.1–2.2와 Algorithm 1의 champion 선택, critique, prompt update 및 이전 champion 유지 연산을 기준으로 분류했다.
7. **Physics Question Scene Graph: Fine-grained Evaluation of Physical Plausibility in Text-to-Video Generation.** [arXiv v1](https://arxiv.org/html/2606.25306v1). §3은 질문 그래프, §5.5·Fig. 4는 prompt 수정→새 영상 생성의 반복 실험이다. history 전체 입력 여부는 이 절에서 특정하지 않는다.
8. **NEWTON: Agentic Planning for Physically Grounded Video Generation.** [arXiv v2](https://arxiv.org/html/2605.18396v2). §4.1.1–4.1.3은 Planner·Executor·Verifier와 memory 계약을 설명하며, §4.1.2가 generated-video 제외를 명시한다. §4.2는 Planner 학습을 설명한다.
9. **VideoAgent: Self-Improving Video Generation for Embodied Planning.** [arXiv v3](https://arxiv.org/html/2410.10076v3), [공식 코드](https://github.com/Video-as-Agent/VideoAgent). §3.1–3.2·Fig. 2·Eq. 12–13은 self-conditioning refinement, §4.3.1은 VLM stopping과 Suggestive 변형의 근거다. 동명의 video understanding 연구와 구분한다.
10. **Bootstrapping Physics-Grounded Video Generation through VLM-Guided Iterative Self-Refinement.** [arXiv v1](https://arxiv.org/html/2511.20280v1). §2의 prefix/continuation 설정, Step 1–3과 prompt 갱신식을 기준으로 분류했다.
11. **SPIRAL: Self-Evolving Action-Conditioned Video Generation via Reflective Planning Agents.** [arXiv v3](https://arxiv.org/html/2603.08403v3). §3.1–3.4는 memory·수정 loop, Appendix B.1/B.3은 Planner와 Critic의 입력, B.4와 F.2는 실패 step 재생성과 성공 prefix 보존의 근거다. §3.5의 GRPO 학습은 위 추론 과정과 구분한다.
