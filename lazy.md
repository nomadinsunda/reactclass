# `React.lazy()` — 컴포넌트를 필요할 때 로드하는 React의 Code Splitting 도구

React 애플리케이션의 규모가 커지면 JavaScript 코드도 함께 커집니다.

```text
Home
About
Dashboard
Admin
Chart
Editor
Settings
...
```

그런데 사용자가 처음 접속한 페이지가 `/` 하나뿐이라면, 처음부터 모든 페이지의 JavaScript가 필요한 것은 아닙니다.

이 문제를 해결하는 대표적인 방법이 **Code Splitting**이고, React에서는 컴포넌트 단위의 Code Splitting을 쉽게 연결하기 위해 `React.lazy()`를 사용할 수 있습니다.

한 문장으로 먼저 정리하면:

> **`React.lazy()`는 컴포넌트 모듈을 처음부터 즉시 로드하지 않고, 해당 컴포넌트가 실제 렌더링에 필요해졌을 때 로드하도록 React의 렌더링 과정과 `import()`를 연결하는 API입니다.**

핵심 구조는 다음과 같습니다.

```text
큰 JavaScript Bundle
        ↓
   Code Splitting
        ↓
여러 JavaScript Chunk
        ↓
필요한 시점에 import()
        ↓
    React.lazy()
        ↓
로딩 중 → <Suspense fallback>
        ↓
로딩 완료 → 실제 Component
```

---

# 1. 왜 `React.lazy()`가 필요한가?

SPA(Single Page Application)는 애플리케이션 규모가 커질수록 JavaScript 코드도 커지는 경향이 있습니다.

예를 들어 다음과 같은 페이지가 있다고 하겠습니다.

```text
/
├── Home
├── About
├── Dashboard
├── Admin
├── Statistics
├── Chart
└── Editor
```

모든 컴포넌트를 일반적인 `import`로 가져오면:

```jsx
import Home from "./pages/Home";
import About from "./pages/About";
import Dashboard from "./pages/Dashboard";
import Admin from "./pages/Admin";
```

이 모듈들은 애플리케이션의 정적 모듈 의존성에 포함됩니다.

빌드 결과에서도 초기 실행에 필요한 JavaScript가 커질 수 있습니다.

개념적으로:

```text
사용자
  │
  │ /
  ▼
브라우저
  │
  │ 처음부터 많은 JavaScript 다운로드
  ▼
┌───────────────────────────┐
│ Home                      │
│ About                     │
│ Dashboard                 │
│ Admin                     │
│ Chart                     │
│ Editor                    │
│ Settings                  │
└───────────────────────────┘
```

하지만 사용자는 현재:

```text
/
```

페이지만 보고 있을 수 있습니다.

그런데 아직 방문하지도 않은:

```text
/admin
/dashboard
/settings
```

관련 코드까지 처음부터 가져와야 한다면 초기 로딩 비용이 커질 수 있습니다.

그래서 다음과 같은 아이디어가 등장합니다.

> **지금 당장 필요하지 않은 코드는 별도의 파일로 나누고, 실제 필요해졌을 때 가져오자.**

이것이 **Code Splitting**입니다.

---

# 2. 먼저 Code Splitting을 이해해야 한다

Code Splitting은 큰 JavaScript 코드를 여러 개의 작은 단위로 나누는 전략입니다.

기존:

```text
bundle.js
──────────────────────────────
Home
About
Dashboard
Admin
Chart
Editor
Settings
──────────────────────────────
```

Code Splitting 적용 후에는 개념적으로:

```text
main.js
├── Home 관련 코드
└── 공통 코드

About-xxxx.js
Dashboard-xxxx.js
Admin-xxxx.js
Chart-xxxx.js
...
```

처럼 나눌 수 있습니다.

그러면 애플리케이션 시작 시 필요한 코드만 먼저 로드하고:

```text
첫 접속

Browser
  │
  └── main.js
```

나중에 `/admin`으로 이동했을 때:

```text
/admin 이동
    │
    ▼
Admin chunk 필요
    │
    ▼
Admin-xxxx.js 로드
```

하는 것이 가능해집니다.

여기서 중요한 것은:

> **Code Splitting 자체를 수행하는 주체는 React가 아니라 Vite, Webpack, Rollup 같은 번들러입니다.**

그리고 번들러에게:

> "이 모듈은 별도로 로드할 가능성이 있어."

라는 정보를 제공하는 대표적인 JavaScript 문법이 바로 **dynamic `import()`**입니다.

---

# 3. 일반 `import`와 dynamic `import()`의 차이

## 일반 import

```jsx
import AdminPage from "./AdminPage";
```

이것은 **정적 import(static import)**입니다.

모듈 관계가 처음부터 결정됩니다.

```text
App.jsx
   │
   └── import AdminPage
              │
              ▼
        AdminPage.jsx
```

---

## Dynamic Import

JavaScript에는 함수 형태의 `import()`도 있습니다.

```jsx
import("./AdminPage");
```

이것을 **dynamic import**라고 합니다.

중요한 차이가 있습니다.

```jsx
const result = import("./AdminPage");
```

`import()`는 모듈 객체를 즉시 반환하지 않고 **Promise를 반환합니다.**

개념적으로:

```text
import("./AdminPage")
        │
        ▼
     Promise
        │
        │ 모듈 로딩
        ▼
   Module Object
```

사용할 수도 있습니다.

```jsx
import("./AdminPage")
  .then(module => {
    console.log(module);
  });
```

따라서 번들러는 `import()`를 Code Splitting 지점으로 활용할 수 있습니다.

```text
source code

import("./AdminPage")
        │
        ▼
      Bundler
        │
        ▼
별도의 JavaScript Chunk
```

---

# 4. 그렇다면 `React.lazy()`는 무엇을 하는가?

여기서 React의 문제가 하나 남습니다.

`import()`는 Promise를 반환합니다.

```jsx
import("./AdminPage")
```

하지만 JSX에서는 결국 컴포넌트를 이렇게 사용하고 싶습니다.

```jsx
<AdminPage />
```

즉 다음 두 세계를 연결할 필요가 있습니다.

```text
JavaScript Module Loading
       import()
          │
          ▼
       Promise

          ?

React Rendering
          │
          ▼
   <AdminPage />
```

이 둘을 연결해주는 것이 `React.lazy()`입니다.

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);
```

구조를 분해하면:

```text
React.lazy(
    () => import("./AdminPage")
)
          │
          │
          └── 모듈을 비동기로 로드하는 함수
```

따라서 역할을 정확히 나누면:

```text
import()
   │
   └── 모듈을 비동기로 로드
          │
          ▼
       Promise


Bundler
   │
   └── import()를 보고 Code Splitting


React.lazy()
   │
   └── 비동기로 로드되는 모듈을
       React Component로 사용할 수 있게 연결


Suspense
   │
   └── 준비되지 않은 동안
       fallback UI 제공
```

이 역할 분리가 `React.lazy()`를 이해하는 핵심입니다.

---

# 5. `React.lazy()` 기본 문법

가장 기본적인 형태는 다음과 같습니다.

```jsx
import React, { Suspense } from "react";

const AdminPage = React.lazy(
  () => import("./AdminPage")
);

export default function App() {
  return (
    <div>
      <h1>My App</h1>

      <Suspense fallback={<p>Loading...</p>}>
        <AdminPage />
      </Suspense>
    </div>
  );
}
```

여기에는 두 개의 핵심 요소가 있습니다.

## ① `React.lazy()`

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);
```

React에게:

> "`AdminPage`는 필요할 때 비동기로 로드되는 컴포넌트다."

라는 정보를 제공합니다.

---

## ② `<Suspense>`

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <AdminPage />
</Suspense>
```

비동기 컴포넌트가 아직 준비되지 않았을 때 대신 보여줄 UI를 정의합니다.

```text
AdminPage 필요
      │
      ▼
아직 모듈이 준비되지 않음
      │
      ▼
<Suspense>
      │
      ▼
fallback 렌더링
      │
      ▼
Loading...
```

모듈 로딩이 완료되면:

```text
import() 완료
      │
      ▼
AdminPage 준비
      │
      ▼
React 재렌더링
      │
      ▼
fallback 제거
      │
      ▼
<AdminPage /> 렌더링
```

---

# 6. 실제로 어떤 순서로 동작하는가?

다음 코드가 있다고 하겠습니다.

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);
```

그리고:

```jsx
<Suspense fallback={<p>Loading...</p>}>
  <AdminPage />
</Suspense>
```

React가 `AdminPage`를 렌더링해야 하는 순간 전체 흐름은 개념적으로 다음과 같습니다.

```text
① React 렌더링 시작
        │
        ▼
② <AdminPage /> 만남
        │
        ▼
③ 아직 AdminPage 모듈이 없음
        │
        ▼
④ lazy의 load 함수 실행
        │
        ▼
⑤ import("./AdminPage")
        │
        ▼
⑥ Promise Pending
        │
        ├─────────────┐
        │             │
        ▼             ▼
  React Suspends   Browser/Module Loader
        │             │
        ▼             ▼
   Suspense        JS Chunk 로드
        │             │
        ▼             ▼
 fallback 표시    Promise resolve
        │             │
        └──────┬──────┘
               ▼
         React 다시 렌더링
               │
               ▼
         <AdminPage />
```

화면만 놓고 보면:

```text
처음

┌──────────────────────┐
│ Loading...           │
└──────────────────────┘
```

잠시 후:

```text
┌──────────────────────┐
│ Admin Page           │
│                      │
│ ...                  │
└──────────────────────┘
```

으로 바뀝니다.

---

# 7. `Suspense`는 왜 필요한가?

Lazy Component는 다른 일반 컴포넌트와 중요한 차이가 있습니다.

일반 컴포넌트는 렌더링할 때 이미 컴포넌트 코드가 존재합니다.

```text
<Component />
     │
     ▼
Component 함수 실행
     │
     ▼
JSX 반환
```

하지만 Lazy Component는:

```text
<AdminPage />
     │
     ▼
AdminPage 모듈이 있는가?
     │
  ┌──┴──┐
 YES    NO
  │      │
렌더링   로딩 필요
```

와 같은 상황이 발생합니다.

React는 해당 렌더링 작업을 **suspend(일시 중단)**할 수 있습니다.

그리고 가장 가까운 `Suspense` boundary가:

```jsx
<Suspense fallback={<p>Loading...</p>}>
```

fallback을 표시합니다.

따라서 다음과 같이 생각하면 좋습니다.

> **`React.lazy()`는 "아직 컴포넌트가 준비되지 않았다"는 상황을 React 렌더링에 연결하고, `Suspense`는 그동안 무엇을 보여줄지를 결정합니다.**

즉:

```text
React.lazy()
     │
     │ 컴포넌트 준비 상태
     ▼
 Suspense
     │
     │ 기다리는 동안
     ▼
 fallback
```

입니다.

---

# 8. `React.lazy()`와 `import()`의 관계

다음 코드를 다시 보겠습니다.

```jsx
const SomeComponent = React.lazy(
  () => import("./SomeComponent")
);
```

여기서:

```jsx
() => import("./SomeComponent")
```

는 React 컴포넌트가 아닙니다.

**모듈을 비동기로 로드하는 함수**입니다.

이 함수의 반환값은:

```text
Promise<Module>
```

입니다.

따라서:

```text
React.lazy()
    │
    ▼
load 함수
    │
    ▼
import()
    │
    ▼
Promise
    │
    ▼
Module Object
    │
    ▼
default export
    │
    ▼
React Component
```

라는 관계가 만들어집니다.

`React.lazy()` 자체가 파일을 잘라서 Chunk를 만드는 것이 아니라는 점도 중요합니다.

```text
React.lazy()
    │
    └── React Rendering 담당

import()
    │
    └── Dynamic Module Loading 표현

Vite / Webpack / Rollup
    │
    └── Code Splitting / Chunk 생성
```

---

# 9. 가장 좋은 사용 사례 — Route-Level Code Splitting

`React.lazy()`는 페이지 단위 라우팅과 결합했을 때 특히 이해하기 쉽습니다.

```jsx
import React, { Suspense } from "react";
import {
  BrowserRouter,
  Routes,
  Route
} from "react-router-dom";

const Home = React.lazy(
  () => import("./pages/Home")
);

const About = React.lazy(
  () => import("./pages/About")
);

const Admin = React.lazy(
  () => import("./pages/Admin")
);

export default function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<p>페이지 로딩 중...</p>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/admin" element={<Admin />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

이 구조는 다음과 같이 이해할 수 있습니다.

```text
현재 URL
    │
    ▼
React Router
    │
    ▼
Route Matching
    │
    ▼
어떤 Page Component가 필요한가?
    │
    ├── /       → Home
    ├── /about  → About
    └── /admin  → Admin
                       │
                       ▼
                  React.lazy()
                       │
                       ▼
                해당 모듈 필요
                       │
                       ▼
                    import()
                       │
                       ▼
                 Chunk 로드
```

예를 들어 사용자가 처음 `/`에 접속했다면:

```text
/
│
▼
Home 필요
```

그리고 나중에 `/admin`으로 이동하면:

```text
/admin
   │
   ▼
Admin 필요
   │
   ▼
Admin Chunk가 아직 없다면 로드
   │
   ▼
로딩 중
   │
   ▼
Suspense fallback
   │
   ▼
Admin 준비
   │
   ▼
<Admin />
```

가 됩니다.

이것을 **Route-Level Code Splitting**이라고 부릅니다.

---

# 10. 일반 `import`와 `React.lazy()` 비교

| 구분             | 일반 `import`                 | `React.lazy()` + `import()`    |
| -------------- | --------------------------- | ------------------------------ |
| 형태             | Static Import               | Dynamic Import 기반              |
| 예              | `import Page from "./Page"` | `lazy(() => import("./Page"))` |
| 모듈 로딩          | 정적 의존성으로 처리                 | 컴포넌트가 필요해질 때 로드 가능             |
| Code Splitting | 해당 import 자체는 분할 지점이 아님     | `import()`가 분할 지점이 될 수 있음      |
| 로딩 상태          | 별도 처리 필요 없음                 | `Suspense`와 함께 처리              |
| 주요 사용처         | 항상 필요한 코드                   | 페이지, 차트, 에디터 등 늦게 필요한 코드       |
| 초기 로딩 최적화      | 코드 규모에 따라 불리할 수 있음          | 초기 JS 양을 줄이는 데 도움              |

한눈에 보면:

```text
일반 import

App
 │
 ├── Home
 ├── About
 ├── Admin
 └── Chart
      ↑
 초기 의존성에 포함


React.lazy()

App
 │
 ├── Home
 │
 ├── About ───── 필요할 때
 │
 ├── Admin ───── 필요할 때
 │
 └── Chart ───── 필요할 때
```

---

# 11. `default export`와의 관계

기본 사용법은 모듈의 `default export`를 대상으로 합니다.

예를 들어:

```jsx
// AdminPage.jsx

export default function AdminPage() {
  return <h2>Admin</h2>;
}
```

그리고:

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);
```

처럼 사용할 수 있습니다.

하지만 다음처럼 named export만 있다면:

```jsx
export function AdminPage() {
  return <h2>Admin</h2>;
}
```

그대로는 기본 형태와 맞지 않습니다.

필요하다면 Promise 결과를 변환할 수 있습니다.

```jsx
const AdminPage = React.lazy(() =>
  import("./AdminPage").then(module => ({
    default: module.AdminPage
  }))
);
```

즉 React가 최종적으로 기대하는 결과 형태를:

```js
{
  default: Component
}
```

로 만들어주는 것입니다.

---

# 12. Suspense Boundary는 어디에 둘 것인가?

`Suspense`를 어디에 배치하는지도 중요합니다.

예를 들어 애플리케이션 전체를 감싸면:

```jsx
<Suspense fallback={<Loading />}>
  <Routes>
    ...
  </Routes>
</Suspense>
```

구조가 단순합니다.

하지만 하나의 Lazy Component 때문에 너무 넓은 UI 영역이 fallback으로 바뀔 수도 있습니다.

반대로:

```jsx
<Suspense>
  <Widget1 />
</Suspense>

<Suspense>
  <Widget2 />
</Suspense>

<Suspense>
  <Widget3 />
</Suspense>
```

처럼 지나치게 세분화하면 로딩 UI가 여러 곳에서 자주 나타날 수 있습니다.

따라서 일반적으로는 애플리케이션 구조에 맞춰 적절한 경계를 잡습니다.

예:

```text
Route Level

App
 │
 └── Suspense
       │
       └── Page


또는


Page
 │
 ├── Header
 │
 ├── Content
 │
 └── Suspense
       │
       └── HeavyChart
```

대표적인 Lazy Loading 대상은:

```text
Page
Chart
Editor
Modal
Admin
Statistics
Heavy Widget
```

등입니다.

---

# 13. 로딩 실패는 `ErrorBoundary`와 구분해야 한다

`Suspense`가 담당하는 것은 기본적으로:

```text
아직 준비되지 않음
```

입니다.

반면 Chunk 로딩 자체가 실패했다면:

```text
네트워크 오류
404
Chunk 로딩 실패
모듈 실행 오류
```

와 같은 **Error** 상황이 됩니다.

따라서 개념적으로 역할을 나누면:

```text
Lazy Component
      │
      ├── 아직 로딩 중
      │       │
      │       ▼
      │   Suspense
      │       │
      │       ▼
      │    fallback
      │
      └── 로딩/렌더링 실패
              │
              ▼
        ErrorBoundary
              │
              ▼
           Error UI
```

입니다.

예:

```jsx
<ErrorBoundary>
  <Suspense fallback={<p>로딩 중...</p>}>
    <AdminPage />
  </Suspense>
</ErrorBoundary>
```

이렇게 하면 책임이 명확하게 분리됩니다.

```text
Suspense
→ 기다리는 상태 처리

ErrorBoundary
→ 실패 상태 처리
```

---

# 14. Preloading으로 사용자 경험을 개선할 수도 있다

Lazy Loading은 장점이 있지만 사용자가 페이지를 클릭한 **후에야** 다운로드를 시작한다면 잠깐의 대기가 생길 수도 있습니다.

그래서 사용자가 곧 방문할 가능성이 높은 페이지를 조금 먼저 로드하는 패턴을 사용할 수도 있습니다.

예를 들어:

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);

function preloadAdmin() {
  import("./AdminPage");
}
```

그리고:

```jsx
<button
  onMouseEnter={preloadAdmin}
  onFocus={preloadAdmin}
  onClick={() => setOpen(true)}
>
  Admin 열기
</button>
```

사용자가 버튼 위에 마우스를 올렸을 때:

```text
Mouse Enter
     │
     ▼
preloadAdmin()
     │
     ▼
import("./AdminPage")
     │
     ▼
미리 모듈 로딩
```

실제 클릭 시점에는:

```text
Click
  │
  ▼
Admin 필요
  │
  ▼
이미 로딩되어 있을 가능성 ↑
```

이 되어 Lazy Loading으로 인한 대기 시간을 줄이는 데 도움이 될 수 있습니다.

다만 이것은 `React.lazy()` 자체의 기능이라기보다는 **dynamic import를 활용한 별도의 최적화 전략**으로 이해해야 합니다.

---

# 15. `React.lazy()` 전체 동작을 한 번에 보자

마지막으로 전체 관계를 하나로 연결해보겠습니다.

```jsx
const Admin = React.lazy(
  () => import("./pages/Admin")
);
```

사용자가:

```text
/admin
```

으로 이동했다고 하겠습니다.

```text
사용자
  │
  │ /admin 이동
  ▼

React Router
  │
  │ Route Matching
  ▼

<Admin /> 필요
  │
  ▼

React.lazy()
  │
  │ Admin 모듈 확인
  ▼

아직 준비되지 않음
  │
  ▼

() => import("./pages/Admin")
  │
  ▼

Promise
  │
  ├───────────────────────┐
  │                       │
  ▼                       ▼

React Suspends        Module Loading
  │                       │
  ▼                       ▼

<Suspense>            Admin Chunk
  │                       │
  ▼                       ▼

fallback 표시         Promise resolve
  │                       │
  └───────────┬───────────┘
              ▼

        React 재렌더링
              │
              ▼

           <Admin />
              │
              ▼

         실제 UI 표시
```

이 그림에서 각각의 역할을 구분하는 것이 가장 중요합니다.

```text
React Router
→ 어떤 페이지가 필요한지 결정

import()
→ 모듈을 비동기로 로드

Bundler
→ 코드를 Chunk로 분리

React.lazy()
→ 비동기 모듈 로딩을 React Component 렌더링과 연결

Suspense
→ 기다리는 동안 fallback 표시

ErrorBoundary
→ 실패한 경우 Error UI 처리
```

---

# 16. 최종 정리

`React.lazy()`를 단순히:

> "컴포넌트를 늦게 불러오는 함수"

라고만 설명하면 내부 구조를 이해하기 어렵습니다.

가장 중요한 흐름은 다음입니다.

```text
SPA 규모 증가
     │
     ▼
JavaScript 규모 증가
     │
     ▼
초기 로딩 비용 증가
     │
     ▼
Code Splitting
     │
     ▼
dynamic import()
     │
     ▼
React.lazy()
     │
     ▼
Lazy Component
     │
     ├── 로딩 중
     │      ↓
     │   Suspense
     │      ↓
     │   fallback
     │
     └── 로딩 완료
            ↓
       실제 Component
```

따라서 가장 정확하게 한 문장으로 정의하면:

> **`React.lazy()`는 `import()`로 비동기 로드되는 컴포넌트 모듈을 React의 렌더링 시스템과 연결하여, 해당 컴포넌트가 실제 필요할 때 로드할 수 있도록 해주는 API입니다.**

그리고 `Suspense`까지 함께 설명하면:

> **`React.lazy()`가 비동기 컴포넌트 로딩을 React 렌더링과 연결하고, `<Suspense>`는 그 컴포넌트가 준비될 때까지 보여줄 fallback UI를 담당합니다.**

마지막으로 역할을 반드시 구분해서 기억해야 합니다.

```text
┌──────────────────────────────────────┐
│           Code Splitting             │
├──────────────────────────────────────┤
│                                      │
│ import()                             │
│ → 비동기 Module Loading              │
│                                      │
│ Vite / Webpack / Rollup              │
│ → Chunk 생성                         │
│                                      │
│ React.lazy()                         │
│ → Module Loading과 Component 연결    │
│                                      │
│ Suspense                             │
│ → Loading UI                         │
│                                      │
│ ErrorBoundary                        │
│ → Error UI                           │
│                                      │
└──────────────────────────────────────┘
```

이 구조가 잡히면 `React.lazy()`는 단순한 성능 최적화 API가 아니라,

**JavaScript의 Dynamic Import와 React의 렌더링 시스템을 연결하는 도구**

라는 점이 명확해집니다.
