# 🌲 Node.js에서 **트리 쉐이킹(Tree Shaking)** 이란?

트리 쉐이킹은 **사용되지 않는(dead) 코드(모듈)를 번들에서 자동 제거하여 결과물을 더 작고 빠르게 만드는 최적화 기술**입니다.

단, **중요 포인트**가 있습니다:

> 💡 **Node.js 자체의 런타임이 “트리 쉐이킹” 기능을 제공하는 것은 아닙니다.**
> 트리 쉐이킹은 Webpack, Rollup, esbuild 같은 **번들러 혹은 빌드 시스템**에서 수행하는 최적화 과정입니다.

즉, Node.js 환경에서도 **번들링을 할 때** 트리 쉐이킹이 적용되는 것이지, Node.js가 자동으로 적용해주는 것은 아닙니다.

---

# 🧠 왜 ‘트리(Tree)’인가?

JavaScript 모듈을 하나의 그래프(트리)로 보면:

```
entry.js
 ├─ moduleA.js
 │    ├─ moduleC.js
 │    └─ moduleD.js
 └─ moduleB.js
```

만약 `moduleD.js`에 선언된 함수가 아무 곳에서도 사용되지 않는다면?

👉 **번들러가 이 노드를 "가지치기(shake off)" 하고 제거**

그 결과, 최종 번들에서 `moduleD`가 사라져 파일 크기가 줄어듭니다.
이 작업이 바로 트리 쉐이킹입니다.

---

# ⚙️ Node.js에서 트리쉐이킹이 제대로 동작하려면?

다음 조건들이 충족되어야 합니다.

---

## ✔️ 1. **ES Module(ESM) 방식이어야 함**

CommonJS(`require`) 방식에서는 트리쉐이킹이 거의 불가능합니다.

왜냐하면:

* CommonJS는 `require()` 호출 시점에 **동적 로딩** 가능
* 프로퍼티/함수를 객체 형태로 모아서 export하는 구조
* 사이드 이펙트(side effects) 여부를 정적 분석하기 매우 어려움

반면 ESM(`import/export`)의 경우:

* 정적 분석(Static Analysis) 가능
* 정확한 의존성 관계 파악 가능
* Dead Code를 확실하게 판별 가능
  → 그래서 트리 쉐이킹이 가능합니다.

---

## ✔️ 2. **번들러가 있어야 함**

Node.js 런타임은 파일을 번들링하지 않기 때문에,
**esbuild, tsup, Webpack, Rollup, Vite(내부는 Rollup)** 같은 도구가 필요합니다.

예: tsup

```json
{
  "tsup": {
    "entry": ["src/index.ts"],
    "format": ["esm"],
    "treeshake": true,
    "splitting": false
  }
}
```

---

## ✔️ 3. **사이드 이펙트가 없어야 함(side-effect free)**

예:

```js
// 👍 트리쉐이킹 가능
export function add(a, b) {
  return a + b;
}

// 👎 트리쉐이킹 어려움
console.log("run"); // 사이드 이펙트
```

또는 package.json에:

```json
{
  "sideEffects": false
}
```

이렇게 명시하면 트리쉐이킹을 더 강하게 적용할 수 있습니다.

---

# 📦 Node.js 프로젝트에서 트리쉐이킹이 중요한 이유

### 1️⃣ 서버 코드도 번들링하면 속도가 빨라집니다.

* ts-node 대신 번들링 → 시작 속도 개선
* 서버리스 환경(AWS Lambda 등)에서 cold-start 시간 단축

### 2️⃣ 사용하지 않는 유틸, 미들웨어, 라이브러리를 제거

→ 배포 파일 크기 감소 → 배포 속도 증가
→ Docker 이미지 경량화(사용자님 강의에 매우 유용)

### 3️⃣ 프런트엔드 빌드와 동일한 최적화 방식 적용

React + Vite 수업에서 다루시는 module tree-shake와 같은 원리

---

# 🧪 간단 예제로 보는 트리쉐이킹

### 🔹 utils.js

```js
export function used() {
  return "I am used!";
}

export function unused() {
  return "I am unused!";
}
```

### 🔹 index.js

```js
import { used } from "./utils.js";

console.log(used());
```

**Rollup / esbuild / Vite**로 빌드하면?

👉 `unused()`는 존재하지 않음
👉 번들 크기 감소
👉 호출되지 않는 코드 제거

---

# 🟢 요약

| 질문                  | 답변                                    |
| ------------------- | ------------------------------------- |
| Node.js에서 트리 쉐이킹이란? | 번들러가 사용하지 않는 코드를 제거하는 최적화 과정          |
| Node가 해주는가?         | ❌ Node.js 런타임이 지원하는 기능이 아님            |
| 언제 적용됨?             | esbuild, Rollup, Webpack 등을 통해 번들링할 때 |
| 조건                  | ES Module(정적 분석 가능), 사이드 이펙트 없을 것     |
| 왜 중요함?              | 성능 향상, 배포 최소화, 서버리스 최적화               |


