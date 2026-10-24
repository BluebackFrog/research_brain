## 한 줄 정리

- **비디오 생성 모델의 추론 능력을 평가하는 26개 벤치마크를, 무엇을 평가하는지와 누가 최종 점수를 판단하는지를 기준으로 정리했다.** VBVR처럼 코드로 채점하는 경우, GPT·Gemini가 판정하는 경우, 두 방식을 섞는 경우를 구분한다.
- 정리 기준: **2026-10-23**. [비디오 추론 벤치마크 조사](thread://01a16659-7fb8-7632-bf41-6019ce6358a3?hostId=local)의 조사 내용과 공식 논문·저장소를 바탕으로 작성했다. 평가 모델과 구현은 아래 출처에서 확인한 설정을 뜻한다.

## 핵심 아이디어 먼저

- 이 문서의 중심은 **모델이 영상을 생성해서 문제를 푸는 능력**이다. 기존 영상을 보고 텍스트로 답하는 Video QA와는 평가 대상이 다르다.
- 예를 들어 미로 그림과 지시를 입력하고, 출발점에서 도착점까지 이동하는 영상을 생성하게 한다. 마지막에 도착했는지뿐 아니라 중간에 벽을 통과하거나 순간이동하지 않았는지도 평가할 수 있다.
- 이때 코드가 경로 좌표를 추출해 벽과의 충돌을 검사하면 **규칙 기반 평가**, VLM이 영상을 보고 위반 여부를 판단하면 **VLM 평가**이다. “물리 법칙이나 게임 규칙을 평가한다”는 사실만으로 규칙 기반 evaluator가 되는 것은 아니다.
- **최종 결과를 보는지, 중간 과정까지 보는지**는 별도의 비교 축이다. 과정 평가를 하면서 VLM을 사용할 수도 있고, 마지막 프레임만 규칙으로 검사할 수도 있다.

## 분류 기준

| 평가 방식 | 점수를 만드는 방법 | 구분할 점 |
| --- | --- | --- |
| **규칙 기반** | 좌표·기호·격자·경로를 추출하고 정답·제약 조건을 코드로 검사 | OCR·검출·추적에 학습 모델을 사용해도, 최종 판정이 명시적 규칙이면 여기에 해당 |
| **정답 자료와 수치 비교** | 정답 영상·이미지와 마스크 겹침, 픽셀 오차, 특징 유사도 등을 계산 | 기호적 규칙이나 물리 법칙 자체를 검증하는 것과는 다름 |
| **범용 VLM 판정** | GPT·Gemini 등이 영상과 평가 기준을 보고 정답 여부·점수를 판단 | 체크리스트가 자세하거나 점수를 코드로 합산해도 VLM 판정이라는 점은 같음 |
| **학습한 평가 모델** | 사람의 평가 라벨 등으로 학습한 전용 모델이 점수를 예측 | 범용 VLM에 프롬프트만 주는 방식과 구분하되, 둘 다 모델 기반 판정 |
| **시뮬레이터 성과** | 생성 영상을 행동으로 바꿔 환경에서 실행하고 성공률·충돌 등을 측정 | 영상 파일만으로 평가가 끝나지 않음 |
| **혼합형** | 과제 또는 지표에 따라 규칙 검사·수치 비교·VLM 판정을 함께 사용 | 일부 과제가 규칙 기반이어도 전체 벤치마크를 규칙 기반이라고 부르면 안 됨 |

- **표를 읽는 기준:** `평가 대상`은 모델이 영상으로 구현해야 하는 능력, `실제 채점 방식`은 생성 결과를 점수로 바꾸는 절차이다. 사람 평가는 자동 평가와의 일치도 검증 또는 별도 평가로 사용될 수 있다.
- **최초 공개일:** 해당 벤치마크를 제안한 논문의 **arXiv v1 제출일**을 기준으로 적는다. 최신 개정일이나 학회 개최일과 구분하며, 날짜 링크에서 원 논문과 제출 이력을 확인할 수 있다.
- **학회·저널:** 공식 proceedings 또는 저자의 공식 논문·프로젝트에서 accept를 명시한 경우만 기재한다. 본회의·Findings·워크숍·별도 트랙을 구분하며, **“accept 확인 안 됨”은 미채택·거절을 뜻하지 않고 확인일 기준으로 공개 근거를 찾지 못했다는 뜻**이다.
- **입력과 출력:** 생성형 추론은 지시문과 과제 이미지 등을 입력해 풀이·상태 변화 영상을 생성한다. 물리·월드모델 평가는 프롬프트, 초기 이미지 또는 영상의 앞부분을 조건으로 후속 영상을 생성하며, 정확한 입력 형식은 벤치마크·트랙마다 다르다.

## 1. 생성형 추론 — 규칙 기반·혼합형

- **두 태스크 열의 범위:** 물리 현상 중심 태스크를 제외해 정리했다. `rule-based metric`은 여기서 **VLM-free 수치 평가**를 뜻하며, OpenCV·OCR·검출·추적·특징 모델의 결과를 코드로 비교하는 방식도 포함한다. VLM이 숫자·기호를 추출한 뒤 코드가 정답을 비교하는 경로는 VLM 열에 넣었다.
- **두 열은 상호 배타적이지 않다.** 같은 태스크에 VLM-free 점수와 VLM 점수가 병행될 수 있으므로, 해당 metric 또는 평가 경로까지 함께 적었다. 과제 이름에 규칙·추론이 들어간다는 이유로 분류하지 않는다.

- **공개 여부 확인일: 2026-10-24.** 공식 배포처의 실제 파일을 기준으로 확인했다. `부분 공개 확인`은 공개된 코드와 공개를 확인하지 못한 평가 경로를 구분한 표시다. 파일 공개 여부를 뜻하며 전체 파이프라인의 실행 재현까지 검증한 것은 아니다.

| 벤치마크 | 최초 공개일 | 학회·저널 | 벤치마크 데이터 공개 여부 | evaluation 코드 공개 여부 | rule-based metric으로 평가하는 태스크 (VLM-free) | VLM으로 평가하는 태스크 | 실제 채점 방식 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **[VBVR-Bench](https://www.video-reason.com/bench/)** | [2026-02-23](https://arxiv.org/abs/2602.20159) | [ICML 2026 본회의](https://proceedings.mlr.press/v306/wang26ia.html) | **공개.** 입력 이미지·프롬프트·정답 이미지/영상. [데이터](https://huggingface.co/datasets/Video-Reason/VBVR-Bench-Data/tree/main) | **공개.** 100태스크 scorer와 실행 도구. [VBVR-EvalKit](https://github.com/Video-Reason/VBVR-EvalKit/tree/275472ffe5ccf76f3a41fb55379ce170397c0c12) | **비물리 태스크 전부.** 정렬, 개수·영역 판별, 객체 추적, 미로·격자 경로, 도형·패턴 변환 등. 공식 평가 설계는 전체 100태스크에 VLM-free scorer를 사용한다. | **없음.** 평가 과정에서 VLM judge를 사용하지 않는다. | 태스크별 scorer가 색·객체·좌표·관계·상태 변화를 추출하고, 정답 및 제약 조건과 비교해 0–1 점수를 계산한다. [공식 평가 코드](https://github.com/Video-Reason/VBVR-EvalKit) |
| **[VBVR-Pro-Bench](https://github.com/Video-Reason/VBVR-Pro-Bench)** | [2026-08-26](https://arxiv.org/abs/2608.26105) | accept 확인 안 됨 | **공개.** 이미지·영상 버전 데이터. [데이터](https://huggingface.co/datasets/Video-Reason/VBVR-Pro-Bench/tree/main) | **공개.** 영상·단계별 이미지 평가 코드. [코드](https://github.com/Video-Reason/VBVR-Pro-Bench/tree/95265d088437db6f1f9511552fce44bfd425569c) | **비물리 태스크 전부.** 도형·격자·기호·객체·경로 관련 과제의 영상 출력 및 단계별 이미지 출력. 공식 전체 100태스크에 neural judge 없는 평가를 제공한다. | **없음.** 일부 태스크의 EasyOCR 사용은 VLM 판정이 아니다. | OpenCV·NumPy, 객체 매칭, 다중 객체 추적, 조합 알고리즘으로 검증한다. 문자·숫자는 일부 과제에서 EasyOCR로 읽고 코드로 채점한다. 영상과 단계별 이미지 설정을 구분한다. [공식 평가 안내](https://github.com/Video-Reason/VBVR-Pro-Bench#readme) |
| **[VR-Bench](https://arxiv.org/html/2511.15065)** | [2025-11-19](https://arxiv.org/abs/2511.15065) | accept 확인 안 됨 | **공개.** 평가 데이터·메타데이터·기준 모델 출력 영상. [데이터](https://huggingface.co/datasets/amagipeng/VR-Bench/tree/main) | **부분 공개 확인.** 궤적 기반 평가 코드는 공개. 영상 Rule Compliance용 VLM judge 코드는 확인되지 않음. [평가 코드](https://github.com/FoundationAgents/VR-Bench/tree/89b29a9c2b2d42b838d28e2ef37d682ba9571850/evaluation) | **5종 모두의 Path Matching:** Regular Maze, Irregular Maze, Trapfield, 3D Maze, Sokoban. 추적 궤적에서 EM·SR·PR·Step Deviation을 계산한다. | **동일한 5종 모두의 Rule Compliance.** 움직임 연속성, 대상의 시간 일관성, 궤적 타당성, 미로 구조 유지, 대상–미로 상호작용을 VLM이 별도로 판정한다. | 추적기로 복원한 경로를 정답 경로와 비교하는 수치 지표와, VLM의 5개 항목 판정으로 만든 VLM-score를 병행한다. 태스크가 분리된 것이 아니라 **같은 영상에 서로 다른 지표**를 적용한다. [논문 §3.2·§8](https://arxiv.org/html/2511.15065#S3.SS2) |
| **[TiViBench](https://arxiv.org/html/2511.13704)** | [2025-11-17](https://arxiv.org/abs/2511.13704) | [CVPR 2026 본회의](https://openaccess.thecvf.com/content/CVPR2026/html/Chen_TiViBench_Benchmarking_Think-in-Video_Reasoning_for_Video_Generation_CVPR_2026_paper.html) | **미공개(공식 저장소 기준).** 실제 데이터 배포 링크 미제공. [공개 상태](https://github.com/EnVision-Research/TiViBench/tree/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8) | **미공개.** README에 실행 예시는 있으나 평가 파일은 없고, VideoTPO 추론 코드만 공개. [저장소](https://github.com/EnVision-Research/TiViBench/tree/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8) | **OpenCV/OCR:** Sudoku, 수식·숫자 수열, 시각 선택형, 그래픽 정렬, match-3형 게임.<br>**DINO 특징 비교:** 도형 수열, 스케치 완성, 시간 순서, 퍼즐, 대칭·회전, 보드게임 인식.<br>**DINO-X 검출·추적:** 그래프 탐색, 객체 수 세기, odd-one-out, 자유 배치 수식, 미로, 순차 소거. | **행동 계획·도구 사용 등 QA 평가 대상 샘플.** Gemini-2.5-Pro가 중간 행동·인과관계·목표 달성 질문에 답한다. **24태스크 전체의 고정 metric 대응표는 공개 자료에서 확인되지 않는다.** | 앞 열은 논문 Appendix C에서 metric별로 명시한 적용 과제·출력 형식이다. 같은 태스크도 샘플의 답변 형식에 따라 평가기가 달라질 수 있다. DINO는 정답 관심 영역과 특징 유사도를, DINO-X는 검출 위치·개수·궤적을 비교한다. [평가 상세](https://arxiv.org/html/2511.13704#A3) |
| **[V-ReasonBench](https://arxiv.org/html/2511.16668)** | [2025-11-20](https://arxiv.org/abs/2511.16668) | accept 확인 안 됨 | **공개.** 입력 이미지·프롬프트·과제별 GT. [데이터](https://github.com/yangluo7/V-ReasonBench/tree/08b37b337c7545603ab92ecb71d27f3b5ef9ae5c/dataset) · [GT](https://github.com/yangluo7/V-ReasonBench/tree/08b37b337c7545603ab92ecb71d27f3b5ef9ae5c/evaluations) | **공개.** 실행 진입점과 과제별 평가 모듈. [evaluate.py](https://github.com/yangluo7/V-ReasonBench/blob/08b37b337c7545603ab92ecb71d27f3b5ef9ae5c/evaluate.py) · [모듈](https://github.com/yangluo7/V-ReasonBench/tree/08b37b337c7545603ab92ecb71d27f3b5ef9ae5c/vreason_bench) | **Sequence Completion, Analogy Solving, Tic-Tac-Toe, Visual Symmetry, Rule Following.** 정답과 마스크 영역의 픽셀 또는 격자 상태를 비교한다. | **Arithmetic Operation, Code Execution, Sudoku, Shape Fitting, Color Connection.** 앞의 세 과제는 VLM이 숫자·기호를 읽는 단계부터 관여하므로 VLM-free가 아니다. | 마지막 프레임을 평가한다. VLM-free 과제는 마스크·격자 비교 결과에 임계값을 적용한다. 나머지는 Gemini-2.5-Pro의 기호 추출 결과를 정답과 대조하거나, 맞춤·연결 관계를 VLM이 판정한다. [논문 §3.5·§7](https://arxiv.org/html/2511.16668#S3.SS5) |
| **[VideoThinkBench](https://arxiv.org/html/2511.04570)** | [2025-11-06](https://arxiv.org/abs/2511.04570) | [CVPR 2026 본회의](https://openaccess.thecvf.com/content/CVPR2026/html/Tong_Thinking_with_Video_Video_Generation_as_a_Promising_Multimodal_Reasoning_CVPR_2026_paper.html) | **공개.** Text-/Vision-centric 전체 및 mini-test 데이터. [데이터](https://huggingface.co/datasets/OpenMOSS-Team/VideoThinkBench/tree/main) | **공개.** 자동 채점·프레임 추출 코드와 VisionCentric 하위 저장소. Visual Puzzles 최종 판정은 사람. [코드](https://github.com/tongjingqi/Thinking-with-Video/tree/a89c06ae13da07ee85d0ec1ef485f3acafe3d02a) · [시각 평가](https://github.com/betmma/VLMPuzzle/tree/ead5807319b61496b557e22b602ece93793ad18d) | **Eyeballing 21종의 Last Frame / Major Frame**, **Mazes 3종**(square·hexagon·circle), **ARC-AGI-2**. 각각 선택 표시·연결성, 미로 경로, 출력 격자를 코드로 비교한다. | **Text-centric 과제:** GSM8K, MATH-500, AIME24/25, BBH, MMLU, MMLU-Pro, GPQA-diamond, SuperGPQA, MathVista, MathVision, MMBench, MMMU. GPT-4o가 마지막 프레임 또는 음성 전사문을 채점한다. | Eyeballing은 표시 추출·프레임 다수결, 미로는 빨간 경로의 연결성·벽 겹침, ARC는 셀 색상의 정답 일치를 계산한다. **Visual Puzzles 10종은 best frame만 픽셀 차이로 고르고 최종 채점은 사람이 한다.** Eyeballing 음성 경로는 아래 주석 참조. [공식 코드](https://github.com/tongjingqi/Thinking-with-Video) · [논문](https://arxiv.org/html/2511.04570) |
| **[MMGR](https://arxiv.org/html/2512.14691)** | [2025-12-16](https://arxiv.org/abs/2512.14691) | [EMNLP 2026 본회의](https://zefan-cai.github.io/research/mmgr/) | **부분 공개.** 입력·프롬프트·추상 과제 GT는 공개, 탐색 과제의 GT solution 매핑은 미공개. [데이터](https://huggingface.co/datasets/ZefanCai/MMGR/tree/main) · [누락 명세](https://github.com/Zefan-Cai/MMGR/blob/c1847addbc7457830493e1a9f54c40d931c3be54/dataset_manifest.json) | **부분 공개 확인.** 주 평가 코드는 공개. Geo-Align·View-Syn 보조 파이프라인의 실행 코드는 확인되지 않음. [코드](https://github.com/Zefan-Cai/MMGR/tree/c1847addbc7457830493e1a9f54c40d931c3be54) | **주 점수:** Maze(픽셀 경로), Sudoku(OCR·정답·제약 검사).<br>**보조 지표:** Panoramic View Last-Mile Navigation의 Geo-Align(복원 궤적 비교); Top-down View·3D Real-world Navigation·SLAG의 View-Syn(정답 영상·이미지와 수치 비교). | **ARC, Math, 탐색 4종**(Panoramic Last-Mile, Top-down, 3D Real-world, SLAG)의 주 점수. **Maze·Sudoku에도 별도 VLM 진단 평가**가 있다. | Maze·Sudoku는 코드 판정과 Gemini-2.5-Pro 판정을 모두 제공한다. 탐색의 주 점수는 VLM이 결정하며, Geo-Align·View-Syn은 별도 보조 평가이다. 보조 지표 및 논문·문서의 설명 차이는 아래 주석에 구분했다. [공식 평가 설명](https://github.com/Zefan-Cai/MMGR#evaluation-protocol) · [논문 Appendix G.3](https://arxiv.org/html/2512.14691#A7.SS3) |
| **[RISE-Video](https://arxiv.org/html/2602.05986)** | [2026-02-05](https://arxiv.org/abs/2602.05986) | [NeurIPS 2026 Datasets & Benchmarks 트랙](https://neurips.cc/virtual/2026/poster/139505) | **공개.** 입력 이미지·프롬프트·평가 메타데이터. [데이터](https://huggingface.co/datasets/VisionXLab/RISE-Video/tree/main) | **공개.** 추론·시간 일관성·시각 품질 평가 코드. [코드](https://github.com/VisionXLab/RISE-Video/tree/3819a04dbac227691fd92dff9d8de44a0db234e7) · [추론 평가](https://github.com/VisionXLab/RISE-Video/tree/3819a04dbac227691fd92dff9d8de44a0db234e7/reasoning_eval) | **Maze Navigation, Symmetry Generation의 Reasoning Alignment만.** 미로는 색 기반 궤적의 벽 침범·목표 도달, 대칭은 마지막 프레임의 격자 위치와 정답을 비교한다. | **그 외 비물리 태스크의 Reasoning Alignment:** 보드게임·단어 퍼즐, 생활 상식, 크기·색·개수·위치·가림, 사회 규칙, 의도·정체성·절차, 공간·시간 추론 등.<br>**미로·대칭을 포함한 전체 비물리 태스크의 Temporal Consistency·Visual Quality**도 VLM이 평가한다. | Reasoning Alignment·시간 일관성은 GPT-5, 시각 품질은 GPT-5-mini가 담당한다. 미로·대칭의 추론 점수만 코드로 대체하며, 보드게임은 정답 이미지를 제공해도 VLM이 판정한다. 따라서 미로·대칭의 종합 점수 전체가 VLM-free인 것은 아니다. [논문 §3.2](https://arxiv.org/html/2602.05986#S3.SS2) |

### 태스크별 평가를 사용할 때의 주의점

- **VR-Bench 공개 코드:** `evaluation/videomodel_eval`은 생성 영상의 궤적 평가를 제공한다. `evaluation/vlm_eval`은 VLM이 미로를 풀어 행동을 출력하는 **비교 모델 평가**이며, 생성 영상의 Rule Compliance를 판정하는 VLM judge가 아니다. 후자의 별도 구현은 확인한 저장소에서 찾지 못했다. [공식 실행 안내](https://github.com/FoundationAgents/VR-Bench/blob/89b29a9c2b2d42b838d28e2ef37d682ba9571850/README.md#-evaluation-method)
- **TiViBench:** 논문 §3.3은 같은 태스크도 샘플 형식에 따라 평가 방법이 달라진다고 명시한다. 표는 Appendix C에 명시된 적용 과제·형식을 정리한 것이며, 24개 태스크를 임의로 두 집합에 완전히 분할한 목록이 아니다. 확인한 공식 저장소에는 README가 안내하는 `evaluate.py`·`metrics/`와 샘플별 evaluator 매핑이 공개되어 있지 않다. [논문 §3.3](https://arxiv.org/html/2511.13704#S3.SS3) · [확인한 저장소 버전](https://github.com/EnVision-Research/TiViBench/tree/cc5ee077e77821ae6fc4b8e1e085ebb7ce3a38a8)
- **VideoThinkBench:** Visual Puzzles 10종은 사람 판정이므로 두 자동 평가 열 어디에도 포함하지 않았다. Eyeballing의 Audio 점수는 Whisper 전사에서 첫 NATO 선택지 단어를 추출해 정답과 비교하므로 VLM judge는 없지만, 순수 픽셀 기반 시각 평가와 구분해야 한다. 공개 `evaluate()`는 시각 점수와 함께 음성 전사도 호출한다. ARC의 사람 사례 분석 역시 자동 격자 평가와 별도다. [시각·음성 평가 코드](https://github.com/betmma/VLMPuzzle/blob/ead5807319b61496b557e22b602ece93793ad18d/puzzle/point_target_base.py#L691) · [Visual Puzzles 평가](https://arxiv.org/html/2511.04570#A3.SS5)
- **MMGR:** Maze·Sudoku의 주 점수에 관해 본문과 부록의 서술이 서로 다르지만, 현재 공식 README는 수치 대조를 근거로 **deterministic 평가가 주 점수, VLM 평가는 진단용**이라고 명시한다. Geo-Align은 VGGT로 복원한 궤적에 SPL·nDTW·ATE 등의 수치 지표를 적용하고, View-Syn은 FVD/FID·SSIM·PSNR·LPIPS로 정답 자료와 비교하는 **보조 평가**다. 두 보조 파이프라인은 논문에 제시되어 있으나, 확인한 공개 저장소에서 실행 코드는 찾지 못했다. [README의 설명](https://github.com/Zefan-Cai/MMGR/blob/c1847addbc7457830493e1a9f54c40d931c3be54/README.md#deterministic-evaluation) · [Geo-Align·View-Syn](https://arxiv.org/html/2512.14691#A7.SS3)

## 2. 생성형 추론 — VLM·사람 판정

| 벤치마크 | 최초 공개일 | 학회·저널 | 평가 대상 | 평가자 | 실제 채점 방식 |
| --- | --- | --- | --- | --- | --- |
| **[RULER-Bench](https://arxiv.org/html/2512.02622)** | [2025-12-02](https://arxiv.org/abs/2512.02622) | accept 확인 안 됨 | 시각·과학·가설·게임·의미·사회 규칙 | **o3** | 지시 준수·시각 일관성·시각 품질·규칙 일관성의 체크리스트를 good/medium/bad로 판정한다. **제목의 “Rule-based”는 추론 과제의 성격이며 evaluator가 규칙 코드라는 뜻이 아니다.** |
| **[VIPER](https://arxiv.org/html/2512.24952)** | [2025-12-31](https://arxiv.org/abs/2512.24952) | [ACL 2026 본회의 · Long Papers](https://aclanthology.org/2026.acl-long.934/) | 시간·구조·기호·공간·물리·계획 추론, 16태스크 | **GPT-5** | 샘플링한 프레임을 보고 최종 결과의 정확성과 과정의 제약 준수를 각각 판단한다. 기본 **POC@1.0은 결과와 과정이 모두 맞아야 성공**이다. [공식 코드](https://github.com/RUCAIBox/VIPER) |
| **[MME-CoF](https://arxiv.org/html/2510.26802)** | [2025-10-30](https://arxiv.org/abs/2510.26802) | [CVPR 2026 Findings](https://openaccess.thecvf.com/content/CVPR2026F/html/Guo_Are_Video_Models_Ready_as_Zero-Shot_Reasoners_An_Empirical_Study_CVPRF_2026_paper.html) | 여러 프레임을 연결한 공간·기하·물리 등 추론, 12종 | **Gemini-2.5-Pro** | 지시 정합성·시간 일관성·시각 안정성·내용 충실도·관련성을 각각 0–4점으로 평가한다. 필수 추론 단계별 성공률을 직접 계산하는 지표는 아니다. |
| **[MME-CoF-Pro](https://arxiv.org/html/2603.20194)** | [2026-03-20](https://arxiv.org/abs/2603.20194) | [ECCV 2026 본회의](https://link.springer.com/chapter/10.1007/978-3-032-37467-7_5) | 추론 과정의 일관성, 텍스트·시각 힌트 활용, 16범주 | **Gemini-2.5-Flash** | 매 10번째 프레임을 보고, 사람이 검증한 필수 중간 단계가 각각 성공했는지 판단한다. **성공한 단계 수 / 전체 필수 단계 수**를 Reasoning Score로 계산하고 생성 품질도 별도로 평가한다. |
| **[VMEvalKit](https://arxiv.org/html/2512.05969)** | [2025-11-02](https://arxiv.org/abs/2512.05969v1) | accept 확인 안 됨 | 체스·미로·Sudoku·머릿속 회전·도형 규칙 추론 | **GPT-4o + 사람** | 입력·정답 목표 이미지·생성 결과를 비교해 correctness를 1–5점으로 평가하며 4–5점을 성공으로 본다. 주로 마지막 상태를 평가하고 사람이 같은 척도로 검증하는 소규모 평가 suite이다. |
| **[Gen-ViRe](https://arxiv.org/html/2511.13853)** | [2025-11-17](https://arxiv.org/abs/2511.13853) | accept 확인 안 됨 | 지각·유추·추상·계획·시공간·알고리즘 추론, 24태스크 | **Gemini-2.5-Pro** | 정적 과제는 최종 이미지, 동적 과제는 영상 과정을 태스크별 기준으로 평가한다. 항목 점수를 0–1로 정규화하고 과제 단위 평균을 집계한다. “Hybrid”라는 표현이 별도 규칙 기반 scorer를 뜻하지는 않는다. [공식 코드](https://github.com/L-CodingSpace/GVR) |
| **[VWG-Bench](https://arxiv.org/html/2609.11242)** | [2026-09-10](https://arxiv.org/abs/2609.11242) | [ECCV 2026 본회의](https://link.springer.com/chapter/10.1007/978-3-032-37595-7_30) | 9개 추론 차원·38태스크 | **Gemini-3.0-Pro** | 영상 품질·과정 일관성·암묵적 규칙 준수·중간 목표·최종 목표를 각각 1–5점으로 평가한다. 규칙 목록은 judge에게 제공하는 판단 기준이다. |
| **[VGI-Bench](https://arxiv.org/html/2608.19583v3)** | [2026-08-20](https://arxiv.org/abs/2608.19583) | [EMNLP 2026 본회의](https://hexuan21.github.io/VGI-Bench/) | 시각 조직화·시공간 변화·퍼즐·물체 조작, 27태스크 | **Gemini-3-Flash** | **목표 달성도(Completeness) × 과정 규칙 준수 점수(Rubric Score)**로 평가한다. 짧은 시간 구간별로 관찰하고, 위반이 의심되는 구간은 프레임을 더 촘촘히 확인한다. |
| **[IntelligentVBench](https://github.com/Tencent-Hunyuan/OmniWeaving/blob/main/readme.md)** | [2026-03-25](https://arxiv.org/abs/2603.24458v1) | accept 확인 안 됨 | 암묵적 I2V·시작/끝 프레임 연결·참조 기반 편집·다중 객체 조합 | **Gemini-2.5-Pro** | 태스크별 3개 차원을 1–5점으로 평가하고 평균과 최솟값을 보고한다. 퍼즐 풀이뿐 아니라 생성·편집의 추론과 조합 능력도 포함한다. |

## 3. 물리 추론·월드모델·조합적 생성

- 아래 항목은 VBVR 같은 퍼즐·추상 추론과 평가 범위가 다르지만, **물리적으로 올바른 상태 변화와 지시 준수**를 다룬다는 점에서 PPVDM의 인접 비교 대상이다.

| 벤치마크 | 최초 공개일 | 학회·저널 | 평가 대상 | 평가 방식 | 실제 채점 방식 |
| --- | --- | --- | --- | --- | --- |
| **[Physics-IQ](https://arxiv.org/html/2501.09038v1) / [Verified](https://arxiv.org/html/2606.18943v1)** | 원본 [2025-01-14](https://arxiv.org/abs/2501.09038)<br>Verified [2026-06-17](https://arxiv.org/abs/2606.18943) | 원본 [WACV 2026 본회의](https://openaccess.thecvf.com/content/WACV2026/html/Motamed_Do_Generative_Video_Models_Understand_Physical_Principles_WACV_2026_paper.html)<br>Verified: accept 확인 안 됨 | 실제 물리 실험의 후속 움직임 예측 | **정답 영상과 수치 비교, VLM 없음** | 배경 차분·임계값·형태학적 처리로 움직임 마스크를 추출하고, 실제 촬영 정답 영상과 3종 IoU·RGB MSE를 비교한다. Verified는 정답 마스크·프롬프트·집계 등을 개선한다. **물리 법칙을 직접 검사하는 것이 아니라 실제 움직임과 비교한다.** [공식 코드](https://github.com/google-deepmind/physics-IQ-benchmark) |
| **[PhyGenBench / PhyGenEval](https://github.com/OpenGVLab/PhyGenBench)** | [2024-10-07](https://arxiv.org/abs/2410.05363) | [ICML 2025 본회의](https://proceedings.mlr.press/v267/meng25c.html) | 160프롬프트·27물리 법칙 | **VQA/VLM 다단계 평가** | 핵심 현상은 VQAScore, 사건 순서는 GPT-4o 또는 LLaVA-Interleave, 전체 자연스러움은 GPT-4o 또는 InternVideo2 등으로 판정한다. 정답 후속 영상 대신 질문·평가 기준을 사용한다. |
| **[VideoPhy](https://github.com/Hritikbansal/videophy)** | [2024-06-05](https://arxiv.org/abs/2406.03520) | [ICLR 2025 본회의](https://proceedings.iclr.cc/paper_files/paper/2025/hash/fce2d8a485746f76aac7b5650db2679d-Abstract-Conference.html) | 고체·유체 상호작용의 물리 상식 | **사람 + 학습한 VLM** | 프롬프트와 의미가 맞는지(SA), 물리 상식에 맞는지(PC)를 평가한다. 자동 evaluator는 사람 라벨로 학습한 **VideoCon-Physics**이다. 사람 평가와 자동 평가의 leaderboard·집계 방식을 구분해야 한다. |
| **[VideoPhy-2](https://github.com/Hritikbansal/videophy/tree/main/VIDEOPHY2)** | [2025-03-09](https://arxiv.org/abs/2503.06800) | [ICLR 2026 본회의](https://proceedings.iclr.cc/paper_files/paper/2026/hash/c02f6a1d5c55e16db50d339dad905b4d-Abstract-Conference.html) | 스포츠·인체 동작·물체 조작 | **사람 + 학습한 VLM** | **VideoPhy-2-AutoEval**이 SA·PC를 1–5점으로 평가하고 개별 물리 규칙의 준수·위반·판단 불가도 분류한다. Joint 성공률은 **SA≥4와 PC≥4를 동시에 만족하는 비율**이다. |
| **[WorldModelBench](https://arxiv.org/html/2502.20694v1)** | [2025-02-28](https://arxiv.org/abs/2502.20694) | [NeurIPS 2025 Datasets & Benchmarks 트랙](https://papers.nips.cc/paper_files/paper/2025/hash/4ec03ed08a3fcb59e1c815b5598beff1-Abstract-Datasets_and_Benchmarks_Track.html) | 로봇·운전·인간 활동 등 월드모델 능력 | **학습한 VLM** | VILA 계열 평가기가 지시 준수, 5개 물리 항목, 2개 상식·품질 항목을 평가해 0–10점으로 집계한다. [공식 코드](https://github.com/WorldModelBench-Team/WorldModelBench) |
| **[PhyWorldBench](https://arxiv.org/html/2507.13428)** | [2025-07-17](https://arxiv.org/abs/2507.13428) | [ICLR 2026 본회의 · Oral](https://proceedings.iclr.cc/paper_files/paper/2026/hash/79d86433c2acd12b6fa98553435d226e-Abstract-Conference.html) | 기본·복합 물리와 의도적인 비물리적 현상(Anti-Physics) | **사람 + 범용 VLM** | 주 실험에서는 **o1**이 8프레임을 보고 영상을 설명한 뒤, 시나리오별 기본·핵심 기준을 Yes/No로 판정한다. 명시적 기준 문장이 있어도 코드가 물리 규칙을 직접 검사하는 방식은 아니다. |
| **[PhyGround / PhyJudge](https://github.com/NU-World-Model-Embodied-AI/PhyGround)** | [2026-05-11](https://arxiv.org/abs/2605.10806) | accept 확인 안 됨 | 250개 텍스트·이미지 조건 생성 프롬프트, 13물리 법칙 | **학습한 VLM** | Qwen3.5-9B+LoRA 기반 **PhyJudge-9B**가 의미 일치·물리적 시간 순서·객체 지속성·해당 물리 법칙을 1–5점으로 평가한다. 원문 프롬프트와 법칙별 기준·관찰 질문을 함께 사용한다. |
| **[WorldSimBench](https://arxiv.org/html/2410.18072)** | [2024-10-23](https://arxiv.org/abs/2410.18072) | [ICML 2025 본회의](https://proceedings.mlr.press/v267/qin25f.html) | Minecraft·자율주행·로봇 조작 | **학습한 VLM + 시뮬레이터 성과** | 지각 평가 트랙은 Flash-VStream 기반 **Human Preference Evaluator**를 사용한다. 행동 평가 트랙은 생성 영상을 행동으로 변환해 MineRL·CARLA·CALVIN에서 실행하고 과제 성과를 측정하므로, 영상 파일만으로 평가가 끝나지 않는다. |
| **[T2V-CompBench](https://arxiv.org/html/2407.14505)** | [2024-07-19](https://arxiv.org/abs/2407.14505) | [CVPR 2025 본회의](https://openaccess.thecvf.com/content/CVPR2025/html/Sun_T2V-CompBench_A_Comprehensive_Benchmark_for_Compositional_Text-to-video_Generation_CVPR_2025_paper.html) | 속성·행동·공간관계·움직임·개수의 조합 | **VLM + 인식 결과의 규칙 검사** | 속성·행동·상호작용은 LLaVA 계열이 판정한다. 공간관계·개수는 검출·분할·깊이 모델의 출력에 규칙을 적용하고, 움직임은 객체 추적 궤적의 상대 이동으로 평가한다. [공식 코드](https://github.com/KaiyueSun98/T2V-CompBench/blob/V2/README.md) |

## 최종 결과와 중간 과정의 차이

- **최종 상태 중심:** V-ReasonBench, VMEvalKit, VideoThinkBench의 미로 등은 주로 마지막 프레임에서 정답을 확인한다.

	- 마지막 프레임에 올바른 답이나 경로가 있어도, 앞선 프레임의 잘못된 상태 전환까지 모두 검증했다는 뜻은 아니다.
- **결과와 과정의 동시 성공:** VIPER는 결과가 맞는지와 풀이 과정이 제약을 지켰는지를 구분하고, 기본 지표에서 둘 다 성공해야 정답으로 처리한다.
- **중간 단계별 진단:** MME-CoF-Pro는 필수 추론 단계 중 몇 개를 성공했는지 평가하므로, 과정의 어느 부분이 실패했는지 분석하기 좋다.
- **목표 달성도에 과정 점수 반영:** VGI-Bench는 목표 달성도와 과정 규칙 준수 점수를 곱한다. 최종 목표에 도달해도 과정이 잘못되면 점수가 낮아진다.
- **궤적 수치와 과정 판정의 병행:** VR-Bench는 경로 지표와 VLM 규칙 준수 평가를 함께 제공한다. 궤적 지표만 사용한 결과와 전체 평가 결과를 구분해야 한다.

## PPVDM에서의 선택 기준

- **VBVR와 가까운 규칙 기반 벤치마크를 추가:** `VBVR-Pro-Bench`가 가장 직접적인 후보이다. 영상 출력과 단계별 이미지 출력 설정을 구분해 사용할 수 있다.
- **VLM judge 없이 일부 추론 능력만 비교:** `TiViBench`, `V-ReasonBench`, `VR-Bench`, `MMGR`의 규칙 기반·수치 비교 지표 또는 해당 과제만 선택한다.

	- 이 경우 **선택한 과제와 지표를 명시**하고, 전체 벤치마크의 종합 점수처럼 보고하지 않는다.
- **실제 물리 영상에서 모델 판정을 피하기:** `Physics-IQ / Verified`가 별도의 비교 축이다. 정답 후속 영상과 움직임 마스크가 필요하며, 점수는 물리 법칙의 직접 검증이 아니라 정답 움직임과의 일치도를 뜻한다.
- **최종 결과는 맞지만 과정은 틀린 영상 검출:** `VIPER`, `MME-CoF-Pro`, `VGI-Bench`가 직접적인 후보이다. 세 벤치마크 모두 실제 판정은 VLM이 수행한다.
- **현실적인 장면의 물리 타당성 평가:** `PhyGenBench`, `VideoPhy-2`, `PhyGround / PhyJudge`, `WorldModelBench`를 비교할 수 있다. 법칙별 피드백이 필요한지, 의미 일치와 물리성을 분리할지에 따라 선택한다.
- 위 선택은 **평가 목적에 따른 적합성 판단**이다. 서로 다른 데이터·척도의 점수를 하나의 성능 순위로 합치지는 않는다.

## 해석할 때 주의할 점

- **후속 논문에 원본의 채택 이력을 옮겨 쓰지 않는다.** VBVR-Pro-Bench는 VBVR-Bench와 별도 논문이고, Physics-IQ Verified도 원본과 별도 논문이다. VMEvalKit의 옛 저장소가 현재 VBVR-EvalKit로 이동하더라도 그곳의 ICML 2026 표기는 VBVR 논문의 이력이다.
- **논문 공개와 데이터 공개를 구분한다.** IntelligentVBench를 소개한 OmniWeaving 논문은 2026-03-25에 처음 제출됐고 데이터는 2026-04-03에 공개됐다. 표에는 논문 날짜를 적었다. [공식 저장소](https://github.com/Tencent-Hunyuan/OmniWeaving)
- **VMEvalKit의 날짜는 arXiv 표시를 따른다.** 식별자는 `2512.05969`이지만 공식 페이지의 제출 이력은 2025-11-02로 표시되어 있어, 식별자의 월을 추정해 쓰지 않았다. [arXiv v1](https://arxiv.org/abs/2512.05969v1)
- **규칙을 평가하는 것과 규칙으로 채점하는 것은 다르다.** RULER-Bench의 rule coherence, PhyWorldBench의 물리 기준, VWG-Bench의 규칙 목록은 VLM에게 주는 판단 기준이다.
- **평가 기준을 만드는 모델과 실제 judge를 구분한다.**

	- RULER-Bench: GPT-5는 체크리스트 작성에 사용되고, 실제 채점은 o3가 수행한다.
	- MME-CoF-Pro: Gemini-2.5-Pro는 평가 기준 초안 작성용이고, 실제 judge는 Gemini-2.5-Flash이다.
	- TiViBench: VideoTPO의 GPT-4o optimizer와 벤치마크의 Gemini-2.5-Pro QA evaluator는 역할이 다르다.
- **MMGR의 주 점수 설명에는 본문·부록 간 차이가 있다.** 확인한 HTML의 §4.2·Table 4 설명은 미로·Sudoku의 주 점수를 결정적 검사로 설명하지만, Appendix C.5.1·D.4.1에는 VLM 주 점수와 결정적 부가 진단으로 설명하는 문장이 남아 있다. 따라서 여기서는 **두 평가 방식을 제공하는 혼합형**으로 분류하며, 실제 실험에서는 사용할 버전과 코드의 집계 경로를 추가 확인해야 한다. [논문](https://arxiv.org/html/2512.14691)
- **Gen-ViRe의 “Hybrid”는 규칙 기반 scorer의 존재를 뜻하지 않는다.** 사람과 VLM을 통한 평가 기준 작성 및 정적·동적 과제의 평가 분기를 가리킨다.
- **이름이 비슷한 대상을 구분한다.** 여기의 VGI-Bench는 *Probing Visual Intelligence*이며, 동명의 Seldon 영상 이해 QA 벤치마크가 아니다. T2V-CompBench 저장소의 `V2`는 브랜치 이름이므로 별도 `T2V-CompBench++ / 2.0`으로 세지 않았다.
- **범위 밖:** VBench·EvalCrafter 같은 일반 영상 품질 평가, Video-MME·MVBench·VSI-Bench 같은 영상 이해 QA, WISA 같은 생성 방법·학습 데이터는 위 26개 항목과 구분한다. VMEvalKit도 대형 독립 데이터셋이 아니라 논문에 딸린 소규모 평가 suite이다.

## 용어 메모

- **VLM judge:** 이미지·영상을 읽는 언어 모델이 정답 여부나 점수를 직접 판정하는 방식.
- **GT / reference:** 비교 기준으로 주어진 정답 영상·이미지·기호·경로. 생성기의 입력 이미지와 평가용 정답 자료는 역할이 다르다.
- **OCR:** 이미지 속 문자·숫자를 읽는 과정. 읽은 결과를 코드로 검사할 수도 있고 VLM이 판정할 수도 있다.
- **Trajectory:** 시간에 따른 객체의 위치를 연결한 이동 경로.
- **IoU / MSE:** IoU는 두 영역이 얼마나 겹치는지, MSE는 대응하는 값·픽셀이 얼마나 다른지를 수치로 나타낸다.
- **Rubric:** 무엇을 만족하면 몇 점을 줄지 정한 평가 기준. rubric이 있다고 해서 규칙 기반 evaluator인 것은 아니다.
- **SA / PC:** 각각 프롬프트의 의미를 따르는 정도(semantic adherence)와 물리 상식에 맞는 정도(physical commonsense).
