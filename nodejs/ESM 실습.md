
# ✅ ESM 실습 예제

---

## 1️⃣ 프로젝트 개요

| 목표                      | 설명                                 |
| ----------------------- | ---------------------------------- |
| 기본 ESM 구조               | `import`, `export`, `.js` 확장자 필수   |
| default vs named export | 사용법 차이와 혼합 사용                      |
| top-level await         | fetch를 통해 외부 API 데이터를 모듈 레벨에서 가져오기 |
| import.meta.url         | 현재 모듈 경로 및 디렉터리 확인                 |

---

## 2️⃣ 폴더 구조

```
esm-demo/
├── package.json
├── math.js          ← default + named export
├── fetcher.js       ← top-level await 사용
├── app.js           ← 모든 기능 통합
```

---

## 3️⃣ package.json 설정

```bash
npm init -y
```

그리고 `package.json` 수정:

```json
{
  "name": "esm-demo",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node app.js"
  }
}
```

> `"type": "module"`은 `.js` 확장자 파일에서 ESM 문법을 사용할 수 있게 해줍니다.

---

## 4️⃣ math.js (default + named export)

```js
// math.js

// named exports
export function add(a, b) {
  return a + b;
}
export const PI = 3.14159;

// default export
export default function subtract(a, b) {
  return a - b;
}
```

---

## 5️⃣ fetcher.js (top-level await)

```js
// fetcher.js

const response = await fetch('https://jsonplaceholder.typicode.com/todos/1');
const json = await response.json();

export const todo = json;
```

> ✅ ESM이기 때문에 최상위에서 `await` 사용이 가능합니다 (CommonJS에서는 불가능).

---

## 6️⃣ app.js (전체 기능 통합)

```js
// app.js

// 1. named exports
import { add, PI } from './math.js';

// 2. default export
import subtract from './math.js';

// 3. top-level await로 불러온 값
import { todo } from './fetcher.js';

// 4. 현재 파일의 경로 출력
console.log('🧭 현재 모듈 경로:', import.meta.url);
console.log('📁 현재 디렉터리:', new URL('.', import.meta.url).pathname);

// 5. 수학 함수 테스트
console.log(`🧮 add(2, 3) = ${add(2, 3)}`);
console.log(`➖ subtract(5, 2) = ${subtract(5, 2)}`);
console.log(`📐 PI = ${PI}`);

// 6. fetcher.js에서 불러온 데이터 출력
console.log('📡 가져온 TODO 데이터:', todo);
```

---

## 7️⃣ 실행

```bash
npm start
```

### ✅ 예상 출력

```
🧭 현재 모듈 경로: file:///Users/yourname/esm-demo/app.js
📁 현재 디렉터리: /Users/yourname/esm-demo/
🧮 add(2, 3) = 5
➖ subtract(5, 2) = 3
📐 PI = 3.14159
📡 가져온 TODO 데이터: {
  userId: 1,
  id: 1,
  title: 'delectus aut autem',
  completed: false
}
```

---

## ✅ 정리 요약

| 기능                | 예제 설명                                          |
| ----------------- | ---------------------------------------------- |
| `named export`    | `add`, `PI`를 각각 `import { ... }`로 불러옴          |
| `default export`  | `subtract`를 기본값으로 `import subtract`로 불러옴       |
| `top-level await` | fetcher.js에서 API 호출을 최상위 await으로 처리            |
| `import.meta.url` | 현재 실행 중인 모듈의 경로를 확인 (브라우저에서 `import.meta`와 동일) |

---

## 💡 팁: 왜 이렇게 쓰는가?

* **default export**는 하나의 주요 기능이 있는 모듈에 적합
* **named export**는 다양한 유틸성 함수들을 동시에 제공할 때 유리
* **top-level await**은 ESM의 진정한 장점으로, 비동기 초기화를 간편하게 만듦
* **import.meta.url**은 파일 시스템과의 통합에 자주 사용됨 (예: `__dirname` 대체)


