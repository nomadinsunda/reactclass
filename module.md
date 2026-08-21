
# 🔍 자바스크립트 모듈 완벽 정리: ES6 모듈부터 CommonJS까지

자바스크립트가 등장한 초기에는 모듈 시스템이 존재하지 않았습니다. 전역 스코프에 모든 코드가 노출되었기 때문에 충돌이 자주 발생했고, 코드 유지보수가 매우 어려웠습니다. 시간이 흐르면서 다양한 **모듈 시스템** 이 등장했고, 그 정점으로 **ES6(ECMAScript 2015)** 에서 표준 모듈 시스템이 도입되었습니다.

이 글에서는 자바스크립트의 다양한 모듈 시스템을 **역사적인 배경**, **구문**, **용도**, 그리고 **환경별 차이점**을 중심으로 체계적으로 정리합니다.

---

## 📦 모듈이란?

**모듈(Module)** 이란 하나의 파일로 캡슐화된 코드 단위를 말합니다. 모듈은 다음과 같은 기능을 가집니다.

* 외부에 공개(export)할 기능을 명시
* 다른 모듈로부터 기능을 가져오기(import)
* 내부 스코프를 보호하여 전역 오염 방지
* 재사용성과 유지보수성 향상

---

## 🕰️ 자바스크립트 모듈의 역사

| 모듈 시스템     | 환경             | 특징                             |
| ---------- | -------------- | ------------------------------ |
| IIFE       | 브라우저           | 초창기 모듈 형태. 전역 변수 오염 방지         |
| CommonJS   | Node.js        | `require`, `module.exports` 사용 |
| AMD        | 브라우저           | 비동기 로딩 지원                      |
| UMD        | 브라우저 + Node.js | 범용 모듈 시스템                      |
| ES Modules | 브라우저 + Node.js | 공식 표준 모듈 시스템 (import/export)   |

---

## 1. 🔄 IIFE (즉시 실행 함수 표현식)

```javascript
const myModule = (function () {
  const privateVar = '비공개 변수';

  function publicFunc() {
    console.log('공개 함수입니다');
  }

  return {
    publicFunc
  };
})();
```

* **장점**: 전역 네임스페이스 오염 방지
* **단점**: 모듈 간 의존성 관리가 어려움

---

## 2. 📦 CommonJS (Node.js 전용)

### 📍 사용 예시

```javascript
// math.js
function add(a, b) {
  return a + b;
}
module.exports = { add };

// app.js
const math = require('./math');
console.log(math.add(2, 3)); // 5
```

### 📌 특징

* **정적 로딩**: 모듈을 동기적으로 불러옴 (`require`)
* **서버 환경(Node.js)** 에 최적화
* 파일 단위로 모듈화

---

## 3. ⚡ AMD (Asynchronous Module Definition)

### 📍 사용 예시

```javascript
define(['math'], function (math) {
  console.log(math.add(2, 3));
});
```

* 비동기적으로 모듈 로딩
* 브라우저 환경에서 모듈화 지원
* **RequireJS**로 대표되는 방식

### 📌 특징

* **비동기** 로딩으로 페이지 초기화 속도 향상
* 코드가 복잡해지고, 디버깅 어려움

---

## 4. 🌐 UMD (Universal Module Definition)

```javascript
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['dep'], factory); // AMD
  } else if (typeof exports === 'object') {
    module.exports = factory(require('dep')); // CommonJS
  } else {
    root.myModule = factory(root.dep); // 브라우저 전역
  }
}(this, function (dep) {
  return {}; // 모듈 내용
}));
```

* **범용 호환성**이 목적
* AMD, CommonJS, 브라우저 환경 모두에서 동작

---

## 5. 🚀 ES6 Modules (ECMAScript Modules, ESM)

### 📍 기본 사용 예시

```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// app.js
import { add } from './math.js';
console.log(add(2, 3)); // 5
```

### 📌 특징

| 항목                | 설명                     |
| ----------------- | ---------------------- |
| 선언적 문법            | `import` / `export` 사용 |
| 정적 분석 가능          | 번들러가 의존성을 미리 분석        |
| 파일 스코프            | 전역 오염 없음               |
| 지연 로딩             | `import()`로 동적 로딩 가능   |
| 브라우저 & Node.js 지원 | 현대적 JS 런타임에 모두 내장      |

### 🧠 Named vs Default Export

```javascript
// default export
export default function () {
  console.log('기본 함수');
}

// named export
export const name = '홍길동';
```

```javascript
// import
import myFunc, { name } from './module.js';
```

---

## 🆚 ESM vs CommonJS 차이점

| 항목              | CommonJS         | ESM            |
| --------------- | ---------------- | -------------- |
| 로딩 방식           | 동기 (`require`)   | 비동기 (`import`) |
| 환경              | Node.js (기본)     | 브라우저, Node.js  |
| 내보내기            | `module.exports` | `export`       |
| 가져오기            | `require()`      | `import`       |
| Top-level await | ❌ 지원 안 함 (부분적)   | ✅ 지원됨          |

**Node.js**에서는 `.mjs` 확장자나 `"type": "module"`이 `package.json`에 설정되어 있어야 ESM이 사용됩니다.

---

## 🌐 브라우저에서 ESM 사용하기

```html
<script type="module">
  import { hello } from './hello.js';
  hello();
</script>
```

* **주의사항**

  * 모듈은 기본적으로 **strict mode**로 실행
  * 모듈 간 경로는 반드시 **확장자(.js)** 필요
  * 모듈은 **CORS 정책** 적용됨

---

## 📦 번들러와 모듈 시스템

모듈을 직접 브라우저에서 사용하는 데는 제약이 많기 때문에, 대부분의 실무 프로젝트에서는 **Webpack**, **Rollup**, **Vite**, **Parcel** 등의 **번들러**를 사용합니다.

### 번들러의 주요 역할

* 여러 모듈을 하나의 파일로 번들링
* 트리 쉐이킹(Tree Shaking)으로 사용되지 않는 코드 제거
* 폴리필 적용 및 트랜스파일링(Babel)

---

## ✅ 결론

| 시스템      | 언제 사용할까?                         |
| -------- | -------------------------------- |
| CommonJS | Node.js 환경에서 빠르게 시작할 때           |
| ESM      | 표준화된 최신 방식. 브라우저 & Node.js 모두 지원 |
| AMD      | 브라우저에서 비동기 모듈 로딩이 중요할 때          |
| UMD      | 범용 라이브러리를 만들 때                   |
| IIFE     | 단순한 스크립트를 모듈처럼 감쌀 때              |

---

## 🔗 참고자료

* [MDN Web Docs - JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
* [Node.js Module System](https://nodejs.org/api/modules.html)
* [ECMAScript Modules in Node.js](https://nodejs.org/docs/latest-v18.x/api/esm.html)

---

## ✍️ 마치며

오늘날 자바스크립트에서 모듈은 선택이 아닌 **필수**입니다. 프로젝트 규모가 커질수록 모듈 시스템은 코드의 **구조화**, **재사용성**, **협업 생산성**을 획기적으로 향상시킵니다.


