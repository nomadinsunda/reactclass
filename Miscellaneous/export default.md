# 🚀 export default란?

`export default`는 **ES Modules(ECMAScript Module)** 시스템에서 사용되는 문법으로,
하나의 모듈이 **“디폴트로 내보내는 값(Default Export)”** 을 지정하는 기능입니다.

즉,

> 👉 “이 파일에서 가장 중요한 것 하나를 대표로 내보낸다”
> 라는 의미를 가진 모듈의 **디폴트 Export** 입니다.

---

# 🌟 1. 왜 `export default`가 필요한가?

JS 모듈 시스템에서는 파일마다 여러 개 또는 하나의 값을 밖으로 내보낼 수 있습니다.

* `named export` → 여러 개를 이름 기반으로 내보냄
* `default export` → 딱 하나, “대표값”을 내보냄

`export default`는 주로 다음 상황에 사용됩니다:

### ✔ 파일의 주 기능이 단 하나인 경우

예: React 컴포넌트, 유틸 함수, 클래스 등

### ✔ 소비자가 원하는 이름으로 import하도록 하고 싶을 때

기본값은 **이름 변경(import 시 rename)** 이 자유롭습니다.

---

# 🧩 2. export default의 기본 예제

```js
// math.js
export default function add(a, b) {
  return a + b;
}
```

가져오는 쪽에서는 이렇게 사용합니다:

```js
import add from "./math.js";

console.log(add(3, 4)); // 7
```

여기서 중요한 포인트:

> 📌 import 시 이름이 “add”일 필요 없음
> 원하는 이름으로 바꿀 수 있음

```js
import sum from "./math.js"; // sum = default export
```

이게 named export와 가장 큰 차이입니다.

---

# 🎯 3. export default의 특징 (매우 중요)

### ① 모듈당 **오직 하나만** 존재

하나의 파일에서 default export는 하나만 가능합니다.

---

### ② import 시 이름을 마음대로 정할 수 있음

💡 이것이 React 컴포넌트 import 방식과 잘 맞아떨어집니다.

```js
import App from "./App";   // OK
import MyApp from "./App"; // OK
import Whatever from "./App"; // OK
```

---

### ③ 중괄호 `{}`를 사용하지 않음

default export는 중괄호 없이 가져옵니다.

```js
// default
import App from "./App";

// named
import { App } from "./App";
```

---

### ④ 객체, 함수, 클래스, 원시값 무엇이든 default로 export 가능

예:

```js
export default 123;
export default "hello";
export default () => {};
export default class Person {};
export default { a: 1 };
```

---

# 🔥 4. export default vs named export 비교

React를 강의하시는 사용자님께 중요한 비교표부터 드립니다:

| 항목        | export default        | named export              |
| --------- | --------------------- | ------------------------- |
| 개수 제한     | 파일당 1개                | 여러 개 가능                   |
| import 방식 | `import A from "..."` | `import { A } from "..."` |
| 이름 변경     | 자유롭게 가능               | 반드시 원래 이름 유지              |
| 사용 목적     | 파일 대표 기능              | 여러 기능을 노출                 |

---

### ✔ default 예

```js
export default function App() {}
```

이는 import 시:

```js
import App from "./App";
```

### ✔ named 예

```js
export function add() {}
export function subtract() {}
```

불러올 때:

```js
import { add, subtract } from "./math";
```

---

# 👑 5. React에서 export default가 거의 표준인 이유

React 컴포넌트 파일에서는 일반적으로 **하나의 컴포넌트만 제공**합니다.

예:

```jsx
function App() {
  return <div>Hello</div>;
}

export default App;
```

그러면 가져올 때 매우 자연스럽습니다:

```jsx
import App from "./App";
```

---

### 이유 1) 파일당 주요 컴포넌트는 하나 → default로 내보내기 적합

### 이유 2) import 시 이름을 자유롭게 바꿀 수 있어 협업 시 유연함

### 이유 3) ESLint, CRA, Vite 템플릿에서도 권장 패턴

---

# 🧠 6. export default의 내부 동작 이해하기

`export default`는 모듈이 싱글톤처럼 동작합니다.

예:

```js
// config.js
export default {
  host: "localhost",
  port: 8080
};
```

어디에서든 기본 export 값을 import하여 공유합니다.

즉:

```js
import config from "./config";
```

→ 같은 config 객체를 참조합니다 (복사본 아님)

---

# 🧪 7. default + named를 함께 사용하는 패턴

ESM에서는 한 파일에서 default export와 named export를 같이 쓸 수 있습니다.

예:

```js
export default function App() {}

export function Header() {}
export function Footer() {}
```

불러오는 쪽:

```jsx
import App, { Header, Footer } from "./App";
```

React 라이브러리도 이런 형태를 많이 사용합니다.

---

# ⚠️ 8. 흔히 하는 실수들

### ❌ 하나의 파일에서 default exports 두 번 선언

```js
export default function A() {}
export default function B() {} // 에러!
```

---

### ❌ named export를 default처럼 import

```js
// math.js
export function add() {}
```

이렇게 하면 에러 ↑

```js
import add from "./math"; // ❌
```

이렇게 해야 함:

```js
import { add } from "./math";
```

---

# 📌 9. 결론 요약 (강의 슬라이드용)

> ✔ `export default`는 파일에서 대표로 내보낼 값을 지정하는 문법
> ✔ 파일당 1개만 존재
> ✔ import 시 이름 변경 가능
> ✔ React 컴포넌트와 매우 잘 맞아떨어지는 패턴
> ✔ named export와 함께 사용 가능

