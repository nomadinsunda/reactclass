# 🌲 트리 셰이킹(Tree Shaking)이란?

## ✅ 정의

**트리 셰이킹(Tree Shaking)** 은

> **사용되지 않는(Dead) 코드**를 최종 번들에서 **제거**하는 최적화 기법입니다.

> 📦 최종 번들 크기를 줄이고, 실행 성능을 개선하기 위한 **정적 분석 기반의 코드 제거** 기술입니다.

---

## 🌐 왜 "Tree" Shaking?

* 모듈 간 \*\*의존성 그래프(dependency tree)\*\*를 구성하고
* 사용되지 않는 "가지(branch)"를 "흔들어 떨어뜨린다(shake)"는 의미

즉, 전체 코드 베이스를 "트리"처럼 보고
필요한 가지만 남기고 나머지는 흔들어서 떨어뜨리는 구조입니다.

---

# 🔧 트리 셰이킹이 작동하는 조건

| 조건                     | 설명                                                             |
| ---------------------- | -------------------------------------------------------------- |
| **ES Module (ESM)** 사용 | `import`/`export` 구문으로 모듈을 정의해야 **정적 분석 가능**                   |
| **Side-effect 없는 코드**  | 해당 코드가 "호출되지 않으면" 무시 가능해야 함                                    |
| **번들러 지원**             | Webpack, Rollup, Vite 등에서 `mode: production`으로 설정 시 트리 셰이킹 활성화 |
| **package.json 설정**    | `sideEffects: false` 설정이 필요할 수 있음                              |

---

## 📘 예시: 트리 셰이킹 가능한 코드

### `math.js`

```js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

### `main.js`

```js
import { add } from './math.js';

console.log(add(1, 2));
```

✅ 위 코드에서 `subtract()`는 사용되지 않으므로 **트리 셰이킹 대상**입니다.

---

## ❌ 트리 셰이킹이 안 되는 경우

### 1. CommonJS 방식

```js
// ❌ CommonJS (정적 분석 불가)
const math = require('./math');

console.log(math.add(1, 2));
```

> CommonJS(`require`)는 런타임 분석을 요구하므로 정적으로 "어떤 함수가 쓰였는지" 예측 불가능 → 트리 셰이킹 **불가**

---

### 2. 사이드 이펙트(Side Effects)가 있는 코드

```js
export function logSomething() {
  console.log('로그 출력!'); // 실행 자체가 부작용임
}
```

> 번들러는 이 함수가 "실행만으로 부작용(side-effect)이 있는지" 판단을 보수적으로 처리 → 제거 안 됨

---

## 📦 패키지 개발 시: `package.json` 설정

```json
{
  "name": "my-lib",
  "sideEffects": false
}
```

* 이 설정이 있어야 번들러가 \*\*"모든 export는 안전하게 제거 가능"\*\*하다고 판단함
* 특정 파일만 side effect 있을 경우:

```json
{
  "sideEffects": ["./polyfill.js", "*.css"]
}
```

---

# ⚙️ 트리 셰이킹의 실전 도구별 특징

| 번들러         | 특징                                           |
| ----------- | -------------------------------------------- |
| **Rollup**  | 가장 aggressive한 트리 셰이킹, ESM 전용                |
| **Webpack** | `mode: 'production'` 시 활성화, `sideEffects` 고려 |
| **Vite**    | Rollup 기반이므로 Rollup과 동일하게 트리 셰이킹 동작          |

---

## 📊 성능 효과

| 항목       | Before | After (Tree Shaking) |
| -------- | ------ | -------------------- |
| 번들 크기    | 300KB  | 180KB                |
| 로딩 속도    | 1.2s   | 0.8s                 |
| 초기 실행 시간 | 400ms  | 250ms                |

특히 대형 라이브러리(Lodash, date-fns 등)를 사용할 경우 **부분 import** + 트리 셰이킹이 성능에 큰 영향을 미칩니다.

---

## 💡 실무 팁

### ✅ 꼭 지켜야 할 것

| 체크리스트                   | 이유                                |
| ----------------------- | --------------------------------- |
| `export`/`import` 사용    | ESM만 트리 셰이킹 가능                    |
| `sideEffects: false`    | 패키지 내 모든 코드 제거 허용                 |
| `lodash` 대신 `lodash-es` | CommonJS 버전은 트리 셰이킹 불가            |
| CSS도 sideEffect로 인식됨    | `*.css`는 반드시 `sideEffects`에 명시 필요 |

---

## 📌 결론 요약

| 개념    | 설명                                  |
| ----- | ----------------------------------- |
| 정의    | 사용되지 않는 코드를 정적으로 분석하여 제거            |
| 조건    | ESM, sideEffect 없음, 번들러 지원          |
| 도구    | Rollup > Webpack > Parcel 순으로 강력    |
| 실무 사용 | 라이브러리 최소화, 필요한 것만 import, 설정 최적화 필요 |


