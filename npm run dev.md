# 🚀 `npm run dev` 실행 시 Vite Web Server 구동 원리

---

## ✅ 1. `npm run dev` 실행

```bash
npm run dev
```

### 📦 실행되는 스크립트

`package.json` 내에 다음과 같은 항목이 있습니다:

```json
{
  "scripts": {
    "dev": "vite"
  }
}
```

즉, `npm run dev`는 내부적으로 아래 명령을 실행합니다:

```bash
npx vite
```

---

## ✅ 2. Vite의 개발 서버(`vite`)가 실행됨

`vite` CLI는 아래 작업들을 수행합니다:

### 1️⃣ `vite.config.js` 파일을 로딩

* 사용자가 정의한 플러그인, 경로 별칭, 포트 등 설정을 읽습니다.
* `@vitejs/plugin-react`도 이 단계에서 활성화됩니다.

### 2️⃣ 개발용 웹 서버(Vite Dev Server) 생성

* Node.js 기반 **커스텀 Express-like 서버**를 생성
* `http://localhost:5173` 기본 포트로 HTTP 서버 바인딩

### 3️⃣ `index.html`을 루트 리소스로 등록

* Vite는 **SPA 엔트리포인트가 HTML**이라는 특징이 있음
* `index.html`을 단순 정적 파일로 처리하지 않고 **파싱하여 중간에 개입**

---

## ✅ 3. 브라우저가 `localhost:5173` 접속 시 내부 처리 흐름

```plaintext
브라우저 요청: GET /index.html
↓
Vite 서버: index.html 열람 및 변형
↓
- <script type="module" src="/src/main.jsx"> 구문 파싱
↓
- /src/main.jsx 요청 처리 → esbuild로 변환
↓
- 모듈 그래프 생성 및 캐싱
↓
응답: 변환된 JS 코드를 브라우저에 전송
```

### 🔍 핵심: Vite는 **ESM(import/export)** 을 브라우저에 직접 제공합니다.

---

## ✅ 4. `.jsx`, `.js` 등 모듈 요청 시

### 브라우저가 `/src/App.jsx`를 요청하면:

```plaintext
1. 서버는 App.jsx 파일을 esbuild로 트랜스파일
2. JSX → React.createElement()
3. export된 모듈을 브라우저에 ESM 형태로 전송
```

→ 이 과정은 매우 빠르며, 캐시됨

---

## ✅ 5. HMR (Hot Module Replacement) 작동 방식

### 🔄 예: `App.jsx` 수정 시

```plaintext
1. 파일 시스템에서 App.jsx 변경 감지 (chokidar 기반)
2. 모듈 그래프에서 App.jsx에 의존한 모듈을 추적
3. 웹소켓을 통해 브라우저에 변경 통지
4. 브라우저는 해당 모듈만 다시 요청
5. React Fast Refresh가 상태 유지하며 렌더링 갱신
```

> 즉, **전체 페이지를 새로고침하지 않고도 부분만 업데이트**됩니다.

---

## ✅ 6. 콘솔에 출력되는 Vite 개발 서버 메시지

```plaintext
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

### 이 메시지는 다음 이벤트 루프를 대기합니다:

| 키 입력        | 동작                          |
| ----------- | --------------------------- |
| `r` + Enter | 서버 재시작 (`server.restart()`) |
| `o` + Enter | 브라우저로 자동 열기                 |
| `q` + Enter | `process.exit()` 호출         |

---

# 🔧 요약: `npm run dev` → 어떤 일이 벌어지나?

| 단계 | 설명                                                  |
| -- | --------------------------------------------------- |
| 1  | `vite` CLI 실행 → `vite.config.js` 읽기                 |
| 2  | `createServer()`로 개발용 웹 서버 구성                       |
| 3  | `index.html`을 파싱하고 ESM 스크립트 분석                      |
| 4  | `esbuild`로 필요한 `.jsx`, `.js`, `.ts` 등을 on-demand 변환 |
| 5  | 브라우저 요청마다 해당 모듈만 전송                                 |
| 6  | 파일 변경 감지 시 HMR 작동 (React Fast Refresh 포함)           |

---

## ✅ 내부 기술 스택 요약

| 구성 요소                         | 역할                         |
| ----------------------------- | -------------------------- |
| **esbuild**                   | 개발 시 JSX/TS 트랜스파일          |
| **Node.js + Koa-like server** | HTTP 요청 처리                 |
| **chokidar**                  | 파일 변경 감지                   |
| **WebSocket**                 | HMR 핫 모듈 전달                |
| **@vitejs/plugin-react**      | JSX → JS + Fast Refresh 통합 |
| **module graph**              | 모듈 간 의존성 추적 (HMR 시 활용)     |


