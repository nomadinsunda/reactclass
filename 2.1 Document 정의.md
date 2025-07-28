# ✅ DOM의 "D = Document"는 왜 필요한가?

---

## 📖 1. HTML은 “문서(Document)”로 시작되었기 때문

HTML은 원래 **정적인 하이퍼텍스트 문서(HTML Document)** 를 표현하기 위해 만들어졌습니다.
웹의 초기 철학은 웹 페이지를 **문서처럼 구성하고 연결하는 것**이었습니다.

### 예:

```html
<!DOCTYPE html>
<html>
  <head><title>문서 제목</title></head>
  <body>
    <p>이것은 문단입니다.</p>
  </body>
</html>
```

이처럼 웹의 기본 단위는 `문서(Document)`이며, 브라우저가 이것을 해석하고 표시합니다.
→ 즉, 브라우저가 다루는 **주체이자 최상위 대상이 “문서”** 입니다.

---

## 🧠 2. DOM은 문서 전체를 추상화한 “객체 모델”이기 때문에

* DOM은 HTML/XML을 읽어들인 뒤,
* 이를 트리 형태의 **객체(Object) 구조로 표현**한 모델입니다.
* 이 모델이 다루는 최상위 대상이 **문서 전체(document)** 이고,
* 이 모델은 그 문서를 조작할 수 있게 해주는 **API 인터페이스이자 자료 구조**입니다.

### 공식 정의 (W3C):

> The Document Object Model is a platform- and language-neutral interface that allows programs and scripts to dynamically access and update the content, structure, and style of **documents**.

즉,

> DOM = “문서를 조작할 수 있는 객체 모델”

---

## 📎 용어 분해: **Document Object Model**

| 용어           | 의미                              |
| ------------ | ------------------------------- |
| **Document** | HTML/XML과 같은 문서 (웹 페이지 전체)      |
| **Object**   | JS 등 프로그래밍 언어에서 접근 가능한 구조화된 데이터 |
| **Model**    | 문서 구조를 추상화한 데이터 표현 방식 (트리 구조 등) |

---

## 🧬 3. 기술적으로도 "document"는 DOM 트리의 루트

DOM 트리의 최상단 루트 객체는 항상 **`document`** 입니다.

```js
console.log(document); // HTML 전체를 나타내는 DOM 최상위 객체
```

그리고 이 `document` 객체를 통해 모든 DOM 요소에 접근합니다:

```js
document.querySelector("p")
document.getElementById("main")
document.createElement("div")
```

→ 이처럼 **“문서”를 다룬다(document-centric)** 는 개념이 기술의 중심이기 때문에, 이름도 자연스럽게 `Document Object Model`이 된 것입니다.

---

## 📜 4. XML 기반 문서 모델링에서의 역사적 배경

* DOM은 HTML만이 아니라 **XML의 문서 구조**를 다루기 위해 만들어졌습니다.
* XML은 어떤 종류의 데이터도 **document** 로 표현합니다.

  * 예: XHTML, SVG, RSS, Atom 등

따라서 DOM은 “웹 페이지”뿐 아니라 **구조화된 모든 문서형 데이터**에 공통으로 적용할 수 있어야 했고,
이러한 일반성을 표현하기 위해 **"Document"라는 용어를 썼습니다.**

---

## ✅ 요약: 왜 Document인가?

| 이유           | 설명                             |
| ------------ | ------------------------------ |
| 문서가 웹의 기본 단위 | HTML/XML은 모두 문서(document)      |
| DOM의 조작 대상   | 문서 전체를 구성하는 노드 트리              |
| 기술 구조        | 루트가 항상 `document` 객체           |
| 일반화 가능성      | XML 등 모든 문서 기반 언어에 적용 가능       |
| 역사적 맥락       | HTML 파싱 → 문서 구조 생성 → 문서 객체 모델링 |

---

## 📌 한 줄 요약

> DOM에서의 "Document"는 단지 `<html>` 문서뿐 아니라,
> **브라우저가 이해하고 조작할 수 있는 “전체 문서 구조”** 를 의미합니다.
> 따라서 DOM은 **문서를 위한 객체 기반 모델**이라는 뜻에서 이름 붙여졌습니다.

---

<br/>



# ✅ `document` 객체의 주요 프로퍼티 및 메서드

---

## 📌 1. 구조 이해

* `document` 객체는 `HTMLDocument` 또는 `XMLDocument` 타입입니다.
* `Document`는 `Node` → `EventTarget` → `Object`를 상속하는 트리의 상단 클래스입니다.

```js
console.log(document instanceof HTMLDocument); // true
console.log(document instanceof Document);     // true
```

---

## ✅ 2. 주요 프로퍼티 (속성)

| 프로퍼티                         | 설명                                                   |
| ---------------------------- | ---------------------------------------------------- |
| `document.documentElement`   | `<html>` 요소 (DOM의 최상단)                               |
| `document.body`              | `<body>` 요소                                          |
| `document.head`              | `<head>` 요소                                          |
| `document.title`             | `<title>` 태그 내용 (문서 제목)                              |
| `document.URL`               | 현재 문서의 URL                                           |
| `document.location`          | Location 객체 (주소, 이동 등 제어)                            |
| `document.cookie`            | 쿠키 읽기/쓰기                                             |
| `document.forms`             | 문서 내의 모든 `<form>` 요소 목록 (HTMLFormElementsCollection) |
| `document.images`            | 모든 `<img>` 요소                                        |
| `document.scripts`           | 모든 `<script>` 요소                                     |
| `document.readyState`        | 문서 로딩 상태 (`loading`, `interactive`, `complete`)      |
| `document.visibilityState`   | 탭의 표시 상태 (`visible`, `hidden`)                       |
| `document.activeElement`     | 현재 포커스를 갖는 요소                                        |
| `document.fullscreenElement` | 전체 화면 모드 중인 요소                                       |

---

## ✅ 3. 주요 메서드

### 🔍 DOM 탐색/선택 관련

| 메서드                             | 설명                          |
| ------------------------------- | --------------------------- |
| `getElementById(id)`            | 특정 ID를 가진 요소 반환             |
| `getElementsByClassName(class)` | 클래스 이름으로 요소 컬렉션 반환          |
| `getElementsByTagName(tag)`     | 태그 이름으로 요소 컬렉션 반환           |
| `querySelector(selector)`       | CSS 선택자 기반 단일 요소            |
| `querySelectorAll(selector)`    | CSS 선택자 기반 모든 요소 (NodeList) |
| `getElementsByName(name)`       | name 속성 기반으로 요소 찾기          |

### 🔍 DOM 생성 및 조작 관련

| 메서드                        | 설명                     |
| -------------------------- | ---------------------- |
| `createElement(tag)`       | 새로운 요소 노드 생성           |
| `createTextNode(text)`     | 텍스트 노드 생성              |
| `createDocumentFragment()` | 가상 DOM 조각 생성 (성능 최적화용) |
| `importNode(node, deep)`   | 외부 문서에서 노드 복사          |
| `adoptNode(node)`          | 외부 문서의 노드를 현재 문서로 가져오기 |

### 🔍 이벤트 및 상태 관련

| 메서드                     | 설명         |
| ----------------------- | ---------- |
| `addEventListener()`    | 이벤트 리스너 등록 |
| `removeEventListener()` | 이벤트 리스너 제거 |

### 🔍 기타 기능성 메서드

| 메서드                 | 설명                          |
| ------------------- | --------------------------- |
| `write(html)`       | 문서에 문자열 HTML을 삽입 (동기적, 비권장) |
| `open()`, `close()` | 문서 스트림 제어 (write와 함께 사용됨)   |
| `execCommand()`     | 편집 명령 실행 (deprecated)       |
| `exitFullscreen()`  | 전체화면 종료                     |
| `hasFocus()`        | 현재 문서가 포커스를 가지고 있는지 여부      |

---

## 🧪 예제: 실전 사용 시나리오

```js
// 특정 요소 접근
const el = document.getElementById("main");

// 새 요소 생성 및 추가
const newDiv = document.createElement("div");
newDiv.textContent = "Hello!";
document.body.appendChild(newDiv);

// 문서 정보 확인
console.log(document.URL);
console.log(document.readyState);
```

---

## 📊 문서 상태 감지 (`DOMContentLoaded`, `readyState`)

```js
document.addEventListener("DOMContentLoaded", () => {
  console.log("문서 파싱 완료. DOM 접근 가능.");
});

if (document.readyState === "complete") {
  console.log("모든 리소스 로딩 완료됨");
}
```

---

## 🧠 고급 팁

* `document.createDocumentFragment()`를 사용하면 여러 노드를 묶어서 한 번에 추가 가능 → Reflow 최소화
* `document.activeElement`를 통해 포커스를 가진 요소를 추적 가능 (접근성, UX 개선)
* `document.visibilityState`를 활용해 **탭이 백그라운드일 때 애니메이션/타이머 중단** 가능

---

## ✅ 요약

| 카테고리   | 대표 프로퍼티/메서드                                      | 역할         |
| ------ | ------------------------------------------------ | ---------- |
| 구조 접근  | `body`, `head`, `documentElement`                | 문서 구조 탐색   |
| DOM 탐색 | `getElementById`, `querySelector`                | 요소 선택      |
| DOM 조작 | `createElement`, `appendChild`                   | 요소 생성/삽입   |
| 이벤트/상태 | `readyState`, `addEventListener`                 | 문서 상태와 이벤트 |
| 기능 확장  | `cookie`, `fullscreenElement`, `visibilityState` | 사용자/환경 정보  |


