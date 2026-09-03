# PART 10. Practical CSS

## 1. 지금까지 배운 CSS를 실제 UI로 연결하기

지금까지의 흐름은 다음과 같습니다.

```text
PART 1
CSS Introduction

        ↓

PART 2
Selector & Cascade

        ↓

PART 3
Box Model

        ↓

PART 4
Size & Units

        ↓

PART 5
Display & Normal Flow

        ↓

PART 6
Position

        ↓

PART 7
Flexbox

        ↓

PART 8
Grid

        ↓

PART 9
Responsive CSS
```

이제 마지막 PART에서는 이 개념들을 실제 웹 UI에 조합합니다.

핵심은 다음입니다.

```text
HTML Structure
      ↓
Box Model
      ↓
Typography / Color
      ↓
Layout
      ↓
Spacing
      ↓
Visual Style
      ↓
Responsive
      ↓
Interaction
```

---

# 2. Practical CSS의 목표

이번 PART의 목표는 CSS Property를 더 많이 외우는 것이 아닙니다.

다음과 같은 실제 UI를 만들 수 있어야 합니다.

```text
Header
Navigation
Button
Card
Form
Badge
Modal
Sidebar
Dashboard
Footer
Responsive Page
```

그리고 각 UI를 만들 때:

```text
어떤 HTML 구조가 필요한가?
        ↓
어떤 Box Model이 필요한가?
        ↓
Flexbox인가 Grid인가?
        ↓
Spacing은 어떻게 줄 것인가?
        ↓
Typography는 어떻게 정할 것인가?
        ↓
Hover/Focus는 어떻게 처리할 것인가?
```

를 스스로 판단할 수 있어야 합니다.

---

# 3. CSS 디자인의 기본 구성 요소

실전 UI Style은 크게 다음 요소들의 조합입니다.

```text
Visual Style
│
├── Typography
├── Color
├── Background
├── Border
├── Border Radius
├── Shadow
├── Spacing
├── Size
└── Interaction
```

예를 들어 하나의 Card는:

```css
.card {
  padding: 24px;

  background: white;

  border: 1px solid #ddd;
  border-radius: 12px;

  box-shadow:
    0 4px 12px rgba(0, 0, 0, 0.08);
}
```

처럼 구성할 수 있습니다.

---

# 4. Typography

Typography는 단순히 Font Size만 의미하지 않습니다.

대표적인 Property:

```css
font-family
font-size
font-weight
font-style
line-height
letter-spacing
text-align
text-decoration
```

예:

```css
body {
  font-family:
    Arial,
    sans-serif;

  font-size: 1rem;

  line-height: 1.6;
}
```

---

# 5. `font-family`

글꼴을 지정합니다.

```css
body {
  font-family:
    Arial,
    Helvetica,
    sans-serif;
}
```

브라우저는 앞에서부터 사용할 수 있는 Font를 찾습니다.

```text
Arial
  ↓ 없으면

Helvetica
  ↓ 없으면

sans-serif
```

이러한 구조를 **Font Stack**이라고 합니다.

---

# 6. Generic Font Family

마지막에는 일반적으로 Generic Family를 지정하는 것이 좋습니다.

대표적으로:

```text
serif
sans-serif
monospace
cursive
fantasy
```

예:

```css
code {
  font-family:
    Consolas,
    monospace;
}
```

---

# 7. `font-size`

글자 크기입니다.

```css
body {
  font-size: 1rem;
}

h1 {
  font-size: 2.5rem;
}
```

Responsive Typography에서는:

```css
h1 {
  font-size:
    clamp(2rem, 5vw, 4rem);
}
```

처럼 사용할 수 있습니다.

---

# 8. `font-weight`

글자의 굵기를 지정합니다.

```css
.title {
  font-weight: 700;
}
```

대표적인 값:

```text
400
→ Normal

500
→ Medium

600
→ Semi Bold

700
→ Bold
```

사용 가능한 Weight는 실제 Font가 제공하는 범위의 영향을 받습니다.

---

# 9. `line-height`

줄 높이입니다.

```css
p {
  line-height: 1.6;
}
```

텍스트의 가독성에 매우 중요합니다.

```text
line-height가 너무 작음
────────────────────
Text
Text
Text

줄이 너무 붙음


적절한 line-height
────────────────────

Text

Text

Text
```

본문 Text에서는 보통 Font Size보다 충분한 Line Height를 주는 것이 좋습니다.

---

# 10. `letter-spacing`

글자 간 간격입니다.

```css
.title {
  letter-spacing: -0.02em;
}
```

또는:

```css
.label {
  letter-spacing: 0.05em;
}
```

작은 Label이나 Heading에서 디자인 조정에 사용됩니다.

---

# 11. 텍스트 정렬

```css
text-align: left;
text-align: center;
text-align: right;
```

예:

```css
.hero {
  text-align: center;
}
```

이는 주로 Inline Content의 정렬에 영향을 줍니다.

Flexbox의:

```css
justify-content
```

와 혼동하면 안 됩니다.

---

# 12. Color

CSS에서는 다양한 방식으로 색상을 표현할 수 있습니다.

```css
color: red;

color: #ff0000;

color: rgb(255, 0, 0);

color: rgba(255, 0, 0, 0.5);

color: hsl(0 100% 50%);
```

실전 프로젝트에서는 Design System에 맞는 Color Token을 정의하는 경우가 많습니다.

---

# 13. CSS Custom Properties

색상이나 Spacing 같은 값을 반복해서 사용한다면 **CSS Custom Property**를 활용할 수 있습니다.

```css
:root {
  --color-primary: #2563eb;
  --color-text: #1f2937;
  --color-border: #e5e7eb;

  --radius-md: 8px;

  --space-md: 16px;
}
```

사용:

```css
.button {
  background:
    var(--color-primary);

  border-radius:
    var(--radius-md);

  padding:
    var(--space-md);
}
```

---

# 14. CSS Custom Property의 장점

다음처럼 같은 색상이 여러 곳에 있다고 하겠습니다.

```css
.button {
  background: #2563eb;
}

.link {
  color: #2563eb;
}

.badge {
  border-color: #2563eb;
}
```

변경하려면 여러 곳을 수정해야 합니다.

Custom Property를 사용하면:

```css
:root {
  --primary: #2563eb;
}
```

한 곳에서 관리할 수 있습니다.

```text
Design Token
     ↓
CSS Variable
     ↓
Button
Link
Badge
```

---

# 15. Background

대표적인 Background Property:

```css
background-color
background-image
background-position
background-size
background-repeat
```

예:

```css
.hero {
  background-color:
    #f5f7fa;
}
```

---

# 16. Background Image

```css
.hero {
  background-image:
    url("hero.jpg");
}
```

보통 다음과 함께 사용합니다.

```css
.hero {
  background-image:
    url("hero.jpg");

  background-size: cover;
  background-position: center;
}
```

---

# 17. `background-size: cover`

이미지가 Box를 꽉 채우도록 확대/축소합니다.

```text
Container
┌─────────────────────────┐
│                         │
│      Background         │
│        Image            │
│                         │
└─────────────────────────┘
```

Aspect Ratio를 유지하기 때문에 일부 이미지가 잘릴 수 있습니다.

---

# 18. Gradient

CSS만으로 Gradient를 만들 수 있습니다.

```css
.hero {
  background:
    linear-gradient(
      135deg,
      #2563eb,
      #7c3aed
    );
}
```

또는 이미지와 함께:

```css
.hero {
  background:
    linear-gradient(
      rgba(0, 0, 0, 0.5),
      rgba(0, 0, 0, 0.5)
    ),
    url("hero.jpg") center / cover;
}
```

Overlay 효과에 많이 사용됩니다.

---

# 19. Border

기본:

```css
.card {
  border:
    1px solid #ddd;
}
```

방향별로:

```css
border-top
border-right
border-bottom
border-left
```

를 사용할 수 있습니다.

예:

```css
.header {
  border-bottom:
    1px solid #eee;
}
```

---

# 20. `border-radius`

UI 디자인에서 매우 자주 사용합니다.

```css
.card {
  border-radius: 12px;
}
```

버튼:

```css
.button {
  border-radius: 8px;
}
```

원:

```css
.avatar {
  width: 48px;
  height: 48px;

  border-radius: 50%;
}
```

---

# 21. `box-shadow`

Box에 그림자를 추가합니다.

```css
.card {
  box-shadow:
    0 4px 12px
    rgba(0, 0, 0, 0.08);
}
```

구조:

```text
box-shadow:
x-offset
y-offset
blur
spread
color
```

예:

```css
box-shadow:
  0 4px 12px 0
  rgba(0, 0, 0, 0.08);
```

---

# 22. Shadow는 적게 사용하는 것이 좋다

과도한 Shadow는 UI를 복잡하게 만들 수 있습니다.

```text
Bad

████████████
▓▓ Card ▓▓
████████████


Better

┌─────────────┐
│    Card     │
└─────────────┘
  subtle shadow
```

일반적으로:

```text
Border
또는
약한 Shadow
```

정도로 Layer를 표현하는 것이 좋습니다.

---

# 23. `overflow`

Content가 Box보다 클 때 어떻게 처리할지 결정합니다.

```css
overflow: visible;
overflow: hidden;
overflow: auto;
overflow: scroll;
```

예:

```css
.list {
  max-height: 300px;
  overflow-y: auto;
}
```

---

# 24. Overflow 예제

```text
Container
┌─────────────────┐
│ Item 1          │
│ Item 2          │
│ Item 3          │
│ Item 4          │
│ Item 5          │ ↑
│ Item 6          │ │ Scroll
└─────────────────┘ ↓
```

CSS:

```css
.list {
  max-height: 300px;
  overflow-y: auto;
}
```

---

# 25. `overflow: hidden`

Box 밖으로 나가는 부분을 잘라냅니다.

```css
.thumbnail {
  overflow: hidden;
  border-radius: 12px;
}
```

이미지 확대 효과에서도 많이 사용됩니다.

```css
.thumbnail img {
  transition:
    transform 0.3s;
}

.thumbnail:hover img {
  transform:
    scale(1.05);
}
```

---

# 26. `object-fit`

이미지나 Video 같은 Replaced Element가 지정된 Box 안에서 어떻게 배치될지 결정합니다.

```css
img {
  width: 100%;
  height: 100%;

  object-fit: cover;
}
```

대표 값:

```text
cover
→ Box를 채움
→ 일부 잘릴 수 있음

contain
→ 전체 이미지 표시
→ 빈 공간 생길 수 있음
```

---

# 27. `aspect-ratio`

반응형 이미지 Box에서 매우 유용합니다.

```css
.thumbnail {
  aspect-ratio: 16 / 9;
}
```

```text
┌─────────────────────────┐
│                         │
│          16 : 9         │
│                         │
└─────────────────────────┘
```

---

# 28. `transform`

Element를 시각적으로 변형합니다.

대표적인 함수:

```css
translate()
scale()
rotate()
skew()
```

예:

```css
.card:hover {
  transform:
    translateY(-4px);
}
```

---

# 29. `translate`

```css
transform:
  translateX(20px);
```

```text
원래 위치
[Box]

     → 20px

     [Box]
```

Layout에서 주변 Element의 위치를 다시 계산하는 Margin 이동과는 성격이 다릅니다.

Transform은 주로 **시각적 변형**에 사용됩니다.

---

# 30. `scale`

```css
transform:
  scale(1.05);
```

```text
Before

┌─────────┐
│  Card   │
└─────────┘


After

┌───────────┐
│   Card    │
│           │
└───────────┘
```

Hover Effect에서 자주 사용합니다.

---

# 31. `rotate`

```css
transform:
  rotate(5deg);
```

Icon이나 Decoration 등에 사용할 수 있습니다.

```text
Before

□

After

◇
```

---

# 32. Transition

State가 바뀔 때 Style 변화가 즉시 일어나지 않고 부드럽게 이어지도록 합니다.

예:

```css
.button {
  background:
    #2563eb;

  transition:
    background-color 0.2s;
}

.button:hover {
  background:
    #1d4ed8;
}
```

---

# 33. 여러 Property Transition

```css
.card {
  transition:
    transform 0.2s,
    box-shadow 0.2s;
}
```

Hover:

```css
.card:hover {
  transform:
    translateY(-4px);

  box-shadow:
    0 8px 20px
    rgba(0, 0, 0, 0.12);
}
```

---

# 34. Transition 기본 구조

```text
transition
│
├── property
├── duration
├── timing-function
└── delay
```

예:

```css
transition:
  transform
  0.3s
  ease
  0s;
```

---

# 35. `transition: all`은 신중하게

다음처럼 작성할 수도 있습니다.

```css
transition:
  all 0.3s;
```

하지만 모든 Property에 Transition이 걸릴 수 있어 예상하지 못한 효과가 생길 수 있습니다.

가능하면:

```css
transition:
  background-color 0.2s,
  transform 0.2s;
```

처럼 필요한 Property를 명시하는 것이 좋습니다.

---

# 36. Button 만들기

HTML:

```html
<button class="button">
  저장
</button>
```

CSS:

```css
.button {
  padding:
    10px 18px;

  border: 0;
  border-radius: 8px;

  background:
    #2563eb;

  color: white;

  font-weight: 600;

  cursor: pointer;

  transition:
    background-color 0.2s,
    transform 0.2s;
}
```

Hover:

```css
.button:hover {
  background:
    #1d4ed8;

  transform:
    translateY(-1px);
}
```

---

# 37. Button Focus

Keyboard User를 위해 Focus Style도 중요합니다.

```css
.button:focus-visible {
  outline:
    3px solid
    rgba(37, 99, 235, 0.4);

  outline-offset:
    2px;
}
```

Focus Outline을 무조건 제거하는 것은 좋지 않습니다.

```css
outline: none;
```

만 사용하면 Keyboard 사용자가 현재 Focus 위치를 알기 어려울 수 있습니다.

---

# 38. Input 만들기

HTML:

```html
<input
  class="input"
  type="text"
  placeholder="이름"
/>
```

CSS:

```css
.input {
  width: 100%;

  padding:
    12px 14px;

  border:
    1px solid #ccc;

  border-radius:
    8px;

  font: inherit;
}
```

Focus:

```css
.input:focus {
  border-color:
    #2563eb;

  outline:
    3px solid
    rgba(37, 99, 235, 0.15);
}
```

---

# 39. Card 만들기

HTML:

```html
<article class="card">

  <img
    src="keyboard.jpg"
    alt="Keyboard"
  >

  <div class="card-body">

    <h2>
      Keyboard
    </h2>

    <p>
      Mechanical Keyboard
    </p>

    <strong>
      50,000원
    </strong>

  </div>

</article>
```

---

# 40. Card CSS

```css
.card {
  overflow: hidden;

  background:
    white;

  border:
    1px solid #e5e7eb;

  border-radius:
    12px;

  transition:
    transform 0.2s,
    box-shadow 0.2s;
}
```

Hover:

```css
.card:hover {
  transform:
    translateY(-4px);

  box-shadow:
    0 8px 24px
    rgba(0, 0, 0, 0.1);
}
```

---

# 41. Card Image

```css
.card img {
  width: 100%;

  aspect-ratio:
    4 / 3;

  object-fit:
    cover;
}
```

Card Body:

```css
.card-body {
  padding:
    20px;
}
```

---

# 42. Badge 만들기

HTML:

```html
<span class="badge">
  NEW
</span>
```

CSS:

```css
.badge {
  display:
    inline-flex;

  align-items:
    center;

  padding:
    4px 8px;

  border-radius:
    999px;

  background:
    #eef2ff;

  color:
    #4338ca;

  font-size:
    0.75rem;

  font-weight:
    600;
}
```

---

# 43. 왜 `border-radius: 999px`인가?

높이가 변해도 양쪽 끝을 충분히 둥글게 만들기 위한 실전 패턴입니다.

```text
[ NEW ]
```

Pill 형태의 Badge, Chip, Tag 등에 자주 사용합니다.

---

# 44. Navigation 만들기

HTML:

```html
<header class="header">

  <a
    href="#"
    class="logo"
  >
    MySite
  </a>

  <nav class="nav">

    <a href="#">
      Home
    </a>

    <a href="#">
      Products
    </a>

    <a href="#">
      About
    </a>

  </nav>

  <button class="button">
    Login
  </button>

</header>
```

---

# 45. Navigation CSS

```css
.header {
  display:
    flex;

  align-items:
    center;

  gap:
    32px;

  padding:
    16px 24px;

  border-bottom:
    1px solid #eee;
}
```

Menu:

```css
.nav {
  display:
    flex;

  gap:
    24px;
}
```

Login Button을 오른쪽으로:

```css
.header .button {
  margin-left:
    auto;
}
```

---

# 46. Responsive Navigation

Mobile:

```css
@media (max-width: 768px) {

  .header {
    flex-wrap:
      wrap;
  }

  .nav {
    order:
      3;

    width:
      100%;
  }

}
```

더 복잡한 Mobile Navigation은 JavaScript나 React State와 연결할 수 있습니다.

---

# 47. Page Layout 만들기

HTML:

```html
<div class="page">

  <header class="header">
    Header
  </header>

  <aside class="sidebar">
    Sidebar
  </aside>

  <main class="main">
    Main
  </main>

  <footer class="footer">
    Footer
  </footer>

</div>
```

Grid:

```css
.page {
  min-height:
    100dvh;

  display:
    grid;

  grid-template-columns:
    240px minmax(0, 1fr);

  grid-template-rows:
    auto 1fr auto;

  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}
```

---

# 48. Grid Areas

```css
.header {
  grid-area:
    header;
}

.sidebar {
  grid-area:
    sidebar;
}

.main {
  grid-area:
    main;

  min-width:
    0;
}

.footer {
  grid-area:
    footer;
}
```

결과:

```text
┌──────────────────────────────┐
│ Header                       │
├──────────┬───────────────────┤
│ Sidebar  │ Main              │
│          │                   │
├──────────┴───────────────────┤
│ Footer                       │
└──────────────────────────────┘
```

---

# 49. Main 안의 Card Grid

```css
.cards {
  display:
    grid;

  grid-template-columns:
    repeat(
      auto-fit,
      minmax(240px, 1fr)
    );

  gap:
    24px;
}
```

따라서:

```text
Page Grid
   ↓
Main
   ↓
Card Grid
```

처럼 Grid를 중첩할 수 있습니다.

---

# 50. Responsive Page

```css
@media (max-width: 768px) {

  .page {
    grid-template-columns:
      1fr;

    grid-template-areas:
      "header"
      "main"
      "footer";
  }

  .sidebar {
    display:
      none;
  }

}
```

Desktop:

```text
Sidebar | Main
```

Mobile:

```text
Main
```

으로 Layout이 변합니다.

---

# 51. Modal 기본 구조

HTML:

```html
<div class="modal-backdrop">

  <div class="modal">
    Modal Content
  </div>

</div>
```

CSS:

```css
.modal-backdrop {
  position:
    fixed;

  inset:
    0;

  display:
    grid;

  place-items:
    center;

  background:
    rgba(0, 0, 0, 0.5);
}
```

---

# 52. Modal Box

```css
.modal {
  width:
    min(90%, 500px);

  padding:
    24px;

  border-radius:
    12px;

  background:
    white;
}
```

여기에는 지금까지 배운 개념이 모두 들어 있습니다.

```text
position: fixed
        ↓
Viewport Overlay

inset: 0
        ↓
화면 전체 덮기

display: grid
place-items: center
        ↓
Modal 중앙 정렬

width: min()
        ↓
Responsive Width
```

---

# 53. Tooltip

Tooltip은 Position의 대표적인 실전 사용 사례입니다.

HTML:

```html
<div class="tooltip">

  <button>
    ?
  </button>

  <div class="tooltip-content">
    도움말입니다.
  </div>

</div>
```

CSS:

```css
.tooltip {
  position:
    relative;
}
```

```css
.tooltip-content {
  position:
    absolute;

  bottom:
    calc(100% + 8px);

  left:
    50%;

  transform:
    translateX(-50%);
}
```

---

# 54. CSS `calc()`

서로 다른 Unit이나 값을 계산할 수 있습니다.

```css
width:
  calc(100% - 40px);
```

예:

```css
height:
  calc(100dvh - 64px);
```

Header가 64px일 때 남는 Viewport 높이를 계산할 수 있습니다.

---

# 55. `calc()` 예제

```text
Viewport
100dvh

      -

Header
64px

      ↓

Main
calc(100dvh - 64px)
```

단, Layout이 Flex/Grid로 자연스럽게 해결된다면 `calc()`보다 Layout System을 사용하는 것이 더 좋은 경우가 많습니다.

---

# 56. CSS Naming

Class 이름은 의미를 전달해야 합니다.

좋은 예:

```html
<div class="product-card">
```

```html
<button class="login-button">
```

나쁜 예:

```html
<div class="box1">
```

```html
<div class="red-big">
```

Style 자체보다 **역할과 Component 의미**를 나타내는 이름이 유지보수에 유리합니다.

---

# 57. Component 단위 CSS

예:

```text
ProductCard
│
├── product-card
├── product-card__image
├── product-card__title
├── product-card__price
└── product-card__button
```

프로젝트에서는 CSS Module, Tailwind CSS, CSS-in-JS 등 다양한 방식으로 Scope를 관리할 수도 있습니다.

하지만 어떤 방식을 사용하더라도:

> **Component 단위로 Style 책임을 나눈다**

는 생각은 중요합니다.

---

# 58. CSS Reset / Normalize 개념

브라우저에는 User Agent Stylesheet가 있습니다.

예:

```text
h1 기본 margin

button 기본 font

body 기본 margin
```

때문에 Browser마다 차이가 보일 수 있습니다.

간단한 Reset 예:

```css
*,
*::before,
*::after {
  box-sizing:
    border-box;
}

body {
  margin:
    0;
}
```

---

# 59. 기본 Global CSS 예

```css
*,
*::before,
*::after {
  box-sizing:
    border-box;
}

html {
  font-family:
    system-ui,
    sans-serif;
}

body {
  margin:
    0;

  color:
    #1f2937;

  background:
    #f8fafc;

  line-height:
    1.6;
}

img {
  display:
    block;

  max-width:
    100%;
}
```

이런 코드가 프로젝트 CSS의 기본 출발점이 될 수 있습니다.

---

# 60. CSS 작성 순서

하나의 Component를 만들 때 다음 순서를 추천합니다.

```text
1
Structure
HTML

     ↓

2
Layout
display / flex / grid

     ↓

3
Size
width / min / max

     ↓

4
Spacing
padding / gap / margin

     ↓

5
Typography

     ↓

6
Color / Border / Background

     ↓

7
Interaction
hover / focus

     ↓

8
Responsive
```

이 순서는 CSS를 디버깅할 때도 유용합니다.

---

# 61. CSS Debugging 순서

화면이 이상할 때 무작정 Property를 추가하지 않습니다.

먼저:

```text
1
Element가 제대로 선택되었는가?
        ↓

2
Cascade에 의해 덮어쓰이지 않았는가?
        ↓

3
Box Model 크기는 정상인가?
        ↓

4
display는 무엇인가?
        ↓

5
Normal Flow인가?
        ↓

6
Position 영향이 있는가?
        ↓

7
Flex/Grid Container와 Item 관계는 맞는가?
        ↓

8
Overflow가 발생하는가?
        ↓

9
Breakpoint / Media Query 문제인가?
```

이 순서로 확인합니다.

---

# 62. DevTools 활용

CSS 문제를 해결할 때 Browser DevTools가 매우 중요합니다.

확인할 항목:

```text
Elements

Styles

Computed

Box Model

Layout

Grid Overlay

Flex Overlay

Media Query
```

특히:

```text
어떤 Rule이 적용되는가?

어떤 Rule이 취소선인가?

Computed width는 얼마인가?

Margin/Padding은 얼마인가?

Flex/Grid Container인가?
```

를 확인합니다.

---

# 63. 실전 UI 제작 사고 순서

예를 들어 다음 UI를 만든다고 하겠습니다.

```text
┌───────────────────────────────┐
│ Product                       │
│                               │
│ ┌─────────────────────────┐   │
│ │        Image            │   │
│ └─────────────────────────┘   │
│                               │
│ Keyboard                      │
│ 50,000원                      │
│                               │
│ [ 구매 ]                      │
└───────────────────────────────┘
```

사고 순서:

```text
HTML 구조
     ↓
Card Box
     ↓
Padding
     ↓
Image Ratio
     ↓
Typography
     ↓
Button
     ↓
Hover
     ↓
Responsive Width
```

입니다.

---

# 64. Final Project 구조

PART 10 마지막 실습으로 다음 Page를 만들 수 있습니다.

```text
┌─────────────────────────────────────┐
│ Header                              │
│ Logo       Navigation       Login   │
├──────────┬──────────────────────────┤
│ Sidebar  │ Main                     │
│          │                          │
│ Category │ Product Grid             │
│ Menu     │                          │
│          │ Card Card Card           │
│          │ Card Card Card           │
│          │                          │
├──────────┴──────────────────────────┤
│ Footer                              │
└─────────────────────────────────────┘
```

---

# 65. Final Project에 사용되는 기술

이 Page 하나에 지금까지 배운 CSS가 거의 모두 들어갑니다.

```text
Selector
      ↓
Class 기반 Style

Box Model
      ↓
Card / Container

Units
      ↓
rem / % / minmax

Display
      ↓
block / flex / grid

Position
      ↓
Badge / Overlay

Flexbox
      ↓
Header Navigation

Grid
      ↓
Page Layout
Product Grid

Responsive CSS
      ↓
Mobile Layout

Practical CSS
      ↓
Typography
Color
Border
Shadow
Transition
```

---

# 66. Desktop Layout

```text
Desktop

┌─────────────────────────────────────┐
│ Header                              │
├───────────┬─────────────────────────┤
│ Sidebar   │ Main                    │
│           │                         │
│           │ [Card][Card][Card]      │
│           │ [Card][Card][Card]      │
│           │                         │
├───────────┴─────────────────────────┤
│ Footer                              │
└─────────────────────────────────────┘
```

Grid:

```css
.page {
  display:
    grid;

  grid-template-columns:
    240px minmax(0, 1fr);

  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}
```

---

# 67. Mobile Layout

```text
Mobile

┌──────────────────┐
│ Header           │
├──────────────────┤
│ Main             │
│                  │
│ Card             │
│ Card             │
│ Card             │
├──────────────────┤
│ Footer           │
└──────────────────┘
```

Media Query:

```css
@media (max-width: 768px) {

  .page {
    grid-template-columns:
      1fr;

    grid-template-areas:
      "header"
      "main"
      "footer";
  }

  .sidebar {
    display:
      none;
  }

}
```

---

# 68. Design System의 기초

실전 프로젝트에서는 반복되는 값을 체계화하는 것이 좋습니다.

예:

```css
:root {
  --color-primary:
    #2563eb;

  --color-text:
    #1f2937;

  --color-muted:
    #6b7280;

  --color-border:
    #e5e7eb;

  --radius-sm:
    6px;

  --radius-md:
    10px;

  --radius-lg:
    16px;

  --space-1:
    4px;

  --space-2:
    8px;

  --space-3:
    12px;

  --space-4:
    16px;

  --space-6:
    24px;
}
```

---

# 69. 왜 Design Token이 필요한가?

다음과 같이 여러 Component가 있다고 하겠습니다.

```text
Button
Card
Input
Modal
Badge
```

모두 서로 다른 임의의:

```text
7px
13px
21px
17px
```

를 사용하면 Style 일관성이 떨어집니다.

Token을 사용하면:

```text
4
8
12
16
24
32
```

같은 Scale을 공유할 수 있습니다.

```text
Design Token
      ↓
Consistency
      ↓
Maintainability
```

---

# 70. CSS에서 중요한 것은 "적게 쓰기"가 아니다

좋은 CSS는 단순히 코드 줄 수가 적은 CSS가 아닙니다.

중요한 것은:

```text
의도가 명확한가?

Layout 책임이 명확한가?

중복이 적은가?

Responsive한가?

Override하기 쉬운가?

Component 간 영향이 적은가?
```

입니다.

---

# 71. 자주 하는 실수 1

### Margin으로 모든 Layout을 맞춘다

```css
.button {
  margin-left:
    380px;
}
```

화면 크기가 바뀌면 깨지기 쉽습니다.

먼저:

```text
Flexbox
Grid
auto margin
gap
```

을 고려합니다.

---

# 72. 자주 하는 실수 2

### Position으로 Page Layout을 만든다

```css
.sidebar {
  position:
    absolute;

  left:
    0;
}

.main {
  position:
    absolute;

  left:
    300px;
}
```

Page Layout에는 보통 Grid나 Flexbox가 더 적합합니다.

Position은:

```text
Badge
Overlay
Tooltip
Floating Button
```

처럼 겹침이나 독립 위치가 필요한 경우에 사용합니다.

---

# 73. 자주 하는 실수 3

### 고정 Width/Height를 남발한다

```css
.card {
  width:
    400px;

  height:
    500px;
}
```

반응형 Layout과 Content 변화에 취약합니다.

대신:

```css
.card {
  width:
    100%;

  max-width:
    400px;

  min-height:
    300px;
}
```

처럼 제한과 유연성을 조합할 수 있습니다.

---

# 74. 자주 하는 실수 4

### 모든 간격을 Margin으로 처리한다

Sibling Item 사이의 간격이라면:

```css
gap
```

이 더 적합한 경우가 많습니다.

```css
.menu {
  display:
    flex;

  gap:
    24px;
}
```

Container 내부 공간은:

```css
padding
```

을 사용합니다.

```text
Container 내부
→ padding

Item 사이
→ gap

Component 외부
→ margin
```

로 생각하면 구분하기 쉽습니다.

---

# 75. 자주 하는 실수 5

### Hover만 만들고 Focus를 무시한다

Mouse:

```css
.button:hover {
}
```

뿐 아니라 Keyboard:

```css
.button:focus-visible {
}
```

도 고려해야 합니다.

Interactive UI는:

```text
default
hover
focus
active
disabled
```

상태를 함께 설계하는 것이 좋습니다.

---

# 76. 자주 하는 실수 6

### `transition: all`을 모든 곳에 사용한다

필요한 Property만 지정하는 것이 더 예측 가능합니다.

```css
transition:
  transform 0.2s,
  background-color 0.2s;
```

---

# 77. 자주 하는 실수 7

### Shadow와 Border Radius를 과도하게 사용한다

모든 Box에:

```text
강한 Shadow
큰 Radius
Gradient
```

를 넣으면 정보 구조가 오히려 흐려집니다.

Visual Style은 Content 구조를 보조해야 합니다.

---

# 78. CSS 전체 사고 모델

이제 CSS 전체를 하나의 흐름으로 연결할 수 있습니다.

```text
HTML Element
      ↓
Selector Matching
      ↓
Cascade
      ↓
Computed Style
      ↓
CSS Box
      ↓
Box Model
      ↓
Size
      ↓
Display
      ↓
Formatting Context
      ↓
Layout
      │
      ├── Normal Flow
      ├── Position
      ├── Flexbox
      └── Grid
      ↓
Responsive Rules
      ↓
Paint
      ↓
Visual Effects
      ↓
Screen
```

---

# 79. CSS를 배웠다는 것은 무엇인가?

CSS Property를 많이 암기하는 것이 목표가 아닙니다.

다음 질문에 답할 수 있어야 합니다.

```text
어떤 Element가 선택되었는가?

왜 이 Style이 최종 적용되었는가?

이 Element는 어떤 Box를 만드는가?

실제 크기는 얼마인가?

왜 이 위치에 배치되었는가?

누가 Layout Container인가?

Flexbox와 Grid 중 무엇이 적합한가?

왜 Overflow가 생겼는가?

화면이 작아지면 어떻게 변해야 하는가?
```

이 질문에 답할 수 있다면 CSS를 구조적으로 이해하고 있는 것입니다.

---

# 80. PART 1 ~ PART 10 전체 구조

```text
PART 1
CSS Introduction
CSS는 무엇인가?

        ↓

PART 2
Selector & Cascade
어디에 어떤 Style이 적용되는가?

        ↓

PART 3
Box Model
Element는 어떤 Box를 만드는가?

        ↓

PART 4
Size & Units
Box의 크기를 어떻게 표현하는가?

        ↓

PART 5
Display & Normal Flow
Box가 기본적으로 어떻게 배치되는가?

        ↓

PART 6
Position
특정 Box의 위치를 어떻게 제어하는가?

        ↓

PART 7
Flexbox
한 축을 중심으로 Item들을 어떻게 배치하는가?

        ↓

PART 8
Grid
Row와 Column으로 Layout을 어떻게 설계하는가?

        ↓

PART 9
Responsive CSS
화면 크기에 따라 Layout을 어떻게 변화시키는가?

        ↓

PART 10
Practical CSS
이 모든 것을 실제 UI에 어떻게 조합하는가?
```

---

# 81. PART 10 핵심 정리

실전 CSS는 새로운 Property를 많이 사용하는 것이 아니라 지금까지 배운 개념을 목적에 맞게 조합하는 것입니다.

```text
Structure
HTML

   ↓

Style
Typography / Color

   ↓

Box
Padding / Border / Radius

   ↓

Layout
Flexbox / Grid

   ↓

Position
Overlay / Badge

   ↓

Responsive
Media Query / Fluid CSS

   ↓

Interaction
Hover / Focus / Transition
```

대표적인 실전 Property:

```css
font-size
font-weight
line-height

background
border
border-radius

box-shadow

overflow
object-fit
aspect-ratio

transform
transition

var()
calc()
```

그리고 가장 중요한 기준은:

```text
Layout 문제
→ Flexbox / Grid

위치 겹침 문제
→ Position

간격 문제
→ Padding / Gap / Margin

크기 문제
→ width / min / max

반응형 문제
→ Flexible Layout
   + Media Query

Interaction 문제
→ Pseudo-class
   + Transition
```

입니다.

# PART 10의 가장 중요한 한 문장

> **실전 CSS의 핵심은 Property를 많이 사용하는 것이 아니라, Element → Box → Size → Layout → Responsive → Interaction이라는 순서로 문제를 분석하고 가장 적합한 CSS 기능을 선택하는 것이다.**
