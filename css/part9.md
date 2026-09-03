# PART 9. Responsive CSS

## 1. 반응형 웹 디자인이란?

지금까지 우리는 다양한 CSS Layout 방법을 배웠습니다.

```text
Box Model
   ↓
Size & Units
   ↓
Display
   ↓
Position
   ↓
Flexbox
   ↓
Grid
```

하지만 실제 웹 페이지는 하나의 화면 크기에서만 실행되지 않습니다.

사용자는 다음과 같은 다양한 Device에서 웹 페이지를 봅니다.

```text
Desktop
1920px

Laptop
1440px

Tablet
768px

Mobile
390px
```

같은 HTML을 사용하더라도 화면 크기에 따라 Layout이 달라져야 합니다.

이처럼 **Viewport 크기와 Device 환경에 맞춰 Layout과 Style이 자연스럽게 변하도록 설계하는 방식**을 **Responsive Web Design(반응형 웹 디자인)**이라고 합니다.

---

# 2. 반응형 Layout이 필요한 이유

다음과 같은 Desktop Layout이 있다고 하겠습니다.

```text
┌─────────────────────────────────────────┐
│ Header                                  │
├───────────┬─────────────────────────────┤
│ Sidebar   │ Main                        │
│           │                             │
│           │ Card Card Card              │
│           │ Card Card Card              │
└───────────┴─────────────────────────────┘
```

이 Layout을 Mobile 화면에서 그대로 보여주면 문제가 발생합니다.

```text
Mobile Viewport
┌───────────────┐
│ Header        │
├─────┬─────────┼────────────→
│Side │ Main    │
│bar  │         │
│     │ Card ...│
└─────┴─────────┘
```

수평 Scroll이 생기거나 Content가 너무 좁아질 수 있습니다.

Mobile에서는 다음처럼 바뀌는 것이 자연스럽습니다.

```text
┌──────────────────┐
│ Header           │
├──────────────────┤
│ Main             │
│                  │
│ Card             │
│ Card             │
│ Card             │
└──────────────────┘
```

즉 반응형 디자인의 핵심은:

> 화면이 작아졌을 때 단순히 전체를 축소하는 것이 아니라 **Layout 자체를 재구성하는 것**입니다.

---

# 3. Viewport란?

Responsive CSS를 이해하려면 먼저 **Viewport**를 이해해야 합니다.

Viewport는 브라우저에서 웹 페이지가 실제로 보이는 영역을 의미합니다.

```text
Browser Window
┌───────────────────────────────────────┐
│ Toolbar                               │
├───────────────────────────────────────┤
│                                       │
│              Viewport                 │
│                                       │
│                                       │
└───────────────────────────────────────┘
```

CSS의 다음 단위들이 Viewport를 기준으로 합니다.

```text
vw
vh
dvw
dvh
```

예:

```css
.hero {
  min-height: 100dvh;
}
```

---

# 4. Mobile Viewport 설정

HTML 문서에서는 일반적으로 다음 Meta Tag를 사용합니다.

```html
<meta
  name="viewport"
  content="width=device-width, initial-scale=1.0"
/>
```

이 설정은 Mobile Browser에서 CSS Layout Viewport를 Device Width에 맞게 설정하는 데 중요합니다.

구조:

```text
width=device-width
        ↓
CSS Viewport Width를
Device Width에 맞춤


initial-scale=1.0
        ↓
초기 확대 비율 지정
```

일반적인 React/Vite 프로젝트에서도 기본 HTML에 이 Meta Tag가 들어 있는 경우가 많습니다.

---

# 5. 반응형 CSS의 기본 전략

Responsive Layout은 크게 다음 요소들의 조합으로 만들어집니다.

```text
Responsive CSS
│
├── Flexible Width
│     %
│     max-width
│
├── Relative Units
│     rem
│     vw
│     vh
│
├── Flexible Layout
│     Flexbox
│     Grid
│
├── Media Query
│
└── Responsive Images
```

즉 Media Query만 사용한다고 반응형 디자인이 되는 것은 아닙니다.

가능한 부분은 먼저 **유연한 Layout**으로 해결하고, 특정 시점에 Layout 구조를 변경해야 할 때 Media Query를 사용하는 것이 좋습니다.

---

# 6. 고정 Layout의 문제

다음 CSS를 보겠습니다.

```css
.container {
  width: 1200px;
}
```

Desktop에서는 잘 보일 수 있습니다.

```text
Desktop 1440px

┌──────────────────────────────────────────┐
│     ┌──────────────────────────────┐     │
│     │       Container 1200px       │     │
│     └──────────────────────────────┘     │
└──────────────────────────────────────────┘
```

하지만 Mobile 390px에서는:

```text
Viewport 390px
┌───────────────┐
│ ┌───────────────────────────────→
│ │ Container 1200px
│ └───────────────────────────────→
└───────────────┘
```

수평 Overflow가 발생합니다.

---

# 7. 유연한 Container

다음처럼 만들 수 있습니다.

```css
.container {
  width: 90%;
  max-width: 1200px;

  margin: 0 auto;
}
```

작은 화면:

```text
Viewport 390px

┌───────────────────┐
│ ┌───────────────┐ │
│ │      90%      │ │
│ └───────────────┘ │
└───────────────────┘
```

큰 화면:

```text
Viewport 1600px

┌─────────────────────────────────────────┐
│       ┌───────────────────────────┐     │
│       │       max 1200px          │     │
│       └───────────────────────────┘     │
└─────────────────────────────────────────┘
```

이것은 PART 4에서 배운:

```text
width
max-width
margin: auto
```

의 실전 활용입니다.

---

# 8. `max-width: 100%`

이미지나 Media가 부모보다 커지는 것을 방지할 때 자주 사용하는 패턴입니다.

```css
img {
  max-width: 100%;
  height: auto;
}
```

이렇게 하면 이미지가 부모 Box보다 커지지 않습니다.

```text
Parent
┌───────────────────────┐
│ ┌───────────────────┐ │
│ │       Image       │ │
│ └───────────────────┘ │
└───────────────────────┘
```

`height: auto`는 원래 Aspect Ratio를 유지하는 데 도움을 줍니다.

---

# 9. Media Query란?

화면이나 Device 환경에 따라 CSS Rule을 조건부로 적용할 수 있습니다.

기본 문법:

```css
@media (조건) {
  /* 조건이 참일 때 적용 */
}
```

예:

```css
@media (max-width: 768px) {

  .container {
    width: 100%;
  }

}
```

의미:

```text
Viewport Width
      ↓
768px 이하인가?
   ↙       ↘
 Yes       No
  ↓         ↓
Rule 적용  적용 안 함
```

---

# 10. `max-width` Media Query

예:

```css
.page {
  display: flex;
}

@media (max-width: 768px) {

  .page {
    flex-direction: column;
  }

}
```

Desktop:

```text
┌─────────┬──────────────────────┐
│ Sidebar │ Main                 │
└─────────┴──────────────────────┘
```

768px 이하:

```text
┌───────────────────────────────┐
│ Sidebar                       │
├───────────────────────────────┤
│ Main                          │
└───────────────────────────────┘
```

즉 Media Query는 단순한 Style 변경뿐 아니라 **Layout 구조 자체를 변경**할 수 있습니다.

---

# 11. `min-width` Media Query

반대로 다음처럼 작성할 수도 있습니다.

```css
@media (min-width: 768px) {

  .page {
    display: flex;
  }

}
```

의미:

```text
768px 이상
   ↓
Desktop/Tablet용 Layout 적용
```

이 방식은 **Mobile First** 전략에서 매우 중요합니다.

---

# 12. Mobile First란?

Mobile 화면을 기본 CSS로 작성하고, 화면이 커질수록 Layout을 확장하는 방식입니다.

예:

```css
.cards {
  display: grid;

  grid-template-columns: 1fr;
}
```

Mobile 기본:

```text
[ Card ]

[ Card ]

[ Card ]
```

Tablet 이상:

```css
@media (min-width: 768px) {

  .cards {
    grid-template-columns:
      repeat(2, 1fr);
  }

}
```

```text
[ Card ] [ Card ]

[ Card ] [ Card ]
```

Desktop 이상:

```css
@media (min-width: 1200px) {

  .cards {
    grid-template-columns:
      repeat(4, 1fr);
  }

}
```

```text
[Card] [Card] [Card] [Card]
```

---

# 13. Desktop First

반대 방식도 있습니다.

Desktop Layout을 먼저 작성합니다.

```css
.cards {
  display: grid;
  grid-template-columns:
    repeat(4, 1fr);
}
```

작은 화면에서 줄입니다.

```css
@media (max-width: 768px) {

  .cards {
    grid-template-columns: 1fr;
  }

}
```

이를 **Desktop First**라고 합니다.

---

# 14. Mobile First vs Desktop First

개념적으로:

```text
Mobile First

작은 화면
   ↓
기본 CSS
   ↓
min-width
   ↓
큰 화면으로 확장
```

반면:

```text
Desktop First

큰 화면
   ↓
기본 CSS
   ↓
max-width
   ↓
작은 화면으로 축소
```

현대 웹에서는 Mobile First를 많이 사용하지만, 프로젝트 상황에 따라 둘 다 사용할 수 있습니다.

---

# 15. Breakpoint란?

Layout이 의미 있게 바뀌는 지점을 **Breakpoint**라고 합니다.

예:

```css
@media (min-width: 768px) {
}
```

여기서:

```text
768px
```

이 하나의 Breakpoint입니다.

하지만 중요한 점은:

> Breakpoint는 특정 Device 이름 때문에 정하는 것이 아니라 **Content와 Layout이 깨지는 시점**을 기준으로 정하는 것이 좋습니다.

즉:

```text
iPhone breakpoint
Tablet breakpoint
```

라고만 생각하기보다:

```text
"이 너비에서 Navigation이 더 이상
한 줄에 자연스럽게 들어가지 않는다."
```

와 같은 이유로 결정하는 것이 좋습니다.

---

# 16. Breakpoint 예

예를 들어 Navigation이:

```text
Logo   Home Products Services About Login
```

으로 잘 보이다가 화면이 줄어들면서:

```text
Logo Home Products Services
About Login
```

처럼 깨진다면:

```text
이 시점
   ↓
Breakpoint 후보
```

가 됩니다.

즉:

```text
Device 기준
X

Content 기준
O
```

라는 사고방식이 중요합니다.

---

# 17. 대표적인 Breakpoint 예시

강의나 실습에서는 다음처럼 단순화할 수 있습니다.

```text
Small
< 768px

Medium
>= 768px

Large
>= 1024px

Extra Large
>= 1280px
```

예:

```css
@media (min-width: 768px) {
}

@media (min-width: 1024px) {
}

@media (min-width: 1280px) {
}
```

하지만 이것은 절대적인 표준값이 아닙니다.

실제 Breakpoint는 프로젝트 Layout에 맞게 결정합니다.

---

# 18. Media Query 여러 조건

조건을 결합할 수 있습니다.

```css
@media
  (min-width: 768px) and
  (max-width: 1199px) {

  .container {
    width: 90%;
  }

}
```

의미:

```text
768px 이상
   AND
1199px 이하
```

즉 특정 범위에만 적용됩니다.

---

# 19. Range Syntax

현대 Media Query에서는 범위 문법을 사용할 수도 있습니다.

```css
@media (768px <= width < 1200px) {
}
```

또는:

```css
@media (width >= 768px) {
}
```

의미가 더 직관적일 수 있습니다.

입문 단계에서는 전통적인:

```css
min-width
max-width
```

를 먼저 익힌 후 이런 문법을 소개하면 좋습니다.

---

# 20. Orientation

화면 방향을 조건으로 사용할 수도 있습니다.

```css
@media (orientation: portrait) {
}
```

세로:

```text
┌────────┐
│        │
│        │
│        │
└────────┘
```

가로:

```css
@media (orientation: landscape) {
}
```

```text
┌──────────────────┐
│                  │
└──────────────────┘
```

---

# 21. Hover Capability

Device가 Hover를 지원하는지 확인할 수도 있습니다.

```css
@media (hover: hover) {

  .card:hover {
    transform: translateY(-4px);
  }

}
```

Touch Device에서는 Hover 개념이 Desktop Mouse와 다르게 동작할 수 있으므로 이런 조건이 유용할 수 있습니다.

---

# 22. Pointer

Pointer의 정밀도를 기준으로도 조건을 만들 수 있습니다.

```css
@media (pointer: coarse) {
}
```

`coarse`는 Touch처럼 상대적으로 정밀하지 않은 입력을 의미합니다.

예:

```css
@media (pointer: coarse) {

  button {
    min-height: 44px;
  }

}
```

Touch UI에서 버튼 영역을 크게 만드는 데 활용할 수 있습니다.

---

# 23. Reduced Motion

사용자의 접근성 설정을 확인할 수도 있습니다.

```css
@media (prefers-reduced-motion: reduce) {

  * {
    animation-duration: 0.01ms;
    transition-duration: 0.01ms;
  }

}
```

사용자가 Motion 감소를 선호하는 환경에서는 과도한 Animation을 줄일 수 있습니다.

Responsive CSS는 단순히 화면 크기뿐 아니라 **User Environment에 대응하는 CSS**로 확장됩니다.

---

# 24. Dark Mode Media Query

OS나 Browser의 색상 모드 선호를 감지할 수도 있습니다.

```css
@media (prefers-color-scheme: dark) {

  body {
    background: #111;
    color: #eee;
  }

}
```

다만 애플리케이션에서 사용자가 직접 Theme을 선택할 수 있다면 Class나 Data Attribute 방식과 함께 설계하는 경우도 많습니다.

---

# 25. Flexbox와 Responsive Layout

PART 7에서 배운 Flexbox는 Responsive Layout에 매우 유용합니다.

Desktop:

```css
.nav {
  display: flex;

  justify-content: space-between;
  align-items: center;
}
```

Mobile:

```css
@media (max-width: 768px) {

  .nav {
    flex-direction: column;
  }

}
```

Desktop:

```text
Logo     Menu     Login
```

Mobile:

```text
Logo

Menu

Login
```

---

# 26. `flex-wrap`을 이용한 자연스러운 반응형

항상 Media Query가 필요한 것은 아닙니다.

```css
.cards {
  display: flex;

  flex-wrap: wrap;

  gap: 20px;
}

.card {
  flex: 1 1 250px;
}
```

화면이 넓으면:

```text
[A] [B] [C] [D]
```

좁아지면:

```text
[A] [B]

[C] [D]
```

더 좁아지면:

```text
[A]

[B]

[C]

[D]
```

처럼 자연스럽게 줄바꿈됩니다.

---

# 27. Grid와 Responsive Layout

PART 8에서 배운 Grid는 Responsive Layout에 더욱 강력합니다.

예:

```css
.cards {
  display: grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(250px, 1fr)
    );

  gap: 20px;
}
```

화면 크기에 따라 Column 수가 자동으로 바뀝니다.

```text
Desktop

[A] [B] [C] [D]


Tablet

[A] [B]

[C] [D]


Mobile

[A]

[B]

[C]

[D]
```

Media Query 없이도 반응형 Card Layout을 만들 수 있습니다.

---

# 28. 언제 Media Query가 필요한가?

다음처럼 단순히 Item 개수만 조절되는 경우:

```text
4열 → 3열 → 2열 → 1열
```

Grid의:

```css
auto-fit
minmax()
```

만으로 해결할 수 있습니다.

하지만 Layout 구조 자체가 바뀌는 경우:

```text
Desktop

Sidebar | Main


Mobile

Main
```

처럼 Sidebar를 숨기거나 위치를 바꾸는 경우에는 Media Query가 필요할 수 있습니다.

---

# 29. Responsive Page Layout

Desktop:

```text
┌───────────────────────────────┐
│ Header                        │
├─────────┬─────────────────────┤
│ Sidebar │ Main                │
│         │                     │
└─────────┴─────────────────────┘
```

CSS:

```css
.page {
  display: grid;

  grid-template-columns:
    250px 1fr;
}
```

Mobile:

```css
@media (max-width: 768px) {

  .page {
    grid-template-columns: 1fr;
  }

  .sidebar {
    display: none;
  }

}
```

결과:

```text
┌──────────────────┐
│ Header           │
├──────────────────┤
│ Main             │
└──────────────────┘
```

---

# 30. Responsive Typography

Text 크기도 반응형으로 만들 수 있습니다.

전통적인 방법:

```css
.title {
  font-size: 32px;
}

@media (min-width: 1200px) {

  .title {
    font-size: 48px;
  }

}
```

하지만 현대 CSS에서는 `clamp()`를 사용할 수 있습니다.

---

# 31. `clamp()`

`clamp()`는 최소값, 선호값, 최대값을 지정합니다.

```css
font-size:
  clamp(2rem, 5vw, 4rem);
```

구조:

```text
clamp(
  최소값,
  선호값,
  최대값
)
```

즉:

```text
2rem보다 작지 않고

5vw를 기준으로 유연하게 변하며

4rem보다 커지지 않음
```

입니다.

---

# 32. `clamp()` 동작

예:

```css
.title {
  font-size:
    clamp(2rem, 5vw, 4rem);
}
```

작은 화면:

```text
2rem
```

중간 화면:

```text
5vw
```

큰 화면:

```text
4rem
```

개념:

```text
Small
│
│  min
│
▼
2rem
     ↗
      Fluid
       5vw
          ↗
           4rem
             ▲
             │
            max
```

이를 **Fluid Typography**에 많이 활용합니다.

---

# 33. `min()`

여러 값 중 더 작은 값을 사용합니다.

```css
width:
  min(90%, 1200px);
```

의미:

```text
90%
vs
1200px

둘 중 작은 값 사용
```

이 코드는 다음 패턴과 유사한 효과를 만들 수 있습니다.

```css
width: 90%;
max-width: 1200px;
```

---

# 34. `max()`

여러 값 중 더 큰 값을 사용합니다.

```css
padding:
  max(20px, 5vw);
```

화면이 작아도 최소한 20px의 Padding을 확보할 수 있습니다.

```text
20px
vs
5vw
  ↓
더 큰 값
```

---

# 35. `min()` + `max()` + `clamp()`

현대 Responsive CSS에서 매우 유용한 함수입니다.

```text
min()
→ 최대 제한 표현에 유용

max()
→ 최소 제한 표현에 유용

clamp()
→ 최소 + 유동 + 최대
```

예:

```css
.container {
  width:
    min(90%, 1200px);
}

.title {
  font-size:
    clamp(2rem, 5vw, 4rem);
}
```

---

# 36. Responsive Spacing

Spacing도 Fluid하게 만들 수 있습니다.

```css
.section {
  padding:
    clamp(2rem, 5vw, 6rem);
}
```

작은 화면:

```text
2rem
```

중간:

```text
5vw
```

큰 화면:

```text
6rem
```

따라서 Media Query를 여러 개 작성하지 않고도 자연스럽게 변하는 Spacing을 만들 수 있습니다.

---

# 37. Responsive Image

기본 패턴:

```css
img {
  display: block;

  max-width: 100%;
  height: auto;
}
```

이렇게 하면 이미지가 Container 폭에 맞춰 줄어듭니다.

하지만 큰 이미지를 Mobile에서 그대로 다운로드하면 Network 비용은 줄어들지 않습니다.

이를 해결하기 위해 HTML에서는:

```html
srcset
sizes
picture
```

등을 사용할 수 있습니다.

Responsive CSS와 Responsive Image는 서로 연결되지만 다른 문제입니다.

---

# 38. `aspect-ratio`

Element의 비율을 유지할 수 있습니다.

```css
.thumbnail {
  aspect-ratio: 16 / 9;
}
```

```text
┌──────────────────────────┐
│                          │
│        16 : 9            │
│                          │
└──────────────────────────┘
```

Responsive Card Image, Video, Thumbnail 등에 매우 유용합니다.

예:

```css
.thumbnail img {
  width: 100%;
  height: 100%;

  object-fit: cover;
}
```

---

# 39. `object-fit`

이미지가 지정된 Box 안에 어떻게 들어갈지 결정합니다.

```css
img {
  object-fit: cover;
}
```

대표 값:

```text
contain
→ 전체 이미지가 보이도록

cover
→ Box를 꽉 채우면서 일부 잘릴 수 있음
```

반응형 Thumbnail이나 Card Image에서 자주 사용합니다.

---

# 40. Mobile Navigation 예제

Desktop:

```text
┌───────────────────────────────────┐
│ Logo   Home Products About Login │
└───────────────────────────────────┘
```

Mobile:

```text
┌──────────────────┐
│ Logo        ☰    │
└──────────────────┘
```

CSS:

```css
.menu-button {
  display: none;
}

@media (max-width: 768px) {

  .menu {
    display: none;
  }

  .menu-button {
    display: block;
  }

}
```

실제로 메뉴 열기/닫기는 JavaScript나 React State와 함께 구현할 수 있습니다.

---

# 41. Responsive Card 예제

HTML:

```html
<section class="cards">

  <article class="card">
    Card 1
  </article>

  <article class="card">
    Card 2
  </article>

  <article class="card">
    Card 3
  </article>

</section>
```

CSS:

```css
.cards {
  display: grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(250px, 1fr)
    );

  gap:
    clamp(1rem, 2vw, 2rem);
}
```

Card:

```css
.card {
  padding:
    clamp(1rem, 2vw, 2rem);

  border: 1px solid #ddd;
  border-radius: 10px;
}
```

이런 방식은 Media Query 없이도 상당한 Responsive Layout을 제공합니다.

---

# 42. Responsive Header 예제

HTML:

```html
<header class="header">

  <div class="logo">
    MySite
  </div>

  <nav class="menu">
    <a href="#">Home</a>
    <a href="#">Products</a>
    <a href="#">About</a>
  </nav>

</header>
```

Desktop CSS:

```css
.header {
  display: flex;

  align-items: center;
  justify-content: space-between;

  padding:
    1rem 2rem;
}
```

Mobile:

```css
@media (max-width: 768px) {

  .header {
    flex-direction: column;
    align-items: stretch;
  }

  .menu {
    flex-direction: column;
  }

}
```

---

# 43. Responsive Sidebar 예제

Desktop:

```css
.layout {
  display: grid;

  grid-template-columns:
    250px minmax(0, 1fr);

  gap: 24px;
}
```

Mobile:

```css
@media (max-width: 768px) {

  .layout {
    grid-template-columns: 1fr;
  }

  .sidebar {
    display: none;
  }

}
```

구조:

```text
Desktop

Sidebar | Main


Mobile

Main
```

---

# 44. Container Query란?

Responsive CSS에서 중요한 현대 기능이 하나 더 있습니다.

Media Query는 주로 **Viewport**를 기준으로 합니다.

```text
Viewport
   ↓
Media Query
```

하지만 Component가 어디에 배치되느냐에 따라 자신의 크기가 달라질 수 있습니다.

이럴 때 부모 Container의 크기를 기준으로 Style을 바꾸고 싶을 수 있습니다.

이것이 **Container Query**입니다.

---

# 45. Media Query vs Container Query

Media Query:

```text
Browser Viewport
       ↓
조건 판단
       ↓
Component Style
```

Container Query:

```text
Component의 Container
       ↓
조건 판단
       ↓
Component Style
```

즉 Component 단위 Responsive Design에 더 적합한 경우가 있습니다.

---

# 46. Container Query 기본 예제

Parent:

```css
.card-wrapper {
  container-type: inline-size;
}
```

Child:

```css
@container (min-width: 500px) {

  .card {
    display: flex;
  }

}
```

Container가 좁으면:

```text
┌───────────────┐
│ Image         │
│ Title         │
│ Description   │
└───────────────┘
```

Container가 넓으면:

```text
┌───────────┬──────────────────┐
│ Image     │ Title            │
│           │ Description      │
└───────────┴──────────────────┘
```

처럼 Component Layout 자체가 변할 수 있습니다.

---

# 47. Responsive Design의 사고 순서

Responsive CSS를 작성할 때 무조건 Media Query부터 작성하지 않는 것이 좋습니다.

다음 순서로 생각합니다.

```text
1
고정 크기를 제거할 수 있는가?
        ↓

2
% / max-width 등으로
유연하게 만들 수 있는가?
        ↓

3
Flexbox wrap으로 해결 가능한가?
        ↓

4
Grid auto-fit / minmax()로
해결 가능한가?
        ↓

5
clamp()로 크기를
유연하게 만들 수 있는가?
        ↓

6
그래도 Layout 구조 변경이 필요한가?
        ↓

7
Media Query 사용
```

즉:

> **Media Query는 Responsive CSS의 유일한 도구가 아니라, 여러 Responsive 도구 중 하나입니다.**

---

# 48. Responsive Design에서 피해야 할 것

다음과 같은 고정 크기를 과도하게 사용하면 반응형 Layout이 어려워집니다.

```css
width: 1200px;
height: 800px;
margin-left: 400px;
```

또한 Position으로 Layout 전체를 맞추는 것도 좋지 않습니다.

```css
.item-a {
  position: absolute;
  left: 300px;
}

.item-b {
  position: absolute;
  left: 600px;
}
```

화면 크기가 변하면 Layout이 쉽게 깨집니다.

---

# 49. 반응형에서 Position을 조심해야 하는 이유

예:

```css
.button {
  position: absolute;

  left: 700px;
}
```

Desktop에서는 맞아 보일 수 있습니다.

하지만 Mobile에서는:

```text
Viewport
┌───────────────┐

                 Button →
```

처럼 화면 밖으로 나갈 수 있습니다.

반응형 Layout에서는 가능하면:

```text
Normal Flow
Flexbox
Grid
```

로 구조를 만들고, Position은 Badge, Overlay, Floating UI 등 필요한 곳에만 사용하는 것이 좋습니다.

---

# 50. Responsive Testing

Responsive CSS는 브라우저 크기만 몇 번 줄여 보는 것으로 끝나면 안 됩니다.

다음 항목을 확인합니다.

```text
좁은 Mobile

넓은 Mobile

Tablet

Laptop

Large Desktop
```

또한 특정 숫자만 확인하는 것이 아니라 **중간 Width**도 계속 확인해야 합니다.

예:

```text
320
375
390
430
600
768
900
1024
1280
1440
1920
```

모든 값을 Breakpoint로 만들라는 의미는 아닙니다.

Layout이 자연스럽게 유지되는지를 확인하는 것입니다.

---

# 51. DevTools Responsive Mode

Browser DevTools의 Device Toolbar를 이용하면 다양한 Viewport를 테스트할 수 있습니다.

확인할 항목:

```text
Viewport Width

Viewport Height

Touch Simulation

Orientation

Device Pixel Ratio

Network 환경
```

실제 Mobile Device에서도 최종 확인하는 것이 좋습니다.

---

# 52. 자주 하는 실수 1

### Breakpoint를 Device 이름으로만 외운다

예:

```text
Mobile = 768px
Tablet = 1024px
Desktop = 1200px
```

처럼 절대 규칙으로 생각하면 안 됩니다.

Breakpoint는 **Layout이 깨지는 지점**을 기준으로 결정하는 것이 좋습니다.

---

# 53. 자주 하는 실수 2

### 모든 Responsive 문제를 Media Query로 해결한다

예:

```css
@media (...) { ... }
@media (...) { ... }
@media (...) { ... }
@media (...) { ... }
```

Media Query가 너무 많아질 수 있습니다.

먼저:

```text
Flexible Width
Flex Wrap
Grid auto-fit
minmax()
clamp()
```

등을 고려합니다.

---

# 54. 자주 하는 실수 3

### Mobile 화면을 Desktop의 축소판으로 만든다

Responsive Design은:

```text
Desktop
   ↓
비율 축소
   ↓
Mobile
```

이 아닙니다.

더 정확하게는:

```text
같은 Content
     ↓
화면 크기에 맞는
다른 Layout 구조
```

입니다.

---

# 55. 자주 하는 실수 4

### 고정 Height를 너무 많이 사용한다

예:

```css
.card {
  height: 300px;
}
```

Mobile에서 Text가 여러 줄이 되면 Overflow가 생길 수 있습니다.

가능하면:

```css
min-height
height: auto
```

를 고려합니다.

---

# 56. 자주 하는 실수 5

### `100vw`를 무조건 Page Width로 사용한다

일부 환경에서 Scrollbar 등을 고려하면 예상치 못한 수평 Overflow가 발생할 수 있습니다.

일반적인 Page Container에서는:

```css
width: 100%;
```

또는:

```css
width: min(90%, 1200px);
```

같은 방식을 먼저 고려할 수 있습니다.

---

# 57. 자주 하는 실수 6

### Font Size를 모두 `vw`로 만든다

```css
font-size: 8vw;
```

만 사용하면 매우 작은 화면에서는 글자가 지나치게 작아지고 큰 화면에서는 너무 커질 수 있습니다.

다음처럼 제한을 두는 것이 더 안전합니다.

```css
font-size:
  clamp(1.5rem, 4vw, 3rem);
```

---

# 58. 반응형 Layout 실전 패턴

전체 Container:

```css
.container {
  width:
    min(90%, 1200px);

  margin-inline: auto;
}
```

Responsive Grid:

```css
.cards {
  display: grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(250px, 1fr)
    );

  gap:
    clamp(1rem, 2vw, 2rem);
}
```

Fluid Typography:

```css
h1 {
  font-size:
    clamp(2rem, 5vw, 4rem);
}
```

Responsive Media:

```css
img {
  max-width: 100%;
  height: auto;
}
```

---

# 59. PART 9 핵심 구조

Responsive CSS의 전체 흐름은 다음처럼 이해할 수 있습니다.

```text
Responsive CSS
│
├── Flexible Size
│    ├── %
│    ├── max-width
│    ├── min()
│    └── clamp()
│
├── Flexible Layout
│    ├── Flexbox
│    └── Grid
│
├── Viewport
│    ├── vw
│    ├── vh
│    └── dvh
│
├── Conditional CSS
│    └── Media Query
│
├── Component Responsive
│    └── Container Query
│
└── Responsive Media
     ├── max-width
     ├── aspect-ratio
     └── object-fit
```

---

# 60. 지금까지의 CSS Layout 전체 흐름

PART 1부터 연결하면:

```text
HTML Element
      ↓
CSS
      ↓
Selector & Cascade
      ↓
CSS Box
      ↓
Box Model
      ↓
Size & Units
      ↓
Display & Normal Flow
      ↓
Position
      ↓
Flexbox
      ↓
Grid
      ↓
Responsive CSS
      ↓
다양한 Viewport에 맞는
최종 Layout
```

이제 학생은 단순히 CSS Property를 외우는 것이 아니라:

```text
Element
   ↓
Box
   ↓
Size
   ↓
Layout
   ↓
Responsive Layout
```

이라는 전체 구조로 CSS를 바라볼 수 있어야 합니다.

---

# 61. PART 9 핵심 정리

Responsive Web Design은:

> **다양한 Viewport와 환경에서도 Content를 읽기 쉽고 사용하기 좋도록 Layout과 Style을 유연하게 변화시키는 설계 방식**

입니다.

기본 Container 패턴:

```css
.container {
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
}
```

Media Query:

```css
@media (min-width: 768px) {
}
```

Responsive Grid:

```css
grid-template-columns:
  repeat(
    auto-fit,
    minmax(250px, 1fr)
  );
```

Fluid Typography:

```css
font-size:
  clamp(2rem, 5vw, 4rem);
```

Responsive Image:

```css
img {
  max-width: 100%;
  height: auto;
}
```

그리고 가장 중요한 사고 순서는:

```text
Flexible Size
      ↓
Flexible Layout
      ↓
Flex / Grid 자동 대응
      ↓
필요하면 Media Query
      ↓
Component 단위라면
Container Query 고려
```

입니다.

# PART 9의 가장 중요한 한 문장

> **반응형 CSS의 핵심은 화면 크기마다 별도의 웹 페이지를 만드는 것이 아니라, 같은 HTML 구조를 기반으로 크기와 Layout 규칙이 환경에 맞게 유연하게 변화하도록 설계하는 것이다.**
