## 분류 요약

- **조사 기준일:** 기존 비교군 2026-10-14, sampling·국소 편집 비교군 2026-10-17. 총 22편.
- **시드·초기 noise 추가 확인일:** 2026-10-15. 동일 사례의 refinement 반복 사이를 비교하며, 논문별 근거는 아래 **시드·초기 noise** 항목에 표시했다.
- **Reference video 추가 확인일:** 2026-10-15. 아래 표는 현재 생성 목표를 위한 외부 참조 영상 사용 여부를 기준으로 한다.
- **공식 코드·재현 구성 확인일:** 2026-10-19. 공식 프로젝트·저장소의 실제 구현과 실행 안내를 정적으로 확인했으며, 설치·모델 추론·유료 API 실행은 하지 않았다.
- **비교 범위:** 영상·shot·segment를 생성한 뒤 피드백이나 생성 결과를 활용해 다시 생성하는 추론 시 반복 개선을 중심으로 한다. NoisEasier의 noise 최적화와 FreeInit의 재초기화 외에, **sampling 내부 보정(Self-Refining Video Sampling)**과 **원본 영상의 국소 편집(VideoGrain)**도 관련 메커니즘으로 비교한다. 이 두 방법은 완성 영상을 평가해 다시 만드는 feedback loop와 구분한다.
- **연도 기준:** ‘최초 공개’는 arXiv v1 연도이며, 학회 발표 연도와 구분했다. ‘채택 미확인’은 확인한 1차 출처에서 채택 근거를 확보하지 못했다는 뜻이다.
- **기호:** O = 해당함, X = 해당하지 않음, △ = 조건부·선택적 지원 또는 비교 범위가 다른 이력, — = 정책·범위 미확인 또는 해당 비교축 적용 대상 아님(본문에서 구분). 우리 방법은 구현 검증 결과가 아닌 설계안이다.

| 논문 | 최초 공개 | 발표처·연도 | 공식 코드 | 메소드 재현 구성 | 공간 국소 수정 | 시간 선택 수정 | 누적 텍스트 이력 | 누적 시각 이력 | 시드 변경 | 초기 noise 변경 | Reference video 사용 |
|---|:---:|---| :---: | :---: |:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **PhyT2V** | 2024 | CVPR 2025 | O | △ | X | X | — | — | X | X | X |
| **VideoTPO / TiViBench** | 2025 | CVPR 2026 | O | O | X | X | X | X | X | X | X |
| **VideoRepair** | 2024 | Findings of ACL 2026 | O | △ | O | X | — | — | △ | O | X |
| GenMAC | 2024 | AAAI 2026 | O | O | X | X | X | X | X | X | X |
| VISTA | 2025 | CVPR 2026 | X | X | X | X | X | X | — | — | X |
| PQSG refinement | 2026 | ECCV 2026 | O | X | X | X | — | — | — | — | X |
| MotionAgent · rethinking | 2025 | ICCV 2025 | O | O | X | X | — | — | X | X | X |
| P-Flow | 2026 | CVPR 2026 | X | X | X | X | O | X | — | O | O |
| CineCrew | 2026 | ECCV 2026 | O | △ | X | O | O | △ | △ | — | X |
| NoisEasier | 2026 | ECCV 2026 | O | O | X | X | X | X | — | O | X |
| FreeInit | 2023 | ECCV 2024 | O | O | X | X | X | X | X | O | X |
| Self-Refining Video Sampling    | 2026  | ICML 2026             | O | O |    △     |    X     |     X     |    X     |   —   |      —      |         X          |
| VideoGrain                      | 2025  | ICLR 2025             | O | O |    O     |    X     |     X     |    X     |   —   |      —      |         X          |
| VideoAgent | 2024 | RLBRew @ RLC 2025 워크숍 | O | O | X | X | X | X | X | O | X |
| Bootstrapping Physics-Grounded… | 2025 | ICCV 2025 워크숍 챌린지 보고 | X | X | X | X | — | — | — | — | O |
| AIGVE-MACS refinement | 2025 | CVPR 2026 VGBE 워크숍 | △ | X | X | X | — | — | — | — | X |
| **RAPO++** | 2025 | 채택 미확인 | O | △ | X | X | O | — | X | X | X |
| NEWTON | 2026 | 채택 미확인 | O | △ | X | X | O | X | — | — | X |
| SPIRAL | 2026 | 채택 미확인 | X | X | X | O | △ | △ | — | — | X |
| CoAgent | 2025 | 채택 미확인 | X | X | X | O | △ | △ | — | — | X |
| A²RD · 기본형 | 2026 | 채택 미확인 | X | X | X | O | △ | △ | — | — | X |
| VQQA | 2026 | 채택 미확인 | X | X | X | X | △ | X | O | O | X |
| **우리 방법 · 설계안** | — | — | — | — | O | O | O | O | — | — | X |

- **공식 코드:** O = 방법 또는 평가기의 실제 소스 공개, △ = 추론 예제 등 제한적인 코드만 공개, X = 확인일 기준 공식 방법 소스가 없거나 GitHub 재검색에서도 공개된 공식 구현을 찾지 못함. — = 점검 대상 아님(우리 설계안). README·소개 영상·checkpoint만 있다는 이유로 O를 주지 않는다.
- **메소드 재현 구성:** O = 본문에 명시한 backbone·변형에서 핵심 refinement 로직, 실행 진입점, 설정·의존성·필요 모델/입력의 준비 안내를 확인함. △ = 일부 구현은 있으나 필수 단계·실행 파일·서버 구성 누락, 알려진 실행 오류 또는 논문 방법과의 차이를 보완해야 함. X = 비교 대상 refinement 구현이 공개되지 않았거나 공개 구현을 찾지 못함. — = 점검 대상 아님(우리 설계안).
- **재현 판정의 한계:** O는 **소스와 실행 구성이 갖춰졌다는 정적 판정**이며, 실제 실행 성공·논문 수치의 동일 재현을 뜻하지 않는다. 공개 checkpoint 준비·API key 설정은 통상적인 실행 조건으로 보고, 핵심 코드 미공개와 구분했다. demo 기본값과 논문 실험값의 차이는 본문에 적었다. 코드 링크는 확인한 commit으로 고정했으며, 우리 설계안은 이번 점검 대상이 아니다.
- **GitHub 재확인:** 2026-10-19. 논문명·약칭·arXiv ID 검색과 저자·프로젝트 계정의 공개 저장소를 대조했다. 공식 방법 구현을 찾지 못한 VISTA·Bootstrapping Physics-Grounded Video Generation·SPIRAL·CoAgent·VQQA는 X로 분류했다. 소개 웹사이트·논문 목록·동명 프로젝트는 방법 코드로 세지 않았다.
- **코드·재현 구성의 △ 사유:** 두 코드 관련 열의 △는 아래 이유로 표시했다. 다른 분류 축의 △는 각 논문의 해당 항목에서 설명한다.
	- **PhyT2V — 재현 구성 △:** 공개 demo는 고정 round·GPT-4를 사용한다. 논문의 점수/수렴 기반 자동 종료가 없고 o1-preview 설정과도 다르다.
	- **VideoRepair — 재현 구성 △:** benchmark용 국소 수정은 구현되어 있으나, 자유 prompt용 demo·질문 생성은 미공개다. README에서 안내하는 일부 실행 파일도 없다.
	- **CineCrew — 재현 구성 △:** 검수·수정·재시도 loop는 있지만, 논문 기본 Qwen-Image·Wan2.2 생성 서버의 구현·배포 구성이 없어 별도 구축이 필요하다.
	- **AIGVE-MACS — 공식 코드 △:** 평가 checkpoint와 추론 예제만 제공하며, 예제가 import하는 `models` 모듈도 공개 tree에 없다. refinement loop는 없어 재현 구성은 X이다.
	- **RAPO++ — 재현 구성 △:** SSPO loop는 있지만 누적 history/backtracking과 Stage3 학습 코드·설정이 없다. 공개 SSPO 파일의 문자열 문법 오류도 수정해야 한다.
	- **NEWTON — 재현 구성 △:** 공개판은 API planner+Seedance이다. 논문의 학습된 planner·Flow-GRPO 학습 코드와 LTX/Veo 실험 구성이 갖춰져 있지 않다.
- **Sampling·편집 비교군:** Self-Refining의 공간 △는 uncertainty-aware 확장형에만 해당하며 기본 P&P는 X이다. Self-Refining·VideoGrain의 시드/초기 noise —는 완성 영상 간 반복이 없어 해당 축의 적용 대상이 아니라는 뜻이다. VideoGrain의 필수 원본 영상은 편집 대상이므로 아래 reference 정의에서는 X로 구분한다.
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
- **공식 코드·재현 구성:** O / △. CogVideoX의 caption→평가→prompt 수정→재생성 loop와 환경·모델 안내는 있다. 다만 공개 demo는 고정 round·GPT-4를 사용하며, 논문의 점수/수렴 기반 자동 종료와 o1-preview 설정까지 동일하게 구현한 것은 아니다. README의 환경 파일명도 실제 `environment.yml`과 달라 실행 안내 보정이 필요하다. [공개 loop](https://github.com/pittisl/PhyT2V/blob/7416e18da2c516caf6efb1e60dc142b6745e947b/inference.py#L310-L375) · [설치·모델](https://github.com/pittisl/PhyT2V/blob/7416e18da2c516caf6efb1e60dc142b6745e947b/README.md#L29-L55).
- **근거:** [§3, Fig. 6, §3.2](https://arxiv.org/abs/2412.00596v2).

### VideoTPO / TiViBench

- **발표:** 최초 공개 **2025** → **CVPR 2026**. VideoTPO는 *TiViBench: Benchmarking Think-in-Video Reasoning for Video Generation*의 방법명이다. [arXiv 이력](https://arxiv.org/abs/2511.13704), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Chen_TiViBench_Benchmarking_Think-in-Video_Reasoning_for_Video_Generation_CVPR_2026_paper.html).
- **핵심 방법:** 같은 입력 이미지·prompt로 만든 두 영상 후보를 비교 → 장단점을 textual loss로 표현 → prompt의 수정 방향인 textual gradient 계산 → 공통 prompt 갱신.
- **Reference video:** X. 입력은 단일 I2V 이미지와 prompt이고, 비교하는 두 영상은 그 round에서 자체 생성한 후보이다. [§4 Eq. 1–3](https://arxiv.org/html/2511.13704v2#S4).
- **공간·시간 범위:** 생성 구간 전체를 다시 만든다. 공개 Wan 구현의 첫·마지막 프레임 추출은 평가용 관찰이며, 그 프레임만 수정하는 기능이 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 Wan 구현 기준). 두 후보의 seed 42·123은 서로 다르지만, 다음 round에도 같은 두 seed를 재사용하고 생성마다 Generator를 재설정한다. 따라서 후보별 시작 noise는 유지된다. [round별 후보 생성](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L352-L403), [Generator 재설정](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L130-L144).
- **History:** 현재 후보 쌍·prompt·loss·gradient를 사용한다. 공개 기본 설정은 gradient memory가 0이며, 여러 과거 시도를 누적 입력하지 않는다.
- **비교 포인트:** 두 후보의 비교를 prompt 최적화에 쓰지만, 두 iteration의 영상 차이로 수정 ROI를 계산하는 방식은 아니다.
- **공식 코드·재현 구성:** O / O(Wan2.1 VideoTPO 기준). 두 후보 생성→textual loss/gradient→prompt 갱신 loop와 전용 TextGrad 구현·실행 명령이 있다. TiViBench 전체 데이터·평가기는 별도 미공개 TODO이다. [반복 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L428) · [공개 범위](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/README.md#L39-L43).
- **근거:** [§4, Eq. 1–3, Appendix D.1](https://arxiv.org/abs/2511.13704v2), [공식 Wan 구현](https://github.com/EnVision-Research/TiViBench/blob/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8/VideoTPO/videotpo_wan.py#L278-L345).

### VideoRepair

- **발표:** 최초 공개 **2024** → **Findings of ACL 2026**. 최종 제목은 *Self-Correcting Text-to-Video Generation with Misalignment Detection and Localized Refinement*. [arXiv 이력](https://arxiv.org/abs/2411.15115), [공식 proceedings](https://aclanthology.org/2026.findings-acl.1817/).
- **핵심 방법:** 객체별 QA로 정합성 평가 → pointing·segmentation으로 잘된 객체의 보존 mask 구성 → 나머지 영역에 새 noise와 refinement prompt 적용.
- **Reference video:** X. 원본 V0는 prompt·noise로 자체 생성한 수정 대상이며, 별도의 외부 목표 clip이 아니다. [§2 Problem statement](https://arxiv.org/html/2411.15115v3#S2).
- **공간·시간 범위:** 공간 국소 수정 O, 시간 선택 수정 X. 보존 영역도 함께 denoising하므로 영역 밖 픽셀의 hard freeze는 아니다.
- **시드·초기 noise:** 시드 △ / 초기 noise O(원문 기준). score=0이면 새 seed로 전체를 재생성한다. 국소 수정에서는 보존 영역의 기존 초기 noise와 수정 영역에 새로 뽑은 Gaussian noise를 혼합한다. [§2.1, §2.3 Eq. 2](https://arxiv.org/html/2411.15115v3#S2.SS3). 공식 T2V-Turbo 구현은 국소 수정용 seed도 round별로 바꾼다. [local seed·초기 noise 구성](https://github.com/daeunni/VideoRepair/blob/a0a261f55e771403de44accded15c7c5eb0b1370/t2vturbo/pipeline/multidiffusion.py#L349-L368).
- **History:** 현재 영상·QA·직전 선택 결과를 사용하며, 전체 수정 이력의 누적 입력 범위는 미명시이다.
- **비교 포인트:** 영상 점수가 0이면 전체 재생성, 1이면 중단하는 분기가 이미 있다. 우리 방법과는 **프레임별 ROI 타당성 검증으로 region/whole-frame을 선택한다**는 판단 기준을 비교해야 한다.
- **공식 코드·재현 구성:** O / △. benchmark용 평가→mask→국소 재생성·ranking과 noise/denoising 혼합은 공개됐다. **자유 prompt용 demo·질문 생성은 미공개 TODO**이며 README의 `main_iter_demo.py`도 없다(공개 실행 경로는 `run.sh`→`main.py`). [공개 loop](https://github.com/daeunni/VideoRepair/blob/a0a261f55e771403de44accded15c7c5eb0b1370/main.py#L144-L314) · [미공개 항목](https://github.com/daeunni/VideoRepair/blob/a0a261f55e771403de44accded15c7c5eb0b1370/README.md#L117-L120).
- **근거:** [§2.1–2.3, Appendix E.1](https://aclanthology.org/2026.findings-acl.1817.pdf). 전역 재생성·중단 분기는 §2.1.

### GenMAC

- **발표:** 최초 공개 **2024** → **AAAI 2026**. [arXiv 이력](https://arxiv.org/abs/2412.04440), [공식 proceedings](https://ojs.aaai.org/index.php/AAAI/article/view/37418).
- **핵심 방법:** prompt를 프레임별 객체 box로 설계 → 영상 생성 → 검증·제안·수정·구조화 agent가 prompt·layout·guidance를 다시 설계.
- **Reference video:** X. 텍스트에서 구성한 layout·prompt·guidance로 영상을 생성하고 검증하며, 외부 reference video는 사용하지 않는다. [§3.1–3.3](https://arxiv.org/html/2412.04440v1#S3.SS1).
- **공간·시간 범위:** 객체 box로 생성을 유도하지만 기존 영상의 ROI나 성공 프레임을 보존하는 수정은 아니다. 전체 재생성으로 분류한다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 기본 단일-prompt demo 한정). 매 round 새 생성 프로세스에 동일 seed 12345678을 전달해 RNG를 재설정하고 같은 시작 noise를 재현한다. 여러 prompt를 batch로 처리하면 앞선 사례의 skip·종료로 RNG 소비 순서가 달라질 수 있어 일반화하지 않는다. [launcher](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/scripts/run_t2v.py#L36-L74), [seed 초기화](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/scripts/evaluation/inference_pipeline_iter.py#L560-L570), [시작 noise 생성](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/lvdm/models/samplers/ddim.py#L130-L138).
- **History:** 현재 영상과 직전 iteration의 구조화된 설계를 사용하며, 모든 과거 수정본을 누적 입력하는 방식은 아니다.
- **비교 포인트:** 수정 대상이 prompt뿐 아니라 layout·guidance까지 확장된다. 생성 위치 제어와 기존 결과의 국소 수정을 구분하는 데 유용하다.
- **공식 코드·재현 구성:** O / O(VideoCrafter2 demo 기준). 반복 launcher, 검증·제안·수정 agent의 prompt와 호출 코드, layout guidance 생성기, 환경·checkpoint·실행 설정이 연결돼 있다. [반복 실행](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/scripts/run_t2v.py#L22-L74) · [agent 연결](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/utils/llm.py#L254-L353) · [설치](https://github.com/Karine-Huang/GenMAC/blob/8aee92a2ac30d63e876fc36867d83b1d27a208a3/README.md#L14-L85).
- **근거:** [§3.1–3.3, Appendix A Eq. A3–A6](https://arxiv.org/html/2412.04440v1).

### VISTA

- **발표:** 최초 공개 **2025** → **CVPR 2026**. [arXiv 이력](https://arxiv.org/abs/2510.15831), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Long_VISTA_A_Test-Time_Self-Improving_Video_Generation_Agent_CVPR_2026_paper.html).
- **핵심 방법:** 여러 후보를 tournament로 비교해 champion 선택 → 시각·오디오·맥락 critique → prompt 개선 → 새 후보 생성·비교.
- **Reference video:** X. 비교 대상은 자체 생성 후보와 이전 winner이며, 외부 목표 영상 없이 사용자 prompt와 평가 기준으로 개선한다. [§2 Algorithm 1](https://arxiv.org/html/2510.15831v1#S2).
- **공간·시간 범위:** 영상 전체를 재생성하며, 공간 ROI나 특정 실패 프레임만 수정하지 않는다.
- **시드·초기 noise:** 시드 — / 초기 noise —. Veo 후보 생성·선택과 prompt 수정은 설명하지만, refinement round 사이 seed나 시작 latent의 유지·변경 정책은 명시하지 않는다. [§2 Algorithm 1](https://arxiv.org/html/2510.15831v1).
- **History:** 현재 champion·prompt·critique와 최선 후보의 유지가 핵심이다. 과거 여러 시도의 이력을 다음 수정에 누적 입력하는 방식은 아니다.
- **비교 포인트:** best-so-far 후보 보관과 전체 수정 history 활용의 차이를 보여 준다.
- **공식 코드·재현 구성:** X / X. GitHub의 논문명·arXiv ID 검색과 저자·프로젝트 계정을 재확인했다. [공식 프로젝트 저장소](https://github.com/g-vista/g-vista.github.io/tree/6e03d369d25b6b4a3adf869730c31fbf8f729878)는 HTML·그림·결과 영상만 제공하며, 생성→평가→prompt 수정 구현은 없다.
- **근거:** [§2.1–2.2, Algorithm 1](https://arxiv.org/html/2510.15831v1).

### PQSG의 refinement 실험

- **발표:** 최초 공개 **2026** → **ECCV 2026**. [arXiv 이력](https://arxiv.org/abs/2606.25306), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37035-8_17).
- **핵심 방법:** 객체→행동→물리 질문 그래프로 오류 진단 → 질문별 피드백으로 GPT가 prompt 수정 → Wan으로 새 영상 생성·재평가.
- **Reference video:** X(§5.5 기본 refinement 기준). 자체 생성 영상의 PQSG 피드백으로 prompt를 수정한다. reference video를 QG에 추가하는 실험은 별도 ablation이며 기본 refinement 입력과 구분한다. [§5.5–5.6, Appendix B Table 11](https://arxiv.org/html/2606.25306v1#S5.SS5).
- **공간·시간 범위:** 질문별 진단을 mask나 프레임 선택으로 연결하지 않으므로 전체 재생성이다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 평가 피드백으로 prompt를 수정하고 Wan2.2로 재생성하지만, round별 seed·초기 noise 정책은 미명시이다. 공개 평가 코드나 별도의 재현 구현 설정으로 이를 대신 판정하지 않았다. [§5.5](https://arxiv.org/html/2606.25306v1#S5.SS5).
- **History:** 누적 prompt·피드백·시각 입력의 범위는 미명시이다.
- **비교 포인트:** 주목적은 평가이며, refinement는 §5.5의 응용 실험이다. 세밀한 오류 진단과 세밀한 수정 범위가 동일한 기능은 아니다.
- **공식 코드·재현 구성:** O / X. [공식 평가 코드·설치 안내](https://github.com/atinpothiraj/pqsg/blob/03afdb0e6f9ed68d363a68eea5ca4fd90d53848e/README.md)는 공개되어 있으나, 표의 **refinement loop는 제공하지 않는다**. [plot_refinement.py](https://github.com/atinpothiraj/pqsg/blob/03afdb0e6f9ed68d363a68eea5ca4fd90d53848e/scripts/plot_refinement.py#L1-L40)는 저장된 점수로 그림만 재작성하며, prompt 수정→Wan 재생성→재평가 연결은 별도 구현이 필요하다.
- **근거:** [§3, §5.5, Fig. 4](https://arxiv.org/html/2606.25306v1).

### MotionAgent · rethinking 옵션

- **발표:** 최초 공개 **2025** → **ICCV 2025**. [arXiv 이력](https://arxiv.org/abs/2502.03207), [공식 proceedings PDF](https://openaccess.thecvf.com/content/ICCV2025/papers/Liao_MotionAgent_Fine-grained_Controllable_Video_Generation_via_Motion_Field_Agent_ICCV_2025_paper.pdf).
- **핵심 방법:** 입력 이미지의 객체를 Grounded-SAM으로 찾음 → 객체 궤적·카메라 움직임 계획 → optical flow로 변환해 I2V 생성 → **선택적 rethinking**에서 생성 영상과 이전 action을 검토하고 제어 조건 수정.
- **Reference video:** X. 입력은 이미지와 motion text이며, rethinking에서 검토하는 영상은 직전 생성 결과이다. [§3.1.2–3.1.4](https://arxiv.org/html/2502.03207v2#S3.SS1).
- **공간·시간 범위:** 전체 I2V를 다시 생성한다. SAM은 입력 객체의 grounding·운동 제어에 쓰이며, 기존 출력의 ROI만 보정하는 mask는 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise X(공식 rethinking demo 기준). 최초 생성과 rethinking 모두 생성 직전에 seed 42로 재설정하고 같은 shape의 시작 noise를 재현한다. 수정하는 것은 trajectory·camera 조건이다. [공통 생성 함수](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/run_agent.py#L629-L652), [noise 생성](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/pipeline/pipeline.py#L224-L255).
- **History:** 이전 영상·action을 검토하지만, 여러 refinement 라운드의 모든 이력을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 피드백으로 고치는 대상이 prompt뿐 아니라 객체 궤적·카메라 제어이다. 반복 개선은 선택적 단계이므로 기본 생성 전체의 필수 절차로 해석하지 않는다.
- **공식 코드·재현 구성:** O / O(**공개 rethinking demo 기준**). [이전 영상·action 검토와 재생성](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/run_agent.py#L574-L759), [설치·모델·실행 안내](https://github.com/leoisufa/MotionAgent/blob/60f408f7f533e05025ae78e463e1ff3fcc7bfabc/README.md#L52-L148)가 모두 있다. API key와 외부 checkpoint를 준비해야 하며, 논문 전체 정량 실험의 수치 재현까지 확인한 것은 아니다.
- **근거:** [§3.1.2, §3.1.4, §3.2, §4.3.4, Appendix 8](https://arxiv.org/html/2502.03207v2).

### P-Flow

- **발표:** 최초 공개 **2026** → **CVPR 2026 본회의**. *P-Flow: Prompting Visual Effects Generation*. [arXiv 이력](https://arxiv.org/abs/2603.22091), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2026/html/Zhao_P-Flow_Prompting_Visual_Effects_Generation_CVPR_2026_paper.html).
- **핵심 방법:** reference 영상의 폭발·변형 같은 시각 효과와 현재 생성 결과를 VLM이 비교 → 효과를 설명하는 prompt 수정 → reference에서 얻은 시간적 noise prior와 random noise를 혼합해 다시 생성.
- **Reference video:** O. 원하는 동적 효과를 보여 주는 **외부 목표 영상**을 사용한다. 해당 영상에서 temporal noise prior를 추출하고, VLM이 생성 결과와 비교해 prompt를 수정한다. [§3.1, §3.3–3.5](https://arxiv.org/html/2603.22091v1#S3.SS1).
- **공간·시간 범위:** 영상 전체를 재생성한다. 효과 관련 단어만 고치는 것은 해당 객체의 spatial ROI만 수정하는 기능이 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise O. 매 iteration마다 새 Gaussian noise를 뽑아 reference의 temporal noise prior와 혼합한다. seed 번호를 변경하는지는 미명시이며, 새 noise 추출만으로 seed 변경을 단정하지 않는다. [Appendix A.1 Algorithm 1, §3.3](https://arxiv.org/html/2603.22091v1#A1.SS1).
- **History:** **모든 과거 prompt·VLM 분석은 누적**하므로 텍스트 O. 직접 보는 영상은 reference·직전 결과·현재 결과로 제한되어, 다회 시각 archive 입력은 X이다.
- **비교 포인트:** 텍스트와 시각 history의 보존 범위를 명시적으로 나눈 직접 비교군이다. 우리 설계는 ROI가 표시된 과거 프레임을 어떻게 선택·활용하는지, 그 이력이 수정 범위 결정에 어떤 역할을 하는지 대비할 수 있다.
- **공식 코드·재현 구성:** X / X. [공식 저장소](https://github.com/showlab/P-Flow/blob/d441ae2dc895afeb820530b07c1ae5eb730e5e99/README.md#L20-L22)는 존재하지만 확인 시점에는 README만 있고 코드 공개 예정이라고 명시되어 있다. prompt 최적화·noise prior·생성 실행 코드는 제공되지 않는다.
- **근거:** [§3.3–3.5, Appendix A.1 Algorithm 1](https://arxiv.org/html/2603.22091v1), [공식 supplement](https://openaccess.thecvf.com/content/CVPR2026/supplemental/Zhao_P-Flow_Prompting_Visual_CVPR_2026_supplemental.pdf).

### CineCrew

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *Better Call CineCrew: Consistent Ultra-Long Narrative-to-Film Generation*. [arXiv 이력](https://arxiv.org/abs/2609.07720), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37023-5_19).
- **핵심 방법:** 서사를 shot·asset·camera 제약으로 구조화 → keyframe과 clip 생성 → Reviewer가 실패 원인을 진단 → 제약·keyframe prompt·video prompt를 수정해 해당 clip 재생성.
- **Reference video:** X. narrative와 character/set 이미지·생성 keyframe을 조건으로 clip을 만들며, Dailies Reviewer가 검토하는 영상도 자체 생성 clip이다. [§3.1, §3.3–3.4](https://arxiv.org/html/2609.07720v1#S3.SS1).
- **공간·시간 범위:** 실패 clip을 재시도하고 통과한 clip을 모아 장편 영상을 구성한다. 공간 X, 시간 O이며 clip 내부의 개별 프레임 freeze는 아니다.
- **시드·초기 noise:** 시드 △ / 초기 noise —(공식 구현의 API 요청값 기준). seed를 지정한 경우 검수 실패 후 retry마다 1을 더하고, 기본값 None이면 seed를 보내지 않는다. 외부 생성 API 내부의 초기 noise는 확인할 수 없다. [검수·재시도 loop](https://github.com/Ironieser/CineCrew/blob/3eac773704088f65ae2e24054c48fccbd4172f2d/src/agents/production_operator/production_operator_agent.py#L291-L334).
- **History:** Production Rulebook에 실패 tag·확정 prompt·실행 피드백을 누적하고 역할에 맞는 제약을 가져오므로 텍스트 O. 시각 △는 asset·accepted keyframe·continuity anchor의 재사용이며, 모든 실패 영상의 누적 관찰을 뜻하지 않는다.
- **비교 포인트:** 성공 구간 유지와 누적 memory를 함께 다루는 학회 채택 비교군이다. 우리 방법의 프레임 단위 accept/freeze 및 ROI 검증과 판단 단위를 구분해야 한다.
- **공식 코드·재현 구성:** O / △. [생성→VLM 검수→prompt 수정→재시도](https://github.com/Ironieser/CineCrew/blob/3eac773704088f65ae2e24054c48fccbd4172f2d/src/agents/production_operator/production_operator_agent.py#L291-L337)와 실행/config는 공개되었다. 다만 논문 기본 Qwen-Image·Wan2.2는 **사용자가 별도 생성 서버를 제공해야 하며**, [저장소에는 API client와 연결 규격만 있다](https://github.com/Ironieser/CineCrew/blob/3eac773704088f65ae2e24054c48fccbd4172f2d/docs/USAGE.md#L155-L172). 같은 backbone 재현에는 serving 구성을 추가해야 한다.
- **근거:** [§3.3–3.4, Algorithm 1](https://arxiv.org/html/2609.07720v1), [공식 구현의 Production loop 설명](https://github.com/Ironieser/CineCrew).

### NoisEasier

- **발표:** 최초 공개 **2026** → **ECCV 2026 본회의**. *NoisEasier: Test-Time Noise Optimization for Text-to-Video Generation*. [arXiv 이력](https://arxiv.org/abs/2608.30194), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-032-37098-3_3).
- **핵심 방법:** 짧은 sampling으로 영상 생성 → 정합성·시각 품질·motion reward 계산 → gradient로 초기 noise와 sampling 단계의 stochastic noise를 갱신 → 수정된 noise로 영상 재생성.
- **Reference video:** X. 텍스트·noise로 생성한 영상을 reward로 평가하며, 외부 reference clip에 맞추는 목적함수는 사용하지 않는다. [§3.1–3.3](https://arxiv.org/html/2608.30194v1#S3).
- **공간·시간 범위:** 매번 영상 전체를 생성한다. 평가에 일부 프레임·clip·optical flow를 쓰더라도 ROI나 성공 프레임을 선택해 고정하는 기능은 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise O. 초기 noise와 중간 stochastic noise tensor를 reward gradient로 직접 갱신한다. seed 번호의 변경 정책은 미명시이며, 모델 간 평가에 같은 seed를 쓴다는 설명을 refinement 간 고정 근거로 쓰지 않았다. [§3.1 Eq. 4, §3.3 Eq. 7, §4.4](https://arxiv.org/html/2608.30194v1).
- **History:** 현재 생성 결과의 reward로 noise를 갱신하며, 과거 prompt·critique·영상 archive를 누적 재입력하지 않는다. optimizer state와 noise의 계승은 표의 누적 history와 구분한다.
- **비교 포인트:** 모델 가중치와 생성 prompt를 유지하면서 **샘플별 noise 변수**를 최적화한다. prompt를 통한 언어적 수정과 reward gradient를 통한 수치적 수정의 직접 대비가 가능하다.
- **공식 코드·재현 구성:** O / O(**T2V-Turbo 경로 중심 정적 확인**). [초기·중간 noise의 reward gradient 최적화 loop](https://github.com/actionlab-cv/NoisEasier/blob/7f526589d920f5359af207453c2e7219ba254568/t2v-turbo/run_ms.py#L213-L347), [환경·모델·실행 안내](https://github.com/actionlab-cv/NoisEasier/blob/7f526589d920f5359af207453c2e7219ba254568/t2v-turbo/README.md), [실험 설정](https://github.com/actionlab-cv/NoisEasier/blob/7f526589d920f5359af207453c2e7219ba254568/t2v-turbo/configs/noiseasier_t2v.json)가 제공된다. 생성·reward checkpoint를 준비해야 하며 quick-start와 논문 비교 설정은 구분해 적용해야 한다.
- **근거:** [§3.1 Eq. 4, §3.3 Eq. 7, Fig. 2](https://arxiv.org/html/2608.30194v1).

### FreeInit

- **발표:** 최초 공개 **2023** → **ECCV 2024 본회의**. *FreeInit: Bridging Initialization Gap in Video Diffusion Models*. [arXiv 이력](https://arxiv.org/abs/2312.07537), [공식 proceedings](https://link.springer.com/chapter/10.1007/978-3-031-72646-0_22).
- **핵심 방법:** 전체 DDIM sampling으로 clean video latent 생성 → 다시 noise 추가 → 시공간 저주파 성분은 유지하고 고주파는 새 random noise로 교체 → 이 noise에서 전체 sampling을 반복.
- **Reference video:** X. 재초기화에 사용하는 clean latent는 직전 sampling에서 자체 생성한 결과이며, 외부 참조 영상이 아니다. [§4 Eq. 8–11](https://arxiv.org/html/2312.07537v2#S4).
- **공간·시간 범위:** 모든 공간·시간 위치를 다시 생성한다. 3D Fourier filter의 mask는 **주파수 선택**이며 pixel ROI나 특정 프레임 선택이 아니다.
- **시드·초기 noise:** 시드 X / 초기 noise O(공식 AnimateDiff 예제 기준). 영상별 seed는 시작에 한 번 설정하고, 반복 중에는 새 난수를 계속 추출한다. 이전 결과를 re-noise한 저주파와 새 난수의 고주파를 섞으므로 시작 tensor는 달라진다. [seed 설정](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/examples/AnimateDiff/scripts/animate_with_freeinit.py#L94-L114), [반복 구현](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/examples/AnimateDiff/animatediff/pipelines/pipeline_animation.py#L568-L590).
- **History:** 직전 생성 latent의 구조를 다음 초기 noise에 전달한다. 과거 여러 수정본을 명시적으로 보관·재입력하는 텍스트·시각 history는 없다.
- **비교 포인트:** 외부 critic 없이 생성 결과의 저주파 구조를 재사용하는 반복 개선이다. prompt는 고정하며, VLM이 오류를 진단하고 수정 범위를 선택하는 방법과 구분한다.
- **공식 코드·재현 구성:** O / O(AnimateDiff 예제 기준). 주파수 혼합 유틸과 반복 sampling pipeline, iteration·filter 설정, 실행 스크립트·환경·checkpoint 안내가 제공된다. 다른 backbone 적용은 별도 통합이 필요하다. [반복 sampling](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/examples/AnimateDiff/animatediff/pipelines/pipeline_animation.py#L568-L620) · [실행 안내](https://github.com/TianxingWu/FreeInit/blob/410286de018fef339c324e49e894a9cf82824f90/README.md#L37-L83).
- **근거:** [§4, Eq. 8–11, Fig. 7, §5.1](https://arxiv.org/html/2312.07537v2). 통상적인 한 번의 denoising 내부 반복과 달리, clean latent를 만든 뒤 sampling을 다시 시작한다.

### Self-Refining Video Sampling

- **발표:** 최초 공개 **2026** → **ICML 2026 본회의**. [arXiv 이력](https://arxiv.org/abs/2601.18577), [공식 proceedings](https://proceedings.mlr.press/v306/jang26d.html).
- **핵심 방법:** 추가 학습·외부 평가 모델 없이, **한 번의 sampling 내부**에서 생성 모델 자체로 latent를 보정한다.
	- **기본 P&P:** 현재 noisy latent에서 clean latent 예측 → 새 Gaussian noise를 섞어 같은 noise level로 되돌림 → 정해진 횟수만 반복한 뒤 다음 timestep으로 진행한다. 완성 영상의 재생성이 아니며, 주로 sampling 초반에 적용한다.
	- **Uncertainty-aware P&P:** 연속한 clean latent 예측의 위치별 차이가 큰 곳을 mask로 선택한다. 불확실한 영역은 새 보정값을 반영하고, 안정적인 영역은 이전 후보 값을 유지해 과도한 보정을 줄인다.
- **Reference video:** X. 외부 목표 영상은 사용하지 않는다. I2V 실험의 입력 이미지는 별도 이미지 조건이다.
- **공간·시간 범위:** 공간 △, 시간 X. **기본 P&P에는 공간 선택이 없고 uncertainty-aware 확장형에만 국소 제어가 있다.** 비교 대상은 RGB로 디코딩한 영상이 아닌 clean latent 예측이다. 모델 연산은 전체 latent에 수행하며, 성공한 최종 프레임을 골라 영구 freeze하지 않는다.
- **시드·초기 noise:** — / —(완성 영상 간 반복이 없어 해당 축 적용 대상 아님). P&P 내부에서는 새 noise를 뽑지만, 전체 sampling을 새 initial noise로 재시작하는 방식과 구분한다.
- **History:** 텍스트 X / 시각 X. 현재·직전 clean 예측의 차이와 이전 latent 후보를 사용하며, 여러 완성 영상·prompt·피드백을 누적 입력하지 않는다.
- **비교 포인트:** 자체 예측의 불안정성으로 수정 위치를 정하는 방법. 우리 설계의 완성 영상 간 ROI 계산·VLM 판단과 비교할 때 **보정 시점과 판단 신호**를 구분해야 한다.
- **공식 코드·재현 구성:** O / O(**Wan T2V/I2V 구현 기준**). [P&P와 uncertainty mask 실제 구현](https://github.com/agwmon/self-refine-video/blob/271be8d1cbc8ca38e48263b46b9568b74ca023aa/functions/pipeline_wan_pnp.py#L637-L741), [실행·모델·환경 안내](https://github.com/agwmon/self-refine-video/blob/271be8d1cbc8ca38e48263b46b9568b74ca023aa/README.md#L33-L48)가 있다. 다만 [T2V demo의 threshold 0.2](https://github.com/agwmon/self-refine-video/blob/271be8d1cbc8ca38e48263b46b9568b74ca023aa/inference_pnp.py#L35-L46)는 [논문 Table 6의 0.25](https://arxiv.org/html/2601.18577v1#A1.SS2)와 달라, 논문 실험 재현 시 설정을 맞춰야 한다.
- **근거:** [§4.2–4.3, Algorithm 1](https://arxiv.org/html/2601.18577v1#S4.SS2), [공식 프로젝트](https://agwmon.github.io/self-refine-video/).

### VideoGrain

- **발표:** 최초 공개 **2025** → **ICLR 2025 본회의**. *VideoGrain: Modulating Space-Time Attention for Multi-Grained Video Editing*. [arXiv 이력](https://arxiv.org/abs/2502.17258), [공식 proceedings](https://proceedings.iclr.cc/paper_files/paper/2025/hash/4bfcebedf7a2967c410b64670f27f904-Abstract-Conference.html).
- **핵심 방법:** 원본 영상과 전역·영역별 편집 prompt를 받아, **객체 종류·개별 객체·객체 일부**를 구분해 편집하는 training-free 방법이다.
	- **영역 구성:** 원본 영상을 DDIM inversion → self-attention feature clustering과 SAM-Track으로 영역·객체를 구분 → 각 영역에 local prompt를 연결한다.
	- **편집 제어:** cross-attention은 prompt가 해당 영역에 집중하게 하고, 시공간 self-attention은 같은 객체의 프레임 간 연결을 강화하면서 다른 객체와의 혼선을 줄인다. 예를 들어 두 사람을 서로 다른 캐릭터로 바꿀 수 있다.
	- **보존:** 수정 mask 밖은 각 denoising timestep에서 원본 inversion latent와 합치는 latent blending으로 보존한다.
- **Reference video:** 표 기준 X. **편집할 원본 영상 입력은 필수**지만, 이는 현재 수정 대상이다. 별도의 목표 동작·내용을 지정하는 reference video와 구분한다.
- **공간·시간 범위:** 공간 O, 시간 X. 여러 ROI를 서로 다른 prompt로 편집하고 나머지를 보존한다. 모든 프레임에 시공간 제어를 적용하며, 실패 프레임만 선별해 재생성하는 기능은 없다.
- **시드·초기 noise:** — / —(완성 영상 간 반복이 없어 해당 축 적용 대상 아님). 원본의 DDIM inversion latent에서 편집 sampling을 수행하며, 반복적인 seed 탐색·초기 noise 최적화 방법은 아니다.
- **History:** 텍스트 X / 시각 X. 현재 원본 영상·영역 mask·편집 prompt·inversion 정보를 사용하며, 여러 과거 편집 결과와 피드백을 누적하지 않는다.
- **비교 포인트:** ROI에 서로 다른 prompt를 적용하는 구체적인 수정 메커니즘이다. 오류 자동 진단이나 ROI 타당성 검증은 제공하지 않는다. 실험 backbone은 SD 1.5이며 Wan 계열에 그대로 적용된 결과는 아니다.
- **공식 코드·재현 구성:** O / O(SD1.5 편집 경로 기준). inversion·영역별 attention 제어·latent blending, 실행 script, 세 granularity별 config·원본 frame/mask 데이터와 모델 다운로드 안내가 있다. 자체 영상의 mask는 안내된 SAM-Track/SAM2로 준비한다. [핵심 제어](https://github.com/knightyxp/VideoGrain/blob/ab9e226b257fdd724ca59372f601ddab6dab4771/video_diffusion/prompt_attention/attention_util.py#L357-L408) · [실행·입력 준비](https://github.com/knightyxp/VideoGrain/blob/ab9e226b257fdd724ca59372f601ddab6dab4771/README.md#L65-L165).
- **근거:** [§3.2–3.4, §4.1, Appendix I](https://arxiv.org/html/2502.17258v1), [공식 구현](https://github.com/knightyxp/VideoGrain).

## 워크숍

### VideoAgent

- **발표:** 최초 공개 **2024** → **RLBRew @ RLC 2025 워크숍** 채택 확인. 비정규(non-archival) 워크숍이며 RLC 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2410.10076), [공식 accepted papers](https://rlbrew2-workshop.github.io/papers.html), [워크숍 성격](https://rlbrew2-workshop.github.io/submit.html).
- **핵심 방법:** 로봇 행동을 위한 video plan을 생성 → 학습된 self-conditioning 모델이 직전 비디오를 입력받아 보정 → VLM이 수용 여부 판단. Suggestive 변형은 언어 피드백도 사용한다.
- **Reference video:** X. 초기 frame·task text에서 시작하고 자체 생성 영상으로 self-conditioning한다. Descriptive-feedback 변형의 고정 few-shot 영상 예시는 critic의 판정 방법을 알려 주는 용도이므로 이 축에서 제외한다. [§3.1–3.2, Appendix B.2·C/Fig. 6](https://arxiv.org/html/2410.10076v3#A2.SS2).
- **공간·시간 범위:** video plan 전체를 수정하며, 공간 ROI나 실패 프레임만 선택하지 않는다.
- **시드·초기 noise:** 시드 X / 초기 noise O(공식 Meta-World 구현 기준). 시작에 seed를 설정한 뒤 refinement 사이에 재설정하지 않으며, 매 생성 시작에서 새 Gaussian noise를 추출한다. [seed 설정](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/experiment/benchmark_mw.py#L21-L23), [refinement loop](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/flowdiffusion/flowdiffusion/inference_utils.py#L401-L440), [시작 noise 생성](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/flowdiffusion/flowdiffusion/goal_diffusion.py#L539-L565).
- **History:** 직전 비디오 하나를 이어 쓰는 구조이므로, 여러 라운드의 누적 텍스트·시각 이력은 X로 분류한다.
- **비교 포인트:** prompt만 고치는 방법과 달리 기존 비디오 자체가 보정 모델의 입력이다. 동명의 video understanding 논문과 구분한다.
- **공식 코드·재현 구성:** O / O. MetaWorld 기본 refinement는 생성–VLM 판정–이전 영상 조건 재생성 loop, 실행 스크립트, 환경 설정, 공개 checkpoint가 연결되어 있다. Suggestive 등 다른 변형까지 완비된다는 의미는 아니다. [공개 자료](https://github.com/Video-as-Agent/VideoAgent/blob/108548efee6a598da6f99a81244ea97fe8d7dd9e/README.md) · [구현·파일 구성](https://github.com/Video-as-Agent/VideoAgent_exp/blob/1d3dfc7ffd31ec27e50a4c09fba0307e9629afe6/experiment/benchmark_mw.py#L79).
- **근거:** [§3.1–3.2, Eq. 12–13, §4.3.1](https://arxiv.org/html/2410.10076v3).

### Bootstrapping Physics-Grounded Video Generation

- **발표:** 최초 공개 **2025**. **ICCV 2025 Perception Test Workshop의 Physics-IQ 챌린지 3위 솔루션 보고·발표**이며, 정규 proceedings 논문 채택은 미확인이다. [arXiv 이력·보고서 설명](https://arxiv.org/abs/2511.20280), [공식 챌린지 안내](https://physics-iq.github.io/workshop/physics-iq-challenge.html), [저자 발표 기록](https://yafeng19.github.io/).
- **핵심 방법:** VLM이 입력 prefix의 물리 단서에서 미래 서술 생성 → LLM이 생성 prompt로 정리 → continuation 생성 → 물리 오류 피드백으로 prompt 수정.
- **Reference video:** O. **외부 3초 prefix 영상**을 VLM의 물리적 미래 예측과 다음 5초 continuation 생성의 조건으로 사용한다. P-Flow의 목표 효과 예시와 달리, 이어 만들 장면의 시작 상태·동역학을 제공한다. [§2](https://arxiv.org/html/2511.20280v1#S2).
- **공간·시간 범위:** 입력 3초 뒤의 5초 continuation 전체를 재생성한다. 주어진 prefix 유지와 성공한 출력 프레임 선택 freeze는 다르다.
- **시드·초기 noise:** 시드 — / 초기 noise —. prompt 수정 후 continuation을 재생성하지만, 반복 사이 seed·시작 noise를 재사용하는지 새로 뽑는지는 명시하지 않는다. [§2 Step 3](https://arxiv.org/html/2511.20280v1#S2).
- **History:** 현재 결과에서 다음 prompt를 만드는 과정은 설명하지만, 누적 history 입력 범위는 미명시이다.
- **비교 포인트:** 영상 continuation이라는 설정 차이를 반영해 비교해야 한다. ICCV 본회의 채택 논문으로 표기하지 않는다.
- **공식 코드·재현 구성:** X / X. GitHub의 논문명·arXiv ID 검색과 저자들의 공개 저장소를 재확인했으나 해당 논문의 공식 구현을 찾지 못했다. [제1저자 논문 목록](https://yafeng19.github.io/)에는 논문·발표 자료가 있으나 코드 링크는 없으며, [제1저자 GitHub](https://github.com/yafeng19)에서도 해당 refinement 구현을 확인하지 못했다.
- **근거:** [§2, Step 1–3](https://arxiv.org/html/2511.20280v1).

### AIGVE-MACS의 refinement 실험

- **발표:** 최초 공개 **2025** → **CVPR 2026 VGBE 워크숍**. *AIGVE-MACS: Unified Multi-Aspect Commenting and Scoring Model for AI-Generated Video Evaluation*. 본회의와 구분한다. [arXiv 이력](https://arxiv.org/abs/2507.01255), [공식 workshop proceedings](https://openaccess.thecvf.com/content/CVPR2026W/VGBE/html/Liu_AIGVE-MACS_Unified_Multi-Aspect_Commenting_and_Scoring_Model_for_AI-Generated_Video_CVPRW_2026_paper.html).
- **핵심 방법:** 평가 VLM이 생성 영상에 대해 여러 측면의 score·comment 출력 → GPT-4.1 Instruction Revisor가 prompt 수정 → HunyuanVideo로 재생성·재평가.
- **Reference video:** X. 현재 instruction과 자체 생성 영상의 score·comment로 개선하며, 별도의 외부 목표 영상은 사용하지 않는다. [§6 Fig. 5](https://arxiv.org/html/2507.01255v1#S6).
- **공간·시간 범위:** 영상 전체를 다시 생성한다. 평가기의 dynamic frame sampling은 관찰 전략이며, 선택된 프레임만 수정하는 방법은 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. HunyuanVideo 재생성은 명시하지만, refinement 사이 seed 및 시작 noise의 재사용·재추출 방식은 명시하지 않는다. [§6](https://arxiv.org/html/2507.01255v1#S6).
- **History:** 현재 instruction·영상의 평가 피드백을 사용하는 것은 명시되지만, 여러 과거 피드백·영상을 누적 입력하는 범위는 미명시이다.
- **비교 포인트:** 주 기여는 평가 모델이고, 반복 생성은 적용 실험이다. PQSG처럼 평가기의 세밀한 진단이 실제 refinement에 어떻게 쓰이는지를 비교할 수 있다.
- **공식 코드·재현 구성:** △ / X. 평가 checkpoint와 README의 추론 예제는 공개되어 있지만, 표에서 비교하는 prompt 수정→HunyuanVideo 재생성 loop는 없다. 추론 예제의 `models` import 대상도 공개 tree에 없어 완성된 실행 배포는 아니다. [공개 자료](https://huggingface.co/xiaoliux/AIGVE-MACS/blob/8e3721f657e473af8bed6e7cacbfb79602427330/README.md) · [구현·파일 구성](https://huggingface.co/xiaoliux/AIGVE-MACS/tree/8e3721f657e473af8bed6e7cacbfb79602427330).
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
- **공식 코드·재현 구성:** O / △. Stage1과 Wan 기반 **SSPO 생성·flow·VLM 평가·prompt 수정 loop는 존재**한다. 다만 누적 history/backtracking과 Stage3 전용 학습 코드·설정이 없고, 공개 SSPO 파일에는 경로 문자열 문법 오류도 있어 ++ 전체 재현은 보완이 필요하다. [SSPO loop](https://github.com/Vchitect/RAPO/blob/c7c2cb9b3e3f5fa073d05528232ed0e28d1a7912/examples/Stage2_SSPO/phyaware_wan2.1.py#L304-L364) · [공개 범위](https://github.com/Vchitect/RAPO/blob/c7c2cb9b3e3f5fa073d05528232ed0e28d1a7912/README.md#L108-L121) · [문법 오류](https://github.com/Vchitect/RAPO/blob/c7c2cb9b3e3f5fa073d05528232ed0e28d1a7912/examples/Stage2_SSPO/phyaware_wan2.1.py#L247-L256).
- **근거:** [§4.1, Table 4, §4.2](https://arxiv.org/abs/2510.20206v2). 표는 논문의 전체 설계 기준이다.

### NEWTON

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. [arXiv 이력](https://arxiv.org/abs/2605.18396), [공식 프로젝트](https://newton026.github.io/newton/).
- **핵심 방법:** Planner가 물리 계산·keyframe 생성·prompt refinement 도구 선택 → Executor 실행 → Verifier가 의미·물리 점수 반환 → 여러 cycle 후 최고 점수 영상 선택.
- **Reference video:** X. 입력 query를 바탕으로 prompt·keyframe·물리 계산 결과를 구성하며, 주요 pipeline에 외부 reference clip은 없다. keyframe은 생성한 이미지 조건이다. [§4.1.1–4.1.3](https://arxiv.org/html/2605.18396v2#S4.SS1).
- **공간·시간 범위:** 생성 조건을 고쳐 영상 전체를 재생성한다. keyframe 생성은 기존 성공 프레임을 freeze하는 기능이 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 재계획으로 prompt·keyframe 등의 조건을 바꾸지만, 논문의 생성기에서 round별 seed·초기 noise를 어떻게 관리하는지는 미명시이다. [§4.1.2, §5.1](https://arxiv.org/html/2605.18396v2).
- **History:** reasoning·도구 입력/출력·점수를 누적하지만, **생성 비디오는 memory에서 명시적으로 제외**한다. 텍스트 O, 시각 X.
- **비교 포인트:** Generator는 고정하고 Planner를 학습한다. 도구 실행 이력과 실제 생성 영상의 시각 이력을 구분할 수 있다.
- **공식 코드·재현 구성:** O / △. OpenNewton의 실행 가능한 형태의 반복 loop·도구·설정은 공개되어 있다. 다만 공개판은 API planner+Seedance이며, 논문의 Flow-GRPO planner 학습 코드·checkpoint와 LTX/Veo 실험 구성이 갖춰진 배포는 아니다. [공개 자료](https://github.com/CUTEPKQ/NEWTON/blob/072bda240a5991bc4da6ff8b6cd5f6f33fcfc058/loop/run_loop.py#L1) · [구현·파일 구성](https://github.com/CUTEPKQ/NEWTON/blob/072bda240a5991bc4da6ff8b6cd5f6f33fcfc058/README.md#L89).
- **근거:** [§4.1.1–4.1.3, §4.2](https://arxiv.org/html/2605.18396v2).

### SPIRAL

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. 현재 제목은 *SPIRAL: Self-Evolving Action-Conditioned Video Generation via Reflective Planning Agents*. [arXiv 이력](https://arxiv.org/abs/2603.08403), [공식 프로젝트](https://yuyang-cloud.github.io/spiral/).
- **핵심 방법:** 목표를 action step으로 분해 → segment 생성·평가 → 실패한 지시문 수정·재생성 → 반복 실패 시 해당 step부터 이후 계획 변경.
- **Reference video:** X. 초기 visual state와 goal에서 시작하며, 이후 영상 문맥은 성공 판정을 받은 자체 생성 segment의 memory이다. 외부 목표 clip은 입력하지 않는다. [§3.1–3.3, Appendix B.4](https://arxiv.org/html/2603.08403v3#S3.SS1).
- **공간·시간 범위:** 실패 segment의 공간 전체를 다시 만들고 성공한 앞 구간은 유지한다. 공간 X, 시간 O이며 같은 clip 내부의 프레임별 freeze와 구분한다.
- **시드·초기 noise:** 시드 — / 초기 noise —. inference retry의 정책은 미명시이다. shared initial noise 조건은 GRPO 학습의 후보 group에 관한 것이므로, 추론 시 refinement에도 noise를 고정한다고 해석하지 않았다. [Appendix B.4–B.5](https://arxiv.org/html/2603.08403v3#A2.SS4).
- **History:** Planner는 성공한 과거 segment의 시각·의미 이력과 실패 시도·진단을 사용한다. Critic은 현재 segment를 평가한다. 시간적으로 앞선 실행 이력이 중심이므로 두 history 축은 △.
- **비교 포인트:** 시간 선택 수정·시각 이력의 중요한 선행 연구. 동일 clip의 수정본 비교 및 ROI 타당성 판단과 대비해야 한다. 별도의 agent/generator 학습도 다룬다.
- **공식 코드·재현 구성:** X / X. GitHub 검색과 저자 계정을 재확인했다. [저자 프로젝트 저장소](https://github.com/yuyang-cloud/spiral/tree/e7dc280938d59f369e8ac6c1a062bc7facac8bcb)는 HTML·그림·결과 영상만 제공하며, PlanAgent–CriticAgent–VideoGenerator refinement 구현은 없다.
- **근거:** [§3.1–3.5, Appendix B.1/B.3/B.4, F.2](https://arxiv.org/html/2603.08403v3).

### CoAgent

- **발표:** 최초 공개 **2025**. **학회 채택 미확인**. *CoAgent: Collaborative Planning and Consistency Agent for Coherent Video Generation*. [arXiv 이력](https://arxiv.org/abs/2512.22536).
- **핵심 방법:** storyboard를 shot으로 분해 → 객체 상태 memory를 참조해 생성 → VLM 검증 → 의미 오류이면 storyboard 수정, 외관 오류이면 첫 프레임·양끝 프레임 조건 모드로 변경 → 해당 shot 재생성.
- **Reference video:** X. concept·storyboard로 시작하며, 직전 shot과 appearance memory는 자체 생성 결과에서 갱신된다. 양끝 frame 조건도 외부 reference video와 구분한다. [§3.1, §3.3–3.5](https://arxiv.org/html/2512.22536v1#S3.SS1).
- **공간·시간 범위:** 선택한 shot 전체를 다시 생성한다. 공간 X, 시간 O이며 shot 내부의 프레임별 freeze는 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 실패 shot의 description이나 생성 모드를 변경하지만, 재생성 사이 seed·시작 noise 처리 방식은 명시하지 않는다. [§3.5 Algorithm 2](https://arxiv.org/html/2512.22536v1#S3.SS5).
- **History:** 객체 속성과 appearance embedding을 shot 간 memory로 유지한다. 이는 같은 shot의 모든 실패 prompt·영상 이력이 아닌 **장면 간 상태 기억**이므로 두 축은 △.
- **비교 포인트:** 검증 결과가 prompt뿐 아니라 생성 조건 모드 선택으로 이어진다. 객체 시각 memory와 ROI가 표시된 수정 history의 역할 차이를 비교할 수 있다.
- **공식 코드·재현 구성:** X / X. GitHub의 논문명·arXiv ID 검색과 [제1저자 공개 저장소](https://github.com/zqinglin)를 재확인했으나 해당 논문의 공식 구현을 찾지 못했다. 저자의 Wan2.1Server는 생성 서버 코드이며 CoAgent의 planner·verifier·refinement 전체 구현으로 확인되지 않아 포함하지 않았다. [논문](https://arxiv.org/html/2512.22536v1)의 prompt·모델 설정 공개만으로 재현 구성을 O로 판정하지 않았다.
- **근거:** [§3.3–3.5, Algorithm 1–2](https://arxiv.org/html/2512.22536v1). 수정 단위는 Algorithm 2의 shot 재생성 호출을 기준으로 판정했다.

### A²RD · 기본형

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *A²RD: Agentic Autoregressive Diffusion for Long Video Consistency*. [arXiv 이력](https://arxiv.org/abs/2605.06924), [Google Research 공식 목록](https://research.google/pubs/a2rd-agentic-autoregressive-diffusion-for-long-video-consistency-2/).
- **핵심 방법:** 관련 장면 상태·시각 자료 검색 → segment 생성 → 일관성·품질 평가 → 성공·실패 사례를 검색해 prompt 개선(MAPO) → 필요하면 ending frame도 편집한 뒤 segment 재생성.
- **Reference video:** X. 사용자 제공 reference frame은 허용하지만 이미지 조건이며, video memory에는 자체 합성한 이전 segment를 저장한다. 외부 reference clip을 입력받는 설정과 구분한다. [§3.1–3.2](https://arxiv.org/html/2605.06924v1#S3.SS1).
- **공간·시간 범위:** video 수정은 segment 전체를 재생성하므로 공간 X, 시간 O. boundary image 편집은 video ROI만 수정하거나 성공 프레임을 freeze하는 절차와 구분한다.
- **시드·초기 noise:** 시드 — / 초기 noise —. 기본형의 segment self-improvement에서 prompt·ending frame 수정은 설명하지만, 재생성 사이 seed·시작 noise 정책은 확인되지 않는다. [§3.3, Appendix C](https://arxiv.org/html/2605.06924v1#S3.SS3).
- **History:** 텍스트는 사전·온라인 사례 DB에서 관련 성공·실패 사례를 검색한다. 시각은 관련 과거 segment 영상·keyframe을 참조한다. 동일 segment의 전체 수정 archive 입력과 달라 두 축은 △.
- **비교 포인트:** **기본 A²RD만** 이 표의 대상이다. A²RD-Parallel은 video self-improvement를 생략한다. 전체 history 입력과 관련 사례 검색의 차이도 비교할 수 있다.
- **공식 코드·재현 구성:** X / X. 공식 GitHub는 존재하지만 현재 README·라이선스·소개 자산만 있고, MVMem/HITS/MAPO와 실행 코드는 공개되어 있지 않다. [공개 자료](https://github.com/dxlong2000/AARD/tree/05b5bf7bad547cd0cfad8634a54d84f20a784f5c).
- **근거:** [§3.1–3.4, Eq. 9–10, MAPO 문단](https://arxiv.org/html/2605.06924v1).

### VQQA

- **발표:** 최초 공개 **2026**. **학회 채택 미확인**. *VQQA: An Agentic Approach for Video Evaluation and Quality Improvement*. [arXiv 이력](https://arxiv.org/abs/2603.12310), [공식 프로젝트](https://yiwen-song.github.io/vqqa/).
- **핵심 방법:** 입력에 맞는 평가 질문 생성 → 영상의 답변·점수·이유 수집 → 낮은 점수의 QA로 prompt 수정 → 영상 재생성. 최종 후보는 원래 prompt에 대한 평가로 선택한다.
- **Reference video:** X(원문의 T2V/I2V refinement 기준). 원래 prompt·reference image를 조건으로 자체 생성 영상을 개선하며, 외부 reference clip을 사용한 설정은 제시하지 않는다. [§3.1–3.2, Appendix C.1.2](https://arxiv.org/html/2603.12310v1#S3.SS1).
- **공간·시간 범위:** 세부 질문으로 오류를 찾지만 생성은 영상 전체에 적용한다. 국소 QA는 공간 mask나 프레임 선택 수정이 아니다.
- **시드·초기 noise:** 시드 O / 초기 noise O(CogVideoX-5B·I2V 실험 기준). 첫 생성은 고정 seed, 이후 모든 refinement는 새 seed를 균등 추출한다. 초기 noise O는 이 seed 정책과 사용 라이브러리의 Gaussian latent 생성 동작을 대조한 판정이다. seed를 받지 않는 논문의 Veo 3.1 API 실험에는 일반화하지 않는다. [Appendix C.1–C.2](https://arxiv.org/html/2603.12310v1#A3.SS2), [공식 CogVideoX sampler](https://github.com/huggingface/diffusers/blob/0f252be0ed42006c125ef4429156cb13ae6c1d60/src/diffusers/pipelines/cogvideo/pipeline_cogvideox.py#L325-L349).
- **History:** prompt 템플릿은 과거 prompt·VQA·점수의 **선택적 텍스트 history**를 지원하므로 △. 보존 길이는 미명시이며, 이전 영상들을 다음 refinement에 누적 재입력하는 기능은 정의하지 않아 시각 X.
- **비교 포인트:** 질문 기반 피드백과 선택적 history를 결합한다. 최종 후보 pool의 보관·선정은 다음 refinement의 시각 history 입력과 구분한다.
- **공식 코드·재현 구성:** X / X. GitHub 검색과 저자 계정에서 [논문 프로젝트 저장소](https://github.com/yiwen-song/vqqa/tree/f9fabbff6f92a70b8f84526ac3d657e8c140c572)를 찾았으나 README·HTML·그림·결과 영상만 있다. QG–QA–PR 반복과 최종 rater의 실행 코드는 공개되어 있지 않다.
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
