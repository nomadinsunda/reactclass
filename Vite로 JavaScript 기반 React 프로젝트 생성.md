
# 🚀 Vite로 JavaScript 기반 React 프로젝트 생성하기

> ✅ 목적:
> **Vite + React + JavaScript (TypeScript 없이)** 구성된 초경량 프로젝트를 만드는 방법

---

## ✅ 사전 준비

### 🔧 Node.js 설치 확인

```bash
node -v
npm -v
```

* 최소 권장 버전: `Node.js >= 18.x`
* Vite 7 기준: `Node.js >= 20.19.0` 또는 `>= 22.12.0` 권장

> 설치 안 되어 있다면: [https://nodejs.org/](https://nodejs.org/) 에서 LTS 버전 설치

---

## ✅ 1단계: Vite CLI[Visual Studio Code의 터미너(gitBash 추천)]로 프로젝트 생성

```bash
npm create vite@latest
```

실행 시 CLI가 순서대로 묻습니다:

```
✔ Project name: » my-vite-react-app
✔ Select a framework: » React
✔ Select a variant: » JavaScript
```

### 또는 한 줄 명령어:

```bash
npm create vite@latest my-vite-react-app -- --template react
```

📁 이 명령어로 다음과 같은 템플릿이 생성됩니다:

```
my-vite-react-app/
├── index.html
├── package.json
├── vite.config.js
├── /src
│   ├── main.jsx
│   └── App.jsx
└── /node_modules
```

---

## ✅ 2단계: 생성된 프로젝트 디렉터리로 이동

```bash
cd my-vite-react-app
```

---

## ✅ 3단계: 의존성 설치

```bash
npm install
```

설치되는 주요 패키지:

| 패키지                    | 설명                               |
| ---------------------- | -------------------------------- |
| `react`, `react-dom`   | React 라이브러리                      |
| `vite`                 | 개발 서버 및 빌드 도구                    |
| `@vitejs/plugin-react` | React + JSX 지원 (Fast Refresh 포함) |

---

## ✅ 4단계: 개발 서버 실행

```bash
npm run dev
```

출력 예시:

```
VITE v7.x.x  ready in 300ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose

  shortcuts:
  press r + enter to restart the server
  press u + enter to show server url
  press o + enter to open in browser
  press c + enter to clear console
  press q + enter to quit
```

---

## ✅ 5단계: 브라우저에서 앱 확인

```
http://localhost:5173/
```

* React Welcome 화면이 보이면 성공
* 수정 사항은 자동으로 반영됨 (Hot Module Replacement)

---

## ✅ 6단계: Vite 실행 중 사용할 수 있는 단축키 (Shortcuts)

| 단축키         | 동작                  |
| ----------- | ------------------- |
| `r + Enter` | 개발 서버 재시작 (Restart) |
| `u + Enter` | 현재 서버 URL 다시 출력     |
| `o + Enter` | 브라우저로 자동 열기         |
| `c + Enter` | 터미널 화면 클리어          |
| `q + Enter` | 서버 종료 (Quit)        |

---

## 📁 디렉터리 구조 설명

```bash
my-vite-react-app/
├── index.html            # 진입 HTML (Vite의 엔트리)
├── package.json          # 프로젝트 정의, 명령어, 의존성
├── vite.config.js        # Vite 설정 파일 (Rollup 기반)
├── /src
│   ├── main.jsx          # 앱 진입점 (ReactDOM.render)
│   └── App.jsx           # 루트 컴포넌트
└── /node_modules         # 설치된 패키지들
```

---

## ✅ 추가: 주요 명령어 정리

| 명령어               | 설명                   |
| ----------------- | -------------------- |
| `npm run dev`     | 개발 서버 실행 (HMR 지원)    |
| `npm run build`   | 프로덕션 빌드 → `dist/` 생성 |
| `npm run preview` | 빌드 결과물을 로컬 서버로 테스트   |

---

## 🧠 Vite의 장점 요약

| 항목     | 설명                                               |
| ------ | ------------------------------------------------ |
| 빠른 HMR | 모듈 단위로 트랜스파일 & 캐싱                                |
| 즉시 실행  | Cold start 속도가 매우 빠름                             |
| 간단한 설정 | `vite.config.js`는 기본적으로 생략 가능                    |
| JSX 지원 | `@vitejs/plugin-react`로 JSX 및 Fast Refresh 자동 지원 |

---

## ✅ 결론 요약

| 순서 | 명령어                               |
| -- | --------------------------------- |
| 1  | `npm create vite@latest`          |
| 2  | `cd my-vite-react-app`            |
| 3  | `npm install`                     |
| 4  | `npm run dev`                     |
| 5  | `브라우저에서 http://localhost:5173 접속` |
| 6  | `r`, `o`, `q` 등 단축키 활용            |

