# PART 5. daisyUI Navigation & Layout

## 1. 이번 PART에서 배울 것

웹 애플리케이션은 Button이나 Input 하나만으로 만들어지지 않습니다. 여러 Component를 조합해서 **페이지 전체 구조**를 만들어야 합니다.

대표적인 쇼핑몰 화면을 보면:

```text
┌─────────────────────────────────────────────┐
│ Navbar                                      │
├──────────┬──────────────────────────────────┤
│          │                                  │
│ Sidebar  │          Main Content            │
│ / Menu   │                                  │
│          │   Card   Card   Card             │
│          │                                  │
├──────────┴──────────────────────────────────┤
│ Footer                                      │
└─────────────────────────────────────────────┘
```

PART 5에서는 다음 구조를 만드는 방법을 배웁니다.

```text
Navbar
Menu
Breadcrumbs
Tabs
Drawer
Dock
Footer
```

그리고 매우 중요한 원칙 하나를 계속 유지합니다.

```text
HTML / React
→ 구조와 동작

daisyUI
→ Component UI

Tailwind CSS
→ Layout과 세부 Style
```

---

# 2. Navbar

`Navbar`는 페이지 상단에서 Logo, Navigation, 검색, 사용자 메뉴 등을 배치하는 Component입니다.

```html
<div class="navbar bg-base-100 shadow-sm">
  <div class="flex-1">
    <a class="btn btn-ghost text-xl">
      MyShop
    </a>
  </div>

  <div class="flex-none">
    <button class="btn btn-ghost">
      로그인
    </button>
  </div>
</div>
```

구조는 다음과 같습니다.

```text
navbar
│
├─ flex-1
│    └─ Logo
│
└─ flex-none
     └─ Actions
```

여기서 역할을 구분해야 합니다.

```text
navbar
→ daisyUI Component

bg-base-100
→ Theme 기반 Utility

flex-1 / flex-none
→ Tailwind Layout Utility

btn / btn-ghost
→ daisyUI Component
```

Navbar가 애플리케이션의 모든 Navigation 기능을 자동으로 만들어 주는 것은 아닙니다.

---

# 3. Navbar에 Menu 추가

Desktop Navigation을 추가해보겠습니다.

```html
<div class="navbar bg-base-100">
  <div class="navbar-start">
    <a class="btn btn-ghost text-xl">
      MyShop
    </a>
  </div>

  <div class="navbar-center">
    <ul class="menu menu-horizontal">
      <li><a>Home</a></li>
      <li><a>Products</a></li>
      <li><a>Categories</a></li>
    </ul>
  </div>

  <div class="navbar-end">
    <button class="btn btn-primary">
      Login
    </button>
  </div>
</div>
```

구조:

```text
Navbar
│
├─ navbar-start
│    └─ Logo
│
├─ navbar-center
│    └─ Menu
│
└─ navbar-end
     └─ Actions
```

`navbar-start`, `navbar-center`, `navbar-end`를 이용하면 Navbar 내부 영역을 쉽게 구성할 수 있습니다.

---

# 4. Menu

`Menu`는 Navigation 항목이나 Action 목록을 표현할 때 사용하는 Component입니다.

```html
<ul class="menu bg-base-200 rounded-box w-56">
  <li><a>Dashboard</a></li>
  <li><a>Products</a></li>
  <li><a>Orders</a></li>
  <li><a>Users</a></li>
</ul>
```

대략 다음처럼 표현됩니다.

```text
┌──────────────────┐
│ Dashboard        │
│ Products         │
│ Orders           │
│ Users            │
└──────────────────┘
```

Menu는 Sidebar Navigation에 특히 많이 사용됩니다.

---

# 5. Menu의 Active 상태

현재 페이지를 표시하려면 Active 상태를 사용할 수 있습니다.

```html
<ul class="menu">
  <li>
    <a class="menu-active">
      Dashboard
    </a>
  </li>

  <li>
    <a>
      Products
    </a>
  </li>
</ul>
```

개념적으로:

```text
Dashboard    ← Active
Products
Orders
Users
```

하지만 중요한 점이 있습니다.

> **daisyUI가 현재 URL을 분석해서 `menu-active`를 자동으로 붙여주는 것은 아닙니다.**

React Router를 사용한다면 Router 상태를 이용해야 합니다.

---

# 6. React Router와 Menu

예를 들어:

```jsx
import { NavLink } from 'react-router'

function Sidebar() {
  return (
    <ul className="menu">
      <li>
        <NavLink
          to="/dashboard"
          className={({ isActive }) =>
            isActive ? 'menu-active' : ''
          }
        >
          Dashboard
        </NavLink>
      </li>

      <li>
        <NavLink
          to="/products"
          className={({ isActive }) =>
            isActive ? 'menu-active' : ''
          }
        >
          Products
        </NavLink>
      </li>
    </ul>
  )
}
```

흐름은:

```text
현재 URL
   ↓
React Router
   ↓
NavLink
   ↓
isActive
   ↓
menu-active
   ↓
daisyUI UI 표현
```

입니다.

즉 역할은:

```text
React Router
→ 현재 Route 판단

daisyUI
→ Active 상태 표현
```

입니다.

---

# 7. Horizontal Menu와 Vertical Menu

Menu는 방향에 따라 다양하게 사용할 수 있습니다.

Horizontal:

```html
<ul class="menu menu-horizontal">
  <li><a>Home</a></li>
  <li><a>Products</a></li>
  <li><a>About</a></li>
</ul>
```

Vertical:

```html
<ul class="menu menu-vertical">
  <li><a>Dashboard</a></li>
  <li><a>Products</a></li>
  <li><a>Orders</a></li>
</ul>
```

대표적인 사용 위치:

```text
Navbar
  ↓
menu-horizontal


Sidebar
  ↓
menu-vertical
```

입니다.

---

# 8. Menu의 Submenu

Menu 내부에 계층 구조를 만들 수도 있습니다.

```html
<ul class="menu">
  <li>
    <details open>
      <summary>Products</summary>

      <ul>
        <li><a>Electronics</a></li>
        <li><a>Clothing</a></li>
        <li><a>Books</a></li>
      </ul>
    </details>
  </li>
</ul>
```

구조:

```text
Products
│
├─ Electronics
├─ Clothing
└─ Books
```

여기서 `details`와 `summary`는 **HTML Element**입니다.

```text
details / summary
→ HTML의 열기/닫기 동작

menu
→ daisyUI UI 표현
```

이라는 구분이 중요합니다.

---

# 9. Breadcrumbs

현재 페이지가 사이트 구조의 어디에 있는지 보여줄 때 사용합니다.

```html
<div class="breadcrumbs text-sm">
  <ul>
    <li><a>Home</a></li>
    <li><a>Products</a></li>
    <li>Keyboard</li>
  </ul>
</div>
```

결과:

```text
Home
  >
Products
  >
Keyboard
```

Breadcrumbs는 특히 다음과 같은 화면에서 유용합니다.

```text
상품 상세
게시글 상세
관리자 페이지
문서 사이트
다단계 Category
```

---

# 10. Breadcrumbs의 역할

Breadcrumbs와 Menu는 비슷해 보이지만 목적이 다릅니다.

```text
Menu
→ 어디로 이동할 수 있는가?

Breadcrumbs
→ 현재 어디에 있는가?
```

예:

```text
Menu

Home
Products
Orders
Users
```

반면:

```text
Breadcrumbs

Home > Products > Keyboard
```

입니다.

이 차이를 명확하게 이해해야 합니다.

---

# 11. Tabs

같은 화면 안에서 여러 View를 전환할 때 Tabs를 사용할 수 있습니다.

```html
<div role="tablist" class="tabs tabs-border">
  <a role="tab" class="tab tab-active">
    상품 정보
  </a>

  <a role="tab" class="tab">
    리뷰
  </a>

  <a role="tab" class="tab">
    Q&A
  </a>
</div>
```

화면:

```text
상품 정보    리뷰    Q&A
────────
```

Tabs는 보통 **현재 페이지 내부의 View 전환**에 사용합니다.

---

# 12. Menu와 Tabs의 차이

초보자가 자주 혼동하는 부분입니다.

```text
Menu
→ 애플리케이션 Navigation

Tabs
→ 현재 Context 내부 View 전환
```

예를 들어:

```text
Navbar

Home | Products | Cart | MyPage
```

는 Menu입니다.

상품 상세 페이지 안의:

```text
상품 정보 | 리뷰 | Q&A
```

는 Tabs가 더 자연스럽습니다.

---

# 13. Tabs와 상태 관리

daisyUI가 Tab 상태를 React State로 자동 관리해주는 것은 아닙니다.

예:

```jsx
const [tab, setTab] = useState('info')

return (
  <div role="tablist" className="tabs tabs-border">
    <button
      role="tab"
      className={`tab ${
        tab === 'info' ? 'tab-active' : ''
      }`}
      onClick={() => setTab('info')}
    >
      상품 정보
    </button>

    <button
      role="tab"
      className={`tab ${
        tab === 'review' ? 'tab-active' : ''
      }`}
      onClick={() => setTab('review')}
    >
      리뷰
    </button>
  </div>
)
```

흐름:

```text
사용자 클릭
   ↓
setTab()
   ↓
React State
   ↓
tab === ?
   ↓
tab-active
   ↓
daisyUI 표현
```

PART 4 Form과 동일한 원리입니다.

```text
React
→ State / Event

daisyUI
→ UI
```

---

# 14. Drawer

Responsive Layout에서 매우 중요한 Component입니다.

Desktop에서는 Sidebar를 보여주고 Mobile에서는 Button을 눌러 Sidebar를 열도록 만들 수 있습니다.

개념적으로:

```text
Desktop

┌──────────┬─────────────────────┐
│ Sidebar  │ Main Content        │
│          │                     │
│          │                     │
└──────────┴─────────────────────┘


Mobile

┌───────────────────────────────┐
│ ☰  MyShop                    │
├───────────────────────────────┤
│                               │
│ Main Content                  │
│                               │
└───────────────────────────────┘

☰ 클릭

┌──────────────┬────────────────┐
│ Drawer       │                │
│ Menu         │    Overlay     │
│              │                │
└──────────────┴────────────────┘
```

---

# 15. Drawer 기본 구조

```html
<div class="drawer">
  <input
    id="my-drawer"
    type="checkbox"
    class="drawer-toggle"
  />

  <div class="drawer-content">
    <label
      for="my-drawer"
      class="btn btn-primary"
    >
      Menu
    </label>

    <main>
      Content
    </main>
  </div>

  <div class="drawer-side">
    <label
      for="my-drawer"
      class="drawer-overlay"
    ></label>

    <ul class="menu bg-base-200 min-h-full w-80 p-4">
      <li><a>Dashboard</a></li>
      <li><a>Products</a></li>
    </ul>
  </div>
</div>
```

처음 보면 복잡해 보이지만 구조는 단순합니다.

```text
drawer
│
├─ drawer-toggle
│
├─ drawer-content
│    └─ Main Content
│
└─ drawer-side
     ├─ drawer-overlay
     └─ Menu
```

---

# 16. Drawer가 동작하는 핵심

흥미로운 점은 기본 Drawer를 만들기 위해 반드시 JavaScript가 필요한 것은 아니라는 것입니다.

핵심은:

```html
<input
  id="my-drawer"
  type="checkbox"
  class="drawer-toggle"
/>
```

와:

```html
<label for="my-drawer">
```

의 연결입니다.

개념적으로:

```text
Menu Button 클릭
      ↓
<label for="my-drawer">
      ↓
Checkbox 상태 변경
      ↓
drawer-toggle
      ↓
Drawer 열림 / 닫힘
```

즉:

```text
HTML Checkbox
+
CSS
+
daisyUI
```

로 기본 동작을 구현할 수 있습니다.

---

# 17. Responsive Drawer

Desktop에서는 Sidebar를 항상 표시하고 Mobile에서만 Drawer 형태로 만들 수 있습니다.

예:

```html
<div class="drawer lg:drawer-open">
```

핵심:

```text
drawer
→ 기본 Drawer

lg:drawer-open
→ lg 이상에서는 항상 Sidebar Open
```

결과:

```text
Mobile
→ Button으로 열고 닫음

Desktop
→ Sidebar 항상 표시
```

여기서 `lg:`는 Tailwind의 Responsive Variant입니다.

```text
daisyUI
drawer-open

+

Tailwind
lg:

↓

lg:drawer-open
```

daisyUI와 Tailwind CSS가 자연스럽게 결합되는 대표적인 예입니다.

---

# 18. Navbar + Drawer 조합

실제 관리자 페이지에서 자주 사용하는 구조입니다.

```text
┌─────────────────────────────────────┐
│ Navbar                              │
├───────────┬─────────────────────────┤
│           │                         │
│ Sidebar   │ Dashboard               │
│           │                         │
│ Menu      │ Stats                   │
│           │                         │
│           │ Table                   │
│           │                         │
└───────────┴─────────────────────────┘
```

Mobile에서는:

```text
┌─────────────────────┐
│ ☰  Admin            │
├─────────────────────┤
│                     │
│ Dashboard           │
│                     │
└─────────────────────┘
```

이 구조는:

```text
Drawer
│
├─ drawer-content
│    ├─ Navbar
│    └─ Main
│
└─ drawer-side
     └─ Menu
```

로 생각하면 됩니다.

---

# 19. Dock

Mobile App처럼 화면 하단에 Navigation을 배치하고 싶다면 Dock을 사용할 수 있습니다.

개념:

```text
┌──────────────────────────┐
│                          │
│       Main Content       │
│                          │
├──────────────────────────┤
│ Home   Search   Cart     │
└──────────────────────────┘
```

대표적으로:

```text
Home
Search
Cart
MyPage
```

같은 최상위 Navigation에 사용할 수 있습니다.

Desktop Navbar와 Mobile Dock을 조합하는 방식도 가능합니다.

```text
Desktop
→ Navbar

Mobile
→ Dock
```

그리고 Responsive Utility로 표시 여부를 조절할 수 있습니다.

```text
hidden
md:flex
md:hidden
```

---

# 20. Footer

페이지 하단에는 Footer를 사용할 수 있습니다.

```html
<footer class="footer bg-base-200 p-10">
  <nav>
    <h6 class="footer-title">
      Services
    </h6>

    <a class="link link-hover">
      Products
    </a>

    <a class="link link-hover">
      Support
    </a>
  </nav>

  <nav>
    <h6 class="footer-title">
      Company
    </h6>

    <a class="link link-hover">
      About
    </a>

    <a class="link link-hover">
      Contact
    </a>
  </nav>
</footer>
```

Footer 역시 여러 Component와 Utility를 조합합니다.

```text
footer
→ daisyUI Component

footer-title
→ daisyUI Component Class

link
→ daisyUI Component

bg-base-200
→ Theme 기반 Utility

p-10
→ Tailwind Utility
```

---

# 21. Navigation Component를 어떻게 선택할까?

각 Component의 역할을 비교하면 명확해집니다.

| Component   | 주요 역할                  |
| ----------- | ---------------------- |
| Navbar      | 페이지 상단 Navigation      |
| Menu        | Navigation / Action 목록 |
| Breadcrumbs | 현재 위치 표시               |
| Tabs        | 현재 Context 내부 View 전환  |
| Drawer      | Responsive Sidebar     |
| Dock        | 화면 하단 Navigation       |
| Footer      | 페이지 하단 정보/Navigation   |

핵심은 **모양이 아니라 역할을 기준으로 선택하는 것**입니다.

---

# 22. 실제 쇼핑몰 Layout

이제 지금까지의 Component를 조합해 보겠습니다.

```text
┌──────────────────────────────────────────────┐
│ Navbar                                       │
│ Logo       Products   Category       Cart    │
├──────────────────────────────────────────────┤
│ Breadcrumbs                                  │
│ Home > Products > Keyboard                   │
├────────────┬─────────────────────────────────┤
│            │                                 │
│ Menu       │ Main Content                    │
│            │                                 │
│ Category   │ [Card] [Card] [Card]            │
│ Price      │ [Card] [Card] [Card]            │
│ Brand      │                                 │
│            │                                 │
├────────────┴─────────────────────────────────┤
│ Footer                                       │
└──────────────────────────────────────────────┘
```

Mobile에서는:

```text
┌────────────────────────┐
│ ☰  MyShop        Cart  │
├────────────────────────┤
│ Home > Products        │
│                        │
│ [Card]                 │
│ [Card]                 │
│ [Card]                 │
│                        │
├────────────────────────┤
│ Home Search Cart My    │
└────────────────────────┘
```

Sidebar는 Drawer 안으로 들어갑니다.

---

# 23. 실제 관리자 Dashboard Layout

관리자 화면도 거의 같은 원리입니다.

```text
Drawer
│
├─ Sidebar
│   │
│   └─ Menu
│       ├─ Dashboard
│       ├─ Products
│       ├─ Orders
│       └─ Users
│
└─ Content
    │
    ├─ Navbar
    │
    ├─ Breadcrumbs
    │
    └─ Main
        ├─ Stats
        ├─ Table
        └─ Pagination
```

이 구조를 이해하면 이후 PART에서 배울 Data Display Component를 자연스럽게 연결할 수 있습니다.

---

# 24. React 애플리케이션에서는 어떻게 나눌까?

실제 React 프로젝트라면 Layout Component를 분리하는 것이 좋습니다.

```text
src/
│
├─ components/
│   └─ layout/
│       ├─ Navbar.jsx
│       ├─ Sidebar.jsx
│       ├─ Footer.jsx
│       └─ MobileDock.jsx
│
└─ layouts/
    └─ AppLayout.jsx
```

예:

```jsx
function AppLayout() {
  return (
    <div className="drawer lg:drawer-open">
      <Sidebar />

      <div className="drawer-content">
        <Navbar />

        <main className="p-6">
          <Outlet />
        </main>

        <Footer />
      </div>
    </div>
  )
}
```

React Router를 사용한다면:

```text
AppLayout
│
├─ Navbar
├─ Sidebar
│
├─ <Outlet />
│      ↓
│   현재 Route Component
│
└─ Footer
```

가 됩니다.

---

# 25. 중요한 구분: Layout과 Routing

여기에서도 역할을 혼동하면 안 됩니다.

```text
daisyUI
→ Navbar / Menu / Drawer 등의 UI

Tailwind CSS
→ Grid / Flex / Responsive Layout

React Router
→ URL과 화면 연결

React
→ State / Event / Component 구성
```

예를 들어:

```jsx
<NavLink to="/products">
  Products
</NavLink>
```

에서:

```text
NavLink
→ React Router

to="/products"
→ 이동 대상

menu-active
→ daisyUI

flex / gap / lg:
→ Tailwind CSS
```

입니다.

---

# 26. PART 5 핵심 흐름

Navigation을 전체적으로 보면 다음과 같습니다.

```text
사용자
  │
  ▼
Navigation UI
  │
  ├─ Navbar
  ├─ Menu
  ├─ Breadcrumbs
  ├─ Tabs
  ├─ Drawer
  └─ Dock
  │
  ▼
사용자 Action
  │
  ▼
React / React Router
  │
  ├─ State 변경
  │
  └─ URL 변경
  │
  ▼
현재 화면 결정
  │
  ▼
daisyUI
  │
  ▼
Active / Open / Selected UI 표현
```

---

# 27. PART 5 한눈에 정리

```text
                    Application Layout
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     Navigation          Layout          Location
          │                │                │
    ┌─────┼─────┐      ┌───┼───┐      ┌────┼────┐
    ▼     ▼     ▼      ▼       ▼      ▼         ▼
 Navbar  Menu  Dock   Drawer  Footer  Breadcrumbs Tabs
          │
          ▼
   React Router
          │
          ▼
    현재 Route 결정
          │
          ▼
     Main Content
```

**PART 5의 핵심은 이것입니다.**

> **daisyUI는 Navigation과 Layout을 표현하고, Tailwind CSS는 배치와 Responsive 처리를 담당하며, React Router는 실제 Navigation과 현재 Route를 관리합니다.**

이제 `Navbar`, `Menu`, `Drawer`를 조합할 수 있으면 **일반 웹사이트뿐 아니라 쇼핑몰, 게시판, 관리자 Dashboard의 전체 골격을 구성할 수 있습니다.**
