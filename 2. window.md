`window`는 단순히 `alert()`을 호출하는 객체가 아닙니다. 브라우저, 자바스크립트, 렌더링 엔진, 보안 모델 등과 **직접 연결된 핵심 인터페이스**입니다.

---

# ✅ `window`

---

## 🧭 1. `window` 객체란?

> `window`는 브라우저 탭 하나를 대표하는 **전역 컨텍스트(Global Context)** 객체이며,
> 자바스크립트가 웹 브라우저와 상호작용할 수 있도록 브라우저에서 제공하는 **최상위 전역 객체**입니다.

### 브라우저 기반 JavaScript의 진입점이자 루트 환경

```js
console.log(window === globalThis); // true (브라우저에서)
console.log(window === self);       // true (브라우저 메인 스레드에서)
```

---

## 🧬 2. 구조적 위치

```
window
 ├── document        → 현재 로드된 HTML 문서
 ├── location        → 현재 페이지 URL 정보 (window.location.href 등)
 ├── navigator       → 사용자 브라우저 정보 (User Agent, 브라우저 종류 등)
 ├── history         → 브라우저 방문 히스토리 제어
 ├── screen          → 디스플레이 정보 (해상도 등)
 ├── localStorage    → 탭 간 공유 저장소
 ├── sessionStorage  → 탭 내 일시 저장소
 ├── console         → 콘솔 API
 ├── alert, prompt   → 기본 대화 상자
 ├── setTimeout      → 타이머
 └── [기타 수백 개의 API]
```

---

## 🧠 3. 핵심적인 역할 5가지

| 역할                     | 설명                                                          |
| ---------------------- | ----------------------------------------------------------- |
| ① 전역 객체(Global Object) | 브라우저에서 JS가 실행되는 최상위 스코프                                     |
| ② 브라우저 인터페이스           | location, history, screen 등 브라우저 기능 접근                      |
| ③ 이벤트 핸들러 컨테이너         | `onload`, `onresize`, `onscroll`, `onbeforeunload` 등        |
| ④ 글로벌 변수 저장소           | `var`, `function`으로 선언한 전역 식별자는 window에 바인딩                 |
| ⑤ 타이머 및 Web API 접점     | `setTimeout`, `requestAnimationFrame`, `fetch` 등 비동기 API 포함 |

---

## 📌 4. 전역 객체로서의 특징

```js
var a = 123;
console.log(window.a); // 123

let b = 456;
console.log(window.b); // undefined (let은 window에 등록되지 않음)
```

* `var`, `function`은 `window` 객체에 등록됩니다.
* `let`, `const`, `class`는 전역 변수이지만 **window 속성이 아님** (block scoped)

---

## 🔎 5. 주요 프로퍼티

| 프로퍼티                              | 설명                     |
| --------------------------------- | ---------------------- |
| `document`                        | 현재 로드된 DOM 트리          |
| `location`                        | 현재 주소, 이동, 리다이렉트       |
| `history`                         | 브라우저 히스토리 컨트롤 (뒤로가기 등) |
| `navigator`                       | 브라우저 종류/기기/버전 정보       |
| `screen`                          | 모니터 해상도, 색상 등 정보       |
| `localStorage`, `sessionStorage`  | 클라이언트 저장소              |
| `frames`, `self`, `top`, `parent` | 창 간 접근 (iframe 등)      |
| `innerWidth`, `innerHeight`       | 뷰포트 크기                 |
| `devicePixelRatio`                | 화면 해상도 배율 (HiDPI 대응)   |

---

## ⚙️ 6. 주요 메서드

### 📍 사용자 인터페이스

| 메서드            | 설명             |
| -------------- | -------------- |
| `alert(msg)`   | 경고창 표시         |
| `prompt(msg)`  | 입력창 표시         |
| `confirm(msg)` | 확인창 표시 (확인/취소) |

### 📍 타이머

| 메서드                         | 설명                        |
| --------------------------- | ------------------------- |
| `setTimeout(fn, ms)`        | 일정 시간 후 함수 실행             |
| `setInterval(fn, ms)`       | 일정 간격마다 함수 실행             |
| `requestAnimationFrame(fn)` | 다음 리페인트 시점에 실행 (VSync 기반) |

### 📍 윈도우 제어

| 메서드              | 설명                 |
| ---------------- | ------------------ |
| `open(url)`      | 새 창 또는 탭 열기        |
| `close()`        | 현재 창 닫기 (조건 제한 있음) |
| `print()`        | 프린트 다이얼로그 띄우기      |
| `scrollTo(x, y)` | 문서 스크롤 이동          |

---

## 🧩 7. `window`는 실제로 여러 "스레드/컨텍스트"를 가질 수 있다

| 환경             | 설명                               |
| -------------- | -------------------------------- |
| 메인 스레드         | 일반적인 브라우저 JS 실행                  |
| iframe         | 각각 독립적인 window 객체                |
| Web Worker     | 별도의 전역 객체 (`self`) → `window` 없음 |
| Service Worker | `self` 존재, `window`는 없음          |

---

## 🔐 8. 보안과 window (Same-Origin Policy)

* `window.parent`, `window.top`, `window.frames[0]` 등으로 **다른 프레임 접근 가능**
* 하지만 **다른 도메인**이면 JavaScript 접근은 **막힘 (CORS, SOP 제한)**

```js
// A.com에서 B.com iframe을 접근하려 하면?
window.frames[0].document // ❌ SecurityError 발생
```

---

## 🧠 9. window vs globalThis vs self

| 객체           | 브라우저         | Web Worker | 의미                  |
| ------------ | ------------ | ---------- | ------------------- |
| `window`     | ✅ 있음         | ❌ 없음       | 브라우저 전역             |
| `self`       | ✅ window와 동일 | ✅ 전역 객체    | 범용 전역 식별자           |
| `globalThis` | ✅            | ✅          | ECMAScript 표준 전역 객체 |

> `globalThis`는 `window`, `self`, `global`을 추상화한 **ECMAScript 표준 전역 객체**입니다.

---

## ✅ 요약

| 항목        | 설명                                                     |
| --------- | ------------------------------------------------------ |
| 정체        | 브라우저의 전역 객체 (탭 단위)                                     |
| 포함 관계     | `window.document`, `window.location`, `window.alert` 등 |
| 역할        | 브라우저 기능 + DOM 진입점 + 비동기 API 노출                         |
| 접근 방식     | 대부분의 전역 식별자는 window에 귀속됨 (`window.alert()`, `alert()`) |
| 다른 환경과 차이 | Node.js에서는 `global`, Web Worker에서는 `self`              |

---



## ✅ window와 자바스크립트 관계:

> **`window` 객체는 자바스크립트가 웹 브라우저 환경의 리소스(DOM, 위치 정보, 저장소, UI 기능 등)에 접근하고 제어할 수 있도록 브라우저가 제공하는 API(인터페이스)이다.**

---

## 🔍 더 구체적으로 말하면…

* **JavaScript는 원래 브라우저와 무관한 언어**입니다.
  즉, JS 자체만으로는 DOM도 없고, alert도 없고, 화면도 없고, 쿠키도 없어요.

* 하지만 브라우저는 JavaScript를 실행할 수 있는 \*\*실행 환경(runtime environment)\*\*을 제공합니다.
  그 환경이 바로 `window` 객체를 포함한 브라우저 API입니다.

---

## 🧠 JavaScript vs 브라우저의 역할

| 항목     | JavaScript 자체       | 브라우저(window 등)                 |
| ------ | ------------------- | ------------------------------ |
| 연산     | 숫자 계산, 함수 호출, 반복문 등 | ❌                              |
| DOM 조작 | ❌                   | ✅ document.getElementById      |
| UI 제어  | ❌                   | ✅ alert, confirm, prompt       |
| 네트워크   | ❌                   | ✅ fetch, XMLHttpRequest        |
| 시간     | ❌                   | ✅ setTimeout, setInterval      |
| 저장소    | ❌                   | ✅ localStorage, sessionStorage |

---

## 📦 결론적으로:

### ✅ "JavaScript는 문법과 계산의 도구"

### ✅ "브라우저(`window`)는 그 JS에게 현실 세계(화면, 주소, 쿠키, 이벤트 등)를 열어주는 창구"

---

## ✅ 한 줄 요약:

> **`window`는 자바스크립트가 브라우저 세계를 "터치"할 수 있게 해주는 브라우저 측 API 객체이다.**



