# PART 6. CSS Position

## 1. Position을 왜 배우는가?

PART 5에서는 브라우저의 기본 Layout인 **Normal Flow**를 배웠습니다.

```text
Normal Flow

Block Box
    ↓
Block Box
    ↓
Block Box
```

Inline Content는 Line Box 안에서 흐릅니다.

```text
Line Box

Text → span → a → Text
```

Normal Flow만으로도 일반적인 문서 구조를 만들 수 있습니다.

하지만 실제 UI에서는 다음과 같은 요구가 생깁니다.

```text
카드 오른쪽 위에 배지 배치

이미지 위에 버튼 배치

화면 오른쪽 아래에 채팅 버튼 고정

스크롤해도 Header 유지

특정 Box를 기준으로 자식 Box 이동
```

이러한 **Box의 위치 제어**에 사용하는 핵심 Property가 `position`입니다.

---

# 2. `position`이란?

`position`은 Element가 생성하는 Box의 **위치를 어떤 방식으로 결정할 것인가**를 지정하는 CSS Property입니다.

대표적인 값은 다음과 같습니다.

```css
position: static;
position: relative;
position: absolute;
position: fixed;
position: sticky;
```

전체 구조부터 보면:

```text
position
│
├── static
│     기본 위치 방식
│
├── relative
│     Normal Flow의 원래 위치를 기준으로 이동
│
├── absolute
│     Positioning 기준이 되는 Containing Block을 기준으로 배치
│
├── fixed
│     일반적으로 Viewport를 기준으로 고정
│
└── sticky
      일정 지점까지 Normal Flow
      → 임계점 이후 붙어서 이동
```

---

# 3. Position과 `top`, `right`, `bottom`, `left`

`position`과 함께 다음 Property를 자주 사용합니다.

```css
top
right
bottom
left
```

예:

```css
.box {
  position: relative;

  top: 20px;
  left: 30px;
}
```

또는 현대 CSS에서는 논리적 방향 Property도 사용할 수 있습니다.

```css
inset-block-start
inset-inline-start
```

그리고 네 방향을 한 번에 지정하는 Shorthand도 있습니다.

```css
inset: 10px;
```

입문 단계에서는 우선:

```text
top
right
bottom
left
```

를 중심으로 이해하겠습니다.

중요한 점:

> `top`, `right`, `bottom`, `left`의 정확한 동작은 `position` 값에 따라 달라집니다.

---

# 4. `position: static`

`static`은 일반적인 Element의 디폴트 Position 방식입니다.

```css
.box {
  position: static;
}
```

Box는 Normal Flow에 따라 배치됩니다.

```html
<div>A</div>
<div>B</div>
<div>C</div>
```

일반적인 Block Layout:

```text
┌──────────────────┐
│ A                │
└──────────────────┘

┌──────────────────┐
│ B                │
└──────────────────┘

┌──────────────────┐
│ C                │
└──────────────────┘
```

즉:

```text
HTML 순서
   ↓
Normal Flow
   ↓
A
B
C
```

입니다.

---

# 5. `static`에서는 Offset이 적용되지 않는다

다음 CSS를 보겠습니다.

```css
.box {
  position: static;

  top: 100px;
  left: 100px;
}
```

일반적인 `static` Box에는 이러한 inset Property가 위치 이동 효과를 주지 않습니다.

```text
position: static

top
left
right
bottom

   ↓

위치 조정에 사용되지 않음
```

따라서 `top`, `left` 등으로 위치를 조절하려면 다른 Positioning 방식을 사용해야 합니다.

---

# 6. `position: relative`

`relative`는 매우 중요한 Position 방식입니다.

```css
.box {
  position: relative;
}
```

Offset을 지정하지 않으면 Box는 원래 Normal Flow 위치에 있습니다.

```text
Normal Flow

A
↓
B
↓
C
```

하지만:

```css
.box-b {
  position: relative;

  top: 20px;
  left: 30px;
}
```

처럼 Offset을 지정하면 **자신의 원래 위치를 기준으로 시각적으로 이동**합니다.

---

# 7. Relative의 핵심

원래 위치:

```text
┌────────────┐
│ A          │
└────────────┘

┌────────────┐ ← B의 원래 위치
│ B          │
└────────────┘

┌────────────┐
│ C          │
└────────────┘
```

`B`에:

```css
position: relative;
top: 20px;
left: 30px;
```

를 적용하면 개념적으로:

```text
B 원래 위치
┌────────────┐
│            │
└────────────┘
        ↘
         30px →
         ↓ 20px

         ┌────────────┐
         │ B          │
         └────────────┘
```

처럼 이동합니다.

---

# 8. Relative는 원래 공간을 유지한다

이 부분이 매우 중요합니다.

`relative`로 Box를 이동시켜도 **Normal Flow에서 원래 Box가 차지하던 공간은 유지됩니다.**

```text
Normal Flow 계산

A
↓
[B의 원래 공간]
↓
C
```

B는 시각적으로 이동합니다.

```text
A

[B 원래 공간]

        B ← 실제 표시 위치

C
```

따라서 C가 B의 원래 자리로 올라오지 않습니다.

핵심:

```text
relative

Normal Flow 참여
        O

원래 공간 유지
        O

시각적 위치 이동
        O
```

---

# 9. `relative`의 두 가지 주요 용도

### 용도 1 — 자신의 위치를 조금 조정

```css
.icon {
  position: relative;
  top: 2px;
}
```

### 용도 2 — `absolute` 자식의 기준 Box 제공

실무에서는 두 번째 용도가 매우 중요합니다.

```css
.card {
  position: relative;
}

.badge {
  position: absolute;
}
```

이 관계는 `absolute`를 배우면 명확해집니다.

---

# 10. `position: absolute`

`absolute`는 Normal Flow와 매우 다른 방식으로 동작합니다.

```css
.box {
  position: absolute;
}
```

핵심 특징:

```text
absolute

Normal Flow에서
일반적인 공간을 차지하지 않음

        +

Positioning 기준이 되는
Containing Block을 기준으로

top / right / bottom / left
등으로 위치 결정
```

여기서 가장 중요한 개념이 등장합니다.

> **Containing Block**

---

# 11. Absolute는 Normal Flow에서 빠진다

HTML:

```html
<div class="a">A</div>
<div class="b">B</div>
<div class="c">C</div>
```

처음에는:

```text
A
↓
B
↓
C
```

입니다.

B에:

```css
.b {
  position: absolute;
}
```

를 적용하면 B는 일반적인 Normal Flow 공간을 차지하지 않습니다.

따라서 Normal Flow 관점에서는:

```text
A
↓
C
```

처럼 됩니다.

B는 별도의 Positioning 규칙에 따라 배치됩니다.

```text
          B

A
↓
C
```

이것이 `relative`와 `absolute`의 가장 중요한 차이 중 하나입니다.

---

# 12. Relative vs Absolute

비교하면:

```text
relative

Normal Flow
A
↓
[B 공간 유지]
↓
C

B만 시각적으로 이동
```

반면:

```text
absolute

Normal Flow
A
↓
C

B는 별도의 위치 계산
```

정리:

| 특징             |  relative |         absolute |
| -------------- | --------: | ---------------: |
| Normal Flow 참여 |         O |                X |
| 원래 공간 유지       |         O |                X |
| Offset 사용      |         O |                O |
| Positioning 기준 | 자신의 원래 위치 | Containing Block |

---

# 13. Absolute는 무엇을 기준으로 움직이는가?

초보자가 흔히 다음처럼 외웁니다.

> "`absolute`는 부모를 기준으로 움직인다."

이 설명은 정확하지 않습니다.

더 정확한 설명은:

> **`absolute` Box는 자신의 Containing Block을 기준으로 위치가 계산된다.**

입니다.

예:

```css
.badge {
  position: absolute;

  top: 10px;
  right: 10px;
}
```

여기서:

```text
top: 10px
right: 10px
```

은 **Containing Block**을 기준으로 계산됩니다.

---

# 14. Containing Block이란?

Containing Block은 Percentage 크기나 Positioned Element의 Offset 등을 계산할 때 기준이 되는 사각형 영역입니다.

Position을 공부할 때는 특히:

> **Absolute Box의 위치 계산 기준**

으로 이해하는 것이 중요합니다.

예:

```text
Containing Block
┌────────────────────────────┐
│                    10px    │
│                       ↓    │
│                  ┌──────┐  │
│                  │Badge │←10px
│                  └──────┘  │
│                            │
└────────────────────────────┘
```

---

# 15. Absolute의 기준을 만드는 대표적인 패턴

HTML:

```html
<div class="card">
  <img src="product.jpg" alt="상품">

  <span class="badge">
    SALE
  </span>
</div>
```

CSS:

```css
.card {
  position: relative;
}

.badge {
  position: absolute;

  top: 10px;
  right: 10px;
}
```

구조:

```text
.card
position: relative

┌─────────────────────────────┐
│                      SALE   │
│                             │
│                             │
│        Product Image        │
│                             │
│                             │
└─────────────────────────────┘
```

여기서 `.card`가 `.badge`의 Positioning 기준이 되는 Containing Block을 형성하는 대표적인 상황입니다.

---

# 16. 왜 부모에 `position: relative`를 넣는가?

다음 패턴을 매우 자주 볼 수 있습니다.

```css
.parent {
  position: relative;
}

.child {
  position: absolute;

  top: 0;
  right: 0;
}
```

많은 초보자가 이렇게 외웁니다.

```text
absolute를 쓰면
부모에 relative를 넣는다.
```

하지만 이유를 이해해야 합니다.

```text
Parent
position: relative
       ↓
Absolute Child가
위치를 계산할 기준 제공
       ↓
Child
position: absolute
top/right 계산
```

즉 부모를 이동시키려고 `relative`를 넣은 것이 아닙니다.

> **Absolute Child의 위치 기준을 의도한 조상으로 만들기 위한 대표적인 패턴입니다.**

---

# 17. 기준 조상이 없다면?

Absolute Positioned Element는 Positioning 기준을 찾습니다.

단순화하면 조상 방향으로 올라가면서 자신에게 해당하는 Containing Block을 형성하는 조상을 찾습니다.

```text
absolute child
      ↑
parent
position: static
      ↑
grandparent
position: relative
      ↑
기준 발견
```

그러면:

```text
grandparent
┌─────────────────────────────┐
│                             │
│                   Child     │
│                  absolute   │
│                             │
└─────────────────────────────┘
```

처럼 grandparent가 기준이 될 수 있습니다.

아무런 적절한 기준 조상이 없다면 Initial Containing Block 등이 기준이 될 수 있습니다.

입문 단계에서는 다음 규칙을 먼저 기억하면 좋습니다.

```text
absolute를 사용한다
      ↓
어떤 Box를 기준으로
배치하고 싶은가?
      ↓
그 기준 조상에
position: relative
      ↓
absolute child 배치
```

---

# 18. `top`, `right`, `bottom`, `left`

Absolute Positioning에서는 이 Property들의 역할이 매우 직관적입니다.

```css
.child {
  position: absolute;

  top: 20px;
  left: 30px;
}
```

개념적으로:

```text
Containing Block
┌─────────────────────────────┐
│        ↓ 20px               │
│     → 30px                  │
│       ┌─────────┐           │
│       │ Child   │           │
│       └─────────┘           │
│                             │
└─────────────────────────────┘
```

오른쪽 위:

```css
top: 10px;
right: 10px;
```

오른쪽 아래:

```css
right: 10px;
bottom: 10px;
```

등으로 사용할 수 있습니다.

---

# 19. `inset`

네 방향을 한꺼번에 지정할 수도 있습니다.

```css
.overlay {
  position: absolute;

  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
```

이를 다음처럼 작성할 수 있습니다.

```css
.overlay {
  position: absolute;

  inset: 0;
}
```

결과:

```text
Parent
┌─────────────────────────────┐
│┌───────────────────────────┐│
││                           ││
││          Overlay          ││
││                           ││
│└───────────────────────────┘│
└─────────────────────────────┘
```

Overlay UI에서 매우 유용한 패턴입니다.

---

# 20. Absolute를 이용한 중앙 배치

전통적으로 다음과 같은 방법도 사용합니다.

```css
.child {
  position: absolute;

  top: 50%;
  left: 50%;

  transform: translate(-50%, -50%);
}
```

개념적으로:

```text
Parent
┌─────────────────────────────┐
│                             │
│              ●              │
│         Parent 중심         │
│                             │
└─────────────────────────────┘
```

`top: 50%; left: 50%`는 Child의 기준점을 Parent의 중앙 위치로 이동시킵니다.

하지만 그 상태에서는 Child 자신의 왼쪽 위 모서리가 중앙에 위치합니다.

그래서:

```css
transform: translate(-50%, -50%);
```

로 Child 자신의 크기의 절반만큼 다시 이동시킵니다.

다만 단순 중앙 정렬만 목적이라면 현대 CSS에서는 Flexbox나 Grid가 더 편리한 경우가 많습니다.

---

# 21. `position: fixed`

`fixed`는 화면에 고정된 UI를 만들 때 많이 사용합니다.

```css
.chat {
  position: fixed;

  right: 30px;
  bottom: 30px;
}
```

일반적인 경우 Viewport를 기준으로 위치합니다.

```text
Browser Viewport
┌─────────────────────────────┐
│                             │
│                             │
│                             │
│                             │
│                       ┌───┐ │
│                       │ 💬│ │
│                       └───┘ │
└─────────────────────────────┘
```

페이지를 Scroll해도 같은 화면 위치에 유지됩니다.

---

# 22. Fixed도 Normal Flow에서 빠진다

`fixed` 역시 일반적으로 Normal Flow 공간을 차지하지 않습니다.

```text
Document

Header
Content
Content
Content
Content

                [Chat]
                  ↑
              fixed
```

Scroll:

```text
Document 이동

Content
Content
Content
Footer

                [Chat]
                  ↑
             화면 위치 유지
```

대표적인 사용 사례:

```text
Floating Action Button

채팅 버튼

화면 고정 Navigation

Back to Top 버튼
```

---

# 23. Fixed는 항상 Viewport 기준인가?

입문 단계에서는:

```text
fixed
→ Viewport 기준
```

으로 이해해도 대부분의 사례를 설명할 수 있습니다.

하지만 엄밀하게는 특정 CSS Property가 적용된 조상 등이 Fixed Positioned Element의 Containing Block을 형성할 수 있습니다.

예를 들어 일부 `transform` 등의 상황에서는 기대한 Viewport 기준과 달라질 수 있습니다.

따라서 정확한 표현은:

> **Fixed Positioned Element는 일반적으로 Viewport를 기준으로 하지만, 특정 조건에서는 다른 Containing Block을 가질 수 있다.**

입니다.

---

# 24. `position: sticky`

`sticky`는 `relative`와 `fixed`의 특성을 결합한 것처럼 보이는 Positioning 방식입니다.

예:

```css
.header {
  position: sticky;

  top: 0;
}
```

처음에는 Normal Flow 안에서 자신의 위치에 있습니다.

```text
Page

Content

Header

Content
Content
Content
```

Scroll하면서 Header가 `top: 0` 위치에 도달하면:

```text
Viewport
┌──────────────────────┐
│ Header ← top: 0      │
├──────────────────────┤
│ Content              │
│ Content              │
│ Content              │
└──────────────────────┘
```

처럼 붙어서 이동합니다.

---

# 25. Sticky의 동작 단계

개념적으로:

```text
1
Normal Flow 위치

        ↓ Scroll


2
Sticky 임계점 접근

        ↓


3
top: 0 도달

        ↓


4
해당 Scroll Container 안에서
붙어서 이동
```

즉:

```text
relative처럼 흐르다가
        ↓
지정한 임계점에서
        ↓
sticky 상태로 붙음
```

이라고 이해할 수 있습니다.

---

# 26. Sticky에는 Offset이 중요하다

다음처럼만 작성하면:

```css
.header {
  position: sticky;
}
```

어디에 붙어야 하는지 기준이 명확하지 않습니다.

보통 다음처럼 사용합니다.

```css
.header {
  position: sticky;
  top: 0;
}
```

즉:

```text
position: sticky
       +
top: 0
       ↓
Viewport/Scrollport 상단의
Sticky 기준 위치
```

가 됩니다.

---

# 27. Sticky와 Scroll Container

Sticky는 단순히 무조건 브라우저 화면에 붙는다고 생각하면 안 됩니다.

Sticky Positioning은 **자신과 관련된 Scroll Container 및 Containing Block의 제약**을 받습니다.

예:

```text
Scroll Container
┌───────────────────────────┐
│ Sticky Header             │
├───────────────────────────┤
│ Item                      │
│ Item                      │
│ Item                      │
│ Item                      │
│ Item                      │
└───────────────────────────┘
```

Sticky Element는 해당 영역 안에서 움직입니다.

부모 영역을 벗어나 무한정 화면에 붙어 있는 것은 아닙니다.

---

# 28. Fixed와 Sticky 비교

둘 다 Scroll 시 화면에 붙어 있는 것처럼 보일 수 있지만 동작 원리는 다릅니다.

| 특징             | fixed          | sticky                                 |
| -------------- | -------------- | -------------------------------------- |
| Normal Flow 공간 | 일반적으로 X        | O                                      |
| Scroll 전 원래 위치 | 별도 Positioning | Normal Flow                            |
| 기준             | 일반적으로 Viewport | Scroll Container와 Containing Block의 제약 |
| 특정 지점부터 고정     | X              | O                                      |
| 대표 용도          | Floating UI    | Sticky Header                          |

개념적으로:

```text
FIXED

처음부터
Viewport 특정 위치

[Button]


STICKY

Normal Flow
    ↓
Scroll
    ↓
임계점 도달
    ↓
붙음
```

---

# 29. 다섯 가지 Position 비교

전체를 한 번에 비교해 보겠습니다.

| position   | Normal Flow |           Offset | 핵심 기준                                |
| ---------- | ----------: | ---------------: | ------------------------------------ |
| `static`   |           O | 일반적인 inset 효과 없음 | Normal Flow                          |
| `relative` |           O |                O | 자신의 원래 위치                            |
| `absolute` |           X |                O | Containing Block                     |
| `fixed`    |           X |                O | 일반적으로 Viewport                       |
| `sticky`   |           O |                O | Scroll Container/Containing Block 제약 |

핵심 흐름:

```text
static
│
│ Normal Flow 그대로
│
▼
relative
│
│ Flow 유지 + 위치 조정
│
▼
absolute
│
│ Flow에서 제거
│
▼
fixed
│
│ 화면 기준 고정
│
▼
sticky
  Flow + Scroll 임계점
```

---

# 30. Position과 겹침

Positioning을 사용하면 Box들이 서로 겹치는 상황이 자주 발생합니다.

예:

```text
┌────────────────────┐
│       Box A        │
│          ┌─────────┼────┐
│          │  Box B  │    │
└──────────┼─────────┘    │
           └──────────────┘
```

그러면 새로운 질문이 생깁니다.

> **어떤 Box가 위에 표시되는가?**

여기서 `z-index`가 등장합니다.

---

# 31. `z-index`

`z-index`는 겹치는 Box들의 **Stacking Order**에 영향을 주는 Property입니다.

예:

```css
.box-a {
  position: relative;
  z-index: 1;
}

.box-b {
  position: relative;
  z-index: 2;
}
```

단순한 상황에서는:

```text
z-index: 2
      ↓
┌───────────┐
│   Box B   │ ← 위
└───────────┘

z-index: 1
      ↓
┌───────────┐
│   Box A   │ ← 아래
└───────────┘
```

처럼 이해할 수 있습니다.

하지만 `z-index`는 단순히 **숫자가 큰 Element가 페이지 전체에서 무조건 위에 온다**는 Property가 아닙니다.

---

# 32. Stacking Context

`z-index`를 제대로 이해하려면 **Stacking Context**라는 개념이 필요합니다.

입문 단계에서는 다음처럼 생각하면 좋습니다.

> **Stacking Context는 자식들의 앞뒤 순서를 독립적으로 관리하는 하나의 겹침 그룹이다.**

```text
Page
│
├── Stacking Context A
│    ├── Box A1
│    └── Box A2
│
└── Stacking Context B
     ├── Box B1
     └── Box B2
```

각 Context 내부의 `z-index` 경쟁과 Context 자체의 순서를 구분해야 합니다.

---

# 33. `z-index: 9999`가 항상 이기지 않는 이유

예를 들어:

```text
Parent A
z-index: 1

└── Child
    z-index: 9999


Parent B
z-index: 2
```

Parent A와 Parent B가 각각 Stacking Context를 형성하고 Parent B가 위에 있다면, Child의 `9999`만으로 Parent A의 Context를 벗어나 Parent B 위로 올라갈 수 없습니다.

개념적으로:

```text
Stacking Context B
z-index: 2
┌─────────────────────┐
│                     │
└─────────────────────┘
          ↑
          │

Stacking Context A
z-index: 1
┌─────────────────────┐
│ Child               │
│ z-index: 9999       │
└─────────────────────┘
```

즉:

> **`z-index` 문제는 숫자만 볼 것이 아니라 Stacking Context부터 확인해야 합니다.**

---

# 34. Stacking Context를 만드는 대표적인 경우

대표적인 예로 다음과 같은 상황에서 새로운 Stacking Context가 만들어질 수 있습니다.

```css
position: relative;
z-index: 1;
```

또는:

```css
opacity: 0.9;
```

```css
transform: translateX(0);
```

등 여러 조건이 있습니다.

모든 조건을 입문 단계에서 암기할 필요는 없습니다.

중요한 것은:

```text
겹침 문제 발생
      ↓
z-index 숫자만 증가
      ↓
해결 안 됨
      ↓
Stacking Context 확인
```

이라는 사고방식을 갖는 것입니다.

---

# 35. Position과 `display`의 관계

PART 5에서 배운 `display`와 Position은 서로 다른 역할을 합니다.

```text
display
│
├── Outer Display
│
└── Inner Display
      ↓
Box가 Layout에
어떻게 참여하는가?
```

반면:

```text
position
      ↓
Box의 위치를
어떤 방식으로 결정하는가?
```

따라서 동시에 사용할 수 있습니다.

```css
.card {
  display: flex;
  position: relative;
}
```

이 경우:

```text
.card 자신
│
├── position: relative
│   → Positioning 관계
│
└── display: flex
    → 자식들의 Flex Layout
```

입니다.

---

# 36. 실전 예제 1 — 카드의 SALE 배지

HTML:

```html
<div class="card">

  <span class="badge">
    SALE
  </span>

  <h2>Keyboard</h2>

  <p>50,000원</p>

</div>
```

CSS:

```css
.card {
  position: relative;

  width: 300px;
  padding: 20px;

  border: 1px solid #ddd;
}

.badge {
  position: absolute;

  top: 10px;
  right: 10px;
}
```

구조:

```text
Card
position: relative

┌─────────────────────────┐
│                  SALE   │
│                         │
│ Keyboard                │
│                         │
│ 50,000원                │
└─────────────────────────┘
```

핵심 관계:

```text
.card
relative
   ↓
Containing Block

.badge
absolute
   ↓
card 기준으로 위치 계산
```

---

# 37. 실전 예제 2 — 이미지 Overlay

HTML:

```html
<div class="image-box">

  <img src="product.jpg" alt="상품">

  <div class="overlay">
    자세히 보기
  </div>

</div>
```

CSS:

```css
.image-box {
  position: relative;
}

.overlay {
  position: absolute;

  inset: 0;

  display: flex;
  align-items: center;
  justify-content: center;
}
```

구조:

```text
image-box
┌─────────────────────────┐
│                         │
│     Product Image       │
│                         │
│   ┌─────────────────┐   │
│   │   자세히 보기   │   │
│   └─────────────────┘   │
│                         │
└─────────────────────────┘
```

여기서는 여러 CSS Layout 개념이 함께 사용됩니다.

```text
position: relative
→ Overlay 기준

position: absolute
inset: 0
→ 부모 전체 덮기

display: flex
→ Overlay 내부 Content 중앙 정렬
```

---

# 38. 실전 예제 3 — Floating Button

HTML:

```html
<button class="chat">
  Chat
</button>
```

CSS:

```css
.chat {
  position: fixed;

  right: 30px;
  bottom: 30px;

  width: 60px;
  height: 60px;

  border-radius: 50%;
}
```

결과:

```text
Viewport
┌──────────────────────────────┐
│                              │
│          Page Content        │
│                              │
│                              │
│                        ┌───┐ │
│                        │Chat│ │
│                        └───┘ │
└──────────────────────────────┘
```

Scroll해도 같은 화면 위치를 유지합니다.

---

# 39. 실전 예제 4 — Sticky Header

HTML:

```html
<header class="header">
  Navigation
</header>

<main>
  ...
</main>
```

CSS:

```css
.header {
  position: sticky;

  top: 0;

  background: white;
  z-index: 10;
}
```

Scroll 전:

```text
Content
Header
Content
Content
```

Scroll 후:

```text
Viewport
┌─────────────────────────┐
│ Header                  │ ← sticky
├─────────────────────────┤
│ Content                 │
│ Content                 │
│ Content                 │
└─────────────────────────┘
```

---

# 40. 언제 어떤 Position을 사용하는가?

### `static`

```text
특별한 Positioning이 필요 없음
```

대부분의 일반 Content.

### `relative`

```text
원래 위치 기준 미세 조정

또는

absolute 자식의 기준 제공
```

### `absolute`

```text
다른 Box 위에 정확하게 배치

Badge
Overlay
Icon
Tooltip
```

### `fixed`

```text
Viewport에 고정

Floating Button
고정 메뉴
```

### `sticky`

```text
Scroll하다 특정 위치에서 붙기

Header
Table Header
Sidebar
```

---

# 41. 자주 하는 실수 1

### "`absolute`는 부모 기준이다."

항상 그렇지는 않습니다.

정확하게는:

```text
absolute
      ↓
Containing Block
      ↓
위치 계산
```

입니다.

실무에서 부모에:

```css
position: relative;
```

를 많이 넣기 때문에 부모 기준처럼 보이는 것입니다.

---

# 42. 자주 하는 실수 2

### "`relative`를 사용하면 Normal Flow에서 빠진다."

아닙니다.

```text
relative

Normal Flow 공간 유지
        O
```

입니다.

시각적으로 이동해도 원래 공간은 유지됩니다.

반면:

```text
absolute
fixed

Normal Flow 공간
        X
```

입니다.

---

# 43. 자주 하는 실수 3

### "`fixed`는 언제나 무조건 Viewport 기준이다."

일반적으로는 맞지만 엄밀히 항상 그런 것은 아닙니다.

특정 조상의 `transform` 등으로 인해 다른 Containing Block이 형성될 수 있습니다.

입문 단계:

```text
fixed
≈ Viewport 기준
```

심화 단계:

```text
fixed
→ Containing Block 규칙 확인
```

으로 확장하면 됩니다.

---

# 44. 자주 하는 실수 4

### "`sticky`가 동작하지 않는다."

다음부터 확인합니다.

```text
position: sticky가 있는가?
        ↓
top / bottom 등의
임계값이 있는가?
        ↓
어떤 Scroll Container에
속해 있는가?
        ↓
부모의 크기는 충분한가?
        ↓
overflow 구조는 어떠한가?
```

Sticky는 주변 Scroll/Layout 구조의 영향을 많이 받습니다.

---

# 45. 자주 하는 실수 5

### "`z-index`는 숫자가 크면 무조건 위에 나온다."

아닙니다.

```text
z-index
     ↓
Stacking Context 안에서 비교
```

가 핵심입니다.

따라서:

```css
z-index: 999999;
```

로 숫자만 계속 올리는 것은 좋은 디버깅 방법이 아닙니다.

먼저 **Stacking Context 구조**를 확인해야 합니다.

---

# 46. Position을 사용할 때의 사고 순서

Element를 원하는 위치에 놓으려고 바로 `absolute`부터 사용하지 않는 것이 중요합니다.

먼저 다음 순서로 생각합니다.

```text
1
Normal Flow로 가능한가?
        ↓

2
Flexbox / Grid의
정렬 문제인가?
        ↓

3
Box를 독립적으로
겹치거나 고정해야 하는가?
        ↓

4
Position 사용
```

예를 들어:

```text
버튼 3개를 가로 중앙 정렬
```

은 Position 문제가 아니라 Flexbox가 더 적합합니다.

반면:

```text
카드 오른쪽 위에
SALE Badge 겹치기
```

는 Position이 적합합니다.

---

# 47. Layout과 Position의 관계

CSS Layout 전체에서 Position의 위치를 정리해 보겠습니다.

```text
HTML Element
      ↓
CSS 적용
      ↓
CSS Box
      ↓
Box Model
      ↓
Size & Units
      ↓
display
      ↓
Normal Flow
      ↓
position
      ↓
필요하면
Normal Flow 위치 조정
또는 별도 Positioning
```

하지만 Position이 모든 Layout을 담당하는 것은 아닙니다.

```text
CSS Layout
│
├── Normal Flow
│
├── Positioning
│
├── Flexbox
└── Grid
```

각각 역할이 다릅니다.

---

# 48. PART 6 전체 구조

```text
position
│
├── static
│     │
│     └── Normal Flow 그대로
│
├── relative
│     │
│     ├── Normal Flow 유지
│     └── 자신의 원래 위치 기준 이동
│
├── absolute
│     │
│     ├── Normal Flow에서 제거
│     └── Containing Block 기준
│
├── fixed
│     │
│     ├── Normal Flow에서 제거
│     └── 일반적으로 Viewport 기준
│
└── sticky
      │
      ├── Normal Flow 참여
      └── Scroll 임계점에서 붙음
```

그리고 Box가 겹치면:

```text
Positioned Boxes
      ↓
Overlap
      ↓
Stacking Order
      ↓
Stacking Context
      ↓
z-index
```

로 연결됩니다.

---

# 49. PART 5와 PART 6의 연결

PART 5:

```text
display
      ↓
Normal Flow
      ↓
Box가 기본적으로
어떻게 배치되는가?
```

PART 6:

```text
position
      ↓
그 Box의 위치를
어떤 방식으로
조정하거나 고정하는가?
```

따라서:

```text
display
      ↓
기본 Layout 관계
      ↓
position
      ↓
위치 제어
```

로 이해할 수 있습니다.

---

# 50. 다음 단계 — Flexbox

Position까지 배우면 Element를 원하는 곳에 놓을 수 있을 것처럼 보입니다.

하지만 다음과 같은 UI가 있다고 하겠습니다.

```text
┌───────────────────────────────────────┐
│ Logo        Menu              Login   │
└───────────────────────────────────────┘
```

이것을 각각:

```css
position: absolute;
```

로 배치하는 것은 좋은 방법이 아닙니다.

왜냐하면 이것은 **여러 Sibling Box 사이의 정렬과 공간 분배 문제**이기 때문입니다.

이 문제를 해결하기 위한 Layout System이:

```text
Flexbox
```

입니다.

다음 PART에서는:

```css
display: flex;
```

를 중심으로 **1차원 Layout**을 배웁니다.

---

# PART 6 핵심 정리

```text
static
→ Normal Flow 그대로

relative
→ Flow 유지
→ 자신의 원래 위치 기준 이동

absolute
→ Flow에서 제거
→ Containing Block 기준 배치

fixed
→ Flow에서 제거
→ 일반적으로 Viewport 기준 고정

sticky
→ Flow에 참여
→ Scroll 임계점에서 붙음
```

그리고:

```text
Box 겹침
   ↓
Stacking Context
   ↓
z-index
```

를 함께 이해해야 합니다.

# PART 6의 가장 중요한 한 문장

> **CSS `position`을 이해하는 핵심은 `top`과 `left`를 외우는 것이 아니라, 해당 Box가 Normal Flow에 남아 있는지 그리고 어떤 Containing Block을 기준으로 위치가 계산되는지를 이해하는 것이다.**

