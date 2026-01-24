PostCSS는 “그냥 CSS 후처리기”가 아닙니다.
**CSS 생태계의 문제를 해결하기 위해 등장한, 플러그인 기반 “플랫폼”**에 가깝습니다.
이번 글에서는 *“왜 굳이 PostCSS가 필요했는가?”*를 역사·기술적 관점에서 깊이 있게 정리해보겠습니다. 🚀

---

## 0. PostCSS 한 줄 요약 🌱

> **“미래의 CSS를 마음껏 쓰고, 플러그인으로 가공해서 오늘의 브라우저에서 돌게 만드는 엔진”**

PostCSS는 **CSS 코드를 AST(Abstract Syntax Tree)로 파싱한 뒤,
각종 플러그인이 이 AST를 변환하고, 다시 CSS 문자열로 내보내는 플랫폼**입니다.

그런데, *도대체 왜 이런 플랫폼이 필요했을까요?*
이걸 이해하려면, PostCSS가 나오기 전 CSS 세계의 현실부터 봐야 합니다.

---

## 1. 과거 CSS의 현실: “스펙은 멋진데, 브라우저는 아직…” 😵

### 1-1. CSS 스펙은 빠르게 진화하지만, 브라우저는 느리게 따라옴

CSS Working Group은 열심히 새 기능을 설계합니다.

* CSS 변수: `var(--primary-color)`
* `calc()`, `@supports`, `@custom-media`
* Flexbox / Grid / Custom Properties / Nesting 등

하지만 현실은…

* 브라우저가 기능을 **“부분 지원”** 하거나
* 구버전 브라우저는 전혀 지원하지 않거나
* 실험적 기능은 prefix를 붙여야 하거나 (`-webkit-`, `-moz-` 등)

즉, **개발자가 “미래의 CSS”를 공부해도, 실제 서비스에서는 못 쓰는 상황**이 반복됐습니다.

> 🙋‍♂️ “나 최신 CSS 쓰고 싶은데, IE / 구형 브라우저 때문에 못 써…”

이 불만이 커지던 시기에 JavaScript 쪽에서는 이미 **Babel**이 등장해서 상황이 달라지고 있었습니다.

---

## 2. JS 세계에 이미 있던 해답: Babel ✨

JavaScript 생태계는 PostCSS보다 먼저 이런 문제를 해결하고 있었습니다.

* 개발자는 **최신 JavaScript (ES6+)** 로 작성
* Babel이 이를 **구버전 브라우저가 이해할 수 있는 ES5 코드로 변환**
* 결과적으로 개발자는 “미래의 JS 문법”을 마음껏 사용

즉, JS 세계에서는 이미 이런 공식이 자리 잡았습니다.

> “*미래 문법으로 작성 → 트랜스파일러가 현재 브라우저에 맞게 변환*”

CSS 쪽에도 자연스럽게 이런 질문이 생깁니다.

> “CSS에도 Babel 같은 게 있으면 안 되나?”
> “미래 CSS 문법으로 코딩하고, 빌드 단계에서 변환하면 되잖아?”

바로 이 필요성이 **PostCSS의 정신적 출발점**입니다. 🔥

---

## 3. preprocessor(Sass/Less)의 한계: “다 해주긴 하는데, 너무 덩어리” 🧱

PostCSS 이전에도 CSS를 좀 더 편하게 쓰기 위한 **전처리기(preprocessor)** 들이 있었습니다.

* Sass / SCSS
* Less
* Stylus

이들은 이런 기능을 제공했죠.

* 변수 (`$primary: #333;`)
* 중첩 셀렉터
* mixin, 함수, 상속 등

하지만 한계도 분명했습니다.

### 3-1. “자기 왕국” 안에서만 사는 도구들

Sass 문법은 Sass만의 세계, Less는 Less의 세계입니다.

* CSS 표준과는 다른 **별도의 언어**에 가깝고
* 기능은 도구 내부에 “박혀” 있어서

  * *내가 딱 이것만 바꾸고 싶은데…* 라는 유연성이 부족

### 3-2. 플러그인 생태계의 느슨함

Sass에도 확장 개념이 있지만, **“플러그인을 조립해서 도구를 만든다”** 라는 사고보다는

> 하나의 거대한 preprocessor가 많은 걸 제공하고,
> 사용자는 그 안에서 노는 방식

에 가깝습니다.

반면, PostCSS는 아예 출발부터 달랐습니다.

> “**핵심은 파서 + AST + 플러그인 시스템뿐**이고,
> 나머지 기능은 모두 플러그인으로 구현하자.”

이게 나중에 Tailwind, Autoprefixer, CSS Modules 같은 **거대한 생태계**로 이어집니다.

---

## 4. Autoprefixer의 등장: PostCSS 성공의 1차 트리거 🚦

PostCSS는 러시아 개발자 **Andrey Sitnik**에 의해 만들어졌는데,
그가 만든 가장 유명한 플러그인이 바로 **Autoprefixer**입니다.

### 4-1. 과거의 prefix 지옥 👿

한때 CSS 코드를 이렇게 쓰는 것이 일상이었습니다:

```css
.box {
  -webkit-border-radius: 4px;
  -moz-border-radius: 4px;
  border-radius: 4px;
}
```

개발자의 고통:

* 어떤 속성에 어떤 prefix가 필요한지 일일이 조사
* [Can I use]를 수시로 열어봄
* 브라우저 지원 상황이 바뀌면 코드도 일일이 수정

### 4-2. Autoprefixer가 가져온 혁명 ⚡

Autoprefixer 철학:

> “개발자는 **표준 CSS**만 쓰고,
> prefix는 빌드 도구가 브라우저 지원표를 보고 **자동으로 붙여주자**.”

이 플러그인은 PostCSS 위에서 돌아갑니다:

1. PostCSS가 CSS를 파싱해서 AST로 변환
2. Autoprefixer가 AST를 분석하여 필요한 곳에만 prefix를 삽입
3. 다시 문자열로 변환해서 최종 CSS 출력

이 한 가지 성공 사례 덕분에 사람들은 깨닫습니다.

> “어? PostCSS 위에 이렇게 유용한 도구들을 더 많이 만들 수 있겠는데?”

그리고 이때부터 **PostCSS = Autoprefixer 엔진**이 아니라
**PostCSS = CSS 도구들을 올려두는 공용 플랫폼** 으로 인식되기 시작합니다.

---

## 5. CSS 도구 생태계가 파편화되어 있었다 🧩

과거 CSS 관련 도구들을 보면:

* Autoprefixer
* CSS minifier (cssnano, clean-css 등)
* Media query combiner
* Comment stripper
* Custom property polyfill
* Nested rule transformer
* CSS Modules transformer
* …각각 제각각 동작

**각 도구마다 파서가 따로 있고**,
각 도구가 CSS를 문자열로 파싱 → 조작 → 문자열로 내보내고 있었습니다.

문제점:

* 도구마다 구현 중복
* 서로 호환 잘 안 됨
* 빌드에서 여러 도구를 조합하기 어렵고, 속도도 손해

### 5-1. PostCSS의 제안: “엔진은 하나, 기능은 플러그인으로” 🔧

PostCSS는 이렇게 선언합니다.

> “파싱과 AST 관리, 출력은 내가 할게.
> 변환 로직은 플러그인들이 해.”

그래서:

* 모든 CSS 관련 도구가 하나의 AST 포맷을 공유
* 플러그인 간 조합 가능 (`[autoprefixer] → [cssnano] → [custom properties polyfill]` …)
* 빌드 체인(Lint, Optimize, Transform)이 한 파이프라인에서 처리

이게 **“CSS를 위한 공용 런타임”** 같은 역할이 되면서,
PostCSS는 사실상 **CSS 빌드의 기준 플랫폼**이 됩니다.

---

## 6. 빌드 시스템과의 통합 요구: Webpack, Gulp, Parcel, Vite… 🛠️

현대 프론트엔드 빌드는 항상 이런식입니다:

* JS: Babel/TypeScript → 번들러(Webpack, Vite 등)
* CSS: Sass/Less/PostCSS → 번들러

여기서 PostCSS는 **결합성이 좋게 설계**되었습니다.

* CLI 모드
* Node.js API
* Gulp 플러그인
* Webpack loader로 연동
* Vite의 `postcss` 옵션, `postcss.config.js`로 자연스럽게 연동

즉,

> “어떤 빌드 툴을 쓰든, CSS 처리에는 PostCSS를 기본 엔진으로 쓰자.”

라는 흐름을 만들었습니다.
오늘날 Vite + React + Tailwind 쓰시면, 사실 대부분 **PostCSS는 자동으로 끼어 들어가 있습니다.** 😉

---

## 7. Tailwind CSS와 같은 “새로운 유형의 CSS 프레임워크”를 위한 기반 🧬

PostCSS가 없었다면 **Tailwind CSS** 같은 프레임워크도 구현 난이도가 상당히 높았을 것입니다.

Tailwind는 기본적으로:

1. 프로젝트 전체 파일(HTML/JSX/TSX 등)을 스캔해서 **사용된 클래스만 추출**
2. 그에 맞는 CSS 유틸리티를 **동적으로 생성**
3. 최종 CSS를 **PostCSS 파이프라인을 통해 최적화/후처리**

이 과정에서 PostCSS는:

* Tailwind 플러그인으로서 동작하거나
* Tailwind가 생성한 CSS를 다시 처리하는 엔진 역할

을 합니다.

> 즉, PostCSS는 **기존 CSS를 살짝 편하게 만드는 도구를 넘어서,
> 새로운 CSS 프레임워크 자체를 “만들 수 있는 플랫폼”** 이 된 것입니다.

---

## 8. “CSS를 위한 Babel”이라는 철학 🧠

PostCSS의 저자 Andrey Sitnik는 여러 인터뷰와 문서에서 이런 철학을 드러냅니다:

* “PostCSS는 preprocessor가 아니다.”
* “우리는 CSS 생태계를 위한 플랫폼을 만들고 싶었다.”
* “미래 CSS를 유연하게 실험하고, 표준이 정착되면 자연스럽게 플러그인을 교체하거나 제거할 수 있게 하자.”

즉, PostCSS의 설계 철학은 이렇습니다.

1. **CSS의 미래 기능을 먼저 실험하는 실험장**

   * `postcss-nesting`, `postcss-custom-properties`, `postcss-preset-env` 등
   * 나중에 브라우저가 이 기능을 정식 지원하면 플러그인을 제거하면 끝

2. **도메인 특화 CSS를 마음껏 만들 수 있는 런타임**

   * Tailwind, CSS Modules, Linaria, Styled JSX 등 각종 툴과도 연계

3. **도구 개발자를 위한 공용 인프라**

   * 파서/AST/출력기를 직접 구현할 필요 없이, PostCSS 위에 올라타면 됨

---

## 9. 정리: PostCSS가 태어난 진짜 이유  🔍

지금까지 내용을 한 번에 요약해보겠습니다.

### ✅ 배경 요약

1. **CSS 스펙 vs 브라우저 구현 간의 시간차**

   * 최신 CSS를 바로 쓰고 싶지만, 브라우저가 지원을 안 함 → Babel 같은 것이 필요

2. **Vendor prefix 지옥**

   * 프리픽스를 사람이 관리하는 시대를 끝내고 싶었음 → Autoprefixer

3. **Preprocessor(Sass, Less)의 아키텍처 한계**

   * “모든 기능을 한 툴이 책임지는 거대한 왕국” → 플러그인 기반 “플랫폼”이 필요

4. **CSS 도구 생태계의 파편화**

   * 각자 파서/출력/AST를 중복 구현 → 하나의 공용 엔진 필요

5. **현대 빌드 시스템과의 통합 요구**

   * Webpack, Gulp, Vite 등과 자연스럽게 엮일 수 있는 일반적인 엔진 필요

6. **Tailwind 같은 새로운 CSS 패러다임의 기반**

   * 유틸리티-퍼스트, JIT, 동적 CSS 생성 등은 플러그인 플랫폼이 있어야 구현이 쉬움

### 🎯 그래서 PostCSS는…

> **CSS 생태계 전반을 위한 “플러그인 기반 변환 플랫폼”이자,
> “CSS 세계의 Babel/AST 엔진”으로 태어났습니다.**

