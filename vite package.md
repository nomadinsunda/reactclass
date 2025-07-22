`npm create vite@latest` 명령어를 통해 Vite 프로젝트를 생성하면, 자동으로 생성되는 **`package.json`과 node\_modules 내의 패키지들**은 Vite와 React(혹은 선택한 프레임워크)를 구동하기 위한 핵심 요소들입니다. 여기서는 **Vite + React + JavaScript** 템플릿 기준으로 분석해드리겠습니다.

---

# 📦 Vite에 의해 생성되는 `package.json` 구성 요소 상세 설명

```json
{
  "name": "vite-react-js",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.0.3",
    "vite": "^7.0.5"
  }
}
```

---

## 🔹 1. `scripts` 섹션

| 스크립트      | 설명                                        |
| --------- | ----------------------------------------- |
| `dev`     | 개발 서버 실행 (`vite` 명령)                      |
| `build`   | 프로덕션용 정적 파일 번들 생성 (`vite build`)          |
| `preview` | `build` 결과물을 로컬 서버로 미리보기 (`vite preview`) |

---

## 🔹 2. `dependencies`

### ✅ `react`

* React 핵심 라이브러리
* 컴포넌트, Hooks, 상태 관리 등을 제공

### ✅ `react-dom`

* React 컴포넌트를 실제 DOM에 렌더링하는 브릿지 역할
* `ReactDOM.createRoot(...)` 등 제공

---

## 🔹 3. `devDependencies`

### ✅ `vite`

* Vite 본체
* 개발 서버, HMR, 빌드 등 모든 CLI 기능의 핵심
* 내부적으로 Rollup을 사용해 `vite build` 시 프로덕션 번들 생성

### ✅ `@vitejs/plugin-react`

* React를 Vite와 통합하기 위한 플러그인
* 다음을 처리:

  * JSX → JavaScript 변환 (babel 사용)
  * Fast Refresh(HMR) 지원
  * React-specific syntax 최적화

📌 이 플러그인이 없으면 JSX를 브라우저가 이해하지 못함

---

# 📂 생성된 디렉터리 구조와 역할

```bash
vite-react-js/
├── node_modules/        # 설치된 모든 패키지들
├── public/              # 정적 자산 위치 (favicon, 이미지 등)
├── src/                 # 소스 코드
│   ├── main.jsx         # 진입점
│   └── App.jsx          # 루트 컴포넌트
├── index.html           # 앱의 HTML 진입점 (브라우저 로드용)
├── package.json         # 프로젝트 의존성 및 스크립트
├── vite.config.js       # Vite 설정 파일 (필요 시)
└── .gitignore           # Git 제외 파일 목록
```

---

# 🧩 Vite가 설치한 핵심 모듈 설명 (node\_modules 내부)

| 패키지명                                    | 설명                                                  |
| --------------------------------------- | --------------------------------------------------- |
| `vite`                                  | Vite의 CLI, 서버, 번들러 기능 포함                            |
| `@vitejs/plugin-react`                  | React JSX 트랜스파일 + HMR 플러그인                          |
| `esbuild`                               | Vite 개발 서버의 고속 트랜스파일러 (Go로 작성된 빌드 도구)               |
| `rollup`                                | Vite의 프로덕션 번들러 (vite build 시 사용)                    |
| `react` / `react-dom`                   | React 앱의 핵심 라이브러리                                   |
| `@babel/*`                              | JSX → JS 변환 등 Babel 관련 트랜스파일 모듈 (React 플러그인 내부 의존성) |
| `ansi-colors`, `debug`, `picocolors`    | CLI 출력용 유틸 (vite, rollup 내부 의존성)                    |
| `fsevents`                              | 파일 변경 감지용 (macOS 전용)                                |
| `postcss`, `css-loader`, `style-loader` | CSS 처리용 (자동 포함될 수 있음)                               |

---

## ⚠️ 참고: Vite는 Webpack과 달리…

* 기본적으로 **zero-config (설정 거의 없음)** 구조를 가짐
* 설정이 필요하면 `vite.config.js`에 Rollup-style로 설정
* `webpack.config.js`, `babel.config.js`, `tsconfig.json` 없이도 작동함

---

## 🧪 개발자가 알아야 할 필수 Vite 내부 의존성 구조

```plaintext
Vite
 ├─ esbuild (개발용 변환기, JSX + TS 트랜스파일)
 ├─ Rollup (build 시 프로덕션 번들 생성기)
 └─ Plugin System
     ├─ @vitejs/plugin-react
     └─ Babel (JSX 변환 + Fast Refresh)
```

---

## 📌 마무리 정리

| 항목                     | 설명                                   |
| ---------------------- | ------------------------------------ |
| `vite`                 | 개발 서버 + 프로덕션 번들링 핵심                  |
| `@vitejs/plugin-react` | JSX 변환 + HMR 지원                      |
| `react`, `react-dom`   | 필수 UI 라이브러리                          |
| `scripts`              | `vite`, `vite build`, `vite preview` |
| `node_modules`         | 위 모든 패키지와 플러그인의 실제 코드 저장소            |

---



# 🧩 1. `vite.config.js` 커스터마이징 예제

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
    }
  },
  server: {
    port: 5173,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          react: ['react', 'react-dom'],
          vendor: ['axios', 'lodash']
        }
      }
    }
  }
});
```

### 🔍 주요 설정 설명

| 항목                                  | 설명                                  |
| ----------------------------------- | ----------------------------------- |
| `plugins`                           | Vite 플러그인 배열 (여기선 React 지원 플러그인 사용) |
| `resolve.alias`                     | 경로 단축 설정 (ex: `@components/Button`) |
| `server.port`                       | 개발 서버 포트                            |
| `server.proxy`                      | API 프록시 설정 (CORS 회피 등)              |
| `build.outDir`                      | 빌드 결과물 경로                           |
| `sourcemap`                         | 디버깅용 소스맵 생성                         |
| `rollupOptions.output.manualChunks` | 코드 스플리팅 명시적 설정 (캐시 최적화 목적)          |

---

# 🔧 2. Vite 내부의 Rollup 설정 구조

## 📦 `vite build` 명령을 실행하면

1. 내부적으로 `vite`는 Rollup을 import
2. `rollup.rollup()` 호출
3. Vite의 `rollupOptions`를 머지한 설정 객체 생성
4. **코드 분석 → 트리 셰이킹 → 번들링**

### 📌 기본적인 Rollup 설정 예시 (Vite 내부)

```js
{
  input: '/src/main.jsx',
  plugins: [
    viteInternalPlugin,
    reactPlugin,
    esbuildTransformPlugin,
    cssPlugin,
    ...
  ],
  output: {
    dir: 'dist',
    format: 'es',
    entryFileNames: '[name].[hash].js',
    chunkFileNames: 'chunks/[name].[hash].js',
    assetFileNames: 'assets/[name].[ext]',
    manualChunks: { ... }
  }
}
```

---

# 🔌 3. Vite 플러그인 시스템 구조

Vite의 플러그인은 **Rollup 플러그인을 기반으로 설계되었지만**, **Vite 전용 Hook도 지원**합니다.

## ✅ Rollup 플러그인 API

* `name`: 플러그인 이름
* `resolveId(source)`: 모듈 import 경로 분석
* `load(id)`: 모듈 로드 시 동작
* `transform(code, id)`: 코드 변환 시점

## ✅ Vite 전용 Hook

| Hook                       | 설명                |
| -------------------------- | ----------------- |
| `configureServer(server)`  | 개발 서버 인스턴스에 직접 접근 |
| `transformIndexHtml(html)` | `index.html` 조작   |
| `handleHotUpdate(context)` | HMR 핸들링 커스터마이징    |

---

## 📦 예: Vite 전용 플러그인 만들기

```js
export default function myLoggerPlugin() {
  return {
    name: 'vite:my-logger',
    transform(code, id) {
      if (id.endsWith('.jsx')) {
        console.log(`[Transform] ${id}`);
      }
      return null;
    },
    handleHotUpdate(ctx) {
      console.log(`[HMR] ${ctx.file}`);
    }
  };
}
```

* `transform`: 특정 파일 타입을 감지하여 로그 출력
* `handleHotUpdate`: 파일 변경 시 커스텀 동작 가능

---

# 📘 정리: Vite 고급 구성 핵심 요약

| 항목                    | 설명                                   |
| --------------------- | ------------------------------------ |
| `vite.config.js`      | Vite의 모든 설정 엔트리포인트, Rollup 설정도 포함    |
| `build.rollupOptions` | Vite → Rollup으로 전달되는 옵션              |
| Vite Plugin           | Rollup 플러그인 기반 + 자체 Hook도 지원         |
| 수동 청크 설정              | 큰 라이브러리(react 등)를 별도 청크로 분리 → 캐싱 최적화 |
| 프록시 설정                | CORS 우회 또는 백엔드 API 프록시 필요 시 활용       |

