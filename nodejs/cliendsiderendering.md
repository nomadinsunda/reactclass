## ✅ CSR이란?

> \*\*CSR(Client-Side Rendering)\*\*은 브라우저가 HTML을 받은 뒤,
> 자바스크립트가 실행되면서 \*\*화면을 동적으로 구성(rendering)\*\*하는 방식입니다.

즉, \*\*"화면은 서버가 아니라 브라우저가 만든다."\*\*는 것이 핵심입니다.

---

## ✅ CSR의 전체 동작 흐름

1. 사용자가 브라우저에서 URL에 접속하면
2. 서버는 빈 HTML(기본 구조)과 JS 파일(bundle.js)을 내려줍니다.
3. 브라우저는 JS를 다운받고 실행합니다.
4. React 같은 프레임워크가 자바스크립트로 DOM을 생성하여 실제 화면을 구성합니다.

---

## ✅ 실제 CSR 예제

### 📁 프로젝트 구조 (create-react-app 기반)

```
my-csr-app/
├── public/
│   └── index.html      ← 빈 HTML (root만 있음)
├── src/
│   ├── App.js          ← 메인 컴포넌트
│   └── index.js        ← 엔트리 포인트 (ReactDOM.render)
├── package.json
```

---

### 📄 `public/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>CSR Example</title>
  </head>
  <body>
    <div id="root"></div> <!-- 브라우저가 여기에 React 컴포넌트를 끼워 넣음 -->
  </body>
</html>
```

---

### 📄 `src/App.js`

```jsx
import React from 'react';

function App() {
  return (
    <div>
      <h1>안녕하세요, CSR 예제입니다</h1>
      <p>이 내용은 브라우저에서 JavaScript로 렌더링되었습니다.</p>
    </div>
  );
}

export default App;
```

---

### 📄 `src/index.js`

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// 이 코드는 브라우저에서 실행되며 <div id="root">에 컴포넌트를 렌더링함
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

---

## ✅ 실행 순서 시각화

```
브라우저 요청 → 서버
 ↓
[서버 응답] index.html + bundle.js
 ↓
브라우저: JS 실행
 ↓
ReactDOM.render() → App 컴포넌트 생성
 ↓
App → <h1>안녕하세요</h1> 등 HTML 생성
 ↓
DOM에 주입 (document.getElementById('root'))
 ↓
최종 화면 완성
```

* 이 때, 브라우저가 JS를 다운받기 전까지는 **화면이 아무것도 없음**
* JS를 꺼놓은 브라우저에서는 아무것도 보이지 않음 (❌ SEO)

---

## ✅ 브라우저에서 보이는 HTML

초기엔 이렇게 비어있습니다:

```html
<div id="root"></div>
```

JS가 실행되면 이렇게 변합니다:

```html
<div id="root">
  <div>
    <h1>안녕하세요, CSR 예제입니다</h1>
    <p>이 내용은 브라우저에서 JavaScript로 렌더링되었습니다.</p>
  </div>
</div>
```

즉, **HTML은 브라우저가 JS를 실행한 뒤 동적으로 생성**됩니다.

---

## ✅ CSR의 장점과 단점

| 장점                            | 단점                               |
| ----------------------------- | -------------------------------- |
| 💡 빠른 개발                      | 🕐 첫 화면이 늦게 나타남 (JS 로딩 기다려야 함)   |
| 💡 사용자 경험에 강함 (SPA 구현)        | ❌ JS 꺼진 환경에선 화면 없음               |
| 💡 서버 부담 적음                   | ❌ 검색 엔진이 콘텐츠를 못 읽을 수 있음 (SEO 취약) |
| 💡 페이지 전환 빠름 (React Router 등) | ❌ 초기 렌더링에 JS 의존                  |

---

## ✅ CSR이 적합한 경우

* 내부 시스템 (관리자 대시보드, 통계 페이지 등)
* 로그인 후 사용자 맞춤형 콘텐츠
* 검색 엔진 노출이 중요하지 않은 앱
* 앱처럼 동작하는 고도화된 UI 필요

---

## ✅ 요약

| 항목      | 설명                               |
| ------- | -------------------------------- |
| 정의      | 화면을 브라우저가 자바스크립트로 그리는 방식         |
| 핵심 기술   | React, Vue, Angular 등 SPA 프레임워크  |
| 대표 구현   | `ReactDOM.render(<App />, root)` |
| 초기 화면   | 빈 HTML + JS                      |
| SEO 친화도 | 낮음                               |
| 속도      | 초기 느림, 이후 빠름                     |

---

참고사항:

* CSR에서 SEO를 보완하는 기술 (예: Prerendering, CSR + meta 태그 보정)
* React Router 적용 예제 (CSR 기반 SPA 구현)
* CSR 앱의 빌드 결과 분석 (bundle.js 안에 어떤 게 들어가는지)

