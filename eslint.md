# 🔍 ESLint란?

## ✅ 정의

**ESLint**는

> JavaScript/TypeScript 코드에서 **문법 오류, 스타일 위반, 잠재적 버그** 등을 **정적으로 분석(linting)** 하여 찾아내고, 자동으로 수정도 가능하게 해주는 **정적 분석 도구(linter)** 입니다.

### 즉,

* "내 코드를 **사람이 아닌 기계가 검사**한다"
* "버그 가능성 있는 코드나 규칙 위반 코드를 **미리 탐지**한다"
* "팀 규칙(코딩 스타일)을 강제한다"

---

# 🧠 왜 ESLint가 필요한가?

| 문제                     | 해결             |
| ---------------------- | -------------- |
| 코드 스타일이 제각각            | eslint가 규칙 통일  |
| 팀원 간 버그 발생             | eslint가 미리 경고  |
| `var`, `==` 등 구식 문법 사용 | 최신 문법 사용 유도    |
| 린터 없이 배포 후 오류 발생       | 작성 중에 정적 분석 가능 |

---

# 🎯 ESLint의 주요 기능

| 기능        | 설명                                          |
| --------- | ------------------------------------------- |
| 정적 코드 분석  | 코드를 실행하지 않고 구문, 로직, 스타일을 검사                 |
| 사용자 정의 규칙 | 규칙을 JSON 또는 JS 파일로 자유롭게 설정                  |
| 플러그인 시스템  | React, TypeScript, Vue 등 확장 가능              |
| 자동 수정 기능  | `--fix` 옵션으로 일부 오류 자동 수정                    |
| 통합 가능     | VSCode, Webpack, Vite, Git Hook 등과 쉽게 연동 가능 |

---

# 📦 설치 및 기본 사용법

## ✅ 설치

```bash
npm install --save-dev eslint
```

## ✅ 설정 초기화

```bash
npx eslint --init
```

> CLI가 질문을 통해 `.eslintrc` 파일을 자동 생성해줍니다.

## ✅ 기본 명령어

```bash
npx eslint src/**/*.js
npx eslint src --fix
```

* `--fix`: 고칠 수 있는 문제는 자동으로 고쳐줌

---

# ⚙️ `.eslintrc` 설정 파일 예시

```json
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended"
  ],
  "parserOptions": {
    "ecmaVersion": 2021,
    "sourceType": "module"
  },
  "plugins": ["react"],
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off",
    "eqeqeq": "error"
  }
}
```

| 키         | 설명                   |
| --------- | -------------------- |
| `env`     | 실행 환경 (브라우저, Node 등) |
| `extends` | 기본 규칙 세트             |
| `plugins` | 확장 기능 (React, JSX 등) |
| `rules`   | 사용자 지정 규칙 설정 가능      |

---

# 🔌 플러그인 및 확장 기능

| 사용 환경        | 플러그인/설정                                                         |
| ------------ | --------------------------------------------------------------- |
| React        | `eslint-plugin-react`, `plugin:react/recommended`               |
| TypeScript   | `@typescript-eslint/parser`, `@typescript-eslint/eslint-plugin` |
| Prettier 연동  | `eslint-config-prettier`                                        |
| Vite/Next.js | 기본으로 ESLint 통합 지원 가능                                            |

---

## 🧪 예제: React + ESLint 설정

```bash
npm install -D eslint eslint-plugin-react
```

`.eslintrc.js`:

```js
module.exports = {
  env: {
    browser: true,
    es2021: true
  },
  extends: ['eslint:recommended', 'plugin:react/recommended'],
  parserOptions: {
    ecmaFeatures: { jsx: true },
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: ['react'],
  rules: {
    'react/prop-types': 'off',
    'no-unused-vars': 'warn'
  }
};
```

---

# ⚠️ ESLint가 탐지하는 주요 코드 문제

| 규칙 ID                  | 설명                               | 예시                           |
| ---------------------- | -------------------------------- | ---------------------------- |
| `no-unused-vars`       | 사용하지 않는 변수 금지                    | `const a = 10;` ← 미사용 시 경고   |
| `eqeqeq`               | `==` 대신 `===` 사용                 | `a == b` → `a === b`         |
| `no-undef`             | 선언되지 않은 변수 사용 금지                 | `console.log(x);` ← x가 선언 안됨 |
| `no-console`           | `console.log` 사용 금지 (옵션)         | 로그 제거 강제                     |
| `react/jsx-uses-react` | JSX 사용 시 React import 필요 (17 이전) | JSX인데 `import React` 누락      |

---

# 🧩 Prettier와의 차이점

| 항목    | ESLint                         | Prettier                        |
| ----- | ------------------------------ | ------------------------------- |
| 목적    | **문법/스타일/버그 탐지**               | **코드 스타일 자동 포맷팅**               |
| 기능    | if/else 중괄호, 변수 사용, 비교 연산 등 검사 | 줄바꿈, 들여쓰기, 세미콜론 등 일관화           |
| 충돌 여부 | 일부 규칙 중복 가능                    | `eslint-config-prettier`로 해소 가능 |

> ✔️ **실무에서는 둘을 함께 사용하는 것이 일반적**입니다.

---

# ✅ 결론 요약

| 항목       | 설명                                         |
| -------- | ------------------------------------------ |
| ESLint란? | JavaScript 정적 분석 도구                        |
| 필요 이유    | 문법 오류, 스타일 위반, 버그 예방                       |
| 주요 기능    | 규칙 기반 분석, 자동 수정, 확장 플러그인                   |
| 실무 활용    | React, TypeScript, Vue, Node, Prettier와 통합 |
| 결과       | 코드 품질 향상, 협업 효율 증대, 오류 예방                  |

