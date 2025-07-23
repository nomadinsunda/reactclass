# 📦 Babel이란?

## ✅ 정의

**Babel**은

> 최신 JavaScript(ES6+, TypeScript, JSX 등)를 구형 브라우저에서도 호환 가능한 JavaScript로 **트랜스파일(transpile)** 해주는 **자바스크립트 컴파일러**입니다.

즉, **새로운 문법을 예전 문법으로 바꿔주는 컴파일러**입니다.
예: `let`, `const`, `async/await`, `optional chaining`, JSX 등을 ES5 문법으로 변환.

---

## 🧠 Babel의 핵심 목적

| 목적              | 설명                        |
| --------------- | ------------------------- |
| **트랜스파일링**      | 최신 JS 문법 → 구 브라우저 호환 코드   |
| **플러그인 기반 구성**  | 필요 문법만 선택적 변환             |
| **React/TS 지원** | JSX, TypeScript도 변환 가능    |
| **폴리필 적용 가능**   | core-js 등으로 런타임 기능 추가도 가능 |

---

# 🔧 Babel은 어떻게 작동하는가?

## 🔄 Babel 처리 단계

```plaintext
[Source Code]
    ↓
[Parser]
  → AST(Abstract Syntax Tree) 생성
    ↓
[Transformer]
  → AST 변형 (ES6 → ES5)
    ↓
[Generator]
  → 변형된 AST → JS 코드 생성
```

이 구조는 **컴파일러의 표준 3단계 구조**이며,
Babel은 **플러그인으로 Transformer를 확장**합니다.

---

## 📁 Babel 설정 파일 예시 (`babel.config.js`)

```js
module.exports = {
  presets: [
    '@babel/preset-env',     // 최신 JS 문법 지원
    '@babel/preset-react'    // JSX 변환
  ],
  plugins: [
    '@babel/plugin-proposal-optional-chaining',
    '@babel/plugin-transform-runtime'
  ]
};
```

---

## 🔧 주요 구성 요소

| 항목                         | 설명                             |
| -------------------------- | ------------------------------ |
| `@babel/core`              | Babel 핵심 엔진                    |
| `@babel/cli`               | Babel 명령어 실행용 도구               |
| `@babel/preset-env`        | 최신 JS → target 환경에 맞게 변환       |
| `@babel/preset-react`      | JSX 변환                         |
| `@babel/preset-typescript` | TypeScript → JS 변환             |
| `@babel/plugin-*`          | 다양한 실험적 문법 또는 런타임 기능 변환용 플러그인들 |

---

## 🎯 Babel과 Webpack/Vite의 관계

| 도구      | Babel과의 관계                                                                               |
| ------- | ---------------------------------------------------------------------------------------- |
| Webpack | `babel-loader`를 통해 Babel 트랜스파일링                                                          |
| Vite    | 기본적으로 **Babel 미사용**, 대신 **esbuild** 사용 <br> 필요 시 `@vitejs/plugin-react` 내부에서 Babel 사용 가능 |
| Rollup  | `@rollup/plugin-babel`로 통합 가능                                                            |

---

# 🧪 예제: Babel CLI로 트랜스파일하기

```bash
npm install --save-dev @babel/core @babel/cli @babel/preset-env

npx babel src --out-dir dist --presets=@babel/preset-env
```

* `src/` 디렉토리 내의 최신 JS를
* `dist/`에 구문 변환된 JS로 출력

---

## 🔬 Babel이 변환하는 문법 예

| 최신 문법                    | Babel 변환 결과                         |
| ------------------------ | ----------------------------------- |
| `let`, `const`           | → `var`로 변환                         |
| `async/await`            | → Promise + Generator로 변환           |
| JSX                      | → `React.createElement(...)` 호출로 변환 |
| Optional Chaining (`?.`) | → 조건문과 논리 연산으로 변환                   |

---

# ❌ Babel이 하는 일과 안 하는 일

| Babel이 하는 일      | Babel이 안 하는 일    |
| ---------------- | ---------------- |
| 문법 변환            | 코드 압축 (`minify`) |
| JSX → JS 변환      | 번들링 (모듈 병합)      |
| TypeScript 타입 제거 | 정적 타입 검사         |
| 폴리필 주입 (옵션에 따라)  | 브라우저별 버그 패치      |

---

# 🔌 Babel과 Polyfill

Babel은 문법만 바꾸기 때문에,
예를 들어 `Promise`, `Array.prototype.includes` 같은 **런타임 기능**은 변환하지 못합니다.

> 👉 이를 해결하려면 `core-js`나 `@babel/polyfill`, `babel-runtime` 등의 **폴리필**을 함께 사용합니다.

---

# ✅ Babel vs esbuild vs TypeScript 비교

| 항목       | Babel         | esbuild           | TypeScript (tsc)       |
| -------- | ------------- | ----------------- | ---------------------- |
| 속도       | 느림            | **매우 빠름 (Go 기반)** | 중간                     |
| 타입 지원    | ❌ (구문만 처리)    | ❌                 | ✅ (정적 타입 검사)           |
| JSX 지원   | ✅             | ✅                 | ✅ (`tsconfig.json` 필요) |
| 플러그인 확장성 | **강력**        | 제한적               | 없음                     |
| 트리 셰이킹   | ❌ (자체 미지원)    | ✅                 | ❌ (빌드 시 미지원)           |
| 사용 방식    | Webpack 등과 결합 | Vite, 단독 CLI      | `tsc`, `vite` 등과 결합    |

---

# 📌 결론 요약

| 항목      | 설명                                    |
| ------- | ------------------------------------- |
| Babel이란 | 최신 JS/JSX/TS를 구형 브라우저용 JS로 변환하는 컴파일러  |
| 사용 목적   | 문법 호환성 확보, 리액트 및 타입스크립트 지원            |
| 도구 통합   | Webpack, Vite, Rollup 등과 통합 사용 가능     |
| 플러그인 기반 | 매우 유연한 트랜스파일 구성 가능                    |
| 대안 도구   | esbuild, swc, sucrase (더 빠르지만 제어는 덜함) |

