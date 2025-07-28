
## ✅ 1. 인터페이스란?

### 일반 의미:

> 어떤 시스템이 외부와 **소통하는 방식** 또는 **약속된 사용법**을 말함.

---

## ✅ 2. Web API는 브라우저가 제공하지만, **JS 코드로 호출 가능해야 한다**

* 브라우저는 내부적으로 복잡한 C++ 기반 엔진(Blink 등)으로 DOM을 구현합니다.
* 하지만 개발자는 JavaScript로 이렇게 쉽게 호출하죠:

```js
const el = document.createElement("div");
```

### 🧠 이게 가능하려면?

→ 브라우저는 `document` 객체를 **JavaScript에서 접근 가능하게 만들고**,
그 객체에 `.createElement()`라는 함수를 붙여줘야 함.

→ 이를 JavaScript 입장에서 보면:

> document는 브라우저가 제공하는 Document 인터페이스의 인스턴스이며,
> 이 인터페이스는 DOM 표준에 따라 다양한 메서드(createElement, querySelector 등)를 제공합니다.
> JavaScript에서는 해당 객체를 통해 DOM을 제어할 수 있도록 접근 권한이 열려 있습니다.



---

## ✅ 3. JavaScript 인터페이스란?

> 브라우저가 내부적으로 만든 객체(document, window 등)에 대해
> **JavaScript에서 접근하고 조작할 수 있도록 제공되는 메서드와 속성의 집합**입니다.

즉,

* `document.createElement()`
* `window.alert()`
* `navigator.geolocation.getCurrentPosition()`

이 모든 것들은 **브라우저 내부 객체**이지만,
**JavaScript 코드에서 접근 가능**하므로
→ **JavaScript 인터페이스**라고 부릅니다.

---

## ✅ 공식 명칭에서는 이렇게 설명합니다

### \[W3C / WHATWG 표준 문서에서는]:

> *A JavaScript interface is a set of properties and methods, implemented by the host environment (e.g. a browser), and exposed to JavaScript via objects like `window`, `document`, etc.*

> 즉, **호스트 환경이 구현하고**, **자바스크립트에서 사용할 수 있도록 제공한 API**를 JavaScript 인터페이스라고 부릅니다.

---

## 📌 구분 예시

| 항목    | JavaScript 내장 객체          | JavaScript 인터페이스 (Web API)                    |
| ----- | ------------------------- | --------------------------------------------- |
| 정의 주체 | ECMAScript (JS 자체)        | 브라우저 (Web Platform)                           |
| 예시    | `Math`, `Date`, `Promise` | `document`, `window`, `fetch`, `localStorage` |
| 실행 환경 | JS 엔진이면 어디서든              | 브라우저 or 호환 환경에서만                              |
| 타입    | JS 표준 타입                  | 브라우저 확장 객체                                    |
| 사용 목적 | 계산, 날짜 처리 등               | DOM 조작, 네트워크 요청 등                             |

---

## ✅ 한 줄 요약

> **JavaScript 인터페이스란, 브라우저가 자바스크립트로 접근할 수 있도록 제공하는 객체들과 그 메서드들**입니다.
> → `document`, `window`, `navigator`, `fetch` 등이 대표적 예시입니다.

