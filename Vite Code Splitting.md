# Vite의 Code Splitting — Dev Server부터 Production Build까지

앞에서 Code Splitting을 다음과 같이 정의했습니다.

> **Code Splitting은 애플리케이션의 코드를 여러 Chunk로 나누어, 현재 필요하지 않은 JavaScript의 로딩을 나중으로 미루는 전략입니다.**

그렇다면 Vite에서는 이 Code Splitting이 실제로 어떻게 구현될까요?

Vite를 이해할 때 가장 먼저 알아야 하는 것은:

> **Vite의 개발 환경(dev)과 프로덕션 빌드(build)은 동작 방식이 다르다.**

는 점입니다.

전체 구조부터 보면:

```text
              Vite

       ┌────────┴────────┐
       │                 │
       ▼                 ▼

 npm run dev          vite build
 Development          Production

       │                 │
       ▼                 ▼
Native ESM         Production Bundling
Dev Server               │
                         ▼
                      Rolldown
                         │
                         ▼
                    Code Splitting
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           main.js   admin.js   chart.js
```

---

# 1. Vite Dev와 Build를 먼저 구분하자

## 1.1 Development Mode

다음 명령을 실행하면:

```bash
npm run dev
```

Vite Development Server가 실행됩니다.

개발 모드에서는 브라우저의 **Native ES Modules**를 적극적으로 활용합니다.

예를 들어:

```jsx
import App from "./App.jsx";
```

라는 코드가 있다면 브라우저는 필요한 Module을 요청합니다.

개념적으로:

```text
Browser
   │
   │ /src/main.jsx
   ▼
Vite Dev Server
   │
   ▼
main.jsx
   │
   │ import "./App.jsx"
   ▼
Browser
   │
   │ /src/App.jsx
   ▼
Vite Dev Server
   │
   ▼
App.jsx
```

즉 개발 단계에서는 처음부터 애플리케이션 전체를 거대한 `bundle.js` 하나로 만드는 방식이 아닙니다.

그래서:

```text
src/
├── main.jsx
├── App.jsx
├── Home.jsx
├── Admin.jsx
└── Chart.jsx
```

와 같은 Module 구조를 브라우저의 ESM 시스템과 Vite Dev Server가 연결합니다.

---

# 2. 그러면 Dev에서는 Bundler가 전혀 없는가?

입문 단계에서는:

> **"Vite dev는 전체 애플리케이션을 하나의 bundle로 묶지 않고 Native ESM을 사용한다."**

라고 이해하면 좋습니다.

하지만 이것을:

> **"Vite dev에서는 아무런 변환이나 최적화도 일어나지 않는다."**

라고 이해하면 안 됩니다.

Vite는 개발 과정에서도:

```text
JSX / TS 변환
Dependency 처리
Plugin Transform
HMR
Module Resolution
```

등의 작업을 수행합니다.

따라서 정확하게는:

> **Vite dev에서는 Production Build처럼 전체 Module Graph를 하나의 최종 Bundle 구조로 만드는 과정을 거치지 않고, 필요한 Module을 요청 기반으로 변환·제공합니다.**

라고 설명하는 것이 좋습니다.

---

# 3. Production Build에서는 이야기가 달라진다

다음 명령을 실행하면:

```bash
vite build
```

Vite는 배포용 파일을 생성합니다.

개념적으로:

```text
Source Modules
      │
      ▼
Module Graph 분석
      │
      ▼
Production Build
      │
      ▼
Rolldown
      │
      ├── Bundling
      ├── Tree Shaking
      ├── Minification 관련 처리
      └── Code Splitting
      │
      ▼
dist/
```

결과적으로:

```text
dist/
├── index.html
└── assets/
    ├── index-xxxxx.js
    ├── Admin-xxxxx.js
    ├── Chart-xxxxx.js
    └── index-xxxxx.css
```

같은 Production Asset이 만들어질 수 있습니다.

최신 Vite에서는 빌드 커스터마이징을:

```js
build.rolldownOptions
```

를 통해 수행합니다.

---

# 4. Vite Code Splitting의 핵심은 Dynamic Import다

다음 두 코드를 비교해보겠습니다.

## Static Import

```js
import { heavyFunction } from "./heavy-module.js";
```

이 Module은 정적 Module Graph에 포함됩니다.

```text
App.js
   │
   │ Static Import
   ▼
heavy-module.js
```

Bundler는:

```text
App 실행에 필요한 Module
```

로 분석할 수 있습니다.

---

## Dynamic Import

```js
const module = await import("./heavy-module.js");
```

이 경우 Module은 런타임에 필요해질 수 있습니다.

```text
App
 │
 │ runtime
 ▼
import("./heavy-module.js")
 │
 ▼
Heavy Module
```

이 Dynamic Import는 Bundler가 별도의 비동기 Chunk를 만들 수 있는 중요한 경계가 됩니다.

```text
Module Graph
     │
     ▼
 dynamic import()
     │
     ├───────────── Split Boundary
     │
     ▼
 Heavy Module
```

빌드 결과를 개념적으로 보면:

```text
main-xxxx.js

heavy-module-yyyy.js
```

와 같이 나뉠 수 있습니다.

---

# 5. 실제 Runtime에서는 어떻게 동작할까?

다음 코드가 있다고 하겠습니다.

```js
button.addEventListener("click", async () => {
  const module = await import("./heavy-module.js");

  module.heavyFunction();
});
```

초기 페이지 로딩:

```text
Browser
   │
   ▼
main.js
   │
   ▼
Application 실행


heavy-module
   │
   └── 아직 필요 없음
```

사용자가 버튼을 클릭하면:

```text
Click
  │
  ▼
import("./heavy-module.js")
  │
  ▼
해당 Module 필요
  │
  ▼
Browser Network Request
  │
  ▼
Heavy Chunk 다운로드
  │
  ▼
Module 평가
  │
  ▼
heavyFunction()
```

합니다.

즉 Vite에서 Code Splitting의 가장 중요한 흐름은:

```text
Dynamic Import
      │
      ▼
Build-time Split Point
      │
      ▼
별도의 Async Chunk
      │
      ▼
Runtime import()
      │
      ▼
Chunk Network Load
```

입니다.

---

# 6. React에서는 `React.lazy()`와 연결된다

React에서는 일반적으로:

```jsx
const Admin = React.lazy(
  () => import("./pages/Admin")
);
```

처럼 사용합니다.

여기서 역할을 정확하게 구분해야 합니다.

```text
import()
────────────────────────
Dynamic Module Loading


Vite / Rolldown
────────────────────────
Production Build에서
Chunk 생성


React.lazy()
────────────────────────
비동기 Module을
React Component와 연결


Suspense
────────────────────────
Module이 준비되지 않은 동안
fallback UI 제공
```

즉:

> **`React.lazy()`가 Chunk를 만드는 것이 아니라 `import()`가 Code Splitting Boundary를 만들고, Vite의 빌드 시스템이 실제 Chunk를 생성합니다.**

---

# 7. React Router와 결합하면 Route-Level Code Splitting

다음과 같은 애플리케이션을 생각해보겠습니다.

```text
/
├── Home
├── Dashboard
└── Admin
```

각 페이지를 lazy loading할 수 있습니다.

```jsx
import { lazy, Suspense } from "react";
import {
  BrowserRouter,
  Routes,
  Route
} from "react-router-dom";

const Home = lazy(
  () => import("./pages/Home")
);

const Dashboard = lazy(
  () => import("./pages/Dashboard")
);

const Admin = lazy(
  () => import("./pages/Admin")
);

export default function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<p>페이지 로딩 중...</p>}>

        <Routes>
          <Route path="/" element={<Home />} />

          <Route
            path="/dashboard"
            element={<Dashboard />}
          />

          <Route
            path="/admin"
            element={<Admin />}
          />
        </Routes>

      </Suspense>
    </BrowserRouter>
  );
}
```

Production Build에서는 개념적으로:

```text
Module Graph
      │
      ▼
Rolldown
      │
      ├── main chunk
      ├── Home chunk
      ├── Dashboard chunk
      └── Admin chunk
```

같은 구조를 만들 수 있습니다.

---

# 8. `/admin`으로 이동하면 어떤 일이 일어날까?

사용자가 처음 `/`에 접속했습니다.

```text
/
│
▼
Home
```

Admin Chunk는 아직 필요하지 않을 수 있습니다.

그리고 사용자가:

```text
/admin
```

으로 이동합니다.

전체 흐름:

```text
URL
/admin
  │
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
  ▼
() => import("./pages/Admin")
  │
  ▼
Dynamic Import 실행
  │
  ▼
Admin Chunk 필요
  │
  ▼
Network Request
  │
  ▼
Admin-xxxxx.js
  │
  ▼
Module 준비
  │
  ▼
React 재렌더링
  │
  ▼
Admin Page
```

이것이 Vite + React에서 가장 대표적인 Route-Level Code Splitting 패턴입니다.

---

# 9. Vite에서는 CSS도 Code Splitting한다

Code Splitting은 JavaScript만의 이야기가 아닙니다.

예를 들어:

```jsx
// Admin.jsx

import "./Admin.css";

export default function Admin() {
  return <div>Admin</div>;
}
```

그리고 Admin이 Dynamic Import 대상이라면:

```jsx
const Admin = lazy(
  () => import("./Admin")
);
```

Production Build 결과에서 관련 CSS도 Async Chunk와 연결하여 분리될 수 있습니다.

개념적으로:

```text
Admin.jsx
   │
   ├── Admin JS
   │
   └── Admin.css
```

빌드 결과:

```text
Admin-xxxx.js

Admin-yyyy.css
```

같은 형태가 될 수 있습니다.

Vite의 `build.cssCodeSplit` 기본값은 `true`이며, async JS Chunk에서 import한 CSS도 별도의 CSS Chunk로 보존되어 해당 JavaScript Chunk와 함께 로드됩니다.

---

# 10. CSS Chunk는 언제 로드되는가?

예를 들어:

```text
/admin 진입
```

시:

```text
Admin JS Chunk 필요
        │
        ▼
관련 CSS도 필요
        │
        ▼
Admin CSS Load
        │
        ▼
Admin JS 평가
```

가 이루어집니다.

Vite는 관련 CSS가 준비되기 전에 해당 Async Chunk의 JavaScript를 먼저 평가하여 스타일 없는 화면이 나타나는 문제를 줄이도록 처리합니다.

즉:

```text
Async JS Chunk
      │
      ├── JS
      │
      └── Related CSS
```

를 함께 관리합니다.

---

# 11. CSS Code Splitting을 끌 수도 있다

필요하다면:

```js
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    cssCodeSplit: false
  }
});
```

처럼 설정할 수 있습니다.

그러면 프로젝트 전체 CSS를 하나의 CSS 파일로 추출합니다.

개념적으로:

```text
cssCodeSplit: true

Home.css
Admin.css
Chart.css
   │
   ▼
관련 Chunk와 연결


cssCodeSplit: false

모든 CSS
   │
   ▼
style.css
```

입니다.

일반 SPA에서는 기본값인:

```js
cssCodeSplit: true
```

부터 사용하는 것이 자연스럽습니다.

---

# 12. Code Splitting의 문제 — Waterfall

Code Splitting에는 단점도 있습니다.

예를 들어 Chunk A가 Chunk C를 필요로 한다고 해보겠습니다.

단순하게 처리하면:

```text
A 요청
 │
 ▼
A Download
 │
 ▼
A Parse
 │
 ▼
C가 필요하다는 사실 발견
 │
 ▼
C 요청
```

과 같은 Network Waterfall이 발생할 수 있습니다.

```text
Request A
    ↓
Response A
    ↓
Request C
    ↓
Response C
```

요청이 직렬적으로 이어지기 때문에 느려질 수 있습니다.

---

# 13. Vite의 Async Chunk Loading 최적화

Vite는 Production Build에서 Dynamic Import에 필요한 의존 Chunk들을 분석해 로딩을 최적화합니다.

개념적으로:

```text
A Async Chunk
      │
      └── C Common Chunk 필요
```

라면:

```text
A를 받은 다음
C를 요청

        ❌
```

하는 대신 가능한 경우:

```text
      ┌── A
Entry ┤
      └── C

병렬 로딩
```

형태로 의존 Chunk의 로딩을 미리 시작할 수 있도록 처리합니다.

즉:

> **Vite는 Code Splitting으로 발생할 수 있는 추가 Network Round Trip을 줄이는 최적화도 함께 수행합니다.**

---

# 14. `modulepreload`

Production Build 결과의 HTML을 보면:

```html
<link
  rel="modulepreload"
  href="/assets/xxx.js"
/>
```

같은 형태가 등장할 수 있습니다.

`modulepreload`는 브라우저에게:

> **"이 ES Module이 곧 필요하니 미리 가져와 둬."**

라고 알려주는 힌트입니다.

Vite는 Production Build에서 Entry Chunk와 그 direct imports를 대상으로 `modulepreload` directive를 자동 생성합니다.

개념적으로:

```text
HTML Parsing
    │
    ├── main.js
    │
    └── modulepreload
           │
           ▼
       미리 Module 다운로드
```

합니다.

---

# 15. `import.meta.glob()` — Vite의 강력한 기능

Vite에서는 여러 Module을 패턴으로 가져올 수 있는:

```js
import.meta.glob()
```

을 제공합니다.

예:

```js
const modules = import.meta.glob(
  "./pages/**/*.jsx"
);
```

개념적으로 Vite는 이것을:

```js
{
  "./pages/Home.jsx":
    () => import("./pages/Home.jsx"),

  "./pages/About.jsx":
    () => import("./pages/About.jsx"),

  "./pages/Admin.jsx":
    () => import("./pages/Admin.jsx")
}
```

와 유사한 Module Loader Map으로 변환할 수 있습니다.

핵심은 각각:

```js
() => import(...)
```

형태라는 것입니다.

따라서:

```text
pages/
├── Home.jsx
├── About.jsx
└── Admin.jsx
```

각 Module을 필요할 때 Dynamic Import하는 구조를 쉽게 만들 수 있습니다.

---

# 16. `import.meta.glob()`과 Code Splitting

기본 lazy 형태에서는:

```js
const modules = import.meta.glob(
  "./pages/**/*.jsx"
);
```

각 Module Loader가 호출되는 시점에 해당 Module을 로드합니다.

```text
modules
│
├── Home
│     └── () => import(Home)
│
├── About
│     └── () => import(About)
│
└── Admin
      └── () => import(Admin)
```

따라서 파일 기반 라우팅이나 Plugin 개발 등에 유용합니다.

예:

```js
const pages = import.meta.glob(
  "./pages/**/*.jsx"
);

async function loadPage(path) {
  const loader = pages[
    `./pages/${path}.jsx`
  ];

  if (!loader) {
    throw new Error("페이지가 없습니다.");
  }

  const module = await loader();

  return module.default;
}
```

흐름:

```text
path = "Admin"
      │
      ▼
pages["./pages/Admin.jsx"]
      │
      ▼
Loader Function
      │
      ▼
Dynamic Import
      │
      ▼
Admin Module
```

입니다.

---

# 17. 자동 Code Splitting과 수동 Chunking을 구분하자

보통 애플리케이션에서는 먼저:

```text
dynamic import()
```

를 통한 **자동적인 Code Splitting**부터 사용합니다.

예:

```jsx
lazy(() => import("./pages/Admin"))
```

대부분의 경우 Route나 Feature Boundary만 잘 잡아도 충분합니다.

하지만 프로젝트가 커지면:

```text
공통 Library가 너무 큼

특정 Chunk가 지나치게 큼

Chunk 구조를 세밀하게 제어해야 함
```

같은 상황이 생길 수 있습니다.

이때 수동 Chunking 전략을 고려할 수 있습니다.

---

# 18. 최신 Vite에서 수동 Chunking

과거 Vite에서는 흔히:

```js
build.rollupOptions.output.manualChunks
```

를 사용했습니다.

하지만 최신 Vite에서는:

```js
build.rolldownOptions
```

가 기본 설정 경로이고, `build.rollupOptions`는 deprecated alias입니다.

또한 최신 Vite의 권장 Chunk 설정은 Rolldown의:

```text
output.codeSplitting
```

입니다.

개념적으로:

```js
export default defineConfig({
  build: {
    rolldownOptions: {
      output: {
        codeSplitting: {
          // custom chunking strategy
        }
      }
    }
  }
});
```

와 같은 방향입니다.

세부 설정 문법은 Rolldown 버전에 따라 달라질 수 있으므로 실제 적용 시 해당 버전의 Rolldown 문서를 기준으로 확인하는 것이 좋습니다.

---

# 19. 왜 `manualChunks` 설명을 먼저 가르치지 않는가?

Code Splitting을 처음 배울 때:

```text
React.lazy
dynamic import
manualChunks
vendor chunk
```

를 한꺼번에 배우면 핵심이 흐려집니다.

가장 중요한 순서는:

```text
1. Dynamic Import
       ↓

2. 자동 Code Splitting
       ↓

3. Route / Feature Boundary
       ↓

4. Build 결과 확인
       ↓

5. 정말 필요할 때
   Manual Chunking
```

입니다.

즉:

> **수동 Chunking은 Code Splitting의 출발점이 아니라 자동 분할 결과를 튜닝하는 고급 단계입니다.**

---

# 20. Vendor Chunk도 자동으로 만들 필요는 없다

예전에는 다음과 같은 구성이 흔했습니다.

```text
vendor.js
├── react
├── react-dom
├── lodash
└── axios
```

하지만:

> **`node_modules`에 있으면 무조건 vendor Chunk로 분리한다.**

는 규칙이 항상 최적은 아닙니다.

예를 들어:

```text
React
Chart.js
Monaco Editor
Three.js
```

는 크기와 사용 시점이 매우 다릅니다.

모두 하나의:

```text
vendor.js
```

로 묶으면 오히려 너무 큰 Chunk가 만들어질 수 있습니다.

따라서 최신 환경에서는:

```text
Route Boundary
Feature Boundary
Shared Dependencies
Caching
Chunk Size
```

를 함께 고려하는 것이 좋습니다.

---

# 21. Build 결과는 반드시 확인해야 한다

Code Splitting을 적용했다고 해서:

> **"아마 잘 나눠졌겠지."**

라고 끝내면 안 됩니다.

```bash
npm run build
```

후 생성된:

```text
dist/assets/
```

를 확인하는 것이 중요합니다.

예:

```text
dist/
└── assets/
    ├── index-e41a.js
    ├── Admin-f83a.js
    ├── Dashboard-a21c.js
    ├── Chart-b29d.js
    └── index-d19c.css
```

여기에서:

```text
Chunk 개수

Chunk 크기

어떤 기능이 어떤 Chunk에 포함되는지

공통 Module이 어디에 들어가는지
```

를 확인합니다.

---

# 22. 너무 큰 Chunk가 발견되면?

예를 들어:

```text
Admin Chunk
2 MB
```

가 나왔다고 하겠습니다.

바로 수동으로 쪼개기 전에:

```text
Admin이 왜 큰가?
```

부터 확인해야 합니다.

예:

```text
Admin
 │
 ├── DataGrid
 ├── Chart Library
 ├── Rich Editor
 └── PDF Viewer
```

처럼 무거운 Feature가 모두 Static Import되어 있을 수 있습니다.

그렇다면:

```text
Admin Page
   │
   ├── 기본 UI
   │
   ├── Chart       → lazy
   ├── Editor      → lazy
   └── PDF Viewer  → lazy
```

처럼 **기능 Boundary를 개선하는 것**이 우선일 수도 있습니다.

---

# 23. Vite의 `vite:preloadError`

Code Splitting에서는 Production 배포 후 특유의 문제가 발생할 수 있습니다.

사용자가 오래된 페이지를 열어둔 상태라고 하겠습니다.

브라우저에는:

```text
Old App
```

이 실행되고 있습니다.

그런데 서버에 새로운 버전을 배포했습니다.

```text
Old Chunk

Admin-AAAA.js
```

가 삭제되고 새로운:

```text
Admin-BBBB.js
```

가 배포되었습니다.

사용자가 기존 앱에서 `/admin`으로 이동하면:

```text
Old App
   │
   ▼
Admin-AAAA.js 요청
   │
   ▼
Server
   │
   ▼
404
```

가 발생할 수 있습니다.

Vite는 Dynamic Import 로드가 실패할 때:

```text
vite:preloadError
```

이벤트를 발생시킵니다.

---

# 24. `vite:preloadError` 활용

간단한 대응 예:

```js
window.addEventListener(
  "vite:preloadError",
  () => {
    window.location.reload();
  }
);
```

조금 더 명시적으로 처리하면:

```js
window.addEventListener(
  "vite:preloadError",
  event => {

    event.preventDefault();

    window.location.reload();
  }
);
```

개념적으로:

```text
Dynamic Import 실패
       │
       ▼
vite:preloadError
       │
       ▼
새로고침
       │
       ▼
최신 index.html
       │
       ▼
최신 Chunk URL 사용
```

입니다.

Vite 공식 문서에서도 새 배포 후 이전 Chunk URL을 참조하는 상황을 이 이벤트의 대표적인 사용 사례로 설명합니다.

---

# 25. Vite Code Splitting을 실제로 적용하는 순서

실무에서는 다음 순서가 좋습니다.

```text
① Route-Level Splitting

React.lazy()
+
import()
```

먼저 큰 페이지 단위로 나눕니다.

```text
        ↓
```

```text
② Production Build

npm run build
```

실제 Chunk를 확인합니다.

```text
        ↓
```

```text
③ Heavy Feature 확인

Chart
Editor
Map
PDF
```

필요하면 Feature-Level Splitting을 적용합니다.

```text
        ↓
```

```text
④ CSS / Async Chunk 확인
```

관련 CSS와 공통 Chunk가 어떻게 나뉘는지 확인합니다.

```text
        ↓
```

```text
⑤ UX 최적화

Suspense
Skeleton
Preloading
```

사용자가 Chunk를 기다리는 상황을 개선합니다.

```text
        ↓
```

```text
⑥ 필요하면 Custom Chunking
```

이 단계에서 `rolldownOptions`와 `codeSplitting`을 검토합니다.

---

# 26. 전체 구조를 한 번에 보자

React + Vite 프로젝트에서:

```jsx
const Admin = lazy(
  () => import("./pages/Admin")
);
```

가 있다고 하겠습니다.

Development:

```text
npm run dev

Browser
   │
   ▼
Vite Dev Server
   │
   ▼
Native ESM 기반 Module 요청
```

Production Build:

```text
vite build
    │
    ▼
Module Graph
    │
    ▼
Rolldown
    │
    ▼
Dynamic Import Boundary
    │
    ▼
Code Splitting
    │
    ├── main.js
    │
    └── Admin.js
```

Runtime:

```text
사용자 /admin 이동
       │
       ▼
React Router
       │
       ▼
<Admin /> 필요
       │
       ▼
React.lazy()
       │
       ▼
import("./Admin")
       │
       ▼
Admin Chunk 요청
       │
       ▼
관련 CSS / Dependency Load
       │
       ▼
Module 준비
       │
       ▼
React Rendering
```

이 세 단계를 구분하는 것이 Vite Code Splitting을 이해하는 핵심입니다.

---

# 27. Dev와 Build의 차이를 다시 정리하면

| 구분                     | Development                | Production Build |
| ---------------------- | -------------------------- | ---------------- |
| 대표 명령                  | `vite`, `npm run dev`      | `vite build`     |
| 기본 목적                  | 빠른 개발                      | 배포 최적화           |
| Module 제공              | Native ESM 중심              | Bundled Assets   |
| 전체 Production Bundling | 하지 않음                      | 수행               |
| Code Splitting 결과      | Production Chunk 개념과 구분 필요 | 실제 배포 Chunk 생성   |
| 핵심 엔진                  | Vite Dev Server            | Vite + Rolldown  |
| 결과                     | 개발용 Module 응답              | `dist/`          |

따라서:

> **Development에서 보이는 파일 요청 구조와 Production Chunk 구조는 같은 것으로 생각하면 안 됩니다.**

---

# 28. Vite가 Code Splitting에서 담당하는 것

Vite의 역할을 최종적으로 분리하면:

```text
Dynamic Import 분석
        │
        ▼
Production Chunk 생성
        │
        ▼
CSS Code Splitting
        │
        ▼
Module Preload 정보 생성
        │
        ▼
Async Dependency Loading 최적화
        │
        ▼
Dynamic Import Error 처리 지원
```

입니다.

즉 Vite는 단순히:

> **"`import()`가 있으면 파일 하나를 쪼개주는 도구"**

라고 설명하기에는 훨씬 많은 일을 합니다.

---

# 29. 마지막으로 다른 도구와 역할을 구분하자

```text
React Router
──────────────────────────
URL을 보고 어떤 페이지가
필요한지 결정


React.lazy()
──────────────────────────
Lazy Module Loading을
React Component와 연결


Suspense
──────────────────────────
준비되지 않은 Component의
fallback UI 처리


Dynamic import()
──────────────────────────
Module을 Runtime에
동적으로 로드


Vite
──────────────────────────
개발 서버와 Production
Build Pipeline 제공


Rolldown
──────────────────────────
Production Module Graph를 분석하여
Bundling / Chunking 수행


Browser
──────────────────────────
필요한 Chunk를
Network로 실제 다운로드
```

전체 흐름:

```text
URL 변경
   │
   ▼
React Router
   │
   ▼
Admin 필요
   │
   ▼
React.lazy()
   │
   ▼
Dynamic import()
   │
   ▼
Vite가 Build 때 만들어 둔
Admin Chunk
   │
   ▼
Browser Network Request
   │
   ▼
Module 준비
   │
   ▼
React Rendering
```

---

# 30. 최종 정리

Vite의 Code Splitting을 가장 간단히 설명하면:

> **Vite는 Production Build에서 Module Graph와 Dynamic Import 경계를 기반으로 애플리케이션을 여러 Chunk로 나누고, 런타임에는 필요한 Chunk가 브라우저에 의해 필요한 시점에 로드될 수 있도록 배포 구조를 만들어주는 빌드 도구입니다.**

조금 더 짧게 말하면:

> **Dynamic Import로 "어디서 나눌지" 표현하고, Vite의 Production Build가 실제 Chunk를 만들어준다.**

그리고 React까지 연결하면:

```text
React.lazy()
     │
     ▼
import()
     │
     ▼
Vite Build
     │
     ▼
Rolldown
     │
     ▼
Code Splitting
     │
     ▼
Async Chunk
     │
     ▼
Browser Load
     │
     ▼
Suspense 해제
     │
     ▼
Component Rendering
```

이 흐름을 이해하면 Vite의 Code Splitting은 단순히:

> **"dynamic import 쓰면 자동으로 파일이 쪼개진다."**

가 아니라,

> **"개발 환경에서는 Native ESM 중심으로 빠르게 Module을 제공하고, Production Build에서는 Module Graph를 분석해 JS·CSS Chunk를 만들며, preload와 Async Dependency Loading까지 함께 최적화하는 전체 Build Pipeline"**

으로 이해할 수 있습니다.
