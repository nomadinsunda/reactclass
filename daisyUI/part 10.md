# PART 10. daisyUI 실전 아키텍처 & Best Practice

## 1. 이번 PART의 목표

지금까지 다음 내용을 배웠습니다.

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

PART 8
실전 Application UI

PART 9
Design System / 재사용 Component
```

PART 10에서는 이 모든 내용을 하나의 질문으로 정리합니다.

> **실제 React 프로젝트에서 daisyUI를 어떻게 사용하는 것이 좋은가?**

핵심은 특정 Class를 많이 외우는 것이 아닙니다.

```text
Requirement
    ↓
UI 역할 분석
    ↓
적절한 Component 선택
    ↓
Tailwind Layout
    ↓
React State / Event
    ↓
Server State 연결
    ↓
Feedback
    ↓
Reusable Rule 추출
```

이 흐름을 이해하는 것이 최종 목표입니다.

---

# 2. daisyUI는 어디에 위치하는가?

실제 React Application을 계층으로 보면 다음과 같습니다.

```text
                    User
                     │
                     ▼
               React Application
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
     Routing       State         Server Data
       │             │             │
 React Router      React       Query Library
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                  UI Layer
                     │
       ┌─────────────┴─────────────┐
       ▼                           ▼
    daisyUI                    Tailwind CSS
       │                           │
 Component / Theme          Layout / Utility
       │                           │
       └─────────────┬─────────────┘
                     ▼
                  Final UI
```

즉 daisyUI는 **Application Logic Layer가 아니라 UI Layer**에 위치합니다.

---

# 3. 역할 분담을 다시 정확하게 정리

## React

```text
Component
State
Props
Event
Conditional Rendering
Composition
```

## React Router

```text
URL
Route
Navigation
Layout Route
Active Route
```

## RTK Query / TanStack Query

```text
Server State
Caching
Loading
Error
Mutation
Refetch
```

## daisyUI

```text
Component Class
Variant / Modifier
Theme
Semantic Color
UI Pattern
```

## Tailwind CSS

```text
Layout
Spacing
Sizing
Responsive
Typography
Position
세부 Style
```

핵심:

> **daisyUI가 React, Router, Query Library의 역할을 대신하지 않습니다.**

---

# 4. UI를 만들 때 가장 먼저 할 일

완성된 디자인을 보고 바로 Class를 작성하면 안 됩니다.

먼저 화면을 역할로 나눕니다.

예를 들어 관리자 Dashboard:

```text
┌──────────────────────────────────────┐
│ Navbar                               │
├──────────┬───────────────────────────┤
│ Sidebar  │ Stats                     │
│          │                           │
│ Menu     │ Table                     │
│          │                           │
└──────────┴───────────────────────────┘
```

먼저:

```text
Layout
├─ Navbar
├─ Sidebar
└─ Main

Main
├─ Stats
└─ Table
```

로 분해합니다.

그 다음에야 daisyUI Component를 선택합니다.

---

# 5. Component를 모양이 아니라 역할로 선택한다

예:

```text
"둥근 박스가 필요하다"
```

보다:

```text
"하나의 상품 정보를 묶어 보여준다"
        ↓
Card
```

가 더 좋은 사고방식입니다.

또:

```text
"빨간 박스가 필요하다"
```

보다:

```text
"사용자에게 Error Feedback을 전달한다"
        ↓
Alert + error
```

가 좋습니다.

즉:

```text
Visual 먼저
     X

Semantic Role 먼저
     O
```

입니다.

---

# 6. Class를 읽는 최종 규칙

PART 2에서 배운 내용을 다시 정리합니다.

```html
<button
  class="
    btn
    btn-primary
    btn-outline
    btn-lg
    w-full
    mt-4
  "
>
```

분해:

```text
btn
→ Component

btn-primary
→ Color Variant

btn-outline
→ Style Variant

btn-lg
→ Size Variant

w-full
→ Tailwind Utility

mt-4
→ Tailwind Utility
```

추천 사고 순서:

```text
Component
   ↓
Variant
   ↓
State
   ↓
Utility
```

---

# 7. daisyUI와 Tailwind CSS의 경계

둘을 경쟁 관계로 보면 안 됩니다.

```text
daisyUI
→ "무엇인가?"

Tailwind CSS
→ "어떻게 배치하고 조정할까?"
```

예:

```jsx
<div className="card bg-base-100 shadow-sm w-full max-w-md">
```

여기서:

```text
card
→ daisyUI

bg-base-100
→ Theme 기반 Utility

shadow-sm
w-full
max-w-md
→ Tailwind CSS
```

입니다.

---

# 8. Layout은 Tailwind CSS가 중심

실제 화면 Layout:

```jsx
<div
  className="
    grid
    grid-cols-1
    md:grid-cols-2
    lg:grid-cols-3
    gap-6
  "
>
```

이 부분은 daisyUI보다 Tailwind CSS가 중심입니다.

```text
daisyUI
→ 개별 UI Component

Tailwind
→ Component 간 배치
```

이 원칙을 기억하면 Class 역할을 구분하기 쉽습니다.

---

# 9. Theme은 Design System의 중심

PART 3의 핵심을 다시 정리합니다.

```text
Theme
  ↓
Semantic Color
  ↓
Component / Utility
  ↓
Final UI
```

예:

```html
<button class="btn btn-primary">
```

여기서:

```text
primary ≠ blue

primary
→ Semantic Role
```

입니다.

실제 색상은 Theme이 결정합니다.

---

# 10. 고정 Palette와 Semantic Color를 구분한다

예:

```html
<div class="bg-blue-600">
```

vs

```html
<div class="bg-primary">
```

차이:

```text
bg-blue-600
→ 특정 Palette Token 직접 선택

bg-primary
→ Theme의 Semantic Role 선택
```

추천 기준:

```text
Theme에 따라 바뀌어야 하는 UI
→ Semantic Color

Theme과 무관한 장식 / 특수 표현
→ 일반 Tailwind Palette 사용 가능
```

---

# 11. Form에서의 역할 분담

로그인 Form:

```text
HTML
→ input / label / form

daisyUI
→ input / checkbox / button UI

Tailwind
→ width / gap / spacing

React
→ value / onChange / State

Validation
→ 입력 검증

API
→ Login Request
```

중요한 점:

> **daisyUI는 Validation Library가 아닙니다.**

---

# 12. Navigation에서의 역할 분담

예:

```jsx
<NavLink
  to="/products"
  className={({ isActive }) =>
    isActive ? 'menu-active' : ''
  }
>
  Products
</NavLink>
```

역할:

```text
NavLink / isActive
→ React Router

menu-active
→ daisyUI
```

즉 daisyUI가 URL을 판단하지 않습니다.

---

# 13. Server Data 화면의 역할 분담

```text
Server
  ↓
Query Library
  ↓
React
  ↓
daisyUI
```

예:

```text
RTK Query / TanStack Query
→ Data / Loading / Error

React
→ 조건부 Rendering

daisyUI
→ Card / Table / Badge / Loading UI
```

---

# 14. 반드시 4가지 상태를 생각한다

Server Data 화면은 최소 다음 상태를 고려합니다.

```text
Loading
Error
Empty
Success
```

예:

```jsx
if (isLoading) {
  return <LoadingState />
}

if (isError) {
  return <ErrorState />
}

if (products.length === 0) {
  return <EmptyState />
}

return <ProductGrid products={products} />
```

이 패턴은 거의 모든 데이터 기반 화면에서 반복됩니다.

---

# 15. Action 흐름을 하나의 Pipeline으로 본다

예: 상품 삭제

```text
사용자
  ↓
Delete Button
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

이 흐름에서:

```text
React
→ Event / State

Query Library
→ Mutation

daisyUI
→ Modal / Button / Loading / Toast
```

입니다.

---

# 16. Modal을 Styling Component로만 생각하지 않는다

Modal에는 다음도 중요합니다.

```text
Open / Close
Focus
Keyboard
Confirm / Cancel
Accessibility
```

daisyUI가 모든 Interaction Logic을 자동으로 해결하는 것은 아닙니다.

HTML Dialog API 또는 React Logic과 함께 사용해야 합니다.

---

# 17. Toast도 Lifecycle을 자동 관리하지 않는다

```text
daisyUI toast
→ UI + Position

Application
→ 언제 생성?
→ 언제 제거?
→ Queue?
→ Timer?
```

프로젝트가 커지면 Toast System을 별도로 관리할 수 있습니다.

---

# 18. React Component 분리 기준

Component를 나누는 기준:

```text
독립적인 역할이 있는가?
재사용되는가?
State / Logic 경계가 있는가?
변경 이유가 다른가?
```

예:

```text
ShopPage
├─ ShopNavbar
├─ ProductFilter
├─ ProductGrid
│   └─ ProductCard
└─ Pagination
```

이런 구조가 자연스럽습니다.

---

# 19. 너무 큰 Component의 문제

좋지 않은 구조:

```text
ShopPage.jsx
│
├─ Navbar
├─ Search
├─ Filter
├─ Card
├─ Pagination
├─ Modal
├─ Toast
└─ API Logic
```

한 파일이 모든 것을 담당하면:

```text
수정 어려움
재사용 어려움
테스트 어려움
책임 불명확
```

해질 수 있습니다.

---

# 20. 너무 잘게 나누는 것도 문제

반대로:

```text
ProductTitle.jsx
ProductPrice.jsx
ProductPriceText.jsx
ProductButtonText.jsx
```

처럼 의미 없이 작은 Component를 계속 만들 필요도 없습니다.

추천:

> **역할과 변경 이유가 분명할 때 분리한다.**

---

# 21. daisyUI Class를 모두 감싸지 않는다

다음처럼:

```jsx
<span className="badge badge-info">
  INFO
</span>
```

를 굳이:

```jsx
<InfoBadge />
```

로 만들 필요가 없을 수 있습니다.

daisyUI 자체가 이미 재사용 가능한 UI Class를 제공합니다.

---

# 22. 공통 Component로 만들 가치가 있는 경우

예:

```text
Button
+
Loading
+
Variant 규칙
+
Size 규칙
+
Accessibility
```

이 반복된다면:

```text
AppButton
```

으로 승격할 가치가 있습니다.

또:

```text
Server Status
+
Label Mapping
+
Badge Variant Mapping
```

이 반복된다면:

```text
StatusBadge
```

가 좋습니다.

---

# 23. 추상화의 기준

```text
Class 반복
     ↓
즉시 추상화
     X

Project Rule 반복
     ↓
추상화
     O
```

PART 9의 가장 중요한 원칙입니다.

---

# 24. Reusable UI 계층

```text
daisyUI
   ↓
Project UI Components
   │
   ├─ AppButton
   ├─ AppInput
   ├─ StatusBadge
   ├─ ConfirmModal
   ├─ LoadingState
   └─ ErrorState
   ↓
Feature Components
   │
   ├─ ProductCard
   ├─ OrderTable
   └─ LoginForm
   ↓
Pages
```

---

# 25. `ui`와 `feature`를 분리한다

예:

```text
components/ui/
→ 범용 UI

features/product/
→ Product Domain UI
```

구조:

```text
AppButton
→ 상품을 몰라도 사용 가능

ProductCard
→ Product 구조를 알아야 함
```

즉:

```text
Generic UI
vs
Domain UI
```

를 분리합니다.

---

# 26. 추천 프로젝트 구조

```text
src/
│
├─ components/
│   │
│   ├─ ui/
│   │   ├─ AppButton.jsx
│   │   ├─ AppInput.jsx
│   │   ├─ StatusBadge.jsx
│   │   ├─ ConfirmModal.jsx
│   │   ├─ LoadingState.jsx
│   │   ├─ EmptyState.jsx
│   │   └─ ErrorState.jsx
│   │
│   └─ layout/
│       ├─ Navbar.jsx
│       ├─ Sidebar.jsx
│       └─ Footer.jsx
│
├─ features/
│   ├─ product/
│   ├─ order/
│   └─ auth/
│
├─ layouts/
│   └─ AppLayout.jsx
│
└─ pages/
```

---

# 27. 공통 Component의 Props는 Semantic하게

좋지 않은 예:

```jsx
<Button blue />
<Button red />
<Button green />
```

더 좋은 예:

```jsx
<Button variant="primary" />
<Button variant="danger" />
<Button variant="success" />
```

왜냐하면 Theme이 바뀌어도 의미는 유지되기 때문입니다.

---

# 28. Boolean Props 남발을 피한다

좋지 않은 예:

```jsx
<Button
  primary
  secondary
  outline
  ghost
  small
  large
/>
```

문제:

```text
primary + secondary?
small + large?
outline + ghost?
```

추천:

```jsx
<Button
  variant="primary"
  size="lg"
  appearance="outline"
/>
```

즉:

> **하나의 선택 축에는 하나의 값**

을 사용하는 것이 좋습니다.

---

# 29. Component 내부와 외부 Layout을 구분

예:

```jsx
<AppButton className="w-full mt-4">
  로그인
</AppButton>
```

역할:

```text
AppButton
→ Button 자체의 규칙

w-full
mt-4
→ 사용하는 화면의 Layout
```

공통 Component가 주변 Layout까지 지나치게 통제하지 않는 것이 좋습니다.

---

# 30. Responsive는 Layout 단계에서 생각한다

예:

```jsx
<div
  className="
    grid
    grid-cols-1
    sm:grid-cols-2
    lg:grid-cols-3
    xl:grid-cols-4
    gap-6
  "
>
```

Mobile First:

```text
Mobile
→ 1 Column

sm
→ 2 Columns

lg
→ 3 Columns

xl
→ 4 Columns
```

daisyUI Component 자체와 Responsive Layout을 분리해서 생각합니다.

---

# 31. Mobile에서는 Component 자체가 달라질 수도 있다

같은 게시글 데이터라도:

```text
Desktop
→ Table

Mobile
→ Card / List
```

가 더 자연스러울 수 있습니다.

즉 Responsive는:

```text
크기만 줄인다
```

가 아니라:

```text
필요하면 정보 구조와 표현 Component도 변경
```

할 수 있습니다.

---

# 32. Theme과 Responsive는 서로 다른 축

```text
Theme
→ Color / Design 표현

Responsive
→ Screen Size / Layout
```

예:

```text
Dark + Mobile
Dark + Desktop
Light + Mobile
Light + Desktop
```

처럼 두 축은 서로 독립적으로 조합됩니다.

---

# 33. State 역시 또 다른 축

UI를 생각할 때 최소 세 축을 구분하면 좋습니다.

```text
Theme
→ light / dark / custom

Viewport
→ mobile / tablet / desktop

State
→ normal / loading / error / disabled
```

최종 UI는 이 조합의 결과입니다.

---

# 34. UI를 상태 공간으로 생각하기

예: Save Button

```text
Theme
├─ light
└─ dark

Size
├─ mobile
└─ desktop

State
├─ normal
├─ loading
└─ disabled
```

즉 하나의 Component라도 여러 상황을 고려해야 합니다.

---

# 35. 접근성은 마지막에 추가하는 기능이 아니다

처음부터 다음을 고려합니다.

```text
Form
→ Label

Image
→ alt

Button
→ 명확한 Text / aria-label

Modal
→ Focus / Close

Navigation
→ Semantic HTML

Error
→ 색상 외 Text 정보

Loading
→ 상태 전달
```

daisyUI를 사용한다고 자동으로 모든 접근성이 해결되는 것은 아닙니다.

---

# 36. Semantic HTML을 유지한다

좋은 예:

```html
<nav>
<button>
<form>
<label>
<table>
dialog
```

단순히:

```html
<div>
<div>
<div>
```

에 Class만 붙이는 방식보다 HTML 의미를 유지하는 것이 좋습니다.

---

# 37. 접근성과 daisyUI의 관계

```text
HTML
→ Semantic / Accessibility 기반

daisyUI
→ Visual UI

React
→ Interaction

Developer
→ 올바른 연결
```

입니다.

---

# 38. Theme 변경 시 Contrast를 확인한다

Custom Theme에서는:

```text
primary
+
primary-content
```

를 함께 설계합니다.

그리고 실제 Contrast를 확인해야 합니다.

```text
Theme Token이 존재
      ≠
접근성 자동 보장
```

입니다.

---

# 39. Color 이름보다 의미를 사용한다

프로젝트 코드에서:

```text
blue
red
green
```

보다:

```text
primary
danger
success
warning
```

같은 Semantic Naming을 우선 고려합니다.

Design System의 핵심입니다.

---

# 40. Loading UX

모든 API 요청마다 페이지 전체 Spinner를 띄우는 것은 좋지 않을 수 있습니다.

예:

```text
상품 하나 삭제
      ↓
Delete Button만 Loading
```

반면 첫 페이지 전체 데이터를 처음 로딩할 때:

```text
Page Skeleton
```

이 더 자연스러울 수 있습니다.

Loading의 범위를 Action 범위에 맞게 선택합니다.

---

# 41. Error UX

Error도 수준이 다릅니다.

```text
Field Error
→ Input 주변

Form Error
→ Form Alert

Action Error
→ Toast / Inline Alert

Page Error
→ ErrorState
```

하나의 Error Component만 모든 상황에 사용하는 것보다 Context에 맞게 선택합니다.

---

# 42. Feedback Component 선택

```text
사용자가 반드시 확인
→ Modal / Alert

잠깐 알려주면 충분
→ Toast

작업 진행
→ Loading

데이터 구조 대기
→ Skeleton
```

즉 Component는 **사용자 경험의 목적**으로 선택합니다.

---

# 43. Server State와 UI State를 구분

예:

```text
Server State
────────────
products
orders
user
isFetching

UI State
────────────
modalOpen
selectedTab
drawerOpen
theme
```

Query Library와 React Local State의 역할을 구분합니다.

---

# 44. 예: Product Page의 State

```text
Server State
│
├─ products
├─ categories
└─ cart


UI State
│
├─ filterOpen
├─ selectedSort
└─ deleteModalOpen
```

이 두 종류의 State를 하나의 방식으로 관리할 필요는 없습니다.

---

# 45. 실제 전체 Request 흐름

```text
User
 ↓
UI Event
 ↓
React
 ↓
Query / Mutation
 ↓
API
 ↓
Server Response
 ↓
Server State Update
 ↓
React Rendering
 ↓
daisyUI Component
 ↓
User Feedback
```

이 구조에서 daisyUI가 담당하는 위치를 이해해야 합니다.

---

# 46. 쇼핑몰 전체 아키텍처 예

```text
App
│
├─ AppLayout
│   ├─ Navbar
│   ├─ Drawer
│   └─ Footer
│
├─ ProductListPage
│   ├─ ProductFilter
│   ├─ ProductGrid
│   │   └─ ProductCard
│   └─ Pagination
│
├─ ProductDetailPage
│   ├─ Breadcrumbs
│   ├─ ProductDetail
│   ├─ Tabs
│   └─ AddToCartButton
│
└─ CartPage
    ├─ CartTable
    ├─ Summary
    └─ CheckoutButton
```

---

# 47. 관리자 Dashboard 아키텍처 예

```text
AdminLayout
│
├─ Sidebar
│   └─ Menu
│
├─ Navbar
│
└─ Outlet
    │
    ├─ DashboardPage
    │   ├─ Stats
    │   └─ RecentOrders
    │
    ├─ ProductManagement
    │   ├─ Filter
    │   ├─ Table
    │   └─ ConfirmModal
    │
    └─ OrderManagement
        ├─ Table
        └─ StatusBadge
```

---

# 48. 게시판 아키텍처 예

```text
BoardLayout
│
├─ Navbar
├─ Breadcrumbs
└─ Outlet
    │
    ├─ PostListPage
    │   ├─ SearchForm
    │   ├─ PostList
    │   └─ Pagination
    │
    ├─ PostViewPage
    │   ├─ PostCard
    │   ├─ AuthorAvatar
    │   └─ Comments
    │
    └─ PostEditorPage
        ├─ Form
        └─ SubmitButton
```

---

# 49. Design System이 애플리케이션 위에 존재한다

```text
                 Design System
                      │
       ┌──────────────┼──────────────┐
       ▼              ▼              ▼
     Theme          UI Rules     Accessibility
       │              │              │
       └──────────────┼──────────────┘
                      ▼
               Shared UI Component
                      │
              Feature Components
                      │
                    Pages
```

Design System은 특정 Page 하나를 위한 것이 아닙니다.

Application 전체에서 공유되는 규칙입니다.

---

# 50. 좋은 daisyUI 프로젝트의 특징

```text
Component 역할이 명확하다

Theme 기반 Semantic Color를 사용한다

Tailwind Layout과 daisyUI Component를 구분한다

React Logic과 UI Styling을 분리한다

Loading / Error / Empty를 고려한다

Responsive를 고려한다

공통 Project Rule만 추상화한다

접근성을 고려한다

Domain Component와 Generic UI를 구분한다
```

---

# 51. 좋지 않은 패턴 1 — 모든 것을 daisyUI가 해준다고 생각

잘못된 생각:

```text
Modal
→ Open State까지 자동 관리?

Toast
→ Timer까지 자동 관리?

Menu
→ Router까지 자동 연결?

Input
→ Validation까지 자동 처리?
```

아닙니다.

```text
daisyUI
→ UI Layer
```

라는 경계를 항상 기억해야 합니다.

---

# 52. 좋지 않은 패턴 2 — Tailwind를 거의 사용하지 않는다

daisyUI를 사용한다고:

```text
Layout까지 전부 daisyUI Component로 해결
```

하려고 하면 오히려 부자연스러울 수 있습니다.

실전에서는:

```text
daisyUI
+
Tailwind CSS
```

가 기본입니다.

---

# 53. 좋지 않은 패턴 3 — 고정 색상을 남발

예:

```jsx
bg-blue-600
bg-gray-100
text-gray-900
```

를 프로젝트의 핵심 Theme 영역에 계속 사용하면 Theme 전환이 어려워질 수 있습니다.

Theme 기반 영역에서는:

```text
bg-primary
bg-base-100
text-base-content
```

같은 Semantic Color를 우선 고려합니다.

---

# 54. 좋지 않은 패턴 4 — 지나친 추상화

```text
daisyUI Component
      ↓
Wrapper
      ↓
Wrapper
      ↓
Wrapper
      ↓
실제 사용
```

이렇게 되면 오히려 daisyUI의 단순함을 잃을 수 있습니다.

추상화에는 항상 이유가 있어야 합니다.

---

# 55. 좋지 않은 패턴 5 — Class만 보고 UI를 설계

```text
btn이 있으니까 Button 사용

card가 있으니까 Card 사용
```

이 아니라:

```text
이 UI의 역할은 무엇인가?
       ↓
적절한 Component 선택
```

순서가 되어야 합니다.

---

# 56. 개발 순서 추천

실제 화면 하나를 만들 때:

```text
1. Requirement 분석

2. 화면 영역 분해

3. Semantic HTML 결정

4. daisyUI Component 선택

5. Tailwind Layout 구성

6. Theme 적용

7. React State / Event 연결

8. Server State 연결

9. Loading / Error / Empty 처리

10. Responsive

11. Accessibility

12. 반복 규칙 추상화
```

이 순서를 추천합니다.

---

# 57. 처음부터 완벽한 Design System을 만들지 않는다

자연스러운 성장 과정:

```text
Stage 1
daisyUI 직접 사용
        ↓
Stage 2
실전 화면 개발
        ↓
Stage 3
반복 패턴 발견
        ↓
Stage 4
Reusable UI 추출
        ↓
Stage 5
Project Design System
```

이 방식이 현실적입니다.

---

# 58. daisyUI를 잘 쓰는 기준

daisyUI Class를 많이 아는 사람이 잘 쓰는 것이 아닙니다.

다음 판단을 잘하는 사람이 잘 쓰는 것입니다.

```text
어떤 Component가 적절한가?

어디까지 daisyUI를 사용할까?

어디서 Tailwind를 사용할까?

State는 누가 관리할까?

Theme을 어떻게 적용할까?

어떤 패턴을 재사용할까?
```

---

# 59. PART 1~10 전체 흐름

```text
PART 1
daisyUI란?
      ↓
PART 2
Class / Modifier
      ↓
PART 3
Color / Theme
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
실전 Application
      ↓
PART 9
Design System
      ↓
PART 10
Architecture / Best Practice
```

---

# 60. daisyUI 전체 아키텍처

```text
                         Requirement
                              │
                              ▼
                           React
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          Routing          UI State       Server State
             │                │                │
      React Router         React       Query Library
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                         UI Structure
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
               daisyUI                 Tailwind CSS
                 │                         │
        Component / Theme          Layout / Utility
                 │                         │
                 └────────────┬────────────┘
                              ▼
                       Reusable UI Rules
                              │
                              ▼
                        Design System
                              │
                              ▼
                         Final Product
```

---

# 61. 마지막 핵심 원칙

daisyUI를 사용할 때 계속 기억해야 할 원칙은 다섯 가지입니다.

```text
1.
Component를 모양이 아니라 역할로 선택한다.

2.
daisyUI와 Tailwind CSS를 함께 사용한다.

3.
Theme에서는 실제 색상보다 Semantic Role을 우선한다.

4.
Application Logic과 UI Styling을 구분한다.

5.
반복되는 Project Rule이 보일 때만 추상화한다.
```

---

# 62. 한 문장으로 정리

> **daisyUI는 React Application의 Logic을 대신하는 도구가 아니라, Tailwind CSS 위에서 Theme과 Component 패턴을 제공하여 일관된 UI를 빠르게 구성하도록 돕는 UI Layer입니다.**

그리고 실제 프로젝트의 전체 흐름은:

```text
Requirement
    ↓
React / Router / Query
    ↓
State & Data
    ↓
daisyUI + Tailwind CSS
    ↓
Reusable Project Rules
    ↓
Design System
    ↓
Final Application
```

입니다.

---

# 63. 최종 체크리스트

프로젝트에 daisyUI를 적용할 때 다음을 확인합니다.

```text
□ Component의 역할을 먼저 분석했는가?

□ daisyUI Component와 Tailwind Utility를 구분했는가?

□ Theme 기반 Semantic Color를 적절히 사용했는가?

□ React State와 UI 표현을 분리했는가?

□ React Router가 Navigation을 담당하고 있는가?

□ Query Library가 Server State를 담당하고 있는가?

□ Loading / Error / Empty / Success를 모두 고려했는가?

□ Action 결과에 적절한 Feedback UI를 사용했는가?

□ Mobile / Desktop Responsive를 고려했는가?

□ Semantic HTML과 접근성을 고려했는가?

□ 단순 Class 반복이 아니라 Project Rule을 기준으로 추상화했는가?

□ Design System이 Feature Logic과 분리되어 있는가?
```

---

# 64. 강의 전체 마무리

처음에는 다음과 같이 시작했습니다.

```html
<button class="btn btn-primary">
  저장
</button>
```

하지만 이제 이 코드 하나에서도 다음 구조를 읽을 수 있어야 합니다.

```text
button
→ Semantic HTML

btn
→ daisyUI Component

btn-primary
→ Semantic Color Variant

primary
→ Theme Token

Theme
→ Design System

Button Click
→ React Event

Loading
→ Application State

API
→ Query / Mutation

w-full / mt-4
→ Tailwind Layout
```

즉 단순히:

```text
"btn-primary를 쓰면 예쁜 Button이 나온다"
```

에서 끝나는 것이 아니라:

```text
UI Component
+
Theme
+
Layout
+
State
+
Data
+
Interaction
+
Design System
```

의 관계를 이해하는 것이 전체 강의의 목표입니다.

> **daisyUI를 잘 사용하는 핵심은 Class를 많이 외우는 것이 아니라, UI의 역할과 각 기술의 책임을 정확하게 구분하고 필요한 곳에서 조합하는 것입니다.**
