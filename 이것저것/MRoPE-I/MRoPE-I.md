## 기존 MRoPE

MRoPE(Multimodal RoPE)는 기존 1D RoPE의 position id를 멀티모달 입력에 맞게 3차원 좌표로 확장한 방식이다. 각 토큰은 하나의 위치값 대신 다음과 같은 위치 tuple을 갖는다.

```text
m_i = (t_i, h_i, w_i)
```

- `t`: temporal axis. 텍스트 순서 또는 비디오 프레임 방향
- `h`: height axis. 이미지/비디오 패치의 세로 위치
- `w`: width axis. 이미지/비디오 패치의 가로 위치

텍스트 토큰은 보통 `(p, p, p)`처럼 세 축에 같은 위치값을 넣어 기존 1D RoPE와 호환되게 만든다. 이미지나 비디오 토큰은 패치의 시공간 위치에 따라 `(t, h, w)`를 부여한다. 단일 이미지라면 모든 패치의 `t`는 같고, `h`, `w`만 달라진다.

MRoPE는 attention의 query/key feature를 세 구간으로 나눈 뒤, 각 구간에 서로 다른 축의 위치값을 사용해 RoPE 회전을 적용한다.

```text
query/key feature
= [temporal chunk | height chunk | width chunk]
```

```text
temporal chunk -> t 위치값으로 회전
height chunk   -> h 위치값으로 회전
width chunk    -> w 위치값으로 회전
```

따라서 feature가 처음부터 `t/h/w` 의미를 갖는 것은 아니지만, 각 feature 구간에 다른 위치 변환을 강제로 적용하기 때문에 attention 계산에서 축별 상대 위치가 반영된다. 예를 들어 두 이미지 패치가 좌우로만 다르면 `w` chunk에서만 위치 차이가 생기고, 상하로만 다르면 `h` chunk에서만 위치 차이가 생긴다.

기존 MRoPE의 핵심 한계는 feature dimension을 연속된 chunk로 나눈다는 점이다.

```text
기존 MRoPE: [T T T T | H H H H | W W W W]
```

RoPE의 frequency는 channel index에 따라 달라지므로, 연속 chunk로 나누면 각 축이 서로 다른 frequency range를 받게 된다. 이로 인해 temporal 축은 long-range 관계에 불리해질 수 있고, `h/w` 축도 서로 다른 decay 특성을 가져 spatial 관계가 비대칭적으로 처리될 수 있다.

이 문제의식이 MRoPE-I의 출발점이다. MRoPE-I는 chunk를 연속으로 나누지 않고 interleave해서 각 축이 더 다양한 frequency를 받도록 만든다.

## MRoPE-I
거의 비슷한데 그냥 t, h, w 의 frequency 를 잘 분배.
```text
MRoPE-I
[T H W | T H W | T H W | T H W]
```

