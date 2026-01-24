[![PostCSS まとめ #postcss - Qiita](https://tse2.mm.bing.net/th/id/OIP.t1a_rR7QO_xWxDcYUJchQwHaFj?pid=Api)](https://qiita.com/morishitter/items/4a04eb144abf49f41d7d?utm_source=chatgpt.com)

PostCSS AST는 “CSS를 이해하기 쉬운 자바스크립트 객체 그래프로 바꾼 구조”입니다.
Tailwind, Autoprefixer, nested 플러그인 등이 **전부 이 AST를 만지면서** CSS를 변신시키죠.

---

## 1. PostCSS 파이프라인 안에서 AST의 위치 🧬

위 그림처럼 PostCSS는 보통 이런 단계를 거칩니다:

1. **Parser**

   * 문자열 CSS → 토큰화(tokenize) → **AST 생성**
2. **Plugins (Transform 단계)**

   * 각 플러그인이 AST를 순회(walk)하며 노드를 추가/수정/삭제
3. **Stringifier**

   * 수정된 AST → 다시 CSS 문자열로 직렬화(stringify)

그 가운데 있는 “PostCSS AST”가 바로 오늘의 주인공입니다.
즉, 모든 플러그인은 “텍스트”를 직접 편집하는 게 아니라,
**CSS를 데이터 구조(트리)로 본 다음 그 트리를 조작**합니다.

---

## 2. AST란 뭔데, 왜 굳이 쓰는 거야? 🤔

일반적인 **AST(추상 구문 트리)** 개념부터 짚고 가면:

* “문자열로 작성된 코드/DSL을 **트리 구조**로 표현한 것”
* 괄호, 공백 같은 **문법적 잡음은 최대한 제거**하고,
* **의미(구조)** 에 집중해 표현한 트리

예를 들어 이런 CSS가 있다고 해보겠습니다:

```css
.button {
  color: red;
  margin-top: 12px;
}
```

이걸 AST로 바꾸면 대략 이런 계층이 됩니다(의사코드):

```json
{
  "type": "root",
  "nodes": [
    {
      "type": "rule",
      "selector": ".button",
      "nodes": [
        {
          "type": "decl",
          "prop": "color",
          "value": "red"
        },
        {
          "type": "decl",
          "prop": "margin-top",
          "value": "12px"
        }
      ]
    }
  ]
}
```

여기서 중요한 포인트:

* `.button { ... }` → **rule 노드**
* `color: red` → **decl 노드**
* 전체 파일 → **root 노드**

이렇게 바뀌면 플러그인이 해야 할 일은
`"margin-top"` 선언을 찾고, `"mt-3"` 같은 클래스로 바꾸거나,
벤더 프리픽스를 하나 더 추가하거나… 하는 **구조적 조작**이 됩니다.

문자열 뒤지면서 `indexOf('{')`, `split(';')` 이런 거 안 해도 되는 거죠. 👍

---

## 3. PostCSS AST의 핵심 노드 종류 정리 🌳

PostCSS AST의 노드는 모두 `Node`라는 공통 기반 클래스를 상속하며,
주요 타입은 다음과 같습니다.

### 3.1 Root 노드

* CSS 문서 전체를 대표
* `type: 'root'`
* `root.nodes` 아래에 rule, at-rule, comment 등이 나열

```js
Root {
  type: 'root',
  nodes: [ /* Rule, AtRule, Comment ... */ ]
}
```

### 3.2 Rule 노드 (일반 셀렉터 규칙)

`.btn { ... }`, `div > p { ... }` 같은 규칙 하나씩이 `Rule` 이 됩니다.

주요 필드:

* `type: 'rule'`
* `selector: '.btn'`
* `nodes: [ Declaration, Comment, Rule(중첩) ... ]`

예시:

```css
.btn {
  color: white;
  background: black;
}
```

→ AST:

```json
{
  "type": "rule",
  "selector": ".btn",
  "nodes": [
    { "type": "decl", "prop": "color", "value": "white" },
    { "type": "decl", "prop": "background", "value": "black" }
  ]
}
```

### 3.3 Declaration 노드 (속성 한 줄)

각 `color: red;` 같은 선언 하나가 `Declaration`입니다.

* `type: 'decl'`
* `prop: 'color'`
* `value: 'red'`
* `important: true/false` (`!important`인지 여부)

```css
.button {
  margin-top: 12px !important;
}
```

→ AST의 일부:

```json
{
  "type": "decl",
  "prop": "margin-top",
  "value": "12px",
  "important": true
}
```

### 3.4 AtRule 노드 (`@media`, `@import`, `@tailwind` 등)

`@media (min-width: 768px) { ... }` 같은 것들이 `AtRule`입니다.

* `type: 'atrule'`
* `name: 'media'`         // @ 뒤 이름
* `params: '(min-width: 768px)'`
* `nodes: [...]` (블록을 가지는 at-rule일 경우)

```css
@media (min-width: 768px) {
  .btn {
    padding: 1rem;
  }
}
```

→ AST 일부:

```json
{
  "type": "atrule",
  "name": "media",
  "params": "(min-width: 768px)",
  "nodes": [
    {
      "type": "rule",
      "selector": ".btn",
      "nodes": [
        { "type": "decl", "prop": "padding", "value": "1rem" }
      ]
    }
  ]
}
```

Tailwind의 `@tailwind base;`, `@tailwind components;` 도 결국 `atrule`입니다.

### 3.5 Comment 노드

```css
/* TODO: refactor */
```

* `type: 'comment'`
* `text: 'TODO: refactor'`

---

## 4. Node 공통 필드: parent, source, raws ✏️

PostCSS의 모든 노드는 공통적으로 이런 속성들을 가집니다.

### 4.1 parent

* 트리 상에서 자신의 부모 노드를 가리킵니다.
* 예: `decl.parent` → 해당 선언이 들어있는 `rule` 객체

이 덕분에:

```js
if (decl.parent.selector === '.btn') {
  // .btn 안에 있는 선언들에만 뭔가 하기
}
```

같은 로직이 쉬워집니다.

### 4.2 source (파일 위치 정보)

* 어느 파일/몇 번째 줄/열에서 온 것인지 위치 정보
* 소스맵(source map) 생성이나 에러 메시지를 위해 중요

```json
"source": {
  "start": { "line": 10, "column": 3 },
  "end":   { "line": 10, "column": 16 },
  "input": { "file": "src/styles.css" }
}
```

이 정보 덕분에 **lint 에러**가 정확히 어느 줄로 가리킬 수 있습니다.

### 4.3 raws (원래 문자열 포맷 유지용)

AST는 “추상적”이지만,
PostCSS는 포매터 역할도 해야 해서 **공백/개행/코멘트 위치**를 최대한 기억합니다.

```json
"raws": {
  "before": "\n  ",
  "between": ": "
}
```

* `before`: 이 선언 앞에 있던 공백/개행 등
* `between`: `prop`와 `value` 사이 원래 구분 문자열 (`": "` 등)

그래서 플러그인이 선언을 수정해도,
원래 코드 스타일을 어느 정도 그대로 유지할 수 있습니다.

---

## 5. PostCSS AST 직접 만져보는 예제 💻

아주 단순한 코드로 AST를 살짝 만져보겠습니다.

```js
import postcss from 'postcss'

const css = `
.button {
  color: red;
}
`

postcss([
  (root) => {
    // 1. 모든 선언(walkDecls)을 순회
    root.walkDecls('color', (decl) => {
      // 2. color: red -> blue 로 교체
      if (decl.value === 'red') {
        decl.value = 'blue'
      }

      // 3. 같은 rule 아래에 새로운 선언 추가
      decl.after({ prop: 'background-color', value: 'black' })
    })
  }
])
  .process(css, { from: undefined })
  .then((result) => {
    console.log(result.css)
  })
```

이 플러그인이 하는 일:

* AST의 모든 `decl` 중 `prop === 'color'` 인 노드를 찾는다.
* 값이 `red`면 `blue`로 바꾼다.
* 그 바로 뒤에 `background-color: black;` 선언을 추가한다.

**핵심:** `.after()` 같은 메서드는 내부적으로 AST의 siblings 배열을 조작합니다.
텍스트가 아니라 **노드 리스트**를이다루는 느낌이죠.

PostCSS는 이런 헬퍼를 많이 제공해서,

* `node.remove()`
* `node.cloneBefore()`
* `rule.append(...)`
* `root.prepend(...)`

처럼 트리를 편하게 바꿀 수 있습니다.

---

## 6. AST 생성 과정: 토큰 → 노드 🧩

내부적으로는 대략 이런 순서가 진행됩니다.

1. **Tokenizer**

   * CSS 문자열을 한 글자씩 읽으면서
   * `{`, `}`, `:`, `;`, `@`, 식별자, 공백 등을 토큰으로 분해
2. **Parser**

   * 토큰 스트림을 읽으며
   * `@` → AtRule, `{` → block 시작, `}` → block 종료 등
   * 규칙에 맞춰 Node 인스턴스 생성
3. **트리 구성**

   * 중첩 구조(예: `@media` 안에 rule, rule 안에 decl)를 parent/children 관계로 연결
4. **source/raws 세팅**

   * 각 노드에 위치/공백 정보 저장

이 전체 과정을 PostCSS가 대신 해주기 때문에,
플러그인 작성자는 “AST를 어떻게 변형하면 될까?”만 고민하면 됩니다.

---

## 7. AST 순회(walk) 패턴 정리 🚶‍♂️

PostCSS AST를 다룰 때 자주 쓰는 패턴은 다음과 같습니다.

```js
// 1. 모든 rule 순회
root.walkRules((rule) => {
  console.log('selector:', rule.selector)
})

// 2. 특정 속성을 가진 선언만
root.walkDecls('color', (decl) => {
  console.log('color value:', decl.value)
})

// 3. @media 같은 AtRule만
root.walkAtRules('media', (atrule) => {
  console.log('media query:', atrule.params)
})
```

내부적으로는 DFS 방식으로 트리를 모두 돌면서,
매칭되는 타입/조건의 노드에 대해서만 콜백을 실행합니다.

실전 플러그인은 이 walk 계열 API 위에 거의 다 서 있습니다.

---

## 8. Tailwind + PostCSS에서 AST가 어떻게 쓰이는지 🔧

지금 선생님께서 진행 중이신 **Vite + React + Tailwind** 환경에서도
PostCSS AST는 뒤에서 열심히 일하고 있습니다.

대표적인 흐름을 하나 그려보면:

1. 개발자가 `class="bg-blue-500 hover:bg-blue-700"` 같이 작성
2. Tailwind는 **컴파일 단계에서**

   * JSX/TS/HTML에서 class 문자열을 추출
   * “필요한 유틸리티 클래스 목록”을 만든 뒤,
3. PostCSS AST 위에 Tailwind 플러그인으로

   * 각 유틸리티를 **실제 CSS 규칙(Rule/Decl) 노드로 삽입**
   * 예: `.bg-blue-500 { background-color: #3b82f6; }` rule 노드를 AST에 추가
4. 마지막에 Stringifier가 AST를 다시 CSS로 바꿔서 브라우저로 전달

즉 Tailwind 플러그인은

* “어떤 `Rule`을 만들지”
* “어떤 `Decl`을 넣을지”
* “어떤 `AtRule`(@media, @supports 등) 안에 넣을지”

를 전부 PostCSS AST 단계에서 제어합니다.

---

## 9. 강의에 바로 쓸 수 있는 요약 슬라이드용 정리 📚

마지막으로, 수업 슬라이드 한 장에 넣을 수 있는 Bullet 버전으로 정리해보면:

### 🧱 PostCSS AST란?

* CSS를 **Root → Rule → Declaration/AtRule → ...** 형태의 트리로 표현한 것
* 플러그인은 이 트리를 **순회/변형**하면서 CSS를 가공한다.
* 문자열 편집 대신 구조 조작을 하므로, 안정적이고 확장성이 좋다.

### 🌲 주요 노드 타입

* `Root`: 문서 전체
* `Rule`: `.btn { ... }` 같은 셀렉터 블록
* `Declaration`: `color: red` 한 줄
* `AtRule`: `@media`, `@import`, `@tailwind` 등
* `Comment`: `/* ... */`

### 🧩 공통 속성

* `parent`: 부모 노드
* `nodes`: 자식 노드 배열
* `source`: 파일/줄/열 정보(소스맵, 에러 위치용)
* `raws`: 공백/개행/콜론 주변 문자열(포맷 유지용)

### 🛠️ AST를 다루는 대표 API

* `root.walkRules()`, `root.walkDecls()`, `root.walkAtRules()`
* `node.remove()`, `node.after()`, `rule.append()`, `root.prepend()`


