
Node.js에서 **"모듈(module)"** 은 아주 핵심적인 개념입니다.
**Node.js는 모든 코드를 "모듈" 단위로 관리하며, 애초에 Node 자체도 여러 내부 모듈로 구성되어 있습니다.**

---

# ✅ Node.js의 **모듈(module)** 이란?

> 모듈이란 **특정 기능을 캡슐화하여 코드 파일 단위로 분리**한 것
> 다른 파일에서 **불러와서 재사용**할 수 있는 코드 단위

---

## 📦 1. 왜 모듈이 필요한가?

| 전통적인 방식       | 모듈화 방식                          |
| ------------- | ------------------------------- |
| 하나의 파일에 모든 코드 | 파일 단위로 기능 분리                    |
| 전역 변수 충돌 위험   | 스코프가 격리됨 (파일마다 독립적 환경)          |
| 재사용 어려움       | 쉽게 `require()` 또는 `import`로 불러옴 |

모듈을 사용하면 다음과 같은 장점이 있습니다:

* 코드 **재사용성 증가**
* 코드 **유지보수성 향상**
* 전역 스코프 오염 방지
* 테스트 및 디버깅 용이

---

## ✅ 2. Node.js의 모듈 시스템 종류

| 종류                  | 설명                 | 문법                          |
| ------------------- | ------------------ | --------------------------- |
| **CommonJS (기본)**   | Node.js의 기본 모듈 시스템 | `require`, `module.exports` |
| **ES Module (ESM)** | 최신 자바스크립트 표준       | `import`, `export`          |

기본은 CommonJS지만, `package.json`에 `"type": "module"`을 지정하면 ESM 사용 가능

---

## ✅ 3. CommonJS 모듈 예제

### 📁 math.js

```js
// math.js
function add(a, b) {
  return a + b;
}
function multiply(a, b) {
  return a * b;
}

// 외부에서 사용할 함수만 내보내기
module.exports = {
  add,
  multiply
};
```

### 📁 app.js

```js
const math = require('./math'); // .js 생략 가능

console.log(math.add(3, 4));      // 7
console.log(math.multiply(3, 4)); // 12
```

---

## ✅ 4. ES Module 예제 (`type: "module"`일 때)

### 📁 math.js

```js
export function add(a, b) {
  return a + b;
}
export function multiply(a, b) {
  return a * b;
}
```

### 📁 app.js

```js
import { add, multiply } from './math.js';

console.log(add(2, 5));       // 7
console.log(multiply(2, 5));  // 10
```

> 📌 ESM을 사용할 경우 확장자 `.js`를 반드시 포함해야 합니다 (`.mjs`도 가능)

---

## ✅ 5. Node.js 내장 모듈

Node.js는 기본적으로 다음과 같은 **수많은 모듈을 내장**하고 있어 설치 없이 바로 사용 가능:

| 모듈       | 설명                       |
| -------- | ------------------------ |
| `fs`     | 파일 시스템 조작 (읽기, 쓰기, 삭제 등) |
| `path`   | 파일 경로 처리 도구              |
| `http`   | HTTP 서버 생성 도구            |
| `url`    | URL 파싱/구성 도구             |
| `os`     | 운영체제 정보 조회               |
| `crypto` | 암호화 유틸리티                 |

예시:

```js
const fs = require('fs');
const data = fs.readFileSync('README.md', 'utf8');
console.log(data);
```

---

## ✅ 6. 사용자 정의 모듈 vs 외부 모듈

| 구분           | 설명                                                  |
| ------------ | --------------------------------------------------- |
| 📁 사용자 정의 모듈 | 내가 만든 `.js` 파일. `require('./filename')`             |
| 📦 외부 패키지 모듈 | npm에서 설치 (`npm install axios`) 후 `require('axios')` |

---

## ✅ 7. 정리 요약

| 개념        | 설명                                          |
| --------- | ------------------------------------------- |
| 모듈        | 기능을 파일 단위로 분리한 재사용 가능한 코드 블록                |
| CommonJS  | 기본 모듈 시스템 (require, module.exports)         |
| ES Module | 최신 표준 (`import`, `export`)                  |
| 내장 모듈     | Node.js가 자체 제공 (`fs`, `path`, `http` 등)     |
| 외부 모듈     | npm을 통해 설치 (`axios`, `express`, `dotenv` 등) |

