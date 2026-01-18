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
     → `IDENT(body)`, `{`, `IDENT(color)`, `:`, `IDENT(red)`, `;`, `항:

* **Render Tree / Layout / Paint 과정에서 CSSOM이 어떻게 사용되는지**
* **React/Vite 기반 프로젝트에서 CSSOM과 DevTools를 활용한 렌더링 최적화 예시**
* **CSS-in-JS(Styled-Components) 내부 구현을 CSSOM 관점에서 뜯어보기**

