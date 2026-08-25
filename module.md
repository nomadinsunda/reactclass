# JavaScript 모듈 

## — 전역 스코프에서 ES Modules까지

JavaScript가 처음 등장했을 때는 지금과 같은 **언어 차원의 모듈 시스템(Module System)**이 존재하지 않았습니다.

여러 `.js` 파일을 사용할 수는 있었지만, 전통적인 `<script>`로 로드한 파일들은 기본적으로 **하나의 전역 환경을 공유**했습니다.

```html
<script src="utils.js"></script>
<script src="user.js"></script>
<script src="app.js"></script>
```

개념적으로 보면 다음과 같습니다.

```text
utils.js ─┐
          │
user.js  ─┼──→ Global Scope
          │
app.js   ─┘
```

따라서 앞에서 로드된 파일이 만든 전역 함수나 변수를 뒤의 파일에서 바로 사용할 수 있었습니다.

```javascript
// utils.js
function formatName(name) {
  return name.toUpperCase();
}
```

```javascript
// app.js
console.log(formatName("hong"));
```

작은 프로그램에서는 편리했지만 프로젝트가 커지면서 문제가 발생합니다.

* 전역 변수와 함수의 이름 충돌
* 어떤 파일이 어떤 코드에 의존하는지 파악하기 어려움
* `<script>` 태그의 로딩 순서에 의존
* 내부 구현을 숨기기 어려움
* 코드 재사용과 유지보수가 어려움

이 문제를 해결하기 위해 JavaScript 개발자들은 여러 가지 모듈 패턴과 모듈 시스템을 발전시켰습니다.

```text
초기 JavaScript
       ↓
전역 스코프를 공유하는 여러 <script>
       ↓
전역 오염 / 이름 충돌 / 의존성 문제
       ↓
IIFE / Namespace
       ↓
CommonJS / AMD
       ↓
UMD
       ↓
ES Modules (ES2015)
       ↓
현대 JavaScript 개발 환경
Node.js / npm / Vite / Rollup / Webpack ...
```

---

# 1. 모듈(Module)이란?

모듈은 프로그램을 **독립적인 코드 단위로 나누어 관리하기 위한 구조**입니다.

현대 JavaScript에서는 일반적으로 **하나의 파일이 하나의 모듈 역할**을 합니다.

예를 들어:

```text
src/
├── main.js
├── user.js
├── math.js
└── api.js
```

각 파일은 자신의 스코프를 가지며, 필요한 기능만 외부에 공개하고 다른 모듈에서 필요한 기능을 가져올 수 있습니다.

ES Modules에서는 이를 `export`와 `import`로 표현합니다.

```javascript
// math.js
export function add(a, b) {
  return a + b;
}
```

```javascript
// app.js
import { add } from "./math.js";

console.log(add(10, 20));
```

핵심은 다음과 같습니다.

```text
┌──────────── math.js ────────────┐
│                                 │
│  function internalFunc() {}     │
│                                 │
│  export function add() {} ──────┼────┐
│                                 │    │
└─────────────────────────────────┘    │
                                       │ import
                                       ▼
                              ┌────── app.js ──────┐
                              │                    │
                              │  add(10, 20)       │
                              │                    │
                              └────────────────────┘
```

즉, 모듈 시스템의 핵심은 단순히 **“파일을 여러 개로 나누는 것”**이 아닙니다.

> **각 파일에 독립적인 스코프를 제공하고, 외부에 공개할 것과 가져올 것을 명시적으로 관리하는 것**이 핵심입니다.

---

# 2. 왜 모듈이 필요했을까?

예전 방식에서는 다음과 같이 여러 파일이 같은 전역 공간을 공유할 수 있었습니다.

```javascript
// library-a.js
var utils = {
  format() {
    // ...
  }
};
```

```javascript
// library-b.js
var utils = {
  validate() {
    // ...
  }
};
```

두 파일 모두 전역에 `utils`를 만들기 때문에 충돌할 수 있습니다.

```text
library-a.js
     │
     ├── var utils
     │
     ▼
Global Scope
     ▲
     │
     ├── var utils
     │
library-b.js

      이름 충돌
```

또 다른 문제는 **암묵적 의존성**입니다.

```html
<script src="math.js"></script>
<script src="app.js"></script>
```

`app.js`가 `math.js`의 함수를 사용한다면 반드시 `math.js`가 먼저 실행되어야 합니다.

```text
math.js
   ↓
전역에 add 생성
   ↓
app.js
   ↓
add() 사용
```

HTML의 순서가 바뀌면 문제가 발생할 수 있습니다.

```html
<script src="app.js"></script>
<script src="math.js"></script>
```

즉, 코드만 보고서는 의존 관계가 명확하게 드러나지 않습니다.

모듈 시스템은 이를 다음처럼 바꿉니다.

```javascript
import { add } from "./math.js";
```

이 한 줄만 봐도:

```text
app.js ──depends on──→ math.js
```

라는 관계를 알 수 있습니다.

---

# 3. IIFE — 모듈 시스템 이전의 캡슐화 기법

ES Modules가 등장하기 전에는 **IIFE(Immediately Invoked Function Expression)**를 이용해 전역 오염을 줄이는 방식이 많이 사용되었습니다.

```javascript
const myModule = (function () {

  const privateVar = "비공개 변수";

  function privateFunc() {
    console.log(privateVar);
  }

  function publicFunc() {
    privateFunc();
  }

  return {
    publicFunc
  };

})();
```

함수 내부에 선언된 변수는 함수 스코프에 있기 때문에 외부에서 직접 접근할 수 없습니다.

```text
Global Scope

   myModule
      │
      ▼
┌───────────────────────────┐
│ IIFE Scope                │
│                           │
│ privateVar     ← private  │
│ privateFunc()  ← private  │
│                           │
│ publicFunc()   ← 공개     │
└───────────────────────────┘
```

### 장점

* 전역 오염 감소
* 내부 구현 은닉 가능
* 비교적 간단한 캡슐화

### 한계

IIFE 자체는 **표준적인 파일 간 의존성 관리 시스템이 아닙니다.**

파일이 많아질수록 어떤 모듈이 어떤 모듈을 사용하는지 관리하기 어려웠습니다.

---

# 4. CommonJS

CommonJS는 특히 **Node.js 생태계에서 널리 사용되어 온 모듈 시스템**입니다.

대표적인 문법은:

```text
require()
module.exports
exports
```

입니다.

## 내보내기

```javascript
// math.js

function add(a, b) {
  return a + b;
}

function sub(a, b) {
  return a - b;
}

module.exports = {
  add,
  sub
};
```

## 가져오기

```javascript
// app.js

const math = require("./math");

console.log(math.add(10, 20));
```

흐름은 다음과 같습니다.

```text
math.js
   │
   │ module.exports
   ▼
{ add, sub }
   │
   │ require("./math")
   ▼
app.js
```

CommonJS의 전통적인 `require()`는 **동기적으로 모듈을 로드하는 방식**입니다.

서버의 로컬 파일 시스템에서 모듈을 읽는 Node.js 환경에는 잘 맞았지만, 네트워크를 통해 파일을 받아야 하는 브라우저의 기본 모듈 시스템으로는 적합하지 않았습니다.

> CommonJS를 단순히 “Node.js 전용”이라고 정의하기보다는 **Node.js 생태계에서 널리 사용된 모듈 시스템**이라고 이해하는 것이 더 정확합니다.

---

# 5. AMD

AMD는 **Asynchronous Module Definition**의 약자입니다.

브라우저에서는 JavaScript 파일을 네트워크를 통해 가져와야 하기 때문에 비동기 모듈 로딩이 중요했습니다.

대표적인 구현이 **RequireJS**입니다.

```javascript
define(["math"], function (math) {

  console.log(math.add(2, 3));

});
```

개념적으로:

```text
Browser
   │
   ├── math.js ────┐
   │               │
   ├── user.js ────┼──→ 비동기 로딩
   │               │
   └── app.js ─────┘
```

### 장점

* 브라우저 환경을 고려한 비동기 모듈 로딩
* 의존성을 명시적으로 표현

### 단점

* 문법이 복잡해질 수 있음
* 오늘날에는 ESM이 표준으로 자리 잡아 신규 프로젝트에서 직접 사용할 일은 많지 않음

---

# 6. UMD

CommonJS와 AMD가 서로 다른 환경에서 사용되면서 라이브러리 제작자에게 문제가 생겼습니다.

```text
Node.js       → CommonJS
Browser       → AMD
일반 Browser → Global Variable
```

하나의 라이브러리를 여러 환경에서 사용할 수 있도록 만든 패턴이 **UMD(Universal Module Definition)**입니다.

```javascript
(function (root, factory) {

  if (typeof define === "function" && define.amd) {

    // AMD
    define(["dep"], factory);

  } else if (
    typeof module === "object" &&
    module.exports
  ) {

    // CommonJS
    module.exports = factory(require("dep"));

  } else {

    // Browser Global
    root.myModule = factory(root.dep);
  }

})(this, function (dep) {

  return {};

});
```

UMD는 새로운 언어 문법이라기보다 **여러 모듈 환경에서 동일한 라이브러리를 사용할 수 있도록 만든 호환 패턴**에 가깝습니다.

---

# 7. ES Modules(ESM)의 등장

2015년 ES2015(ES6)에서 JavaScript 언어 자체에 표준 모듈 시스템이 추가되었습니다.

바로 **ECMAScript Modules(ESM)**입니다.

핵심 문법은 두 가지입니다.

```text
export
import
```

## export

```javascript
// math.js

export function add(a, b) {
  return a + b;
}

export function sub(a, b) {
  return a - b;
}
```

## import

```javascript
// app.js

import { add, sub } from "./math.js";

console.log(add(10, 20));
```

이제 의존 관계가 코드에 명확하게 표현됩니다.

```text
             import
app.js ─────────────────→ math.js
                           │
                           ├── export add
                           └── export sub
```

---

# 8. ESM이 중요한 이유

ESM의 가장 중요한 특징 중 하나는 **모듈 구조를 정적으로 분석하기 좋은 선언적 문법**을 제공한다는 것입니다.

```javascript
import { add } from "./math.js";
```

일반적인 정적 `import` 선언은 모듈의 최상위 수준에 위치하며, 이를 통해 JavaScript 엔진이나 개발 도구가 실행 전에 모듈 간 의존 관계를 분석할 수 있습니다.

```text
main.js
  │
  ├── import App
  │        │
  │        ├── Header.js
  │        └── Main.js
  │
  └── import api
           │
           └── http.js
```

이러한 구조는 Vite, Rollup, Webpack 등의 도구가 의존성 그래프를 분석하고 최적화하는 데 유리합니다.

---

# 9. ESM의 모듈 스코프

전통적인 `<script>`와 ESM의 중요한 차이 중 하나입니다.

전통적인 스크립트:

```html
<script src="a.js"></script>
<script src="b.js"></script>
```

```text
a.js ──┐
       ├──→ Global Scope
b.js ──┘
```

ESM:

```html
<script type="module" src="./a.js"></script>
```

각 모듈은 자신의 **모듈 스코프(Module Scope)**를 가집니다.

```text
┌──── a.js Module Scope ────┐
│                           │
│ const value = 10          │
│                           │
└───────────────────────────┘

┌──── b.js Module Scope ────┐
│                           │
│ const value = 20          │
│                           │
└───────────────────────────┘
```

따라서 같은 이름의 변수가 존재해도 서로 직접 충돌하지 않습니다.

---

# 10. Named Export와 Default Export

ESM에는 대표적으로 두 가지 export 방식이 있습니다.

## Named Export

```javascript
// math.js

export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}
```

가져올 때 이름을 사용합니다.

```javascript
import { PI, add } from "./math.js";
```

별칭도 사용할 수 있습니다.

```javascript
import { add as sum } from "./math.js";
```

## Default Export

모듈은 하나의 `default export`를 가질 수 있습니다.

```javascript
// User.js

export default function User() {

  return "User";

}
```

가져올 때 `{ }`를 사용하지 않습니다.

```javascript
import User from "./User.js";
```

가져오는 쪽에서 원하는 로컬 이름을 사용할 수도 있습니다.

```javascript
import MyUser from "./User.js";
```

React에서는 컴포넌트를 default export하는 코드를 자주 볼 수 있습니다.

```jsx
export default function App() {

  return <h1>Hello React</h1>;

}
```

---

# 11. 정적 import와 동적 import()

ESM의 `import`에는 두 가지 형태를 구분해서 이해해야 합니다.

## 정적 import

```javascript
import { add } from "./math.js";
```

의존 관계를 정적으로 표현합니다.

## 동적 import()

```javascript
const module = await import("./math.js");

console.log(module.add(10, 20));
```

`import()`는 Promise를 반환하며 런타임에 필요한 모듈을 동적으로 로드할 때 사용할 수 있습니다.

```text
Static Import

app.js
  │
  ├────→ math.js
  └────→ user.js


Dynamic Import

사용자 동작
    │
    ▼
import("./admin.js")
    │
    ▼
필요한 시점에 로드
```

React의 코드 스플리팅과 Lazy Loading에서도 이러한 개념이 중요합니다.

---

# 12. 브라우저에서 ESM 사용하기

현대 브라우저는 ESM을 직접 지원합니다.

```html
<script type="module" src="./app.js"></script>
```

`app.js`:

```javascript
import { hello } from "./hello.js";

hello();
```

`hello.js`:

```javascript
export function hello() {

  console.log("Hello");

}
```

브라우저는 의존 관계를 따라 필요한 모듈들을 로드합니다.

```text
index.html
    │
    ▼
<script type="module" src="./app.js">
    │
    ▼
  app.js
    │
    │ import
    ▼
 hello.js
```

브라우저의 ESM에는 몇 가지 중요한 특징이 있습니다.

* 모듈 코드는 자동으로 strict mode로 실행
* 각 모듈은 독립적인 모듈 스코프를 가짐
* 정적 `import` / `export` 사용 가능
* `import()`를 통한 동적 import 가능
* 모듈 요청에는 CORS 규칙이 적용됨
* 브라우저에서 상대 경로로 직접 모듈을 가져올 때는 일반적으로 정확한 URL을 지정해야 함

예:

```javascript
import { add } from "./math.js";
```

여기서 중요한 것은 단순히 **“.js 확장자가 문법적으로 무조건 필요하다”**가 아니라, 브라우저가 해당 모듈을 가져올 수 있는 **올바른 URL을 제공해야 한다는 것**입니다.

---

# 13. Node.js의 CommonJS와 ESM

Node.js는 역사적으로 CommonJS를 중심으로 발전했습니다.

```javascript
const fs = require("fs");
```

하지만 현대 Node.js는 ESM도 공식 지원합니다.

```javascript
import fs from "node:fs";
```

Node.js에서 `.js` 파일을 ESM으로 해석하도록 하는 대표적인 방법은 `package.json`에 다음과 같이 설정하는 것입니다.

```json
{
  "type": "module"
}
```

또는 `.mjs` 확장자를 사용할 수도 있습니다.

```text
app.mjs
```

반대로 `.cjs`는 명시적으로 CommonJS 파일을 나타내는 데 사용할 수 있습니다.

```text
app.cjs
```

따라서 현대 Node.js는 단순히:

```text
Node.js = CommonJS
```

라고 이해하면 안 됩니다.

현재는:

```text
              Node.js
                 │
        ┌────────┴────────┐
        ▼                 ▼
    CommonJS             ESM
   require()          import/export
```

두 모듈 시스템을 모두 지원합니다.

---

# 14. CommonJS와 ESM 비교

| 항목              | CommonJS          | ES Modules    |
| --------------- | ----------------- | ------------- |
| 대표 문법           | `require()`       | `import`      |
| 내보내기            | `module.exports`  | `export`      |
| 주요 역사적 환경       | Node.js           | JavaScript 표준 |
| 모듈 구조 분석        | 상대적으로 동적          | 정적 분석에 유리     |
| 브라우저 기본 지원      | 직접 지원하지 않음        | 지원            |
| Node.js 지원      | 지원                | 지원            |
| 동적 로딩           | `require()`       | `import()`    |
| Top-level await | 일반적인 CJS에서는 사용 불가 | 지원            |

여기서 주의해야 할 것이 있습니다.

> **ESM의 `import`를 단순히 “비동기 로딩”이라고 정의하면 부정확합니다.**

정적 `import` 선언과 동적 `import()`는 구분해야 합니다.

```javascript
// 정적 import 선언
import { add } from "./math.js";
```

```javascript
// 동적 import 표현식
const module = await import("./math.js");
```

---

# 15. React 프로젝트와 ESM

현대 React 프로젝트에서는 ESM 문법을 자연스럽게 사용합니다.

```jsx
// App.jsx

import Header from "./Header.jsx";
import Main from "./Main.jsx";

function App() {

  return (
    <>
      <Header />
      <Main />
    </>
  );

}

export default App;
```

```jsx
// main.jsx

import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

개념적으로:

```text
main.jsx
   │
   ├──→ react
   │
   ├──→ react-dom/client
   │
   └──→ App.jsx
            │
            ├──→ Header.jsx
            │
            └──→ Main.jsx
```

이렇게 `import` 관계를 따라가면 전체 애플리케이션의 **모듈 의존성 그래프(Module Dependency Graph)**가 만들어집니다.

---

# 16. 그렇다면 Vite는 왜 필요한가?

여기서 중요한 질문이 생깁니다.

> 브라우저가 이미 ESM을 지원한다면 왜 Vite 같은 개발 도구가 필요할까요?

단순한 JavaScript 프로젝트라면 브라우저의 ESM만으로도 개발할 수 있습니다.

하지만 실제 React 프로젝트에서는 다음과 같은 개발 기능이 필요합니다.

```text
React Source Code
      │
      │ .jsx / .tsx / CSS / assets
      ▼
┌───────────────────────────┐
│           Vite            │
│                           │
│ 개발 서버                  │
│ 모듈 변환                  │
│ HMR                       │
│ 플러그인 시스템             │
│ 의존성 처리                 │
│ Production Build          │
│ 최적화                     │
└───────────────────────────┘
      │
      ▼
    Browser
```

Vite는 단순히 “여러 JS 파일을 하나로 합치는 프로그램”이라고 이해하면 부족합니다.

특히 개발 모드에서 Vite는 브라우저의 **native ESM을 적극 활용하는 개발 서버와 빌드 도구 체계**를 제공합니다.

```bash
npm run dev
```

를 실행하면 일반적으로:

```text
npm
 │
 ▼
Vite 실행
 │
 ▼
Development Server
 │
 ├── 파일 요청 처리
 ├── 필요한 코드 변환
 ├── 모듈 의존성 처리
 └── HMR
 │
 ▼
Browser
```

와 같은 개발 환경이 만들어집니다.

---

# 17. Vite와 Node.js의 관계

Vite로 React를 개발할 때 **Node.js가 필요한 이유**도 여기에서 연결됩니다.

React 애플리케이션의 최종 실행 장소는 브라우저입니다.

하지만 Vite 자체와 npm 등의 **개발 도구는 개발자의 컴퓨터에서 실행되어야 합니다.**

```text
                개발자의 컴퓨터

┌────────────────────────────────────────────┐
│                                            │
│            Node.js Environment             │
│                                            │
│   npm                                      │
│    │                                       │
│    ├── npm install                         │
│    │                                       │
│    └── npm run dev                         │
│             │                              │
│             ▼                              │
│           Vite                             │
│             │                              │
│       Development Server                   │
│             │                              │
└─────────────┼──────────────────────────────┘
              │ HTTP
              ▼
┌────────────────────────────────────────────┐
│                 Browser                    │
│                                            │
│       HTML / CSS / JavaScript / React      │
│                                            │
│              React App 실행                │
└────────────────────────────────────────────┘
```

따라서 역할을 구분해야 합니다.

```text
Node.js
  ↓
JavaScript 개발 도구를 실행할 수 있는 런타임

npm
  ↓
패키지 설치 및 script 실행

Vite
  ↓
React 개발 서버 / 변환 / HMR / 빌드

React
  ↓
UI를 만들기 위한 라이브러리

Browser
  ↓
최종 React 애플리케이션 실행
```

즉,

> **React 앱을 브라우저에서 실행하기 위해 Node.js가 필요한 것이 아니라, React 앱을 효율적으로 개발하고 빌드하기 위한 Vite·npm 등의 개발 도구를 실행하기 위해 Node.js가 필요한 것입니다.**

이 구분은 매우 중요합니다.

---

# 18. 번들러와 모듈 시스템은 다르다

또 하나 자주 혼동하는 개념이 있습니다.

**모듈 시스템(Module System)**과 **번들러(Bundler)**는 같은 것이 아닙니다.

### 모듈 시스템

코드를 모듈로 나누고 서로 연결하는 규칙입니다.

```javascript
export function add() {}
```

```javascript
import { add } from "./math.js";
```

즉:

```text
ESM
CommonJS
AMD
```

등이 여기에 해당합니다.

### 번들러

여러 모듈의 의존 관계를 분석하여 배포하기 좋은 형태로 처리하는 도구입니다.

```text
main.js
 ├── App.js
 ├── Header.js
 ├── Main.js
 └── api.js
       │
       ▼
     Bundler
       │
       ▼
배포용 JavaScript / CSS / Assets
```

대표적인 도구로 Rollup, Webpack 등이 있습니다.

Vite는 이보다 더 넓은 개념의 **프론트엔드 개발 도구(Build Tool)**로 보는 것이 좋습니다.

개발 시에는 개발 서버와 native ESM 기반의 빠른 개발 환경을 제공하고, 프로덕션 빌드에서는 코드를 최적화하여 배포 가능한 결과물을 생성합니다.

---

# 19. 전체 역사를 한눈에 정리

```text
1995
JavaScript 등장
    │
    ▼
브라우저 <script>
    │
    ▼
여러 JS 파일이 전역 환경 공유
    │
    ├── 전역 변수 충돌
    ├── 로딩 순서 문제
    ├── 암묵적 의존성
    └── 유지보수 문제
    │
    ▼
IIFE / Namespace Pattern
    │
    ▼
CommonJS ─────────── AMD
    │                 │
    │                 │
    └────── UMD ──────┘
              │
              ▼
        ES2015 (ES6)
              │
              ▼
       ES Modules
      import / export
              │
              ▼
       현대 JavaScript
              │
      ┌───────┼─────────┐
      ▼       ▼         ▼
   Browser  Node.js   Build Tools
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
           Webpack   Rollup    Vite
```

---

# 20. 최종 비교

| 방식       | 핵심 목적                | 대표 문법                       | 현재 관점                |
| -------- | -------------------- | --------------------------- | -------------------- |
| IIFE     | 전역 오염 감소             | `(function(){})()`          | 역사적 패턴               |
| CommonJS | 모듈화                  | `require`, `module.exports` | Node.js 생태계에서 여전히 사용 |
| AMD      | 브라우저 비동기 모듈 로딩       | `define()`                  | 주로 레거시               |
| UMD      | 여러 모듈 환경 호환          | Wrapper Pattern             | 주로 라이브러리 배포의 역사적 패턴  |
| ESM      | JavaScript 표준 모듈 시스템 | `import`, `export`          | 현대 JavaScript의 표준    |

---

# 핵심 정리

JavaScript 모듈의 역사를 이해할 때 가장 중요한 흐름은 다음입니다.

```text
파일 분리
   ≠
모듈화
```

예전에도 JavaScript 파일을 여러 개 만들 수 있었습니다.

하지만 전통적인 `<script>` 방식에서는 여러 파일이 하나의 전역 환경을 공유했기 때문에 **파일 자체가 독립적인 모듈을 의미하지 않았습니다.**

모듈 시스템이 발전하면서:

```text
전역 공유
   ↓
캡슐화
   ↓
의존성 관리
   ↓
명시적인 import / export
   ↓
정적 분석 가능한 모듈 그래프
```

라는 구조로 발전했습니다.

그리고 현대 JavaScript의 표준은 **ES Modules(ESM)**입니다.

```javascript
export function add(a, b) {
  return a + b;
}
```

```javascript
import { add } from "./math.js";
```

React와 Vite 역시 이러한 현대적인 JavaScript 모듈 생태계를 적극적으로 활용합니다.

> **JavaScript 모듈은 단순히 코드를 여러 파일로 나누는 기술이 아니라, 각 코드 단위의 스코프를 분리하고 외부에 공개할 기능과 의존 관계를 명시적으로 관리하기 위한 시스템입니다.**

이 개념을 이해하면 이후의 **npm → Node.js → Vite → React → 번들링 → Tree Shaking → Code Splitting**의 관계도 훨씬 자연스럽게 이해할 수 있습니다.
