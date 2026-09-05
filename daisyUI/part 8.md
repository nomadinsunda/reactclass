# PART 8. daisyUI 실전 UI 프로젝트

## 1. 이번 PART의 목표

PART 1~7에서는 daisyUI의 개별 개념과 Component를 학습했습니다.

```text
PART 1
daisyUI 기본 개념

PART 2
Component Class / Modifier

PART 3
Color / Theme

PART 4
Form

PART 5
Navigation / Layout

PART 6
Data Display

PART 7
Overlay / Feedback
```

이제 중요한 질문이 남습니다.

> **이 Component들을 실제 프로젝트에서는 어떻게 조합해야 하는가?**

PART 8에서는 개별 Component가 아니라 **Application UI 전체를 설계하는 방법**을 학습합니다.

실전 예제로 다음 세 가지 화면을 구성합니다.

```text
① 쇼핑몰

② 관리자 Dashboard

③ 게시판
```

이번 PART의 핵심은 다음입니다.

> **실제 애플리케이션은 하나의 daisyUI Component로 만드는 것이 아니라, 여러 Component와 Tailwind CSS Layout을 조합해서 만든다.**

---

# 2. 실전 UI를 보는 관점

완성된 웹 화면을 보면 복잡해 보입니다.

예:

```text
┌────────────────────────────────────────────┐
│ Navbar                                     │
├────────────────────────────────────────────┤
│ Hero                                       │
├──────────┬─────────────────────────────────┤
│ Filter   │ Product Grid                    │
│          │                                 │
│          │ Card   Card   Card              │
│          │ Card   Card   Card              │
├──────────┴─────────────────────────────────┤
│ Footer                                     │
└────────────────────────────────────────────┘
```

하지만 이를 Component 단위로 분해하면 단순해집니다.

```text
Application
│
├─ Navbar
├─ Hero
├─ Sidebar
│   └─ Menu / Form
├─ Product Grid
│   └─ Card
├─ Pagination
└─ Footer
```

즉 실전 UI 개발의 첫 단계는:

> **화면을 Component 단위로 분해하는 것**

입니다.

---

# 3. UI 설계 순서

실전에서는 다음 순서로 접근하는 것이 좋습니다.

```text
① 화면의 역할 결정
      ↓
② 큰 Layout 분해
      ↓
③ daisyUI Component 선택
      ↓
④ Tailwind CSS로 배치
      ↓
⑤ React로 데이터 연결
      ↓
⑥ State / Event 연결
      ↓
⑦ Loading / Error / Empty 처리
      ↓
⑧ Responsive 적용
```

이 순서를 기억하면 복잡한 화면도 단계적으로 만들 수 있습니다.

---

# 4. 역할 분담 다시 정리

실전 프로젝트에서는 다음 기술이 동시에 사용됩니다.

```text
React
→ Component
→ State
→ Event
→ Conditional Rendering


React Router
→ URL
→ Navigation
→ Layout Route


RTK Query / TanStack Query
→ Server State
→ Loading
→ Error
→ Mutation


daisyUI
→ UI Component
→ Theme
→ Variant


Tailwind CSS
→ Layout
→ Spacing
→ Responsive
→ 세부 Style
```

이 다섯 가지 역할을 혼동하지 않는 것이 중요합니다.

---

# 5. 프로젝트 1 — 쇼핑몰 UI

먼저 쇼핑몰을 만들어보겠습니다.

전체 화면:

```text
┌───────────────────────────────────────────────┐
│ Navbar                                        │
│ Logo   Search          Login   Cart           │
├───────────────────────────────────────────────┤
│ Breadcrumbs                                   │
├──────────────┬────────────────────────────────┤
│ Filter       │ Product Grid                   │
│              │                                │
│ Category     │ Card    Card    Card            │
│ Price        │ Card    Card    Card            │
│ Brand        │                                │
│              │ Pagination                     │
├──────────────┴────────────────────────────────┤
│ Footer                                        │
└───────────────────────────────────────────────┘
```

Mobile:

```text
┌─────────────────────────────┐
│ ☰  MyShop       Search Cart │
├─────────────────────────────┤
│ Breadcrumbs                 │
├─────────────────────────────┤
│ [ Product Card ]            │
│ [ Product Card ]            │
│ [ Product Card ]            │
├─────────────────────────────┤
│ Pagination                  │
├─────────────────────────────┤
│ Dock                        │
└─────────────────────────────┘
```

---

# 6. 쇼핑몰 Component 분해

```text
ShopPage
│
├─ ShopNavbar
│
├─ Breadcrumbs
│
├─ Drawer
│   ├─ FilterSidebar
│   │
│   └─ ProductArea
│       ├─ ProductToolbar
│       ├─ ProductGrid
│       │    └─ ProductCard × N
│       └─ Pagination
│
└─ Footer
```

이제 각각의 Component에 daisyUI를 적용합니다.

---

# 7. 쇼핑몰 Navbar

```jsx
function ShopNavbar() {
  return (
    <div className="navbar bg-base-100 shadow-sm">
      <div className="navbar-start">
        <a className="btn btn-ghost text-xl">
          MyShop
        </a>
      </div>

      <div className="navbar-center hidden lg:flex">
        <label className="input flex items-center gap-2">
          <input
            type="search"
            placeholder="상품 검색"
          />
        </label>
      </div>

      <div className="navbar-end gap-2">
        <button className="btn btn-ghost">
          Login
        </button>

        <button className="btn btn-primary">
          Cart
        </button>
      </div>
    </div>
  )
}
```

여기서:

```text
navbar
btn
input
→ daisyUI

hidden
lg:flex
gap-2
→ Tailwind CSS
```

입니다.

---

# 8. 상품 Filter Sidebar

Filter에는 PART 4에서 배운 Form Component를 사용합니다.

```jsx
function ProductFilter() {
  return (
    <div className="space-y-6">
      <fieldset className="fieldset">
        <legend className="fieldset-legend">
          Category
        </legend>

        <label className="flex gap-2">
          <input
            type="checkbox"
            className="checkbox"
          />
          Keyboard
        </label>

        <label className="flex gap-2">
          <input
            type="checkbox"
            className="checkbox"
          />
          Mouse
        </label>
      </fieldset>

      <fieldset className="fieldset">
        <legend className="fieldset-legend">
          Price
        </legend>

        <input
          type="range"
          min="0"
          max="500000"
          className="range"
        />
      </fieldset>
    </div>
  )
}
```

Component:

```text
fieldset
checkbox
range
```

를 조합하고 있습니다.

---

# 9. Product Grid

상품 Card를 Grid로 배치합니다.

```jsx
function ProductGrid({ products }) {
  return (
    <div
      className="
        grid
        grid-cols-1
        sm:grid-cols-2
        xl:grid-cols-3
        gap-6
      "
    >
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
        />
      ))}
    </div>
  )
}
```

여기서는 daisyUI가 거의 보이지 않습니다.

왜일까요?

```text
ProductGrid
→ Layout 역할

따라서
Tailwind CSS가 중심
```

이기 때문입니다.

이것이 실전 Component 설계에서 매우 중요합니다.

---

# 10. Product Card

```jsx
function ProductCard({ product }) {
  return (
    <div className="card bg-base-100 shadow-sm">
      <figure className="h-48">
        <img
          src={product.image}
          alt={product.name}
          className="h-full w-full object-cover"
        />
      </figure>

      <div className="card-body">
        <div className="flex justify-between">
          <h2 className="card-title">
            {product.name}
          </h2>

          {product.isNew && (
            <span className="badge badge-accent">
              NEW
            </span>
          )}
        </div>

        <p className="text-lg font-bold">
          {product.price.toLocaleString()}원
        </p>

        <div className="card-actions justify-end">
          <button className="btn btn-primary">
            장바구니
          </button>
        </div>
      </div>
    </div>
  )
}
```

역할:

```text
React
→ product 데이터 Rendering

daisyUI
→ card / badge / button

Tailwind
→ image 크기 / 정렬 / 간격
```

---

# 11. Product Card의 상태

상품에는 상태가 존재할 수 있습니다.

```text
NEW
SALE
품절
재고 부족
추천
```

이를 Badge로 표현할 수 있습니다.

```jsx
{product.stock === 0 ? (
  <span className="badge badge-error">
    품절
  </span>
) : product.stock < 5 ? (
  <span className="badge badge-warning">
    재고 부족
  </span>
) : (
  <span className="badge badge-success">
    판매 중
  </span>
)}
```

흐름:

```text
Server Data
    ↓
stock
    ↓
React 조건 판단
    ↓
Badge Variant
    ↓
daisyUI UI
```

---

# 12. 상품 상세 페이지

상품 상세 화면은 다음과 같이 구성할 수 있습니다.

```text
Breadcrumbs
   ↓
Product Detail

┌────────────────┬───────────────────┐
│ Product Image  │ Product Name      │
│                │ Badge             │
│                │ Price             │
│                │ Rating            │
│                │ Quantity          │
│                │ [Cart] [Buy]      │
└────────────────┴───────────────────┘

Tabs
├─ 상품 정보
├─ 리뷰
└─ Q&A
```

사용 Component:

```text
Breadcrumbs
Badge
Rating
Input
Button
Tabs
Card
```

---

# 13. 장바구니 추가 흐름

실제 Action까지 연결해 보겠습니다.

```text
[ 장바구니 ]
     ↓
Mutation
     ↓
Loading
     ↓
Success?
 ┌───┴────┐
Yes       No
 │         │
 ▼         ▼
Toast     Error Toast
```

UI 구성:

```jsx
<button
  className="btn btn-primary"
  disabled={isLoading}
>
  {isLoading && (
    <span className="loading loading-spinner loading-sm" />
  )}

  장바구니
</button>
```

Success:

```text
toast
+
alert-success
```

Error:

```text
toast
+
alert-error
```

---

# 14. 프로젝트 2 — 관리자 Dashboard

다음은 Dashboard입니다.

전체 구조:

```text
┌────────────────────────────────────────────┐
│ Navbar                                     │
├────────────┬───────────────────────────────┤
│ Sidebar    │ Dashboard                     │
│            │                               │
│ Menu       │ [Stat] [Stat] [Stat]          │
│            │                               │
│            │ Recent Orders                 │
│            │ ┌───────────────────────────┐ │
│            │ │ Table                     │ │
│            │ └───────────────────────────┘ │
└────────────┴───────────────────────────────┘
```

Mobile에서는 Sidebar가 Drawer로 바뀝니다.

---

# 15. Dashboard Component 구조

```text
AdminLayout
│
├─ Drawer
│
├─ Sidebar
│   └─ Menu
│
└─ drawer-content
    │
    ├─ AdminNavbar
    │
    └─ Main
        │
        ├─ Breadcrumbs
        ├─ DashboardStats
        └─ RecentOrderTable
```

PART 5와 PART 6의 내용을 그대로 조합합니다.

---

# 16. Dashboard Stat

```jsx
function DashboardStats({ data }) {
  return (
    <div className="stats stats-vertical lg:stats-horizontal shadow">
      <div className="stat">
        <div className="stat-title">
          Revenue
        </div>

        <div className="stat-value">
          {data.revenue.toLocaleString()}원
        </div>

        <div className="stat-desc">
          +12% from last month
        </div>
      </div>

      <div className="stat">
        <div className="stat-title">
          Orders
        </div>

        <div className="stat-value">
          {data.orders}
        </div>
      </div>

      <div className="stat">
        <div className="stat-title">
          Users
        </div>

        <div className="stat-value">
          {data.users}
        </div>
      </div>
    </div>
  )
}
```

---

# 17. 최근 주문 Table

```jsx
function OrderTable({ orders }) {
  return (
    <div className="overflow-x-auto">
      <table className="table">
        <thead>
          <tr>
            <th>주문번호</th>
            <th>사용자</th>
            <th>금액</th>
            <th>상태</th>
            <th></th>
          </tr>
        </thead>

        <tbody>
          {orders.map((order) => (
            <OrderRow
              key={order.id}
              order={order}
            />
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

---

# 18. 주문 상태 Badge

```jsx
const statusClass = {
  pending: 'badge-warning',
  paid: 'badge-info',
  shipping: 'badge-primary',
  completed: 'badge-success',
  cancelled: 'badge-error',
}

function OrderStatus({ status }) {
  return (
    <span
      className={`badge ${statusClass[status]}`}
    >
      {status}
    </span>
  )
}
```

이것은 PART 2, PART 3, PART 6이 모두 결합된 예입니다.

```text
Modifier
+
Semantic Color
+
Server Data
+
React Conditional UI
```

---

# 19. 삭제 Action

관리자 Table에서 삭제 Button을 누른다고 하겠습니다.

```text
Table Row
   ↓
[삭제]
   ↓
Modal
   ↓
Confirm
   ↓
Mutation
   ↓
Loading
   ↓
Success / Error
   ↓
Toast
```

이것은 PART 7의 대표적인 실전 흐름입니다.

---

# 20. Delete Modal

```jsx
function DeleteProductModal({
  product,
  onConfirm,
  loading,
}) {
  return (
    <dialog className="modal" open>
      <div className="modal-box">
        <h3 className="font-bold text-lg">
          상품 삭제
        </h3>

        <p className="py-4">
          {product.name}을 삭제하시겠습니까?
        </p>

        <div className="modal-action">
          <button className="btn">
            취소
          </button>

          <button
            className="btn btn-error"
            disabled={loading}
            onClick={onConfirm}
          >
            {loading && (
              <span className="loading loading-spinner loading-sm" />
            )}

            삭제
          </button>
        </div>
      </div>
    </dialog>
  )
}
```

---

# 21. 프로젝트 3 — 게시판 UI

게시판은 지금까지 배운 Component를 또 다른 방식으로 조합합니다.

```text
Navbar
   ↓
Breadcrumbs
   ↓
Post List
   │
   ├─ Search Form
   ├─ Table / List
   └─ Pagination
   ↓
Footer
```

게시글 상세:

```text
Breadcrumbs
   ↓
Card
├─ Title
├─ Author + Avatar
├─ Badge
├─ Content
└─ Action Buttons

Comments
└─ Card / Avatar / Textarea
```

---

# 22. 게시글 목록

Desktop에서는 Table이 자연스럽습니다.

```jsx
<table className="table">
  <thead>
    <tr>
      <th>번호</th>
      <th>제목</th>
      <th>작성자</th>
      <th>작성일</th>
      <th>조회수</th>
    </tr>
  </thead>

  <tbody>
    {posts.map((post) => (
      <tr key={post.id}>
        <td>{post.id}</td>

        <td>
          <a className="link link-hover">
            {post.title}
          </a>
        </td>

        <td>{post.author}</td>
        <td>{post.createdAt}</td>
        <td>{post.views}</td>
      </tr>
    ))}
  </tbody>
</table>
```

---

# 23. Mobile 게시글 목록

Mobile에서는 Card/List 형태가 더 자연스러울 수 있습니다.

```jsx
<div className="space-y-3 md:hidden">
  {posts.map((post) => (
    <div
      key={post.id}
      className="card bg-base-100 shadow-sm"
    >
      <div className="card-body">
        <h2 className="card-title text-base">
          {post.title}
        </h2>

        <div className="text-sm opacity-70">
          {post.author} · {post.createdAt}
        </div>
      </div>
    </div>
  ))}
</div>
```

즉 같은 Data도 화면 크기에 따라 표현 Component가 달라질 수 있습니다.

```text
Desktop
→ Table

Mobile
→ Card / List
```

---

# 24. 검색 Form

```jsx
<form className="flex gap-2">
  <select className="select">
    <option>제목</option>
    <option>내용</option>
    <option>작성자</option>
  </select>

  <input
    className="input flex-1"
    placeholder="검색어"
  />

  <button className="btn btn-primary">
    검색
  </button>
</form>
```

PART 4에서 배운 Form Component를 Navigation/Data Display 화면에 결합합니다.

---

# 25. 게시글 상세

```jsx
<article className="card bg-base-100 shadow-sm">
  <div className="card-body">
    <div className="flex items-center justify-between">
      <h1 className="card-title text-2xl">
        게시글 제목
      </h1>

      <span className="badge badge-primary">
        Notice
      </span>
    </div>

    <div className="flex items-center gap-3">
      <div className="avatar">
        <div className="w-8 rounded-full">
          <img src="/user.jpg" alt="작성자" />
        </div>
      </div>

      <span>
        홍길동
      </span>
    </div>

    <div className="divider" />

    <div>
      게시글 내용...
    </div>
  </div>
</article>
```

---

# 26. 댓글 UI

```text
Comments
│
├─ Comment
│   ├─ Avatar
│   ├─ User Name
│   ├─ Content
│   └─ Actions
│
├─ Comment
│
└─ Comment Form
    ├─ Textarea
    └─ Button
```

여기서 사용되는 Component:

```text
Avatar
Card
Textarea
Button
Dropdown
```

입니다.

---

# 27. 모든 프로젝트에 공통으로 등장하는 패턴

쇼핑몰, Dashboard, 게시판은 서로 다른 앱처럼 보입니다.

하지만 내부 구조는 매우 비슷합니다.

```text
Application
│
├─ Layout
│   ├─ Navbar
│   ├─ Sidebar / Drawer
│   └─ Footer
│
├─ Navigation
│   ├─ Menu
│   ├─ Breadcrumbs
│   └─ Tabs
│
├─ Form
│   ├─ Input
│   ├─ Select
│   └─ Button
│
├─ Data Display
│   ├─ Card
│   ├─ Table
│   ├─ Badge
│   └─ Stat
│
└─ Feedback
    ├─ Modal
    ├─ Loading
    ├─ Alert
    └─ Toast
```

이것이 daisyUI를 Component 단위로 배우는 이유입니다.

---

# 28. Loading / Error / Empty는 모든 화면에서 필요하다

서버 데이터를 사용하는 화면에는 공통적으로 다음 상태가 존재합니다.

```text
Loading
Error
Empty
Success
```

예:

```jsx
if (isLoading) {
  return (
    <div className="flex justify-center p-10">
      <span className="loading loading-spinner loading-lg" />
    </div>
  )
}

if (isError) {
  return (
    <div className="alert alert-error">
      데이터를 불러오지 못했습니다.
    </div>
  )
}

if (products.length === 0) {
  return (
    <div className="card bg-base-100">
      <div className="card-body items-center">
        상품이 없습니다.
      </div>
    </div>
  )
}
```

---

# 29. 화면 전체 Loading보다 부분 Loading을 고려한다

예를 들어 장바구니 상품 하나를 삭제한다고 하겠습니다.

좋지 않은 UX:

```text
삭제 클릭
   ↓
전체 페이지 Spinner
```

더 자연스러운 UX:

```text
삭제 클릭
   ↓
해당 Delete Button만 Loading
   ↓
나머지 UI는 계속 사용 가능
```

상황에 맞게 Loading 범위를 결정해야 합니다.

---

# 30. Theme은 실전 프로젝트 전체에 적용된다

PART 3에서 배운 Theme은 실제 프로젝트의 모든 화면에 연결됩니다.

```text
Theme
  │
  ├─ Navbar
  ├─ Sidebar
  ├─ Card
  ├─ Table
  ├─ Form
  ├─ Modal
  └─ Toast
```

Theme 기반 Class:

```text
bg-base-100
bg-base-200
text-base-content
btn-primary
badge-success
alert-error
```

를 사용하면 전체 UI의 디자인 일관성을 유지하기 쉽습니다.

---

# 31. Layout에서는 Semantic Color를 우선 고려

예:

```jsx
<div className="min-h-screen bg-base-200 text-base-content">
```

Navbar:

```jsx
<nav className="navbar bg-base-100">
```

Card:

```jsx
<div className="card bg-base-100">
```

이를 모두:

```text
bg-white
bg-gray-100
text-gray-900
```

같은 고정 Palette로 작성하는 것보다 Theme 전환에 유리합니다.

단, 고정 Palette 자체가 잘못된 것은 아닙니다.

Theme에 따라 변경되어야 하는 영역인지 먼저 판단해야 합니다.

---

# 32. Component를 너무 크게 만들지 않는다

좋지 않은 구조:

```text
ShopPage.jsx

Navbar
Filter
Search
Product Grid
Product Card
Pagination
Modal
Toast
API Logic

→ 전부 하나의 Component
```

좋은 구조:

```text
ShopPage
│
├─ ShopNavbar
├─ ProductFilter
├─ ProductGrid
│   └─ ProductCard
├─ Pagination
├─ DeleteModal
└─ Toast
```

Component의 책임을 나누는 것이 좋습니다.

---

# 33. 그렇다고 너무 잘게 나누지도 않는다

반대로 모든 `<div>`를 Component로 만드는 것도 좋지 않습니다.

예:

```text
ProductName.jsx
ProductPrice.jsx
ProductButtonWrapper.jsx
ProductButtonText.jsx
```

처럼 의미 없이 분리하면 관리하기 더 어려워집니다.

추천 기준:

> **독립적인 역할, 재사용성, 상태/로직의 경계가 있을 때 Component 분리를 고려합니다.**

---

# 34. 재사용 가능한 UI Component

프로젝트가 커지면 반복되는 UI 규칙을 Component로 만들 수 있습니다.

예:

```jsx
function StatusBadge({ status }) {
  const variants = {
    pending: 'badge-warning',
    success: 'badge-success',
    error: 'badge-error',
  }

  return (
    <span className={`badge ${variants[status]}`}>
      {status}
    </span>
  )
}
```

그러면 여러 화면에서 같은 규칙을 사용합니다.

```text
Order Table
Product Table
Payment History
User Management
```

---

# 35. Layout Component와 Feature Component 구분

실전 React 프로젝트에서는 다음처럼 구분할 수 있습니다.

```text
components/
├─ layout/
│   ├─ Navbar.jsx
│   ├─ Sidebar.jsx
│   └─ Footer.jsx
│
├─ ui/
│   ├─ StatusBadge.jsx
│   ├─ LoadingState.jsx
│   └─ ConfirmModal.jsx
│
features/
├─ product/
│   ├─ ProductCard.jsx
│   └─ ProductList.jsx
│
├─ order/
│   ├─ OrderTable.jsx
│   └─ OrderStatus.jsx
```

즉 UI의 책임을 구조적으로 분리합니다.

---

# 36. daisyUI Class를 추상화해야 하는가?

다음 코드가 여러 곳에 반복된다고 하겠습니다.

```jsx
<button className="btn btn-primary">
```

무조건:

```jsx
<AppButton>
```

으로 감싸야 하는 것은 아닙니다.

다음 기준을 생각합니다.

```text
단순 Class 반복
→ 반드시 Component화할 필요 없음

공통 동작 / Variant 규칙 / 접근성 규칙
→ Component화 가치가 큼
```

예:

```jsx
function AppButton({
  variant = 'primary',
  loading = false,
  children,
  ...props
}) {
  return (
    <button
      className={`btn btn-${variant}`}
      disabled={loading || props.disabled}
      {...props}
    >
      {loading && (
        <span className="loading loading-spinner loading-sm" />
      )}

      {children}
    </button>
  )
}
```

---

# 37. 실제 Server State와 연결

쇼핑몰 상품 목록이라면:

```text
TanStack Query / RTK Query
        ↓
    products
        ↓
     React
        ↓
  ProductGrid
        ↓
 ProductCard
```

Mutation:

```text
Add To Cart
    ↓
Mutation
    ↓
Loading Button
    ↓
Success / Error
    ↓
Toast
```

즉 지금까지 배운 모든 PART가 하나의 흐름에 들어옵니다.

---

# 38. 실전 UI 전체 아키텍처

```text
                        Application
                             │
              ┌──────────────┼───────────────┐
              │              │               │
              ▼              ▼               ▼
            Layout         Routing        Server State
              │              │               │
         Navbar/Drawer   React Router   Query Library
              │              │               │
              └──────────────┼───────────────┘
                             ▼
                           React
                             │
            ┌────────────────┼────────────────┐
            ▼                ▼                ▼
           Form          Data Display      Feedback
            │                │                │
     Input / Select     Card / Table    Modal / Toast
            │                │                │
            └────────────────┼────────────────┘
                             ▼
                           daisyUI
                             │
                             ▼
                         Tailwind CSS
                             │
                             ▼
                       Final Application
```

---

# 39. 실전 개발 체크리스트

화면을 만들기 전에 다음 질문을 확인합니다.

```text
□ 화면을 큰 영역으로 분리했는가?

□ 적절한 daisyUI Component를 선택했는가?

□ Layout은 Tailwind CSS로 구성했는가?

□ Theme Semantic Color를 사용해야 할 영역을 구분했는가?

□ Loading / Error / Empty 상태를 처리했는가?

□ Action 결과를 Modal / Alert / Toast 중 적절하게 전달하는가?

□ Mobile / Desktop Responsive를 고려했는가?

□ Component의 책임이 너무 크지 않은가?

□ Server State와 UI State를 구분했는가?

□ 접근성을 고려했는가?
```

---

# 40. PART 8 핵심 정리

지금까지 학습한 내용을 실전 프로젝트에 연결하면 다음과 같습니다.

```text
PART 1
daisyUI란?
       ↓
PART 2
Component + Modifier
       ↓
PART 3
Theme
       ↓
PART 4
Form
       ↓
PART 5
Navigation / Layout
       ↓
PART 6
Data Display
       ↓
PART 7
Overlay / Feedback
       ↓
PART 8
실전 Application UI
```

쇼핑몰:

```text
Navbar
Drawer
Filter
Card
Badge
Modal
Toast
```

Dashboard:

```text
Drawer
Menu
Stat
Table
Badge
Modal
```

게시판:

```text
Navbar
Breadcrumbs
Table
Card
Avatar
Textarea
Dropdown
```

하지만 Component 이름을 많이 아는 것 자체가 목표는 아닙니다.

최종적으로 기억해야 할 것은 다음입니다.

> **화면의 역할을 먼저 분석하고, 그 역할에 맞는 daisyUI Component를 선택한 뒤, Tailwind CSS로 Layout을 구성하고, React 및 Query Library로 실제 State와 데이터를 연결한다.**

최종 흐름:

```text
Requirement
     ↓
UI 분해
     ↓
Component 선택
     ↓
Layout
     ↓
Data / State 연결
     ↓
Interaction
     ↓
Feedback
     ↓
Responsive
     ↓
Final Application
```

이 단계까지 이해하면 daisyUI를 단순한 CSS Component 모음이 아니라 **실제 React Application UI를 빠르게 구성하기 위한 도구**로 사용할 수 있습니다.
