# ✅ React 프런트엔드 개발을 위한 필수 설치 항목

## 🧰 1. **Visual Studio Code (VS Code)** – 코드 편집기

* 공식 사이트: [https://code.visualstudio.com/](https://code.visualstudio.com/)
* React 개발에 가장 널리 쓰이는 무료 에디터
* JavaScript, JSX, TypeScript 지원 탁월
* 확장 기능 (Extensions) 풍부

### 📦 추천 확장 기능

| 확장명                          | 설명                        |
| ---------------------------- | ------------------------- |
| **ESLint**                   | 코드 문법 검사                  |
| **Prettier**                 | 코드 자동 정렬                  |
| **React Developer Tools**    | React 컴포넌트 디버깅 (브라우저 연동용) |
| **Auto Import**              | import 자동 삽입              |
| **Path Intellisense**        | 경로 자동 완성                  |
| **npm Intellisense**         | 패키지 자동 완성                 |
| **Bracket Pair Colorizer 2** | 괄호 구분 색상 표시               |

---

## 🟩 2. **Node.js + npm** – 패키지 실행 및 설치 기반

* 공식 사이트: [https://nodejs.org/ko/](https://nodejs.org/ko/)
* **LTS (장기 지원 버전)** 설치 추천
* Node.js를 설치하면 **npm(Node Package Manager)** 도 자동으로 함께 설치됨

```bash
node -v      # Node.js 버전 확인
npm -v       # npm 버전 확인
```

---

## ⚙️ 3. **npx** (npm ≥ 5.2 포함) – React 프로젝트 생성에 필요

* `npx`는 npm에 포함된 실행기입니다.
* 설치는 따로 필요 없지만, 구버전 npm에서는 사용 불가합니다.
* React 프로젝트 초기 생성 시 사용됨:

```bash
npx create-react-app my-app
```

> 또는 Vite 기반일 경우:

```bash
npm create vite@latest my-app --template react
```

---

## 🧱 4. Git – 버전 관리 도구 (선택이지만 사실상 필수)

* 설치: [https://git-scm.com/](https://git-scm.com/)
* Git은 팀 협업 및 GitHub와의 연동에 필수적
* VS Code 내장 Git 연동 기능과 매우 잘 호환

---

## 🌐 5. Chrome 브라우저 + React Developer Tools 확장

* Chrome: [https://www.google.com/chrome/](https://www.google.com/chrome/)
* React Developer Tools:
  [https://chrome.google.com/webstore/detail/react-developer-tools/](https://chrome.google.com/webstore/detail/react-developer-tools/)

> React 컴포넌트 구조, 상태(state), props 등을 브라우저에서 시각적으로 디버깅 가능

---

## 🧪 (선택) 6. Postman 또는 Thunder Client

* API 테스트 도구 (백엔드 연동 테스트용)
* Thunder Client는 VS Code 확장으로 내장 가능

---

# 🧰 전체 요약: 설치 체크리스트

| 항목                              | 필수 여부  | 설치 위치                                                            | 설명                |
| ------------------------------- | ------ | ---------------------------------------------------------------- | ----------------- |
| ✅ Visual Studio Code            | 필수     | [https://code.visualstudio.com](https://code.visualstudio.com)   | React 전용 편집기      |
| ✅ Node.js + npm                 | 필수     | [https://nodejs.org/ko/](https://nodejs.org/ko/)                 | React 프로젝트 실행 기반  |
| ✅ Git                           | 권장     | [https://git-scm.com](https://git-scm.com)                       | 버전 관리 및 GitHub 연동 |
| ✅ Chrome 브라우저                   | 필수     | [https://www.google.com/chrome/](https://www.google.com/chrome/) | 개발자 도구와 React 디버깅 |
| ✅ React Developer Tools         | 필수     | Chrome 확장                                                        | React 상태/구조 확인용   |
| ✅ ESLint, Prettier (VS Code 확장) | 권장     | VS Code 내부                                                       | 코드 품질 관리 도구       |
| 🔄 Thunder Client or Postman    | 선택     | VS Code 확장 / postman.com                                         | API 테스트 툴         |
| ✅ npx                           | npm 포함 | Node.js 설치 시 자동 포함                                               | React 프로젝트 생성에 필요 |

---

필요하시면 다음 단계로 **설치 후 React 프로젝트 생성 및 실행** 방법,
또는 **TypeScript 기반 React 개발 환경 구성**도 정리해드릴 수 있습니다.

원하시는 방향 알려주세요.
