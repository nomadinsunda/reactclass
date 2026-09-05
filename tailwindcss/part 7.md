# PART 7. Sizing & Spacing

## Tailwind CSS로 크기와 여백 제어하기

PART 6에서는 Flexbox와 Grid를 이용해 Layout을 구성했습니다.

이번 PART에서는 Layout을 실제 화면에 맞게 다듬는 핵심 요소인 **크기(Sizing)**와 **여백(Spacing)**을 학습합니다.

```text
Layout
  ↓
요소를 어디에 배치할 것인가?

Sizing
  ↓
요소를 얼마나 크게 만들 것인가?

Spacing
  ↓
요소 사이에 얼마나 공간을 둘 것인가?
```

Tailwind에서는 대표적으로 다음 Utility를 사용합니다.

```text
Sizing
├─ w-*          width
├─ h-*          height
├─ size-*       width + height
├─ min-w-*      min-width
├─ max-w-*      max-width
├─ min-h-*      min-height
└─ max-h-*      max-height

Spacing
├─ p-*          padding
├─ m-*          margin
├─ gap-*        gap
└─ space-*      자식 사이 spacing
```

---

# 1. Tailwind의 Spacing Scale

Tailwind의 많은 크기와 여백 Utility는 **일관된 spacing scale**을 공유합니다.

예를 들어:

```jsx
<div className="p-4">
```

```jsx
<div className="w-64">
```

```jsx
<div className="gap-6">
```

처럼 숫자를 사용합니다.

여기서 중요한 점은:

> `4`, `6`, `64`를 CSS의 `4px`, `6px`, `64px`로 해석하면 안 됩니다.

Tailwind의 spacing token을 참조합니다.

대표적인 예:

```text
Utility     대표적인 디폴트 값

p-1         0.25rem
p-2         0.5rem
p-4         1rem
p-6         1.5rem
p-8         2rem
```

일반적인 root font size가 `16px`라면:

```text
1rem = 16px

p-4
→ 1rem
→ 약 16px
```

입니다.

하지만 Tailwind에서는 단순히 px 값을 외우기보다 **일관된 spacing system을 사용한다**는 관점이 더 중요합니다.

---

# 2. `w-*` — Width

요소의 width를 설정합니다.

```jsx
<div className="w-64">
  Sidebar
</div>
```

대표적인 형태:

```text
w-4
w-8
w-16
w-32
w-64

w-1/2
w-1/3
w-2/3

w-full
w-screen
w-auto
```

예:

```jsx
<div className="w-1/2">
```

개념적으로:

```css
width: 50%;
```

입니다.

---

# 3. 고정 크기와 상대 크기

다음 두 코드는 성격이 다릅니다.

```jsx
<div className="w-64">
```

```jsx
<div className="w-1/2">
```

`w-64`는 spacing scale에 따른 크기이고,

`w-1/2`는 부모 공간에 대한 비율입니다.

```text
부모
┌──────────────────────────────┐
│                              │
│ ┌──────────────┐             │
│ │    w-1/2     │             │
│ └──────────────┘             │
│                              │
└──────────────────────────────┘
```

`w-full`은:

```text
부모
┌──────────────────────────────┐
│┌────────────────────────────┐│
││           w-full           ││
│└────────────────────────────┘│
└──────────────────────────────┘
```

처럼 사용 가능한 부모 너비를 채웁니다.

---

# 4. `h-*` — Height

높이는 `h-*`로 설정합니다.

```jsx
<div className="h-32">
```

대표:

```text
h-8
h-16
h-32
h-64

h-full
h-screen
h-auto
```

예:

```jsx
<section className="h-screen">
```

은 viewport 높이를 기준으로 화면 높이만큼의 영역을 만드는 대표적인 패턴입니다.

다만 모바일 브라우저 UI까지 고려해야 하는 Layout에서는 viewport unit의 특성을 이해해야 합니다.

---

# 5. `size-*` — Width와 Height를 동시에

정사각형 요소에서는:

```jsx
<div className="w-12 h-12">
```

보다:

```jsx
<div className="size-12">
```

처럼 표현할 수 있습니다.

개념:

```text
size-12

width
  ↔
┌──────┐
│      │ ↕ height
│      │
└──────┘
```

Avatar, Icon Container, 정사각형 Thumbnail 등에 유용합니다.

예:

```jsx
<img
  src={user.avatar}
  alt={user.name}
  className="
    size-12
    rounded-full
    object-cover
  "
/>
```

---

# 6. `min-w-*`와 `max-w-*`

실전 Responsive UI에서는 단순한 `width`보다 최소/최대 크기가 더 중요한 경우가 많습니다.

```text
width
→ 현재 width

min-width
→ 이것보다 작아지지 마라

max-width
→ 이것보다 커지지 마라
```

예:

```jsx
<div className="w-full max-w-md">
```

의 의미는:

```text
가능하면
width = 100%

하지만

max-width 이상으로
커지지는 않음
```

입니다.

시각적으로:

```text
넓은 부모

┌────────────────────────────────────┐
│                                    │
│       ┌────────────────────┐       │
│       │ w-full max-w-md    │       │
│       └────────────────────┘       │
│                                    │
└────────────────────────────────────┘
```

---

# 7. 가운데 정렬과 `max-w-*`

페이지 Content Container에서 매우 자주 사용합니다.

```jsx
<main
  className="
    mx-auto
    w-full
    max-w-7xl
    px-4
  "
>
  ...
</main>
```

각 Utility의 역할:

```text
w-full
→ 사용 가능한 width를 사용

max-w-7xl
→ 너무 넓어지는 것을 제한

mx-auto
→ 남는 좌우 margin을 자동 분배

px-4
→ 작은 viewport에서도 좌우 내부 여백 확보
```

결과:

```text
큰 viewport

┌────────────────────────────────────────────┐
│                                            │
│   ┌────────────────────────────────────┐   │
│   │              Content               │   │
│   └────────────────────────────────────┘   │
│                                            │
└────────────────────────────────────────────┘
       ↑                                ↑
                mx-auto
```

이 패턴은 실무에서 매우 중요합니다.

---

# 8. `min-h-*`와 `max-h-*`

Height에도 같은 개념이 있습니다.

```jsx
<div className="min-h-screen">
```

페이지 Layout에서 매우 자주 사용합니다.

```jsx
<div className="min-h-screen flex flex-col">
  <Header />

  <main className="grow">
    ...
  </main>

  <Footer />
</div>
```

여기서:

```text
min-h-screen
→ Content가 적어도 viewport 높이만큼은 확보

grow
→ Main이 남는 공간을 차지
```

합니다.

Content가 길어지면 `min-height`이므로 페이지가 더 길어질 수도 있습니다.

따라서 일반적인 페이지 Layout에서는 `h-screen`보다 `min-h-screen`이 더 자연스러운 경우가 많습니다.

---

# 9. Padding — `p-*`

Padding은 **요소의 border 안쪽 공간**입니다.

```text
┌──────────────────────────┐ ← Border
│          Padding         │
│   ┌──────────────────┐   │
│   │     Content      │   │
│   └──────────────────┘   │
│          Padding         │
└──────────────────────────┘
```

Tailwind:

```jsx
<div className="p-4">
```

방향별 Utility:

```text
p-*    모든 방향

px-*   좌우
py-*   상하

pt-*   위
pr-*   오른쪽
pb-*   아래
pl-*   왼쪽
```

예:

```jsx
<button
  className="
    px-4
    py-2
  "
>
  저장
</button>
```

---

# 10. Margin — `m-*`

Margin은 **요소 border 바깥쪽 공간**입니다.

```text
          Margin
             ↓

      ┌───────────────┐
      │    Element    │
      └───────────────┘

             ↑
          Margin
```

Tailwind:

```text
m-*    모든 방향

mx-*   좌우
my-*   상하

mt-*   위
mr-*   오른쪽
mb-*   아래
ml-*   왼쪽
```

예:

```jsx
<h2 className="mb-6">
  인기 상품
</h2>
```

---

# 11. Padding과 Margin의 차이

이 둘은 반드시 구분해야 합니다.

```text
Margin
┌────────────────────────────────┐
│                                │
│    Border                      │
│    ┌──────────────────────┐    │
│    │       Padding        │    │
│    │   ┌──────────────┐   │    │
│    │   │   Content    │   │    │
│    │   └──────────────┘   │    │
│    │                      │    │
│    └──────────────────────┘    │
│                                │
└────────────────────────────────┘
```

핵심:

```text
Padding
→ Element 내부의 공간

Margin
→ Element 외부의 공간
```

예:

```jsx
<div className="m-8 p-4">
  Content
</div>
```

```text
m-8
→ 다른 요소와의 외부 거리

p-4
→ 자신의 Content와 Border 사이 거리
```

---

# 12. `mx-auto`

매우 자주 사용되는 패턴입니다.

```jsx
<div className="w-64 mx-auto">
```

좌우 margin을 `auto`로 설정하여 **남는 inline 방향 공간을 양쪽에 분배**하는 대표적인 block layout 패턴입니다.

일반적인 horizontal writing mode에서는 결과적으로 가로 중앙 배치에 사용됩니다.

```text
Parent

┌──────────────────────────────────┐
│                                  │
│        ┌──────────────┐          │
│        │   Element    │          │
│        └──────────────┘          │
│                                  │
└──────────────────────────────────┘
```

중요:

> `mx-auto`가 모든 상황에서 모든 요소를 무조건 가운데 배치하는 마법의 Utility는 아닙니다.

Container의 남는 공간과 요소의 sizing/layout 조건이 필요합니다.

---

# 13. Negative Margin

Tailwind에서는 음수 Margin도 사용할 수 있습니다.

```jsx
<div className="-mt-8">
```

예:

```text
Banner
┌──────────────────────────────┐
│                              │
│                              │
└──────────────────────────────┘
             ▲
       ┌──────────────┐
       │     Card     │
       └──────────────┘
          -mt-8
```

Card가 위쪽 영역과 겹치는 효과를 만들 수 있습니다.

하지만 Negative Margin을 Layout 문제를 억지로 해결하는 수단으로 남용하면 유지보수가 어려워질 수 있습니다.

---

# 14. `gap-*`

PART 6에서도 사용했던 Utility입니다.

Flexbox와 Grid의 Item 사이 간격을 설정합니다.

```jsx
<div className="flex gap-4">
```

또는:

```jsx
<div className="grid grid-cols-3 gap-6">
```

방향별로:

```text
gap-*     row + column

gap-x-*   column 방향 간격

gap-y-*   row 방향 간격
```

을 사용할 수 있습니다.

---

# 15. `gap`과 Margin의 차이

예를 들어 버튼 그룹을 만들겠습니다.

Margin 방식:

```jsx
<div className="flex">
  <button className="mr-2">저장</button>
  <button className="mr-2">수정</button>
  <button>삭제</button>
</div>
```

마지막 Item의 Margin을 따로 고려해야 합니다.

`gap`을 사용하면:

```jsx
<div className="flex gap-2">
  <button>저장</button>
  <button>수정</button>
  <button>삭제</button>
</div>
```

훨씬 명확합니다.

```text
[저장]  ←gap→  [수정]  ←gap→  [삭제]
```

핵심:

> **Flex/Grid 자식들 사이에 동일한 간격을 만들 목적이라면 `gap-*`을 우선 고려하세요.**

---

# 16. `space-x-*`, `space-y-*`

Tailwind에는 형제 요소 사이의 spacing을 만드는 Utility도 있습니다.

```jsx
<div className="space-y-4">
  <Card />
  <Card />
  <Card />
</div>
```

개념:

```text
[ Card ]
    ↕
  space
    ↕
[ Card ]
    ↕
  space
    ↕
[ Card ]
```

대표:

```text
space-x-*
space-y-*
```

다만 Flexbox/Grid Layout에서는 `gap-*`이 의도를 더 직접적으로 표현하는 경우가 많습니다.

따라서:

```text
Flex / Grid Item 사이
→ gap-* 우선 고려

일반적인 형제 요소 사이의 spacing
→ space-*도 선택 가능
```

정도로 이해하면 좋습니다.

---

# 17. Responsive Spacing

Spacing에도 Responsive Variant를 사용할 수 있습니다.

```jsx
<section
  className="
    px-4
    py-8

    md:px-8
    md:py-12

    lg:px-12
    lg:py-16
  "
>
```

의미:

```text
base
px-4 py-8

        ↓ md 이상

px-8 py-12

        ↓ lg 이상

px-12 py-16
```

viewport가 넓어지면서 Section의 breathing room을 자연스럽게 늘릴 수 있습니다.

---

# 18. Responsive Sizing

Sizing에도 같은 방식으로 적용합니다.

```jsx
<img
  className="
    size-16
    md:size-20
    lg:size-24
  "
/>
```

또는:

```jsx
<aside
  className="
    w-full
    lg:w-64
  "
>
```

즉:

```text
Sizing Utility
      +
Responsive Variant
      ↓
Responsive Sizing
```

입니다.

---

# 19. Arbitrary Value

디자인 요구사항에 따라 디폴트 Utility로 표현하기 어려운 값이 필요할 수도 있습니다.

예:

```jsx
<div className="w-[420px]">
```

```jsx
<header className="h-[72px]">
```

```jsx
<div className="mt-[18px]">
```

Tailwind의 arbitrary value를 이용하면 CSS 값을 직접 지정할 수 있습니다.

하지만:

```text
w-[421px]
mt-[17px]
gap-[13px]
h-[73px]
```

처럼 arbitrary value를 지나치게 많이 사용하면 **일관된 Design System의 장점이 약해집니다.**

따라서:

> **먼저 Tailwind의 spacing/theme token을 사용하고, 실제 디자인 요구사항이 있을 때 arbitrary value를 사용합니다.**

---

# 20. Arbitrary Property와 혼동하지 않기

다음은 arbitrary **value**입니다.

```jsx
className="w-[420px]"
```

이미 존재하는 `width` Utility에 임의 값을 전달합니다.

반면 다음처럼 CSS property 자체를 직접 표현하는 문법도 있습니다.

```jsx
className="[mask-type:luminance]"
```

이번 PART에서 주로 사용하는 것은:

```text
w-[...]
h-[...]
p-[...]
m-[...]
gap-[...]
```

형태의 **arbitrary value**입니다.

---

# 21. `calc()` 사용

복합적인 크기가 필요하다면 arbitrary value 안에서 CSS 함수를 사용할 수도 있습니다.

예:

```jsx
<main className="w-[calc(100%-16rem)]">
```

개념:

```text
전체 width
────────────────────────────────────

Sidebar             Main
16rem                나머지
┌───────────┬───────────────────────┐
│           │                       │
└───────────┴───────────────────────┘
```

다만 PART 6에서 배운 Flexbox를 사용하면:

```jsx
<div className="flex">
  <aside className="w-64 shrink-0">
    ...
  </aside>

  <main className="min-w-0 grow">
    ...
  </main>
</div>
```

처럼 `calc()` 없이 더 자연스럽게 해결할 수 있는 경우도 많습니다.

> **Sizing 계산보다 Layout system이 문제를 해결할 수 있는지 먼저 확인하세요.**

---

# 22. Overflow와 Sizing

고정된 크기를 설정하면 Content가 영역보다 커질 수 있습니다.

```jsx
<div className="h-40">
  아주 많은 Content...
</div>
```

이때 overflow가 발생할 수 있습니다.

대표 Utility:

```text
overflow-hidden
overflow-auto
overflow-scroll

overflow-x-auto
overflow-y-auto
```

예:

```jsx
<div
  className="
    max-h-80
    overflow-y-auto
  "
>
  ...
</div>
```

결과:

```text
┌────────────────────┐
│ Item 1             │
│ Item 2             │
│ Item 3             │
│ Item 4             │ ↑
│ Item 5             │ │ scroll
└────────────────────┘ ↓
```

Sizing과 Overflow는 함께 생각해야 합니다.

---

# 23. `truncate`

긴 문자열을 한 줄에서 줄임표로 처리하는 대표적인 Utility입니다.

```jsx
<p className="truncate">
  아주 길고 긴 상품 이름입니다...
</p>
```

결과:

```text
아주 길고 긴 상품 이름입...
```

실전에서는 부모/Flex Item의 width 제약이 중요합니다.

PART 6에서 배운:

```jsx
<div className="min-w-0 grow">
  <p className="truncate">
    ...
  </p>
</div>
```

패턴과 함께 자주 사용합니다.

---

# 24. 실전 — Content Container

많은 웹사이트에서 사용하는 대표적인 Container 패턴입니다.

```jsx
<div
  className="
    mx-auto
    w-full
    max-w-7xl
    px-4
    sm:px-6
    lg:px-8
  "
>
  ...
</div>
```

구조:

```text
넓은 viewport

┌──────────────────────────────────────────────┐
│                                              │
│    ┌────────────────────────────────────┐    │
│    │            Content                 │    │
│    └────────────────────────────────────┘    │
│                                              │
└──────────────────────────────────────────────┘

     ↑ px                        px ↑
        ←──── max-width ────→
```

핵심:

```text
mx-auto
→ 중앙 배치

w-full
→ 좁은 영역에서는 사용 가능한 width 활용

max-w-7xl
→ 큰 화면에서 지나치게 넓어지는 것을 제한

Responsive px
→ viewport에 따라 좌우 padding 조절
```

---

# 25. 실전 — Product Card

```jsx
<article
  className="
    overflow-hidden
    rounded-xl
    border
    bg-white
  "
>
  <img
    src={product.image}
    alt={product.name}
    className="
      aspect-square
      w-full
      object-cover
    "
  />

  <div className="p-4">
    <h3 className="truncate font-medium">
      {product.name}
    </h3>

    <p className="mt-2 font-bold">
      {product.price.toLocaleString()}원
    </p>

    <button
      className="
        mt-4
        w-full
        rounded-lg
        px-4
        py-2
      "
    >
      장바구니
    </button>
  </div>
</article>
```

여기에는 여러 Sizing/Spacing Utility가 동시에 사용됩니다.

```text
w-full
→ Image / Button width

aspect-square
→ Image 비율

p-4
→ Card 내부 padding

mt-2 / mt-4
→ 요소 사이의 수직 spacing

px-4 / py-2
→ Button 내부 padding
```

---

# 26. 실전 — Page Layout

PART 6과 PART 7을 연결해 보겠습니다.

```jsx
<div className="min-h-screen flex flex-col">
  <Header />

  <div className="flex grow">
    <aside
      className="
        hidden
        w-64
        shrink-0
        border-r
        lg:block
      "
    >
      ...
    </aside>

    <main className="min-w-0 grow">
      <div
        className="
          mx-auto
          w-full
          max-w-7xl
          px-4
          py-8
          sm:px-6
          lg:px-8
        "
      >
        ...
      </div>
    </main>
  </div>

  <Footer />
</div>
```

전체 관계:

```text
Page
│
├─ min-h-screen
│
├─ Header
│
├─ Main Area
│   │
│   ├─ Sidebar
│   │   ├─ w-64
│   │   └─ shrink-0
│   │
│   └─ Main
│       ├─ min-w-0
│       └─ grow
│           │
│           └─ Content Container
│               ├─ mx-auto
│               ├─ w-full
│               ├─ max-w-7xl
│               └─ Responsive padding
│
└─ Footer
```

이제 Layout과 Sizing/Spacing이 하나로 연결됩니다.

---

# 27. 자주 하는 실수 ① 모든 크기를 고정하기

예:

```jsx
<div className="w-[1200px]">
```

좁은 viewport에서 문제가 발생할 수 있습니다.

대부분의 Content Container에서는:

```jsx
<div className="w-full max-w-7xl">
```

처럼 **유연한 width + 최대 width 제한**이 더 자연스럽습니다.

---

# 28. 자주 하는 실수 ② Margin으로 모든 간격 만들기

예:

```jsx
<div className="flex">
  <Card className="mr-4" />
  <Card className="mr-4" />
  <Card />
</div>
```

Flex/Grid Item 간 일정한 간격이라면:

```jsx
<div className="flex gap-4">
```

가 의도를 더 명확하게 표현합니다.

---

# 29. 자주 하는 실수 ③ Arbitrary Value 남용

다음과 같은 코드가 계속 등장한다면:

```jsx
p-[17px]
mt-[13px]
gap-[11px]
w-[347px]
```

Design System의 spacing 규칙이 없는 상태일 가능성이 있습니다.

가능하면:

```jsx
p-4
mt-3
gap-3
```

처럼 theme에 정의된 값을 우선 사용합니다.

특수한 디자인 요구사항에는 arbitrary value가 유용하지만 **디폴트 선택지가 되어서는 안 됩니다.**

---

# 30. 자주 하는 실수 ④ `h-screen` 남용

페이지 전체 높이를 만들 때 무조건:

```jsx
h-screen
```

을 사용하는 경우가 있습니다.

하지만 Content가 viewport보다 길어질 수 있는 일반 페이지라면:

```jsx
min-h-screen
```

이 더 적합할 수 있습니다.

```text
h-screen
→ height 자체를 viewport 크기로 설정

min-h-screen
→ 최소 높이를 viewport 크기로 설정
→ Content가 많으면 더 커질 수 있음
```

---

# 31. 자주 하는 실수 ⑤ 부모 크기를 무시한 `%`

```jsx
<div className="h-full">
```

이라고 했다고 항상 화면 전체 높이가 되는 것은 아닙니다.

Percentage height는 containing block의 높이 조건에 영향을 받습니다.

마찬가지로:

```jsx
w-full
```

도 “화면 전체”가 아니라 **해당 요소가 놓인 containing context에서 사용 가능한 width**를 기준으로 이해해야 합니다.

---

# 32. Sizing을 결정하는 사고 순서

요소의 크기를 정할 때 바로 `w-[...]`부터 작성하지 않는 것이 좋습니다.

```text
① Content가 자연스럽게 크기를 결정해도 되는가?
              ↓
             YES
              ↓
          auto 활용


② 부모 공간을 채워야 하는가?
              ↓
             YES
              ↓
          w-full / grow


③ 최대 크기를 제한해야 하는가?
              ↓
             YES
              ↓
          max-w-*


④ 최소 크기를 보장해야 하는가?
              ↓
             YES
              ↓
          min-w-*


⑤ 정확한 특수 값이 필요한가?
              ↓
             YES
              ↓
       arbitrary value
```

---

# 33. Spacing을 결정하는 사고 순서

```text
공간이 필요한 위치는 어디인가?
              │
      ┌───────┴────────┐
      │                │
 요소 내부          요소 사이
      │                │
      ↓                ↓
   padding        Flex / Grid?
                       │
                  YES  ↓
                      gap
                       │
                  NO   ↓
               margin / space
```

즉:

```text
Content ↔ Border
→ padding

Flex/Grid Item ↔ Item
→ gap 우선 고려

독립적인 Element 사이 외부 공간
→ margin
```

이라는 기준을 가지면 Utility 선택이 쉬워집니다.

---

# 34. PART 7 핵심 Utility

```text
Sizing
─────────────────────
w-*
h-*
size-*

min-w-*
max-w-*
min-h-*
max-h-*


Spacing
─────────────────────
p-*
px-* / py-*

m-*
mx-* / my-*

gap-*
gap-x-* / gap-y-*

space-x-*
space-y-*


실전
─────────────────────
w-full
max-w-*
mx-auto
min-h-screen
grow
shrink-0
min-w-0
overflow-*
truncate
```

---

# 35. PART 7 핵심 정리

Sizing과 Spacing은 단순히 숫자를 지정하는 작업이 아닙니다.

```text
Layout
→ 구조

Sizing
→ 크기

Spacing
→ 공간

Responsive Variant
→ viewport 조건에 따른 변화
```

이 네 가지가 함께 동작합니다.

대표적인 실전 패턴:

```jsx
<div
  className="
    mx-auto
    w-full
    max-w-7xl
    px-4
    py-8
    sm:px-6
    lg:px-8
  "
>
```

이 짧은 코드 안에도:

```text
Sizing
+
Spacing
+
Responsive
+
Container Layout
```

이 모두 들어 있습니다.

> **좋은 Responsive UI는 모든 값을 고정하는 것이 아니라 Content와 부모 공간에 따라 자연스럽게 변하도록 크기와 여백에 제약을 설계하는 것에서 시작합니다.**

---

# PART 7 이미지 구성 — 총 8장

강의 이미지는 다음 **8장**으로 구성하는 것이 가장 좋습니다.

```text
1/8
Sizing & Spacing 전체 개념
+ Tailwind Spacing Scale

2/8
Width / Height / size
+ 고정·비율·full 크기

3/8
min/max Sizing
+ w-full max-w-*
+ Content Container

4/8
Padding vs Margin
+ 방향별 Utility
+ mx-auto

5/8
gap vs margin vs space-*
+ Flex/Grid spacing

6/8
Responsive Sizing & Spacing
+ Arbitrary Value
+ calc()

7/8
Overflow / truncate
+ min-w-0
+ 실전 Sizing 문제 해결

8/8
Responsive Page 종합 실전
+ Sidebar
+ Content Container
+ Product Card
+ PART 7 핵심 정리
```

PART 6에서 사용했던 **밝은 배경 + 블루 계열 + 사람 캐릭터 없음 + 8장 연속 인포그래픽** 스타일을 그대로 유지하면 PART 1~7 전체 강의자료의 시각적 일관성도 유지할 수 있습니다.
