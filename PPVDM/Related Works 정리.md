## 분류 요약

**O**는 해당 기능을 사용함, **X**는 사용하지 않음, **△**는 다른 범위의 history를 사용함, **—**는 원문에 누적 입력 범위가 명확히 제시되지 않음을 뜻한다. 우리 방법의 O는 현재 설계 기준이다.

| 논문 | 공간 국소 수정 | 시간 선택 수정 | 누적 텍스트 이력 | 누적 시각 이력 |
|---|:---:|:---:|:---:|:---:|
| **PhyT2V** | X | X | — | — |
| **VideoTPO** | X | X | X | X |
| **RAPO++** | X | X | O | — |
| **VideoRepair** | O | X | — | — |
| GenMAC | X | X | X | X |
| VISTA | X | X | X | X |
| PQSG refinement | X | X | — | — |
| NEWTON | X | X | O | X |
| VideoAgent | X | X | X | X |
| Bootstrapping Physics-Grounded… | X | X | — | — |
| SPIRAL | X | O | △ | △ |
| **우리 방법 · 설계안** | **O** | **O** | **O** | **O** |

**공간 국소 수정**은 mask 등으로 특정 영역에 수정 제어를 적용하는지를 뜻한다. 영역 밖 픽셀이 정확히 고정된다는 의미는 아니다. **시간 선택 수정**은 실패한 프레임이나 segment를 골라 다시 생성하는지를 뜻하며, VLM이 일부 프레임만 관찰하거나 입력 첫 프레임을 유지하는 것은 포함하지 않는다.

**누적 이력**은 여러 과거 결과·피드백을 다음 수정에 참조하는 경우를 뜻한다. 직전 prompt·영상 하나를 이어 쓰거나 최선 후보 하나를 보관하는 것은 X로 구분했다. 시각 이력은 과거에 생성한 영상·프레임을 뜻한다. SPIRAL의 △는 같은 clip의 수정 이력 대신, 시간적으로 앞선 성공 segment의 이력을 사용한다는 뜻이다.

조사 기준일은 2026-10-13이다. 생성 결과를 평가한 뒤 수정하는 추론 시 반복 개선을 비교하며, 단일 diffusion 생성 과정 내부의 반복은 제외했다.

## 핵심 4편

### PhyT2V

**물리적 오류를 설명하고 prompt를 고쳐 영상 전체를 다시 만드는 방법이다.** 현재 영상의 caption과 prompt 사이의 불일치, 관련 물리 법칙, 평가 점수를 LLM에 제공하고, 단계적 추론과 상위 물리 원칙 재검토를 거쳐 다음 prompt를 만든다. 공간 ROI나 성공 프레임을 따로 고정하지 않는다. 수정된 prompt는 다음 라운드로 이어지지만, 과거 라운드별 prompt·평가·영상을 누적 입력하는 범위는 원문에 명확히 제시되지 않아 표에서는 —로 남겼다. [원문: §3, Fig. 6, §3.2](https://arxiv.org/abs/2412.00596v2)

### VideoTPO

**두 생성 후보를 비교한 피드백으로 공통 prompt를 반복 최적화한다.** 같은 입력 이미지와 prompt로 만든 두 후보의 장단점을 VLM이 textual loss로 표현하고, 이를 prompt의 수정 방향인 textual gradient로 바꿔 다음 prompt를 만든다. 매번 생성 구간 전체를 다시 만들며, 갱신에는 현재 후보 쌍·prompt·loss·gradient를 사용한다. 공개 Wan 구현에서 첫·마지막 프레임을 뽑는 것은 평가를 위한 관찰 방식이지 그 프레임만 수정하는 기능이 아니다. VideoTPO는 별도 논문 제목이 아니라 **TiViBench**에서 제안한 방법명이다. [원문: §4, Eq. 1–3, Appendix D.1](https://arxiv.org/abs/2511.13704v2), [공식 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L345)

### RAPO++

**과거 수정·평가 이력을 활용해 prompt를 개선하고 영상 전체를 재생성한다.** 검색과 문장 정리로 초기 prompt를 만든 뒤, SSPO 단계에서 영상의 불일치 평가·점수·선택적 optical-flow 피드백으로 prompt를 고친다. Feedback Memory는 이전 optimized prompt와 평가·수정 결과를 참조하므로 누적 텍스트 이력은 O이다. 다만 과거 영상 프레임까지 모두 다시 입력하는지는 명시하지 않는다. 전체 방법에는 수집한 prompt pair로 rewriter를 학습하는 Stage 3도 포함된다. [원문: §4.1, Table 4, §4.2](https://arxiv.org/abs/2510.20206v2)

### VideoRepair

**잘 생성된 객체를 보존하도록 유도하면서, 잘못된 영역을 중심으로 수정한다.** 객체별 질문으로 정합성을 평가하고, pointing model과 segmentation으로 보존할 객체의 mask를 만든다. 나머지 영역에는 새 noise와 refinement prompt를 적용하지만 보존 영역도 함께 denoising하므로, 영역 밖 픽셀의 hard freeze는 아니다. 시간축 전체를 처리하며 성공 프레임을 따로 골라 고정하지 않는다. 현재 영상·QA·직전 선택 결과를 사용하고, 전체 수정 이력의 누적 입력 범위는 미명시다. [원문: §2.1–2.3, Appendix E.1](https://aclanthology.org/2026.findings-acl.1817.pdf)

이 방법에도 **영상 정합성 점수가 0이면 전체 재생성하고, 1이면 중단하는 분기**가 있다. 따라서 우리 방법과 비교할 때는 단순히 ‘국소 수정과 전체 재생성을 함께 쓴다’는 점보다, **프레임별 ROI 타당성을 검증해 region/whole-frame을 선택한다**는 점을 구분해야 한다. 최종 학회 제목은 *Self-Correcting Text-to-Video Generation with Misalignment Detection and Localized Refinement*이다. [전역 분기 근거: §2.1](https://aclanthology.org/2026.findings-acl.1817.pdf#page=3)

## 추가 후보

### GenMAC

**여러 agent가 prompt뿐 아니라 객체 배치와 guidance까지 다시 설계한다.** prompt를 프레임별 bounding box로 바꿔 생성한 뒤, 검증·제안·수정·구조화 agent가 오류를 고친다. 객체 box로 attention을 유도하지만 기존 영역이나 프레임을 고정하는 방식은 아니므로 공간·시간 모두 전체 재생성에 해당한다. 수정 agent는 현재 영상과 직전 iteration의 구조화된 설계를 사용하며, 전체 과거 이력을 누적 입력하는 방식은 아니다. [원문: §3.1–3.3, Appendix A Eq. A3–A6](https://arxiv.org/html/2412.04440v1)

### VISTA

**여러 후보 중 최선의 영상을 고르고, 그 영상에 대한 critique로 prompt를 개선한다.** 시각·오디오·맥락을 평가해 새 후보를 생성하고 tournament 방식으로 다시 비교한다. 영상 전체를 재생성하며, 현재 champion·prompt·critique와 최선 후보의 유지가 핵심이다. 과거 여러 시도의 이력을 함께 참조하는 방식은 아니므로, 표의 누적 이력은 X이다. [원문: §2.1–2.2, Algorithm 1](https://arxiv.org/html/2510.15831v1)

### PQSG의 refinement 실험

**객체→행동→물리 질문 그래프로 오류를 진단하고, 그 결과로 prompt를 수정한다.** 주목적은 평가이지만 §5.5에서는 질문별 피드백을 GPT에 주어 prompt를 고치고 Wan으로 새 영상을 생성·재평가하는 실험을 제시한다. 질문별로 오류를 세밀하게 찾더라도 공간 mask나 프레임 선택 수정으로 연결하는 것은 아니어서 전체 재생성으로 분류했다. 누적 history의 입력 범위는 미명시다. [원문: §3, §5.5, Fig. 4](https://arxiv.org/html/2606.25306v1)

### NEWTON

**물리 계산·keyframe 생성·prompt refinement 도구를 조합해 생성 조건을 개선한다.** Planner가 도구를 선택하고 Verifier가 의미 정합성과 물리 점수를 반환하면, 여러 cycle을 거쳐 최고 점수 영상을 선택한다. reasoning·도구 입력/출력·점수는 누적하지만, **생성 비디오는 memory에서 명시적으로 제외**한다. keyframe은 새 영상의 생성 조건이며 기존 성공 프레임의 freeze가 아니다. Generator는 고정하고 Planner를 학습한다. [원문: §4.1.1–4.1.3, §4.2](https://arxiv.org/html/2605.18396v2)

### VideoAgent

**직전 생성 비디오 자체를 입력으로 받아 다음 비디오를 보정한다.** 로봇 행동을 위한 video plan을 학습된 self-conditioning 모델로 반복 개선하고, VLM이 수용 여부를 판단한다. Suggestive 변형은 언어 피드백도 보정에 활용한다. plan 전체를 수정하며 직전 비디오 하나를 참조하는 구조이므로, 공간·시간 선택 수정과 누적 history는 X로 분류했다. 동명의 video understanding 논문과 구분해야 한다. [원문: §3.1–3.2, Eq. 12–13, §4.3.1](https://arxiv.org/html/2410.10076v3)

### Bootstrapping Physics-Grounded Video Generation

**입력 영상 뒤에 이어질 내용을 생성하고, 물리 오류 피드백으로 prompt를 반복 수정한다.** VLM이 입력 prefix에서 물리 단서를 포함한 미래 서술을 만들면 LLM이 생성 prompt로 정리하고, 생성된 continuation을 다시 진단한다. 입력 3초를 유지하면서 5초 continuation 전체를 재생성하는 설정이므로, 성공한 출력 프레임을 선택해 freeze하는 방법은 아니다. 현재 결과에서 다음 prompt를 만드는 과정은 설명하지만 누적 history의 입력 범위는 미명시다. [원문: §2, Step 1–3](https://arxiv.org/html/2511.20280v1)

### SPIRAL

**실패한 action segment만 다시 생성하고, 성공한 앞 구간은 유지한다.** 목표를 action step으로 나눠 순차 생성하며, 실패하면 해당 지시문을 수정하고 반복 실패 시 그 step부터 이후 계획을 바꾼다. 선택 segment의 공간 전체를 다시 만들기 때문에 공간 국소 수정은 X, 시간 선택 수정은 O이다. Planner는 성공한 과거 segment의 시각·의미 이력과 실패 시도·진단을 사용하지만 Critic은 현재 segment를 평가한다. 이는 **같은 clip의 여러 수정본을 비교하는 history와 다른 시간적 실행 이력**이므로 표에서는 △로 구분했다. 별도의 agent/generator 학습과 GRPO도 다룬다. [원문: §3.1–3.5, Appendix B.1/B.3/B.4, F.2](https://arxiv.org/html/2603.08403v3)

## 우리 아이디어와의 비교

우리 방법은 초기 영상과 prompt 수정 후 영상으로 ROI를 계산하고, **ROI가 표시된 프레임들과 prompt의 수정 이력**을 GPT에 제공하는 설계다. 잘된 프레임은 통과시켜 freeze하고, 실패 프레임은 ROI가 적절한지 다시 판단한다. 적절하면 region에만 refined prompt를 적용하고, 부적절하면 해당 프레임 전체를 수정한다.

비교의 중심은 **두 iteration 기반 ROI + 동일 clip의 시각적 수정 이력 + 프레임별 accept/freeze + ROI 검증에 따른 수정 범위 전환**이다. RAPO++에는 누적 텍스트 피드백이 있고 SPIRAL에는 선택적 segment 재생성과 시각 이력이 있으므로, history 사용이나 시간 선택 수정 자체만으로 차별점을 주장하기는 어렵다. 현재 조사에서는 위 요소들의 결합과 판단 단위를 중심으로 대비하는 것이 적절하다.

설계를 구체화할 때는 freeze가 픽셀·latent·noise 중 무엇을 고정하는지, history가 모든 iteration을 포함하는지 또는 일부만 선택하는지를 명시해야 한다. 또한 두 영상 사이의 변화가 실제 오류 위치와 항상 같지는 않으므로, **ROI를 제안하는 과정과 그 타당성을 검증하는 과정**을 구분하는 것이 중요하다.

## 추가로 둘 만한 축

**수정 위치의 선택·검증 방식**을 우선 추천한다. 현재 영상에서 객체를 찾아 mask를 만드는지, 두 iteration의 변화로 ROI를 만드는지, 제안된 ROI가 오류를 제대로 포함하는지 다시 검증하는지를 비교할 수 있다. 우리 방법의 region/whole-frame 전환 규칙을 설명하기에 가장 직접적인 축이다.

**성공한 결과의 보존 방식**도 우선 추천한다. VideoRepair처럼 noise와 prompt로 보존을 유도하는 것, SPIRAL처럼 성공 segment를 재사용하는 것, 프레임을 명시적으로 freeze하는 것은 서로 다르다. 공간·시간 선택 여부만으로 드러나지 않는 차이를 보여 준다.

**수정 대상**은 prompt, layout·guidance, noise·latent, 기존 video 등으로 나눌 수 있다. VideoTPO의 prompt 최적화, GenMAC의 배치 제어, VideoAgent의 video conditioning을 구분하는 데 유용하다.

**피드백의 판단 단위**는 영상 전체 점수, 질문·객체별 진단, segment 판정, frame·ROI별 판정으로 나눌 수 있다. PQSG의 질문별 진단, NEWTON의 영상 점수, 우리 방법의 프레임 성공 여부와 ROI 타당성 판단을 대비할 수 있다. 표를 간결하게 유지하려면 수정 대상과 피드백 단위는 위의 논문별 설명에 남겨도 충분하다.
