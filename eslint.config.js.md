## ✅ 1. VS Code ESLint 확장은 **런타임이 아닌 클라이언트 도구**입니다

| 항목                   | 설명                                                              |
| -------------------- | --------------------------------------------------------------- |
| ✅ VS Code ESLint 확장  | 편집기에서 ESLint가 제공하는 진단 메시지를 보여주고 자동 수정(Lint on Save)을 지원합니다.     |
| ❌ `.eslintrc.*` 생성 X | 구성성 파일은 **자동으로 생성되지 않으며**, 프로젝트에 ESLint가 이미 설치되어 있어야 정상 작동합니다.   |
| ❌ ESLint 설치 X        | 확장만 설치해도 실제 프로젝트에는 ESLint가 **설치되지 않습니다.** (`node_modules`에는 없음) |

즉, **VS Code 확장은 보조 도구일 뿐, ESLint 자체를 설치하거나 설정하지 않습니다.**

---

## ✅ 2. ESLint는 프로젝트마다 `npm install`로 설치해야 작동함

당신의 프로젝트에 ESLint가 아직 설치되어 있지 않다면, 다음을 실행해야 합니다:

```bash
npm install -D eslint
```

또는 생성까지 포함하려면:

```bash
npx eslint --init
```

---

## ✅ 3. `.eslintrc.*` 설정 파일은 직접 생성하거나 `--init`으로 자동 생성해야 함

VS Code 확장을 설치했다고 해서 `.eslintrc.cjs`, `.eslintrc.json` 같은 설정 파일이 **자동으로 생기지 않습니다.**

👉 **직접 `npx eslint --init`으로 생성**하거나,
👉 **아래와 같이 수동으로 작성해야 합니다:**

```bash
// .eslintrc.cjs (예시)
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: ['eslint:recommended', 'plugin:react/recommended'],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react'],
  rules: {
    'react/react-in-jsx-scope': 'off', // React 17+는 필요 없음
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

---

## ✅ 4. VS Code ESLint 확장이 작동하려면 이 파일이 필수

🔔 `.eslintrc.*` 설정 파일이 없으면 **VS Code 상에서 ESLint 확장도 작동하지 않으며**, `"ESLint is disabled"`라는 메시지가 보일 수 있습니다.

---

## ✅ 5. ESLint + VS Code 연동 제대로 하려면?

### 📁 프로젝트 루트 예시

```
my-react-app/
├── .eslintrc.cjs
├── package.json
├── node_modules/
├── src/
```

### 📦 필요한 패키지 예시

```bash
npm install -D eslint eslint-plugin-react
```

### 📄 VSCode 설정 (선택적)

`.vscode/settings.json`:

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": ["javascript", "javascriptreact"]
}
```

---

## ✅ 결론 요약

| 상황                  | 설명                                            |
| ------------------- | --------------------------------------------- |
| VSCode 확장만 설치       | `.eslintrc`는 생성되지 않음. 작동도 안 함                 |
| `.eslintrc.*` 생성하려면 | `npx eslint --init`을 실행해야 함                   |
| 프로젝트에 ESLint 설치 필수  | `npm install -D eslint`                       |
| VSCode에서 제대로 쓰려면    | `.eslintrc.*` + `eslint-plugin-react` + 설정 필요 |

```
$ npx eslint --init
You can also run this command directly using 'npm init @eslint/config@latest'.
Need to install the following packages:
  @eslint/create-config@1.9.0
Ok to proceed? (y) y
@eslint/create-config: v1.9.0

√ What do you want to lint? · javascript
√ How would you like to use ESLint? · problems
√ What type of modules does your project use? · esm
√ Which framework does your project use? · react
√ Does your project use TypeScript? · no / yes
√ Where does your code run? · browser
The config that you've selected requires the following dependencies:

eslint, @eslint/js, globals, eslint-plugin-react
√ Would you like to install them now? · No / Yes
√ Which package manager do you want to use? · npm
☕️Installing...

added 112 packages, and audited 265 packages in 8s

112 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
Successfully created C:\development\Workspace\codes\frontend\reactclasses\class2\eslint.config.js file.
```

