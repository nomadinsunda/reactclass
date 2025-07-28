

# 🧠 웹 브라우저의 렌더링 과정(Rendering Pipeline & CRP)

> HTML과 CSS, JavaScript가 어떻게 브라우저에서 픽셀로 변환되어 사용자에게 보여지는가?

프론트엔드 개발자로서 **렌더링 파이프라인(Rendering Pipeline)** 에 대한 이해는 퍼포먼스 최적화의 핵심입니다.
이 글에서는 **렌더링의 내부 구조와 흐름을 전문가 관점에서** 설명하고,
JavaScript 실행이 **Critical Rendering Path**에 어떻게 영향을 미치는지까지 정리합니다.

---

## 🚀 렌더링 파이프라인 개요

웹 브라우저는 다음과 같은 **단계적 처리 파이프라인**을 통해 웹 페이지를 화면에 표시합니다.


<img src="./images/render_pipeline_vertical.svg" width=40% height="auto" /><br>

---

## 1️⃣ Loading: 리소스 수신

* 브라우저는 HTML 요청을 시작으로 모든 리소스를 **네트워크 계층**을 통해 가져옵니다.
* 이 과정에서 **HTTP/2, 캐시, CDN, TCP 연결** 등이 영향을 미칩니다.
* HTML 문서가 도착하면 브라우저의 \*\*렌더링 엔진(Rendering Engine)\*\*이 본격적으로 작동합니다.

### 🔸 렌더링 차단 리소스

* CSS: 렌더링 전 스타일 계산 필요 → 반드시 먼저 처리되어야 함.
* JavaScript:

  * `<script>`는 HTML 파싱을 **블로킹**합니다.
  * `async`, `defer`로 이 문제를 완화할 수 있습니다.

---

## 2️⃣ HTML 파싱 → DOM Tree 생성

* HTML은 **토큰 → 노드 → 트리 구조**로 파싱됩니다.
* 생성된 트리는 \*\*DOM (Document Object Model)\*\*이라고 불리며, 메모리에 구조화됩니다.

### 📌 예시: 중첩 DOM 트리

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

### ▶️ DOM 트리 구조

<img src="./images/domtree.svg" width=50% height="auto" /><br>

> 📌 `<script>` 태그를 만나면 **HTML 파싱은 중단**되고, JS 엔진이 해당 스크립트를 먼저 실행합니다.

---

## 3️⃣ CSS 파싱 → CSSOM 생성

* CSS 파일이 로딩되면 브라우저는 이를 **CSSOM (CSS Object Model)** 트리로 파싱합니다.
* 계산에는 다음 요소들이 고려됩니다:

  * 선택자 우선순위(Specificity)
  * 상속
  * `!important`
  * 캐스케이딩 규칙

> ⚠ CSS가 준비되지 않으면 **렌더 트리 생성이 불가능하므로 페이지는 비어 있게 됩니다.**

---

## 4️⃣ DOM + CSSOM → Render Tree 생성

* 브라우저는 DOM과 CSSOM을 조합하여 **Render Tree**를 생성합니다.
* **화면에 실제로 표시될 요소만 포함**:

  * `display: none`은 제외
  * `visibility: hidden`은 포함되지만 보이지 않음

---

## 5️⃣ Layout (또는 Reflow)

* Render Tree의 각 노드에 대해 **좌표(x, y)**, **크기(width, height)**, \*\*박스 모델(box-sizing)\*\*을 계산합니다.
* 요소의 위치, margin, padding, transform 등이 모두 반영됩니다.

> 💡 이 작업은 **연산 비용이 매우 높기 때문에 자주 발생하면 성능 저하**를 유발합니다.

---

## 6️⃣ Paint (Rasterization)

* 각 노드를 **픽셀로 변환(painting)** 하여 실제 시각 요소를 렌더링합니다.
* 색상, 배경, 그림자, 이미지 등이 포함됩니다.
* **GPU 가속**이 지원되는 브라우저는 이 작업을 더 빠르게 수행합니다.

---

## 7️⃣ Compositing (합성 단계)

* Paint된 레이어를 GPU에서 **합성(Compositing)** 하여 최종 화면을 만듭니다.
* 일부 요소들은 별도의 **레벨(layer)** 로 분리되어 처리됩니다:

  * `will-change`, `transform`, `opacity` 등

---



## 🎯 Reflow vs Repaint: 성능에 민감한 변화

| 구분                  | 발생 조건                     | 비용     |
| ------------------- | ------------------------- | ------ |
| **Repaint**         | 시각적 스타일만 변경 (ex. `color`) | 낮음     |
| **Reflow (Layout)** | 레이아웃 구조 변경 (DOM 위치/크기 등)  | **높음** |

```js
// 강제로 Reflow를 유발하는 예
element.style.width = '500px';
console.log(element.offsetHeight);  // 레이아웃 강제 계산
```

---

## 🔁 JavaScript의 이벤트 루프와 렌더링의 관계

JavaScript는 **싱글 스레드 기반**으로, 렌더링과 같은 **브라우저 UI 작업과 같은 Main Thread를 공유**합니다.

### 주요 지점에서 JS가 끼어드는 시점

* HTML 파싱 중 `<script>` 태그 발견 → HTML 파서 중단 → JS 실행
* JS가 DOM을 조작할 경우 → Reflow or Repaint 유발
* `setTimeout`, `requestAnimationFrame`, `Promise` 등의 비동기 작업 → 이벤트 루프 처리

---



## ⚙ 전문가용 최적화 팁

| 항목         | 전략                             |
| ---------- | ------------------------------ |
| JS 로딩      | `defer`로 HTML 파싱 병렬화           |
| CSS 최적화    | 크리티컬 CSS 분리 & 최소화              |
| 이미지        | lazy-loading, WebP             |
| DOM 구조     | 깊이 최소화, 레이아웃 단순화               |
| 레이아웃 연산    | `getBoundingClientRect` 호출 최소화 |
| GPU 레이어 분리 | `will-change: transform` 활용    |

---

## 🧩 브라우저 엔진별 특징

| 브라우저         | 엔진     | 특징                        |
| ------------ | ------ | ------------------------- |
| Chrome, Edge | Blink  | WebKit 기반, V8 탑재, GPU 최적화 |
| Safari       | WebKit | Apple 전용, 퍼포먼스 좋음         |
| Firefox      | Gecko  | 독립 엔진, 렌더링 전략이 상이함        |

---

## ✅ 마무리

렌더링 파이프라인을 정확히 이해하면 다음과 같은 영역에서 큰 장점을 가집니다:

* 성능 최적화 (CLS, LCP 등 Core Web Vitals 개선)
* 애니메이션, 스크롤, 인터랙션 최적화
* 레이아웃 구조 설계에 대한 직관

<img src="./images/webrenderingseq.png" width=90% /><br>
