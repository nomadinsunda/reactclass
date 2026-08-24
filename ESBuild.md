# esbuild

## — 초고속 JavaScript 빌드 도구의 내부 구조와 동작 원리

> **먼저 짚고 갈 점**
>
> esbuild는 단순히 “Webpack보다 빠른 번들러”가 아닙니다.
>
> esbuild가 JavaScript 도구 생태계에 남긴 가장 큰 영향은
>
> **“컴파일러·번들러 자체를 JavaScript가 아닌 네이티브 코드로 구현하고, 처음부터 병렬 처리와 메모리 효율을 고려하면 빌드 속도를 극적으로 끌어올릴 수 있다”**
>
> 는 것을 실제 도구로 보여준 것입니다.

이후 SWC, Oxc, Rolldown, Bun 등은 서로 다른 언어와 아키텍처를 사용하지만, **JavaScript 개발 도구 자체의 실행 성능을 중요하게 보기 시작했다는 점**에서는 esbuild의 영향을 받은 흐름이라고 볼 수 있습니다.

---

# 1. esbuild란?

**esbuild는 Go로 작성된 고성능 JavaScript/TypeScript 빌드 도구입니다.**

하나의 도구에서 다음 기능을 제공합니다.

```text
JavaScript / TypeScript / JSX / TSX
                  │
                  ▼
              esbuild
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
   Transform    Bundle      Minify
       │          │           │
       └──────────┼───────────┘
                  ▼
             JavaScript
```

대표적으로 다음 작업을 수행할 수 있습니다.

* JavaScript / TypeScript 파싱
* JSX / TSX 변환
* 최신 JavaScript 구문 변환
* ESM / CommonJS 처리
* 모듈 의존성 분석
* Bundling
* Tree Shaking
* Minification
* Source Map 생성
* Code Splitting
* CSS Bundling
* Plugin API 제공

즉 esbuild는 단순한 번들러라기보다

> **Parser + Transformer + Bundler + Optimizer + Code Generator**

가 하나의 프로그램에 통합된 빌드 도구라고 보는 것이 좋습니다.

---

# 2. 왜 esbuild가 등장했을까?

기존 JavaScript 빌드 도구의 상당수는 JavaScript 자체로 작성되어 있었습니다.

예를 들면:

```text
Webpack  → JavaScript
Rollup   → JavaScript
Terser   → JavaScript
Babel    → JavaScript
```

이것 자체가 잘못된 것은 아닙니다.

그러나 CLI 빌드 도구에서는 다음과 같은 비용이 발생할 수 있습니다.

```text
빌드 명령 실행
      │
      ▼
Node.js 시작
      │
      ▼
빌드 도구의 JavaScript 코드 로딩
      │
      ▼
JIT 준비 / 실행
      │
      ▼
사용자 JavaScript 파싱
      │
      ▼
실제 빌드
```

반면 esbuild는 Go로 컴파일된 **native executable**입니다.

```text
빌드 명령 실행
      │
      ▼
Native esbuild 실행
      │
      ▼
즉시 Parsing / Linking / Code Generation
```

esbuild 공식 문서도 빠른 이유 중 하나로 **Go로 작성되어 native code로 컴파일된다는 점**을 명시하고 있습니다. ([esbuild][1])

---

# 3. esbuild가 빠른 진짜 이유

단순히

```text
Go라서 빠르다
```

라고 설명하면 부족합니다.

esbuild의 속도는 여러 설계가 결합된 결과입니다.

```text
┌──────────────────────────────────────┐
│            esbuild가 빠른 이유        │
├──────────────────────────────────────┤
│ ① Native Code                       │
│ ② 강력한 Parallelism                │
│ ③ 자체 Parser                       │
│ ④ 적은 AST Pass                     │
│ ⑤ 메모리 효율적인 Data Structure     │
│ ⑥ 중간 표현 변환 최소화              │
│ ⑦ 전체 시스템을 처음부터 직접 구현   │
└──────────────────────────────────────┘
```

---

# 4. 이유 ① — Go + Native Code

esbuild는 Go로 작성됩니다.

```text
esbuild source
      │
      │ Go Compiler
      ▼
Native Executable
```

따라서 Node.js 위에서 실행되는 JavaScript 프로그램과 달리 빌드 도구 자체를 실행하기 위해 JavaScript 엔진이 빌드 도구의 소스를 파싱하고 JIT 최적화하는 과정이 필요하지 않습니다.

```text
JavaScript Build Tool

source
  ↓
Node.js
  ↓
Parse
  ↓
JIT
  ↓
Execution


esbuild

Native Binary
     ↓
Execution
```

특히 짧게 실행되고 종료되는 CLI 프로그램에서는 이러한 시작 비용도 의미가 있습니다.

---

# 5. 이유 ② — 멀티코어 병렬 처리

esbuild는 **CPU의 여러 코어를 적극적으로 활용하도록 설계**되어 있습니다.

Go는 goroutine을 통해 동시성 작업을 쉽게 구성할 수 있고, esbuild는 이를 활용합니다.

하지만 다음과 같이 이해하면 안 됩니다.

```text
8 Core CPU
   ↓
정확히 8개의 파일을 동시에 처리
```

실제 처리는 훨씬 복잡합니다.

파일과 작업의 종류에 따라 병렬화 가능한 부분이 달라집니다.

esbuild 공식 설명에서는 전체 작업을 대략 다음과 같이 구분합니다.

```text
Parsing
   ↓
Linking
   ↓
Code Generation
```

이 가운데 특히:

```text
Parsing         → 병렬화하기 좋음
Code Generation → 병렬화하기 좋음
Linking         → 상대적으로 직렬 작업이 많음
```

입니다. ([esbuild][1])

예를 들어:

```text
              CPU Core
       ┌────┬────┬────┬────┐
       │ C1 │ C2 │ C3 │ C4 │
       └─┬──┴─┬──┴─┬──┴─┬──┘
         │    │    │    │
       A.js B.js C.js D.js
         │    │    │    │
         └────┴────┴────┘
                │
              Linking
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     chunk1   chunk2   chunk3
```

와 같은 형태로 이해할 수 있습니다.

---

# 6. 이유 ③ — Parser를 직접 만들었다

esbuild는 Babel이나 TypeScript Compiler의 parser를 그대로 사용하지 않습니다.

JavaScript, TypeScript, JSX 등을 처리하는 parser를 자체적으로 구현했습니다.

```text
JavaScript
TypeScript
JSX
TSX
   │
   ▼
┌─────────────────┐
│ esbuild Parser  │
└────────┬────────┘
         ▼
        AST
```

이것의 중요한 장점은 **전체 파이프라인의 데이터 구조를 esbuild 자체에 맞게 최적화할 수 있다는 것**입니다.

다른 도구들을 조합하면 다음과 같은 일이 발생할 수 있습니다.

```text
Source
  ↓
Parser A
  ↓
AST A
  ↓
String
  ↓
Parser B
  ↓
AST B
  ↓
String
  ↓
Parser C
  ↓
AST C
```

각 단계마다

* parsing
* serialization
* allocation
* AST 변환

비용이 추가됩니다.

esbuild는 많은 작업을 하나의 내부 표현에서 처리하려고 합니다.

---

# 7. 이유 ④ — AST를 여러 번 왕복하지 않는다

esbuild의 중요한 설계 특징 중 하나입니다.

공식 설명에 따르면 JavaScript AST 전체를 크게 **세 번 정도 순회하는 구조**로 많은 작업을 결합합니다. ([esbuild][1])

개념적으로:

```text
PASS 1
────────────────────────
Lexing
Parsing
Scope Setup
Symbol Declaration


PASS 2
────────────────────────
Symbol Binding
Syntax Minification
TypeScript → JavaScript
JSX → JavaScript
Syntax Lowering


PASS 3
────────────────────────
Identifier Minification
Whitespace Minification
Code Generation
Source Map Generation
```

즉,

```text
Parse
 ↓
AST
 ↓
Transform
 ↓
새 AST
 ↓
Minifier
 ↓
또 다른 AST
 ↓
Generator
```

식으로 각 도구 사이에서 자료구조를 반복적으로 변환하는 방식을 피합니다.

---

# 8. CPU Cache까지 고려한 구조

컴파일러에서 중요한 성능 요소 중 하나는 **메모리 접근**입니다.

CPU가 데이터를 계속 RAM에서 가져오는 것보다 캐시에 존재하는 데이터를 재사용하는 것이 훨씬 빠릅니다.

esbuild는 AST를 여러 다른 형식으로 바꾸지 않고 가능한 한 같은 자료구조를 반복 사용합니다.

```text
AST
 │
 ├── Parse
 │
 ├── Transform
 │
 ├── Minify
 │
 └── Generate
```

따라서 데이터가 CPU cache에 남아 있는 동안 다음 작업을 수행할 가능성이 높아집니다.

```text
        CPU
         │
         ▼
    ┌─────────┐
    │ L1/L2   │
    │ Cache   │
    │         │
    │   AST   │
    └─────────┘
```

이런 작은 최적화들이 합쳐지면서 큰 성능 차이가 발생합니다.

---

# 9. esbuild의 실제 내부 처리 흐름

교육적으로 다음과 같이 설명할 수 있습니다.

```text
Source Files
     │
     ▼
① Parse
     │
     ▼
    AST
     │
     ▼
② Transform / Analyze
     │
     ▼
③ Link
     │
     ▼
④ Code Generation
     │
     ▼
Output
```

다만 이것은 **개념적 흐름**입니다.

실제 구현에서는 Transformation, Minification, Tree Shaking 등의 여러 작업이 서로 완전히 독립적인 단계로 한 번씩 실행되는 것이 아니라 여러 AST pass에 결합되어 있습니다.

---

# 10. ① Parsing

먼저 소스 코드를 읽어 AST(Abstract Syntax Tree)를 생성합니다.

예를 들어:

```javascript
const result = add(10, 20);
```

개념적으로:

```text
VariableDeclaration
        │
        ├── Identifier: result
        │
        └── CallExpression
                │
                ├── Identifier: add
                ├── Literal: 10
                └── Literal: 20
```

이 과정에서 다음 정보도 함께 처리됩니다.

```text
import / export
scope
identifier
JS / TS syntax
JSX / TSX
```

---

# 11. ② Transform

AST를 바탕으로 필요한 구문 변환이 이루어집니다.

예:

```javascript
const result = user?.profile?.name;
```

설정된 target에 따라 이전 문법 형태로 변환될 수 있습니다.

또한:

```tsx
const element = <h1>Hello</h1>;
```

같은 JSX도 JavaScript로 변환할 수 있습니다.

TypeScript의 타입 정보도 제거합니다.

```typescript
const count: number = 10;
```

↓

```javascript
const count = 10;
```

---

# 12. 아주 중요한 점 — esbuild는 TypeScript 타입 검사를 하지 않는다

esbuild는 TypeScript 문법을 이해하지만 **Type Checker가 아닙니다.**

다음 코드가 있어도:

```typescript
const count: number = "hello";
```

esbuild의 핵심 역할은 타입 annotation을 제거하는 것입니다.

```javascript
const count = "hello";
```

즉:

```text
TypeScript Source
      │
      ▼
   esbuild
      │
      ├── Type Syntax Parse
      │
      ├── Type Annotation 제거
      │
      └── JavaScript 생성
      │
      ▼
JavaScript
```

그러나:

```text
number인데 string을 넣었는가?
```

를 검사하지 않습니다.

esbuild 공식 문서도 TypeScript를 파싱하고 타입 annotation을 버리지만 **type checking은 하지 않는다**고 명시합니다. ([esbuild][2])

따라서 별도의:

```bash
tsc --noEmit
```

등을 사용해 타입 검사를 수행할 수 있습니다.

---

# 13. 왜 TypeScript 파일을 독립적으로 처리할까?

esbuild는 TypeScript 변환 시 각 파일을 가능한 한 독립적으로 처리할 수 있도록 설계되어 있습니다.

```text
A.ts ──→ Transform
B.ts ──→ Transform
C.ts ──→ Transform
D.ts ──→ Transform
```

이 구조는 병렬 처리에 유리합니다.

하지만 TypeScript compiler의 완전한 type checking은 프로그램 전체의 타입 관계를 추적해야 합니다.

```text
A.ts ─┐
      │
B.ts ─┼──→ Type Graph
      │
C.ts ─┼──→ Cross-file Analysis
      │
D.ts ─┘
```

그래서 esbuild를 사용하는 TypeScript 프로젝트에서는 `isolatedModules`와 호환되는 코드를 작성하는 것이 중요합니다. 공식 문서도 파일별 독립 변환 때문에 `isolatedModules` 사용을 권장합니다. ([esbuild][2])

하지만 다음처럼 단정하는 것은 피하는 것이 좋습니다.

> “esbuild가 빠른 이유가 곧 타입 검사를 못 하는 이유다.”

더 정확하게는:

> **esbuild는 빠른 변환을 목표로 TypeScript의 전체 타입 분석을 자신의 책임 범위에서 제외했고, 파일 단위 변환이 가능한 구조를 선택했습니다.**

입니다.

---

# 14. JSX / TSX도 자체 처리한다

React 프로젝트에서 매우 중요한 기능입니다.

```jsx
const element = <button>Click</button>;
```

esbuild는 JSX를 파싱하고 설정에 따라 일반 JavaScript 호출 형태로 변환할 수 있습니다.

```text
JSX
 │
 ▼
esbuild Parser
 │
 ▼
AST
 │
 ▼
JSX Transform
 │
 ▼
JavaScript
```

따라서 Babel을 반드시 거쳐야만 JSX를 처리할 수 있는 것은 아닙니다.

---

# 15. JavaScript Target 변환

esbuild는 target 옵션을 통해 최신 문법을 특정 JavaScript 환경에 맞게 변환할 수 있습니다.

예:

```bash
esbuild app.js --target=es2017
```

또는:

```bash
esbuild app.js --target=chrome100
```

예를 들어:

```javascript
const name = user?.profile?.name;
```

대상 브라우저가 optional chaining을 지원하지 않으면 더 오래된 문법으로 변환할 수 있습니다.

하지만 여기서 매우 중요한 제한이 있습니다.

---

# 16. esbuild는 ES5 Transpiler가 아니다

기존 설명의:

```text
JS → ES5
```

는 수정하는 것이 좋습니다.

esbuild 공식 문서는 명확하게:

> **ES6+ syntax를 ES5로 완전히 변환하는 것은 잘 지원하지 않는다**

고 설명합니다. ([esbuild][2])

따라서:

```bash
--target=es5
```

를 지정한다고 해서 Babel처럼 모든 최신 JavaScript를 완벽하게 ES5로 변환해 준다고 생각하면 안 됩니다.

더 정확한 설명은:

```text
Modern JavaScript
       │
       ▼
     esbuild
       │
       ▼
지원 가능한 범위 안에서
target 환경에 맞게 syntax lowering
```

입니다.

또한 esbuild의 `target`은 주로 **syntax 변환**에 관한 것입니다.

예를 들어:

```javascript
fetch()
Promise
Array.prototype.flat()
```

같은 API를 자동으로 polyfill해 주지는 않습니다. ([esbuild][3])

---

# 17. Bundling

번들링을 활성화하면 entry point부터 import를 따라갑니다.

```text
main.js
 │
 ├── App.js
 │     ├── Header.js
 │     └── Main.js
 │
 └── api.js
       └── http.js
```

esbuild는 이를 이용해 **Module Dependency Graph**를 구성합니다.

```text
               main.js
              /      \
          App.js     api.js
          /   \        |
    Header.js Main.js http.js
```

그리고 필요한 모듈을 결과물에 포함합니다.

```text
Dependency Graph
       │
       ▼
     Link
       │
       ▼
   Output Chunk
```

---

# 18. Linking

esbuild 공식 설명에서 중요한 내부 단계가 바로 **Linking**입니다.

Parsing이 각각의 파일을 이해하는 과정이라면, Linking은 파일 사이의 관계를 연결하는 과정입니다.

```text
A.js
 export foo
    │
    │
    ▼
B.js
 import foo
```

Linker는 개념적으로 다음을 처리합니다.

```text
Module Graph
   │
   ├── imports
   ├── exports
   ├── symbol references
   ├── module boundaries
   └── chunks
```

이 과정은 다른 파일과의 관계를 확인해야 하기 때문에 Parsing처럼 완전히 독립적으로 병렬 처리하기 어렵습니다.

---

# 19. Tree Shaking

예를 들어:

```javascript
// math.js

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}
```

```javascript
// app.js

import { add } from "./math.js";

console.log(add(10, 20));
```

`multiply()`가 사용되지 않는다면 조건이 맞을 경우 출력에서 제거할 수 있습니다.

```text
math.js

add()        ─────→ USED ─────→ OUTPUT
multiply()   ─────→ UNUSED ───→ REMOVE
```

이것이 Tree Shaking입니다.

Tree Shaking은 특히 ESM의 정적인 import/export 구조를 분석하는 데 유리합니다.

---

# 20. Minification

esbuild는 자체 Minifier도 포함합니다.

```javascript
function calculateTotal(itemList) {
  let total = 0;

  for (const item of itemList) {
    total += item.price;
  }

  return total;
}
```

Minify 후에는 상황에 따라 더 짧은 형태가 생성됩니다.

```javascript
function a(t){let e=0;for(const n of t)e+=n.price;return e}
```

esbuild의 minification에는 대표적으로 다음이 있습니다.

```text
Whitespace Minification
Identifier Minification
Syntax Minification
```

즉:

```text
공백 제거
      +
식별자 이름 축약
      +
더 짧은 동등 표현으로 변환
```

을 수행합니다. ([esbuild][3])

---

# 21. Minify와 Obfuscation은 다르다

매우 중요한 개념입니다.

```text
Minification
      ≠
Obfuscation
```

Minification의 목적:

```text
파일 크기 감소
전송량 감소
파싱 비용 감소
```

Obfuscation의 목적:

```text
코드 분석을 어렵게 만들기
```

변수명이:

```javascript
calculateTotal
```

에서:

```javascript
a
```

로 변경되었다고 해서 그것이 보안 기능은 아닙니다.

브라우저에 전달된 JavaScript는 결국 사용자가 받을 수 있기 때문에 클라이언트 JavaScript를 완전히 숨기는 것은 불가능합니다.

---

# 22. esbuild Minifier에 대한 중요한 오해

기존 설명에:

```text
inlining
```

을 일반적인 esbuild Minifier 기능으로 넣는 것은 조심해야 합니다.

esbuild 공식 문서는 Minifier가 **고급 최적화를 목표로 하지 않는다**고 설명하며, 일반적인 function inlining 등 여러 aggressive optimization은 수행하지 않는다고 명시합니다. ([esbuild][3])

즉 esbuild의 목표는:

```text
최고 수준의 압축률
```

보다는:

```text
매우 빠른 속도
      +
충분히 좋은 압축 결과
```

에 가깝습니다.

---

# 23. Code Generation

마지막에는 분석과 변환이 끝난 AST를 다시 JavaScript 코드로 생성합니다.

```text
AST
 │
 ├── Identifier Renaming
 ├── Whitespace Minification
 ├── Source Map
 │
 ▼
Generated JavaScript
```

이 단계 역시 esbuild에서 병렬화하기 좋은 작업 중 하나입니다.

---

# 24. 전체 내부 구조

esbuild를 이해할 때 가장 좋은 전체 그림은 다음과 같습니다.

```text
                  Entry Points
                       │
                       ▼
              ┌─────────────────┐
              │     Parsing     │
              │                 │
              │ JS / TS / JSX   │
              │ Scope Analysis  │
              │ Symbol Declare  │
              └────────┬────────┘
                       │
                       ▼
                     AST
                       │
              ┌────────┴────────┐
              │                 │
       Syntax Transform    Type Removal
              │                 │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │     Linking     │
              │                 │
              │ Module Graph    │
              │ Import / Export │
              │ Tree Shaking    │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Code Generation │
              │                 │
              │ Minification    │
              │ Source Maps     │
              └────────┬────────┘
                       │
                       ▼
               JavaScript Output
```

---

# 25. 실전 사용 예

```bash
esbuild src/main.ts \
  --bundle \
  --minify \
  --sourcemap \
  --outfile=dist/main.js
```

개념적으로:

```text
main.ts
  │
  ├── import A
  ├── import B
  └── import C
       │
       ▼
    esbuild
       │
       ├── Parse
       ├── TS Type 제거
       ├── Module Graph
       ├── Link
       ├── Tree Shake
       ├── Minify
       └── Source Map
       │
       ▼
dist/main.js
dist/main.js.map
```

---

# 26. esbuild가 빠르다는 것은 어느 정도인가?

esbuild 공식 benchmark에서는 특정 테스트 조건에서 기존 JavaScript 기반 번들러보다 수십 배 이상 빠른 결과를 보여줍니다.

예를 들어 공식 JavaScript benchmark의 한 측정에서는:

```text
esbuild             0.39s
Parcel 2           14.91s
Rollup + Terser    34.10s
Webpack 5          41.21s
```

가 기록되어 있습니다. ([esbuild][1])

그러나 이를:

> “esbuild는 항상 다른 모든 도구보다 10~100배 빠르다.”

라고 일반화하면 안 됩니다.

성능은 다음에 따라 달라집니다.

```text
프로젝트 크기
플러그인
캐시
빌드 설정
CPU
I/O
Code Splitting
Minification 방식
```

따라서 더 정확한 표현은:

> **esbuild는 특정 빌드 작업에서 기존 JavaScript 기반 도구보다 압도적으로 빠른 성능을 보여주면서 빌드 도구 성능의 기준을 크게 끌어올린 도구입니다.**

입니다.

---

# 27. esbuild vs Webpack vs Rollup vs SWC

단순한 “누가 가장 빠르다” 식의 비교보다는 **역할을 구분하는 것이 중요합니다.**

| 도구       | 구현         | 핵심 역할                             |
| -------- | ---------- | --------------------------------- |
| esbuild  | Go         | Transform + Bundle + Minify       |
| Webpack  | JavaScript | 범용 애플리케이션 Bundler                 |
| Rollup   | JavaScript | Bundler, 특히 라이브러리/최적화된 출력         |
| SWC      | Rust       | Compiler / Transformer / Minifier |
| Oxc      | Rust       | Parser / Transformer / Minifier 등 |
| Rolldown | Rust       | Bundler                           |

이 도구들은 기능이 일부 겹치지만 정확히 동일한 범주의 제품은 아닙니다.

---

# 28. Vite와 esbuild

esbuild가 많은 프론트엔드 개발자에게 알려진 가장 큰 이유 중 하나가 **Vite**입니다.

Vite 7까지의 대표적인 구조는 크게:

```text
               Vite
                 │
        ┌────────┴────────┐
        │                 │
   Development        Production
        │                 │
        ▼                 ▼
     esbuild            Rollup
```

형태였습니다.

esbuild는 특히:

```text
Dependency Pre-Bundling
TypeScript Transform
JSX Transform
```

등에서 빠른 개발 경험을 제공했습니다.

---

# 29. 그러나 Vite 8에서는 구조가 바뀌었다

2026년 3월 12일 **Vite 8이 정식 출시**되면서 중요한 변화가 일어났습니다. ([vitejs][4])

기존:

```text
Vite 7

Development
     │
 esbuild

Production
     │
 Rollup
```

에서:

```text
Vite 8

          Vite
            │
            ▼
        Rolldown
            │
            ▼
           Oxc
```

중심 구조로 통합되었습니다.

Vite 공식 문서에 따르면 Vite 8은 **Rolldown을 단일 Rust 기반 번들러로 사용**합니다. ([vitejs][4])

또한 migration 문서에서는:

> esbuild는 더 이상 Vite가 직접 사용하는 필수 엔진이 아니며 optional dependency가 되었다.

고 설명합니다. ([vitejs][5])

---

# 30. Vite가 왜 esbuild를 떠났을까?

이것을:

```text
esbuild가 느려져서
```

라고 이해하면 완전히 잘못된 것입니다.

오히려 핵심은 **개발과 프로덕션에서 서로 다른 번들러를 사용하던 구조를 하나로 통합하기 위해서**입니다.

기존:

```text
DEV                         BUILD

Vite                         Vite
 │                            │
 ▼                            ▼
esbuild                    Rollup
```

문제는 두 시스템이 서로 다른:

```text
Parser
Resolver
Transform
Bundling behavior
```

를 가지고 있다는 점입니다.

Vite 8에서는:

```text
                 Vite
                   │
                   ▼
               Rolldown
                   │
                   ▼
                  Oxc
```

라는 통합된 Rust 기반 toolchain을 지향합니다. ([vitejs][6])

---

# 31. 그렇다면 esbuild는 끝난 기술인가?

전혀 아닙니다.

Vite에서 기본 경로가 변경된 것과 esbuild 자체의 가치가 사라지는 것은 다른 문제입니다.

esbuild는 여전히 다음과 같은 용도에 적합합니다.

```text
빠른 TypeScript Transform

JSX / TSX Transform

CLI Bundling

Node.js 코드 Bundling

Library Build Tool 내부 엔진

테스트 도구의 코드 변환

간단한 Production Build

JavaScript Minification
```

특히:

```text
“복잡한 확장성보다
 빠른 JavaScript 변환이 중요하다”
```

는 상황에서 강력한 도구입니다.

---

# 32. esbuild의 진짜 유산

esbuild의 가치를 단순히 현재 market share로 평가하면 핵심을 놓칩니다.

esbuild 이전의 대표적인 JavaScript toolchain은 대체로:

```text
JavaScript
    │
    ▼
Node.js
    │
    ▼
JavaScript Build Tool
```

중심이었습니다.

esbuild 이후에는:

```text
        JavaScript Tooling

              │
      ┌───────┼─────────┐
      ▼       ▼         ▼
      Go     Rust       Zig
      │       │          │
      ▼       ▼          ▼
  esbuild   SWC/Oxc     Bun
               │
               ▼
            Rolldown
```

처럼 네이티브 언어로 구현된 고성능 도구들이 중요한 흐름으로 자리 잡았습니다.

따라서 esbuild의 가장 중요한 역사적 의의는:

> **JavaScript 개발 도구도 컴파일러 수준의 성능 설계를 적용할 수 있다는 것을 대중적으로 증명했다는 점**

이라고 볼 수 있습니다.

---

# 33. 전체 흐름 한눈에 보기

```text
              JavaScript / TypeScript
                        │
                        ▼
                 ┌─────────────┐
                 │   esbuild   │
                 └──────┬──────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Parsing     Linking   Code Generation
             │          │          │
       Scope 분석   Module Graph   Minify
       TS 제거      Import/Export  Source Map
       JSX 변환     Tree Shaking   Output
             │          │          │
             └──────────┼──────────┘
                        ▼
                   JavaScript
```

그리고 성능의 핵심은:

```text
Native Go Binary
       +
Multi-Core Parallelism
       +
Custom Parser
       +
Few AST Passes
       +
Memory Efficient Structures
       +
Intermediate Conversion 최소화
       =
Very Fast Build
```

입니다.

---

# 핵심 정리

esbuild를 단순히:

> **“Go로 만들어서 빠른 번들러”**

라고 설명하면 핵심의 절반밖에 설명하지 못합니다.

esbuild는 처음부터 **컴파일러 전체 파이프라인을 속도 중심으로 설계한 JavaScript build tool**입니다.

```text
Go Native Code
      ↓
Custom Parser
      ↓
Parallel Parsing
      ↓
AST 재사용
      ↓
Transform
      ↓
Module Linking
      ↓
Tree Shaking
      ↓
Parallel Code Generation
      ↓
Minification
      ↓
Output
```

그리고 가장 중요한 점은:

> **esbuild의 속도는 특정 한 가지 기술 때문이 아니라 Native Code, 병렬 처리, 자체 Parser, 적은 AST 순회, 효율적인 메모리 구조, 중간 변환 최소화가 결합된 결과입니다.**

Vite 8에서는 기본 엔진의 자리를 Rolldown/Oxc에게 넘겼지만, 이것은 esbuild가 실패했다는 의미가 아닙니다.

오히려:

```text
esbuild
   ↓
“JavaScript Tooling도 훨씬 빨라질 수 있다”
   ↓
SWC
   ↓
Oxc
   ↓
Rolldown
   ↓
현대 Native JavaScript Toolchain
```

이라는 흐름을 이해하는 출발점으로 보는 것이 좋습니다.

**esbuild를 이해하면 오늘날의 SWC, Oxc, Rolldown이 왜 등장했는지도 자연스럽게 이해할 수 있습니다.**

[1]: https://esbuild.github.io/faq/?utm_source=chatgpt.com "esbuild - FAQ"
[2]: https://esbuild.github.io/content-types/?utm_source=chatgpt.com "esbuild - Content Types"
[3]: https://esbuild.github.io/api/?utm_source=chatgpt.com "esbuild - API"
[4]: https://vite.dev/blog/announcing-vite8?utm_source=chatgpt.com "Vite 8.0 is out! | Vite"
[5]: https://vite.dev/guide/migration?utm_source=chatgpt.com "Migration from v7 | Vite"
[6]: https://vite.dev/blog/announcing-vite8-beta?utm_source=chatgpt.com "Vite 8 Beta: The Rolldown-powered Vite | Vite"
