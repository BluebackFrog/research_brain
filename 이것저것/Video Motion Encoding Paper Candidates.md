# Video motion 분리 인코딩 논문 후보

## 결론

- **가장 유력한 논문은 DisMo**다. 현재 저장소에도 [정리 노트](../PPVDM/DisMo/DisMo.md)와 [원문 PDF](../PPVDM/DisMo/DisMo.pdf)가 이미 있다.
- 기억한 내용이 “reference video에서 appearance와 무관한 motion embedding을 뽑아 다른 대상에 전달한다”였다면 거의 확실히 **DisMo**다.
- 반면 “영상 생성 비용을 줄이려고 content frame과 motion latent를 따로 생성한다”였다면 **CMD**, “video tokenizer가 keyframe과 motion token을 나눈다”였다면 **Video-LaVIT**, “video VAE가 structure와 dynamics latent를 나눈다”였다면 **VidTwin**일 가능성이 높다.

## 1. DisMo — 최유력

- raw video에서 timestep별 **motion embedding**을 추출하고, appearance·object identity·pose·viewpoint 같은 static content와 무관한 표현을 학습한다. 추출한 motion은 lightweight adapter를 통해 기존 video generator에 condition으로 넣어 서로 다른 category 사이에서도 motion transfer에 사용한다. [공식 프로젝트](https://compvis.github.io/DisMo/) · [arXiv](https://arxiv.org/abs/2511.23428)
- 학습 시 motion extractor가 augmented video를 인코딩하고, 별도 frame generator는 현재 frame과 motion embedding으로 미래 frame을 재구성한다. 현재 frame이 appearance를 이미 제공하므로 bottleneck을 통과한 embedding은 주로 frame 간 변화, 즉 motion을 설명하도록 유도된다. [공식 프로젝트의 Method](https://compvis.github.io/DisMo/)
- 구별 단서: **open-world motion transfer**, DINOv2 + 3D ViT, appearance-invariant embedding, frozen VDM에 conditional LoRA, V-JEPA보다 강한 zero-shot action classification.

## 2. CMD

- video를 하나의 **content frame**과 저차원 **motion latent**의 조합으로 인코딩한다. content frame은 pretrained image diffusion model을 fine-tuning해 생성하고, motion latent는 별도의 lightweight diffusion model이 생성한다. [ICLR 2024 OpenReview](https://openreview.net/forum?id=dQVtTdsvZH) · [arXiv](https://arxiv.org/abs/2403.14148)
- 목적은 disentanglement 자체보다 고차원 video 전체를 직접 diffusion하는 비용을 줄이는 데 가깝다.
- 구별 단서: **content-frame / motion-latent decomposition**, pretrained image diffusion 재사용, 512×1024·16-frame 영상을 약 3.1초에 생성했다는 효율성 강조.

## 3. Video-LaVIT

- video를 **keyframe**과 **temporal motion**으로 분해한 뒤, 각각을 별도 tokenizer로 소수의 discrete visual/motional token으로 바꾸어 LLM에 입력한다. 생성 시 이 token들을 다시 pixel video로 복원한다. [공식 프로젝트](https://video-lavit.github.io/) · [arXiv](https://arxiv.org/abs/2402.03161)
- 구별 단서: video-language LLM, image/video/text의 unified generative pretraining, 제목의 **decoupled visual-motional tokenization**.

## 4. VidTwin

- video VAE가 영상을 두 latent space로 분리한다. **Structure latent**는 전체 content와 저주파·global movement를, **Dynamics latent**는 세부 묘사와 빠른 local movement를 담당한다. 따라서 단순한 appearance/motion 이분법과는 조금 다르다. [CVPR 2025 원문](https://openaccess.thecvf.com/content/CVPR2025/html/Wang_VidTwin_Video_VAE_with_Decoupled_Structure_and_Dynamics_CVPR_2025_paper.html) · [arXiv](https://arxiv.org/abs/2412.17726)
- 구별 단서: Q-Former로 low-frequency trend를 뽑고 spatial averaging으로 rapid motion을 포착하는 compact video tokenizer, 0.20% compression rate.

## 빠른 판별

| 기억나는 표현 | 해당 논문 |
|---|---|
| reference video의 동작만 뽑아 전혀 다른 객체에 전달 | **DisMo** |
| content frame 1장 + motion latent로 효율적 diffusion | **CMD** |
| keyframe token + motion token을 LLM이 처리 | **Video-LaVIT** |
| structure latent + dynamics latent를 쓰는 Video VAE | **VidTwin** |

