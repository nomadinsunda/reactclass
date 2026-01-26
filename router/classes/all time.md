# ⏰ 1시간차 — SPA·라우팅·History API의 근본 이해하기 🧭

## 1. 왜 “라우팅” 이야기를 React에서 하게 될까?

전통적인 웹에서는 “라우팅”이 **서버의 역할**이었습니다.

* `/` → index.html
* `/about` → about.html
* `/products` → products.html

브라우저는 단순히:

> “이 URL의 HTML 파일 주세요 🙋‍♀️”
> 라고 서버에 요청하고, 서버가 새 HTML을 내려주면 **전체 페이지가 새로 그려지는 방식(MPA, Multi Page Application)**이었죠.

그런데 React 앱은 보통 이렇게 시작합니다:

```html
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
```

그리고 `main.jsx`에서 이렇게 렌더링합니다:

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

📌 즉, **HTML은 딱 1개** (`index.html`)이고, 그 안에 React가 전체 화면을 구성합니다.
이게 바로 **SPA(Single Page Application)**입니다.

> ❗ 문제: 그럼 `/about` 같은 URL로 들어오면 누구 책임?
> 서버? React? 둘 다?

여기서부터 **클라이언트 사이드 라우팅**이 시작됩니다.

---

## 2. MPA vs SPA 라우팅 🥊

### 🔷 MPA 라우팅

* URL 변경 → 서버로 HTTP 요청
* 서버가 새로운 HTML 문서를 내려줌
* 브라우저는 페이지 전체를 다시 그림
* 장점: 단순, SEO에 유리
* 단점: 매번 풀 리로드 → UX 저하

### 🔶 SPA 라우팅

* 초기에 `index.html`만 내려받음
* 이후 화면 전환은 JS(React)가 DOM을 교체
* URL은 바뀌지만 네트워크 요청은 최소화
* 장점: 빠른 화면 전환, 풍부한 UX
* 단점: 직접 라우팅을 구현해야 함, SEO 신경 필요

👉 그래서 SPA에서는 **“URL ↔ 화면 상태”를 직접 매핑**해줘야 합니다.
이 일을 도와주는 대표적인 라이브러리가 바로 **React Router**입니다.

---

## 3. History API: React Router의 비밀 무기 🧨

SPA에서 라우팅을 구현하려면 **새로고침 없이 URL을 바꾸는 기술**이 필요합니다.
HTML5에서 새로 도입된 것이 바로 **History API**입니다.

### 3.1 pushState / replaceState

```js
history.pushState({}, '', '/about')
```

* 주소창이 `/about`으로 바뀝니다.
* 하지만 서버에 네트워크 요청은 가지 않습니다.
* 브라우저는 “히스토리 스택”에 새로운 상태를 추가할 뿐입니다.

```js
history.replaceState({}, '', '/login')
```

* 현재 항목을 교체 (뒤로 가기 시 이전 페이지로 돌아갈 수 없음)

### 3.2 popstate 이벤트

뒤로 가기/앞으로 가기 버튼을 누르면 `popstate` 이벤트가 발생합니다.

```js
window.addEventListener('popstate', (event) => {
  console.log('URL changed!', location.pathname)
})
```

React Router는 이런 이벤트를 감지해서 내부적으로 적절한 컴포넌트를 렌더링합니다.

---

## 4. 아주 간단한 “수제 SPA 라우터” 만들어보기 🛠

React Router 없이, 오로지 History API만으로 간단히 구현하면 감이 잘 옵니다.

```jsx
// router.js
const routes = {
  '/': () => '<h1>Home</h1>',
  '/about': () => '<h1>About</h1>',
}

function render(pathname) {
  const app = document.getElementById('root')
  const page = routes[pathname] ? routes[pathname]() : '<h1>Not Found</h1>'
  app.innerHTML = page
}

window.addEventListener('popstate', () => {
  render(window.location.pathname)
})

export function navigate(pathname) {
  history.pushState({}, '', pathname)
  render(pathname)
}
```

```html
<a href="/about" onclick="event.preventDefault(); navigate('/about')">About</a>
```

📌 여기서 알 수 있는 핵심:

* **라우팅의 본질은 “URL ↔ 렌더링 함수” 매핑**
* React Router는 이 일을 더 편하게/안전하게/확장 가능하게 해 주는 라이브러리

---

## 5. 1시간차 정리 ✨

* 라우팅은 원래 **서버의 일**이었지만, SPA에서는 **클라이언트(React)의 일**이 됨
* SPA에서 URL을 바꾸되 새로고침하지 않는 비밀은 **History API**
* React Router는 History API + 라우트 매칭 + 렌더링 제어를 합친 **라우팅 엔진**

> 👉 2시간차부터는 실제 Vite + React + React Router를 연결하면서
> 실전 라우팅 구성을 하나씩 만들어 보겠습니다! 🚀

---

# ⏰ 2시간차 — React Router 기초 문법: BrowserRouter · Routes · Route · Link · Navigate 🧱

이번 시간의 목표는:

> “React Router를 프로젝트에 붙이고,
> 최소한 **홈 / 소개 / 프로필** 정도는 왔다 갔다 할 수 있는 수준” ✨

---

## 1. React Router 설치 & Vite 프로젝트 연결 ⚙️

### 1.1 설치

```bash
npm install react-router-dom
```

### 1.2 `main.jsx`에서 BrowserRouter로 감싸기

```jsx
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
)
```

> 📌 BrowserRouter:
>
> * 전체 앱의 “라우팅 컨텍스트”를 제공
> * 내부에서 History API를 사용해 URL 변화를 관리합니다.

---

## 2. Routes & Route: URL ↔ 컴포넌트 매핑 🗺

```jsx
// src/App.jsx
import { Routes, Route } from 'react-router-dom'
import Home from './pages/Home'
import About from './pages/About'
import Profile from './pages/Profile'

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/profile" element={<Profile />} />
    </Routes>
  )
}
```

중요 포인트:

* v6부터는 `Switch`가 아니라 `Routes`
* `component={Home}`가 아니라 `element={<Home />}`
* `<Routes>` 안에는 `<Route>`만 와야 함

---

## 3. Link: a 태그 대신 쓰는 네비게이션 컴포넌트 🔗

```jsx
import { Link } from 'react-router-dom'

export default function NavBar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      {' | '}
      <Link to="/about">About</Link>
      {' | '}
      <Link to="/profile">Profile</Link>
    </nav>
  )
}
```

왜 `<a>`를 쓰면 안 되나요?

```jsx
<a href="/about">About</a>
```

* 클릭 시 **브라우저가 서버로 요청을 보내고 전체 페이지 리로드**
* SPA의 이점(빠른 전환) 사라짐
* React 상태도 초기화됨

반면 `Link`는:

* History API로 URL만 변경
* React Router가 해당 URL에 맞는 컴포넌트를 렌더링
* 전체 리렌더 X (필요 부분만 렌더)

---

## 4. NavLink: 활성 메뉴 스타일링 🎨

현재 페이지인지에 따라 스타일을 바꾸고 싶다면 `NavLink`를 활용합니다.

```jsx
import { NavLink } from 'react-router-dom'

export default function NavBar() {
  return (
    <nav>
      <NavLink
        to="/"
        style={({ isActive }) => ({
          fontWeight: isActive ? 'bold' : 'normal',
          color: isActive ? 'tomato' : 'black',
        })}
      >
        Home
      </NavLink>
      {' | '}
      <NavLink to="/about">About</NavLink>
    </nav>
  )
}
```

`style` 혹은 `className`에 콜백을 주면 `isActive`로 분기 가능.

---

## 5. useNavigate: 코드로 이동시키기 🧭

로그인 후 자동으로 `/profile`로 이동시키는 경우:

```jsx
import { useNavigate } from 'react-router-dom'

export default function Login() {
  const navigate = useNavigate()

  function handleLogin() {
    // 1. 로그인 처리 (가짜)
    localStorage.setItem('token', 'abc123')

    // 2. 페이지 이동
    navigate('/profile')
  }

  return <button onClick={handleLogin}>Login</button>
}
```

옵션:

```js
navigate('/profile', { replace: true })
```

* `replace: true` → 뒤로 가기 시 로그인 페이지로 돌아가지 않게 하고 싶을 때

---

## 6. useLocation: 현재 URL 정보 읽기 🕵️‍♂️

```jsx
import { useLocation } from 'react-router-dom'

export default function DebugLocation() {
  const location = useLocation()

  return (
    <pre>{JSON.stringify(location, null, 2)}</pre>
  )
}
```

`location`에는:

* `pathname` → `/about`
* `search` → `?keyword=react`
* `hash` → `#section-1`
* `state` → `navigate('/detail', { state: {...} })`로 전달한 상태

---

## 7. 2시간차 실습 구성 ✍️

* `/`, `/about`, `/profile` 라우트 구성
* NavBar에서 `Link` / `NavLink` 사용
* `Login` 페이지를 만들어서 로그인 버튼 클릭 시 `/profile`로 이동
* `DebugLocation` 컴포넌트를 만들어 화면 구석에 찍어보기

---

# ⏰ 3시간차 — 중첩 라우팅 & 레이아웃 패턴 (Dashboard 예제) 🏗

이번 시간의 키워드: **`Outlet`**, **Layout Route**, **index route** ✨

---

## 1. 왜 중첩 라우팅이 필요한가?

예를 들어 관리자 페이지:

* `/admin` → 공통 레이아웃 + 대시보드
* `/admin/users` → 공통 레이아웃 + 사용자 목록
* `/admin/settings` → 공통 레이아웃 + 설정

공통인 것:

* 상단 Navbar
* 사이드바 메뉴
* 푸터

매 페이지마다 복붙하면?

* 유지보수 지옥 🥲
* 변경 시 모든 파일 수정

👉 라우트 레벨에서 “레이아웃”을 정의하고, 그 안에 **자식 라우트들을 중첩**하는 패턴이 필요합니다.

---

## 2. Layout Route와 Outlet 🧩

```jsx
// App.jsx
<Routes>
  <Route path="/" element={<Home />} />

  <Route path="/admin" element={<AdminLayout />}>
    <Route index element={<AdminDashboard />} />
    <Route path="users" element={<AdminUsers />} />
    <Route path="settings" element={<AdminSettings />} />
  </Route>
</Routes>
```

`AdminLayout`:

```jsx
import { Outlet, NavLink } from 'react-router-dom'

export default function AdminLayout() {
  return (
    <div className="admin-layout">
      <aside>
        <h2>Admin</h2>
        <nav>
          <NavLink to="">Dashboard</NavLink>
          <NavLink to="users">Users</NavLink>
          <NavLink to="settings">Settings</NavLink>
        </nav>
      </aside>

      <main>
        <Outlet /> {/* 여기 자식 라우트가 렌더링됩니다 */}
      </main>
    </div>
  )
}
```

핵심:

* 부모 `<Route>`: `element={<AdminLayout />}`
* 자식 `<Route>`들: `<Route path="users" ... />`처럼 **상대 경로**
* `<Outlet />`: 자식 라우트 화면이 이름 그대로 “빠져나와서” 렌더링되는 자리

---

## 3. index Route: `/admin` 기본 화면 지정하기

```jsx
<Route path="/admin" element={<AdminLayout />}>
  <Route index element={<AdminDashboard />} />   // path 없이 index
  <Route path="users" element={<AdminUsers />} />
</Route>
```

* `/admin` → `AdminDashboard`
* `/admin/users` → `AdminUsers`

---

## 4. 중첩 라우팅에서 경로 작성 시 주의점 ⚠️

잘못된 예:

```jsx
<Route path="/admin" element={<AdminLayout />}>
  <Route path="/admin/users" element={<AdminUsers />} /> // ❌
</Route>
```

* 자식 라우트 경로에 `/admin`을 다시 쓰면 안 됩니다.
* 자식은 **부모 기준 “상대 경로”**로 작성해야 합니다.

정답:

```jsx
<Route path="/admin" element={<AdminLayout />}>
  <Route path="users" element={<AdminUsers />} /> // ✅
</Route>
```

---

## 5. 3시간차 실습: Dashboard 레이아웃 구축 🧱

* `/admin` 레이아웃 구성
* 사이드바 + Outlet 영역 분리
* `/admin`, `/admin/users`, `/admin/settings` 3개 자식 라우트 구성
* `NavLink`로 현재 메뉴 강조

---

# ⏰ 4시간차 — 동적 라우팅 & URL Params & Search Params 🔍

이번 시간의 키워드: **`:id`**, **`useParams`**, **쿼리스트링**, **`useSearchParams`**

---

## 1. 동적 세그먼트: `/products/:id`

```jsx
<Route path="/products/:id" element={<ProductDetail />} />
```

이제 `/products/10`, `/products/20` 모두 `ProductDetail`로 매칭됩니다.

```jsx
import { useParams } from 'react-router-dom'

export default function ProductDetail() {
  const { id } = useParams()

  return <h1>Product Detail: {id}</h1>
}
```

---

## 2. 중첩 + 동적 라우팅: `/products/:id/reviews`

```jsx
<Route path="/products" element={<ProductsLayout />}>
  <Route index element={<ProductList />} />
  <Route path=":id" element={<ProductDetail />} />
  <Route path=":id/reviews" element={<ProductReviews />} />
</Route>
```

---

## 3. Search Params (쿼리스트링) 다루기

### 3.1 `useSearchParams`

```jsx
import { useSearchParams } from 'react-router-dom'

export default function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams()
  const keyword = searchParams.get('keyword') ?? ''

  function handleSearch(e) {
    setSearchParams({ keyword: e.target.value })
  }

  return (
    <div>
      <input
        value={keyword}
        onChange={handleSearch}
        placeholder="검색어 입력"
      />
      {/* keyword를 기준으로 필터링된 리스트 렌더링 */}
    </div>
  )
}
```

* URL: `/products?keyword=bag`
* 새로고침해도 검색어 유지 💡

---

## 4. `location.state`로 추가 상태 전달하기 📦

자주 쓰는 패턴:

* 목록에서 카드 클릭 → 상세 페이지로 이동
* 상세 페이지에서 “목록에서 어떤 워크플로우로 왔는지” 알고 싶을 때

```jsx
// 목록에서
navigate(`/products/${id}`, {
  state: { from: 'list', keyword }
})
```

```jsx
import { useLocation } from 'react-router-dom'

export default function ProductDetail() {
  const location = useLocation()
  console.log(location.state) // { from: 'list', keyword: 'bag' } 등
}
```

---

## 5. 4시간차 실습 🧪

* `/products` → 목록
* `/products/:id` → 상세
* `/products?keyword=xx` → 검색어 유지
* 상세 페이지에서 `useParams`, `useLocation` 모두 활용

---

# ⏰ 5시간차 — Protected Route, 인증/인가 라우팅 구조 🛡

이제 실제 서비스에 가까운 “접근 제어”를 라우팅에 녹여봅니다.

---

## 1. Protected Route 패턴의 기본 아이디어

> “로그인하지 않은 사용자는 특정 페이지에 접근할 수 없게 막고,
> 로그인 페이지로 보내자!”

구현 방법:

1. `isLoggedIn`을 판단할 수 있는 무언가가 필요 (예: 토큰, 전역 상태 등)
2. 라우트 렌더링 전에 `isLoggedIn` 확인
3. 아니면 `<Navigate to="/login" />`를 렌더링

---

## 2. 단순 버전: LocalStorage 기반 🧪

```jsx
function ProtectedRoute({ element }) {
  const isLoggedIn = !!localStorage.getItem('token')
  return isLoggedIn ? element : <Navigate to="/login" replace />
}
```

사용:

```jsx
<Route
  path="/mypage"
  element={<ProtectedRoute element={<MyPage />} />}
/>
```

---

## 3. 레이아웃 수준에서 보호하기

`/dashboard/*` 전체를 보호하고 싶다면:

```jsx
function ProtectedLayout() {
  const isLoggedIn = !!localStorage.getItem('token')
  if (!isLoggedIn) return <Navigate to="/login" replace />
  return <DashboardLayout />  // 안에 Outlet 있음
}
```

라우팅:

```jsx
<Route path="/dashboard" element={<ProtectedLayout />}>
  <Route index element={<DashboardHome />} />
  <Route path="reports" element={<DashboardReports />} />
</Route>
```

---

## 4. Role 기반 인가 (Admin / User) 🔐

```jsx
function AdminRoute({ element }) {
  const user = JSON.parse(localStorage.getItem('user') || '{}')
  if (!user.token) return <Navigate to="/login" replace />
  if (user.role !== 'admin') return <Navigate to="/forbidden" replace />

  return element
}
```

---

## 5. 5시간차 실습 🧑‍💻

* `/login` 페이지에서 “로그인 버튼 클릭 시 토큰 저장”
* `/mypage`는 로그인해야만 접근 가능
* 로그인 안된 상태로 `/mypage` 접근 → `/login`으로 리다이렉트
* `/admin`은 role === 'admin'만 접근 가능

---

# ⏰ 6시간차 — Data API(Loader/Action) & 서버 상호작용 패턴 ⚙️

이 시간은 React Router v6.4+의 **Data API(선택)**를 소개하는 파트로 쓸 수 있습니다.
(이미 이전 시간까지의 내용만으로도 충분히 깊지만, 개념 확장용으로 좋습니다.)

---

## 1. 왜 Data API가 나왔나?

기존 패턴:

* 컴포넌트 내부의 `useEffect`에서 fetch
* 로딩 상태, 에러 상태, 데이터 상태를 전부 컴포넌트에서 관리
* 라우트가 바뀔 때마다 이런 패턴 반복 → 중복, 복잡

Data API 아이디어:

> “라우트에 진입하기 전에 데이터를 먼저 불러오고,
> 컴포넌트에서는 그냥 `useLoaderData()`로 사용하자!”

---

## 2. Loader 기본 예제

```jsx
// usersLoader.js
export async function usersLoader() {
  const res = await fetch('/api/users')
  if (!res.ok) throw new Response('Failed to fetch', { status: res.status })
  return res.json()
}
```

라우트 구성:

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom'
import UsersPage, { usersLoader } from './UsersPage'

const router = createBrowserRouter([
  {
    path: '/users',
    element: <UsersPage />,
    loader: usersLoader,
  },
])

function App() {
  return <RouterProvider router={router} />
}
```

컴포넌트:

```jsx
import { useLoaderData } from 'react-router-dom'

export function UsersPage() {
  const users = useLoaderData()
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  )
}
```

---

## 3. Action: 폼 전송 처리

```jsx
export async function createUserAction({ request }) {
  const formData = await request.formData()
  const name = formData.get('name')

  await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ name }),
  })

  return redirect('/users')
}
```

컴포넌트에서 `Form` 사용:

```jsx
import { Form } from 'react-router-dom'

export function NewUserPage() {
  return (
    <Form method="post">
      <input name="name" />
      <button type="submit">Create</button>
    </Form>
  )
}
```

---

## 4. 6시간차 실습 💾

* `/users` → loader로 유저 목록 미리 로딩
* `/users/new` → action으로 신규 유저 생성 처리
* 에러 발생 시 에러 페이지 혹은 메시지 렌더링

---

# ⏰ 7시간차 — 코드 스플리팅 & 성능 최적화 패턴 ⚡

이번 시간은 `React.lazy`, `Suspense`, 라우트 단위 코드 분할 얘기입니다.

---

## 1. 왜 코드 스플리팅이 필요한가?

* SPA는 보통 **번들 하나가 매우 커지기 쉬움**
* 유저가 당장 안 들어갈 admin 페이지, settings 페이지까지
  초기에 다 내려받는 건 낭비

👉 **라우트 단위로 코드를 분리해서, 실제 진입할 때만 로드**하면 좋음.

---

## 2. React.lazy + Suspense

```jsx
import React, { Suspense } from 'react'
const About = React.lazy(() => import('./pages/About'))

export default function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <About />
    </Suspense>
  )
}
```

React Router와 함께:

```jsx
const Home = React.lazy(() => import('./pages/Home'))
const About = React.lazy(() => import('./pages/About'))

<Routes>
  <Route
    path="/"
    element={
      <Suspense fallback={<Spinner />}>
        <Home />
      </Suspense>
    }
  />
  <Route
    path="/about"
    element={
      <Suspense fallback={<Spinner />}>
        <About />
      </Suspense>
    }
  />
</Routes>
```

---

## 3. Suspense 범위를 어떻게 나눌까?

* 페이지 단위로 wrapping
* 레이아웃 단위로 wrapping
* 특정 위젯만 lazy 로딩

실무에서 자주 쓰는 패턴:

```jsx
function LazyPage(Component) {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Component />
    </Suspense>
  )
}

<Route path="/about" element={LazyPage(About)} />
```

---

## 4. 7시간차 실습 ⚙️

* `/`, `/about`, `/products`, `/admin` 페이지를 모두 lazy 로딩
* 모든 라우트에 공통 `PageSkeleton` 적용
* DevTools Network 탭에서 실제로 JS 로딩이 분리되는지 확인해보기

---

# ⏰ 8시간차 — 종합 미니 프로젝트: 쇼핑몰 라우팅 구축 🛒

마지막 1시간은 지금까지 배운 것을 **하나의 라우팅 구조로 묶어보는 시간**입니다.

---

## 1. 목표 라우팅 구조

```text
/
├── /products
│     ├── index (상품 목록)
│     ├── :id (상품 상세)
│     └── :id/reviews (리뷰)
├── /cart
├── /login
├── /mypage (로그인 필요)
└── * (NotFound)
```

---

## 2. 라우팅 코드 스케치

```jsx
<BrowserRouter>
  <Layout> {/* 헤더/푸터 공통 */}
    <Routes>
      <Route path="/" element={<Home />} />
      
      <Route path="/products" element={<ProductsLayout />}>
        <Route index element={<ProductList />} />
        <Route path=":id" element={<ProductDetail />} />
        <Route path=":id/reviews" element={<ProductReviews />} />
      </Route>

      <Route path="/cart" element={<Cart />} />
      <Route path="/login" element={<Login />} />

      <Route
        path="/mypage"
        element={<ProtectedRoute element={<MyPage />} />}
      />

      <Route path="*" element={<NotFound />} />
    </Routes>
  </Layout>
</BrowserRouter>
```

---

## 3. 실습 포인트 체크리스트 ✅

* [ ] Layout + Outlet 구조 이해
* [ ] Nested route (products)
* [ ] Dynamic route (`:id`)
* [ ] Search params (`?keyword=xxx`)
* [ ] Protected route (`/mypage`)
* [ ] NotFound 처리
* [ ] 필요하다면 lazy 로딩까지 적용

---

## 4. 마무리 멘트 🧡

이 8시간 동안 다룬 내용은 React Router의:

* **개념적 근간(SPA·History API)**
* **기본 사용법(BrowserRouter, Routes, Route, Link, Navigate)**
* **구조 설계(Nested Routes, Layout, Outlet)**
* **실용 기능(Dynamic Route, Search Params, Location State)**
* **실전 패턴(Protected Route, Role-based Access)**
* **성능 최적화(lazy, Suspense, 코드 스플리팅)**
* **Data API(Loader/Action)의 방향성**

까지 포함하는 꽤 탄탄한 전체 스펙트럼입니다 🚀

