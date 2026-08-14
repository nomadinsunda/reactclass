# `npm run dev` 실행 시 Vite Web Server 구동 원리

---

## O 1. `npm run dev` 실행

```bash
npm run dev
```

### 실행되는 스크립트

`package.json` 내에 다음과 같은 항목이 있습니다:

```json
{
  "scripts": {
    "dev": "vite"
  }
}
```

즉, `npm run dev`는 내부적으로 아래 실행 파일을 호출합니다:

```bash
./node_modules/.bin/vite
```

> **`npm run dev` ≠ `npx vite`**
> `npm run`은 `node_modules/.bin`을 **PATH 앞에 끼워 넣고** 스크립트 문자열을 셸에서 실행합니다.
> 반면 `npx`는 로컬에 없으면 **레지스트리에서 임시로 내려받아** 실행합니다.
> 그래서 `npm install`을 깜빡했을 때 `npm run dev`는 `vite: command not found`로 실패하지만, `npx vite`는 엉뚱한 최신 버전을 받아 실행되어 원인을 헷갈리게 만듭니다.
> `node_modules/.bin/vite`는 `npm install` 시 `vite` 패키지의 `bin` 필드를 보고 만들어진 심볼릭 링크입니다.

---

## O 2. Vite의 개발 서버(`vite`)가 실행됨

`vite` CLI는 아래 작업들을 수행합니다:

### 1⃣ `vite.config.js` 파일을 로딩

* 사용자가 정의한 플러그인, 경로 별칭, 포트 등 설정을 읽습니다.
* `@vitejs/plugin-react`도 이 단계에서 활성화됩니다.

### 2⃣ 의존성 사전 번들링(Dependency Pre-Bundling) 수행

* `node_modules`에서 실제로 import되는 패키지를 스캔
* 이를 **ESM 하나로 합쳐** `node_modules/.vite/deps/`에 캐싱 (Vite 7까지 esbuild, Vite 8부터 Rolldown이 담당)
* **왜?** ① `react` 등 CommonJS 패키지를 브라우저가 읽을 수 있는 ESM으로 바꾸고 ② 내부 파일이 수백 개인 패키지 때문에 요청이 폭증하는 것을 막기 위해
* 이미 캐시가 있고 의존성/lock 파일/설정이 그대로면 **이 단계를 건너뜁니다** → 두 번째 실행부터 "ready in 300ms"가 나오는 이유

### 3⃣ 개발용 웹 서버(Vite Dev Server) 생성

* Node.js의 **connect 미들웨어 스택** 위에 구성된 HTTP 서버를 생성
* `http://localhost:5173` 기본 포트로 바인딩 (사용 중이면 5174, 5175…로 자동 증가)

### 4⃣ `index.html`을 루트 리소스로 등록

* Vite는 **엔트리포인트가 JS가 아니라 HTML**이라는 점이 Webpack과 결정적으로 다름
* `index.html`을 단순 정적 파일로 처리하지 않고 **파싱하여 중간에 개입** (`transformIndexHtml` 훅, HMR 클라이언트 스크립트 주입 등)

---

## O 3. 브라우저가 `localhost:5173` 접속 시 내부 처리 흐름

```plaintext
브라우저 요청: GET /index.html
↓
Vite 서버: index.html 열람 및 변형
↓
- <script type="module" src="/src/main.jsx"> 구문 파싱
↓
- /src/main.jsx 요청 처리 → esbuild(Vite 8부터는 Oxc)로 변환
↓
- 모듈 그래프 생성 및 캐싱
↓
응답: 변환된 JS 코드를 브라우저에 전송
```

### 핵심: Vite는 **ESM(import/export)** 을 브라우저에 직접 제공합니다.

---

## O 4. `.jsx`, `.js` 등 모듈 요청 시

### 브라우저가 `/src/App.jsx`를 요청하면:

```plaintext
1. 서버는 App.jsx 파일을 esbuild(Vite 8부터는 Oxc)로 트랜스파일
2. JSX → jsx()/jsxs()  ※ react/jsx-runtime 에서 자동 import
3. import 경로를 브라우저가 해석 가능한 경로로 재작성
   예) import React from 'react'
       → import React from '/node_modules/.vite/deps/react.js'
4. export된 모듈을 브라우저에 ESM 형태로 전송
```

→ 이 과정은 매우 빠르며, 파일이 바뀌지 않는 한 캐시됨(`304 Not Modified`)

> **2번은 예전 방식과 다릅니다.** React 16까지는 JSX가 `React.createElement()`로 변환되어
> 모든 파일 맨 위에 `import React from 'react'`가 필요했습니다.
> React 17에서 도입된 **automatic JSX runtime** 이후로는 `react/jsx-runtime`의 `jsx()` 호출로 변환되며,
> **컴파일러가 필요한 import를 자동으로 넣어 주기 때문에 `import React`를 쓰지 않아도 됩니다.**
> Vite React 템플릿의 `App.jsx`에 `import React`가 없는 이유가 바로 이것입니다.

> **3번(경로 재작성)도 핵심입니다.** 브라우저의 네이티브 ESM은 `'react'` 같은 **bare import를 해석하지 못합니다**(`/`, `./`, `../`로 시작해야 함).
> Vite는 이 경로를 사전 번들링해 둔 실제 파일 경로로 바꿔서 내려보냅니다. 이것이 "번들러 없이 브라우저 ESM을 쓴다"는 말이 성립하게 만드는 장치입니다.

---

## O 5. HMR (Hot Module Replacement) 작동 방식

### 예: `App.jsx` 수정 시

```plaintext
1. 파일 시스템에서 App.jsx 변경 감지 (chokidar 기반)
2. 모듈 그래프에서 App.jsx에 의존한 모듈을 추적
3. 웹소켓을 통해 브라우저에 변경 통지
4. 브라우저는 해당 모듈만 다시 요청
5. React Fast Refresh가 상태 유지하며 렌더링 갱신
```

> 즉, **전체 페이지를 새로고침하지 않고도 부분만 업데이트**됩니다.

---

## O 6. 콘솔에 출력되는 Vite 개발 서버 메시지

```plaintext
VITE v8.x.x  ready in 300ms

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

# 요약: `npm run dev` → 어떤 일이 벌어지나?

| 단계 | 설명                                                  |
| -- | --------------------------------------------------- |
| 1  | `vite` CLI 실행 → `vite.config.js` 읽기                 |
| 2  | **의존성 사전 번들링** → `node_modules/.vite/deps` (캐시가 유효하면 생략) |
| 3  | `createServer()`로 개발용 웹 서버 구성                       |
| 4  | `index.html`을 파싱하고 ESM 스크립트 분석 + HMR 클라이언트 주입       |
| 5  | 요청받은 `.jsx`, `.js`, `.ts` 등을 on-demand 변환 + import 경로 재작성 |
| 6  | 브라우저 요청마다 해당 모듈만 전송                                 |
| 7  | 파일 변경 감지 시 HMR 작동 (React Fast Refresh 포함)           |

---

## O 내부 기술 스택 요약

| 구성 요소                             | 역할                                          |
| --------------------------------- | ------------------------------------------- |
| **esbuild** (Vite 8부터 **Oxc**)     | 개발 시 JSX/TS 트랜스파일 (타입 검사는 하지 않음)             |
| **esbuild** (Vite 8부터 **Rolldown**) | 의존성 사전 번들링 → `node_modules/.vite/deps`       |
| **Node.js + connect 미들웨어**         | HTTP 요청 처리 (Vite 2부터 Koa가 아닌 connect 기반)     |
| **chokidar**                      | 파일 변경 감지                                    |
| **WebSocket**                     | HMR 갱신 메시지 전달                               |
| **@vitejs/plugin-react**          | React Fast Refresh 주입 + JSX automatic runtime 설정 |
| **module graph**                  | 모듈 간 의존성 추적 → 파일 변경 시 HMR 경계 계산에 사용          |


