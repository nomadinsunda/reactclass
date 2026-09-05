# PART 6. Layout — Flexbox & Grid

## Tailwind CSS로 Layout 만들기

지금까지 우리는 Tailwind CSS의 기본 Utility와 Responsive/State Variant를 배웠습니다.

이제 실제 웹 페이지의 **Layout**을 만드는 핵심 Utility를 학습합니다.

이번 PART의 핵심은 두 가지입니다.

```text
Flexbox
→ 한 방향을 중심으로 배치

Grid
→ 행(Row)과 열(Column)을 이용한 2차원 배치
```

Tailwind는 새로운 Layout 시스템을 제공하는 것이 아닙니다.

```text
CSS Flexbox / CSS Grid
        ↓
Tailwind Utility
        ↓
className으로 표현
```

즉 CSS의 Layout 원리를 이해하는 것이 먼저입니다.

---

# 1. `flex` — Flex Container 만들기

일반 CSS:

```css
.container {
  display: flex;
}
```

Tailwind:

```jsx
<div className="flex">
  <div>A</div>
  <div>B</div>
  <div>C</div>
</div>
```

`flex`는:

```css
display: flex;
```

에 대응합니다.

구조:

```text
Flex Container
┌──────────────────────────┐
│  Item A   Item B   Item C│
└──────────────────────────┘
```

중요:

> **`flex`는 자신을 배치하는 것이 아니라 해당 요소를 Flex Container로 만들고, 직접 자식들을 Flex Item으로 만듭니다.**

```text
<div class="flex">       ← Flex Container
   │
   ├─ <div>A</div>       ← Flex Item
   ├─ <div>B</div>       ← Flex Item
   └─ <div>C</div>       ← Flex Item
</div>
```

---

# 2. Main Axis와 Cross Axis

Flexbox를 제대로 이해하려면 두 개의 축을 알아야 합니다.

기본값:

```css
flex-direction: row;
```

따라서:

```text
              Main Axis
        ──────────────────→

┌─────────────────────────────┐
│                             │
│   A       B       C         │
│                             │
└─────────────────────────────┘

              ↑
              │
          Cross Axis
              │
              ↓
```

`justify-*`는 **Main Axis**를 기준으로 동작하고,

`items-*`는 **Cross Axis**를 기준으로 동작합니다.

이 관계가 Flexbox의 핵심입니다.

---

# 3. `flex-row` / `flex-col`

방향은 `flex-direction`으로 결정됩니다.

Tailwind:

```text
flex-row
flex-row-reverse
flex-col
flex-col-reverse
```

예:

```jsx
<div className="flex flex-row">
```

```text
A → B → C
```

반면:

```jsx
<div className="flex flex-col">
```

```text
A
↓
B
↓
C
```

여기서 매우 중요한 점:

> **`flex-direction`이 바뀌면 Main Axis도 함께 바뀝니다.**

```text
flex-row

Main Axis  ───────→
Cross Axis    ↓


flex-col

Main Axis      ↓
Cross Axis ───────→
```

따라서 `justify-center`의 방향도 달라집니다.

---

# 4. `justify-*` — Main Axis 정렬

대표 Utility:

```text
justify-start
justify-center
justify-end
justify-between
justify-around
justify-evenly
```

예:

```jsx
<div className="flex justify-center">
  <div>A</div>
  <div>B</div>
</div>
```

`flex-row`라면:

```text
┌───────────────────────────────┐
│          A    B               │
└───────────────────────────────┘
             ↑
        justify-center
```

`justify-between`:

```text
┌───────────────────────────────┐
│ A                           B │
└───────────────────────────────┘
```

중요:

```text
justify-* = 수평 정렬
```

이라고 외우면 안 됩니다.

정확히는:

> **`justify-*`는 Main Axis 방향으로 Flex Item을 정렬합니다.**

---

# 5. `items-*` — Cross Axis 정렬

대표 Utility:

```text
items-start
items-center
items-end
items-stretch
items-baseline
```

예:

```jsx
<div className="flex items-center h-32">
  <div>A</div>
  <div>B</div>
</div>
```

`flex-row`에서는 Cross Axis가 세로 방향이므로:

```text
┌───────────────────────────────┐
│                               │
│        A       B              │
│                               │
└───────────────────────────────┘
         ↑
    items-center
```

하지만:

```jsx
<div className="flex flex-col items-center">
```

이면 Cross Axis가 가로 방향입니다.

따라서:

```text
items-* = 세로 정렬
```

도 정확한 설명이 아닙니다.

정확히는:

> **`items-*`는 Cross Axis 방향으로 Flex Item을 정렬합니다.**

---

# 6. `justify-center + items-center`

화면 중앙 배치에서 매우 자주 사용하는 조합입니다.

```jsx
<div className="
  flex
  justify-center
  items-center
  min-h-screen
">
  <div>Login</div>
</div>
```

기본 `flex-row` 기준:

```text
justify-center
→ Main Axis 중앙

items-center
→ Cross Axis 중앙
```

결과:

```text
┌─────────────────────────────┐
│                             │
│                             │
│          Login              │
│                             │
│                             │
└─────────────────────────────┘
```

---

# 7. `gap-*` — Item 사이의 간격

예:

```jsx
<div className="flex gap-4">
  <button>저장</button>
  <button>취소</button>
</div>
```

결과:

```text
┌──────┐    ┌──────┐
│ 저장 │    │ 취소 │
└──────┘    └──────┘
           ↑
          gap
```

대표 Utility:

```text
gap-2
gap-4
gap-6

gap-x-4
gap-y-4
```

`gap`은 Flexbox와 Grid 모두에서 사용할 수 있습니다.

---

# 8. `flex-wrap`

기본 Flexbox는:

```css
flex-wrap: nowrap;
```

입니다.

Tailwind:

```text
flex-nowrap
flex-wrap
flex-wrap-reverse
```

예:

```jsx
<div className="flex flex-wrap gap-4">
  ...
</div>
```

공간이 부족하면:

```text
넓은 화면

[A] [B] [C] [D]


좁은 화면

[A] [B]
[C] [D]
```

처럼 다음 줄로 이동할 수 있습니다.

---

# 9. Flex Item — `grow`, `shrink`, `basis`

Flex Container뿐 아니라 **Flex Item 자체의 크기 동작**도 제어할 수 있습니다.

핵심 개념:

```text
flex-grow
→ 남는 공간을 얼마나 늘려 차지할 것인가?

flex-shrink
→ 공간이 부족할 때 얼마나 줄어들 것인가?

flex-basis
→ Flex Item 크기 계산의 시작점
```

Tailwind에서는 대표적으로:

```text
grow
grow-0

shrink
shrink-0

basis-*
```

를 사용합니다.

---

# 10. `grow`

예:

```jsx
<div className="flex">
  <div className="w-20">Menu</div>

  <main className="grow">
    Content
  </main>
</div>
```

구조:

```text
┌────────┬─────────────────────────┐
│ Menu   │ Content                 │
│ w-20   │ grow                    │
└────────┴─────────────────────────┘
          ↑
       남는 공간
       Content가 차지
```

`grow`는 대략:

```css
flex-grow: 1;
```

에 대응합니다.

---

# 11. `shrink-0`

예:

```jsx
<div className="flex">
  <img
    className="
      size-20
      shrink-0
    "
  />

  <div>
    매우 긴 상품 설명...
  </div>
</div>
```

`shrink-0`을 사용하면 이미지가 공간 부족 때문에 축소되는 것을 방지할 수 있습니다.

```text
┌─────────┬──────────────────┐
│ Image   │ 긴 텍스트...     │
│ 고정    │                  │
└─────────┴──────────────────┘
```

---

# 12. `basis-*`

`basis-*`는 Flex Item의 `flex-basis`를 설정합니다.

예:

```jsx
<div className="flex">
  <aside className="basis-1/4">
    Sidebar
  </aside>

  <main className="basis-3/4">
    Content
  </main>
</div>
```

개념:

```text
Container
┌──────────┬──────────────────────────┐
│  1/4     │          3/4             │
│ Sidebar  │        Content           │
└──────────┴──────────────────────────┘
```

다만 실제 최종 크기는 `grow`, `shrink`, available space 등의 영향을 함께 받을 수 있습니다.

따라서:

> **`basis-*`는 최종 width를 무조건 고정하는 Utility가 아니라 Flexbox 크기 계산의 기준값을 설정합니다.**

---

# 13. `flex-1`

실무에서 매우 자주 보는 Utility입니다.

```jsx
<div className="flex">
  <div className="flex-1">A</div>
  <div className="flex-1">B</div>
  <div className="flex-1">C</div>
</div>
```

결과:

```text
┌─────────┬─────────┬─────────┐
│    A    │    B    │    C    │
└─────────┴─────────┴─────────┘
```

세 Item이 사용 가능한 공간을 균등하게 나누는 대표적인 패턴입니다.

`flex-1`은 단순히 `flex-grow: 1`만 설정하는 Utility로 이해하면 안 됩니다.

Flex shorthand를 설정하므로 `grow`와는 의미가 다릅니다.

---

# 14. `self-*`

Container의 `items-*`가 모든 Flex Item의 Cross Axis 정렬을 결정한다면, 특정 Item만 다르게 배치할 수도 있습니다.

```jsx
<div className="flex items-start h-40">
  <div>A</div>

  <div className="self-center">
    B
  </div>

  <div>C</div>
</div>
```

대표 Utility:

```text
self-auto
self-start
self-center
self-end
self-stretch
self-baseline
```

구조:

```text
Container
items-start

A                 C
        B
        ↑
   self-center
```

---

# 15. Responsive Flex Layout

PART 4에서 배운 Responsive Variant를 Flexbox에 적용할 수 있습니다.

```jsx
<div className="
  flex
  flex-col
  md:flex-row
  gap-6
">
```

의미:

```text
기본
flex-col

        ↓ viewport 증가

md 이상
flex-row
```

결과:

```text
좁은 화면

[Image]
[Content]


md 이상

[Image] [Content]
```

이것이 Tailwind에서 매우 자주 사용하는 Responsive Layout 패턴입니다.

---

# 16. Grid란?

Flexbox 다음은 CSS Grid입니다.

```jsx
<div className="grid">
```

는:

```css
display: grid;
```

에 대응합니다.

Flexbox가 주로 한 축을 중심으로 Item을 배치한다면 Grid는 **행과 열을 함께 설계하는 2차원 Layout에 특히 적합합니다.**

```text
Grid Container

        Column
        ↓

┌────────┬────────┬────────┐
│   A    │   B    │   C    │ ← Row
├────────┼────────┼────────┤
│   D    │   E    │   F    │
└────────┴────────┴────────┘
```

---

# 17. `grid-cols-*`

가장 기본적인 Grid Utility입니다.

```jsx
<div className="
  grid
  grid-cols-3
  gap-4
">
```

결과:

```text
┌───────┬───────┬───────┐
│   A   │   B   │   C   │
├───────┼───────┼───────┤
│   D   │   E   │   F   │
└───────┴───────┴───────┘
```

`grid-cols-3`는 3개의 column track을 만듭니다.

대표:

```text
grid-cols-1
grid-cols-2
grid-cols-3
grid-cols-4
grid-cols-6
grid-cols-12
```

---

# 18. Responsive Grid

상품 목록 같은 UI에서는 Responsive Grid가 매우 자주 사용됩니다.

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-4
  gap-6
">
```

결과:

```text
기본

┌──────────────┐
│      A       │
├──────────────┤
│      B       │
└──────────────┘


md 이상

┌───────┬───────┐
│   A   │   B   │
├───────┼───────┤
│   C   │   D   │
└───────┴───────┘


lg 이상

┌────┬────┬────┬────┐
│ A  │ B  │ C  │ D  │
└────┴────┴────┴────┘
```

핵심:

> **column 수는 기기 이름으로 결정하는 것이 아니라 콘텐츠가 자연스럽게 배치되는 viewport 구간에 맞춰 결정합니다.**

---

# 19. `col-span-*`

Grid Item이 여러 column을 차지하도록 만들 수 있습니다.

```jsx
<div className="grid grid-cols-4 gap-4">
  <div className="col-span-2">
    A
  </div>

  <div>B</div>
  <div>C</div>
</div>
```

결과:

```text
┌───────────────┬───────┬───────┐
│       A       │   B   │   C   │
│  col-span-2   │       │       │
└───────────────┴───────┴───────┘
```

Responsive Variant와도 조합할 수 있습니다.

```jsx
<div className="
  col-span-1
  md:col-span-2
">
```

---

# 20. `row-span-*`

여러 Row를 차지하도록 할 수도 있습니다.

```jsx
<div className="grid grid-cols-3 gap-4">
  <div className="row-span-2">
    A
  </div>

  <div>B</div>
  <div>C</div>
  <div>D</div>
  <div>E</div>
</div>
```

개념:

```text
┌───────┬───────┬───────┐
│       │   B   │   C   │
│   A   ├───────┼───────┤
│       │   D   │   E   │
└───────┴───────┴───────┘
```

---

# 21. Grid의 `gap-*`

Flexbox와 마찬가지로 Grid에서도 `gap`을 사용할 수 있습니다.

```jsx
<div className="
  grid
  grid-cols-3
  gap-6
">
```

또는:

```jsx
<div className="
  grid
  grid-cols-3
  gap-x-6
  gap-y-8
">
```

```text
gap-x
→ column 사이 간격

gap-y
→ row 사이 간격
```

---

# 22. `place-items-*`

Grid에서 각 Item을 정렬할 때 사용할 수 있습니다.

```jsx
<div className="
  grid
  place-items-center
  h-64
">
  <div>Center</div>
</div>
```

결과:

```text
┌───────────────────────┐
│                       │
│        Center         │
│                       │
└───────────────────────┘
```

`place-items-center`는 Grid Item들의 block/inline axis 정렬을 한 번에 설정하는 shorthand입니다.

Flexbox의:

```text
justify-center + items-center
```

와 결과가 비슷해 보일 수 있지만 **CSS property와 동작 모델은 다릅니다.**

---

# 23. `place-content-*`

Grid 전체가 Grid Container 안에서 어떻게 배치될지도 제어할 수 있습니다.

대표:

```text
place-content-start
place-content-center
place-content-end
place-content-between
```

중요한 차이:

```text
place-items-*
→ Grid Item 자체의 정렬

place-content-*
→ Grid Track 전체의 정렬
```

입니다.

---

# 24. Flexbox와 Grid, 무엇을 선택할까?

단순히:

```text
Flexbox = 1차원
Grid = 2차원
```

으로 끝내면 부족합니다.

실무에서는 **무엇을 제어하고 싶은가**를 기준으로 선택하면 됩니다.

### Flexbox가 자연스러운 경우

```text
Navigation
Button Group
Toolbar
Icon + Text
Card 내부
Form Row
```

예:

```text
Logo       Menu        Login
─────────────────────────────→
```

하나의 흐름을 중심으로 정렬하는 UI입니다.

### Grid가 자연스러운 경우

```text
Product List
Dashboard
Gallery
Card Collection
Page Section Layout
```

예:

```text
┌──────┬──────┬──────┐
│Card  │Card  │Card  │
├──────┼──────┼──────┤
│Card  │Card  │Card  │
└──────┴──────┴──────┘
```

---

# 25. Flexbox와 Grid는 경쟁 관계가 아니다

실제 웹 페이지에서는 둘을 함께 사용합니다.

```text
Page
│
├─ Header                 ← Flexbox
│   ├─ Logo
│   ├─ Navigation
│   └─ Actions
│
├─ Product Section        ← Grid
│   │
│   ├─ Product Card       ← Flexbox
│   ├─ Product Card       ← Flexbox
│   ├─ Product Card       ← Flexbox
│   └─ Product Card       ← Flexbox
│
└─ Footer                 ← Grid 또는 Flexbox
```

즉:

> **큰 구조에는 Grid, 컴포넌트 내부 정렬에는 Flexbox**

라는 패턴이 자주 나오지만 이것 역시 절대적인 규칙은 아닙니다.

Layout 요구사항에 맞는 도구를 선택합니다.

---

# 26. 실전 ① Header

```jsx
<header
  className="
    flex
    items-center
    justify-between
    px-6
    py-4
    border-b
  "
>
  <h1 className="text-xl font-bold">
    MyShop
  </h1>

  <nav className="flex items-center gap-6">
    <a href="/">홈</a>
    <a href="/products">상품</a>
    <a href="/about">소개</a>
  </nav>

  <button>로그인</button>
</header>
```

구조:

```text
Flex Container

Logo              Navigation              Login
│                      │                     │
└──────── justify-between ──────────────────┘
```

그리고:

```text
items-center
→ Cross Axis 중앙 정렬
```

입니다.

---

# 27. 실전 ② Responsive Product Grid

```jsx
<section
  className="
    grid
    grid-cols-1
    md:grid-cols-2
    lg:grid-cols-4
    gap-6
  "
>
  {products.map((product) => (
    <ProductCard
      key={product.id}
      product={product}
    />
  ))}
</section>
```

여기서는:

```text
Grid
→ Card들의 전체 배치

ProductCard 내부
→ Flexbox
```

를 사용할 수 있습니다.

---

# 28. 실전 ③ Sidebar + Content

```jsx
<div
  className="
    flex
    flex-col
    lg:flex-row
    gap-8
  "
>
  <aside
    className="
      lg:basis-64
      lg:shrink-0
    "
  >
    Sidebar
  </aside>

  <main className="min-w-0 grow">
    Content
  </main>
</div>
```

기본:

```text
┌─────────────────────┐
│ Sidebar             │
├─────────────────────┤
│ Content             │
└─────────────────────┘
```

`lg` 이상:

```text
┌─────────┬─────────────────────────┐
│ Sidebar │ Content                 │
│         │                         │
└─────────┴─────────────────────────┘
```

여기서:

```text
Sidebar
→ 크기 유지

Content
→ 남은 공간 차지
```

라는 역할이 명확합니다.

---

# 29. `min-w-0`이 필요한 이유

Flex Layout에서 자주 만나는 실전 문제입니다.

```jsx
<div className="flex">
  <aside className="shrink-0">
    Sidebar
  </aside>

  <main className="min-w-0 grow">
    매우 긴 Content...
  </main>
</div>
```

Flex Item 내부에 긴 text나 넓은 content가 있으면 예상보다 Item이 줄어들지 않아 overflow가 발생할 수 있습니다.

이때:

```text
min-w-0
```

을 사용하면 Flex Item이 필요한 경우 container 폭 안에서 줄어들 수 있도록 허용할 수 있습니다.

이후:

```text
truncate
overflow-hidden
```

등과 함께 사용하는 경우가 많습니다.

---

# 30. 자주 하는 실수 ① `justify-*` = 수평이라고 외우기

잘못된 이해:

```text
justify-center
= 수평 중앙
```

정확한 이해:

```text
justify-*
= Main Axis 정렬
```

따라서:

```text
flex-row
→ Main Axis = 가로

flex-col
→ Main Axis = 세로
```

입니다.

---

# 31. 자주 하는 실수 ② `items-*` = 세로라고 외우기

마찬가지입니다.

```text
items-*
= Cross Axis 정렬
```

입니다.

따라서 `flex-direction`에 따라 실제 화면상의 방향이 달라집니다.

```text
              justify-*       items-*

flex-row      가로             세로

flex-col      세로             가로
```

이 표는 반드시 기억해야 합니다.

---

# 32. 자주 하는 실수 ③ 모든 Layout을 Flexbox로 만들기

Flexbox로 많은 Layout을 구현할 수 있습니다.

하지만 Card 목록처럼 명확한 행/열 구조를:

```jsx
<div className="flex flex-wrap">
```

만으로 해결하려고 하면 Layout 계산이 복잡해질 수 있습니다.

이런 경우:

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-4
">
```

가 의도를 더 명확하게 표현할 수 있습니다.

---

# 33. 자주 하는 실수 ④ 모든 Layout을 Grid로 만들기

반대도 마찬가지입니다.

버튼 두 개를 단순히 옆으로 정렬하려는데:

```jsx
<div className="grid grid-cols-2">
```

를 반드시 사용할 필요는 없습니다.

```jsx
<div className="flex gap-2">
```

가 더 자연스러울 수 있습니다.

중요한 것은:

> **Utility 개수가 아니라 Layout의 의도가 코드에서 잘 드러나는가?**

입니다.

---

# 34. Responsive + Layout + State 조합

지금까지 배운 PART들이 이제 하나로 연결됩니다.

```jsx
<div
  className="
    flex
    flex-col
    gap-4

    md:flex-row
    md:gap-6
  "
>
```

그리고:

```jsx
<article
  className="
    group
    flex
    flex-col
    rounded-xl
    border
    hover:shadow-lg
    transition-shadow
  "
>
```

또:

```jsx
<section
  className="
    grid
    grid-cols-1
    md:grid-cols-2
    lg:grid-cols-4
    gap-6
  "
>
```

즉:

```text
Layout Utility
      +
Responsive Variant
      +
State Variant
      ↓
실제 Responsive Interactive UI
```

가 됩니다.

---

# 35. PART 6 핵심 정리

## Flexbox

```text
flex
│
├─ Direction
│   ├─ flex-row
│   └─ flex-col
│
├─ Main Axis
│   └─ justify-*
│
├─ Cross Axis
│   └─ items-*
│
├─ Spacing
│   └─ gap-*
│
├─ Wrapping
│   └─ flex-wrap
│
└─ Flex Item
    ├─ grow
    ├─ shrink
    ├─ basis-*
    ├─ flex-1
    └─ self-*
```

## Grid

```text
grid
│
├─ Tracks
│   └─ grid-cols-*
│
├─ Item Span
│   ├─ col-span-*
│   └─ row-span-*
│
├─ Spacing
│   └─ gap-*
│
└─ Alignment
    ├─ place-items-*
    └─ place-content-*
```

---

# 가장 중요한 세 가지

### 1. `flex`와 `grid`는 자식 배치를 위한 Layout Mode다

```text
Container
   ↓
display: flex / grid
   ↓
직접 자식의 Layout 결정
```

### 2. Flexbox에서는 Axis를 먼저 생각한다

```text
flex-direction
      ↓
Main Axis 결정
      ↓
justify-*

Cross Axis
      ↓
items-*
```

### 3. Flexbox와 Grid는 함께 사용한다

```text
Page Layout
     ↓
Grid

Component 내부
     ↓
Flexbox
```

처럼 조합하는 경우가 매우 많습니다.

하지만 어느 하나를 특정 용도로 고정하는 것이 아니라 **Layout 요구사항에 맞춰 선택합니다.**

---

# PART 6 최종 실전 구조

```text
Responsive Shopping Page
│
├─ Header
│    └─ Flexbox
│
├─ Main
│    │
│    ├─ Sidebar + Content
│    │       └─ Responsive Flexbox
│    │
│    └─ Product List
│            └─ Responsive Grid
│
└─ Product Card
     │
     ├─ 내부 정보 배치 → Flexbox
     └─ hover → State Variant
```

결국 지금까지의 PART들이 하나로 연결됩니다.

```text
Utility
   +
Responsive Variant
   +
State Variant
   +
Flexbox / Grid
   ↓
Responsive Interactive Layout
```

> **Tailwind에서 Layout을 잘한다는 것은 Utility 이름을 많이 외우는 것이 아니라 CSS Flexbox와 Grid의 원리를 이해하고 필요한 Utility를 정확한 위치에 조합하는 것입니다.**
