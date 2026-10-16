## 분류 요약

- **조사 기준일:** 2026-10-16. 총 **13편**: 본회의·저널 12편, 데모 트랙 1편.
- **조사 출발점:** VideoRepair의 Related Works에서 직접 인용한 **OPT2I, SLD, T2I-Copilot, PromptSculptor**를 포함하고, 관련 연구의 참고문헌에서 반복 개선·국소 편집 방법을 확장했다. [VideoRepair §4](https://arxiv.org/html/2411.15115v3#S4).
- **비교 범위:** diffusion 기반 이미지 생성에서 ① 완성 이미지의 피드백을 통한 자동 반복 개선, ② 기존 이미지의 공간 국소 편집, ③ sampling 내부의 별도 반복 최적화. 단순히 denoising step을 여러 번 수행한다는 이유만으로 포함하지 않았다.
- **연도 기준:** 최초 공개는 arXiv v1 연도, 발표처는 확인한 최종 학회·저널을 기준으로 한다. 표와 아래 설명의 순서는 같다.
- **기호:** O = 방법에 명시적으로 포함, X = 해당하지 않음, △ = 선택적 기능·분기·제한된 이력, — = 정책 미확인 또는 비교할 외부 반복이 없음. 해당 이유는 논문별로 설명한다.

| 논문 | 최초 공개 | 발표처·연도 | 완성본 자동 반복 | 공간 국소 수정 | 누적 텍스트 이력 | 누적 시각 이력 | 시드 변경 | 초기 noise 변경 | 외부 이미지 입력 |
| --- | :---: | --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **SLD** | 2023 | CVPR 2024 | O | O | X | X | — | O | △ |
| **OPT2I · 기본 LDM 설정** | 2024 | TMLR 2024 | O | X | △ | X | X | X | X |
| T2I-Copilot | 2025 | ICCV 2025 | O | O | X | X | O | O | △ |
| Idea2Img | 2023 | ECCV 2024 | O | X | O | O | — | O | △ |
| GenArtist | 2024 | NeurIPS 2024 | O | O | — | — | △ | △ | △ |
| ReflectionFlow | 2025 | ICCV 2025 | O | X | X | X | O | O | X |
| Divide, Evaluate, and Refine | 2023 | NeurIPS 2023 | O | X | X | X | X | X | X |
| Ranni | 2023 | CVPR 2024 | X | O | X | X | — | — | X |
| DiffEdit | 2022 | ICLR 2023 | X | O | X | X | — | — | O |
| Prompt-to-Prompt | 2022 | ICLR 2023 | X | △ | X | X | X | X | △ |
| Blended Latent Diffusion | 2022 | ACM TOG / SIGGRAPH 2023 | X | O | X | X | — | — | O |
| Attend-and-Excite | 2023 | ACM TOG / SIGGRAPH 2023 | X | X | X | X | — | — | X |
| PromptSculptor | 2025 | EMNLP 2025 System Demonstrations | O | X | — | X | — | — | X |

- **완성본 자동 반복:** 완성된 이미지 평가 → 수정 결정 → 다음 이미지 생성·편집의 자동 loop. 사용자 지시에 따른 연속 편집과 sampling 내부 최적화는 X로 구분한다. 따라서 이 열이 X여도 국소 편집 방법으로 포함될 수 있다.
- **공간 국소 수정:** 기존 이미지의 일부를 mask·latent 합성·편집 도구로 수정하는 기능. 최초 생성의 layout 지정이나 특정 token의 attention 강화만으로는 O가 아니다. O는 모든 분기에서 국소 수정한다는 뜻이 아니며, **국소/전체 수정 선택은 T2I-Copilot·GenArtist 본문에 구분**했다. ROI 밖 픽셀의 완전 고정도 뜻하지 않는다.
- **누적 이력:** 여러 과거 round의 정보를 다음 의사결정에 다시 입력하는지. O는 각 round의 대표 결과를 누적하는 경우도 포함한다. OPT2I처럼 전체 기록 중 top-k만 유지하면 △, 현재 상태·직전 결과·best-so-far 하나만 사용하면 X이다. 로그 저장, 현재 latent, 누적 가중치, 실행 tree만으로 누적 시각 이력 O로 보지 않는다.
- **시드·초기 noise:** 같은 목표의 생성·수정 시도 사이를 비교한다. seed는 RNG의 숫자 설정, 초기 noise는 다음 sampling의 시작 tensor이다. 부분 재추출·re-noising도 noise 변경에 포함하지만, 한 sampling의 정상적인 timestep 갱신이나 같은 round의 후보 다양화만으로 O로 판정하지 않는다. **seed를 다시 설정하지 않아도 새 noise를 추출할 수 있다.**
- **단일 편집의 noise:** DiffEdit·Blended Latent Diffusion·Attend-and-Excite에는 비교할 자동 외부 round가 없어 —로 표시했다. Prompt-to-Prompt는 예외적으로 원본/편집 경로에 동일 seed·시작 noise를 쓰는 명시적 비교가 있어 X/X이다. Ranni의 —는 연속 편집 사이 정책 미확인이다.
- **외부 이미지 입력:** 비디오 문서의 reference 축을 이미지에 맞게 확장한 항목. 사용자가 제공한 참조 이미지뿐 아니라 **외부 편집 원본**도 포함하며, 역할은 본문에 구분한다. 자체 생성한 직전 결과·과거 draft는 history/수정 대상으로 취급하고 외부 reference로 세지 않는다.
- **구현 확인 범위:** T2I-Copilot·Idea2Img·ReflectionFlow·Divide, Evaluate, and Refine의 seed/noise 또는 history 판정은 아래 링크의 공식 구현으로 보완했다. 다른 backbone·API의 정책까지 일반화하지 않는다.

## 본회의·저널

### SLD

- **논문·발표:** *Self-correcting LLM-controlled Diffusion Models*. 최초 공개 **2023** → **CVPR 2024**. [arXiv](https://arxiv.org/abs/2311.16090), [공식 proceedings](https://openaccess.thecvf.com/content/CVPR2024/html/Wu_Self-correcting_LLM-controlled_Diffusion_Models_CVPR_2024_paper.html).
- **핵심 방법:** 원문 prompt에서 필요한 객체를 추출 → 현재 이미지의 객체·bbox 검출 → LLM이 올바른 layout 제안 → 두 layout의 차이를 객체 추가·삭제·이동·속성 수정으로 실행한다. [§3, Algorithm 1](https://openaccess.thecvf.com/content/CVPR2024/papers/Wu_Self-correcting_LLM-controlled_Diffusion_Models_CVPR_2024_paper.pdf).
- **반복·공간 범위:** 완성 이미지의 오류를 다시 검사하는 자동 loop이며, 객체 단위의 국소 수정을 수행한다. 수정할 layout이 없거나 최대 round에 도달하면 종료한다.
- **영역 선택·보존:** 검출 bbox를 SAM으로 정밀화하고 객체 latent를 합성한다. 삭제·이동으로 빈 영역은 noise로 채운다. 초기 denoising에는 보존 latent를 유지하지만 **후반에는 전체를 unfreeze**하므로 영역 밖의 최종 픽셀을 정확히 고정하지는 않는다.
- **History:** controller는 원문 prompt와 **현재 검출 layout**을 본다. 여러 과거 이미지·prompt·피드백을 누적 입력하는 구조는 아니다.
- **시드·초기 noise:** 시드 — / 초기 noise O. round별 seed 규칙은 미명시이나 수정 영역을 Gaussian noise로 재초기화하는 연산은 명시되어 있다. [§3.2.1](https://openaccess.thecvf.com/content/CVPR2024/papers/Wu_Self-correcting_LLM-controlled_Diffusion_Models_CVPR_2024_paper.pdf).
- **외부 이미지·추가 학습:** 기본 T2I 교정은 자체 생성 이미지를 사용하며 외부 이미지 편집도 지원한다(△). 기존 모델을 활용하는 training-free 방법이고, 공식 실험의 latent operation 기반은 **LMD+**이다.
- **비교 포인트:** 자동 오류 진단과 국소 수정이 연결된 직접 비교 대상. ROI는 **현재 이미지의 검출 결과와 목표 layout의 차이**에서 얻으며, 연속 생성 결과 두 장의 변화에서 얻는 방식과 구분된다.

### OPT2I

- **논문·발표:** *Improving Text-to-Image Consistency via Automatic Prompt Optimization*. 최초 공개 **2024** → **TMLR 2024, 11월 게재**. [arXiv](https://arxiv.org/abs/2403.17804), [TMLR 최종 논문](https://openreview.net/pdf?id=g12Gdl6aDL).
- **핵심 방법:** LLM이 prompt 후보들을 제안 → 이미지 생성 → 원래 요구에 대한 DSG 등의 정합성 점수 계산 → 점수가 높은 prompt·평가 정보를 다음 제안에 사용한다. 생성 모델의 가중치를 바꾸지 않고 prompt를 최적화한다. [§2](https://arxiv.org/html/2403.17804v1#S2).
- **반복·공간 범위:** 완성 이미지 평가 후 **전체 이미지를 재생성**한다. 특정 객체의 점수를 평가하더라도 그 객체 영역만 수정하는 것은 아니다.
- **영역 선택·보존:** ROI mask나 기존 이미지의 국소 보존 경로는 없다. 원문 의미를 유지하면서 생성기가 더 잘 이해할 prompt를 찾는다.
- **History:** 누적 후보 중 **상위 k개의 prompt·점수·세부 평가 정보**를 텍스트로 사용한다(기본 k=5). 전체 round 기록은 아니므로 텍스트 △이며, 이미지 자체를 누적 입력하지 않으므로 시각 X이다.
- **시드·초기 noise:** 표의 X/X는 **기본 LDM 설정** 기준. prompt별 4개 seed를 iteration 사이에도 동일하게 사용한다. CDM-M의 batch 생성은 seed 고정이 불가능하며 non-fixed-seed ablation도 있으므로 모든 실험이 고정인 것은 아니다. [§3.1, Appendix B.3](https://arxiv.org/html/2403.17804v1).
- **외부 이미지·추가 학습:** 외부 reference image 없이 텍스트에서 생성한다. 추가 학습은 필요 없다.
- **비교 포인트:** prompt 최적화와 제한된 텍스트 memory의 비교 대상. 시각 history와 공간 보존은 별도로 설계해야 하는 부분이다.

### T2I-Copilot

- **논문·발표:** *A Training-Free Multi-Agent Text-to-Image System for Enhanced Prompt Interpretation and Interactive Generation*. 최초 공개 **2025** → **ICCV 2025**. [arXiv](https://arxiv.org/abs/2507.20536), [공식 proceedings](https://openaccess.thecvf.com/content/ICCV2025/html/Chen_T2I-Copilot_A_Training-Free_Multi-Agent_Text-to-Image_System_for_Enhanced_Prompt_Interpretation_ICCV_2025_paper.html).
- **핵심 방법:** 사용자 의도 해석 → 생성·편집 모델 선택 → 이미지 생성 → 품질·정합성 평가 → 수정 지시와 다음 도구 선택을 반복한다. FLUX, PowerPaint, Grounding-SAM2 등을 연결한다. [§3](https://arxiv.org/html/2507.20536v1#S3).
- **반복·공간 범위:** 완성 이미지의 자동 피드백 loop. **국소 편집이 가능한 경우 PowerPaint로 수정하고, 해당 편집으로 해결하기 어려우면 FLUX로 전체 재생성**한다.
- **영역 선택·보존:** 언어로 지목한 객체의 위치를 찾는 모델과 segmentation, 또는 사용자 canvas에서 mask를 얻는다. 보존은 선택한 편집 모델에 의존하며 시스템 전체의 픽셀 freeze를 보장하지 않는다. [§3.2](https://arxiv.org/html/2507.20536v1#S3.SS2).
- **History:** 공개 구현의 evaluator는 현재 이미지·prompt·최초 의도 보고서를, 모델 선택기는 직전 feedback·보고서를 사용한다. 전체 round의 로그 저장과 누적 LLM 입력은 다르다. [입력 구성](https://github.com/SHI-Labs/T2I-Copilot/blob/d6f702267ef830088871289d30a0a0ba4af83f53/main.py#L1021-L1066), [평가 입력](https://github.com/SHI-Labs/T2I-Copilot/blob/d6f702267ef830088871289d30a0a0ba4af83f53/main.py#L1540-L1583).
- **시드·초기 noise:** 공식 구현 O/O. 첫 생성 이후 재시도마다 새 seed를 선택해 생성·편집 모델에 전달한다. FLUX의 전체 noise 재추출과 PowerPaint의 원본·mask를 이용한 편집을 구분해야 한다. [재시도 분기](https://github.com/SHI-Labs/T2I-Copilot/blob/d6f702267ef830088871289d30a0a0ba4af83f53/main.py#L1490-L1534).
- **외부 이미지·추가 학습:** 선택적으로 외부 reference/편집 원본을 입력할 수 있다(△). 시스템 구성에 추가 학습은 필요 없다.
- **비교 포인트:** 국소 편집과 전체 재생성을 선택하는 분기가 이미 존재한다. 우리 설계와 비교할 때에는 **ROI의 타당성을 무엇으로 판정하는지**, 분기가 이미지 전체와 프레임 중 어느 단위인지까지 명시해야 한다.

### Idea2Img

- **논문·발표:** *Iterative Self-Refinement with GPT-4V for Automatic Image Design and Generation*. 최초 공개 **2023** → **ECCV 2024**. [arXiv](https://arxiv.org/abs/2310.08541), [공식 proceedings](https://www.ecva.net/papers/eccv_2024/papers_ECCV/html/5515_ECCV_2024_paper.php).
- **핵심 방법:** GPT-4V가 사용자의 IDEA를 여러 prompt로 구체화 → draft 이미지들 생성 → 가장 좋은 draft 선택 → 부족한 점을 피드백 → 다음 prompt를 제안한다. [§3](https://arxiv.org/html/2310.08541v2#S3).
- **반복·공간 범위:** 완성 draft 사이의 자동 반복이며 매번 전체 이미지를 생성한다. 피드백이 특정 객체를 지목하더라도 실제 수정 공간을 제한하지는 않는다.
- **영역 선택·보존:** mask 등으로 기존 이미지의 공간적 보존을 강제하는 연산은 없다.
- **History:** **모든 과거 round의 선택된 prompt·draft 이미지·feedback**을 image-text memory로 누적한다. 각 round의 후보 전체가 아니라 **선택된 대표 이미지 한 장씩**이라는 점에 유의한다. [§3, Eq.5](https://arxiv.org/html/2310.08541v2#S3).
- **시드·초기 noise:** 시드 — / 초기 noise O(공개 T2I 구현). 호출마다 이전 noise를 전달하지 않고 새로 sampling하며 seed를 재설정하는 정책은 없다. [공식 inference](https://github.com/zyang-ur/idea2img/blob/dd3a25e0ff2994f423616322cd9ab99d9e3ba024/idea2img_pipeline.py#L294-L303).
- **외부 이미지·추가 학습:** IDEA에 참조 이미지를 넣어 GPT-4V가 해석할 수 있다(△). 기본 generator에는 텍스트가 전달되고 별도의 img2img 실험도 있다. 추가 학습은 필요 없다. [§4.1](https://arxiv.org/html/2310.08541v2#S4.SS1).
- **비교 포인트:** 누적 시각·텍스트 이력의 직접 비교 대상. 우리 설계의 ROI 표시 history와 비교하려면 **어떤 대표 결과를 저장하고 어디를 주목하도록 표시하는지**를 구분할 수 있다.

### GenArtist

- **논문·발표:** *Multimodal LLM as an Agent for Unified Image Generation and Editing*. 최초 공개 **2024** → **NeurIPS 2024 본회의**. [arXiv](https://arxiv.org/abs/2407.05600), [공식 proceedings](https://proceedings.neurips.cc/paper_files/paper/2024/hash/e7c786024ca718f2487712bfe9f51030-Abstract-Conference.html).
- **핵심 방법:** 요구를 생성·편집 연산으로 분해하고 실행 tree를 만든다. 각 도구의 결과를 검증해 성공하면 다음 연산으로 진행하고, 실패하면 다른 도구·seed를 사용하는 분기로 되돌아간다. [§3.1](https://arxiv.org/html/2407.05600v2#S3.SS1).
- **반복·공간 범위:** 완성 이미지 검증과 재시도·교정을 수행한다. 객체 편집 도구를 통한 국소 수정과 전체 생성 도구 재호출을 모두 지원한다.
- **영역 선택·보존:** detector·segmentation 등 보조 도구로 객체 위치를 제공하고 DiffEdit·AnyDoor 등의 도구를 사용한다. 새 국소 diffusion 연산 하나를 제안하기보다 **기존 편집 도구의 선택·조합·검증**이 중심이며 보존 강도는 도구별로 다르다. [§3.2–3.3, Table 1](https://arxiv.org/html/2407.05600v2#S3.SS2).
- **History:** 실행 tree와 성공/실패 상태는 유지하지만, 과거 text/image를 어느 범위까지 MLLM 입력에 누적하는지는 확인되지 않아 —/—로 표시한다.
- **시드·초기 noise:** △/△. 품질이 부족할 때 seed를 바꿔 전체 재생성하는 분기가 명시되어 있다. 모든 국소 편집 도구가 매번 같은 정책을 사용한다는 뜻은 아니다. [§3.1](https://arxiv.org/html/2407.05600v2#S3.SS1).
- **외부 이미지·추가 학습:** 외부 편집 원본·subject image·조건 이미지를 선택적으로 사용한다(△). agent를 별도 학습하지 않고 기존 생성·편집 도구를 조합한다.
- **비교 포인트:** 수정 결과의 검증과 실패 시 다른 연산으로 전환하는 구조. ROI를 선택하는 단계뿐 아니라 **수정 실행 후 성공 여부를 확인하는 단계**도 비교할 수 있다.

### ReflectionFlow

- **논문·발표:** *From Reflection to Perfection: Scaling Inference-Time Optimization for Text-to-Image Diffusion Models via Reflection Tuning*. 최초 공개 **2025** → **ICCV 2025**. [arXiv](https://arxiv.org/abs/2504.16080), [공식 proceedings](https://openaccess.thecvf.com/content/ICCV2025/html/Zhuo_From_Reflection_to_Perfection_Scaling_Inference-Time_Optimization_for_Text-to-Image_Diffusion_ICCV_2025_paper.html).
- **핵심 방법:** 이미지 후보 생성 → verifier 평가 → 오류와 수정 방향을 reflection으로 작성 → **현재 이미지·prompt·reflection을 함께 받는 학습된 corrector**로 다음 이미지 생성. 후보 수 확대와 순차 수정 횟수 확대를 결합한다. [§3.1, §3.3–3.4](https://arxiv.org/html/2504.16080v1#S3).
- **반복·공간 범위:** 완성 이미지 사이의 자동 loop. 수정할 내용은 reflection에 적지만 명시적 ROI mask로 sampling 공간을 제한하지 않으므로 공간 국소 수정 X이다.
- **영역 선택·보존:** 직전 이미지를 시각 조건으로 넣어 활용한다. 특정 영역 밖의 latent나 픽셀을 고정하는 방식은 아니다.
- **History:** 기본 알고리즘은 직전 이미지·reflection과 현재 후보 평가를 사용한다. 모든 결과를 최종 후보 선택용으로 보관하는 것과 다음 수정 입력에 전체 history를 넣는 것은 다르다. [Appendix C, Algorithm 1](https://arxiv.org/html/2504.16080v1#A3).
- **시드·초기 noise:** 공개 FLUX 구현 O/O. round마다 새 random seed들을 뽑아 RNG를 설정하고 corrector 생성에서 새 noise를 sampling한다. 표의 판정은 공식 구현 기준이다. [round loop](https://github.com/Diffusion-CoT/ReflectionFlow/blob/5aaddacd0e7e799f617846bdffd0dc7828eabd72/tts/tts_reflectionflow.py#L591-L625), [seed 설정](https://github.com/Diffusion-CoT/ReflectionFlow/blob/5aaddacd0e7e799f617846bdffd0dc7828eabd72/tts/utils.py#L131-L155), [corrector 호출](https://github.com/Diffusion-CoT/ReflectionFlow/blob/5aaddacd0e7e799f617846bdffd0dc7828eabd72/tts/tts_reflectionflow.py#L310-L323).
- **외부 이미지·추가 학습:** T2I loop의 이미지 조건은 자체 생성 결과이므로 외부 이미지 X. **GenRef로 FLUX corrector를 추가 학습**하며, training-free prompt 최적화와 구분해야 한다. [공식 모델 구성](https://github.com/Diffusion-CoT/ReflectionFlow#-models-and-datasets).
- **비교 포인트:** 피드백을 prompt 재작성에만 쓰지 않고 **이미지와 함께 생성기에 직접 조건으로 전달**하는 방법. 추가 학습 비용과 실제 공간 제어의 유무가 비교 축이다.

### Divide, Evaluate, and Refine

- **논문·발표:** *Evaluating and Improving Text-to-Image Alignment with Iterative VQA Feedback*. 최초 공개 **2023** → **NeurIPS 2023 본회의**. [arXiv](https://arxiv.org/abs/2307.04749), [공식 proceedings](https://papers.nips.cc/paper_files/paper/2023/hash/dfd0bd56e8a6f82d1619f5d093d5f9ca-Abstract-Conference.html).
- **핵심 방법:** prompt를 확인 가능한 assertion·질문으로 분해 → 완성 이미지에 VQA 적용 → 가장 잘 반영되지 않은 assertion의 가중치 증가 → 다시 생성한다. prompt embedding 가중치 또는 attention 기반 guidance 강도를 조절한다. [§3.2, Eq.3–6](https://arxiv.org/html/2307.04749v2#S3.SS2).
- **반복·공간 범위:** **완성 이미지 → VQA → 전체 재생성**의 외부 loop. attention variant에는 sampling 내부 latent 갱신도 있지만, VQA 피드백을 받는 반복은 완성본 사이에서 일어난다.
- **영역 선택·보존:** 수정할 assertion/token을 고르는 것이며 공간 ROI를 고르는 것은 아니다. 이전 이미지의 정답 영역을 고정하는 mask도 없다.
- **History:** 현재 VQA 결과로 assertion 가중치를 누적 갱신하고 최고 결과를 보관한다. 과거 여러 prompt·평가·이미지를 evaluator에 다시 입력하는 누적 history는 아니다.
- **시드·초기 noise:** 공식 구현 X/X. 매 refinement round마다 **동일 seed로 Generator를 재초기화**해 같은 시작 noise를 사용한다. [외부 loop](https://github.com/1jsingh/Divide-Evaluate-and-Refine/blob/main/t2i_improve/pipeline_evaluate_and_refine.py#L1049-L1067).
- **외부 이미지·추가 학습:** 외부 이미지 없이 텍스트에서 생성하며 추가 학습은 필요 없다. attention variant의 latent 최적화는 모델 학습과 구분한다.
- **비교 포인트:** 평가를 세분화해도 수정 범위는 전역일 수 있다는 사례. **VQA gradient로 이미지를 직접 최적화하는 방법은 아니며**, VQA는 다음 생성에서 강조할 항목을 선택한다.

### Ranni

- **논문·발표:** *Taming Text-to-Image Diffusion for Accurate Instruction Following*. 최초 공개 **2023** → **CVPR 2024**. [arXiv](https://arxiv.org/abs/2311.17002), [CVPR 최종 논문](https://openaccess.thecvf.com/content/CVPR2024/papers/Feng_Ranni_Taming_Text-to-Image_Diffusion_for_Accurate_Instruction_Following_CVPR_2024_paper.pdf).
- **핵심 방법:** LLM이 prompt를 객체·bbox·색상 등의 **semantic panel**로 구조화 → panel 조건으로 이미지 생성 → 사용자 편집 지시에 따라 panel을 갱신하고 이미지를 수정한다. [§3.1–3.2](https://openaccess.thecvf.com/content/CVPR2024/papers/Feng_Ranni_Taming_Text-to-Image_Diffusion_for_Accurate_Instruction_Following_CVPR_2024_paper.pdf).
- **반복·공간 범위:** 사용자 지시에 따른 여러 차례의 국소 편집이다. 생성 오류를 스스로 평가해 다음 수정을 결정하는 자동 외부 loop는 아니므로 완성본 자동 반복 X이다.
- **영역 선택·보존:** **이전/새 panel에서 바뀐 bbox**로 편집 mask를 만든다. 매 denoising step에 mask 안은 새 latent, 밖은 이전 생성의 같은 timestep latent로 합성한다.
- **History:** 현재 panel과 현재 편집 지시를 사용한다. panel에 이전 수정 결과가 반영되어 있어도 모든 과거 이미지·대화를 누적 입력하는 history와는 다르다.
- **시드·초기 noise:** —/—. 연속 편집 사이 seed·초기 noise 정책은 미확인이다. 논문의 특정 색상 비교에서 seed를 고정한 것을 전체 편집 정책으로 확대하지 않았다.
- **외부 이미지·추가 학습:** 핵심 편집 대상은 자체 생성 이미지와 panel이며 외부 실사 입력을 전제로 하지 않는다(X). **panel 조건 diffusion과 text-to-panel LLM을 추가 학습**한다. [§4.1, §4.3](https://openaccess.thecvf.com/content/CVPR2024/papers/Feng_Ranni_Taming_Text-to-Image_Diffusion_for_Accurate_Instruction_Following_CVPR_2024_paper.pdf).
- **비교 포인트:** 수정 전후 차이로 ROI를 얻고 이전 latent를 보존한다는 점이 관련 있다. 차이를 계산하는 대상은 **생성 이미지가 아니라 구조화된 panel**이다.

### DiffEdit

- **논문·발표:** *Diffusion-based Semantic Image Editing with Mask Guidance*. 최초 공개 **2022** → **ICLR 2023**. [arXiv](https://arxiv.org/abs/2210.11427), [ICLR 논문](https://openreview.net/pdf?id=3lge0p5o-M-).
- **핵심 방법:** 같은 입력 이미지의 noisy state에 source text와 target text를 각각 적용 → 두 조건의 **noise prediction 차이**로 mask 추정 → 입력 이미지의 DDIM inversion 경로를 이용해 target text로 편집한다. [§3.2, Fig.2](https://arxiv.org/pdf/2210.11427).
- **반복·공간 범위:** 한 번의 국소 image editing을 위한 sampling 제어. 완성 결과의 오류를 자동 평가해 prompt를 반복 수정하는 방법은 아니다.
- **영역 선택·보존:** 여러 noise sample에서 예측 차이를 평균·정규화·thresholding해 mask를 얻는다. denoising 중 mask 밖은 **원본의 해당 timestep DDIM 경로**로 교체한다. VAE를 사용하는 경우 latent 보존과 정확한 최종 픽셀 일치는 별개이다.
- **History:** source image 한 장과 source/target text 쌍을 사용하며 누적 수정 history는 없다.
- **시드·초기 noise:** —/—. 자동 외부 round 사이를 비교하는 축은 적용되지 않는다. source inversion으로 편집 시작 상태를 만들며, mask 추정용 여러 noise를 생성하는 것은 refinement round 사이 noise 변경과 다르다.
- **외부 이미지·추가 학습:** 편집 원본 이미지가 필요하며 외부 이미지도 입력할 수 있다(O). 기존 diffusion을 사용하며 추가 학습은 필요 없다.
- **비교 포인트:** detector 없이 **조건 변화에 따른 diffusion 응답 차이**에서 ROI를 얻는 대안이다. 비교하는 값은 두 완성 이미지나 두 예측 clean image가 아니라 **두 text 조건의 noise prediction**이다.

### Prompt-to-Prompt

- **논문·발표:** *Image Editing with Cross-Attention Control*. 최초 공개 **2022** → **ICLR 2023**. [arXiv](https://arxiv.org/abs/2208.01626), [ICLR 논문](https://openreview.net/pdf?id=_CDixzkzeyb).
- **핵심 방법:** 원본/수정 prompt의 두 생성 경로를 함께 진행하며 원본의 token별 cross-attention을 수정 경로에 주입한다. 단어 교체·구절 추가·attention 가중치 변경으로 이미지를 편집한다. [§3, Algorithm 1](https://arxiv.org/html/2208.01626v1#S3).
- **반복·공간 범위:** 한 번의 편집을 수행하는 sampling 내부 제어이며 자동 오류 피드백 loop는 없다. 일반 attention control은 국소·전역 편집을 모두 지원한다.
- **영역 선택·보존:** 공식 구현의 **선택적 LocalBlend**를 켜면 지정 단어의 attention으로 mask를 구성하고 해당 영역 중심으로 latent를 합성한다. 기본 attention 주입만으로 명시적 ROI 제한을 주장할 수 없어 표는 △이다. [공식 LocalBlend 설명](https://github.com/google/prompt-to-prompt#attention-control-options).
- **History:** 원본/수정 prompt 쌍과 원본 attention·latent 경로를 참조한다. 여러 과거 완성 이미지·피드백을 누적 입력하는 구조는 아니다.
- **시드·초기 noise:** X/X. Algorithm 1은 원본과 편집 경로에서 **동일 seed·동일 시작 noise**를 사용한다. [Algorithm 1](https://arxiv.org/html/2208.01626v1).
- **외부 이미지·추가 학습:** 기본은 자체 생성 이미지 편집이며 DDIM inversion을 이용한 외부 실사 편집 확장도 있다(△). 원래 방법은 추가 학습이 필요 없다.
- **비교 포인트:** prompt 수정에도 기존 구조를 유지하는 방법과 token에서 ROI를 얻는 방법을 참고할 수 있다. attention reuse가 최종 픽셀 freeze를 의미하지는 않는다.

### Blended Latent Diffusion

- **논문·발표:** *Blended Latent Diffusion*. 최초 공개 **2022** → **ACM TOG 42(4) / SIGGRAPH 2023**. [arXiv](https://arxiv.org/abs/2206.02779), [저자 프로젝트·서지](https://omriavrahami.com/blended-latent-diffusion-page/), [ACM DOI](https://doi.org/10.1145/3592450).
- **핵심 방법:** 원본 이미지·사용자 mask·target text를 입력받는다. 이미지를 latent로 옮기고 noise를 넣은 뒤, denoising마다 수정 영역의 새 latent와 보존 영역의 source latent를 혼합한다. [§4.1](https://arxiv.org/pdf/2206.02779).
- **반복·공간 범위:** 단일 국소 편집의 sampling 내부 반복이다. 여러 후보를 만들거나 편집을 연속 적용할 수 있지만 자동 오류 피드백 loop를 제안하지는 않는다.
- **영역 선택·보존:** 사용자 mask를 사용한다. 얇은 mask는 초기에는 넓히고 후반에 줄여 수정을 가능하게 한다. VAE 복원 손실로 배경이 달라지는 문제에는 **선택적 이미지별 decoder 최적화**를 사용한다. [§4.2–4.3](https://arxiv.org/pdf/2206.02779).
- **History:** 현재 source image·mask·text만 사용하며 누적 이력은 없다.
- **시드·초기 noise:** —/—. 자동 외부 refinement round가 없다. 서로 다른 후보를 생성해 CLIP으로 고르는 기능은 있지만, 후보 다양화 자체를 feedback 기반 noise 변경으로 분류하지 않았다.
- **외부 이미지·추가 학습:** 편집 원본 이미지가 필요하며 외부 이미지도 입력할 수 있다(O). 기본 latent blending은 사전학습 모델을 쓰지만 **선택적 decoder-weight 최적화까지 포함하면 모델 파라미터가 바뀐다**.
- **비교 포인트:** 국소 수정의 실행·경계 연결·보존 문제를 다루는 기반 방법. ROI 자동 검출이나 ROI 타당성 판단은 별도로 필요하다.

### Attend-and-Excite

- **논문·발표:** *Attention-Based Semantic Guidance for Text-to-Image Diffusion Models*. 최초 공개 **2023** → **ACM TOG 42(4) / SIGGRAPH 2023**. [arXiv](https://arxiv.org/abs/2301.13826), [공식 프로젝트](https://yuval-alaluf.github.io/Attend-and-Excite/).
- **핵심 방법:** 현재 noisy latent의 cross-attention에서 가장 약하게 반영된 subject token을 찾고 attention이 커지도록 latent를 gradient로 수정한다. 일부 timestep에서는 기준을 만족할 때까지 이 최적화를 반복한다. [§4, Algorithm 1](https://arxiv.org/html/2301.13826v2#S4).
- **반복·공간 범위:** **한 번의 sampling 내부**에 있는 추가 최적화 loop다. 완성 이미지 사이의 재생성도, ROI만의 복구도 아니다.
- **영역 선택·보존:** spatial attention map을 관찰하지만 수정 범위를 mask로 제한하거나 바깥을 원본으로 고정하지 않으므로 공간 국소 수정 X이다.
- **History:** 현재 timestep의 latent·attention을 사용하며 이전 완성 이미지나 텍스트 비평을 누적하지 않는다.
- **시드·초기 noise:** —/—. 외부 round의 seed/noise 비교는 해당하지 않는다. 처음 뽑은 noise에서 sampling하며 **현재 latent는 반복 최적화**하지만 이를 완성본 간 초기 noise 교체와 혼동하면 안 된다.
- **외부 이미지·추가 학습:** 외부 reference 없이 텍스트에서 생성한다. 모델 가중치를 학습하지 않고 추론 중 latent를 최적화한다.
- **비교 포인트:** sampling 내부 refinement의 대표 사례. **예측 clean image를 decode해 평가하는 것이 아니라 attention을 직접 평가**하므로, 다른 sampling 내부 방법과도 피드백 신호를 구분해야 한다.

## 데모 트랙

### PromptSculptor

- **논문·발표:** *Multi-Agent Based Text-to-Image Prompt Optimization*. 최초 공개 **2025** → **EMNLP 2025 System Demonstrations**, pp.774–786. 본회의 연구 track과 구분한다. [공식 proceedings](https://aclanthology.org/2025.emnlp-demos.59/).
- **핵심 방법:** 의도 추론 → 장면·스타일 구체화 → 생성 결과 평가 → 피드백·prompt 조정을 담당하는 agent를 연결한다. CLIP score가 낮으면 BLIP-2 caption 등을 이용해 prompt를 수정하며 사용자 피드백도 선택적으로 반영한다. [§3, Algorithm 1](https://aclanthology.org/2025.emnlp-demos.59.pdf).
- **반복·공간 범위:** 완성 이미지 평가 후 prompt 수정·전체 이미지 생성을 반복한다. 공간 mask로 오류 영역만 수정하는 방법은 아니다.
- **영역 선택·보존:** ROI 선택·기존 이미지의 영역 보존 기능은 없다. 모호한 의도·비유를 생성 가능한 구체적 표현으로 바꾸는 데 초점이 있다.
- **History:** 현재 이미지·원문 prompt·현재 최적화 prompt가 중심이다. 과거 텍스트를 누적하는 범위는 미확인(—)이며, 여러 과거 이미지를 시각 memory로 사용하는 구조는 없다(X).
- **시드·초기 noise:** —/—. 논문·부록에서 round 사이 정책을 명시하지 않아 prompt 수정만으로 noise 변경을 추정하지 않았다.
- **외부 이미지·추가 학습:** 사용자 텍스트·선택적 피드백을 입력받으며 외부 reference image는 사용하지 않는다. 기존 모델을 조합하는 training-free 시스템이다.
- **비교 포인트:** VideoRepair가 직접 인용한 prompt-level agent refinement. 객체별 국소 복구와 구분하여 의도 해석·prompt 구체화의 선행연구로 비교할 수 있다.

## 우리 방법과 비교할 때 분리할 축

- **피드백의 위치와 수정 결정 주체:** 완성 이미지 평가를 통한 자동 수정(SLD·Idea2Img), 사용자 지시 편집(Ranni), sampling 내부 최적화(Attend-and-Excite)를 구분한다. ‘iterative’라는 명칭만으로 같은 범주로 묶지 않는다.
- **ROI를 얻는 신호:** 검출 layout과 목표의 차이(SLD), 두 text 조건의 noise prediction 차이(DiffEdit), 수정 전후 semantic panel의 차이(Ranni), 사용자 mask(Blended Latent Diffusion), token attention(Prompt-to-Prompt LocalBlend)이다. 우리 설계의 **연속 생성 결과 차이에서 얻은 ROI**와 비교할 수 있다.
- **국소/전체 수정의 선택 기준:** T2I-Copilot은 편집 도구로 처리 가능한지에 따라 전체 재생성으로 전환한다. 우리 설계의 **ROI 선택이 올바른지 검증한 후 region/whole-frame을 결정하는 판단**은 구체적인 판정 입력·기준으로 설명해야 한다.
- **보존 방식과 강도:** 원본 trajectory 주입, attention 재사용, 이전 이미지 조건 입력, 후반 전체 unfreeze를 구분한다. 좋은 부분을 활용하는 것과 최종 픽셀·latent를 정확히 freeze하는 것은 다른 속성이다.
- **수정 변수와 추가 학습:** prompt·가중치만 바꾸는 방법, latent·영역을 직접 수정하는 방법, 학습된 corrector로 수정하는 방법을 나눈다. Ranni·ReflectionFlow의 추가 학습과 Blended Latent Diffusion의 선택적 decoder 최적화는 비용·적용 가능성 비교에 필요하다.
