# Code Splitting — JavaScript를 필요한 순간에 나눠 로드하는 전략

코드 스플리팅(Code Splitting)을 한 문장으로 설명하면:

> **JavaScript 코드를 하나의 거대한 파일로 모두 전달하지 않고, 여러 개의 작은 청크(chunk)로 나누어 필요한 시점에 필요한 코드만 로드하는 성능 최적화 전략입니다.**

핵심은 단순히 **"파일을 여러 개로 쪼갠다"**가 아닙니다.

더 중요한 것은:

> **초기 화면에 필요하지 않은 코드를 초기 다운로드 대상에서 제외하고, 실제 필요한 순간까지 로딩을 늦추는 것**

입니다.

전체 구조를 먼저 보면 다음과 같습니다.

```text
Application Source Code
          │
          ▼
       Bundler
          │
          │ Module Graph 분석
          ▼
    Code Splitting
          │
          ▼
┌─────────┼─────────────┐
│         │             │
▼         ▼             ▼
main.js  admin.js     chart.js
│
│ 초기 로드
▼
Browser

나중에 /admin 진입
          │
          ▼
      admin.js 요청
```

---

# 1. 왜 Code Splitting이 필요한가?

애플리케이션 규모가 작을 때는 JavaScript 전체를 한 번에 내려받아도 큰 문제가 없습니다.

예를 들어:

```text
Home
About
Login
```

정도의 작은 애플리케이션이라면 하나의 JavaScript 파일에 들어가도 부담이 크지 않습니다.

하지만 애플리케이션이 커지면:

```text
Home
About
Login
Dashboard
Admin
Statistics
Chart
Editor
Settings
Map
Reports
...
```

JavaScript 코드도 함께 증가합니다.

이 모든 코드가 하나의 큰 초기 번들에 포함되어 있다고 생각해보겠습니다.

```text
bundle.js

┌─────────────────────────┐
│ Home                    │
│ About                   │
│ Login                   │
│ Dashboard               │
│ Admin                   │
│ Statistics              │
│ Chart                   │
│ Editor                  │
│ Settings                │
│ Map                     │
└─────────────────────────┘
```

사용자가 접속한 페이지는:

```text
/
```

하나뿐인데도 브라우저가:

```text
Admin
Chart
Editor
Settings
```

코드까지 모두 다운로드하고 파싱·실행해야 할 수 있습니다.

문제는 단순한 **다운로드 크기**만이 아닙니다.

브라우저는 받은 JavaScript를:

```text
Download
   ↓
Parse
   ↓
Compile
   ↓
Execute
```

해야 합니다.

따라서 초기 JavaScript가 지나치게 크면:

```text
큰 JavaScript
     │
     ├── Network 비용 증가
     ├── Parsing 비용 증가
     ├── Compilation 비용 증가
     └── Execution 비용 증가
```

로 이어질 수 있습니다.

그래서 나온 전략이 **Code Splitting**입니다.

---

# 2. Code Splitting의 핵심 아이디어

기존 방식:

```text
Application
    │
    ▼
bundle.js

┌──────────────────────────────┐
│ Home                         │
│ About                        │
│ Dashboard                    │
│ Admin                        │
│ Chart                        │
│ Editor                       │
└──────────────────────────────┘
```

Code Splitting을 적용하면 개념적으로:

```text
Application
    │
    ▼
Bundler
    │
    ├── main.js
    ├── dashboard.js
    ├── admin.js
    ├── chart.js
    └── editor.js
```

처럼 여러 조각으로 나눌 수 있습니다.

처음 접속할 때는:

```text
Browser
   │
   ▼
main.js
```

만 먼저 받고,

사용자가 나중에 Admin 페이지로 이동하면:

```text
/admin
   │
   ▼
admin.js 필요
   │
   ▼
Network Request
   │
   ▼
admin.js 다운로드
   │
   ▼
실행
```

하는 방식입니다.

따라서 Code Splitting의 핵심은:

```text
모든 코드
   ↓
처음부터 다운로드

          ❌
```

가 아니라:

```text
현재 필요한 코드
       ↓
    즉시 로드


나중에 필요한 코드
       ↓
  필요한 순간 로드
```

입니다.

---

# 3. Bundle과 Chunk부터 정확히 구분하자

Code Splitting을 이해하려면 `Module`, `Bundle`, `Chunk`를 구분하는 것이 좋습니다.

## 3.1 Module

개발자가 작성하는 개별 파일들을 생각하면 됩니다.

```text
src/
├── main.jsx
├── App.jsx
├── Home.jsx
├── Admin.jsx
├── Chart.jsx
└── utils.js
```

각 파일은 다른 모듈을 import할 수 있습니다.

```jsx
import Admin from "./Admin";
```

이 관계가 모이면 **Module Graph**가 됩니다.

```text
main.jsx
   │
   ▼
App.jsx
 ├── Home.jsx
 ├── Admin.jsx
 └── Chart.jsx
```

---

# 4. Module Graph는 왜 중요한가?

Bundler는 단순히 파일 이름만 보고 코드를 묶는 것이 아닙니다.

`import` 관계를 따라가면서:

```text
누가 누구를 import하는가?
```

를 분석합니다.

예:

```text
main.jsx
   │
   ▼
App.jsx
   │
   ├─────────────┐
   ▼             ▼
Home.jsx      Admin.jsx
                  │
                  ▼
              Chart.jsx
```

이런 전체 의존 관계가 **Module Graph**입니다.

Code Splitting도 결국 이 Module Graph를 기준으로 일어납니다.

---

# 5. Bundle이란?

Bundling은 여러 모듈을 브라우저에서 효율적으로 사용할 수 있도록 묶는 과정입니다.

```text
수많은 Module

A.js
B.js
C.js
D.js
E.js
    │
    ▼
 Bundler
    │
    ▼
JavaScript Output
```

빌드 결과 예:

```text
dist/
└── assets/
    └── index-a82df1.js
```

이런 빌드 결과를 일반적으로 **bundle**이라고 부릅니다.

하지만 현대 번들러의 빌드 결과가 반드시 하나의 JavaScript 파일일 필요는 없습니다.

Code Splitting을 적용하면 여러 출력 파일이 만들어질 수 있습니다.

---

# 6. Chunk란?

Code Splitting을 적용하면 Module Graph의 일부가 별도의 출력 단위로 분리될 수 있습니다.

예:

```text
dist/
└── assets/
    ├── index-a82df1.js
    ├── AdminPage-c391aa.js
    ├── Chart-f21b8c.js
    └── Editor-a192bd.js
```

이처럼 번들러가 생성한 **분할된 코드 단위**를 일반적으로 `chunk`라고 부릅니다.

개념적으로:

```text
Module Graph
     │
     ▼
   Bundler
     │
     ▼

┌────────────┐
│ main chunk │
└────────────┘

┌─────────────┐
│ admin chunk │
└─────────────┘

┌─────────────┐
│ chart chunk │
└─────────────┘
```

라고 볼 수 있습니다.

따라서 입문 단계에서는:

> **Bundle은 빌드된 JavaScript 묶음이고, Chunk는 Code Splitting 과정에서 만들어지는 분할된 코드 조각이다.**

라고 이해하면 충분합니다.

---

# 7. 그런데 번들러는 어디를 기준으로 코드를 나눌까?

여기서 Code Splitting의 핵심 질문이 나옵니다.

> **Bundler는 Module Graph의 어디를 기준으로 별도의 Chunk를 만들까?**

대표적인 기준 중 하나가 **Dynamic Import**입니다.

---

# 8. Static Import부터 보자

일반적으로 우리가 사용하는 import는:

```js
import { heavyFunction } from "./heavy-module.js";
```

입니다.

이를 **Static Import**라고 합니다.

Bundler 입장에서 보면:

```text
App.js
  │
  │ static import
  ▼
heavy-module.js
```

입니다.

즉:

> **App을 실행하려면 heavy-module도 필요하다.**

라는 정적인 의존 관계입니다.

개념적으로:

```text
App
 │
 ├── Home
 ├── Header
 └── HeavyModule
```

이 모두 초기 Module Graph에 연결되어 있습니다.

---

# 9. Dynamic Import `import()`

JavaScript에는 함수처럼 호출하는 `import()`도 있습니다.

```js
import("./heavy-module.js");
```

이것을 **Dynamic Import**라고 합니다.

중요한 차이는:

```js
const modulePromise = import("./heavy-module.js");
```

처럼 **Promise를 반환한다는 것**입니다.

```text
import("./heavy-module.js")
          │
          ▼
        Promise
          │
          ▼
     Module Object
```

따라서:

```js
const module = await import("./heavy-module.js");
```

처럼 사용할 수 있습니다.

---

# 10. Dynamic Import가 Split Point가 된다

다음 코드를 보겠습니다.

```js
button.addEventListener("click", async () => {
  const module = await import("./heavy-module.js");

  module.heavyFunction();
});
```

여기서 `heavy-module.js`는 애플리케이션이 시작되는 순간 반드시 필요한 코드가 아닙니다.

필요한 순간은:

```text
사용자가 Button을 클릭했을 때
```

입니다.

Bundler는 이런 dynamic import 지점을 **코드를 분할할 수 있는 지점**으로 사용할 수 있습니다.

개념적으로:

```text
App
 │
 ├── Header
 ├── Home
 │
 └── click
      │
      ▼
   import()
      │
      ▼
HeavyModule
```

빌드 결과:

```text
main.js

heavy-module.js
```

와 같은 형태로 분리될 수 있습니다.

즉:

> **Dynamic Import는 현대 JavaScript Code Splitting에서 가장 중요한 Split Point 중 하나입니다.**

---

# 11. 순수 JavaScript로 Code Splitting을 확인해보자

React 없이 생각하면 오히려 원리가 더 명확합니다.

```html
<button id="chart-btn">차트 보기</button>

<div id="chart-container"></div>

<script type="module">
  const button = document.getElementById("chart-btn");

  button.addEventListener("click", async () => {
    const { renderChart } = await import("./chart.js");

    renderChart(
      document.getElementById("chart-container")
    );
  });
</script>
```

초기 상태:

```text
페이지 로드
   │
   ▼
main module 실행

chart.js
   │
   └── 아직 필요하지 않음
```

사용자가 버튼을 클릭하면:

```text
Click
  │
  ▼
import("./chart.js")
  │
  ▼
Module 필요
  │
  ▼
Network
  │
  ▼
chart 관련 Chunk 로드
  │
  ▼
Promise resolve
  │
  ▼
renderChart()
```

이 흐름이 Code Splitting의 가장 기본적인 동작 원리입니다.

---

# 12. Code Splitting과 Lazy Loading은 같은 말일까?

둘은 밀접한 관계가 있지만 완전히 같은 개념은 아닙니다.

## Code Splitting

```text
코드를 여러 Chunk로 나누는 것
```

에 초점을 둡니다.

## Lazy Loading

```text
그 코드나 리소스를
필요할 때까지 로딩하지 않는 것
```

에 초점을 둡니다.

즉:

```text
Code Splitting
      │
      │ 코드를 나눈다
      ▼
여러 Chunk
      │
      │ 필요한 순간까지 기다린다
      ▼
Lazy Loading
```

이라고 이해하면 좋습니다.

그래서 실무에서는 두 개가 함께 사용되는 경우가 많습니다.

---

# 13. React에서는 `React.lazy()`가 이 과정과 어떻게 연결되는가?

React에서 페이지를 Dynamic Import하고 싶다고 하겠습니다.

```jsx
const AdminPage = React.lazy(
  () => import("./AdminPage")
);
```

여기에서 역할을 정확히 구분해야 합니다.

```text
import()
   │
   └── Dynamic Module Loading


Bundler
   │
   └── import() 지점을 보고
       별도의 Chunk 생성 가능


React.lazy()
   │
   └── 비동기로 로드되는 Module을
       React Component Rendering과 연결


Suspense
   │
   └── Component가 준비되지 않은 동안
       fallback UI 제공
```

즉:

> **`React.lazy()` 자체가 코드를 물리적으로 분할하는 것은 아닙니다.**

Code Splitting의 중요한 단서는:

```jsx
import("./AdminPage")
```

입니다.

그리고 React는 그 비동기 Module Loading을:

```jsx
<AdminPage />
```

라는 컴포넌트 렌더링 방식으로 편하게 사용할 수 있도록 `lazy()`를 제공합니다.

---

# 14. React에서 실제 흐름

```jsx
import { lazy, Suspense } from "react";

const AdminPage = lazy(
  () => import("./AdminPage")
);

function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <AdminPage />
    </Suspense>
  );
}
```

전체 흐름:

```text
<AdminPage />
      │
      ▼
React.lazy()
      │
      ▼
Admin Module 필요
      │
      ▼
import("./AdminPage")
      │
      ├───────────────┐
      │               │
      ▼               ▼
Network Request    React Suspend
      │               │
      ▼               ▼
Admin Chunk        Suspense
      │               │
      ▼               ▼
Download          fallback
      │               │
      └───────┬───────┘
              ▼
        Module 준비 완료
              │
              ▼
        React 재렌더링
              │
              ▼
         <AdminPage />
```

---

# 15. React Router와 만나면 Route-Level Code Splitting이 된다

Code Splitting의 대표적인 실전 기준은 **Route**입니다.

예:

```text
/
 /about
 /dashboard
 /admin
 /settings
```

각 페이지를 Lazy Component로 만들 수 있습니다.

```jsx
const Home = lazy(() => import("./pages/Home"));
const Dashboard = lazy(() => import("./pages/Dashboard"));
const Admin = lazy(() => import("./pages/Admin"));
```

그리고:

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/admin" element={<Admin />} />
</Routes>
```

그러면 개념적으로:

```text
현재 URL
   │
   ▼
React Router
   │
   ▼
필요한 Page 결정
   │
   ├── /          → Home
   ├── /dashboard → Dashboard
   └── /admin     → Admin
                       │
                       ▼
                  해당 Chunk 필요
```

가 됩니다.

예를 들어 `/admin`을 방문하지 않는 사용자는 Admin 페이지 코드를 굳이 초기 단계에서 로드하지 않아도 됩니다.

---

# 16. Route-Level Splitting이 많이 사용되는 이유

Route는 Code Splitting Boundary로 사용하기 좋은 자연스러운 단위입니다.

예를 들어:

```text
/dashboard
```

에서:

```text
Dashboard에 필요한 코드
```

가 하나의 논리적인 집합을 형성합니다.

또:

```text
/admin
```

은 일반 사용자가 방문하지 않을 수도 있습니다.

따라서:

```text
Application

├── Home Chunk
├── Dashboard Chunk
├── Admin Chunk
└── Settings Chunk
```

처럼 페이지 단위로 나누는 것이 이해하기 쉽습니다.

사용자 입장에서도 페이지 이동 시 새로운 코드를 로딩하는 것은 비교적 자연스러운 UX입니다.

---

# 17. Feature-Level Code Splitting

항상 Route 단위로만 나눌 필요는 없습니다.

페이지 안에서도 매우 무거운 기능은 따로 분리할 수 있습니다.

대표적으로:

```text
Rich Text Editor
Chart
Map
3D Viewer
PDF Viewer
Video Editor
```

등입니다.

예:

```jsx
const Chart = lazy(
  () => import("./Chart")
);
```

그리고 실제 차트를 표시할 때만:

```jsx
{showChart && (
  <Suspense fallback={<ChartSkeleton />}>
    <Chart />
  </Suspense>
)}
```

렌더링할 수 있습니다.

흐름:

```text
Dashboard Page
      │
      ▼
사용자가 "차트 보기" 클릭
      │
      ▼
Chart Component 필요
      │
      ▼
Chart Chunk 로딩
```

입니다.

---

# 18. Role / Domain 기반 Splitting

특정 사용자만 사용하는 기능도 좋은 Splitting 대상입니다.

예:

```text
일반 사용자
   │
   ├── Home
   └── MyPage


관리자
   │
   ├── Admin Dashboard
   ├── Member Management
   └── Statistics
```

Admin 기능이 매우 크다면:

```text
일반 사용자 초기 코드

         +

Admin 영역 Chunk
```

로 나누는 전략을 사용할 수 있습니다.

장점은 분명합니다.

> **Admin 권한이 없는 사용자는 Admin 관련 JavaScript를 사용할 일이 거의 없기 때문입니다.**

---

# 19. Vendor Splitting은 조금 다른 종류의 전략이다

Code Splitting에서 자주 등장하는 것이 `vendor` Chunk입니다.

예:

```text
react
react-dom
lodash
axios
chart library
```

같은 외부 라이브러리를 애플리케이션 코드와 분리하는 전략입니다.

개념적으로:

```text
Application Code
      │
      └── app.js


Third-party Libraries
      │
      └── vendor.js
```

이런 구성이 가능한 이유는 외부 라이브러리와 애플리케이션 코드의 변경 주기가 다를 수 있기 때문입니다.

예:

```text
React          → 거의 변경되지 않음
App.jsx        → 자주 변경
Product.jsx    → 자주 변경
```

잘 설계하면 변경되지 않은 Chunk가 브라우저 캐시에 남아 있는 이점을 얻을 수 있습니다.

다만 실제 Chunk 분할 방식은 사용하는 번들러와 빌드 설정에 따라 달라집니다.

따라서:

> **Vendor Splitting은 반드시 해야 하는 고정 규칙이 아니라 캐싱과 Chunk 구조를 고려해 적용하는 빌드 전략입니다.**

---

# 20. Code Splitting의 가장 큰 장점

## 20.1 초기 JavaScript 감소

가장 직접적인 목적입니다.

기존:

```text
Initial Load

main.js
5MB
```

Code Splitting 후 개념적으로:

```text
Initial Load

main.js
500KB


Later

dashboard.js
admin.js
chart.js
...
```

처럼 만들 수 있습니다.

---

# 21. 사용하지 않는 기능은 다운로드하지 않을 수도 있다

예를 들어 사용자가:

```text
/
/products
/cart
```

만 사용하고:

```text
/admin
```

페이지에는 한 번도 접근하지 않는다고 하겠습니다.

적절히 Lazy Loading된 경우:

```text
Admin Chunk
    │
    └── 요청되지 않을 수도 있음
```

입니다.

즉:

> **사용자가 필요로 하지 않는 코드의 네트워크 비용을 줄일 가능성이 있습니다.**

---

# 22. 브라우저가 처리해야 하는 JavaScript도 줄일 수 있다

JavaScript 성능에서는 파일 다운로드 크기뿐만 아니라:

```text
Parsing
Compilation
Execution
```

비용도 중요합니다.

따라서 초기 JavaScript 자체를 줄이면:

```text
Network 비용
+
CPU 비용
```

모두 줄이는 데 도움이 될 수 있습니다.

---

# 23. Code Splitting에도 비용이 있다

코드를 잘게 나눈다고 무조건 빨라지는 것은 아닙니다.

극단적으로:

```text
Component A → a.js
Component B → b.js
Component C → c.js
Component D → d.js
...
```

처럼 너무 잘게 나누면 문제가 생길 수 있습니다.

```text
너무 많은 Chunk
      │
      ├── 네트워크 요청 증가
      ├── 로딩 관리 복잡
      ├── fallback 증가
      └── 사용자 경험 저하
```

따라서 핵심은:

> **많이 나누는 것이 아니라 적절한 경계에서 나누는 것입니다.**

---

# 24. React에서는 UX 문제도 함께 생각해야 한다

예를 들어 페이지에:

```text
Header
Sidebar
Chart
Table
Editor
Footer
```

가 있다고 하겠습니다.

모든 컴포넌트를 Lazy Loading하면:

```text
Header Loading...
Sidebar Loading...
Chart Loading...
Table Loading...
Editor Loading...
```

처럼 화면 곳곳에서 스피너가 발생할 수 있습니다.

이것은 좋은 UX라고 보기 어렵습니다.

보통은:

```text
Route 단위
+
정말 무거운 Feature
```

정도를 먼저 Splitting 대상으로 고려하는 것이 이해하기 쉽습니다.

---

# 25. 좋은 Splitting Boundary를 찾는 기준

대표적인 기준은 다음과 같습니다.

### Route

```text
/admin
/dashboard
/settings
```

### Heavy Feature

```text
Chart
Editor
Map
3D Viewer
```

### Rarely Used Feature

```text
Advanced Settings
Debug Panel
Admin Tool
```

### Role / Domain

```text
User Domain
Admin Domain
Analytics Domain
```

즉:

> **크고, 늦게 필요하고, 일부 사용자만 사용하는 코드일수록 좋은 Code Splitting 후보가 될 가능성이 높습니다.**

---

# 26. Lazy Loading의 단점 — 필요한 순간에 기다려야 한다

Code Splitting은 초기 로딩을 줄여주지만 새로운 문제가 생깁니다.

```text
사용자 Click
      │
      ▼
Chunk가 아직 없음
      │
      ▼
Download
      │
      ▼
잠시 대기
```

입니다.

즉 비용이 사라지는 것이 아니라:

```text
초기 비용
```

의 일부를:

```text
나중의 특정 시점
```

으로 이동시키는 전략이라고 볼 수도 있습니다.

따라서 Code Splitting은 항상 UX 설계와 함께 생각해야 합니다.

---

# 27. 그래서 Preload / Prefetch 전략이 등장한다

예를 들어 사용자가 Admin 버튼에 마우스를 올렸습니다.

이 행동은:

> **곧 Admin 페이지를 열 가능성이 있다.**

는 신호가 될 수 있습니다.

그러면 클릭하기 전에 미리 모듈 로딩을 시작할 수도 있습니다.

```js
function preloadAdmin() {
  import("./AdminPage");
}
```

예:

```jsx
<button
  onMouseEnter={preloadAdmin}
  onFocus={preloadAdmin}
  onClick={() => setOpen(true)}
>
  Admin 열기
</button>
```

흐름:

```text
Mouse Enter
     │
     ▼
Admin이 필요할 가능성 증가
     │
     ▼
import("./AdminPage")
     │
     ▼
미리 다운로드 시작
     │

잠시 후 Click
     │
     ▼
이미 Module이 준비되어 있을 가능성 증가
```

즉:

```text
Lazy Loading
→ 너무 일찍 받지 않는다


Preloading / Prefetching
→ 너무 늦게 받지도 않는다
```

라는 균형을 맞추는 전략입니다.

---

# 28. Code Splitting과 Bundling은 반대 개념이 아니다

처음 보면:

```text
Bundling
→ 합친다

Code Splitting
→ 나눈다
```

이므로 서로 반대처럼 보입니다.

하지만 실제로는 같은 빌드 과정 안에서 함께 사용될 수 있습니다.

예를 들어 수백 개의 모듈을:

```text
500 Modules
```

그대로 배포하는 것이 아니라:

```text
500 Modules
     │
     ▼
Bundler
     │
     ▼

main chunk
admin chunk
chart chunk
vendor chunk
```

처럼 **논리적인 여러 출력 단위로 묶는 것**입니다.

즉:

> **Bundling은 모듈을 배포 가능한 출력 단위로 묶는 과정이고, Code Splitting은 그 결과를 하나의 거대한 출력으로 만들지 않고 여러 Chunk로 나누는 전략입니다.**

---

# 29. Tree Shaking과도 완전히 다른 개념이다

Tree Shaking은:

```text
사용하지 않는 코드
       ↓
      제거
```

하는 최적화입니다.

Code Splitting은:

```text
사용할 수도 있는 코드
       ↓
필요한 시점별로 나눔
```

입니다.

예를 들어:

```js
export function a() {}
export function b() {}
export function c() {}
```

에서 애플리케이션이 `a()`만 사용한다면 Tree Shaking은:

```text
a → 유지
b → 제거 가능
c → 제거 가능
```

를 목표로 합니다.

반면 Code Splitting에서는:

```text
Admin Code
```

가 실제로 필요한 코드이지만 초기 페이지에서는 필요하지 않으므로:

```text
Admin Chunk
```

로 따로 둘 수 있습니다.

---

# 30. 세 개념을 함께 보면

```text
Source Modules
      │
      ▼
Tree Shaking
      │
      │ 사용하지 않는 코드 제거
      ▼
필요한 Code
      │
      ▼
Bundling
      │
      │ 모듈을 배포 가능한 단위로 구성
      ▼
Bundles / Chunks
      │
      ▼
Code Splitting
      │
      │ 여러 로딩 경계로 분리
      ▼
필요한 시점에 각 Chunk 로드
```

다만 실제 빌드 도구 내부에서는 이 최적화들이 반드시 이렇게 독립된 순차 단계로만 실행된다고 생각할 필요는 없습니다.

이 그림은 **각 개념의 목적을 구분하기 위한 개념도**입니다.

---

# 31. 전체 동작을 한 번에 이해하기

React + Router 애플리케이션을 생각해보겠습니다.

```jsx
const Admin = lazy(
  () => import("./pages/Admin")
);
```

사용자가 처음:

```text
/
```

에 접속합니다.

```text
Browser
   │
   ▼
Initial JavaScript
   │
   ▼
Home 화면
```

이때 Admin Chunk는 아직 필요하지 않을 수 있습니다.

그리고 사용자가:

```text
/admin
```

으로 이동합니다.

전체 흐름:

```text
사용자
  │
  │ /admin 이동
  ▼
React Router
  │
  │ Route Matching
  ▼
Admin Component 필요
  │
  ▼
React.lazy()
  │
  ▼
import("./pages/Admin")
  │
  ▼
Admin Module 필요
  │
  ▼
Browser
  │
  │ Network Request
  ▼
Admin Chunk
  │
  ▼
Download
  │
  ▼
Module 준비
  │
  ▼
React Rendering
  │
  ▼
Admin 화면
```

이것이 Route-Level Code Splitting의 대표적인 실행 흐름입니다.

---

# 32. Code Splitting을 한 문장으로 다시 정의한다면

입문자용으로는:

> **Code Splitting은 큰 JavaScript 코드를 여러 조각으로 나누고, 현재 필요한 코드만 먼저 로드하는 기술입니다.**

조금 더 정확하게 표현하면:

> **Code Splitting은 애플리케이션의 Module Graph를 여러 Chunk로 분리하여, 초기 실행에 필요하지 않은 코드를 별도의 로딩 경계로 만들고 실제 필요한 시점에 로드할 수 있도록 하는 번들링 전략입니다.**

SPA 관점에서는:

> **SPA의 초기 JavaScript 비용을 줄이기 위해 Route나 Feature 등의 경계를 기준으로 코드를 여러 Chunk로 나누고, 사용자의 행동에 따라 필요한 Chunk를 나중에 로드하는 성능 최적화 전략입니다.**

---

# 33. 마지막으로 역할을 반드시 구분하자

Code Splitting을 공부할 때 가장 많이 섞이는 개념들을 한 번에 정리하면 다음과 같습니다.

```text
Module
─────────────────────────
개발자가 작성하는 코드 단위


Module Graph
─────────────────────────
Module 사이의 import 의존 관계


Bundler
─────────────────────────
Module Graph를 분석하고
배포용 JavaScript를 생성


Bundle / Chunk
─────────────────────────
Bundler가 만들어내는
JavaScript 출력 단위


Dynamic import()
─────────────────────────
Module을 동적으로 로드하며
Code Splitting 경계가 될 수 있음


Code Splitting
─────────────────────────
코드를 여러 Chunk로 나누는 전략


Lazy Loading
─────────────────────────
필요한 순간까지
리소스 로딩을 미루는 전략


React.lazy()
─────────────────────────
Dynamic Module Loading을
React Component Rendering과 연결


Suspense
─────────────────────────
Lazy Component가 준비되지 않았을 때
fallback UI를 제공
```

그리고 전체를 하나의 흐름으로 연결하면:

```text
Source Modules
      │
      ▼
Module Graph
      │
      ▼
Bundler
      │
      │ import() 등의 경계 분석
      ▼
Code Splitting
      │
      ├── main chunk
      ├── admin chunk
      ├── chart chunk
      └── editor chunk
             │
             │ 아직 로드하지 않음
             ▼

사용자가 기능 요청
      │
      ▼
Dynamic import()
      │
      ▼
필요한 Chunk 다운로드
      │
      ▼
Module 실행
```

React에서는 그 위에:

```text
Dynamic import()
      │
      ▼
React.lazy()
      │
      ▼
Suspense
      │
      ▼
Lazy Component
```

라는 React의 렌더링 모델이 추가됩니다.

따라서 Code Splitting의 핵심은 단순히:

> **"파일을 쪼갠다."**

가 아니라,

> **"초기 화면에 필요하지 않은 JavaScript를 초기 로딩 경로에서 분리하고, 실제 필요한 시점까지 로딩을 늦춘다."**

는 데 있습니다.
