# O React 프런트엔드 개발을 위한 필수 설치 항목

## 1. **Visual Studio Code (VS Code)** – 코드 편집기

* 공식 사이트: [https://code.visualstudio.com/](https://code.visualstudio.com/)
* React 개발에 가장 널리 쓰이는 무료 에디터
* JavaScript, JSX, TypeScript 지원 탁월
* 확장 기능 (Extensions) 풍부

### 추천 확장 기능

| 확장명                          | 설명                        |
| ---------------------------- | ------------------------- |
| **ESLint**                   | 코드 문법/품질 검사 (직접 설치해서 쓰는 경우) |
| **Oxc (oxlint)**             | 린트 결과를 에디터에 표시 (현재 Vite React 템플릿의 기본 린터) |
| **Prettier**                 | 코드 자동 정렬                  |
| **ES7+ React/Redux snippets** | `rafce` 등 컴포넌트 스니펫 자동 생성 |
| **Auto Import**              | import 자동 삽입              |
| **Path Intellisense**        | 경로 자동 완성                  |
| **npm Intellisense**         | 패키지 자동 완성                 |

> **`Bracket Pair Colorizer 2`는 설치하지 마세요.** 개발이 중단된 확장이며,
> 동일 기능이 **VS Code에 기본 내장**되었습니다(설정에서 `editor.bracketPairColorization.enabled`, 기본값 켜짐).
> 확장으로 설치하면 오히려 에디터가 느려집니다.
>
> **`React Developer Tools`는 VS Code 확장이 아니라 브라우저(Chrome/Edge/Firefox) 확장**입니다. 아래 5번 항목에서 설치합니다.
>
> **린터는 ESLint에서 Oxlint로 바뀌었습니다.** 예전 Vite React 템플릿은 `eslint.config.js`와 ESLint 의존성을 함께 생성했지만,
> 현재(Vite 8 기준) 템플릿은 `.oxlintrc.json` + `"lint": "oxlint"` 스크립트를 생성하며 ESLint를 설치하지 않습니다.
> ESLint를 계속 쓰고 싶다면 직접 설치·설정해야 합니다.

---

## 2. **Node.js + npm** – 패키지 실행 및 설치 기반

* 공식 사이트: [https://nodejs.org/ko/](https://nodejs.org/ko/)
* **LTS (장기 지원 버전)** 설치 추천
* Node.js를 설치하면 **npm(Node Package Manager)** 도 자동으로 함께 설치됨

```bash
node -v      # Node.js 버전 확인
npm -v       # npm 버전 확인
```

> **버전 요구 사항**: Vite 7/8은 **Node.js 20.19.0 이상 또는 22.12.0 이상**이어야 실행됩니다.
> Node 18은 2025년 4월 EOL(지원 종료)되어 더 이상 사용할 수 없습니다.
> `node -v`가 `v18.x` 이하라면 먼저 최신 LTS로 업그레이드하세요.
>
> 프로젝트마다 Node 버전이 다르다면 버전 관리자를 쓰는 편이 안전합니다.
> macOS/Linux는 `nvm`, Windows는 `nvm-windows` 또는 `fnm`을 권장합니다.

---

## 3. **npx / npm create** – React 프로젝트 생성에 필요

* `npx`는 npm에 포함된 **패키지 실행기**로, 별도 설치가 필요 없습니다.
* 프로젝트 생성 명령:

```bash
npm create vite@latest my-app -- --template react
```

> **`--`를 반드시 넣어야 합니다.**
> `npm create vite@latest my-app --template react` 처럼 쓰면 npm이 `--template react`를 **자기 옵션으로 가로채서** create-vite에 전달하지 않습니다. 그러면 템플릿 선택 질문이 다시 뜹니다.

### X `create-react-app`은 더 이상 쓰지 마세요

```bash
npx create-react-app my-app   # 사용 금지 (deprecated)
```

* CRA는 **2025년 2월 React 팀이 공식적으로 지원 중단(deprecated)** 을 선언했고, 현재 React 공식 문서의 시작 가이드에서도 제외되었습니다.
* 실행하면 **deprecation 경고**가 뜨고, 의존성이 오래되어 `npm audit` 취약점 경고가 대량으로 발생합니다.
* 내부 Webpack 설정이 수년째 멈춰 있어 **React 19 및 최신 라이브러리와 충돌**하기 쉽습니다.
* 대안: **Vite**(이 강의 기준), 또는 라우팅·SSR이 필요하면 Next.js / React Router(framework mode).

---

## 4. Git – 버전 관리 도구 (선택이지만 사실상 필수)

* 설치: [https://git-scm.com/](https://git-scm.com/)
* Git은 팀 협업 및 GitHub와의 연동에 필수적
* VS Code 내장 Git 연동 기능과 매우 잘 호환

---

## 5. Chrome 브라우저 + React Developer Tools 확장

* Chrome: [https://www.google.com/chrome/](https://www.google.com/chrome/)
* React Developer Tools:
  [https://chrome.google.com/webstore/detail/react-developer-tools/](https://chrome.google.com/webstore/detail/react-developer-tools/)

> React 컴포넌트 구조, 상태(state), props 등을 브라우저에서 시각적으로 디버깅 가능

---

## (선택) 6. Postman 또는 Thunder Client

* API 테스트 도구 (백엔드 연동 테스트용)
* Thunder Client는 VS Code 확장으로 내장 가능

---

# 전체 요약: 설치 체크리스트

| 항목                              | 필수 여부  | 설치 위치                                                            | 설명                |
| ------------------------------- | ------ | ---------------------------------------------------------------- | ----------------- |
| O Visual Studio Code | 필수 | [https://code.visualstudio.com](https://code.visualstudio.com) | React 전용 편집기 |
| O Node.js + npm | 필수 | [https://nodejs.org/ko/](https://nodejs.org/ko/) | React 프로젝트 실행 기반 |
| O Git | 권장 | [https://git-scm.com](https://git-scm.com) | 버전 관리 및 GitHub 연동 |
| O Chrome 브라우저 | 필수 | [https://www.google.com/chrome/](https://www.google.com/chrome/) | 개발자 도구와 React 디버깅 |
| O React Developer Tools | 필수 | Chrome 확장 | React 상태/구조 확인용 |
| O ESLint, Prettier (VS Code 확장) | 권장 | VS Code 내부 | 코드 품질 관리 도구 |
| Thunder Client or Postman | 선택 | VS Code 확장 / postman.com | API 테스트 툴 |
| O npx | npm 포함 | Node.js 설치 시 자동 포함 | React 프로젝트 생성에 필요 |


