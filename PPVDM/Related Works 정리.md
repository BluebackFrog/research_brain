## 분류 요약

- **조사 기준일:** 2026-10-14. 총 20편. 이번 추가 조사는 학회 채택을 우선해 본회의 4편과 워크숍 1편을 보강했다.
- **비교 범위:** 영상·shot·segment를 생성한 뒤 피드백이나 생성 결과를 활용해 다시 생성하는 추론 시 반복 개선. prompt 수정뿐 아니라 NoisEasier의 reward 기반 noise 최적화와 FreeInit의 clean latent 기반 재초기화를 포함한다. 생성 전 계획 수정, 학습 데이터 구축, 단일 sampling 내부에서만 수행하는 보정·탐색은 별도 범위로 남겼다.
- **연도 기준:** ‘최초 공개’는 arXiv v1 연도이며, 학회 발표 연도와 구분했다. ‘채택 미확인’은 확인한 1차 출처에서 채택 근거를 확보하지 못했다는 뜻이다.
- **기호:** O = 사용, X = 사용하지 않음, △ = 선택적 지원 또는 비교 범위가 다른 이력, — = 원문에서 누적 입력 범위 미명시. 우리 방법은 구현 검증 결과가 아닌 설계안이다.

| 논문 | 최초 공개 | 발표처·연도 | 공간 국소 수정 | 시간 선택 수정 | 누적 텍스트 이력 | 누적 시각 이력 |
|---|:---:|---|:---:|:---:|:---:|:---:|
| **PhyT2V** | 2024 | CVPR 2025 | X | X | — | — |
| **VideoTPO / TiViBench** | 2025 | CVPR 2026 | X | X | X | X |
| **RAPO++** | 2025 | 채택 미확인 | X | X | O | — |
| **VideoRepair** | 2024 | Findings of ACL 2026 | O | X | — | — |
| GenMAC | 2024 | AAAI 2026 | X | X | X | X |
| VISTA | 2025 | CVPR 2026 | X | X | X | X |
| PQSG refinement | 2026 | ECCV 2026 | X | X | — | — |
| NEWTON | 2026 | 채택 미확인 | X | X | O | X |
| VideoAgent | 2024 | RLBRew @ RLC 2025 워크숍 | X | X | X | X |
| Bootstrapping Physics-Grounded… | 2025 | ICCV 2025 워크숍 챌린지 보고 | X | X | — | — |
| SPIRAL | 2026 | 채택 미확인 | X | O | △ | △ |
| MotionAgent · rethinking | 2025 | ICCV 2025 | X | X | — | — |
| CoAgent | 2025 | 채택 미확인 | X | O | △ | △ |
| A²RD · 기본형 | 2026 | 채택 미확인 | X | O | △ | △ |
| VQQA | 2026 | 채택 미확인 | X | X | △ | X |
| P-Flow | 2026 | CVPR 2026 | X | X | O | X |
| CineCrew | 2026 | ECCV 2026 | X | O | O | △ |
| NoisEasier | 2026 | ECCV 2026 | X | X | X | X |
| FreeInit | 2023 | ECCV 2024 | X | X | X | X |
| AIGVE-MACS refinement | 2025 | CVPR 2026 VGBE 워크숍 | X | X | — | — |
| **우리 방법 · 설계안** | — | — | O | O | O | O |

- **공간 국소 수정:** 생성된 영상의 특정 영역에 mask 등으로 수정 제어를 적용하는지. ROI 밖 픽셀이 정확히 고정된다는 뜻은 아니다. 입력 객체를 검출하거나 생성 layout을 지정하는 것만으로는 O로 보지 않는다.
- **시간 선택 수정:** 실패한 프레임·shot·segment를 골라 다시 생성하는지. 평가 시 일부 프레임만 관찰하거나 I2V 입력 첫 프레임을 유지하는 것은 포함하지 않는다. SPIRAL·CoAgent·A²RD·CineCrew의 O는 segment·shot 단위이다.
- **누적 이력:** 다음 수정에 여러 과거 결과·피드백을 참조하는지. 직전 결과 하나 또는 best-so-far 하나만 이어 쓰면 X이다. O도 모든 iteration의 모든 정보를 빠짐없이 입력한다는 뜻은 아니며, 실제 범위는 각 논문의 History 항목에 적었다.
- **시각 이력:** 과거 생성 영상·프레임의 참조를 기준으로 한다. 다른 shot의 상태·시각 embedding이나 관련 사례 검색은 △로 구분한다. 최종 후보 선택을 위해 영상을 모아 두는 것과 다음 refinement에 다시 입력하는 것도 구분한다. 갱신된 noise·latent·optimizer state가 이전 연산의 영향을 계승하는 것만으로 누적 시각 이력 O로 보지 않는다.

## 핵심 4편

### PhyT2V

- **발표:** 최초 공개 **2024** → **CVPR 2025**. [arXiv 이력](https://arxiv.org/abs/2412.00596), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2025/html/Xue_PhyT2V_LLM-Guided_Iterative_Self-Refinement_for_Physics-Grounded_Text-to-Video_Generation_CVPR_2025_paper.html).
- **핵심 방법:** 현재 영상의 caption과 prompt 간 불일치·물리 법칙·평가 점수를 LLM에 제공 → 단계적 추론과 상위 물리 원칙 재검토 → prompt 수정 → 영상 재생성.
- **공간·시간 범위:** 영상 전체를 다시 생성한다. 공간 ROI 선택이나 성공 프레임 freeze는 없다.
- **History:** 수정 prompt는 다음 라운드로 이어지지만, 과거 라운드의 prompt·평가·영상을 얼마나 누적 입력하는지는 미명시이다.
- **비교 포인트:** 물리 피드백 기반 prompt 개선의 직접 비교 대상. 오류 진단을 공간·프레임별 수정 제어로 연결하는지는 별도 차이이다.
- **근거:** [§3, Fig. 6, §3.2](https://arxiv.org/abs/2412.00596v2).

### VideoTPO / TiViBench

- **발표:** 최초 공개 **2025** → **CVPR 2026**. VideoTPO는 *TiViBench: Benchmarking Think-in-Video Reasoning for Video Generation*의 방법명이다. [arXiv 이력](https://arxiv.org/abs/2511.13704), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Chen_TiViBench_Benchmarking_Think-in-Video_Reasoning_for_Video_Generation_CVPR_2026_paper.html).
- **핵심 방법:** 같은 입력 이미지·prompt로 만든 두 영상 후보를 비교 → 장단점을 textual loss로 표현 → prompt의 수정 방향인 textual gradient 계산 → 공통 prompt 갱신.
- **공간·시간 범위:** 생성 구간 전체를 다시 만든다. 공개 Wan 구현의 첫·마지막 프레임 추출은 평가용 관찰이며, 그 프레임만 수정하는 기능이 아니다.
- **History:** 현재 후보 쌍·prompt·loss·gradient를 사용한다. 공개 기본 설정은 gradient memory가 0이며, 여러 과거 시도를 누적 입력하지 않는다.
- **비교 포인트:** 두 후보의 비교를 prompt 최적화에 쓰지만, 두 iteration의 영상 차이로 수정 ROI를 계산하는 방식은 아니다.
- **근거:** [§4, Eq. 1–3, Appendix D.1](https://arxiv.org/abs/2511.13704v2), [공식 Wan 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L345).

### RAPO++

- **발표:** 최초 공개 **2025**. **학회 채택 미확인**. 원래 RAPO의 CVPR 2025와 구분한다. [arXiv 이력](https://arxiv.org/abs/2510.20206), [공식 저장소](https://github.com/Vchitect/RAPO).
- **핵심 방법:** 검색·문장 정리로 초기 prompt를 구성 → SSPO 단계에서 영상의 불일치·점수·선택적 optical-flow 피드백으로 반복 수정 → 수집한 prompt pair로 rewriter 학습.
- **공간·시간 범위:** prompt를 고친 뒤 영상 전체를 재생성한다. ROI나 성공 프레임 선택 보존은 없다.
- **History:** Feedback Memory가 이전 optimized prompt와 평가·수정 결과를 참조하므로 누적 텍스트 O. 과거 영상 프레임까지 누적 재입력하는 범위는 미명시이다.
- **비교 포인트:** history 사용 자체의 선행 연구. 우리 방법은 ROI가 표시된 시각 이력과 프레임별 수정 결정을 어떻게 연결하는지 구체화해야 한다.
- **근거:** [§4.1, Table 4, §4.2](https://arxiv.org/abs/2510.20206v2). 표는 논문의 전체 설계 기준이다.

### VideoRepair

- **발표:** 최초 공개 **2024** → **Findings of ACL 2026**. 최종 제목은 *Self-Correcting Text-to-Video Generation with Misalignment Detection and Localized Refinement*. [arXiv 이력](https://arxiv.org/abs/2411.15115), [공식 proceedings](https://aclanthology.org/2026.findings-acl.1817/).
- **핵심 방법:** 객체별 QA로 정합성 평가 → pointing·segmentation으로 잘된 객체의 보존 mask 구성 → 나머지 영역에 새 noise와 refinement prompt 적용.
- **공간·시간 범위:** 공간 국소 수정 O, 시간 선택 수정 X. 보존 영역도 함께 denoising하므로 영역 밖 픽셀의 hard freeze는 아니다.
- **History:** 현재 영상·QA·직전 선택 결과를 사용하며, 전체 수정 이력의 누적 입력 범위는 미명시이다.
- **비교 포인트:** 영상 점수가 0이면 전체 재생성, 1이면 중단하는 분기가 이미 있다. 우리 방법과는 **프레임별 ROI 타당성 검증으로 region/whole-frame을 선택한다**는 판단 기준을 비교해야 한다.
- **근거:** [§2.1–2.3, Appendix E.1](https://aclanthology.org/2026.findings-acl.1817.pdf). 전역 재생성·중단 분기는 §2.1.

## 추가 확인한 학회 발표 논문 5편

### P-Flow

- **발표:** 최초 공개 **2026** → **CVPR 2026 본회의**. *P-Flow: Prompting Visual Effects Generation*. [arXiv 이력](https://arxiv.org/abs/2603.22091), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Zhao_P-Flow_Prompting_Visual_Effects_Generation_CVPR_2026_paper.html).
- **핵심 방법:** reference 영상의 폭발·변형 같은 시각 효과와 현재 생성 결과를 VLM이 비교 → 효과를 설명하는 prompt 수정 → reference에서 얻은 시간적 noise prior와 random noise를 혼합해 다시 생성.
- **공간·시간 범위:** 영상 전체를 재생성한다. 효과 관련 단어만 고치는 것은 해당 객체의 spatial ROI만 수정하는 기능이 아니다.
- **History:** **모든 과거 prompt·VLM 분석은 누적**하므로 텍스트 O. 직접 보는 영상은 reference·직전 결과·현재 결과로 제한되어, 다회 시각 archive 입력은 X이다.
- **비교 포인트:** 텍스트와 시각 history의 보존 범위를 명시적으로 나눈 직접 비교군이다. 우리 설계는 ROI가 표시된 과거 프레임을 어떻게 선택·활용하는지, 그 이력이 수정 범위 결정에 어떤 역할을 하는지 대비할 수 있다.
- **근거:** [§3.3–3.5, Appendix A.1 Algorithm 1](https://arxiv.org/html/2603.22091v1), [공식 supplement](https://openaccess.thecvf.com/content/CVPR2026/supplemental/Zhao_P-Flow_Prompting_Visual_CVPR_2026_supplemental.pdf).

### CineCrew

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *Better Call CineCrew: Consistent Ultra-Long Narrative-to-Film Generation*. [arXiv 이력](https://arxiv.org/abs/2609.07720), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37023-5_19).
- **핵심 방법:** 서사를 shot·asset·camera 제약으로 구조화 → keyframe과 clip 생성 → Reviewer가 실패 원인을 진단 → 제약·keyframe prompt·video prompt를 수정해 해당 clip 재생성.
- **공간·시간 범위:** 실패 clip을 재시도하고 통과한 clip을 모아 장편 영상을 구성한다. 공간 X, 시간 O이며 clip 내부의 개별 프레임 freeze는 아니다.
- **History:** Production Rulebook에 실패 tag·확정 prompt·실행 피드백을 누적하고 역할에 맞는 제약을 가져오므로 텍스트 O. 시각 △는 asset·accepted keyframe·continuity anchor의 재사용이며, 모든 실패 영상의 누적 관찰을 뜻하지 않는다.
- **비교 포인트:** 성공 구간 유지와 누적 memory를 함께 다루는 학회 채택 비교군이다. 우리 방법의 프레임 단위 accept/freeze 및 ROI 검증과 판단 단위를 구분해야 한다.
- **근거:** [§3.3–3.4, Algorithm 1](https://arxiv.org/html/2609.07720v1), [공식 구현의 Production loop 설명](https://github.com/Ironieser/CineCrew).

### NoisEasier

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *NoisEasier: Test-Time Noise Optimization for Text-to-Video Generation*. [arXiv 이력](https://arxiv.org/abs/2608.30194), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37098-3_3).
- **핵심 방법:** 짧은 sampling으로 영상 생성 → 정합성·시각 품질·motion reward 계산 → gradient로 초기 noise와 sampling 단계의 stochastic noise를 갱신 → 수정된 noise로 영상 재생성.
- **공간·시간 범위:** 매번 영상 전체를 생성한다. 평가에 일부 프레임·clip·optical flow를 쓰더라도 ROI나 성공 프레임을 선택해 고정하는 기능은 아니다.
- **History:** 현재 생성 결과의 reward로 noise를 갱신하며, 과거 prompt·critique·영상 archive를 누적 재입력하지 않는다. optimizer state와 noise의 계승은 표의 누적 history와 구분한다.
- **비교 포인트:** 모델 가중치와 생성 prompt를 유지하면서 **샘플별 noise 변수**를 최적화한다. prompt를 통한 언어적 수정과 reward gradient를 통한 수치적 수정의 직접 대비가 가능하다.
- **근거:** [§3.1 Eq. 4, §3.3 Eq. 7, Fig. 2](https://arxiv.org/html/2608.30194v1).

### FreeInit

- **발표:** 최초 공개 **2023** → **ECCV 2024 본회의**. *FreeInit: Bridging Initialization Gap in Video Diffusion Models*. [arXiv 이력](https://arxiv.org/abs/2312.07537), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-031-72646-0_22).
- **핵심 방법:** 전체 DDIM sampling으로 clean video latent 생성 → 다시 noise 추가 → 시공간 저주파 성분은 유지하고 고주파는 새 random noise로 교체 → 이 noise에서 전체 sampling을 반복.
- **공간·시간 범위:** 모든 공간·시간 위치를 다시 생성한다. 3D Fourier filter의 mask는 **주파수 선택**이며 pixel ROI나 특정 프레임 선택이 아니다.
- **History:** 직전 생성 latent의 구조를 다음 초기 noise에 전달한다. 과거 여러 수정본을 명시적으로 보관·재입력하는 텍스트·시각 history는 없다.
- **비교 포인트:** 외부 critic 없이 생성 결과의 저주파 구조를 재사용하는 반복 개선이다. prompt는 고정하며, VLM이 오류를 진단하고 수정 범위를 선택하는 방법과 구분한다.
- **근거:** [§4, Eq. 8–11, Fig. 7, §5.1](https://arxiv.org/html/2312.07537v2). 통상적인 한 번의 denoising 내부 반복과 달리, clean latent를 만든 뒤 sampling을 다시 시작한다.

### AIGVE-MACS의 refinement 실험

- **발표:** 최초 공개 **2025** → **CVPR 2026 VGBE 워크숍**. *AIGVE-MACS: Unified Multi-Aspect Commenting and Scoring Model for AI-Generated Video Evaluation*. 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2507.01255), [공식 workshop proceedings](https://openaccess.thecvf.com/content/CVPR2026W/VGBE/html/Liu_AIGVE-MACS_Unified_Multi-Aspect_Commenting_and_Scoring_Model_for_AI-Generated_Video_CVPRW_2026_paper.html).
- **핵심 방법:** 평가 VLM이 생성 영상에 대해 여러 측면의 score·comment 출력 → GPT-4.1 Instruction Revisor가 prompt 수정 → HunyuanVideo로 재생성·재평가.
- **공간·시간 범위:** 영상 전체를 다시 생성한다. 평가기의 dynamic frame sampling은 관찰 전략이며, 선택된 프레임만 수정하는 방법은 아니다.
- **History:** 현재 instruction·영상의 평가 피드백을 사용하는 것은 명시되지만, 여러 과거 피드백·영상을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 주 기여는 평가 모델이고, 반복 생성은 적용 실험이다. PQSG처럼 평가기의 세밀한 진단이 실제 refinement에 어떻게 쓰이는지를 비교할 수 있다.
- **근거:** [arXiv §6, Fig. 5](https://arxiv.org/html/2507.01255v1#S6).

## 함께 비교할 기존 7편

### GenMAC

- **발표:** 최초 공개 **2024** → **AAAI 2026**. [arXiv 이력](https://arxiv.org/abs/2412.04440), [공식 proceedings](https://ojs.aaai.org/index.php/AAAI/article/view/37418).
- **핵심 방법:** prompt를 프레임별 객체 box로 설계 → 영상 생성 → 검증·제안·수정·구조화 agent가 prompt·layout·guidance를 다시 설계.
- **공간·시간 범위:** 객체 box로 생성을 유도하지만 기존 영상의 ROI나 성공 프레임을 보존하는 수정은 아니다. 전체 재생성으로 분류한다.
- **History:** 현재 영상과 직전 iteration의 구조화된 설계를 사용하며, 모든 과거 수정본을 누적 입력하는 방식은 아니다.
- **비교 포인트:** 수정 대상이 prompt뿐 아니라 layout·guidance까지 확장된다. 생성 위치 제어와 기존 결과의 국소 수정을 구분하는 데 유용하다.
- **근거:** [§3.1–3.3, Appendix A Eq. A3–A6](https://arxiv.org/html/2412.04440v1).

### VISTA

- **발표:** 최초 공개 **2025** → **CVPR 2026**. [arXiv 이력](https://arxiv.org/abs/2510.15831), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Long_VISTA_A_Test-Time_Self-Improving_Video_Generation_Agent_CVPR_2026_paper.html).
- **핵심 방법:** 여러 후보를 tournament로 비교해 champion 선택 → 시각·오디오·맥락 critique → prompt 개선 → 새 후보 생성·비교.
- **공간·시간 범위:** 영상 전체를 재생성하며, 공간 ROI나 특정 실패 프레임만 수정하지 않는다.
- **History:** 현재 champion·prompt·critique와 최선 후보의 유지가 핵심이다. 과거 여러 시도의 이력을 다음 수정에 누적 입력하는 방식은 아니다.
- **비교 포인트:** best-so-far 후보 보관과 전체 수정 history 활용의 차이를 보여 준다.
- **근거:** [§2.1–2.2, Algorithm 1](https://arxiv.org/html/2510.15831v1).

### PQSG의 refinement 실험

- **발표:** 최초 공개 **2026** → **ECCV 2026**. [arXiv 이력](https://arxiv.org/abs/2606.25306), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37035-8_17).
- **핵심 방법:** 객체→행동→물리 질문 그래프로 오류 진단 → 질문별 피드백으로 GPT가 prompt 수정 → Wan으로 새 영상 생성·재평가.
- **공간·시간 범위:** 질문별 진단을 mask나 프레임 선택으로 연결하지 않으므로 전체 재생성이다.
- **History:** 누적 prompt·피드백·시각 입력의 범위는 미명시이다.
- **비교 포인트:** 주목적은 평가이며, refinement는 §5.5의 응용 실험이다. 세밀한 오류 진단과 세밀한 수정 범위가 동일한 기능은 아니다.
- **근거:** [§3, §5.5, Fig. 4](https://arxiv.org/html/2606.25306v1).

### NEWTON

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. [arXiv 이력](https://arxiv.org/abs/2605.18396), [공식 프로젝트](https://newton026.github.io/newton/).
- **핵심 방법:** Planner가 물리 계산·keyframe 생성·prompt refinement 도구 선택 → Executor 실행 → Verifier가 의미·물리 점수 반환 → 여러 cycle 후 최고 점수 영상 선택.
- **공간·시간 범위:** 생성 조건을 고쳐 영상 전체를 재생성한다. keyframe 생성은 기존 성공 프레임을 freeze하는 기능이 아니다.
- **History:** reasoning·도구 입력/출력·점수를 누적하지만, **생성 비디오는 memory에서 명시적으로 제외**한다. 텍스트 O, 시각 X.
- **비교 포인트:** Generator는 고정하고 Planner를 학습한다. 도구 실행 이력과 실제 생성 영상의 시각 이력을 구분할 수 있다.
- **근거:** [§4.1.1–4.1.3, §4.2](https://arxiv.org/html/2605.18396v2).

### VideoAgent

- **발표:** 최초 공개 **2024** → **RLBRew @ RLC 2025 워크숍** 채택 확인. 비정규(non-archival) 워크숍이며 RLC 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2410.10076), [공식 accepted papers](https://rlbrew2-workshop.github.io/papers.html), [워크숍 성격](https://rlbrew2-workshop.github.io/submit.html).
- **핵심 방법:** 로봇 행동을 위한 video plan을 생성 → 학습된 self-conditioning 모델이 직전 비디오를 입력받아 보정 → VLM이 수용 여부 판단. Suggestive 변형은 언어 피드백도 사용한다.
- **공간·시간 범위:** video plan 전체를 수정하며, 공간 ROI나 실패 프레임만 선택하지 않는다.
- **History:** 직전 비디오 하나를 이어 쓰는 구조이므로, 여러 라운드의 누적 텍스트·시각 이력은 X로 분류한다.
- **비교 포인트:** prompt만 고치는 방법과 달리 기존 비디오 자체가 보정 모델의 입력이다. 동명의 video understanding 논문과 구분한다.
- **근거:** [§3.1–3.2, Eq. 12–13, §4.3.1](https://arxiv.org/html/2410.10076v3).

### Bootstrapping Physics-Grounded Video Generation

- **발표:** 최초 공개 **2025**. **ICCV 2025 Perception Test Workshop의 Physics-IQ 챌린지 3위 솔루션 보고·발표**이며, 정규 proceedings 논문 채택은 미확인이다. [arXiv 이력·보고서 설명](https://arxiv.org/abs/2511.20280), [공식 챌린지 안내](https://physics-iq.github.io/workshop/physics-iq-challenge.html), [저자 발표 기록](https://yafeng19.github.io/).
- **핵심 방법:** VLM이 입력 prefix의 물리 단서에서 미래 서술 생성 → LLM이 생성 prompt로 정리 → continuation 생성 → 물리 오류 피드백으로 prompt 수정.
- **공간·시간 범위:** 입력 3초 뒤의 5초 continuation 전체를 재생성한다. 주어진 prefix 유지와 성공한 출력 프레임 선택 freeze는 다르다.
- **History:** 현재 결과에서 다음 prompt를 만드는 과정은 설명하지만, 누적 history 입력 범위는 미명시이다.
- **비교 포인트:** 영상 continuation이라는 설정 차이를 반영해 비교해야 한다. ICCV 본회의 채택 논문으로 표기하지 않는다.
- **근거:** [§2, Step 1–3](https://arxiv.org/html/2511.20280v1).

### SPIRAL

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. 현재 제목은 *SPIRAL: Self-Evolving Action-Conditioned Video Generation via Reflective Planning Agents*. [arXiv 이력](https://arxiv.org/abs/2603.08403), [공식 프로젝트](https://yuyang-cloud.github.io/spiral/).
- **핵심 방법:** 목표를 action step으로 분해 → segment 생성·평가 → 실패한 지시문 수정·재생성 → 반복 실패 시 해당 step부터 이후 계획 변경.
- **공간·시간 범위:** 실패 segment의 공간 전체를 다시 만들고 성공한 앞 구간은 유지한다. 공간 X, 시간 O이며 같은 clip 내부의 프레임별 freeze와 구분한다.
- **History:** Planner는 성공한 과거 segment의 시각·의미 이력과 실패 시도·진단을 사용한다. Critic은 현재 segment를 평가한다. 시간적으로 앞선 실행 이력이 중심이므로 두 history 축은 △.
- **비교 포인트:** 시간 선택 수정·시각 이력의 중요한 선행 연구. 동일 clip의 수정본 비교 및 ROI 타당성 판단과 대비해야 한다. 별도의 agent/generator 학습도 다룬다.
- **근거:** [§3.1–3.5, Appendix B.1/B.3/B.4, F.2](https://arxiv.org/html/2603.08403v3).

## 앞서 추가한 4편

### MotionAgent · rethinking 옵션

- **발표:** 최초 공개 **2025** → **ICCV 2025**. [arXiv 이력](https://arxiv.org/abs/2502.03207), [공식 proceedings PDF](https://openaccess.thecvf.com/content/ICCV2025/papers/Liao_MotionAgent_Fine-grained_Controllable_Video_Generation_via_Motion_Field_Agent_ICCV_2025_paper.pdf).
- **핵심 방법:** 입력 이미지의 객체를 Grounded-SAM으로 찾음 → 객체 궤적·카메라 움직임 계획 → optical flow로 변환해 I2V 생성 → **선택적 rethinking**에서 생성 영상과 이전 action을 검토하고 제어 조건 수정.
- **공간·시간 범위:** 전체 I2V를 다시 생성한다. SAM은 입력 객체의 grounding·운동 제어에 쓰이며, 기존 출력의 ROI만 보정하는 mask는 아니다.
- **History:** 이전 영상·action을 검토하지만, 여러 refinement 라운드의 모든 이력을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 피드백으로 고치는 대상이 prompt뿐 아니라 객체 궤적·카메라 제어이다. 반복 개선은 선택적 단계이므로 기본 생성 전체의 필수 절차로 해석하지 않는다.
- **근거:** [§3.1.2, §3.1.4, §3.2, §4.3.4, Appendix 8](https://arxiv.org/html/2502.03207v2).

### CoAgent

- **발표:** 최초 공개 **2025**. **학회 채택 미확인**. *CoAgent: Collaborative Planning and Consistency Agent for Coherent Video Generation*. [arXiv 이력](https://arxiv.org/abs/2512.22536).
- **핵심 방법:** storyboard를 shot으로 분해 → 객체 상태 memory를 참조해 생성 → VLM 검증 → 의미 오류이면 storyboard 수정, 외관 오류이면 첫 프레임·양끝 프레임 조건 모드로 변경 → 해당 shot 재생성.
- **공간·시간 범위:** 선택한 shot 전체를 다시 생성한다. 공간 X, 시간 O이며 shot 내부의 프레임별 freeze는 아니다.
- **History:** 객체 속성과 appearance embedding을 shot 간 memory로 유지한다. 이는 같은 shot의 모든 실패 prompt·영상 이력이 아닌 **장면 간 상태 기억**이므로 두 축은 △.
- **비교 포인트:** 검증 결과가 prompt뿐 아니라 생성 조건 모드 선택으로 이어진다. 객체 시각 memory와 ROI가 표시된 수정 history의 역할 차이를 비교할 수 있다.
- **근거:** [§3.3–3.5, Algorithm 1–2](https://arxiv.org/html/2512.22536v1). 수정 단위는 Algorithm 2의 shot 재생성 호출을 기준으로 판정했다.

### A²RD · 기본형

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *A²RD: Agentic Autoregressive Diffusion for Long Video Consistency*. [arXiv 이력](https://arxiv.org/abs/2605.06924), [Google Research 공식 목록](https://research.google/pubs/a2rd-agentic-autoregressive-diffusion-for-long-video-consistency-2/).
- **핵심 방법:** 관련 장면 상태·시각 자료 검색 → segment 생성 → 일관성·품질 평가 → 성공·실패 사례를 검색해 prompt 개선(MAPO) → 필요하면 ending frame도 편집한 뒤 segment 재생성.
- **공간·시간 범위:** video 수정은 segment 전체를 재생성하므로 공간 X, 시간 O. boundary image 편집은 video ROI만 수정하거나 성공 프레임을 freeze하는 절차와 구분한다.
- **History:** 텍스트는 사전·온라인 사례 DB에서 관련 성공·실패 사례를 검색한다. 시각은 관련 과거 segment 영상·keyframe을 참조한다. 동일 segment의 전체 수정 archive 입력과 달라 두 축은 △.
- **비교 포인트:** **기본 A²RD만** 이 표의 대상이다. A²RD-Parallel은 video self-improvement를 생략한다. 전체 history 입력과 관련 사례 검색의 차이도 비교할 수 있다.
- **근거:** [§3.1–3.4, Eq. 9–10, MAPO 문단](https://arxiv.org/html/2605.06924v1).

### VQQA

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *VQQA: An Agentic Approach for Video Evaluation and Quality Improvement*. [arXiv 이력](https://arxiv.org/abs/2603.12310), [공식 프로젝트](https://yiwen-song.github.io/vqqa/).
- **핵심 방법:** 입력에 맞는 평가 질문 생성 → 영상의 답변·점수·이유 수집 → 낮은 점수의 QA로 prompt 수정 → 영상 재생성. 최종 후보는 원래 prompt에 대한 평가로 선택한다.
- **공간·시간 범위:** 세부 질문으로 오류를 찾지만 생성은 영상 전체에 적용한다. 국소 QA는 공간 mask나 프레임 선택 수정이 아니다.
- **History:** prompt 템플릿은 과거 prompt·VQA·점수의 **선택적 텍스트 history**를 지원하므로 △. 보존 길이는 미명시이며, 이전 영상들을 다음 refinement에 누적 재입력하는 기능은 정의하지 않아 시각 X.
- **비교 포인트:** 질문 기반 피드백과 선택적 history를 결합한다. 최종 후보 pool의 보관·선정은 다음 refinement의 시각 history 입력과 구분한다.
- **근거:** [§3.1.2 Eq. 5, §3.2](https://arxiv.org/html/2603.12310v1), [PDF Appendix F.3, pp. 42–48의 prompt 템플릿](https://arxiv.org/pdf/2603.12310v1). HTML에 빠진 history 세부 내용은 PDF로 확인했다.

## 우리 아이디어와의 비교

- **핵심 흐름:** P0·V0 → prompt 수정 후 P1·V1 → 두 영상으로 ROI 계산 → ROI가 표시된 프레임들과 prompt history를 GPT에 입력.
- **공간·시간 범위:** 잘된 프레임은 통과·freeze. 실패 프레임은 ROI가 적절하면 region만, 부적절하면 해당 프레임 전체에 refined prompt를 적용.
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
