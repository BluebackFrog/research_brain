# DeepStack: Deeply Stacking Visual Tokens

## 한 줄 요약

DeepStack은 LMM에서 visual token을 LLM 입력 앞에 길게 붙이는 대신, 여러 transformer layer에 나눠 residual로 주입해 더 많은 고해상도 시각 정보를 효율적으로 쓰는 방법이다.

## 문제의식

- 기존 LMM은 이미지에서 뽑은 visual token을 1D sequence로 펼쳐 LLM의 첫 layer에 prefix처럼 넣는다.
- 고해상도 이미지, 문서, OCR, 비디오처럼 세밀한 정보가 필요한 경우 visual token 수가 급증한다.
- visual token을 그대로 늘리면 context length, attention 연산량, 메모리 비용이 커진다.
- token compression은 비용은 줄이지만 세밀한 시각 정보를 잃을 수 있다.

## 핵심 아이디어

DeepStack은 visual token을 sequence 방향으로 늘리지 않고, layer 방향으로 쌓는다.

```python
H[vis_pos] += X_stack[i]
H = TransformerLayer(H)
```

- `H`: 현재 hidden state
- `vis_pos`: 기존 visual token 위치
- `X_stack[i]`: i번째 고해상도 visual token 그룹

즉, 새로운 token을 sequence에 추가하는 것이 아니라 기존 visual token 위치에 고해상도 정보를 더한다.

## 동작 방식

- 저해상도 이미지에서 global visual token을 뽑아 기존 방식처럼 LLM 입력에 넣는다.
- 고해상도 이미지에서 추가 visual token을 뽑는다.
- 이 고해상도 token들을 여러 그룹으로 나눈다.
- 각 그룹을 LLM의 앞쪽 또는 중간 transformer layer에 residual connection으로 주입한다.
- 이때 각 고해상도 token 그룹은 기존 global visual token과 공간적으로 대응되도록 sampling한다.

## 어텐션이 꼬이지 않는 이유

- sequence length는 그대로다.
- attention mask도 그대로다.
- positional embedding 구조도 크게 바뀌지 않는다.
- text token이 visual prefix를 보는 방식도 그대로다.

차이는 각 layer에 들어가기 전, 기존 visual token hidden state가 고해상도 정보로 점진적으로 보강된다는 점이다.

## 왜 노이즈가 아니라 이점이 되는가

- 기존 visual token의 위치와 대응되는 고해상도 정보를 더하기 때문에 엉뚱한 정보가 끼어드는 구조가 아니다.
- residual addition이므로 기존 global 정보는 유지되고 detail이 추가된다.
- LLM의 앞쪽 layer는 visual token을 처리할 수 있는 능력이 있으며, 논문 ablation에서도 너무 뒤쪽 layer보다 앞쪽 layer에 주입하는 것이 좋았다.
- 공간 대응이 중요하다. 논문에서는 2D spatial sampling이 1D sequential sampling이나 단순 grid 방식보다 성능이 좋았다.

## 효과

- 같은 context length에서 더 많은 visual token 정보를 사용할 수 있다.
- 특히 TextVQA, DocVQA, InfoVQA처럼 고해상도와 세밀한 시각 정보가 중요한 task에서 효과가 크다.
- 논문에서는 LLaVA-1.5 대비 평균 성능 향상과 문서/텍스트 중심 benchmark에서 큰 개선을 보고했다.
- DeepStack-L은 LLM layer에 적용한 방식이고, DeepStack-V는 ViT layer에 유사한 아이디어를 적용한 방식이다.

## 한계

- 현재 방식은 layer별 residual 주입 위치와 개수를 heuristic하게 정한다.
- 논문에서도 gated fusion, layer-wise positional embedding, 주입 layer 선택 방법 등을 future work로 언급한다.

## 핵심 직관

DeepStack은 visual token을 더 많이 "보이게" 만들되, sequence를 길게 만들지는 않는다. 대신 기존 visual token 자리를 layer가 지날수록 더 정교하게 업데이트해서, LMM이 고해상도 정보를 더 싸게 활용하도록 만든다.
