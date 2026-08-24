# React Router의 `<Link>`와 `<Route>`

React Router를 처음 배울 때 `<Link>`와 `<Route>`를 함께 접하게 됩니다.

둘 다 라우팅과 관련되어 있지만 역할은 완전히 다릅니다.

```text
<Link>                           <Route>

"어디로 이동할 것인가?"          "그 URL에서 무엇을 보여줄 것인가?"
          │                                  │
          ▼                                  ▼
      Navigation                         Route Rule
```

가장 중요한 차이를 한 문장으로 정리하면 다음과 같습니다.

> **`<Link>`는 URL을 변경하는 네비게이션 UI이고, `<Route>`는 특정 URL 패턴에 어떤 UI를 연결할지 선언하는 라우팅 규칙입니다.**

---

# 1. 가장 간단한 예제부터 이해하기

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link
} from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

이 코드에는 서로 다른 역할을 담당하는 세 가지 핵심 요소가 있습니다.

```text
<Link>
   │
   │ 사용자가 클릭
   ▼
URL 변경
   │
   ▼
<Routes>
   │
   │ 현재 URL과 Route Tree를 매칭
   ▼
<Route>
   │
   │ 매칭된 element
   ▼
React Component
```

예를 들어 사용자가 다음 링크를 클릭하면:

```jsx
<Link to="/about">About</Link>
```

URL이:

```text
/
↓
/about
```

으로 변경됩니다.

그러면 `<Routes>`가 현재 URL과 Route 설정을 비교합니다.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

`/about`과 매칭되는 Route가 있으므로:

```jsx
<Route
  path="/about"
  element={<About />}
/>
```

의 `element`인:

```jsx
<About />
```

이 렌더링됩니다.

따라서 전체 흐름은:

```text
사용자
  │
  │ click
  ▼
<Link to="/about">
  │
  ▼
URL → /about
  │
  ▼
<Routes>
  │
  │ Route Matching
  ▼
<Route path="/about">
  │
  ▼
<About />
```

입니다.

---

# 2. `<Link>` — 어디로 이동할 것인가?

## 정의

`<Link>`는 React Router에서 **클라이언트 사이드 네비게이션을 제공하는 컴포넌트**입니다.

```jsx
<Link to="/about">
  About
</Link>
```

사용자가 클릭하면 `/about`으로 이동합니다.

하지만 일반적인 HTML `<a>`를 통한 문서 이동과 중요한 차이가 있습니다.

---

# 3. `<a>`와 `<Link>`의 차이

일반 HTML에서는:

```html
<a href="/about">About</a>
```

를 클릭하면 브라우저의 기본 navigation이 발생할 수 있습니다.

개념적으로:

```text
<a href="/about">
       │
       ▼
Browser Navigation
       │
       ▼
GET /about
       │
       ▼
새 HTML Document
       │
       ▼
페이지 전체 로드
```

React Router의 `<Link>`를 사용하면 같은 SPA 내부 경로에서는 클라이언트 사이드 navigation을 수행합니다.

```jsx
<Link to="/about">
  About
</Link>
```

개념적으로:

```text
<Link>
   │
   │ click
   ▼
React Router
   │
   │ History API 기반 navigation
   ▼
URL 변경
/about
   │
   ▼
Route Matching
   │
   ▼
<About />
```

따라서 일반적인 SPA 내부 이동에서는 새로운 HTML 문서를 받아 애플리케이션 전체를 처음부터 시작하는 방식이 아니라, **현재 React 애플리케이션을 유지한 상태에서 URL과 렌더링되는 UI가 변경됩니다.**

---

# 4. `<Link>`는 실제로 무엇을 렌더링하는가?

`<Link>`는 브라우저 DOM에서는 기본적으로 링크 역할을 하는 `<a>` 엘리먼트로 렌더링됩니다.

React:

```jsx
<Link to="/about">
  About
</Link>
```

브라우저에서 개념적으로:

```html
<a href="/about">
  About
</a>
```

와 같은 링크가 만들어집니다.

그러나 React Router가 일반적인 내부 링크 클릭을 처리하여 브라우저의 전체 문서 navigation 대신 **SPA navigation**을 수행합니다.

따라서:

```text
<Link>

UI 관점
    ↓
링크를 제공

DOM 관점
    ↓
<a> 기반

Router 관점
    ↓
클라이언트 사이드 Navigation
```

이라고 이해하면 좋습니다.

---

# 5. `<Link>`의 핵심 prop — `to`

가장 중요한 prop은 `to`입니다.

```jsx
<Link to="/about">About</Link>
```

`to`는 **이동할 위치**를 지정합니다.

### 절대 경로

```jsx
<Link to="/products">
  Products
</Link>
```

### 상대 경로

```jsx
<Link to="settings">
  Settings
</Link>
```

중첩 라우팅에서 특히 유용합니다.

예를 들어 현재 Route가 `/dashboard`이고 적절한 중첩 Route 문맥에서:

```jsx
<Link to="settings">
```

를 사용하면:

```text
/dashboard/settings
```

같은 경로를 구성할 수 있습니다.

---

# 6. `replace`

기본적인 navigation은 새로운 History Entry를 추가합니다.

```text
/history

/home
  ↓
/products
  ↓
/products/10
```

따라서 Back 버튼을 누르면 이전 위치로 돌아갈 수 있습니다.

하지만:

```jsx
<Link to="/login" replace>
  Login
</Link>
```

처럼 `replace`를 사용하면 현재 History Entry를 새로운 위치로 교체하는 navigation을 요청합니다.

개념적으로:

```text
일반 Link

A → B → C
        ↑
       push


replace

A → C
    ↑
현재 entry 교체
```

---

# 7. `state`

URL 이외의 추가 데이터를 navigation과 함께 전달할 수도 있습니다.

```jsx
<Link
  to="/detail"
  state={{ from: "home" }}
>
  Detail
</Link>
```

대상 컴포넌트에서는:

```jsx
import { useLocation } from "react-router-dom";

function Detail() {
  const location = useLocation();

  console.log(location.state);

  return <div>Detail</div>;
}
```

처럼 읽을 수 있습니다.

다만 중요한 점이 있습니다.

> `state`는 URL에 표시되지 않는다는 뜻이지, 보안 저장소라는 뜻은 아닙니다.

비밀번호, 토큰 같은 민감한 정보를 저장하는 용도로 사용해서는 안 됩니다.

---

# 8. `<Link>`는 Router 안에서 사용한다

`<Link>`는 React Router의 Router context를 사용합니다.

따라서 일반적으로:

```jsx
<BrowserRouter>
  <Link to="/about">
    About
  </Link>
</BrowserRouter>
```

처럼 Router 아래에서 사용해야 합니다.

구조적으로:

```text
BrowserRouter
│
├── Link
│
├── Link
│
└── Routes
    │
    ├── Route
    └── Route
```

가 됩니다.

---

# 9. `<Route>` — URL에서 무엇을 보여줄 것인가?

이번에는 `<Route>`입니다.

```jsx
<Route
  path="/about"
  element={<About />}
/>
```

이 코드는:

> 현재 위치가 이 Route의 `path`와 매칭될 때 사용할 UI는 `<About />`이다.

라는 **라우팅 규칙**을 선언합니다.

따라서 `<Route>`를 단순히 “페이지를 이동시키는 컴포넌트”라고 이해하면 안 됩니다.

```text
<Route>

Navigation       X
URL 변경         X

Route Rule       O
Path 정의        O
UI 연결          O
```

---

# 10. `<Route>`의 핵심 props

대표적으로 `path`와 `element`를 먼저 이해하면 됩니다.

## `path`

매칭할 URL 패턴을 정의합니다.

```jsx
<Route
  path="/about"
  element={<About />}
/>
```

```text
/about
   │
   ▼
path="/about"
```

동적 세그먼트도 사용할 수 있습니다.

```jsx
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

예를 들어:

```text
/users/10
/users/20
/users/100
```

등이 해당 패턴에 매칭될 수 있습니다.

---

# 11. `element`

`element`는 해당 Route가 매칭되었을 때 사용할 React element입니다.

```jsx
<Route
  path="/about"
  element={<About />}
/>
```

중요한 것은:

```jsx
element={<About />}
```

처럼 **React element를 전달한다는 것**입니다.

다음과 혼동하면 안 됩니다.

```jsx
element={About}
```

기본적인 v6 스타일의 Route에서는 위와 같이 컴포넌트 함수 자체를 `element`에 전달하는 방식이 아닙니다.

---

# 12. `<Route>` 하나만 보는 것보다 `<Routes>`와 함께 이해하자

여기서 매우 중요한 구분이 있습니다.

`<Route>` 하나가 모든 Route를 돌아다니면서 URL을 찾는다고 생각하면 정확하지 않습니다.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="/products" element={<Products />} />
</Routes>
```

역할을 구분하면:

```text
<Routes>
    │
    │ 현재 location을 기준으로
    │ Route Tree를 매칭
    ▼

<Route path="/">
<Route path="/about">
<Route path="/products">

    │
    ▼

매칭된 Route branch의
element 렌더링
```

따라서 입문 단계에서는 다음처럼 구분하는 것이 좋습니다.

| 구성 요소      | 핵심 역할                       |
| ---------- | --------------------------- |
| `<Link>`   | navigation을 시작하는 UI         |
| `<Routes>` | 현재 location과 Route Tree를 매칭 |
| `<Route>`  | path와 UI의 관계를 선언            |

이 세 가지를 함께 이해해야 React Router의 구조가 정확하게 보입니다.

---

# 13. `<Route>`는 선언적 조건부 렌더링과 비슷하다

React Router가 없다면 아주 단순한 경우 다음과 비슷한 코드를 생각할 수 있습니다.

```jsx
if (pathname === "/") {
  return <Home />;
}

if (pathname === "/about") {
  return <About />;
}
```

React Router에서는 이를 라우팅 구조로 선언합니다.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

하지만 실제 React Router의 경로 매칭은 단순한 문자열 `===` 비교가 아닙니다.

동적 세그먼트, 중첩 Route, splat 등 다양한 Route 패턴을 처리합니다.

따라서 위 `if` 코드는 **개념을 이해하기 위한 단순화된 비유**입니다.

---

# 14. 중첩 `<Route>`

`<Route>`는 중첩할 수도 있습니다.

```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />

    <Route path="about" element={<About />} />

    <Route path="users">
      <Route index element={<UserList />} />
      <Route path=":userId" element={<UserDetail />} />
    </Route>
  </Route>
</Routes>
```

Route Tree로 보면:

```text
/
│
├── index
│   └── Home
│
├── about
│   └── About
│
└── users
    │
    ├── index
    │   └── UserList
    │
    └── :userId
        └── UserDetail
```

그리고 부모 Route의 element 안에 `<Outlet />`이 있다면 매칭된 자식 Route의 element가 그 위치에 렌더링됩니다.

```jsx
function Layout() {
  return (
    <>
      <Header />

      <Outlet />

      <Footer />
    </>
  );
}
```

즉:

```text
<Route>
   │
   │ Nested Route
   ▼
부모 element
   │
   ▼
<Outlet />
   │
   ▼
자식 Route의 element
```

입니다.

---

# 15. `<Link>`에서 `<Route>`까지 전체 동작

이제 둘을 하나의 흐름으로 연결해 보겠습니다.

다음 코드가 있습니다.

```jsx
<Link to="/about">
  About
</Link>

<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

사용자가 `About`을 클릭합니다.

### STEP 1 — 사용자 클릭

```text
User
 │
 ▼
<Link to="/about">
```

### STEP 2 — Client-side Navigation

React Router가 링크 클릭을 처리합니다.

```text
<Link>
   │
   ▼
Router Navigation
   │
   ▼
History
```

### STEP 3 — Location 변경

```text
location.pathname

/
↓
/about
```

### STEP 4 — Route Matching

`<Routes>`가 현재 location을 기준으로 Route Tree를 매칭합니다.

```text
/about
   │
   ▼
<Routes>
   │
   ├── path="/"
   │
   └── path="/about"  ← match
```

### STEP 5 — UI 렌더링

매칭된 Route의 element가 사용됩니다.

```text
<Route
  path="/about"
  element={<About />}
/>
        │
        ▼
     <About />
        │
        ▼
      화면
```

따라서 전체 과정은:

```text
사용자 클릭
    │
    ▼
  <Link>
    │
    │ navigation
    ▼
 URL 변경
    │
    ▼
location 변경
    │
    ▼
 <Routes>
    │
    │ Route Matching
    ▼
 <Route>
    │
    │ element
    ▼
React Component
    │
    ▼
   화면
```

이 흐름이 `<Link>`와 `<Route>`의 관계를 이해하는 가장 중요한 그림입니다.

---

# 16. `<Link>`와 `<Route>`의 결정적인 차이

| 구분       | `<Link>`       | `<Route>`                  |
| -------- | -------------- | -------------------------- |
| 핵심 질문    | 어디로 갈 것인가?     | 이 경로에서 무엇을 보여줄 것인가?        |
| 역할       | Navigation UI  | Route Rule                 |
| 핵심 prop  | `to`           | `path`, `element`          |
| 사용자 클릭   | 일반적으로 관련 있음    | 필요 없음                      |
| URL 변경   | navigation을 시작 | 직접 변경하지 않음                 |
| Route 매칭 | 하지 않음          | 매칭 대상 규칙을 제공               |
| DOM/UI   | 링크 UI를 렌더링     | Route configuration 성격이 강함 |
| 사용 위치    | 메뉴, 내비게이션 등    | `<Routes>` 내부의 Route Tree  |

---

# 17. 자주 하는 오해

### 오해 1 — `<Link>`가 페이지를 렌더링한다

아닙니다.

```text
<Link>
   ↓
Navigation
   ↓
URL 변경
```

`<Link>`의 핵심 역할은 navigation입니다.

---

### 오해 2 — `<Route>`가 URL을 변경한다

아닙니다.

```text
<Route>

path="/about"
      +
element={<About />}
```

처럼 **URL 패턴과 UI의 관계를 선언**합니다.

---

### 오해 3 — `<Route>`는 `<Link>`를 클릭해야만 동작한다

그렇지 않습니다.

Route Matching의 기준은 **현재 location**입니다.

예를 들어 앱이 `/about` 위치에서 시작하면 `<Link>` 클릭이 없어도 `/about`에 해당하는 Route가 매칭될 수 있습니다.

---

### 오해 4 — `<Link>`와 `<Route>`가 직접 서로 연결되어 있다

다음 두 코드가 직접 연결되는 것은 아닙니다.

```jsx
<Link to="/about">
```

```jsx
<Route path="/about">
```

둘 사이를 연결하는 것은 **현재 URL/location과 Router의 매칭 과정**입니다.

```text
<Link>
   │
   ▼
URL / Location
   │
   ▼
<Routes>
   │
   ▼
<Route>
```

이 점이 매우 중요합니다.

---

# 18. `<Link>`, `<Routes>`, `<Route>`를 하나의 시스템으로 이해하기

React Router를 처음 공부할 때는 각각을 따로 외우기보다 다음 세 문장으로 기억하는 것이 좋습니다.

> **`<Link>` — URL을 어디로 바꿀 것인가?**

> **`<Routes>` — 현재 URL과 어떤 Route branch가 매칭되는가?**

> **`<Route>` — 그 경로에서 어떤 UI를 사용할 것인가?**

이를 그림으로 표현하면:

```text
┌──────────────┐
│    <Link>    │
│              │
│ to="/about"  │
└──────┬───────┘
       │
       │ navigation
       ▼
┌──────────────┐
│   Location   │
│              │
│   /about     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   <Routes>   │
│              │
│ Route Match  │
└──────┬───────┘
       │
       ▼
┌─────────────────────────┐
│        <Route>          │
│                         │
│ path="/about"           │
│ element={<About />}     │
└───────────┬─────────────┘
            │
            ▼
      ┌───────────┐
      │  <About>  │
      └───────────┘
```

---

# 최종 정리

`<Link>`와 `<Route>`의 차이는 결국 **Navigation과 Routing Rule의 차이**입니다.

```text
<Link>
"어디로 갈 것인가?"
       │
       ▼
   Navigation
       │
       ▼
      URL
       │
       ▼
   Route Matching
       │
       ▼
<Route>
"무엇을 보여줄 것인가?"
       │
       ▼
     element
       │
       ▼
      화면
```

따라서 강의에서는 다음 문장으로 정리하는 것이 가장 직관적입니다.

> **`<Link>`는 사용자가 이동할 URL을 선택하게 하고, `<Route>`는 그 URL 패턴에 어떤 UI를 연결할지 선언한다. 그리고 `<Routes>`가 현재 URL을 기준으로 Route Tree를 매칭한다.**


