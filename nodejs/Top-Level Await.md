

# ⏳ Top-Level Await란 무엇인가?  
**Top-level await(TLA)** 은 말 그대로 **모듈의 최상위 스코프(Top Level)** 에서 `await` 키워드를 사용할 수 있게 해주는 JavaScript의 새로운 기능입니다.

```js
// ✔️ 가능 (ESM에서만)
const data = await fetchData();
console.log(data);
```

즉, **async function 내부가 아닌 파일의 루트 레벨에서 await를 실행**할 수 있습니다.  

---

# 왜 이 기능이 중요한가? (핵심 정리)  

### ✅ 1. 모듈 초기화를 비동기로 처리할 수 있다  
기존에는 다음이 불가능했습니다:

```js
// ❌ 오류 (async 함수 없이 await 사용 불가)
const config = await loadConfig();
```

항상 이렇게 감싸야 했죠:

```js
(async () => {
  const config = await loadConfig();
})();
```

Top-level await 덕분에 파일의 초기화를 자연스럽게 처리할 수 있습니다.

---

### ✅ 2. ESM 모듈 로딩 과정에 직접적으로 영향을 준다  
ESM에서는 **각 모듈의 의존성이 로딩될 때까지 기다린 후 실행**합니다.

즉, 만약 A.js가:

```js
await slowInit();
```

을 가지고 있다면,

- A.js를 import하는 모든 모듈은  
- A.js의 await가 완료될 때까지  
- **자동으로 로딩이 지연(Pause)** 됩니다.

이것이 **"모듈 레벨 await의 진짜 파급력"**입니다.

---

### ✅ 3. Node.js, 브라우저 모두 지원 (단, ESM에 한정)

#### Node.js
- 반드시 ES Module이어야 함
- 즉, `.mjs` 또는 `package.json`에 `"type": "module"`

#### 브라우저
- `<script type="module">` 안에서는 정상 동작

---

# 📌 실제 동작 메커니즘 (모듈 그래프 관점)

JavaScript 모듈은 다음과 같은 방향성 그래프로 구성됩니다:

```
A.js -> B.js -> C.js
```

만약 C.js에서 top-level await이 존재한다면?

```js
// C.js
await new Promise(resolve => setTimeout(resolve, 3000));
console.log("C loaded");
```

결과:

1. C.js 로딩 중 pause (await가 해결될 때까지)
2. B.js는 C.js import를 기다림  
3. A.js는 B.js import를 기다림  

즉:

> 📌 **하위 모듈의 top-level await이 상위 모듈 전체를 블로킹한다.**

프런트엔드 번들러(Vite, Webpack, Rollup)는 이 지점을 매우 중요하게 고려합니다.

---

# 🧪 실전 예제 1: 원격 API에서 설정 가져오기

### config.js
```js
export const config = await fetch(
  "https://api.example.com/config"
).then(res => res.json());
```

### index.js
```js
import { config } from "./config.js";

console.log("Loaded config:", config);
```

👉 Top-level await 덕분에  
`config.js` 초기화가 완료되기 전에 index.js가 실행되지 않습니다.

---

# 🧪 실전 예제 2: 동적 DB 연결 초기화 (Node.js)

```js
// db.js
import { MongoClient } from "mongodb";

const client = new MongoClient(process.env.MONGO);
await client.connect();

export const db = client.db();
```

이제 어떤 파일에서든:

```js
import { db } from "./db.js";

const users = await db.collection("users").find().toArray();
```

💡 더 이상 즉시실행(async IIFE) 패턴이 필요 없습니다.  
→ 서버 초기화 코드가 훨씬 “현대적”이고 깔끔해짐.

---

# 🧩 장단점 총정리

## 장점
- async IIFE 없이 간단한 비동기 초기화 가능
- 모듈 로딩 전 데이터 준비 완료
- 서버 초기화·DB 준비·원격 설정 로딩에 매우 유용

## 단점 / 주의점
- **모듈 전체가 await에 의해 블로킹될 수 있음**
- import chain이 길어지면 초기 부팅 시간이 증가  
- 번들링 도구가 모듈 그래프 분석 시 추가 부담 발생

프런트엔드 번들링(Vite)에서는  
Top-level await을 사용하는 모듈이 있으면 코드 분리가 제한되거나  
동적 import가 지연될 수 있습니다.

---

# 🧠 그래서 요약하면?

| 질문 | 답변 |
|------|------|
| Top-level await이란? | 모듈의 최상위에서 await 를 바로 사용 |
| Node.js에서 가능한가? | ✔️ 가능 (ESM일 때만) |
| 이를 사용하면 좋은 점? | 초기화 코드가 비동기적으로 자연스러워짐 |
| 주의해야 할 점? | 해당 모듈을 import하는 상위 모듈 실행이 지연됨 |



🔸 **Top-level await이 모듈 로딩 그래프에 미치는 영향 도식 (SVG)**  
🔸 CommonJS에서 동일 패턴을 구현하는 방법  
🔸 Node.js 서버 초기화를 TLA 기반으로 재설계한 예시  
🔸 Vite/React 강의용 TLA 교육자료(PPT 스타일)

어떤 자료가 더 필요하신가요?
