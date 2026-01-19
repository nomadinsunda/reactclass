

# 🚀 ESLint : 자바스크립트·리액트 개발의 필수 품질 도구

프론트엔드(React, Vite, Next.js) 또는 Node.js 백엔드 개발을 하실 때, **코드 품질을 자동으로 검사하고 통일성 있게 유지**하려면 반드시 필요한 도구가 있습니다. 바로 **ESLint**입니다.

---

# 🔎 ESLint란?

**ESLint(ECMAScript Lint)** 는
👉 **자바스크립트/TypeScript 코드의 오류, 잠재적 버그, 스타일 문제를 자동으로 탐지해주는 정적 분석 도구**입니다.

즉:

* ❌ 문제 있는 코드를 자동으로 감지
* ⚠️ 잠재적 버그를 미리 알려줌
* 🎯 팀 전체의 코드 스타일을 일관성 있게 유지
* 🤖 자동으로 문제를 고치는 기능까지 제공 (`eslint --fix`)

---

# 🧠 왜 ESLint가 중요한가?

### ✔ 1) 자바스크립트의 "너무 유연한" 특성 때문에

JS는 문법적으로 허용하는 범위가 매우 넓습니다.

예:

```js
if(x = 10) { ... }   // ===을 안 써도 오류 아님 → 치명적 버그
```

ESLint는 이런 코드에서 경고를 줍니다.

---

### ✔ 2) 협업할 때 코드 스타일을 강제할 수 있음

다른 개발자가 작성한 코드가 다음과 같다고 가정해 보겠습니다:

```js
const a=1
```

또는

```js
const    a = 1;
```

둘 다 돌아가지만 팀 코드 스타일이 엉망이 됩니다.
ESLint + Prettier를 조합하면 코드 형식이 항상 같아집니다.

---

### ✔ 3) Lint 규칙을 플러그인으로 커스터마이징 가능

React를 쓴다면 `eslint-plugin-react`
Hooks를 쓴다면 `eslint-plugin-react-hooks`
TypeScript를 쓴다면 `@typescript-eslint/eslint-plugin`

필요한 규칙을 원하는 대로 구성할 수 있습니다.

---

# 🛠️ ESLint는 어떻게 동작하나요?

1. 📂 프로젝트 안의 `.eslintrc.js` 또는 `.eslintrc.json` 설정 파일을 읽음
2. 🔍 설정된 규칙대로 JS/TS 파일을 정적 분석
3. ❗ 문제 있는 부분을 터미널, VSCode에 표시
4. 🔧 `eslint --fix` 로 자동 수정 가능

---

# 📦 기본 설치 방법

```bash
npm install eslint --save-dev
```

### 초기 세팅

```bash
npx eslint --init
```

이 과정에서:

* 어떤 환경? (Browser / Node / React / TS 등)
* 어떤 스타일? (Airbnb, Standard, Google)
* TypeScript 쓸 건지?
* ESM/ CommonJS?

등을 선택하면 `.eslintrc.*` 파일이 자동 생성됩니다.

---

# 🧩 ESLint 구성 요소

### 1️⃣ **Parser (파서)**

코드를 AST(Abstract Syntax Tree)로 분석
ex:

* 기본 `espree`
* TypeScript: `@typescript-eslint/parser`

### 2️⃣ **Rules (규칙)**

예:

* `"no-unused-vars": "warn"`
* `"eqeqeq": "error"`
* `"semi": ["error", "always"]`

### 3️⃣ **Plugins (플러그인)**

React: `eslint-plugin-react`
React Hooks: `eslint-plugin-react-hooks`
Import: `eslint-plugin-import`

### 4️⃣ **Config (설정 preset)**

Airbnb, Standard, Google 등
React 앱에서는 아래가 특히 흔합니다:

```json
{
  "extends": ["react-app", "react-app/jest"]
}
```

---

# 🧾 ESLint 설정 파일 예시 (.eslintrc.js)

```js
module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  parserOptions: {
    ecmaFeatures: { jsx: true },
    ecmaVersion: "latest",
    sourceType: "module"
  },
  plugins: ["react", "react-hooks"],
  rules: {
    "no-unused-vars": "warn",
    "react/react-in-jsx-scope": "off",
    "semi": ["error", "always"]
  }
};
```

---

# 🤝 ESLint + Prettier 조합: 최강의 코드 품질 파트너

* **ESLint → 코드 오류/버그 탐지**
* **Prettier → 코드 형식(포맷) 자동 정리**

둘은 역할이 다르기 때문에 함께 쓰는 것이 일반적입니다.

`eslint-config-prettier` 를 추가해 충돌을 방지할 수도 있습니다.

---

# 🧪 예시: ESLint가 잡아주는 문제

### 🚫 잘못된 코드

```js
let x = 10
if (x = 20) {
  console.log("error")
}
```

### 🧹 ESLint 경고

* `=` 대신 `==` 또는 `===`을 사용해야 한다는 경고
* 세미콜론 누락 경고

---

# 🎯 결론: ESLint는 개발 품질의 필수 요소

✔ 버그 예방
✔ 코드 스타일 일관성 유지
✔ 협업 시 필수
✔ React/Vite/Next.js 프로젝트 개발자의 업계 표준
✔ 자동 수정으로 생산성 증가

단순한 “스타일 검사기”가 아니라
👉 **코드 품질 유지, 에러 예방, 리팩터링 기반 도구**입니다.


