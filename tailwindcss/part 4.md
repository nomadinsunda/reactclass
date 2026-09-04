# PART 4. Responsive Design

## 화면 크기에 따라 UI 변경하기

Tailwind CSS의 Responsive Design은 기본 Utility 앞에 breakpoint variant를 붙여 구현합니다.

```jsx
<div className="w-full md:w-1/2 lg:w-1/3">
  Content
</div>
```

핵심 구조는 다음과 같습니다.

```text
기본 Utility
   +
Breakpoint Variant
   ↓
md:w-1/2
   ↓
md 이상에서 w-1/2 적용
```

이번 PART에서는 다음 내용을 학습합니다.

```text
Responsive Design
│
├─ Mobile First
├─ Breakpoint
├─ sm:
├─ md:
├─ lg:
├─ xl:
├─ 2xl:
├─ Responsive Typography
├─ Responsive Spacing
└─ Responsive Layout
```

---

# 1. Responsive Design이란?

웹 페이지는 다양한 크기의 화면에서 실행됩니다.

```text
Mobile
Tablet
Laptop
Desktop
Large Desktop
```

같은 Component라도 화면 크기에 따라 다른 형태가 필요할 수 있습니다.

예를 들어 상품 목록을 생각해 봅시다.

모바일:

```text
┌────────────┐
│ Product 1  │
├────────────┤
│ Product 2  │
├────────────┤
│ Product 3  │
└────────────┘
```

태블릿:

```text
┌──────────┐ ┌──────────┐
│Product 1 │ │Product 2 │
└──────────┘ └──────────┘

┌──────────┐
│Product 3 │
└──────────┘
```

데스크톱:

```text
┌──────────┐ ┌──────────┐ ┌──────────┐
│Product 1 │ │Product 2 │ │Product 3 │
└──────────┘ └──────────┘ └──────────┘
```

Tailwind에서는 이러한 변화를 breakpoint variant로 표현합니다.

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
">
```

의미:

```text
기본        → 1 column
md 이상     → 2 columns
lg 이상     → 3 columns
```

---

# 2. Tailwind의 핵심 — Mobile First

Tailwind의 Responsive Design을 이해할 때 가장 중요한 개념은 **Mobile First**입니다.

예:

```jsx
<h1 className="text-xl md:text-3xl">
  Tailwind CSS
</h1>
```

많은 초보자가 다음과 같이 생각합니다.

```text
text-xl
→ 모바일 전용

md:text-3xl
→ 태블릿 전용
```

정확하지 않습니다.

실제로는:

```text
text-xl
→ 조건 없이 적용

md:text-3xl
→ md breakpoint 이상에서 적용
```

입니다.

따라서 결과는:

```text
작은 화면
text-xl

        ↓ md breakpoint

md 이상
text-3xl
```

입니다.

핵심:

> **prefix가 없는 Utility가 기본 스타일이고, breakpoint variant는 특정 최소 너비 이상에서 스타일을 추가하거나 덮어씁니다.**

---

# 3. Breakpoint란?

Breakpoint는 **viewport 너비를 기준으로 스타일이 변경되는 경계값**입니다.

Tailwind의 디폴트 breakpoint는 일반적으로 다음과 같습니다.

| Variant | 최소 viewport width |
| ------- | ----------------: |
| `sm:`   |             40rem |
| `md:`   |             48rem |
| `lg:`   |             64rem |
| `xl:`   |             80rem |
| `2xl:`  |             96rem |

root font-size가 16px인 일반적인 환경에서 대략:

```text
sm   40rem  ≈ 640px
md   48rem  ≈ 768px
lg   64rem  ≈ 1024px
xl   80rem  ≈ 1280px
2xl  96rem  ≈ 1536px
```

중요한 것은 px 숫자를 암기하는 것이 아니라:

```text
sm < md < lg < xl < 2xl
```

이라는 단계와 **min-width 방식**을 이해하는 것입니다.

또한 breakpoint 값은 theme customization에 따라 변경할 수 있습니다.

---

# 4. Breakpoint Variant의 구조

예:

```text
md:text-xl
```

분해하면:

```text
md : text-xl
│       │
│       └─ Utility
│
└───────── Variant
```

즉:

```text
variant : utility
```

구조입니다.

다른 예:

```text
lg:bg-blue-600

lg
→ breakpoint variant

bg-blue-600
→ Utility
```

또:

```text
md:px-8
```

은:

```text
md 이상에서
좌우 padding을 px-8로 적용
```

이라는 뜻입니다.

---

# 5. CSS Media Query와 비교

Tailwind의 Responsive Variant는 결국 CSS media query에 대응하는 방식으로 이해할 수 있습니다.

일반 CSS:

```css
.card {
  width: 100%;
}

@media (min-width: 48rem) {
  .card {
    width: 50%;
  }
}
```

Tailwind:

```jsx
<div className="w-full md:w-1/2">
```

개념적으로:

```text
w-full
   ↓
width: 100%

md:w-1/2
   ↓
md 이상
   ↓
width: 50%
```

즉 Tailwind에서는 media query를 매번 직접 작성하는 대신 **variant를 Utility에 붙여 표현**할 수 있습니다.

---

# 6. `sm:`은 모바일을 의미하지 않는다

초보자가 가장 많이 하는 오해입니다.

```text
sm = mobile
md = tablet
lg = desktop
```

라고 외우면 안 됩니다.

`sm:`의 정확한 의미는:

> **sm breakpoint 이상의 viewport**

입니다.

예:

```jsx
<div className="hidden sm:block">
  Menu
</div>
```

동작:

```text
sm 미만
hidden

sm 이상
block
```

즉 `sm:`은 “모바일 스타일”이라는 의미가 아닙니다.

---

# 7. Breakpoint는 누적된다

다음 코드를 보겠습니다.

```jsx
<h1 className="
  text-xl
  md:text-3xl
  lg:text-5xl
">
  Tailwind CSS
</h1>
```

동작은:

```text
작은 화면
└─ text-xl

md 이상
└─ text-3xl

lg 이상
└─ text-5xl
```

`lg:`는 lg에서만 적용되는 것이 아닙니다.

```text
lg 이상
```

에서 계속 적용됩니다.

따라서:

```text
        md              lg
────────│───────────────│──────────────>

text-xl
        text-3xl
                        text-5xl
```

처럼 이해하면 됩니다.

---

# 8. Responsive Typography

Typography부터 적용해 보겠습니다.

```jsx
<h1 className="
  text-2xl
  md:text-4xl
  lg:text-5xl
  font-bold
">
  Tailwind CSS
</h1>
```

결과:

```text
Mobile
Tailwind CSS
text-2xl

Tablet+
Tailwind CSS
text-4xl

Desktop+
Tailwind CSS
text-5xl
```

`font-bold`에는 breakpoint가 없습니다.

따라서 모든 화면에서 적용됩니다.

```text
font-bold
→ 항상 적용

text-2xl
→ 기본

md:text-4xl
→ md 이상

lg:text-5xl
→ lg 이상
```

---

# 9. Responsive Spacing

Spacing도 화면 크기에 따라 바꿀 수 있습니다.

```jsx
<div className="
  p-4
  md:p-6
  lg:p-8
">
  Content
</div>
```

동작:

```text
기본
p-4

md 이상
p-6

lg 이상
p-8
```

또는 좌우 padding만 변경할 수 있습니다.

```jsx
<div className="
  px-4
  md:px-8
  lg:px-12
">
```

Responsive Design에서는 화면이 커질수록 여백을 조금씩 증가시키는 패턴을 자주 사용합니다.

---

# 10. Responsive Width

Width 역시 매우 자주 변경합니다.

```jsx
<div className="
  w-full
  md:w-1/2
  lg:w-1/3
">
```

의미:

```text
기본
100%

md 이상
50%

lg 이상
33.333...%
```

다만 실제 여러 column 레이아웃에서는 Grid 또는 Flexbox를 사용하는 경우가 더 많습니다.

---

# 11. Responsive Grid

Responsive Design에서 가장 대표적인 패턴 중 하나입니다.

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
  gap-6
">
```

결과:

```text
Mobile

┌──────────────┐
│      A       │
└──────────────┘
┌──────────────┐
│      B       │
└──────────────┘
┌──────────────┐
│      C       │
└──────────────┘
```

```text
md 이상

┌───────┐ ┌───────┐
│   A   │ │   B   │
└───────┘ └───────┘
┌───────┐
│   C   │
└───────┘
```

```text
lg 이상

┌───────┐ ┌───────┐ ┌───────┐
│   A   │ │   B   │ │   C   │
└───────┘ └───────┘ └───────┘
```

이 패턴은 상품 목록, 게시글 목록, Dashboard Card 등에 매우 자주 사용됩니다.

---

# 12. Responsive Flex Direction

Flexbox의 방향도 변경할 수 있습니다.

```jsx
<div className="
  flex
  flex-col
  md:flex-row
  gap-4
">
```

동작:

```text
작은 화면

A
↓
B
↓
C
```

`flex-col`

```text
md 이상

A → B → C
```

`flex-row`

따라서 모바일에서는 세로로 쌓고 넓은 화면에서는 가로로 배치하는 UI를 쉽게 만들 수 있습니다.

---

# 13. Responsive Display

요소 자체를 특정 화면 크기에서 표시하거나 숨길 수도 있습니다.

예:

```jsx
<div className="hidden md:block">
  Desktop Navigation
</div>
```

동작:

```text
md 미만
hidden

md 이상
block
```

반대로:

```jsx
<div className="block md:hidden">
  Mobile Menu
</div>
```

은:

```text
md 미만
block

md 이상
hidden
```

입니다.

두 패턴을 함께 사용하면:

```text
작은 화면
Mobile Menu 표시

md 이상
Desktop Navigation 표시
```

같은 UI를 만들 수 있습니다.

---

# 14. 여러 Responsive Utility 조합하기

Responsive Variant는 하나의 요소에 여러 번 사용할 수 있습니다.

```jsx
<div
  className="
    w-full
    p-4
    text-sm

    md:w-1/2
    md:p-6
    md:text-base

    lg:w-1/3
    lg:p-8
    lg:text-lg
  "
>
  Product
</div>
```

이 코드는 세 가지를 동시에 변경합니다.

```text
           기본       md 이상      lg 이상

Width      100%        50%          33.3%
Padding    p-4         p-6          p-8
Text       text-sm     text-base    text-lg
```

이것이 Tailwind Responsive Design의 핵심 패턴입니다.

---

# 15. 같은 breakpoint를 묶어서 읽기

코드를 읽을 때 breakpoint별로 묶어서 생각하면 이해하기 쉽습니다.

```jsx
<div className="
  grid
  grid-cols-1
  gap-4

  md:grid-cols-2
  md:gap-6

  lg:grid-cols-3
  lg:gap-8
">
```

다음처럼 읽습니다.

```text
기본
├─ grid
├─ 1 column
└─ gap-4

md 이상
├─ 2 columns
└─ gap-6

lg 이상
├─ 3 columns
└─ gap-8
```

즉 Utility를 하나씩 따로 보는 것보다 **breakpoint별 UI 상태**로 읽는 것이 좋습니다.

---

# 16. Responsive Product Card Grid

실전 예제를 만들어 보겠습니다.

```jsx
const products = [
  { id: 1, name: 'Keyboard', price: '129,000원' },
  { id: 2, name: 'Mouse', price: '69,000원' },
  { id: 3, name: 'Monitor', price: '329,000원' },
]

export default function ProductList() {
  return (
    <main className="px-4 py-8 md:px-8 lg:px-12">
      <h1
        className="
          text-2xl
          md:text-3xl
          lg:text-4xl
          font-bold
        "
      >
        Products
      </h1>

      <div
        className="
          mt-6
          grid
          grid-cols-1
          md:grid-cols-2
          lg:grid-cols-3
          gap-6
        "
      >
        {products.map((product) => (
          <article
            key={product.id}
            className="
              p-6
              bg-white
              border
              border-gray-200
              rounded-xl
              shadow-sm
            "
          >
            <h2 className="text-xl font-semibold">
              {product.name}
            </h2>

            <p className="mt-2 text-gray-600">
              {product.price}
            </p>
          </article>
        ))}
      </div>
    </main>
  )
}
```

이 예제에서 Responsive 부분만 추려보면:

```text
Page Padding

px-4
md:px-8
lg:px-12
```

```text
Title

text-2xl
md:text-3xl
lg:text-4xl
```

```text
Grid

grid-cols-1
md:grid-cols-2
lg:grid-cols-3
```

즉 하나의 페이지 안에서도 여러 스타일을 동일한 breakpoint 전략으로 변경할 수 있습니다.

---

# 17. Mobile First로 설계하는 순서

Responsive UI를 만들 때 처음부터 모든 breakpoint를 작성하지 않는 것이 좋습니다.

먼저 가장 작은 화면을 설계합니다.

```text
STEP 1

Mobile UI
```

예:

```jsx
<div className="grid grid-cols-1 gap-4">
```

그다음 화면을 넓혀 봅니다.

```text
STEP 2

Tablet에서 문제가 있는가?
```

필요하면:

```jsx
md:grid-cols-2
```

를 추가합니다.

다시 화면을 넓힙니다.

```text
STEP 3

Desktop에서 공간을 더 활용할 수 있는가?
```

필요하면:

```jsx
lg:grid-cols-3
```

을 추가합니다.

최종:

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
">
```

이 방식이 Mobile First 설계입니다.

---

# 18. Breakpoint를 무조건 많이 쓰지 않는다

다음과 같이 모든 breakpoint를 채울 필요는 없습니다.

```jsx
className="
  text-base
  sm:text-lg
  md:text-xl
  lg:text-2xl
  xl:text-3xl
  2xl:text-4xl
"
```

UI에 실제 변화가 필요한 지점만 사용하는 것이 좋습니다.

예:

```jsx
className="
  text-2xl
  md:text-3xl
  lg:text-4xl
"
```

핵심은:

> **Device 이름에 맞춰 breakpoint를 사용하는 것이 아니라, UI가 깨지거나 레이아웃 변경이 필요한 지점에서 breakpoint를 사용합니다.**

입니다.

---

# 19. `max-*` Responsive Variant

Tailwind에서는 min-width 방식뿐 아니라 상한 조건을 표현할 수도 있습니다.

예:

```jsx
<div className="md:max-lg:flex">
```

개념적으로:

```text
md 이상
AND
lg 미만
```

범위에서 적용됩니다.

즉:

```text
       md               lg
────────│════════════════│────────>
        적용 구간
```

입니다.

하지만 대부분의 기본 Responsive UI는 Mobile First의:

```text
기본
md:
lg:
```

방식으로 충분합니다.

범위 variant는 특정 구간만 별도로 처리해야 할 때 사용하면 됩니다.

---

# 20. 특정 breakpoint 아래에서만 적용하기

예:

```jsx
<div className="max-md:hidden">
```

의미:

```text
md 미만
→ hidden
```

반대로:

```jsx
<div className="md:block">
```

은:

```text
md 이상
→ block
```

입니다.

따라서 조건을 다음처럼 생각할 수 있습니다.

```text
md:block
→ min-width 조건

max-md:hidden
→ max-width 조건
```

다만 초급 단계에서는 **기본 스타일 + `md:` + `lg:`** 구조를 먼저 충분히 익히는 것이 중요합니다.

---

# 21. Responsive Variant와 State Variant 조합

Breakpoint와 다른 variant를 함께 사용할 수도 있습니다.

예:

```jsx
<button className="
  bg-blue-500
  md:hover:bg-blue-700
">
  Button
</button>
```

구조:

```text
md : hover : bg-blue-700
│      │          │
│      │          └─ Utility
│      └──────────── State Variant
└─────────────────── Breakpoint Variant
```

즉 여러 조건을 조합할 수 있습니다.

다만 `hover:`, `focus:`, `active:` 등의 State Variant는 **PART 5에서 본격적으로 다룹니다.**

이번에는:

> **Responsive Variant도 다른 Variant와 조합할 수 있다.**

정도만 기억하면 됩니다.

---

# 22. 자주 사용하는 Responsive 패턴

### 페이지 Container

```jsx
<div className="
  max-w-7xl
  mx-auto
  px-4
  md:px-6
  lg:px-8
">
```

### 제목

```jsx
<h1 className="
  text-2xl
  md:text-3xl
  lg:text-4xl
  font-bold
">
```

### Card Grid

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
  gap-6
">
```

### Mobile → Desktop 방향 변경

```jsx
<div className="
  flex
  flex-col
  md:flex-row
">
```

### Mobile Menu

```jsx
<div className="md:hidden">
```

### Desktop Navigation

```jsx
<nav className="hidden md:block">
```

이 정도 패턴만 익혀도 대부분의 기본 Responsive UI를 만들 수 있습니다.

---

# 23. 자주 하는 실수 ① `sm:`을 모바일이라고 생각하기

잘못된 이해:

```text
sm:
→ 모바일
```

정확한 이해:

```text
sm:
→ sm breakpoint 이상
```

모바일 스타일은 일반적으로 prefix 없이 작성합니다.

```jsx
text-sm md:text-base
```

```text
text-sm
→ 기본

md:text-base
→ md 이상
```

---

# 24. 자주 하는 실수 ② breakpoint를 구간으로 생각하기

다음:

```jsx
md:text-xl
```

을:

```text
md 구간에서만 text-xl
```

이라고 생각하면 안 됩니다.

실제로는:

```text
md 이상에서 text-xl
```

입니다.

더 큰 breakpoint에서 다른 값으로 덮어쓰지 않는 한 계속 적용됩니다.

---

# 25. 자주 하는 실수 ③ HTML 순서와 화면 순서를 혼동하기

Responsive Layout을 바꾸더라도 DOM의 의미 있는 순서를 먼저 고려해야 합니다.

예:

```text
DOM

Heading
Content
Button
```

을 단순히 화면에서 보기 좋게 만들기 위해 무리하게 순서를 바꾸면 keyboard navigation이나 screen reader 경험에 영향을 줄 수 있습니다.

따라서:

> **Responsive Design은 시각적 배치뿐 아니라 콘텐츠의 논리적 순서와 접근성도 함께 고려해야 합니다.**

---

# 26. 자주 하는 실수 ④ breakpoint를 기기 이름으로 고정하기

다음처럼 생각하지 않는 것이 좋습니다.

```text
sm = 스마트폰
md = 태블릿
lg = 노트북
```

정확하게는:

```text
sm / md / lg
→ viewport width 조건
```

입니다.

기기 종류가 아니라 **UI가 자연스럽게 변화해야 하는 지점**을 기준으로 사용합니다.

---

# 27. Responsive Design 전체 흐름

Tailwind Responsive Design을 하나의 흐름으로 정리하면:

```text
작은 화면의 UI 먼저 작성
          ↓
prefix 없는 Utility
          ↓
viewport가 넓어짐
          ↓
Breakpoint 도달
          ↓
md: / lg: 등의 Utility 활성화
          ↓
기존 스타일 일부 변경
          ↓
새로운 Responsive Layout
```

예:

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
">
```

은:

```text
Mobile First
     ↓
1 column
     ↓
md
     ↓
2 columns
     ↓
lg
     ↓
3 columns
```

이라는 흐름입니다.

---

# 28. PART 4 핵심 정리

반드시 기억해야 할 것은 다음입니다.

```text
Responsive Utility

Breakpoint Variant
       +
     Utility

md : grid-cols-2
│          │
│          └─ Utility
└──────────── Breakpoint
```

그리고:

```text
prefix 없음
→ 기본 스타일

sm:
→ sm 이상

md:
→ md 이상

lg:
→ lg 이상

xl:
→ xl 이상

2xl:
→ 2xl 이상
```

Tailwind Responsive Design의 가장 중요한 원칙은:

> **Mobile First — 작은 화면의 기본 스타일을 먼저 작성하고, 화면이 넓어질 때 필요한 스타일만 breakpoint variant로 추가합니다.**

마지막으로 breakpoint는 특정 기기의 이름이 아닙니다.

> **화면 크기가 아니라 UI가 변해야 하는 지점을 기준으로 breakpoint를 선택합니다.**

---

# 다음 PART

## PART 5. State & Variant

다음 PART에서는 사용자 인터랙션과 상태에 따라 스타일을 변경하는 방법을 학습합니다.

```text
hover:
focus:
focus-visible:
active:
disabled:
checked:
first:
last:
odd:
even:
group-hover:
peer-checked:
```

예:

```jsx
<button
  className="
    bg-blue-500
    hover:bg-blue-600
    active:bg-blue-700
    focus-visible:ring-2
    disabled:opacity-50
  "
>
  저장
</button>
```

PART 4에서 배운:

```text
md:
lg:
```

와 같은 Responsive Variant와도 조합할 수 있습니다.

```jsx
md:hover:bg-blue-600
```

즉 다음 PART에서는 Tailwind의 중요한 사고방식인:

```text
Variant : Utility
```

를 Responsive를 넘어 **사용자 상태와 UI 상태 전체로 확장**합니다.
