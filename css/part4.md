# PART 4. CSS Size & Units

## 1. CSS에서 크기는 어떻게 결정되는가?

PART 3에서는 HTML Element가 생성하는 CSS Box의 구조를 배웠습니다.

```text
HTML Element
      ↓
CSS Box
      ↓
┌─────────────────┐
│ Margin          │
│ ┌─────────────┐ │
│ │ Border      │ │
│ │ ┌─────────┐ │ │
│ │ │ Padding │ │ │
│ │ │ Content │ │ │
│ │ └─────────┘ │ │
│ └─────────────┘ │
└─────────────────┘
```

그리고 다음과 같은 CSS를 사용했습니다.

```css
.card {
  width: 300px;
  padding: 20px;
}
```

여기서 새로운 질문이 생깁니다.

> **`300px`은 무엇이며, CSS에서는 크기를 어떤 방법으로 표현할 수 있을까?**

CSS에서는 다양한 **Unit(단위)**을 사용하여 크기를 표현할 수 있습니다.

```css
width: 300px;

width: 80%;

font-size: 1.5rem;

width: 50vw;

height: 100vh;
```

이 값들은 모두 크기를 표현하지만 **크기가 결정되는 기준이 서로 다릅니다.**

---

# 2. CSS Value와 Unit

다음 CSS를 보겠습니다.

```css
.box {
  width: 300px;
}
```

`width`는 Property이고:

```text
width
```

`300px`는 Value입니다.

조금 더 나누면:

```text
300px
│  │
│  └── Unit
│      px
│
└── Number
    300
```

즉:

```text
300px

300 → 숫자
px  → 단위
```

입니다.

하지만 모든 CSS Value가 단위를 가지는 것은 아닙니다.

예:

```css
opacity: 0.5;

font-weight: 700;

line-height: 1.5;
```

따라서:

> **CSS Unit은 길이(Length) 등을 표현할 때 사용하는 단위 체계이다.**

라고 이해하면 됩니다.

---

# 3. CSS Length

CSS에서 크기나 거리를 나타내는 값을 **Length**라고 합니다.

예:

```css
width: 300px;

padding: 20px;

font-size: 2rem;

margin: 5vw;
```

대표적인 CSS Length Unit은 다음과 같습니다.

```text
CSS Length Units
│
├── Absolute Length
│    └── px
│
└── Relative Length
     ├── %
     ├── em
     ├── rem
     ├── vw
     └── vh
```

실제 CSS에는 더 많은 단위가 있지만 입문 단계에서는 이 여섯 가지를 먼저 이해하는 것이 중요합니다.

---

# 4. Absolute와 Relative

CSS 단위를 크게 이해하면:

```text
Absolute
vs
Relative
```

로 나눌 수 있습니다.

### Absolute

대표적인 예:

```css
width: 300px;
```

`px`는 다른 Element의 크기나 font-size를 직접 참조하여 계산하는 상대 단위가 아닙니다.

### Relative

예:

```css
width: 50%;
```

`50%`는 **어떤 기준 크기의 50%**라는 의미입니다.

또한:

```css
font-size: 2rem;
```

은 Root Element의 font-size를 기준으로 계산됩니다.

즉 Relative Unit을 볼 때 가장 중요한 질문은:

> **“무엇을 기준으로 상대적인가?”**

입니다.

---

# 5. `px`

가장 이해하기 쉬운 단위입니다.

```css
.box {
  width: 300px;
  height: 200px;
}
```

`px`는 CSS Pixel을 의미합니다.

입문 단계에서는 화면상의 크기를 직접 지정하는 대표적인 **고정 길이 단위**로 이해하면 됩니다.

```text
width: 300px

┌─────────────────────────────┐
│                             │
│            Box              │
│                             │
└─────────────────────────────┘
←────────── 300px ───────────→
```

대표적인 사용 예:

```css
.card {
  border: 1px solid gray;
  border-radius: 8px;
  padding: 20px;
}
```

---

# 6. CSS `px`와 물리적 Pixel

주의할 점이 있습니다.

CSS의 `1px`가 항상 디스플레이의 **물리적 Pixel 하나**와 정확히 같은 것은 아닙니다.

고해상도 디스플레이에서는 하나의 CSS Pixel을 여러 Device Pixel로 표현할 수 있습니다.

개념적으로:

```text
CSS Pixel

┌──────────────┐
│              │
│    1 CSS px  │
│              │
└──────────────┘
       ↓
고해상도 디스플레이
       ↓
┌─────┬─────┐
│device│device│
├─────┼─────┤
│device│device│
└─────┴─────┘
```

따라서 CSS의 `px`는 단순히 **모니터의 물리적 픽셀 하나**라고 정의하면 정확하지 않습니다.

웹 개발에서는 **CSS에서 사용하는 기준 길이 단위**로 이해하는 것이 좋습니다.

---

# 7. `%`

`%`는 상대적인 값을 나타냅니다.

예:

```css
.parent {
  width: 800px;
}

.child {
  width: 50%;
}
```

이 경우 Child의 width는:

```text
800px × 50%
=
400px
```

입니다.

그림:

```text
Parent
800px
┌────────────────────────────────────────┐
│                                        │
│ Child                                  │
│ width: 50%                             │
│ ┌──────────────────┐                   │
│ │      400px       │                   │
│ └──────────────────┘                   │
│                                        │
└────────────────────────────────────────┘
```

즉 여기서 `50%`는 부모 쪽에서 정해지는 기준 너비에 대해 계산됩니다.

---

# 8. `%`는 항상 부모의 width를 의미하는가?

아닙니다.

이 부분은 매우 중요합니다.

Percentage의 기준은 **Property에 따라 다릅니다.**

예를 들어:

```css
.child {
  width: 50%;
}
```

의 Percentage 기준과:

```css
.child {
  font-size: 150%;
}
```

의 기준은 같지 않습니다.

`font-size: 150%`는 부모 Element의 font-size를 기준으로 계산됩니다.

```text
Parent
font-size: 20px
      ↓

Child
font-size: 150%
      ↓

20 × 1.5
=
30px
```

따라서 `%`를 볼 때는:

> **이 Property에서 Percentage가 무엇을 기준으로 계산되는가?**

를 확인해야 합니다.

---

# 9. Percentage Padding과 Margin

초보자가 예상하기 어려운 대표적인 예입니다.

다음과 같은 CSS가 있다고 하겠습니다.

```css
.parent {
  width: 500px;
}

.child {
  padding: 10%;
}
```

일반적인 수평 Writing Mode에서 `padding-top`, `padding-bottom`을 포함한 Percentage Padding은 **Containing Block의 너비**를 기준으로 계산됩니다.

따라서:

```text
500px × 10%
=
50px
```

가 될 수 있습니다.

Margin의 Percentage도 일반적인 경우 Containing Block의 너비를 기준으로 계산됩니다.

따라서:

> **Percentage는 무조건 같은 방향의 크기를 기준으로 계산된다**

라고 생각하면 안 됩니다.

---

# 10. `em`

`em`은 **Font Size를 기준으로 하는 상대 단위**입니다.

예:

```css
.parent {
  font-size: 20px;
}

.child {
  font-size: 2em;
}
```

Child의 font-size는:

```text
20px × 2
=
40px
```

입니다.

```text
Parent
font-size: 20px
      │
      ▼
Child
font-size: 2em
      │
      ▼
40px
```

즉 `font-size` Property에서 `em`은 부모의 font-size를 기준으로 계산됩니다.

---

# 11. `em`의 기준은 Property에 따라 이해해야 한다

다음 예를 보겠습니다.

```css
.button {
  font-size: 20px;
  padding: 1em;
}
```

여기서:

```text
font-size = 20px

padding = 1em
        = 20px
```

입니다.

`padding`에서 `em`은 해당 Element 자신의 계산된 font-size를 기준으로 합니다.

따라서:

```css
.button {
  font-size: 30px;
  padding: 1em;
}
```

으로 변경하면 Padding도:

```text
30px
```

가 됩니다.

이 특징을 이용하면 **글자 크기에 비례하여 커지는 UI**를 만들 수 있습니다.

---

# 12. `em`의 장점

예를 들어 버튼을 다음처럼 작성할 수 있습니다.

```css
.button {
  font-size: 16px;

  padding: 0.5em 1em;
  border-radius: 0.4em;
}
```

Font Size를:

```css
font-size: 24px;
```

로 변경하면 Padding과 Border Radius도 함께 커집니다.

```text
font-size
    ↓
    em
    ↓
padding
border-radius
```

즉 Component 내부의 크기를 **Font Size에 비례하여 설계**할 때 유용합니다.

---

# 13. `em` 중첩 문제

`em`을 사용할 때 주의해야 할 부분입니다.

HTML:

```html
<div class="parent">
  Parent

  <div class="child">
    Child
  </div>
</div>
```

CSS:

```css
.parent {
  font-size: 2em;
}

.child {
  font-size: 2em;
}
```

Root에서 상속된 font-size가 `16px`이라고 가정하면:

```text
Parent
2em
↓
16 × 2
=
32px


Child
2em
↓
32 × 2
=
64px
```

처럼 중첩에 따라 크기가 계속 커질 수 있습니다.

```text
16px
 ↓ ×2
32px
 ↓ ×2
64px
 ↓ ×2
128px
```

이러한 특성 때문에 전체 Typography 크기를 관리할 때는 `rem`이 더 직관적인 경우가 많습니다.

---

# 14. `rem`

`rem`은 **Root EM**을 의미합니다.

기준은 Root Element인:

```html
<html>
```

의 font-size입니다.

예:

```css
html {
  font-size: 16px;
}

.title {
  font-size: 2rem;
}
```

계산:

```text
16px × 2
=
32px
```

입니다.

```text
<html>
font-size: 16px
      │
      ├──────────────┐
      ↓              ↓
    1rem           2rem
    16px           32px
```

---

# 15. `em`과 `rem`의 가장 중요한 차이

두 단위 모두 Font Size와 관련된 상대 단위이지만 기준이 다릅니다.

```text
em
↓
Element의 font-size와 관련
font-size 자체에서는 부모 font-size 기준


rem
↓
Root Element의 font-size 기준
```

예를 들어:

```css
html {
  font-size: 16px;
}

.parent {
  font-size: 20px;
}

.child-a {
  font-size: 2em;
}

.child-b {
  font-size: 2rem;
}
```

결과:

```text
child-a

2em
→ parent 20px 기준
→ 40px


child-b

2rem
→ root 16px 기준
→ 32px
```

입니다.

---

# 16. `rem`은 어디에 많이 사용하는가?

`rem`은 일관된 Typography와 Spacing System을 만들 때 유용합니다.

예:

```css
body {
  font-size: 1rem;
}

h1 {
  font-size: 2.5rem;
}

h2 {
  font-size: 2rem;
}

.card {
  padding: 1.5rem;
}

.section {
  margin-bottom: 3rem;
}
```

Root 기준이 하나이므로 전체 크기 체계를 이해하기 쉽습니다.

```text
Root Font Size
      │
      ├── 0.5rem
      ├── 1rem
      ├── 1.5rem
      ├── 2rem
      └── 3rem
```

---

# 17. `vw`

`vw`는 **Viewport Width**를 기준으로 하는 단위입니다.

```text
1vw
=
Viewport Width의 1%
```

따라서:

```css
.box {
  width: 50vw;
}
```

는 Viewport Width의 50%입니다.

Viewport가:

```text
1200px
```

이라면:

```text
1vw = 12px

50vw = 600px
```

입니다.

그림:

```text
Viewport = 1200px
┌──────────────────────────────────────────┐
│                                          │
│ ┌────────────────────┐                   │
│ │      50vw          │                   │
│ │      600px         │                   │
│ └────────────────────┘                   │
│                                          │
└──────────────────────────────────────────┘
```

---

# 18. `vh`

`vh`는 **Viewport Height**를 기준으로 합니다.

```text
1vh
=
Viewport Height의 1%
```

예:

```css
.hero {
  height: 100vh;
}
```

Viewport Height가 `900px`이라면 개념적으로:

```text
100vh
=
900px
```

입니다.

따라서 화면 높이를 가득 채우는 Hero Section 등을 만들 때 사용할 수 있습니다.

```text
Viewport
┌──────────────────────────────┐
│                              │
│                              │
│        Hero Section          │
│                              │ 100vh
│                              │
│                              │
└──────────────────────────────┘
```

---

# 19. 모바일에서 `100vh` 주의하기

모바일 브라우저에서는 주소창이나 브라우저 UI가 나타나고 사라질 수 있습니다.

따라서 전통적인 `100vh`가 사용자가 실제로 보고 있는 영역과 항상 기대한 방식으로 일치하지 않을 수 있습니다.

현대 CSS에는 이를 보완하기 위한 Viewport Unit이 있습니다.

```css
height: 100dvh;
```

대표적으로:

```text
svh
Small Viewport Height

lvh
Large Viewport Height

dvh
Dynamic Viewport Height
```

가 있습니다.

특히 실제 모바일 화면의 동적 변화에 대응할 때:

```css
min-height: 100dvh;
```

를 고려할 수 있습니다.

입문 단계에서는:

```text
vh
→ Viewport Height 기반

dvh
→ 동적으로 변하는 Viewport Height 대응
```

정도로 이해하면 충분합니다.

---

# 20. `%`와 `vw`의 차이

둘 다 상대적인 크기이지만 기준이 다릅니다.

예:

```css
.box-a {
  width: 50%;
}

.box-b {
  width: 50vw;
}
```

개념적으로:

```text
50%

Containing Block 기준
        ↓

Parent
┌───────────────────────┐
│ ┌───────────┐         │
│ │    50%    │         │
│ └───────────┘         │
└───────────────────────┘
```

반면:

```text
50vw

Viewport 기준
        ↓

Browser Viewport
┌────────────────────────────────┐
│ ┌───────────────┐              │
│ │     50vw      │              │
│ └───────────────┘              │
└────────────────────────────────┘
```

즉:

```text
%
→ Property에 정의된 Percentage 기준

vw
→ Viewport Width

vh
→ Viewport Height
```

입니다.

---

# 21. `width`와 `height`

가장 기본적인 Size Property입니다.

```css
.box {
  width: 300px;
  height: 200px;
}
```

하지만 PART 3에서 배운 `box-sizing`에 따라 의미가 달라질 수 있습니다.

```css
box-sizing: content-box;
```

이면 `width/height`는 Content Box 크기에 적용되고:

```css
box-sizing: border-box;
```

이면 지정된 `width/height`가 Border Box 크기를 결정합니다.

따라서:

```text
Size Unit
      +
box-sizing
      ↓
실제 Box 크기
```

로 함께 생각해야 합니다.

---

# 22. `width: auto`

`width`의 디폴트 값은:

```css
width: auto;
```

입니다.

`auto`를 단순히:

> "width: 100%"

라고 이해하면 안 됩니다.

`auto`의 실제 크기는 Formatting Context와 다른 Property의 영향을 받아 계산됩니다.

Normal Flow의 일반적인 Block Box에서는 사용 가능한 가로 공간을 채우는 것처럼 보이는 경우가 많습니다.

```text
Containing Block
┌────────────────────────────────┐
│                                │
│ Block                          │
│ width: auto                    │
│ ┌────────────────────────────┐ │
│ │                            │ │
│ └────────────────────────────┘ │
│                                │
└────────────────────────────────┘
```

이 부분은 PART 5의 `display`와 Normal Flow에서 더 정확하게 연결합니다.

---

# 23. `height: auto`

`height`의 디폴트도 일반적으로:

```css
height: auto;
```

입니다.

Element의 높이를 지정하지 않으면 일반적인 상황에서 Content에 따라 필요한 높이가 결정됩니다.

```html
<div class="box">
  <p>Hello</p>
  <p>CSS</p>
  <p>Layout</p>
</div>
```

```css
.box {
  height: auto;
}
```

개념적으로:

```text
Content
  ↓
Hello
CSS
Layout
  ↓
필요한 만큼
Box Height 증가
```

따라서 일반적인 웹 Layout에서는 높이를 무조건 고정하기보다 Content에 따라 늘어나도록 두는 경우가 많습니다.

---

# 24. `min-width`

Box가 너무 작아지지 않도록 최소 너비를 지정할 수 있습니다.

```css
.box {
  width: 50%;
  min-width: 300px;
}
```

의미:

```text
가능하면
width: 50%

하지만

300px보다 작아지지 않음
```

개념적으로:

```text
width 계산
    ↓
50%
    ↓
300px보다 큰가?
   ↙       ↘
 Yes       No
  ↓         ↓
사용      300px
```

---

# 25. `max-width`

반대로 최대 너비를 제한할 수도 있습니다.

```css
.container {
  width: 90%;
  max-width: 1200px;
}
```

Viewport가 작으면:

```text
width: 90%
```

가 적용되어 유연하게 작아집니다.

Viewport가 매우 커져도:

```text
max-width: 1200px
```

때문에 Container가 계속 커지지는 않습니다.

```text
Small Screen

┌──────────────────────┐
│  ←──── 90% ────→    │
└──────────────────────┘


Large Screen

┌────────────────────────────────────────┐
│       ←──── max 1200px ────→          │
└────────────────────────────────────────┘
```

실제 웹 페이지의 Main Container에서 매우 자주 사용하는 패턴입니다.

---

# 26. 실전 Container 패턴

대표적인 Layout Container를 만들어 보겠습니다.

```css
.container {
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
}
```

각 Property의 역할은:

```text
width: 90%
│
└── 작은 화면에서 유연하게 줄어듦


max-width: 1200px
│
└── 큰 화면에서 너무 넓어지지 않음


margin: 0 auto
│
└── 남는 가로 공간을 이용해 중앙 배치
```

입니다.

결과:

```text
Browser
┌─────────────────────────────────────────────┐
│                                             │
│    ┌───────────────────────────────────┐    │
│    │                                   │    │
│    │            Container              │    │
│    │                                   │    │
│    └───────────────────────────────────┘    │
│                                             │
└─────────────────────────────────────────────┘
```

이 패턴은 이후 Responsive Layout에서도 계속 사용하게 됩니다.

---

# 27. `min-height`

최소 높이도 지정할 수 있습니다.

```css
main {
  min-height: 500px;
}
```

또는 Viewport를 기준으로:

```css
main {
  min-height: 100vh;
}
```

현대 모바일 환경까지 고려하면:

```css
main {
  min-height: 100dvh;
}
```

같은 방법도 사용할 수 있습니다.

`min-height`는 Content가 많아지면 Box가 더 커질 수 있다는 점에서 고정 `height`와 차이가 있습니다.

---

# 28. `height`와 `min-height`

비교해 보겠습니다.

```css
.box {
  height: 300px;
}
```

는 기본적으로 지정 높이를 기준으로 Box 크기를 결정합니다.

Content가 너무 많으면 Overflow 문제가 발생할 수 있습니다.

```text
┌──────────────┐
│ Content      │
│ Content      │
│ Content      │
└──────────────┘
 Content ↓
 Content ↓
```

반면:

```css
.box {
  min-height: 300px;
}
```

는:

```text
최소 300px
+
Content가 많으면
더 커질 수 있음
```

이라는 의미입니다.

따라서 Content 양이 가변적인 UI에서는 `min-height`가 더 적절한 경우가 있습니다.

---

# 29. `max-height`

최대 높이를 제한할 수도 있습니다.

```css
.list {
  max-height: 300px;
}
```

Content가 많다면 `overflow`와 함께 사용할 수 있습니다.

```css
.list {
  max-height: 300px;
  overflow-y: auto;
}
```

개념적으로:

```text
List
┌─────────────────────┐
│ Item 1              │
│ Item 2              │
│ Item 3              │
│ Item 4              │ ↑
│ Item 5              │ │ Scroll
└─────────────────────┘ ↓
```

`overflow`는 이후 Layout 관련 내용에서 다시 다룰 수 있습니다.

---

# 30. 고정 크기와 유연한 크기

두 가지 방식을 비교해 보겠습니다.

### 고정 크기

```css
.container {
  width: 1200px;
}
```

작은 화면에서는 문제가 생길 수 있습니다.

```text
Mobile Viewport
┌───────────────────┐
│                   │
│ ┌───────────────────────────────
│ │      1200px Container
│ └───────────────────────────────
│
└───────────────────┘
          →
       Overflow
```

### 유연한 크기

```css
.container {
  width: 90%;
  max-width: 1200px;
}
```

화면 크기에 맞게 조절됩니다.

```text
Desktop
→ 최대 1200px

Tablet
→ 화면의 90%

Mobile
→ 화면의 90%
```

이것이 Responsive Design의 기초입니다.

---

# 31. 어떤 Unit을 사용해야 하는가?

모든 상황에서 하나의 Unit만 사용하는 것은 좋지 않습니다.

용도에 따라 선택합니다.

| 목적                 | 자주 고려하는 단위                |
| ------------------ | ------------------------- |
| 얇은 Border          | `px`                      |
| 고정된 작은 세부 크기       | `px`                      |
| Container 너비       | `%`, `px`와 `max-width` 조합 |
| Font Size          | `rem`                     |
| Component 내부 비례 크기 | `em`                      |
| Viewport 기반 크기     | `vw`, `vh`, `dvh`         |
| 전체 화면 Section      | `vh`, `dvh`               |

예:

```css
.card {
  width: 90%;
  max-width: 400px;

  padding: 1.5rem;

  border: 1px solid #ddd;
  border-radius: 8px;
}
```

실제 CSS에서는 여러 Unit을 목적에 따라 함께 사용하는 것이 일반적입니다.

---

# 32. 실전 예제

HTML:

```html
<div class="container">

  <article class="card">
    <h2>Keyboard</h2>
    <p>Mechanical Keyboard</p>
    <strong>50,000원</strong>
  </article>

</div>
```

CSS:

```css
html {
  font-size: 16px;
}

* {
  box-sizing: border-box;
}

.container {
  width: 90%;
  max-width: 1200px;

  margin: 0 auto;
}

.card {
  width: 100%;
  max-width: 400px;

  padding: 1.5rem;

  border: 1px solid #ddd;
  border-radius: 8px;
}

.card h2 {
  font-size: 2rem;
}
```

각 크기를 분석하면:

```text
.container

width: 90%
→ 사용 가능한 기준 너비에 대해 유연하게 결정

max-width: 1200px
→ 너무 커지는 것 방지


.card

width: 100%
→ 사용할 수 있는 너비 활용

max-width: 400px
→ 카드 최대 너비 제한

padding: 1.5rem
→ Root Font Size 기준


h2

font-size: 2rem
→ Root Font Size의 2배
```

입니다.

---

# 33. 자주 하는 실수 1 — `%`를 무조건 부모 크기라고 생각하기

다음 두 코드는 `%`를 사용하지만 기준이 다릅니다.

```css
.box {
  width: 50%;
}
```

그리고:

```css
.box {
  font-size: 150%;
}
```

따라서:

> **Percentage의 기준은 Property의 정의에 따라 확인해야 합니다.**

---

# 34. 자주 하는 실수 2 — `em`과 `rem`을 같은 단위라고 생각하기

둘은 기준이 다릅니다.

```text
em
→ 현재 Element/부모의 Font Size 관계


rem
→ Root Element Font Size
```

특히 중첩된 Element에서 차이가 크게 나타날 수 있습니다.

---

# 35. 자주 하는 실수 3 — `100vw`를 Container에 무조건 사용하기

```css
.container {
  width: 100vw;
}
```

가 항상:

```css
width: 100%;
```

와 같은 결과를 만드는 것은 아닙니다.

`vw`는 Viewport를 기준으로 하고 `%`는 해당 Property의 Percentage 기준을 사용합니다.

특히 페이지 구조와 스크롤바 환경 등에 따라 `100vw`가 예상치 못한 수평 Overflow를 만드는 경우도 있습니다.

일반적인 Page Container라면 무조건 `100vw`를 사용하는 것보다:

```css
width: 100%;
```

또는:

```css
width: 90%;
max-width: 1200px;
```

같은 방법이 더 적절할 수 있습니다.

---

# 36. 자주 하는 실수 4 — 모든 높이를 `height`로 고정하기

예:

```css
.card {
  height: 200px;
}
```

Content가 늘어나면 문제가 발생할 수 있습니다.

```text
Fixed Height
┌───────────────────┐
│ Content           │
│ Content           │
│ Content           │
└───────────────────┘
  Content
  Content
     ↓
  Overflow
```

Content가 가변적이라면:

```css
min-height: 200px;
```

또는 `height: auto` 상태를 유지하는 것이 더 적절할 수 있습니다.

---

# 37. Unit을 이해하는 가장 중요한 방법

CSS Unit을 단순히 다음처럼 외우면:

```text
px
%
em
rem
vw
vh
```

실전에서 금방 혼란스러워집니다.

각 Unit을 볼 때 반드시 다음 질문을 해야 합니다.

> **“이 값은 무엇을 기준으로 계산되는가?”**

정리하면:

```text
px
→ CSS Pixel 기준의 길이


%
→ Property마다 정의된 Percentage 기준


em
→ Font Size 기준


rem
→ Root Element의 Font Size 기준


vw
→ Viewport Width


vh
→ Viewport Height
```

이것이 PART 4의 가장 중요한 개념입니다.

---

# 38. PART 4 핵심 정리

CSS에서는 Box의 크기를 다양한 Unit으로 표현할 수 있습니다.

```text
CSS Length
│
├── px
│
├── %
│
├── em
│
├── rem
│
├── vw
└── vh
```

하지만 중요한 것은 Unit의 이름을 암기하는 것이 아니라 **계산 기준을 이해하는 것**입니다.

```text
px
→ CSS Pixel

%
→ Property에 따른 기준

em
→ Font Size

rem
→ Root Font Size

vw
→ Viewport Width

vh
→ Viewport Height
```

Box의 크기는 다음과 같은 Property로 제어할 수 있습니다.

```css
width
height

min-width
max-width

min-height
max-height
```

그리고 실전에서는:

```css
.container {
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
}
```

처럼 **상대적인 크기와 제한 크기를 조합**하는 경우가 많습니다.

---

# 39. PART 3과 PART 4의 연결

PART 3에서는:

```text
Box는 어떻게 구성되는가?
        ↓
Content
Padding
Border
Margin
```

를 배웠습니다.

PART 4에서는:

```text
그 Box의 크기를
어떤 값으로 결정할 것인가?
        ↓
px
%
em
rem
vw
vh
        ↓
width / height
min / max
```

를 배웠습니다.

이제 다음 질문이 남습니다.

> **크기가 결정된 여러 Box는 브라우저 안에서 기본적으로 어떻게 배치되는가?**

이것이 PART 5의 주제입니다.

```text
HTML Element
      ↓
CSS 적용
      ↓
CSS Box
      ↓
Box Model
PART 3
      ↓
Size & Unit
PART 4
      ↓
Display & Normal Flow
PART 5
      ↓
Flexbox / Grid
```

---

# PART 4의 가장 중요한 한 문장

> **CSS Unit을 이해한다는 것은 단위를 외우는 것이 아니라, 그 값이 무엇을 기준으로 계산되는지를 이해하는 것이다.**
