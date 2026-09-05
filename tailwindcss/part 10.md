# PART 10. 실전 프로젝트 & Best Practices

## Tailwind CSS를 실제 React 프로젝트에서 어떻게 사용할 것인가?

지금까지 Tailwind CSS의 핵심 기능을 단계적으로 학습했습니다.

```text
PART 1
Tailwind CSS 개념

PART 2
Vite + React 설치와 연결

PART 3
Core Utility

PART 4
Responsive Design

PART 5
State & Variant

PART 6
Flexbox & Grid

PART 7
Sizing & Spacing

PART 8
Typography & Visual Styling

PART 9
Theme & Design Token
```

이제 마지막 PART에서는 이 모든 내용을 하나의 실제 프로젝트 안에서 연결합니다.

핵심 질문은 다음과 같습니다.

> Tailwind Utility를 실제 대규모 React 프로젝트에서 어떻게 구성해야 코드가 복잡해지지 않고 유지보수가 쉬울까?

---

# 1. Tailwind를 잘 쓴다는 것

Tailwind를 잘 사용한다는 것은 Utility를 많이 알고 있다는 뜻이 아닙니다.

잘못된 접근:

```text
Utility 많이 암기
      ↓
className 많이 작성
      ↓
Tailwind를 잘 사용
```

실제 중요한 것은:

```text
CSS 원리 이해
      ↓
적절한 Utility 선택
      ↓
Layout과 Responsive 설계
      ↓
Component 재사용
      ↓
Design Token 공유
      ↓
일관된 UI
```

입니다.

즉:

> **Tailwind는 CSS 지식을 대체하는 도구가 아니라 CSS를 빠르고 일관되게 표현하기 위한 도구입니다.**

---

# 2. 실전 프로젝트 구조

예를 들어 쇼핑몰 React 프로젝트를 생각해 보겠습니다.

```text
src/
│
├─ components/
│   ├─ layout/
│   │   ├─ Header.jsx
│   │   ├─ Sidebar.jsx
│   │   ├─ Footer.jsx
│   │   └─ AppLayout.jsx
│   │
│   └─ ui/
│       ├─ Button.jsx
│       ├─ Input.jsx
│       ├─ Badge.jsx
│       ├─ Modal.jsx
│       └─ Spinner.jsx
│
├─ features/
│   ├─ product/
│   │   ├─ ProductCard.jsx
│   │   ├─ ProductGrid.jsx
│   │   └─ ProductPage.jsx
│   │
│   ├─ cart/
│   └─ auth/
│
├─ styles/
│   └─ index.css
│
├─ App.jsx
└─ main.jsx
```

Tailwind를 사용한다고 해서 프로젝트 구조가 사라지는 것은 아닙니다.

오히려:

```text
Component 구조
+
Tailwind Utility
+
Design Token
```

이 함께 동작해야 합니다.

---

# 3. Layout Component

전체 페이지 구조부터 만들어 보겠습니다.

```jsx
export default function AppLayout({ children }) {
  return (
    <div className="min-h-screen bg-gray-50">
      <Header />

      <div className="flex">
        <Sidebar />

        <main className="min-w-0 grow">
          {children}
        </main>
      </div>

      <Footer />
    </div>
  )
}
```

여기에는 PART 6과 PART 7에서 배운 내용이 들어 있습니다.

```text
min-h-screen
→ 최소 viewport 높이

flex
→ Sidebar + Content

grow
→ Main이 남는 공간 차지

min-w-0
→ Content overflow 문제 방지
```

---

# 4. Content Container

실제 Content는 너무 넓어지지 않도록 제한합니다.

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
  ...
</div>
```

관계:

```text
w-full
→ 사용 가능한 width 활용

max-w-7xl
→ 최대 width 제한

mx-auto
→ 좌우 중앙 배치

px-*
→ viewport 가장자리와 Content 사이 공간

py-*
→ Section의 상하 spacing
```

이 패턴은 많은 페이지에서 반복해서 사용할 수 있습니다.

---

# 5. Header 실전

```jsx
export default function Header() {
  return (
    <header className="border-b bg-white">
      <div
        className="
          mx-auto
          flex
          h-16
          max-w-7xl
          items-center
          justify-between
          px-4
          sm:px-6
          lg:px-8
        "
      >
        <a
          href="/"
          className="
            text-xl
            font-bold
            text-gray-900
          "
        >
          MyShop
        </a>

        <nav
          className="
            hidden
            items-center
            gap-6
            md:flex
          "
        >
          <a href="/products">상품</a>
          <a href="/categories">카테고리</a>
          <a href="/about">소개</a>
        </nav>

        <button
          className="
            rounded-lg
            p-2
            md:hidden
          "
          aria-label="메뉴 열기"
        >
          ☰
        </button>
      </div>
    </header>
  )
}
```

여기에는:

```text
Flexbox
Responsive Variant
Sizing
Spacing
Typography
Accessibility
```

가 동시에 사용됩니다.

---

# 6. Responsive는 Component 안에서 해결한다

Desktop용 Component와 Mobile용 Component를 무조건 별도로 만들 필요는 없습니다.

예:

```jsx
<nav className="hidden md:flex">
```

```jsx
<button className="md:hidden">
```

같은 패턴으로 같은 Header 안에서 Responsive visibility를 제어할 수 있습니다.

하지만 UI 구조 자체가 완전히 달라진다면 별도 Component가 더 자연스러울 수도 있습니다.

즉:

```text
단순한 style/layout 변화
→ Responsive Variant

Markup/behavior 자체가 크게 다름
→ Component 분리 고려
```

입니다.

---

# 7. Product Grid

```jsx
export default function ProductGrid({ products }) {
  return (
    <section
      className="
        grid
        grid-cols-1
        gap-6
        sm:grid-cols-2
        lg:grid-cols-4
      "
    >
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
        />
      ))}
    </section>
  )
}
```

PART 4와 PART 6의 내용이 그대로 연결됩니다.

```text
base
1 column

sm 이상
2 columns

lg 이상
4 columns
```

중요:

> column 수는 기기 이름이 아니라 Content가 자연스럽게 배치되는 viewport 구간을 기준으로 결정합니다.

---

# 8. Product Card

```jsx
export default function ProductCard({ product }) {
  return (
    <article
      className="
        group
        overflow-hidden
        rounded-xl
        border
        border-gray-200
        bg-white
        shadow-sm

        transition-shadow
        hover:shadow-lg
      "
    >
      <div className="overflow-hidden">
        <img
          src={product.image}
          alt={product.name}
          className="
            aspect-square
            w-full
            object-cover

            transition-transform
            duration-300
            group-hover:scale-105
          "
        />
      </div>

      <div className="p-4">
        <p
          className="
            text-sm
            text-gray-500
          "
        >
          {product.category}
        </p>

        <h3
          className="
            mt-1
            truncate
            text-lg
            font-semibold
            text-gray-900

            group-hover:text-blue-600
          "
        >
          {product.name}
        </h3>

        <p
          className="
            mt-3
            text-xl
            font-bold
            text-blue-600
          "
        >
          {product.price.toLocaleString()}원
        </p>

        <button
          className="
            mt-4
            w-full
            rounded-lg
            bg-blue-600
            px-4
            py-2.5

            text-sm
            font-semibold
            text-white

            transition

            hover:bg-blue-700
            active:bg-blue-800

            focus-visible:outline-none
            focus-visible:ring-2
            focus-visible:ring-blue-500
            focus-visible:ring-offset-2
          "
        >
          장바구니 담기
        </button>
      </div>
    </article>
  )
}
```

이 하나의 Component 안에 PART 3~8의 거의 모든 내용이 들어 있습니다.

---

# 9. 긴 `className`은 문제인가?

Tailwind를 사용하면 이런 코드가 생길 수 있습니다.

```jsx
<button
  className="
    rounded-lg
    bg-blue-600
    px-4
    py-2
    text-sm
    font-semibold
    text-white
    hover:bg-blue-700
    focus-visible:ring-2
  "
>
```

초보자가 자주 하는 생각:

> className이 길다 → 나쁜 코드

항상 그런 것은 아닙니다.

중요한 것은:

```text
Utility가 길다
≠
복잡한 코드
```

입니다.

오히려 이 코드에서는 버튼의 시각적 스타일을 한 위치에서 바로 확인할 수 있습니다.

---

# 10. className이 정말 문제가 되는 경우

문제는 길이 자체보다 다음과 같은 경우입니다.

```text
같은 조합이 반복됨

조건부 className이 지나치게 복잡함

Component 역할이 불분명함

Design Token 없이 값이 제각각임

Variant 조합이 지나치게 길어짐
```

예:

```jsx
className={
  active
    ? loading
      ? disabled
        ? '...'
        : '...'
      : '...'
    : '...'
}
```

이런 코드는 구조를 다시 생각해야 합니다.

---

# 11. 반복되는 UI는 React Component로 추출

예를 들어 버튼 스타일이 여러 곳에서 반복됩니다.

```jsx
<button className="...">
  저장
</button>

<button className="...">
  수정
</button>

<button className="...">
  구매
</button>
```

이 경우:

```jsx
function Button({
  children,
  type = 'button',
  disabled = false,
}) {
  return (
    <button
      type={type}
      disabled={disabled}
      className="
        rounded-lg
        bg-blue-600
        px-4
        py-2
        font-semibold
        text-white

        hover:bg-blue-700

        focus-visible:outline-none
        focus-visible:ring-2
        focus-visible:ring-blue-500

        disabled:cursor-not-allowed
        disabled:opacity-50
      "
    >
      {children}
    </button>
  )
}
```

사용:

```jsx
<Button>
  저장
</Button>
```

이 더 자연스럽습니다.

---

# 12. Utility를 CSS class로 무조건 추출하지 않는다

다음처럼:

```css
.my-button {
  ...
}
```

으로 모든 Tailwind Utility를 다시 CSS class로 옮기면 Tailwind의 장점이 약해질 수 있습니다.

```text
Tailwind

Utility 조합
    ↓

다시 CSS class 생성
    ↓

기존 CSS 구조로 회귀
```

따라서 먼저 다음 순서를 고려합니다.

```text
Utility 직접 사용
      ↓
반복되는 UI인가?
      ↓
React Component 추출
      ↓
그래도 CSS abstraction이 필요한가?
      ↓
Custom Utility / Component Layer 검토
```

---

# 13. Class를 역할별로 읽기

긴 className은 카테고리별로 읽으면 이해하기 쉽습니다.

```jsx
className="
  flex
  items-center
  justify-between

  w-full
  max-w-7xl

  px-4
  py-3

  bg-white
  border-b

  md:px-6
  lg:px-8
"
```

다음처럼 읽습니다.

```text
Layout
flex
items-center
justify-between

Sizing
w-full
max-w-7xl

Spacing
px-4
py-3

Visual
bg-white
border-b

Responsive
md:px-6
lg:px-8
```

이 습관을 들이면 className이 길어져도 구조를 쉽게 이해할 수 있습니다.

---

# 14. className 순서를 일관되게

프로젝트 전체에서 Utility 순서를 일정하게 유지하면 코드 읽기가 쉬워집니다.

예:

```text
1. Layout
2. Position
3. Sizing
4. Spacing
5. Typography
6. Background / Border
7. Effects
8. State
9. Responsive
```

정답은 하나가 아닙니다.

중요한 것은 **팀에서 일관된 규칙을 사용하는 것**입니다.

---

# 15. 의미 없는 arbitrary value 남용 피하기

나쁜 예:

```jsx
<div
  className="
    mt-[13px]
    p-[17px]
    gap-[11px]
    rounded-[9px]
  "
>
```

이렇게 값이 계속 등장한다면 Design System이 없는 것입니다.

가능하면:

```jsx
<div
  className="
    mt-3
    p-4
    gap-3
    rounded-lg
  "
>
```

처럼 theme scale을 우선 사용합니다.

---

# 16. Arbitrary Value가 필요한 경우

반대로 arbitrary value가 항상 나쁜 것도 아닙니다.

예:

```jsx
<div className="top-[117px]">
```

처럼 특정 디자인 요구사항에만 필요한 값이 있을 수 있습니다.

판단:

```text
한 번만 사용하는 특수 값
→ Arbitrary Value

반복되는 디자인 규칙
→ Theme Token
```

입니다.

---

# 17. Theme Token을 적극 활용

PART 9에서 만든 Theme:

```css
@theme {
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;
}
```

Component:

```jsx
<button
  className="
    bg-brand-600
    text-white
    hover:bg-brand-700
  "
>
```

이렇게 사용하면 Component가 특정 raw color보다 **프로젝트의 Design System**을 사용하게 됩니다.

---

# 18. Raw Color를 Component마다 직접 결정하지 않기

예:

```jsx
<ProductCard className="text-blue-500" />

<CheckoutButton className="bg-sky-600" />

<LoginButton className="bg-indigo-600" />
```

각 개발자가 임의로 선택하면 UI가 쉽게 불일치합니다.

Design System:

```text
Primary
Danger
Success
Muted
Surface
```

등의 역할을 먼저 정의하는 것이 좋습니다.

---

# 19. Responsive Utility도 필요한 곳에만

나쁜 예:

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

실제 디자인 요구사항 없이 모든 breakpoint를 채울 필요는 없습니다.

좋은 예:

```jsx
className="
  text-2xl
  md:text-3xl
  lg:text-4xl
"
```

핵심:

> **UI가 실제로 변화해야 하는 지점에서만 breakpoint를 사용합니다.**

---

# 20. Mobile First 사고방식 유지

기본:

```jsx
grid-cols-1
```

필요하면:

```jsx
md:grid-cols-2
```

더 넓은 viewport에서 필요하면:

```jsx
lg:grid-cols-4
```

전체:

```jsx
className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-4
"
```

Mobile First의 핵심은:

```text
모바일 전용 CSS 작성
```

이 아니라:

```text
prefix 없는 기본 스타일
        ↓
min-width Variant로 단계적 변경
```

입니다.

---

# 21. Responsive 테스트

Browser DevTools의 Responsive Mode만 보고 끝내지 않습니다.

확인:

```text
좁은 viewport
중간 viewport
넓은 viewport

긴 텍스트
짧은 텍스트

상품 1개
상품 100개

이미지 있음
이미지 없음

Loading
Error
Empty
```

실제 UI는 viewport 크기뿐 아니라 **Content 변화에도 견뎌야 합니다.**

---

# 22. Responsive Design은 Content 테스트가 중요하다

예를 들어:

```text
Apple
```

만 테스트한 Card와:

```text
Professional Wireless Noise Cancelling Headphones
```

같은 긴 상품명을 가진 Card는 다릅니다.

따라서:

```text
최소 Content
평균 Content
최대 Content
```

를 모두 테스트해야 합니다.

---

# 23. `min-w-0` 실전 적용

Flexbox에서 매우 중요합니다.

```jsx
<div className="flex">
  <img
    className="
      size-16
      shrink-0
    "
  />

  <div className="min-w-0 grow">
    <h3 className="truncate">
      매우 길고 긴 상품 이름...
    </h3>
  </div>
</div>
```

관계:

```text
shrink-0
→ Image 축소 방지

grow
→ Content 영역 확장

min-w-0
→ Content 영역이 필요한 만큼 줄어들 수 있도록 허용

truncate
→ 긴 Text 처리
```

이것이 실제 Flexbox UI에서 매우 자주 등장하는 패턴입니다.

---

# 24. `h-screen`보다 `min-h-screen`

일반적인 페이지 Layout:

```jsx
<div className="min-h-screen">
```

Content가 적으면 viewport 높이를 채우고:

```text
┌──────────────┐
│              │
│   Content    │
│              │
└──────────────┘
```

Content가 많아지면:

```text
┌──────────────┐
│ Content      │
│ Content      │
│ Content      │
│ Content      │
└──────────────┘
       ↓
페이지 확장
```

됩니다.

일반 문서형 페이지에서는 `min-h-screen`이 자연스러운 경우가 많습니다.

---

# 25. Loading UI

실전 프로젝트에서는 정상 데이터만 있는 것이 아닙니다.

```text
Pending
Fulfilled
Rejected
```

상태가 존재합니다.

예:

```jsx
if (isLoading) {
  return <ProductGridSkeleton />
}
```

Tailwind는 Skeleton UI의 styling에도 사용할 수 있습니다.

```jsx
<div className="animate-pulse">
  <div className="aspect-square rounded-xl bg-gray-200" />
  <div className="mt-4 h-4 rounded bg-gray-200" />
  <div className="mt-2 h-4 w-2/3 rounded bg-gray-200" />
</div>
```

---

# 26. Empty State

```jsx
if (products.length === 0) {
  return (
    <div
      className="
        flex
        min-h-64
        flex-col
        items-center
        justify-center
        text-center
      "
    >
      <h2 className="text-lg font-semibold">
        상품이 없습니다.
      </h2>

      <p className="mt-2 text-gray-500">
        다른 조건으로 검색해보세요.
      </p>
    </div>
  )
}
```

중요:

> 이것은 Tailwind의 `empty:` Variant와 다른 개념입니다.

```text
Application Data State
→ React conditional rendering

DOM :empty
→ Tailwind empty:
```

입니다.

---

# 27. Error State

```jsx
if (error) {
  return (
    <div
      className="
        rounded-lg
        border
        border-red-200
        bg-red-50
        p-4
        text-red-800
      "
    >
      상품을 불러오지 못했습니다.
    </div>
  )
}
```

Error UI에서는:

```text
Color
+
Text
+
필요하면 Icon
```

을 함께 사용합니다.

색상 하나만으로 의미를 전달하지 않는 것이 좋습니다.

---

# 28. Disabled State

```jsx
<button
  disabled={isLoading}
  className="
    rounded-lg
    bg-brand-600
    px-4
    py-2
    text-white

    hover:bg-brand-700

    disabled:cursor-not-allowed
    disabled:opacity-50
  "
>
  구매하기
</button>
```

중요:

```text
disabled={isLoading}
→ 실제 DOM 상태

disabled:opacity-50
→ 상태에 따른 styling
```

Tailwind Variant가 상태 자체를 만드는 것은 아닙니다.

---

# 29. Focus Accessibility

Keyboard 사용자는 현재 어느 요소에 focus되어 있는지 알아야 합니다.

```jsx
<button
  className="
    focus-visible:outline-none
    focus-visible:ring-2
    focus-visible:ring-brand-500
    focus-visible:ring-offset-2
  "
>
```

중요:

```text
outline 제거
+
대체 focus indicator 없음
```

은 피해야 합니다.

---

# 30. Semantic HTML 먼저

Tailwind를 사용한다고 해서 모든 요소를 `<div>`로 만들면 안 됩니다.

나쁜 예:

```jsx
<div
  onClick={handleSubmit}
  className="..."
>
  저장
</div>
```

버튼이라면:

```jsx
<button
  type="button"
  onClick={handleSubmit}
>
  저장
</button>
```

이 더 적절합니다.

CSS framework보다 먼저 **HTML semantics**를 생각해야 합니다.

---

# 31. Link와 Button 구분

Navigation:

```jsx
<a href="/products">
  상품
</a>
```

또는 React Router:

```jsx
<Link to="/products">
  상품
</Link>
```

Action:

```jsx
<button onClick={addToCart}>
  장바구니
</button>
```

즉:

```text
이동
→ Link

동작
→ Button
```

입니다.

Tailwind styling과 HTML 의미를 혼동하면 안 됩니다.

---

# 32. 접근성을 Color로만 해결하지 않는다

나쁜 예:

```text
●
```

빨강이면 오류, 초록이면 성공.

좋은 예:

```text
✓ 저장되었습니다.

! 저장에 실패했습니다.
```

Tailwind는 색상을 쉽게 적용할 수 있지만, **색상만으로 의미를 전달하지 않는 것**은 개발자의 설계 책임입니다.

---

# 33. 이미지에 `alt`

```jsx
<img
  src={product.image}
  alt={product.name}
/>
```

상품 이미지처럼 의미 있는 이미지는 적절한 `alt`를 제공합니다.

단순 장식 이미지라면 경우에 따라 빈 `alt=""`가 적절할 수 있습니다.

Tailwind와 상관없는 내용처럼 보이지만 실제 UI 품질에서는 매우 중요합니다.

---

# 34. className 조건 처리

간단한 조건:

```jsx
className={
  selected
    ? 'bg-brand-600 text-white'
    : 'bg-white text-gray-700'
}
```

도 충분할 수 있습니다.

조건이 많아지면 변수로 분리하는 것이 좋습니다.

```jsx
const buttonClass = selected
  ? 'bg-brand-600 text-white'
  : 'bg-white text-gray-700'
```

이후:

```jsx
<button className={buttonClass}>
```

처럼 사용할 수 있습니다.

---

# 35. Boolean 조건은 상태와 Style을 분리해서 생각

예:

```jsx
const isSoldOut = product.stock === 0
```

상태:

```text
isSoldOut
```

Style:

```jsx
className={
  isSoldOut
    ? 'opacity-50'
    : ''
}
```

Behavior:

```jsx
disabled={isSoldOut}
```

즉:

```text
Application State
      ↓
Behavior
+
Styling
```

을 분리해서 생각합니다.

---

# 36. Design Token과 Component Variant

예:

```jsx
<Button variant="primary">
  저장
</Button>

<Button variant="danger">
  삭제
</Button>
```

Component 내부:

```jsx
const variantClasses = {
  primary:
    'bg-brand-600 text-white hover:bg-brand-700',

  danger:
    'bg-red-600 text-white hover:bg-red-700',
}
```

이렇게 하면 Component 사용자는 raw color를 직접 고민하지 않아도 됩니다.

---

# 37. UI Component의 책임

예를 들어 Button Component가 담당할 수 있는 것:

```text
HTML button 구조

공통 Padding

Radius

Typography

Focus Style

Disabled Style

Color Variant
```

Application이 담당하는 것:

```text
Button을 언제 disabled 할지

클릭하면 어떤 동작을 할지

어떤 variant를 사용할지
```

관계를 분리하는 것이 좋습니다.

---

# 38. Feature Component와 UI Component 구분

예:

```text
components/ui/Button.jsx
→ 재사용 가능한 일반 Button

features/cart/AddToCartButton.jsx
→ 장바구니 business logic 포함
```

`AddToCartButton` 내부에서:

```jsx
<Button>
  장바구니
</Button>
```

을 사용할 수 있습니다.

Tailwind 프로젝트에서도 **UI와 Business Logic 분리**는 중요합니다.

---

# 39. Layout Component와 Page Component

```text
AppLayout
→ Header / Sidebar / Footer

ProductPage
→ 상품 화면

ProductGrid
→ Product 목록 Layout

ProductCard
→ 하나의 상품 표현
```

이렇게 역할을 나누면 Tailwind className도 자연스럽게 분산됩니다.

---

# 40. 거대한 JSX 하나로 만들지 않는다

나쁜 구조:

```jsx
function App() {
  return (
    <div>
      {/* Header 수십 줄 */}
      {/* Sidebar 수십 줄 */}
      {/* Product Grid */}
      {/* Card */}
      {/* Footer */}
    </div>
  )
}
```

좋은 구조:

```jsx
<AppLayout>
  <ProductPage />
</AppLayout>
```

그리고:

```jsx
<ProductGrid>
  <ProductCard />
</ProductGrid>
```

Component 분리는 Tailwind className의 복잡도도 줄여줍니다.

---

# 41. 모든 반복을 추상화할 필요는 없다

두 번 반복됐다고 바로 Component를 만들 필요는 없습니다.

```text
반복
      ↓

UI 의미가 같은가?
      ↓

변경 이유가 같은가?
      ↓

재사용 가치가 있는가?
      ↓

Component 추출
```

단순히 코드 줄 수를 줄이는 것이 Component abstraction의 목적은 아닙니다.

---

# 42. Tailwind Utility와 JavaScript를 무리하게 조립하지 않기

이런 코드는 주의가 필요합니다.

```jsx
className={`bg-${color}-500`}
```

Tailwind의 class detection 과정에서 동적으로 조합된 class name은 원하는 Utility가 생성되지 않을 수 있습니다.

따라서:

```jsx
const colors = {
  blue: 'bg-blue-500',
  red: 'bg-red-500',
  green: 'bg-green-500',
}
```

처럼 **완전한 class name을 정적으로 표현**하는 패턴이 안전합니다.

사용:

```jsx
<div className={colors[color]}>
```

---

# 43. 동적 className의 좋은 예

```jsx
const statusClasses = {
  success: 'bg-green-50 text-green-700',
  error: 'bg-red-50 text-red-700',
  warning: 'bg-amber-50 text-amber-700',
}
```

사용:

```jsx
<div className={statusClasses[status]}>
```

Tailwind가 source에서 완전한 Utility 이름을 확인할 수 있습니다.

---

# 44. Inline Style이 더 적절한 경우도 있다

예를 들어 사용자 데이터에서 직접 오는 색상:

```jsx
<div
  style={{
    backgroundColor: userColor,
  }}
/>
```

이런 값까지 억지로 Tailwind Utility로 만들 필요는 없습니다.

판단:

```text
Design System 값
→ Tailwind Theme

동적인 runtime 값
→ style prop을 고려
```

Tailwind와 inline style은 경쟁 관계가 아닙니다.

---

# 45. Tailwind만으로 모든 문제를 해결하려 하지 않는다

실제 React 프로젝트에서는:

```text
Tailwind
→ Styling

React
→ Component / State / Rendering

TanStack Query / RTK Query
→ Server State

React Router
→ Routing
```

처럼 각 도구의 역할이 다릅니다.

Tailwind가 해결하는 것은 **UI Styling**입니다.

---

# 46. Performance 관점

Tailwind를 사용할 때도 성능을 생각해야 합니다.

예:

```text
불필요한 대형 이미지

무거운 animation

과도한 shadow / blur

거대한 Component

불필요한 React re-render
```

등은 Tailwind 자체와 별개의 문제입니다.

Tailwind를 사용한다고 자동으로 빠른 앱이 되는 것은 아닙니다.

---

# 47. Animation 남용 피하기

예:

```jsx
hover:scale-125
duration-1000
```

같은 큰 interaction을 모든 요소에 적용하면 UX가 오히려 불편할 수 있습니다.

좋은 원칙:

```text
Interaction feedback은
짧고
명확하며
목적이 있어야 한다.
```

대부분의 UI에서는 작은:

```text
Color 변화
Shadow 변화
약간의 Scale 변화
```

만으로 충분합니다.

---

# 48. Transition은 필요한 property에

간단하게:

```jsx
transition
```

을 사용할 수 있지만 더 구체적으로:

```jsx
transition-colors
```

```jsx
transition-shadow
```

```jsx
transition-transform
```

처럼 실제 변경 property를 명시할 수도 있습니다.

UI의 의도가 더 명확해집니다.

---

# 49. Dark Mode도 Design System으로 접근

```jsx
className="
  bg-white
  text-gray-900

  dark:bg-gray-900
  dark:text-white
"
```

만으로 시작할 수 있습니다.

하지만 실제 프로젝트에서는:

```text
Surface
Border
Text
Muted
Primary
Hover
Focus
Disabled
```

전체 색상 관계를 확인해야 합니다.

Dark Mode는 단순 색 반전이 아닙니다.

---

# 50. 실전 전체 페이지 예제

```jsx
export default function ProductPage({
  products,
  isLoading,
  error,
}) {
  if (isLoading) {
    return <ProductGridSkeleton />
  }

  if (error) {
    return <ProductError />
  }

  if (products.length === 0) {
    return <ProductEmpty />
  }

  return (
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
      <div
        className="
          mb-8
          flex
          flex-col
          gap-4

          md:flex-row
          md:items-end
          md:justify-between
        "
      >
        <div>
          <p className="text-sm text-gray-500">
            Product
          </p>

          <h1
            className="
              text-3xl
              font-bold
              tracking-tight
              text-gray-900
            "
          >
            인기 상품
          </h1>
        </div>

        <ProductFilter />
      </div>

      <ProductGrid products={products} />
    </div>
  )
}
```

이제 실제 Page Component가 상당히 읽기 쉬워집니다.

---

# 51. 전체 구조

```text
App
│
└─ AppLayout
   │
   ├─ Header
   │
   ├─ Sidebar
   │
   ├─ ProductPage
   │    │
   │    ├─ ProductFilter
   │    │
   │    ├─ ProductGrid
   │    │    └─ ProductCard
   │    │
   │    ├─ ProductGridSkeleton
   │    ├─ ProductEmpty
   │    └─ ProductError
   │
   └─ Footer
```

Tailwind className도 각 Component의 책임에 맞게 자연스럽게 분산됩니다.

---

# 52. PART 1~9와 최종 프로젝트 연결

```text
PART 1
Tailwind 개념
        ↓

PART 2
React + Vite 설치
        ↓

PART 3
Core Utility
        ↓

PART 4
Responsive
        ↓

PART 5
State Variant
        ↓

PART 6
Flex / Grid
        ↓

PART 7
Sizing / Spacing
        ↓

PART 8
Typography / Visual
        ↓

PART 9
Theme / Design Token
        ↓

PART 10
Project Architecture
```

---

# 53. Tailwind Best Practices

핵심 원칙을 정리하면 다음과 같습니다.

```text
CSS 원리를 먼저 이해한다.

Utility를 의미에 맞게 선택한다.

Responsive는 필요한 breakpoint만 사용한다.

State와 Styling을 구분한다.

Semantic HTML을 먼저 선택한다.

Focus Accessibility를 유지한다.

Design Token을 사용한다.

Arbitrary Value를 남용하지 않는다.

반복되는 UI는 Component로 추출한다.

Tailwind를 모든 문제의 해결책으로 사용하지 않는다.
```

---

# 54. 추천 개발 순서

실제 UI를 만들 때 다음 순서를 추천합니다.

```text
① Semantic HTML

        ↓

② 기본 Layout

Flex / Grid

        ↓

③ Sizing / Spacing

        ↓

④ Typography

        ↓

⑤ Color / Border / Radius

        ↓

⑥ Responsive

        ↓

⑦ State Variant

        ↓

⑧ Accessibility

        ↓

⑨ Component 추출

        ↓

⑩ Theme / Design Token 정리
```

처음부터 모든 Utility와 Variant를 한꺼번에 넣지 않는 것이 좋습니다.

---

# 55. UI Debugging 순서

Layout이 이상할 때 무작정 className을 추가하지 않습니다.

먼저:

```text
① 부모 Layout Mode 확인

flex?
grid?
block?

        ↓

② 직접 자식 관계 확인

        ↓

③ width / min-width / max-width 확인

        ↓

④ grow / shrink / basis 확인

        ↓

⑤ margin / padding / gap 확인

        ↓

⑥ overflow 확인

        ↓

⑦ breakpoint 확인
```

이 순서로 보면 문제를 훨씬 빠르게 찾을 수 있습니다.

---

# 56. Flexbox Debugging

문제:

> `justify-center`가 원하는 방향으로 움직이지 않는다.

확인:

```text
flex-direction은?

        ↓

Main Axis 방향은?

        ↓

justify-*가 원하는 property인가?
```

문제:

> Item이 너무 줄어든다.

확인:

```text
shrink

shrink-0

basis

min-width
```

---

# 57. Grid Debugging

문제:

> Card가 원하는 개수로 나오지 않는다.

확인:

```text
grid 설정 여부

grid-cols-*

breakpoint

col-span-*

gap
```

문제:

> Item 위치가 이상하다.

확인:

```text
Grid Track

auto-placement

col-span

row-span
```

---

# 58. Responsive Debugging

문제:

> `md:`가 적용되지 않는다.

확인:

```text
md의 의미
→ min-width 조건

현재 viewport width

같은 CSS property를
더 큰 breakpoint가 override하고 있는가?
```

항상 기기 이름이 아니라 **viewport 조건**으로 생각합니다.

---

# 59. State Debugging

문제:

> `disabled:opacity-50`이 적용되지 않는다.

확인:

```text
실제 disabled attribute가 있는가?
```

문제:

> `peer-checked:*`가 동작하지 않는다.

확인:

```text
peer가 :checked 가능한 요소인가?

peer가 대상 요소보다 앞에 있는가?

같은 sibling context인가?
```

---

# 60. Tailwind 코드를 읽는 최종 방법

예:

```jsx
className="
  group
  flex
  min-w-0
  items-center
  gap-4
  rounded-xl
  border
  bg-white
  p-4
  shadow-sm

  hover:shadow-md

  md:p-6
"
```

다음 순서로 읽습니다.

```text
group
→ Relationship State의 기준

flex
→ Layout

min-w-0
→ Sizing constraint

items-center
→ Cross Axis Alignment

gap-4
→ Item spacing

rounded-xl
border
bg-white
shadow-sm
→ Visual Style

p-4
→ Internal Spacing

hover:shadow-md
→ State

md:p-6
→ Responsive
```

이제 className이 길어도 읽을 수 있습니다.

---

# 61. 최종 실전 사고방식

Tailwind Component를 만들 때:

```text
어떤 태그인가?
        ↓
HTML Semantics

어떻게 배치되는가?
        ↓
Flex / Grid

크기는?
        ↓
Sizing

공간은?
        ↓
Spacing

정보의 중요도는?
        ↓
Typography

어떻게 구분되는가?
        ↓
Color / Border / Shadow

어떤 상태가 있는가?
        ↓
Variant

어떤 viewport에서 변하는가?
        ↓
Responsive

반복되는 규칙인가?
        ↓
Theme / Component
```

이 사고 순서가 중요합니다.

---

# 62. PART 10 핵심 정리

Tailwind CSS를 실제 프로젝트에서 사용하는 전체 구조는 다음과 같습니다.

```text
CSS Fundamentals
       ↓
Tailwind Utility
       ↓
Responsive / State Variant
       ↓
Flexbox / Grid
       ↓
Sizing / Spacing
       ↓
Typography / Visual
       ↓
Theme / Design Token
       ↓
Reusable React Component
       ↓
Application UI
```

---

# 63. 마지막 핵심 메시지

Tailwind를 사용한다고 좋은 UI가 자동으로 만들어지는 것은 아닙니다.

```text
Tailwind
≠
Design

Tailwind
≠
Component Architecture

Tailwind
≠
Accessibility

Tailwind
≠
State Management
```

Tailwind가 하는 일은:

```text
Design Decision
       ↓
CSS
       ↓
Utility Class
       ↓
빠르고 일관되게 표현
```

입니다.

> **Tailwind CSS를 잘 사용한다는 것은 Utility를 많이 아는 것이 아니라, CSS와 UI 설계 원리를 이해하고 필요한 Utility만 적절한 Component에 조합하는 것입니다.**

---

# 전체 강의 최종 구조

```text
PART 1
Tailwind CSS란?

PART 2
React + Vite 환경 구성

PART 3
Core Utility

PART 4
Responsive Design

PART 5
State & Variant

PART 6
Flexbox & Grid

PART 7
Sizing & Spacing

PART 8
Typography, Color & Visual Styling

PART 9
Theme & Design Token

PART 10
실전 프로젝트 & Best Practices
```

이제:

```text
Utility를 사용하는 단계
        ↓
Responsive UI를 만드는 단계
        ↓
Component를 만드는 단계
        ↓
Design System을 만드는 단계
        ↓
실제 Application을 설계하는 단계
```

까지 연결됩니다.

# PART 10 이미지 구성 — 총 8장

```text
1/8
실전 프로젝트 전체 구조
+ PART 1~9 연결 지도

2/8
AppLayout
+ Header / Sidebar / Content / Footer
+ Responsive Layout

3/8
ProductGrid + ProductCard
+ Flex / Grid / Responsive 종합

4/8
Loading / Error / Empty / Disabled
+ Application State와 Tailwind 역할 구분

5/8
Reusable Component
+ Button / Input
+ className 재사용 전략

6/8
Best Practices
+ Utility 순서
+ Arbitrary Value
+ Dynamic Class
+ Theme Token

7/8
Semantic HTML + Accessibility
+ Focus
+ Link vs Button
+ Responsive/Content Testing

8/8
Tailwind CSS 전체 강의 최종 지도
+ 프로젝트 Architecture
+ Debugging 순서
+ 핵심 원칙 총정리
```
