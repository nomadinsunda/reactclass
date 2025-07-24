
# ✅ DOM(Document Object Model)

---

## 1️⃣ DOM이란?

> DOM은 HTML, XML 문서를 브라우저가 **트리 구조의 객체로 메모리에 표현한 모델**입니다.

### 📌 정의적으로는:

* **"문서(document)"를 위한 API 인터페이스 모델**
* 문서의 각 구성 요소(태그, 속성, 텍스트 등)를 \*\*노드(node)\*\*로 구성된 **트리 형태의 객체 모델**로 나타냄
* DOM은 **W3C, WHATWG**에서 명세한 **표준화된 인터페이스 규격**

---

## 2️⃣ DOM의 내부 구조 (트리 모델)

HTML 문서는 다음과 같이 DOM 트리로 변환됩니다:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Demo</title>
  </head>
  <body>
    <div id="container">
      <h1>Welcome</h1>
      <section>
        <p>This is a paragraph.</p>
        <a href="#">Click here</a>
      </section>
    </div>
  </body>
</html>
```

### ▶️ 메모리 상의 DOM 트리

<img src="./images/domtree.svg" style="width:50%; height:auto;" />


---

## 3️⃣ DOM 구성 요소 (Node Interface 계층 구조)

DOM은 다양한 **노드 타입**으로 이루어져 있으며, 모두 `Node` 인터페이스를 상속합니다.

### 📦 주요 노드 타입

| 노드 인터페이스   | 설명                         |
| ---------- | -------------------------- |
| `Document` | 문서 전체, 최상위 노드              |
| `Element`  | HTML 요소 (`<div>`, `<p>` 등) |
| `Text`     | 텍스트 노드                     |
| `Attr`     | 속성 노드 (예: `id="greet"`)    |
| `Comment`  | 주석 노드                      |

> 즉, DOM은 단순한 트리가 아니라 **계층적 인터페이스를 가진 객체 트리**

---

## 4️⃣ DOM은 “실시간 반영되는 객체 모델”

### 특징

* HTML 문서가 로드되면 파서가 DOM으로 구성
* **JavaScript가 DOM을 직접 조작 가능**
* DOM의 변경은 **화면에 즉시 반영** (렌더링 파이프라인 재시작)

### 예:

```js
document.getElementById("greet").textContent = "Hello World";
```

* `Element` 노드를 가져와 `Text` 노드의 값을 변경
* 이로 인해 Paint → Composite 과정이 다시 발생함

---

## 5️⃣ DOM API의 설계 철학

DOM은 **프로그래밍 언어 중립적인 구조**를 갖습니다.
(JavaScript 뿐만 아니라 Java, C#, Python 등에서도 조작 가능)

### 대표적인 DOM API

| 메서드                      | 설명            |
| ------------------------ | ------------- |
| `getElementById(id)`     | ID 기반 탐색      |
| `querySelector(sel)`     | CSS 선택자 기반 탐색 |
| `createElement(tag)`     | 요소 생성         |
| `appendChild(node)`      | 자식 노드 추가      |
| `setAttribute(key, val)` | 속성 설정         |

---

## 6️⃣ DOM은 브라우저 엔진과 어떻게 연결되는가?

### 브라우저 렌더링 파이프라인에서의 DOM:

1. HTML 파싱 → **DOM Tree 생성**
2. CSS 파싱 → CSSOM
3. DOM + CSSOM → Render Tree
4. Layout → Paint → Composite

즉, DOM은 **렌더링의 핵심 데이터 구조**

---

## 7️⃣ DOM 변경이 렌더링에 미치는 영향

| DOM 변경 유형            | 발생하는 렌더링 작업    | 비용    |
| -------------------- | -------------- | ----- |
| 텍스트 변경               | Paint          | 낮음    |
| 스타일 변경 (color, font) | Paint          | 중간    |
| 크기 변경 (width 등)      | Reflow → Paint | 높음    |
| DOM 구조 변경            | Reflow → Paint | 가장 높음 |

---

## 8️⃣ Shadow DOM, Virtual DOM, Custom Elements (고급 확장)

### 📌 Shadow DOM

* 캡슐화된 DOM 영역
* 외부 CSS/JS에서 접근 불가

```js
const shadowRoot = element.attachShadow({ mode: 'open' });
shadowRoot.innerHTML = `<p>Shadow content</p>`;
```

### 📌 Virtual DOM

* 실제 DOM이 아님 → 메모리상의 가상 구조 (React 등에서 사용)
* diffing 알고리즘으로 실제 DOM 변경 최소화

### 📌 Custom Elements (Web Components)

* 사용자 정의 태그와 DOM 조작 가능

```js
class MyComponent extends HTMLElement {
  connectedCallback() {
    this.innerHTML = "<p>Hello</p>";
  }
}
customElements.define('my-comp', MyComponent);
```

---

## ✅ DOM의 특징 요약

| 항목     | 설명                                             |
| ------ | ---------------------------------------------- |
| 데이터 구조 | 트리(Tree) 구조                                    |
| 표준화    | W3C / WHATWG                                   |
| 접근 방식  | JavaScript 기반 객체 조작                            |
| 실시간 반영 | DOM 변경 → 렌더링 자동 반영                             |
| 관련 기술  | CSSOM, Shadow DOM, Virtual DOM, Web Components |

---

## 🎯 DOM에 강해지면 무엇이 좋아지나?

* 렌더링 최적화 가능 (Reflow/Paint 최소화)
* 애플리케이션 상태와 뷰의 일치 제어
* Web Components, Framework 내부 동작 이해
* 디버깅/성능 측정 능력 향상 (DevTools 활용)

---
<br/>


# ✅ DOM은 왜 필요한가?

---

## 🧭 1. **문서를 “프로그래밍 가능하게” 만들기 위해**

### 📌 HTML은 “정적 문서”

HTML은 원래 **정적인 문서 구조**만 표현할 수 있었습니다:

```html
<p>오늘의 날짜: 2024-12-31</p>
```

이렇게 고정된 콘텐츠는 바꾸거나 반응형으로 처리할 수 없습니다.
→ 우리는 이것을 JavaScript로 조작하고 싶어졌습니다:

```js
document.querySelector("p").textContent = `오늘의 날짜: ${new Date().toLocaleDateString()}`;
```

### 🔍 그런데 문제: JS는 HTML 문서 구조를 이해하지 못함

→ 브라우저가 HTML을 **JS가 다룰 수 있는 형태로 가공**해야 했고
그 결과물이 \*\*DOM (Document Object Model)\*\*입니다.

> **DOM은 HTML을 객체(트리)로 표현한 것** → 즉, “프로그래밍 가능한 HTML”

---

## 🧠 2. **브라우저와 사용자 간의 인터랙션을 처리하려면 DOM이 필수**

HTML은 클릭, 입력 등 사용자 상호작용을 고려하지 않았지만,
현대 웹은 **동적 상호작용**이 필수입니다:

* 버튼 클릭 시 새 콘텐츠 출력
* 입력 필드에 따라 실시간 검색 결과 표시
* 드래그 앤 드롭, 마우스 이벤트 등 UI 반응

👉 이 모든 인터랙션은 결국 **HTML 요소를 읽고/수정하고/추적**해야 함
👉 즉, **“문서에 대한 구조적 접근과 변경”을 위한 API가 필요**
→ 이 역할을 DOM이 맡습니다.

---

## 🏗️ 3. **UI를 동적으로 조작할 수 있는 구조 기반 제공**

DOM은 다음과 같은 작업을 가능하게 합니다:

| 기능     | 설명                            |
| ------ | ----------------------------- |
| 요소 생성  | `createElement("div")`        |
| 속성 수정  | `setAttribute("id", "box")`   |
| 콘텐츠 변경 | `textContent = "Hello"`       |
| 삭제     | `removeChild(node)`           |
| 스타일 변경 | `element.style.color = "red"` |

이 모든 작업은 실제로는 **HTML 파일을 바꾸는 것이 아니라**,
**브라우저 메모리에 있는 DOM 객체를 조작**하는 것입니다.
→ 결과적으로 화면이 **즉시 반영**됨

---

## 🕸️ 4. **웹 표준 인터페이스로의 통합 필요성**

과거에는 브라우저마다 JavaScript에서 HTML을 조작하는 방식이 제각각이었습니다.

* Netscape: `document.layers`
* Internet Explorer: `document.all`

➡ 혼란! 비표준! 유지보수 악몽!

🛠 그래서 **W3C가 DOM을 웹 표준 API로 정립**
→ 이제는 어떤 브라우저든 `document.getElementById("id")`는 동작해야 함

---

## 📦 5. **프론트엔드 생태계의 모든 기반**

DOM은 다음과 같은 기술들의 **전제 조건이자 기반**입니다:

| 기술                     | DOM 필요성                   |
| ---------------------- | ------------------------- |
| React / Vue / Svelte   | Virtual DOM, Template 바인딩 |
| CSS / JS 애니메이션         | DOM 노드 기반 트랜지션            |
| Accessibility (A11y)   | DOM 트리 구조에 기반한 스크린 리더     |
| 테스트 도구 (Jest, Cypress) | DOM 상태 확인 및 시뮬레이션         |
| SEO 크롤러                | DOM을 기준으로 콘텐츠 읽음          |

---

## ✅ 요약: DOM이 필요한 이유 5가지

| 이유                | 설명                            |
| ----------------- | ----------------------------- |
| HTML을 JS로 조작하기 위해 | HTML을 객체로 만들어야 함              |
| 사용자 인터랙션 처리       | 클릭, 입력 등 모든 이벤트는 DOM을 통해      |
| 화면을 동적으로 바꾸기 위해   | 실시간 콘텐츠 조작이 가능                |
| 웹 표준 통합           | 브라우저 호환성과 유지보수                |
| 프론트엔드 기술 기반       | 현대 JS 프레임워크와 도구는 모두 DOM 위에 존재 |

---

## 🧠 핵심 한 줄 요약

> **DOM은 정적인 HTML을 "프로그래밍 가능한 웹 UI"로 만드는 연결 고리다.**



