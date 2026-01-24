## 1. 큰 그림부터: “빌드 파이프라인” 생각하기 🧠

Vite + React + Tailwind를 쓰면, 화면에 CSS가 나오기까지 내부에 이런 **파이프라인**이 있습니다:

1. **Vite**

   * 개발 서버(Dev Server) + 번들러(ESBuild/Rollup 역할).
   * `.jsx`, `.js`, `.css` 파일을 읽어서 모듈 그래프를 구성하고, HMR(핫 리로드)도 처리.

2. **PostCSS**

   * CSS를 **JavaScript AST로 파싱해서 변환**하는 빌드 도구.
   * 혼자 일 안 하고 **플러그인**들(Tailwind, Autoprefixer 등)을 줄줄이 실행.

3. **Tailwind CSS (PostCSS 플러그인)**

   * `@tailwind base; @tailwind components; @tailwind utilities;` 같은 지시어를 보고
     → 실제 유틸리티 클래스 CSS를 생성.
   * `content` 옵션에 명시된 파일들(`./src/**/*.jsx`)을 스캔해서 **실제로 쓰는 클래스만** CSS 결과에 포함.

4. **Autoprefixer (PostCSS 플러그인)**

   * Tailwind를 포함한 **최종 CSS**를 받아서
     → 브라우저 지원 범위에 따라 `-webkit-`, `-ms-` 같은 접두사를 자동으로 붙여줌.

**Vite 입장**에서는:

* “나는 CSS 파일을 PostCSS에게 한 번 맡긴다”
* “PostCSS야, 네가 알아서 Tailwind + Autoprefixer 돌리고 결과만 나에게 줘”
  이렇게 위임하는 구조입니다.

이 연결을 실제로 구현하는 파일이 바로:

* `postcss.config.js` (PostCSS 설정)
* `tailwind.config.js` (Tailwind 설정)
* `src/index.css` 또는 `src/main.css` (Tailwind 지시어)
* `src/main.jsx`에서 CSS import

입니다.

---

## 2. Vite + React + Tailwind 기본 세팅 흐름 🏗️

자바스크립트 기반 React 앱 기준으로 정리하겠습니다.

### 2-1. Vite + React 앱 생성

```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
```

이 시점에서 구조 예시:

```text
my-app/
  index.html
  package.json
  vite.con항:

* `content` 옵션이 어떻게 작동하는지 (JIT, purge 개념)
* Tailwind + DaisyUI + 기타 PostCSS 플러그인(예: `postcss-nesting`)을 같이 쓰는 고급 세팅


