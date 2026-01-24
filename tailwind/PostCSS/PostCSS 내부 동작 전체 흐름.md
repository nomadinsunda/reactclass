PostCSS는 “CSS용 Babel”이라고 부르면 감이 딱 옵니다.
**문자열 CSS → AST → 플러그인 파이프라인 → 다시 문자열 CSS** 🎯
이 흐름을 머릿속에 그려두시면, Tailwind, Autoprefixer, Nesting 등 모든 것의 정체가 한 번에 정리됩니다.

---

## 1. 전체 큰 그림부터 보기 🧠

PostCSS 내부 동작은 한 문장으로 요약하면 이렇습니다.

> **CSS 텍스트를 파싱해서 AST로 만든 다음, 플러그인들을 순서대로 적용해서 AST를 수정하고, 마지막에 다시 문자열로 직렬화한다.**

흐름을 단계별로 그려보면:

```text
[1] 입력 CSS 문자열
        │
        ▼
[2] Parser (파서)
        │  → AST 트리 생성 (Root, Rule, Decl, AtRule, Comment...)
        ▼
[3] 플러그인 파이프라인 (Processor)
     ├─ Plugin A (예: tailwindcss)
     ├─ Plugin B (예: autoprefixer)
     └─ Plugin C (...)
        │  → AST를 순차적으로 mutate
        ▼
[4] Stringifier (코드 생성)
        │  → AST → 최종 CSS 문자열(+ source map)
        ▼
[5] 출력 CSS
```

여기서 핵심 객체는 3개입니다.

1. **AST(Node 트리)**
2. **Processor(플러그인 목록을 가진 실행기)**
3. **Result/LazyResult(실행 결과 래퍼)**

---

## 2. PostCSS가 입력 CSS를 받는 순간 📥

### 2-1. Processor 생성

대부분의 빌드 툴(Vite, Webpack, CLI)은 먼저 **Processor 인스턴스**를 만듭니다.

```js
import postcss from 'postcss'
import tailwindcss from 'tailwindcss'
import autoprefixer from 'autoprefixer'

const processor = postcss([
  tailwindcss(),
  autoprefixer(),
])
```

* `postcss([ plugin들 ])` → **Processor 객체** 생성
* Processor는 단순히 **“플러그인 배열을 들고 있는 실행자”**입니다. 아직 CSS는 모름.

### 2-2. `.process()` 호출 → LazyResult 생성

```js
const lazyResult = processor.process(cssInput, {
  from: 'src/input.css',
  to:   'dist/output.css',
  map:  { inline: false },
})
```

* `cssInput`은 문자열 또는 파일 내용입니다.
* 이 순간에도 **실제 변환은 아직 안 됨** ❗
  → PostCSS는 게으른 평가를 위해 `LazyResult`라는 객체를 반환합니다.

---

## 3. Parser: CSS → AST로 바꾸는 과정 🌳

이제 `lazyResult.css`를 읽거나 `await lazyResult`를 할 때 진짜 작업이 시작됩니다.

### 3-1. CSS 파싱

내부적으로 대략 이런 일을 합니다.

1. CSS 문자열을 문자 단위로 스캔
2. `{`, `}`, `:`, `;`, `@`, `/* */`, whitespace 등을 기준으로 토큰화
3. 문법 규칙에 맞추어 **AST 트리**를 구성

PostCSS AST의 대표적인 노드 타입들은:

* `Root` : 전체 문서 루트
* `Rule` : `.btn { ... }` 같은 CSS 규칙
* `Decl` : `color: red` 같은 선언
* `AtRule` : `@media`, `@import`, `@tailwind` 등
* `Comment` : `/* … */`

간단한 예시:

```css
.btn {
  color: red;
}
```

파싱 후 AST 구조(개념적으로):

```js
Root {
  nodes: [
    Rule {
      selector: '.btn',
      nodes: [
        Decl { prop: 'color', value: 'red' }
      ]
    }
  ]
}
```

PostCSS는 이 AST를 **mutable 객체**로 가지고 있으며,
플러그인들은 이 트리를 자유롭게 읽고, 수정하고, 추가/삭제합니다.

---

## 4. 플러그인 파이프라인: AST를 돌면서 변환하기 🧩

이제 Processor가 들고 있는 **플러그인 목록**을 순차적으로 실행합니다.

```text
[AST] → Plugin1 → Plugin2 → Plugin3 → ... → 최종 AST
```

각 플러그인은 보통 다음 형태입니다.

```js
const myPlugin = postcss.plugin('my-plugin', (opts) => {
  // 초기화: 옵션 세팅
  return (root, result) => {
    // 여기서 root(AST)를 마음껏 변형
  }
})
```

최근 스타일은 함수 한 번 더 감싸지 않고 이렇게도 씁니다.

```js
const myPlugin = (opts = {}) => {
  return {
    postcssPlugin: 'my-plugin',
    Once(root, { result }) {
      // root는 AST의 Root 노드
    },
    // Rule, Decl, AtRule 등 특정 노드 타입을 잡고 싶으면:
    Rule(rule) { /* ... */ },
    Decl(decl) { /* ... */ },
  }
}
myPlugin.postcss = true
```

### 4-1. AST 순회(Traversal)

PostCSS가 플러그인을 돌릴 때:

* 전체 AST를 순회하면서
* 각 노드 타입에 맞는 **Visitor 함수**를 호출합니다.

예:

```js
const colorToBlue = (opts = {}) => ({
  postcssPlugin: 'color-to-blue',
  Decl(decl) {
    if (decl.prop === 'color') {
      decl.value = 'blue'    // AST mutate 🎯
    }
  },
})
colorToBlue.postcss = true
```

위 플러그인이 실행되면, 모든 `color: ...` 선언이 `color: blue`로 바뀝니다.
이 모든 것은 **AST 상에서 일어나는 변화**입니다. 아직 문자열로는 안 바뀐 상태.

---

## 5. Tailwind, Autoprefixer는 여기서 무엇을 하냐? 🧵

사용자님이 실제로 쓰시는 예를 기준으로 보겠습니다.

```js
postcss([
  tailwindcss(),
  autoprefixer(),
])
```

### 5-1. Tailwind 플러그인

Tailwind 플러그인은 AST 안의 `@tailwind` 같은 AtRule을 찾아서:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

이 부분을:

* Tailwind가 가진 Utility 규칙들로 **AST 노드들을 생성**하고
* 해당 위치에 **대량의 Rule/Decl 노드를 삽입**합니다.

즉, Tailwind는 사실상:

> “AST 상에서 @tailwind를 **수많은 CSS 규칙으로 확장(expand)**하는 플러그인”

입니다.

### 5-2. Autoprefixer 플러그인

Autoprefixer는 최종적으로 구성된 AST를 순회하면서:

* 특정 속성(예: `display: flex`)이나
* 특정 at-rule, selector, 값 등을 보고,

필요하다면:

* `-webkit-`, `-ms-` 같은 vendor prefix가 붙은 **새 Decl 노드**를 추가하거나
* 기존 Decl의 값을 바꾸거나 합니다.

결과적으로:

1. Tailwind가 **규칙을 폭발적으로 생성**하고
2. Autoprefixer가 그 위에 **브라우저 호환성 prefix를 얹어주는** 구조입니다.

---

## 6. Stringifier: AST → 최종 CSS 문자열로 직렬화 🧾

모든 플러그인이 끝나면, 이제 **AST를 다시 문자열로 변환**해야 합니다.

PostCSS의 Stringifier는:

* 각 노드 타입(`Rule`, `Decl`, `AtRule`, `Comment`)에 대해
* 어떤 식으로 문자열을 생성할지 정의되어 있습니다.

예:

```js
Rule {
  selector: '.btn',
  nodes: [ Decl { prop: 'color', value: 'red' } ]
}
```

→

```css
.btn {
  color: red;
}
```

이 과정에서:

* 들여쓰기
* 줄바꿈
* 공백
* 세미콜론 위치

등이 적절히 반영됩니다.
(옵션에 따라 약간씩 formatting을 조정할 수 있음)

---

## 7. LazyResult / Result: 비동기 파이프라인 처리 ⚙️

아까 `processor.process()`의 반환값이 **LazyResult**라고 했죠?
실제로는 두 가지 패턴을 지원합니다.

### 7-1. 비동기 (추천)

```js
const result = await postcss([ plugins... ]).process(css, options)

console.log(result.css)
console.log(result.map.toString())
```

또는:

```js
postcss([ plugins ])
  .process(css, options)
  .then(result => {
    console.log(result.css)
  })
```

### 7-2. 동기

플러그인들이 모두 sync라면:

```js
const result = postcss([ plugins ]).process(css, options)
console.log(result.css)
```

> ⚠️ 어떤 플러그인이 async 작업(파일 I/O, network 등)을 하면 반드시 `await`을 써야 합니다.

### 7-3. Result 객체가 들고 있는 정보들

`Result` 객체에는:

* `css`: 최종 CSS 문자열
* `map`: Source Map 객체
* `root`: 최종 AST Root 노드
* `messages`: 플러그인들이 추가한 메시지(경고, 정보 등)
* `opts`: 옵션들

등이 들어 있습니다.

---

## 8. Source Map 처리 🗺️

빌드 툴 입장에서 매우 중요한 게 **Source Map**입니다.

* PostCSS는 입력 CSS의 위치 정보(줄, 칼럼)를 AST 노드에 저장해 둡니다.
* 문자열로 다시 만들 때도 이 정보를 사용해 **source map**을 생성합니다.

덕분에:

* devtools에서 **원본 파일 위치로 디버깅**이 가능하고
* 여러 번 변환(예: Sass → CSS → PostCSS → 번들링)을 거쳐도
  각각의 source map을 적절히 merge해서 최종 map을 만듭니다.

---

## 9. PostCSS Config 로딩: postcss.config.js 🔧

Vite, Webpack, CLI 등에서 PostCSS를 사용할 때 보통:

```js
// postcss.config.js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

이런 식으로 설정하죠.

내부적으로는:

1. **Config Loader**가 현재 작업 디렉터리에서

   * `postcss.config.js`, `postcss.config.cjs`, `postcss.config.mjs`, `postcss.config.json` 등을 찾음
2. `plugins` 필드를 읽어서

   * 객체 형태면 `{ tailwindcss: {}, autoprefixer: {} }`
   * 배열 형태면 `[ tailwindcss(), autoprefixer() ]`
3. 이를 **Processor에 넘길 플러그인 배열로 변환**합니다.

이렇게 만들어진 Processor를 나중에 `process()`에 사용합니다.

---

## 10. “플러그인들 사이의 순서”가 왜 중요한가? 🧱

PostCSS 파이프라인에서 **플러그인 순서**는 매우 중요합니다.

예를 들어:

```js
postcss([
  tailwindcss(),
  autoprefixer(),
])
```

이걸:

```js
postcss([
  autoprefixer(),
  tailwindcss(),
])
```

로 바꿔버리면?

* Autoprefixer는 Tailwind가 만들어주기 전에 **원본 CSS만 보고** prefix를 붙임
* Tailwind가 나중에 규칙들을 생성하지만, **이 새 규칙들에는 prefix가 없음**

즉, **실질적으로 Tailwind가 생성한 유틸리티에는 prefix가 안 붙는 결과**가 됩니다.
그래서 문서들이 항상:

> Tailwind → Autoprefixer 순서

를 강조하는 것입니다.

---

## 11. 아주 간단한 PostCSS 플러그인 만들어 보기 🛠️

전체 흐름을 잡기 위해, 정말 단순한 플러그인을 하나 만들어보겠습니다.

> 목표: 모든 `px` 단위를 `rem`으로 바꾸는 플러그인 (매우 단순 버전)

```js
// px-to-rem.js
export default function pxToRemPlugin(opts = { base: 16 }) {
  const base = opts.base ?? 16

  return {
    postcssPlugin: 'px-to-rem',
    Decl(decl) {
      if (!decl.value.includes('px')) return

      decl.value = decl.value.replace(
        /(\d+(\.\d+)?)px/g,
        (_, numStr) => {
          const num = parseFloat(numStr)
          const rem = num / base
          return `${rem}rem`
        }
      )
    },
  }
}
pxToRemPlugin.postcss = true
```

사용:

```js
import postcss from 'postcss'
import pxToRem from './px-to-rem.js'

const input = `
.btn {
  font-size: 16px;
  margin: 8px 0;
}
`

const result = await postcss([pxToRem({ base: 16 })]).process(input, {
  from: undefined,
})

console.log(result.css)
/*
.btn {
  font-size: 1rem;
  margin: 0.5rem 0;
}
*/
```

여기에서 일어난 일:

1. 입력 CSS → AST (파서)
2. `Decl` visitor가 모든 선언을 훑으면서 `value` 안의 `px`를 찾아 `rem`으로 변경 (플러그인 단계)
3. 수정된 AST → 문자열 CSS (stringifier)

딱 이 구조가 **PostCSS 내부 동작의 핵심**입니다.

---

## 12. 정리: PostCSS 내부 동작 한 번 더 요약하기 ✅

1. **Processor 생성**

   * 플러그인 배열을 들고 있는 실행기
2. **process() 호출**

   * LazyResult 생성 (아직 실행 X)
3. **LazyResult 소비( result.css, await )**

   * Parser가 CSS → AST로 파싱
   * 플러그인들을 순서대로 AST에 적용 (mutate)
   * Stringifier가 AST → 최종 CSS 문자열로 변환
   * Source Map까지 함께 생성
4. **Result 반환**

   * `css`, `map`, `root`, `messages` 등을 포함

Tailwind, Autoprefixer, Nesting, CSS Modules 등은
모두 이 **“AST 변환 플러그인”** 모델 위에서 돌아가는 친구들입니다. 😎

---

원하시면 다음 글로는:

* **“Tailwind + PostCSS + Vite” 전체 빌드 파이프라인 흐름** (엔트리 → Vite → PostCSS → Tailwind → Autoprefixer → 번들)
* **PostCSS AST 구조(노드 타입별 속성) 정리표 + 예제 AST 덤프**

같은 걸 “강의 자료용”으로 쫙 정리해 드리겠습니다.
