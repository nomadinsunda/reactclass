

## 1. BOM이란 무엇인가? (What is BOM?) 🧱

### 1-1. BOM의 정의

**BOM(Browser Object Model)**은 말 그대로 “브라우저를 객체로 표현한 모델”입니다.

* DOM: **문서(document)를 객체로 표현** → `<html>`, `<body>`, `<div>` 등
* BOM: **브라우저 환경 전체를 객체로 표현** → 창(window), 주소창(location), 히스토리(history), 화면(screen), 네비게이터(navigator) 등

즉, BOM은 다음과 같은 것들을 제어하는 API 모음입니다.

* 현재 탭/창 자체 (`window`)
* 주소창 URL (`location`)
* 뒤로/앞으로 히스토리 (`history`)
* 브라우저/OS 정보 (`navigator`)
* 화면 해상도/색상 등 디스플레이 정보 (`screen`)
* 팝업, 타이머, 프레임 등

👉 중요한 포인트
BOM은 **어떤 표준 스펙 하나로만 정의된 것이 아니라**, 여러 Web 표준(HTML, WHATWG, WebIDL 등)에 흩어져 있습니다. “BOM이라는 이름의 공식 스펙”이 있는 것이 아니라 **관행적으로 부르는 개념**입니다.

---

## 2. BOM의 루트: `window` 객체 🌐

### 2-1. `window`는 “글로벌 컨텍스트” 그 자체

브라우저에서 자바스크립트가 실행될 때, 최상위 글로벌 객체는 `window`입니다.

```js
console.log(window === globalThis); // 대부분의 브라우저에서 true
console.log(window === self);       // true
console.log(this === window);       // 전역 컨텍스트에서 true
```

`window`는 크게 두 가지 역할을 동시에 수행합니다.

1. **글로벌 네임스페이스**

   * 전역 변수, 전역 함수가 모두 `window`의 프로퍼티로 붙습니다.

   ```js
   var x = 10;
   function foo() {}
   console.log(window.x);   // 10
   console.log(window.foo); // ƒ foo() {}
   ```

2. **브라우저 창/탭 객체 (BOM 루트)**

   * `window.location`, `window.history`, `window.screen`, `window.navigator` 등 **BOM 하위 객체의 루트**입니다.

### 2-2. `window`와 DOM의 연결

DOM의 루트 `document`도 `window` 아래에 있습니다.

```js
console.log(window.document); // DOM Document 객체
console.log(document === window.document); // true
```

즉:

* **DOM**: `window.document` 아래의 트리 구조
* **BOM**: `window` 아래의 다양한 브라우저 관련 객체들의 집합

---

## 3. `window`의 핵심 기능들 ⏰🗣️

### 3-1. 타이머 API (`setTimeout`, `setInterval`)

BOM의 대표적인 기능이 **타이머**입니다.

```js
const id = window.setTimeout(() => {
  console.log("1초 후 실행");
}, 1000);

window.clearTimeout(id);
```

```js
const intervalId = setInterval(() => {
  console.log("1초마다 실행");
}, 1000);

setTimeout(() => clearInterval(intervalId), 5000);
```

* 이벤트 루프, 태스크 큐, Web API와 함께 동작하는 비동기의 핵심.
* 타이머 자체의 구현은 브라우저(Web API) 영역이며, JS 엔진은 콜백을 큐에서 꺼내 실행만 합니다.

### 3-2. 다이얼로그 API (`alert`, `confirm`, `prompt`)

```js
alert("경고 메시지");
const ok = confirm("정말 삭제하시겠습니까?");
const name = prompt("이름을 입력하세요");
```

* **동기적(Synchronous)**으로 실행 → UI 쓰레드를 블로킹합니다.
* UX 관점에서 지양되는 추세이지만, 디버깅/간단한 데모에는 여전히 사용됩니다.
* 브라우저마다 스타일/동작이 다를 수 있어서 실서비스에서는 커스텀 모달로 대체하는 편입니다.

### 3-3. 창/탭 제어 (`open`, `close`, `moveTo`, `resizeTo`)

```js
const child = window.open("https://example.com", "_blank", "width=600,height=400");
// ...
child.close();
```

* 팝업 차단 정책, 보안 정책(CSP) 때문에 실제 동작은 브라우저/환경에 따라 제한될 수 있습니다.
* 보안/UX 측면에서 제한이 많기 때문에 **실무에서는 최소한으로만 사용**하는 영역입니다.

---

## 4. `location` 객체: 주소창 제어 🧭

### 4-1. `window.location`의 구조

```js
console.log(location.href);   // 전체 URL
console.log(location.protocol); // "https:"
console.log(location.host);     // "example.com:443"
console.log(location.hostname); // "example.com"
console.log(location.port);     // "443" 또는 ""(기본 포트)
console.log(location.pathname); // "/users/123"
console.log(location.search);   // "?page=2&size=10"
console.log(location.hash);     // "#section1"
```

### 4-2. 페이지 이동 API

```js
// 현재 페이지 리로드
location.reload();

// 현재 페이지를 새 URL로 교체(뒤로가기 히스토리 남음)
location.href = "/login";

// 뒤로가기 히스토리를 남기지 않고 새로 로드
location.replace("/login");
```

* `href` 변경: 브라우저가 **완전히 새로운 요청**을 서버로 보냄.
* `replace`: 현재 히스토리 항목을 덮어써서 “뒤로가기”로 돌아갈 수 없게 만듦.

### 4-3. SPA에서의 활용

* **해시 라우팅**: `location.hash`를 이용해 라우팅 (`#/users/1`)
* **History API**와 함께 사용하여 주소를 제어 (뒤에서 설명)

---

## 5. `history` 객체: 브라우저 히스토리 제어 📜

### 5-1. 기본 동작

```js
history.back();   // 뒤로가기 1단계
history.forward(); // 앞으로가기 1단계
history.go(-2);   // 두 단계 뒤로
history.go(0);    // 새로고침과 비슷
```

* 이 메서드들은 **사용자의 세션 히스토리 스택**을 제어합니다.
* 실제 브라우저 내비게이션 버튼과 동일한 동작을 프로그래밍적으로 수행.

### 5-2. History API (`pushState`, `replaceState`)

SPA에서 **페이지 전환 없이 URL만 변경**하고 싶을 때 사용하는 중요한 API입니다.

```js
// 새 상태 push (뒤로가기 히스토리에 쌓임)
history.pushState({ page: 2 }, "", "/list?page=2");

// 현재 상태 교체
history.replaceState({ page: 3 }, "", "/list?page=3");

// 사용자가 뒤로가기/앞으로가기 시 발생
window.addEventListener("popstate", (event) => {
  console.log(event.state); // { page: 2 } 등
});
```

특징:

* **전체 페이지 리로드 없이 URL만 변경**
* React Router, Vue Router 등의 SPA 라우터가 내부적으로 사용하는 핵심 API
* 브라우저 히스토리를 직접 다루는 것이므로, UX를 고려하지 않으면 “뒤로가기 미친 앱”이 될 수 있습니다.

---

## 6. `navigator` 객체: 브라우저/환경 정보 🧭💻

`navigator`는 **브라우저의 메타 정보+환경 정보를 모아놓은 객체**입니다.

### 6-1. 대표 프로퍼티

```js
console.log(navigator.userAgent);   // 브라우저, OS 정보 문자열 (점점 덜 신뢰됨)
console.log(navigator.language);    // "ko-KR"
console.log(navigator.platform);    // "Win32" 등
console.log(navigator.onLine);      // true/false (네트워크 연결 여부 대략)
```

### 6-2. 기능성 API 예시

근래에는 여러 기능이 `navigator` 하위에 붙습니다.

* `navigator.geolocation`

  ```js
  navigator.geolocation.getCurrentPosition(
    (pos) => {
      console.log(pos.coords.latitude, pos.coords.longitude);
    },
    (err) => {
      console.error(err);
    }
  );
  ```
* `navigator.clipboard` (HTTPS + 권한 필요)

  ```js
  navigator.clipboard.writeText("복사할 텍스트");
  ```
* `navigator.serviceWorker` (PWA)

  ```js
  navigator.serviceWorker.register("/sw.js");
  ```

👉 보안상 민감한 기능일수록 **HTTPS + 사용자 권한 동의**가 필수입니다.

---

## 7. `screen` 객체: 디스플레이 정보 📺

`screen`은 **사용자 디스플레이(모니터/스크린)에 대한 정보**를 제공합니다.

```js
console.log(screen.width, screen.height);        // 전체 화면 해상도
console.log(screen.availWidth, screen.availHeight); // 작업표시줄 제외 영역
console.log(screen.colorDepth);                  // 색 깊이 (일반적으로 24)
```

* 예전에는 새 창을 열 때 위치와 크기를 계산하거나, 반응형 UI 구현에 활용.
* 현대 프론트엔드에서는 **반응형은 CSS(media query)**, 뷰포트 크기는 `window.innerWidth/innerHeight` 또는 `ResizeObserver` 등으로 처리 → `screen` 직접 사용 빈도는 줄어든 편입니다.

---

## 8. 프레임/윈도우 간 통신: `frames`, `parent`, `opener` 🧩

### 8-1. 윈도우 계층 구조

브라우저에서는 여러 "창/프레임" 사이에 계층 구조가 존재합니다.

* `window` : 현재 창/프레임
* `window.parent` : 현재 프레임을 포함하는 상위
* `window.top` : 최상위 프레임
* `window.opener` : 나를 연 창 (`window.open`으로 열린 경우)

```js
// iframe 안에서
console.log(window.parent === window.top); // 최상위와 부모 비교
```

### 8-2. `postMessage()`를 이용한 크로스 문서 메시지

서로 다른 도메인 간에는 보안상 직접 접근이 불가능합니다.
이때 **안전하게 메시지를 주고받기 위한 API**가 `postMessage`입니다.

```js
// 자식 프레임 혹은 다른 윈도우에 메시지 전송
otherWindow.postMessage({ type: "PING" }, "https://example.com");

// 수신 측
window.addEventListener("message", (event) => {
  if (event.origin !== "https://trusteddomain.com") return; // 보안 필수
  console.log(event.data);
});
```

* CORS와는 별개로, **window 간 통신용 보안 채널**이라고 보면 됩니다.
* iframe 기반 위젯, 외부 결제 창, OAuth 로그인 팝업 등에서 많이 사용됩니다.

---

## 9. BOM과 보안·제한 사항 🔒

### 9-1. 동일 출처 정책(Same-Origin Policy)

BOM의 많은 기능은 **동일 출처 제한**을 받습니다.

* `window.opener.document`에 자유롭게 접근? → 동일 origin이 아니면 `SecurityError`
* iframe 간 DOM 접근? → origin이 다르면 차단
* `localStorage`, `sessionStorage` 등도 origin 단위로 격리

이유:

* 한 사이트가 다른 사이트의 세션/DOM을 훔쳐보지 못하도록 하기 위해
* 브라우저는 BOM을 통해 **사이트 간 강력한 격리**를 수행합니다.

### 9-2. 팝업/창 관련 제한

* `window.open`은 **사용자 제스처(클릭 등)**에 의해 호출된 경우에만 허용
* `window.moveTo`, `resizeTo` 등은 대부분의 브라우저에서 제한 또는 무시
* `alert`, `confirm`, `prompt`는 UX 문제로 사용을 권장하지 않으며, 일부 브라우저는 과도한 사용 시 자동 차단

---

## 10. BOM vs DOM, 그리고 JS 엔진과의 관계 ⚙️

### 10-1. 세 층위 구조 정리

1. **JS 엔진 (V8, SpiderMonkey, JavaScriptCore 등)**

   * ECMAScript 스펙 구현
   * 순수 언어 기능: Number, String, Array, Map, Promise, Class, async/await...

2. **BOM (브라우저 런타임 환경)**

   * `window`, `location`, `history`, `navigator`, `screen`, `fetch`, `XMLHttpRequest`, 타이머...
   * JS 엔진이 제공하는 것이 아니라 **브라우저가 엔진 위에 주입하는 객체들**

3. **DOM (Document Object Model)**

   * `document`, `Element`, `Node`, `Event`, `HTMLDivElement`...
   * HTML 문서를 객체 트리로 표현하는 계층

👉 포인트
자바스크립트 파일을 Node.js에서 실행하면 `window`, `document`가 없는 이유가 바로 이것입니다.
Node.js는 브라우저가 아니므로 DOM/BOM을 제공하지 않고, 대신 `global`, `process`, `require` 등을 제공합니다.

---

## 11. BOM의 현대적 의미와 활용 전략 🚀

### 11-1. 왜 아직도 BOM을 이해해야 할까?

* SPA 라우팅 (`history.pushState`, `location`)
* OAuth/PWA/결제 연동 (`window.open`, `postMessage`, `navigator.serviceWorker`, `navigator.clipboard`)
* 비동기 모델 이해 (`setTimeout`, `setInterval` → 이벤트 루프)
* 브라우저 간 호환성/제한사항 이해

현대 프레임워크(React, Vue, Angular)를 쓰더라도, 결국 **그 아래에서 돌아가는 런타임은 BOM**입니다.

### 11-2. 실무에서의 베스트 프랙티스

* **전역 `window` 접근 최소화**

  * SSR/Node 환경에서 `window`가 없는 문제 → `typeof window !== "undefined"` 체크
  * 프레임워크 훅(`useEffect`) 안에서만 접근하는 패턴 등

* **히스토리와 URL은 라우터로 캡슐화**

  * 직접 `history.pushState`를 호출하기보다는 React Router, Vue Router 등 라우터 라이브러리 사용
  * 단, 내부 동작을 이해하고 있어야 디버깅/커스터마이징에 강해집니다.

* **보안/권한/정책 이해**

  * `postMessage` 사용 시 `origin` 검증 필수
  * `navigator` 기반 API는 HTTPS + 퍼미션 체크 필수

---

## 12. 정리 🧩

한 줄로 요약하면:

> **BOM은 “브라우저라는 운영체제 위에서 자바스크립트가 실행되도록 만들어주는 시스템 콜 인터페이스”와 같다.**

* `window`는 글로벌 컨텍스트이자 BOM의 루트
* `location`은 주소창, `history`는 방문 기록, `navigator`는 브라우저/환경 정보
* `screen`은 디스플레이 정보, 프레임 관련 API는 창/탭/iframe 간 관계를 표현
* 타이머·다이얼로그·팝업·postMessage는 브라우저 UI/비동기를 다루는 핵심 도구
* DOM, JS 엔진, BOM의 경계를 이해하면 **이벤트 루프, SPA 라우팅, 보안, 호환성**까지 자연스럽게 연결됩니다.

