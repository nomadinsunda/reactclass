# PART 7. CSS Flexbox

## 1. 왜 Flexbox가 필요한가?

PART 6에서는 `position`을 이용해 Box의 위치를 조정하는 방법을 배웠습니다.

예를 들어:

```css
.badge {
  position: absolute;
  top: 10px;
  right: 10px;
}
```

이 방법은 다음과 같이 **특정 Box를 특정 위치에 배치**할 때 매우 유용합니다.

```text
Card
┌────────────────────────────┐
│                      SALE  │
│                            │
│         Product            │
│                            │
└────────────────────────────┘
```

하지만 다음과 같은 Layout은 어떨까요?

```text
┌─────────────────────────────────────────┐
│ Logo          Menu              Login   │
└─────────────────────────────────────────┘
```

또는:

```text
┌────────┐ ┌────────┐ ┌────────┐
│ Item 1 │ │ Item 2 │ │ Item 3 │
└────────┘ └────────┘ └────────┘
```

여러 Sibling Element를:

* 가로로 배치하고
* 간격을 조절하고
* 중앙 정렬하고
* 남는 공간을 분배하고
* 화면이 좁아지면 줄바꿈

해야 합니다.

이를 `position: absolute`로 각각 계산하는 것은 적절하지 않습니다.

이러한 문제를 해결하기 위한 CSS Layout System이 **Flexbox**입니다.

---

# 2. Flexbox란?

Flexbox의 정식 명칭은:

```text
CSS Flexible Box Layout
```

입니다.

Flexbox는 **Container 내부의 Item들을 한 방향을 중심으로 배치하고 정렬하며 공간을 분배하기 위한 1차원 Layout System**입니다.

핵심 구조:

```text
Flex Container
┌─────────────────────────────────────┐
│                                     │
│  Item 1      Item 2      Item 3     │
│                                     │
└─────────────────────────────────────┘
```

Flexbox의 핵심은:

> **각 Item의 위치를 직접 지정하는 것이 아니라 Parent인 Flex Container에게 자식들의 배치를 맡기는 것**

입니다.

---

# 3. Position과 Flexbox의 사고방식 차이

Position에서는 주로 특정 Element 자체에 위치를 지정합니다.

```css
.item {
  position: absolute;
  left: 100px;
  top: 50px;
}
```

사고방식:

```text
Item
 ↓
너는 여기 있어
```

Flexbox에서는 Parent에게 Layout 규칙을 지정합니다.

```css
.container {
  display: flex;
}
```

사고방식:

```text
Container
   ↓
내 자식들을
이 규칙으로 배치해
   ↓
Item 1
Item 2
Item 3
```

즉:

```text
Position
→ 개별 Box 위치 제어

Flexbox
→ 여러 Item의 관계와 정렬 제어
```

입니다.

---

# 4. Flex Container 만들기

HTML:

```html
<div class="container">
  <div class="item">A</div>
  <div class="item">B</div>
  <div class="item">C</div>
</div>
```

기본 Block Layout에서는:

```text
┌─────────────────────┐
│ A                   │
└─────────────────────┘
┌─────────────────────┐
│ B                   │
└─────────────────────┘
┌─────────────────────┐
│ C                   │
└─────────────────────┘
```

Parent에:

```css
.container {
  display: flex;
}
```

를 지정하면:

```text
Flex Container
┌─────────────────────────────────┐
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │  A  │ │  B  │ │  C  │       │
│ └─────┘ └─────┘ └─────┘       │
└─────────────────────────────────┘
```

가 됩니다.

---

# 5. Flex Container와 Flex Item

Flexbox에서 반드시 구분해야 하는 두 개념입니다.

```text
Flex Container
│
├── Flex Item
├── Flex Item
└── Flex Item
```

HTML:

```html
<div class="container">
  <div>A</div>
  <div>B</div>
  <div>C</div>
</div>
```

CSS:

```css
.container {
  display: flex;
}
```

그러면:

```text
.container
→ Flex Container

A
B
C
→ Flex Items
```

가 됩니다.

즉 **`display: flex`는 자식에게 지정하는 것이 아니라 Container에 지정합니다.**

---

# 6. 직계 자식이 Flex Item이 된다

이 부분이 중요합니다.

```html
<div class="container">

  <div class="item">
    <span>A</span>
  </div>

  <div class="item">
    <span>B</span>
  </div>

</div>
```

```css
.container {
  display: flex;
}
```

구조:

```text
.container
Flex Container
│
├── .item
│    └── span
│
└── .item
     └── span
```

여기서 직접 Flex Item이 되는 것은 `.container`의 **직계 자식 `.item`들**입니다.

내부의 `<span>`이 자동으로 `.container`의 Flex Item이 되는 것은 아닙니다.

```text
Flex Container
      ↓
직계 자식
      ↓
Flex Item
```

---

# 7. Flexbox의 가장 중요한 개념 — Axis

Flexbox를 제대로 이해하려면 가장 먼저 **축(Axis)**을 이해해야 합니다.

Flexbox에는 두 개의 축이 있습니다.

```text
Main Axis
Cross Axis
```

기본 상태:

```css
.container {
  display: flex;
}
```

에서는:

```text
Main Axis
──────────────────────────────→

┌───────────────────────────────┐
│                               │
│  A       B       C            │
│                               │
└───────────────────────────────┘
               │
               │
               ▼
          Cross Axis
```

즉 기본적으로:

```text
Main Axis
→ 가로

Cross Axis
→ 세로
```

입니다.

하지만 항상 그런 것은 아닙니다.

---

# 8. `flex-direction`

Main Axis의 방향을 결정하는 Property입니다.

```css
flex-direction: row;
flex-direction: row-reverse;
flex-direction: column;
flex-direction: column-reverse;
```

디폴트 값:

```css
flex-direction: row;
```

입니다.

---

# 9. `flex-direction: row`

```css
.container {
  display: flex;
  flex-direction: row;
}
```

Main Axis:

```text
Main Axis
──────────────────────────→

[A] [B] [C]
```

Cross Axis:

```text
↓
```

즉:

```text
Main Axis  → 가로
Cross Axis → 세로
```

입니다.

---

# 10. `flex-direction: column`

```css
.container {
  display: flex;
  flex-direction: column;
}
```

이제 Main Axis가 바뀝니다.

```text
Main Axis
     ↓

[A]
[B]
[C]
```

Cross Axis는:

```text
────────────→
```

가 됩니다.

즉:

```text
Main Axis  → 세로
Cross Axis → 가로
```

입니다.

따라서 다음처럼 외우면 안 됩니다.

```text
justify-content
→ 가로 정렬

align-items
→ 세로 정렬
```

이 설명은 `flex-direction: row`일 때만 맞습니다.

정확하게는:

```text
justify-content
→ Main Axis 정렬

align-items
→ Cross Axis 정렬
```

입니다.

---

# 11. Main Axis와 Cross Axis

Flexbox 전체를 이해하는 핵심입니다.

```text
flex-direction: row

Main Axis
──────────────────────→

[A] [B] [C]

Cross Axis
     ↓
```

반면:

```text
flex-direction: column

Cross Axis
──────────────────────→

[A]
 ↓
[B]   Main Axis
 ↓
[C]
```

따라서:

> **Flexbox에서는 먼저 Main Axis가 어디인지 확인해야 합니다.**

---

# 12. `justify-content`

`justify-content`는 **Main Axis 방향의 남는 공간을 어떻게 분배할 것인지** 결정합니다.

대표적인 값:

```css
justify-content: flex-start;
justify-content: flex-end;
justify-content: center;

justify-content: space-between;
justify-content: space-around;
justify-content: space-evenly;
```

---

# 13. `justify-content: flex-start`

```css
.container {
  display: flex;
  justify-content: flex-start;
}
```

기본적인 배치:

```text
Main Axis →

┌──────────────────────────────────┐
│ [A] [B] [C]                     │
└──────────────────────────────────┘
```

Item들이 Main Axis 시작점에 모입니다.

---

# 14. `justify-content: flex-end`

```text
┌──────────────────────────────────┐
│                     [A] [B] [C] │
└──────────────────────────────────┘
```

Main Axis 끝에 배치됩니다.

---

# 15. `justify-content: center`

```text
┌──────────────────────────────────┐
│          [A] [B] [C]            │
└──────────────────────────────────┘
```

Main Axis 중앙에 배치됩니다.

---

# 16. `space-between`

```css
justify-content: space-between;
```

```text
┌──────────────────────────────────┐
│ [A]          [B]          [C]   │
└──────────────────────────────────┘
```

첫 Item과 마지막 Item은 양 끝에 배치되고 Item 사이에 남는 공간이 분배됩니다.

---

# 17. `space-around`

```css
justify-content: space-around;
```

각 Item 주변에 동일한 몫의 공간이 배정됩니다.

```text
┌──────────────────────────────────┐
│  [A]       [B]       [C]        │
└──────────────────────────────────┘
```

Item 사이 공간은 양 끝 공간보다 크게 보입니다.

---

# 18. `space-evenly`

```css
justify-content: space-evenly;
```

```text
┌──────────────────────────────────┐
│   [A]     [B]     [C]           │
└──────────────────────────────────┘
```

양 끝과 Item 사이 간격이 동일하게 분배됩니다.

---

# 19. `align-items`

`align-items`는 Flex Items를 **Cross Axis 방향으로 정렬**합니다.

대표적인 값:

```css
align-items: stretch;
align-items: flex-start;
align-items: flex-end;
align-items: center;
align-items: baseline;
```

디폴트 값은 일반적으로:

```css
align-items: stretch;
```

입니다.

---

# 20. `align-items: flex-start`

`flex-direction: row`라고 가정하면:

```text
Cross Axis
↓

┌──────────────────────────────┐
│ [A] [B] [C]                 │
│                              │
│                              │
└──────────────────────────────┘
```

위쪽에 정렬됩니다.

---

# 21. `align-items: flex-end`

```text
┌──────────────────────────────┐
│                              │
│                              │
│ [A] [B] [C]                 │
└──────────────────────────────┘
```

Cross Axis 끝에 배치됩니다.

---

# 22. `align-items: center`

```text
┌──────────────────────────────┐
│                              │
│       [A] [B] [C]           │
│                              │
└──────────────────────────────┘
```

Cross Axis 중앙 정렬입니다.

---

# 23. Flexbox 중앙 정렬

Flexbox에서 가장 유명한 패턴입니다.

```css
.container {
  display: flex;

  justify-content: center;
  align-items: center;
}
```

결과:

```text
┌───────────────────────────────┐
│                               │
│                               │
│             [A]               │
│                               │
│                               │
└───────────────────────────────┘
```

왜 중앙에 오는지 이해해야 합니다.

```text
justify-content: center
→ Main Axis 중앙

align-items: center
→ Cross Axis 중앙
```

두 축 모두 중앙이므로 완전 중앙 정렬이 됩니다.

---

# 24. `gap`

Flex Items 사이의 간격을 지정합니다.

```css
.container {
  display: flex;
  gap: 20px;
}
```

결과:

```text
[A] ←20px→ [B] ←20px→ [C]
```

예전에는 Item마다 Margin을 사용하는 경우가 많았습니다.

```css
.item {
  margin-right: 20px;
}
```

하지만 Item 사이 간격이 목적이라면:

```css
gap: 20px;
```

가 훨씬 명확합니다.

---

# 25. `row-gap`과 `column-gap`

따로 지정할 수도 있습니다.

```css
.container {
  row-gap: 20px;
  column-gap: 30px;
}
```

또는:

```css
gap: 20px 30px;
```

```text
gap: row-gap column-gap
```

입니다.

---

# 26. Flex Items가 Container보다 커지면?

예:

```text
Container width = 500px

Item A = 200px
Item B = 200px
Item C = 200px

총 600px
```

Container보다 Item들의 크기가 큽니다.

Flexbox에서는 기본적으로 Flex Item이 **줄어들 수 있습니다.**

이 동작을 제어하는 Property가:

```text
flex-shrink
```

입니다.

하지만 또 다른 방법은 줄바꿈을 허용하는 것입니다.

```text
flex-wrap
```

---

# 27. `flex-wrap`

디폴트:

```css
flex-wrap: nowrap;
```

입니다.

즉 기본적으로:

```text
[A] [B] [C] [D] [E] →
```

한 줄에 배치하려고 합니다.

줄바꿈을 허용하려면:

```css
.container {
  display: flex;
  flex-wrap: wrap;
}
```

를 사용합니다.

결과:

```text
[A] [B] [C]

[D] [E]
```

---

# 28. `flex-flow`

`flex-direction`과 `flex-wrap`을 합친 Shorthand입니다.

```css
.container {
  flex-direction: row;
  flex-wrap: wrap;
}
```

를:

```css
.container {
  flex-flow: row wrap;
}
```

으로 작성할 수 있습니다.

---

# 29. `align-content`

여기서 `align-items`와 혼동하기 쉬운 Property가 등장합니다.

```css
align-content
```

`align-content`는 **여러 Flex Line이 존재할 때 Cross Axis 방향으로 Flex Line들을 정렬**합니다.

즉:

```text
flex-wrap: wrap
```

등으로 여러 줄이 만들어진 상황에서 의미가 있습니다.

```text
Flex Container
┌───────────────────────────┐
│ [A] [B] [C]              │ ← Line 1
│                           │
│ [D] [E] [F]              │ ← Line 2
└───────────────────────────┘
```

---

# 30. `align-items` vs `align-content`

차이를 반드시 구분해야 합니다.

```text
align-items
     ↓
각 Flex Item을
Cross Axis에서 정렬
```

반면:

```text
align-content
     ↓
여러 Flex Line 자체를
Cross Axis에서 정렬
```

즉:

```text
Item 정렬
→ align-items

Line 정렬
→ align-content
```

입니다.

---

# 31. Container Property 정리

지금까지 배운 Property는 모두 **Flex Container에 지정**합니다.

```css
.container {
  display: flex;

  flex-direction: row;

  justify-content: center;
  align-items: center;

  flex-wrap: wrap;

  gap: 20px;
}
```

정리:

```text
Flex Container Properties

display
flex-direction
flex-wrap
flex-flow

justify-content

align-items
align-content

gap
```

---

# 32. 이제 Flex Item을 제어해 보자

Flexbox에는 Container Property만 있는 것이 아닙니다.

각 Flex Item도 자신의 크기와 순서를 제어할 수 있습니다.

대표적인 Property:

```text
order

flex-grow
flex-shrink
flex-basis

flex

align-self
```

---

# 33. `order`

Flex Item의 시각적 배치 순서를 변경합니다.

HTML:

```html
<div class="container">
  <div class="a">A</div>
  <div class="b">B</div>
  <div class="c">C</div>
</div>
```

기본:

```text
A B C
```

CSS:

```css
.a {
  order: 3;
}

.b {
  order: 1;
}

.c {
  order: 2;
}
```

시각적 순서:

```text
B C A
```

가 될 수 있습니다.

다만 `order`는 시각적 순서를 바꾸므로 **DOM 순서와 접근성/키보드 탐색 순서의 관계**를 고려해야 합니다.

---

# 34. `flex-grow`

Container에 남는 공간이 있다고 하겠습니다.

```text
Container
┌──────────────────────────────────────┐
│ [A] [B] [C]          남는 공간      │
└──────────────────────────────────────┘
```

`flex-grow`는:

> **남는 공간을 Flex Item이 얼마나 가져갈 것인가**

를 결정합니다.

---

# 35. `flex-grow` 예제

```css
.a {
  flex-grow: 1;
}

.b {
  flex-grow: 1;
}

.c {
  flex-grow: 1;
}
```

남는 공간을 같은 비율로 나눕니다.

```text
┌──────────┬──────────┬──────────┐
│    A     │    B     │    C     │
└──────────┴──────────┴──────────┘
```

반면:

```css
.a {
  flex-grow: 1;
}

.b {
  flex-grow: 2;
}
```

라면 **남는 공간의 분배 몫**이:

```text
A : B
1 : 2
```

가 됩니다.

주의할 점은 최종 전체 width 자체가 반드시 정확히 1:2가 된다는 뜻은 아니라는 것입니다. `flex-basis` 등의 기본 크기를 먼저 고려한 뒤 **남는 공간**을 해당 비율로 분배합니다.

---

# 36. `flex-shrink`

Container 공간이 부족할 때 Flex Item이 얼마나 줄어들 수 있는지 결정합니다.

```text
필요한 공간
600px

Container
500px

부족
100px
```

이 부족한 공간을 Flex Items가 나누어 줄어듭니다.

디폴트:

```css
flex-shrink: 1;
```

입니다.

줄어들지 않게 하려면:

```css
.item {
  flex-shrink: 0;
}
```

처럼 지정할 수 있습니다.

---

# 37. `flex-basis`

Flex Item의 **Main Axis 방향 초기 크기**를 지정합니다.

```css
.item {
  flex-basis: 200px;
}
```

`flex-direction: row`라면 Main Axis가 가로이므로 너비와 관련됩니다.

```text
Main Axis →

┌───────────────┐
│     Item      │
└───────────────┘
←── 200px ─────→
```

하지만 `flex-direction: column`이면 Main Axis가 세로이므로 높이 방향의 초기 크기와 관련됩니다.

따라서:

> `flex-basis`를 단순히 "width와 같은 것"이라고 외우면 안 됩니다.

정확하게는 **Main Size의 초기 기준값**입니다.

---

# 38. `flex`

가장 많이 사용하는 Flex Item Shorthand입니다.

```css
flex: grow shrink basis;
```

예:

```css
.item {
  flex: 1 1 200px;
}
```

개념적으로:

```text
flex-grow   = 1

flex-shrink = 1

flex-basis  = 200px
```

입니다.

---

# 39. `flex: 1`

실전에서 매우 자주 볼 수 있습니다.

```css
.item {
  flex: 1;
}
```

일반적으로 이는:

```css
flex: 1 1 0%;
```

에 해당하는 Shorthand로 처리됩니다.

여러 Item에:

```css
.item {
  flex: 1;
}
```

을 지정하면 동일한 조건에서 Container의 공간을 균등하게 나누는 Layout을 쉽게 만들 수 있습니다.

```text
Container
┌──────────┬──────────┬──────────┐
│    A     │    B     │    C     │
└──────────┴──────────┴──────────┘
```

---

# 40. `align-self`

Container의:

```css
align-items
```

설정을 특정 Item 하나에서 개별적으로 변경할 수 있습니다.

```css
.container {
  display: flex;
  align-items: flex-start;
}

.b {
  align-self: flex-end;
}
```

결과:

```text
┌───────────────────────────────┐
│ [A]       [C]                │
│                              │
│          [B]                 │
└───────────────────────────────┘
```

즉:

```text
align-items
→ 전체 Flex Items

align-self
→ 특정 Flex Item
```

입니다.

---

# 41. Container와 Item Property 구분

Flexbox에서 가장 중요한 분류입니다.

```text
Flex Container
│
├── display
├── flex-direction
├── flex-wrap
├── justify-content
├── align-items
├── align-content
└── gap
```

Flex Item:

```text
Flex Item
│
├── order
├── flex-grow
├── flex-shrink
├── flex-basis
├── flex
└── align-self
```

초보자는 이 둘을 반드시 구분해야 합니다.

---

# 42. 실전 예제 1 — Navigation

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

  <button>
    Login
  </button>

</header>
```

CSS:

```css
.header {
  display: flex;

  align-items: center;
  justify-content: space-between;

  padding: 16px 24px;
}
```

결과:

```text
┌─────────────────────────────────────────┐
│ MySite      Home Products About   Login │
└─────────────────────────────────────────┘
```

여기서:

```text
justify-content: space-between
→ Main Axis 공간 분배

align-items: center
→ Cross Axis 중앙 정렬
```

입니다.

---

# 43. 실전 예제 2 — 메뉴 내부도 Flexbox

앞의 Navigation에서:

```html
<nav class="menu">
  <a>Home</a>
  <a>Products</a>
  <a>About</a>
</nav>
```

메뉴 Item들도 Flexbox로 배치할 수 있습니다.

```css
.menu {
  display: flex;
  gap: 24px;
}
```

즉 Flex Container 안에 또 다른 Flex Container가 존재할 수 있습니다.

```text
.header
Flex Container
│
├── logo
│
├── menu
│    │
│    └── Flex Container
│         ├── Home
│         ├── Products
│         └── About
│
└── Login
```

Flexbox는 중첩해서 사용할 수 있습니다.

---

# 44. 실전 예제 3 — Card Layout

```html
<div class="cards">

  <article class="card">A</article>
  <article class="card">B</article>
  <article class="card">C</article>

</div>
```

CSS:

```css
.cards {
  display: flex;

  gap: 20px;

  flex-wrap: wrap;
}

.card {
  flex: 1 1 250px;
}
```

의미:

```text
flex-grow: 1
→ 남는 공간 사용 가능

flex-shrink: 1
→ 필요하면 축소 가능

flex-basis: 250px
→ 초기 Main Size 기준
```

화면이 넓으면:

```text
[A] [B] [C]
```

화면이 좁아지면:

```text
[A] [B]

[C]
```

처럼 줄바꿈될 수 있습니다.

---

# 45. 실전 예제 4 — 완전 중앙 정렬

HTML:

```html
<div class="container">
  <div class="login">
    Login
  </div>
</div>
```

CSS:

```css
.container {
  min-height: 100vh;

  display: flex;

  justify-content: center;
  align-items: center;
}
```

결과:

```text
Viewport
┌───────────────────────────────┐
│                               │
│                               │
│           ┌───────┐           │
│           │ Login │           │
│           └───────┘           │
│                               │
│                               │
└───────────────────────────────┘
```

---

# 46. 실전 예제 5 — Sidebar + Main

HTML:

```html
<div class="layout">

  <aside class="sidebar">
    Sidebar
  </aside>

  <main class="main">
    Main Content
  </main>

</div>
```

CSS:

```css
.layout {
  display: flex;
}

.sidebar {
  flex: 0 0 250px;
}

.main {
  flex: 1;
}
```

구조:

```text
┌────────────┬───────────────────────────┐
│            │                           │
│  Sidebar   │       Main Content        │
│   250px    │       남는 공간          │
│            │                           │
└────────────┴───────────────────────────┘
```

---

# 47. `margin-left: auto`와 Flexbox

Flexbox에서 `auto` Margin은 남는 공간을 흡수할 수 있습니다.

HTML:

```html
<header class="header">
  <div>Logo</div>
  <nav>Menu</nav>
  <button>Login</button>
</header>
```

CSS:

```css
.header {
  display: flex;
  align-items: center;
}

.header button {
  margin-left: auto;
}
```

결과:

```text
Logo Menu                     Login
         ← 남는 공간 ───────→
```

매우 유용한 Flexbox 패턴입니다.

---

# 48. Flexbox는 1차원 Layout이다

Flexbox는 흔히 **1차원 Layout System**이라고 합니다.

이 말은:

```text
가로만 가능하다
```

는 뜻이 아닙니다.

Flexbox는:

```text
Main Axis
```

라는 **한 축을 중심으로 Item을 배치하고**, Cross Axis에서 정렬하는 방식이라는 의미입니다.

```text
Main Axis 중심
      ↓
1차원 Layout
```

`row`라면:

```text
가로 중심
```

`column`이라면:

```text
세로 중심
```

입니다.

---

# 49. Flexbox와 Grid의 차이

다음 Layout:

```text
[A] [B] [C] [D]
```

처럼 한 방향을 중심으로 Item을 배치한다면 Flexbox가 매우 적합합니다.

반면:

```text
┌─────┬─────┬─────┐
│  A  │  B  │  C  │
├─────┼─────┼─────┤
│  D  │  E  │  F  │
└─────┴─────┴─────┘
```

처럼 **Row와 Column을 함께 명시적으로 제어**해야 한다면 Grid가 더 적합한 경우가 많습니다.

단순화하면:

```text
Flexbox
→ 1차원 중심

Grid
→ 2차원 중심
```

입니다.

둘은 경쟁 관계가 아니라 서로 보완하는 Layout System입니다.

---

# 50. 자주 하는 실수 1

### `justify-content`는 가로 정렬이다.

항상 그렇지 않습니다.

정확하게는:

```text
justify-content
→ Main Axis
```

입니다.

`flex-direction: column`이면 Main Axis가 세로가 됩니다.

```css
.container {
  display: flex;

  flex-direction: column;

  justify-content: center;
}
```

이 경우 세로 방향 중앙 정렬에 관여합니다.

---

# 51. 자주 하는 실수 2

### `align-items`는 세로 정렬이다.

역시 항상 그렇지 않습니다.

정확하게는:

```text
align-items
→ Cross Axis
```

입니다.

따라서 항상:

```text
Main Axis
Cross Axis
```

부터 확인해야 합니다.

---

# 52. 자주 하는 실수 3

### `flex-grow: 2`이면 width가 두 배이다.

정확하지 않습니다.

`flex-grow`는 **남는 공간을 어떤 비율로 나눌 것인지** 결정합니다.

```text
Item 기본 크기
      +
남는 공간
      ↓
flex-grow 비율로 분배
      ↓
최종 크기
```

따라서 `grow` 값만 보고 최종 Width 비율을 단순 계산하면 안 됩니다.

---

# 53. 자주 하는 실수 4

### Flexbox를 사용하면 모든 자손이 Flex Item이다.

아닙니다.

```text
Flex Container
│
├── Child A ← Flex Item
│    └── Grandchild
│
└── Child B ← Flex Item
     └── Grandchild
```

Flex Container의 **직계 자식**이 Flex Item이 됩니다.

---

# 54. 자주 하는 실수 5

### `align-content`와 `align-items`는 같다.

아닙니다.

```text
align-items
→ Flex Item 정렬

align-content
→ 여러 Flex Line 정렬
```

`align-content`는 여러 Flex Line이 존재해야 의미가 나타납니다.

---

# 55. 자주 하는 실수 6 — `min-width: auto`

Flexbox를 사용하다 보면 다음 문제가 자주 발생합니다.

```text
Sidebar │ Main Content가 너무 길어서
        │ Container 밖으로 넘침 ─────────→
```

Flex Item은 기본적으로 Content의 최소 크기 때문에 예상보다 줄어들지 않을 수 있습니다.

대표적인 해결 패턴:

```css
.main {
  flex: 1;
  min-width: 0;
}
```

특히 긴 문자열, 이미지, `<pre>` 같은 Content가 있는 Flex Item에서 중요합니다.

---

# 56. Flexbox를 사용할 때의 사고 순서

Flexbox Property를 무작정 외우기보다 다음 순서로 생각합니다.

```text
1
누가 Container인가?
        ↓

2
누가 Flex Item인가?
        ↓

3
Main Axis는 어느 방향인가?
        ↓

4
Main Axis에서 어떻게 배치할 것인가?
        ↓
justify-content

5
Cross Axis에서 어떻게 정렬할 것인가?
        ↓
align-items

6
Item 사이 간격은?
        ↓
gap

7
공간이 부족하면 줄바꿈할 것인가?
        ↓
flex-wrap

8
각 Item 크기는 어떻게 분배할 것인가?
        ↓
flex-grow
flex-shrink
flex-basis
```

이 순서만 익혀도 대부분의 Flexbox Layout을 설계할 수 있습니다.

---

# 57. Flexbox 핵심 구조

```text
Flex Container
│
├── Main Axis
│     │
│     ├── flex-direction
│     └── justify-content
│
├── Cross Axis
│     │
│     ├── align-items
│     └── align-content
│
├── 여러 줄
│     │
│     └── flex-wrap
│
├── 간격
│     │
│     └── gap
│
└── Flex Items
      │
      ├── order
      ├── flex-grow
      ├── flex-shrink
      ├── flex-basis
      ├── flex
      └── align-self
```

---

# 58. PART 6과 PART 7의 연결

PART 6:

```text
Position
      ↓
개별 Box의 위치 제어

Badge
Overlay
Floating Button
Sticky Header
```

PART 7:

```text
Flexbox
      ↓
여러 Sibling Item의
배치 관계 제어

정렬
간격
공간 분배
줄바꿈
```

즉:

```text
Position
→ "이 Box를 어디에 둘까?"

Flexbox
→ "이 Item들을 어떻게 배치할까?"
```

라는 차이가 있습니다.

---

# 59. CSS Layout 전체에서 Flexbox의 위치

```text
HTML Element
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
Positioning
      ↓
Flexbox
      ↓
Grid
```

이제 CSS Layout을 바라보는 관점이 다음처럼 확장됩니다.

```text
기본 배치
Normal Flow

       +

개별 위치 제어
Position

       +

1차원 관계형 Layout
Flexbox

       +

2차원 Layout
Grid
```

---

# 60. PART 7 핵심 정리

Flexbox는 **1차원 Layout System**입니다.

```css
.container {
  display: flex;
}
```

를 사용하면 Parent가 Flex Container가 되고 직계 자식들이 Flex Items가 됩니다.

가장 중요한 개념은 두 개의 Axis입니다.

```text
Main Axis
Cross Axis
```

Main Axis 방향:

```css
flex-direction
```

Main Axis 정렬 및 공간 분배:

```css
justify-content
```

Cross Axis 정렬:

```css
align-items
```

Item 간격:

```css
gap
```

줄바꿈:

```css
flex-wrap
```

Flex Item의 크기 분배:

```css
flex-grow
flex-shrink
flex-basis
```

Shorthand:

```css
flex
```

입니다.

---

# PART 7의 가장 중요한 한 문장

> **Flexbox의 핵심은 Property를 외우는 것이 아니라, Flex Container가 Main Axis와 Cross Axis를 기준으로 Flex Items의 정렬과 남는 공간을 관리한다는 원리를 이해하는 것이다.**


