# 🚀 Vite 기반 React 프로젝트 실행 방법 (이미 생성된 프로젝트 기준)

---

## ✅ 1단계: 프로젝트 디렉터리로 이동

```bash
cd [프로젝트_디렉터리명]
```

예:

```bash
cd my-vite-react-app
```

---

## ✅ 2단계: 의존성 설치

```bash
npm install
```

> 📦 이 명령은 `package.json`의 의존성(`react`, `vite`, `@vitejs/plugin-react` 등)을 바탕으로 `node_modules/`를 생성합니다.

---

## ✅ 3단계: 개발 서버 실행

```bash
npm run dev
```

* 기본 포트는 `http://localhost:5173`
* `vite.config.js`에 따로 설정하지 않았다면, 자동으로 이 주소에서 서버가 실행됩니다.
* 터미널에는 다음과 같이 출력됩니다:

```
VITE v7.x.x  ready in 300ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
h
  shortcuts:
  press r + enter to restart the server
  press u + enter to show server url
  press o + enter to open in browser
  press c + enter to clear console
  press q + enter to quit
```

---

## 🧠 4단계: Vite 단축키(Shortcuts) 사용법

Vite 개발 서버 실행 중에 **터미널에 직접 키보드 입력**으로 다음 명령들을 수행할 수 있습니다:

| 단축키         | 동작                          |
| ----------- | --------------------------- |
| `r + Enter` | 서버 재시작 (Restart dev server) |
| `u + Enter` | 현재 서버 URL 표시                |
| `o + Enter` | 브라우저에서 자동으로 열기              |
| `c + Enter` | 콘솔 로그 클리어 (터미널 화면 초기화)      |
| `q + Enter` | 서버 종료 (Quit)                |

> 🔄 이 단축키는 Vite 개발 서버 터미널에서 **`Ctrl + C`를 누르지 않고도 빠른 제어**가 가능하게 해줍니다.

---

## ✅ 5단계: 브라우저에서 확인

```bash
http://localhost:5173
```

* 위 주소를 브라우저에서 열면 React 앱이 정상적으로 실행됩니다.
* 기본 Welcome 화면이 보이면 성공입니다.

---

## 📦 추가 명령어 요약

| 명령어               | 설명                    |
| ----------------- | --------------------- |
| `npm run dev`     | 개발 서버 실행 (HMR 포함)     |
| `npm run build`   | 프로덕션 번들링 (`dist/` 생성) |
| `npm run preview` | `dist/`를 로컬 서버로 미리보기  |

---

## ✅ 전체 실행 흐름 요약

```bash
# 1. 프로젝트 폴더로 이동
cd my-vite-react-app

# 2. 의존성 설치 (최초 1회)
npm install

# 3. 개발 서버 실행
npm run dev

# 4. 브라우저 열기
http://localhost:5173

# 5. 단축키 사용 (dev 서버 실행 중 터미널에서 입력)
r + Enter  → 서버 재시작  
o + Enter  → 브라우저 열기  
q + Enter  → 서버 종료
```


