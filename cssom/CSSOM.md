

## 1. CSSOM이란 무엇인가? 🧱

### 1-1. DOM vs CSSOM

브라우저가 HTML/CSS를 읽으면 대략 이렇게 나뉩니다.

* **DOM (Document Object Model)**

  * HTML 문서의 **구조**와 **콘텐츠**를 트리 구조로 나타낸 것
  * 노드 예: `document`, `html`, `body`, `div`, `span` 등
* **CSSOM (CSS Object Model)**

  * CSS 스타일 시트, 규칙, 선택자, 선언들을 **객체 그래프**(트리)로 나타낸 것
  * 노드 예: `CSSStyleSheet`, `CSSRule`, `CSSStyleRule`, `CSSMediaRule`, `CSSStyleDeclaration` 등

📌 **포인트**
DOM은 “**무엇이 있는가**”를 나타내고,
CSSOM은 “**어떻게 보여야 하는가**”를 나타냅니다.

렌더링 엔진은 이 둘을 합쳐서 **Render Tree(렌더 트리)**를 만들고, 그걸 기반으로 레이아웃/페인팅을 합니다.

---

## 2. CSSOM이 만들어지는 과정 🔄

브라우저 파이프라인을 간단히 단계로 나누면:

1. **HTML 파싱 → DOM 생성**
2. **CSS 파싱 → CSSOM 생성**
3. **DOM + CSSOM → Render Tree 생성**
4. **레이아웃(Layout) 계산**
5. **페인팅(Paint) & 컴포지팅(Compositing)**

여기서 오늘의 주인공은 2단계입니다.

### 2-1. CSS 파싱: 문자열 → 토큰 → AST → CSSOM

브라우저는 `<style>` 태그나 `<link rel="stylesheet">`로 로드된 CSS를 다음 과정으로 처리합니다.

1. **Tokenizer**: CSS 텍스트를 토큰 단위로 쪼갬

   * 예: `body { color: red; }`
     → `IDENT(body)`, `{`, `IDENT(color)`, `:`, `IDENT(red)`, `;`, `}`
2. **Parser**: 토큰 스트림을 **규칙(규범 문법)**에 맞추어 **CSS Rule Tree(AST)** 생성
3. **객체화(Objectification)**: 이 AST를 JS에서 다루기 좋은 객체인 **CSSOM**으로 변환

   * 전체: `CSSStyleSheet`
   * 그 안의 규칙들: `CSSStyleRule`, `CSSMediaRule`, `CSSFontFaceRule` …

즉, CSSOM은 그냥 내부 구현으로 끝나는 게 아니라, **자바스크립트에서 직접 접근 가능한 정식 API**입니다.

---

## 3. CSSOM의 계층 구조 👀

대표적인 구조를 정리해 보면:

```text
document.styleSheets → StyleSheetList
  ├─ CSSStyleSheet
  │   ├─ cssRules → CSSRuleList
  │   │   ├─ CSSStyleRule (일반 규칙: body { ... })
  │   │   ├─ CSSMediaRule (@media ...)
  │   │   ├─ CSSFontFaceRule (@font-face ...)
  │   │   └─ ...
  │   └─ ownerNode (link 또는 style 엘리먼트)
  └─ ...
```

### 3-1. 주요 인터페이스들

#### ① `StyleSheetList` / `CSSStyleSheet` 📄

* `document.styleSheets`: 모든 스타일 시트를 담는 유사 배열
* 각 항목은 `CSSStyleSheet` 객체
* 주요 프로퍼티/메서드:

  * `cssRules`: `CSSRuleList` (규칙 리스트)
  * `insertRule(ruleText, index)`
  * `deleteRule(index)`
  * `disabled`: 스타일시트 활성/비활성 여부

#### ② `CSSRule` 및 서브타입들 📏

* 모든 규칙의 베이스 타입: `CSSRule`
* 타입에 따라 상속 구조:

  * `CSSStyleRule`: 일반 규칙 (예: `div { color: red; }`)
  * `CSSMediaRule`: `@media` 블록
  * `CSSFontFaceRule`: `@font-face`
  * `CSSKeyframesRule`, `CSSKeyframeRule`: 애니메이션
  * …

`CSSStyleRule`의 대표 멤버:

* `selectorText`: `"div.highlight"`
* `style`: `CSSStyleDeclaration` (실제 속성/값의 집합)

#### ③ `CSSStyleDeclaration` 🎨

* 하나의 규칙 안에 있는 `color`, `margin` 같은 선언들을 담는 객체
* 예: `rule.style.color = "red";`
* `element.style`도 같은 타입 (`CSSStyleDeclaration`)

---

## 4. CSSOM vs “Computed Style” vs “사용자 스타일” 🧠

많이 헷갈리는 지점입니다.

* **CSSOM**

  * “원본 스타일 시트의 구조”
  * 예: `CSSStyleSheet`, `CSSRule`, `CSSStyleRule`
* **Element.style (Inline 스타일)**

  * HTML에서 `style` 속성으로 직접 써놓은 스타일
  * 예: `<div style="color: red;">` → `elem.style.color === "red"`
* **Computed Style (계산된 스타일)**

  * 모든 CSS 규칙/상속/우선순위/기본값/브라우저 디폴트를 전부 고려해서 **최종 계산된 값**
  * `window.getComputedStyle(elem)`으로 접근

즉:

```text
CSS 파일 + <style> 태그 + <element style="...">  → CSSOM
CSSOM + UA 스타일 + 상속 + 우선순위           → Computed Style
```

---

## 5. 실제 코드로 보는 CSSOM 조작 예제 💻

### 5-1. 모든 스타일시트와 규칙 출력하기

```js
// 모든 스타일시트 순회
for (const sheet of document.styleSheets) {
  console.log("=== StyleSheet ===");
  try {
    for (const rule of sheet.cssRules) {
      console.log(rule.cssText);
    }
  } catch (e) {
    // CORS 때문에 접근 불가능한 경우 (외부 도메인 CSS)
    console.warn("Cannot access rules of this stylesheet:", e);
  }
}
```

💡 외부 CDN CSS(e.g. Google Fonts)는 보안 정책(CORS) 때문에 `cssRules` 접근 시 예외가 날 수 있습니다.

### 5-2. 동적으로 규칙 추가

```js
const sheet = document.styleSheets[0];
sheet.insertRule(
  "body.custom-theme { background-color: #121212; color: #f1f1f1; }",
  sheet.cssRules.length
);

// 나중에 document.body.classList.add("custom-theme") 하면 적용
```

### 5-3. 기존 규칙 수정

```js
const sheet = document.styleSheets[0];
for (const rule of sheet.cssRules) {
  if (rule.type === CSSRule.STYLE_RULE && rule.selectorText === ".btn-primary") {
    rule.style.backgroundColor = "#007bff";
    rule.style.borderRadius = "999px";
  }
}
```

이건 Sass나 Tailwind가 아니라, **런타임에 CSS를 “코드로” 편집하는 행위**입니다.
CSSOM을 모르고는 이런 작업을 안전하게 하기가 어렵습니다.

---

## 6. 렌더링 엔진과 CSSOM: Reflow/Repaint 관계 ⚙️

### 6-1. CSSOM 변경 = 스타일 재계산

브라우저 입장에서 CSSOM은 **스타일 계산을 위한 입력 값**입니다.

* `CSSOM`이 바뀌면
* 영향을 받는 요소들의 **Computed Style** 재계산
* 필요시 **레이아웃(Reflow) / 페인트(Repaint)** 발생

예를 들어:

```js
rule.style.fontSize = "40px";  // 많은 요소에 적용되는 규칙이라면…
```

이 규칙이 수백 개의 요소에 적용될 경우, 모든 요소에 대해 레이아웃을 다시 계산해야 할 수 있습니다 → 성능 비용이 큽니다.

💣 **성능 관점에서 CSSOM 조작은 매우 비싼 작업**입니다.
가능하면:

* **클래스 토글**(`classList.add/remove`)로 해결하고,
* CSS 규칙 자체를 자주 바꾸는 패턴은 피하는 것이 좋습니다.

---

## 7. CSSOM과 JavaScript 스타일 조작의 차이점 🔍

### 7-1. `element.style` vs CSSOM vs className

1. **`element.style`**

   * 해당 요소의 인라인 스타일만 조작
   * 다른 요소에는 영향 없음
   * **장점:** 특정 요소만 빠르게 바꾸기 좋음
   * **단점:** 스타일 로직이 JS에 흩어짐 (유지보수 어려움)

2. **CSSOM 조작 (스타일시트 기반)**

   * `CSSStyleSheet`를 통해 규칙 자체를 수정
   * 다수 요소에 동시에 영향
   * **장점:** 테마 스위칭, 다크 모드, A/B 테스팅 등에 강력
   * **단점:** 잘못 사용하면 전체 페이지 리플로우, 성능 저하

3. **클래스 토글 (`classList`)**

   * 추천 패턴
   * CSS는 그대로 두고, JS는 **상태(클래스)**만 바꿈
   * CSSOM 변경 없이도 스타일이 바뀌므로 상대적으로 안전

```js
// CSS (정적)
.dark-mode {
  background-color: #121212;
  color: #f1f1f1;
}

// JS
document.body.classList.toggle("dark-mode");
```

실무에서는 대부분 **3번 패턴**을 선호하고, 2번(CSSOM 조작)은 좀 더 특수한 상황(에디터, 디자이너 툴, 런타임 테마 생성 등)에서 사용합니다.

---

## 8. CSSOM과 SPA/React/Vue 프레임워크의 관계 🧬

React, Vue, Svelte, Angular 등의 프레임워크를 쓰더라도 결국 **브라우저는 동일한 CSSOM을 사용**합니다.

* CSS-in-JS 라이브러리(Styled-Components, Emotion 등)는 내부적으로 **스타일 태그를 생성 → CSS 텍스트 삽입 → CSSOM 반영**을 합니다.
* 동적 스타일 생성, 해시된 클래스네임(`.Button__primary___hash`)도 결국 CSSOM 안에 규칙으로 들어갑니다.

즉, 프레임워크/라이브러리는 **CSSOM을 수동으로 만지는 것을 추상화**해주는 도구일 뿐이고,
**하부 레벨에서는 언제나 CSSOM이 돌아가고 있습니다.**

프레임워크를 깊이 이해하고 최적화하려면, 결국 **CSSOM과 렌더링 파이프라인까지 이해해야** 합니다. 🎯

---

## 9. CSSOM과 CORS/보안 제약 🚧

자바스크립트에서 `document.styleSheets`를 순회하다 보면, 다음과 같은 에러를 만날 수 있습니다.

```text
SecurityError: Failed to read the 'cssRules' property from 'CSSStyleSheet': 
Cannot access rules
```

이유:

* 다른 도메인에서 로드된 CSS 파일(예: `https://fonts.googleapis.com/...`)은
  **Same-Origin Policy** 때문에 CSSOM 상세 접근이 막혀 있습니다.
* 스타일시트를 로드할 때 CORS 헤더가 올바르게 설정되어 있으면 예외를 회피할 수 있지만,
  일반적인 CDN 스타일은 대체로 읽기 제한이 걸려 있습니다.

따라서 CSSOM을 다룰 때는 항상 `try { ... } catch (e) { ... }` 패턴을 사용하는 것이 안전합니다.

---

## 10. CSSOM을 이해하면 보이는 것들 🔮

CSSOM을 제대로 이해하면:

1. **렌더링 최적화**

   * 무엇을 변경하면 레이아웃/페인트가 일어나고, 어떤 조작이 비싼지 감이 생깁니다.
2. **CSS-in-JS 내부 동작 이해**

   * 왜 스타일 태그를 동적으로 추가하는지, 왜 해시된 클래스네임이 필요한지 이해할 수 있습니다.
3. **브라우저 DevTools 해석 능력 향상**

   * “Rules”, “Computed”, “Inherited” 탭이 실제로 어느 레벨의 데이터를 보여주는지 명확해집니다.
4. **고급 에디터/디자인 툴 구현**

   * 웹 기반 페이지 빌더, 테마 에디터를 만들 때 CSSOM 조작이 핵심 기술 중 하나가 됩니다.

---

## 마무리 🧩

정리하면,

> **CSSOM = 브라우저가 CSS를 이해하기 위해 만든 공식 객체 모델**
> DOM이 “구조/내용”의 모델이라면, CSSOM은 “스타일 규칙”의 모델입니다.

* 렌더링 파이프라인에서 DOM과 함께 Render Tree를 구성하는 핵심 요소이고,
* 자바스크립트에서 직접 접근/수정 가능한 **강력한 but 위험한(성능 측면에서)** 도구입니다.

---

참고 사항:

* **Render Tree / Layout / Paint 과정에서 CSSOM이 어떻게 사용되는지**
* **React/Vite 기반 프로젝트에서 CSSOM과 DevTools를 활용한 렌더링 최적화 예시**
* **CSS-in-JS(Styled-Components) 내부 구현을 CSSOM 관점에서 뜯어보기**

