# ESBuild: 초고속 번들러의 내부 구조와 동작 원리

*— 왜 "네이티브 언어로 만든 빌드 도구"가 표준이 되었는가?*

> **먼저 짚고 갈 점**: esbuild는 "모든 도구가 내부적으로 쓰는 엔진"이 아닙니다.
> esbuild가 증명한 것은 **"빌드 도구를 JS가 아닌 네이티브 언어로 만들면 10\~100배 빨라진다"** 는 명제이고,
> 이후 도구들은 각자 다른 네이티브 구현(Rust의 SWC/Oxc/Rolldown, Zig의 Bun)으로 갈라졌습니다.
> 이 문서는 그 출발점이 된 esbuild의 설계를 다룹니다.

```plaintext
Parsing → Transform → Bundling → Minify/Tree-shaking
```

> 위 흐름이 esbuild가 수행하는 주요 파이프라인입니다.
> 이 네 단계를 초고속으로 수행하는 것이 핵심이며, 각 단계는 아래 4장에서 자세히 다룹니다.

---

# 1. ESBuild란 무엇인가?

**ESBuild는 Go 언어로 작성된 초고속 JavaScript/TypeScript 번들러이자 트랜스파일러입니다.**

기존 JS 번들러(Webpack, Rollup, Parcel)의 구조적 병목을 해결해 **10~100배 이상의 속도 향상**을 실현했고, 이후 등장한 네이티브 빌드 도구들의 출발점이 되었습니다.

### ESBuild가 유명해진 이유

* **경이적인 속도** (멀티스레드 + Go 언어 성능 기반)
* **Native-level 성능의 Parser**
* **TypeScript, JSX, TSX 내장 지원**
* **Tree-shaking & Minify 내장**
* **플러그인 시스템 지원**
* **Vite 7까지의 내부 트랜스파일 엔진으로 채택** (Bun·Turbopack은 esbuild가 아니라 각자의 자체 엔진을 씁니다 → 7장 참고)

---

# 2. ESBuild의 아키텍처: 왜 이렇게 빠른가?

ESBuild의 성능은 **아키텍처 자체가 빠르도록 설계되어 있기 때문**입니다.
다음은 그 핵심 요소입니다.

---

## 2-1. Go 기반의 멀티스레드 구조

JavaScript 번들러 대부분은 JS 기반이므로 **싱글 스레드 병목**이 발생합니다.

그러나 ESBuild는 **Go 언어의 고루틴(Goroutine) 기반**으로 CPU 코어 수만큼 병렬 처리를 수행합니다.

→ 즉, 8코어 CPU라면 8개의 파일을 동시에 파싱/트랜스폼/번들링합니다.

---

## 2-2. 자체 개발한 초고속 파서(Parser)

ESBuild는 **JavaScript/TypeScript/JSX 파서를 직접 구현**했습니다.

특징:

* AST 변환이 매우 빠름
* Babel보다 수십 배 빠르게 JS/TS를 변환
* JIT(Just-in-time) 없이도 네이티브 속도

---

## 2-3. 디스크 I/O 최소화 & 메모리 기반 처리

Webpack은 다음과 같은 오버헤드를 가집니다.

*파일 읽기 → Babel 변환 → 플러그인 처리 → 캐싱 → 디스크 재쓰기*

반면 ESBuild는:

* 메모리 기반 변환
* 디스크 접근 최소화
* 빠른 AST 캐싱

결과적으로 빌드 시간이 대폭 단축됩니다.

---

# 3. ESBuild의 주요 기능 정리

---

## 3-1. 번들링(Bundling)

ESBuild는 ES Modules을 기본으로 번들링합니다.

* import graph 분석
* 필요 모듈만 포함
* dead code 삭제(Tree shaking)

---

## 3-2. Transpile (JS → ES5 등)

Babel 없이도 다음 기능을 지원합니다:

* TypeScript → JavaScript
* JSX/TSX → JS
* 최신 문법 → 구문 다운그레이드

### 예시

```bash
esbuild app.ts --outfile=bundle.js --target=es2015
```

### 반드시 알아야 할 한계: **타입 검사를 하지 않는다**

esbuild의 TypeScript 지원은 **타입 표기를 지워버리는(strip) 것**이 전부입니다. 타입이 맞는지는 **검사하지 않습니다.**

```ts
const n: number = "문자열입니다";   // esbuild는 아무 불평 없이 통과시킴
```

* 그래서 Vite 기반 TS 프로젝트의 build 스크립트는 `"build": "tsc -b && vite build"` 처럼 **`tsc`를 먼저 돌립니다.**
* 타입 검사는 전체 파일의 관계를 알아야 하는 작업이라 **파일 단위 병렬 처리라는 esbuild의 속도 비결과 근본적으로 충돌**합니다. 빠른 이유가 곧 타입 검사를 못 하는 이유입니다.

같은 이유(**파일 하나만 보고 변환**)로 다음 TS 기능도 제약이 있습니다.

* `const enum` — 다른 파일의 값을 인라인해야 해서 완전 지원 불가
* 타입만 export하는 파일 — `isolatedModules` 규칙에 맞춰 `import type` / `export type`를 명시해야 함
* 실험적 데코레이터의 메타데이터 방출(`emitDecoratorMetadata`) — 미지원

---

## 3-3. Minify (압축)

ESBuild는 자체 압축 알고리즘을 포함합니다.

* whitespace 제거
* 지역 변수/함수 이름 축약(mangling) — `function calculateTotal(itemList)` → `function a(b)`
* dead code 제거
* inlining

> **minify ≠ 난독화(obfuscation)**
> esbuild 공식 문서는 "minification은 난독화가 아니다"라고 명시합니다.
> 이름이 짧아지는 것은 **용량을 줄이려는 부수 효과**일 뿐, 코드를 읽지 못하게 만드는 것이 목적이 아닙니다.
> 실제로 문자열 리터럴, 전역 API 이름, export된 식별자는 그대로 남아 있어 복원이 어렵지 않습니다.
> 코드를 진짜로 숨기고 싶다면 별도의 obfuscator가 필요합니다.

### 속도 비교

| 도구      | 특징    | 속도        |
| ------- | ----- | --------- |
| Terser  | JS 기반 | 느림        |
| SWC     | Rust  | 빠름        |
| ESBuild | Go    | **가장 빠름** |

---

# 4. ESBuild가 번들링할 때 내부적으로 일어나는 일

아래는 앞에서 요약한 파이프라인을 단계별로 풀어 설명한 내용입니다.

---

## ① Parsing

먼저 ESBuild는 파일을 읽어 AST(Abstract Syntax Tree)로 변환합니다.

* import/export 식 분석
* 변수/함수 스코프 분석
* 타입스크립트 타입 제거
* JSX → JS 변환

---

## ② Transform (JS/TS → AST)

AST를 기반으로 다음을 수행합니다.

* 최신 JS 문법 변환 (optional chaining 등)
* 타입 제거
* JSX → JS 변환
* 필요 없는 구문 제거

---

## ③ Bundling

모듈 그래프를 따라가며 dependency를 하나의 파일로 묶습니다.

* 경로 정규화
* 중복 모듈 제거
* tree shaking

---

## ④ Minify / Tree-shaking

마지막 처리 단계입니다.

* 이름 압축
* 빈칸 제거
* dead code 제거
* 사용되지 않는 import 제거

---

# 5. 실전 예시: ESBuild로 Vite 수준의 번들링 해보기

### 명령어

```bash
esbuild src/main.ts --bundle --minify --sourcemap --outfile=dist/main.js
```

결과:

* TypeScript → JS
* import 모듈 자동 번들링
* 압축(minify)
* sourcemap 생성
* 초고속 빌드 (대부분 <100ms)

---

# 6. ESBuild vs Webpack vs Rollup vs SWC

| 항목     | ESBuild | Webpack      | Rollup    | SWC    |
| ------ | ------- | ------------ | --------- | ------ |
| 언어     | Go      | JS           | JS        | Rust   |
| 속도     | **최고**  | 느림           | 중간        | 빠름     |
| 번들링    | O       | O            | O         | 일부만    |
| Minify | O       | Terser 기반    | O         | O      |
| TS 지원  | **내장**  | TS-loader 필요 | Plugin 필요 | 내장(빠름) |

---

# 7. ESBuild의 현재 위치 (정확히 알기)

"모든 도구가 esbuild를 내부에서 쓴다"는 설명은 사실이 아닙니다. 실제 상황은 다음과 같습니다.

| 도구            | 실제 내부 엔진                                                          |
| ------------- | ----------------------------------------------------------------- |
| **Vite 5\~7** | 개발 트랜스파일 + 의존성 사전 번들링에 **esbuild** 사용, 프로덕션 번들은 **Rollup**         |
| **Vite 8+**   | esbuild·Rollup을 걷어내고 **Oxc**(Rust 트랜스파일러) + **Rolldown**(Rust 번들러)로 통합 |
| **Bun**       | esbuild를 쓰지 않음. **Zig로 직접 구현한 자체 번들러**(API 디자인만 esbuild에서 영향받음)    |
| **Turbopack** | esbuild를 쓰지 않음. **Rust 기반 자체 엔진 + SWC** 파서                         |
| **Next.js**   | **SWC / Turbopack** 기반. esbuild는 핵심 경로에 없음                         |

### 그래서 esbuild는 끝났나?

아닙니다. 다만 역할이 바뀌었습니다.

* **여전히 최고인 영역**: 단발성 트랜스파일, 테스트 러너의 TS 변환(예: Vitest 계열 도구), CLI·서버 코드 번들링, 라이브러리 빌드 도구(tsup 등)의 내부 엔진
* **밀려난 영역**: 대규모 앱의 프로덕션 번들. esbuild는 **코드 스플리팅과 CSS 처리, 플러그인 훅이 Rollup만큼 성숙하지 않아** Vite가 프로덕션 번들러로 채택하지 않았고, 그 자리를 Rolldown이 가져갔습니다

> **핵심**: esbuild의 진짜 유산은 점유율이 아니라 **"빌드 도구는 네이티브 언어로 짠다"는 패러다임 전환**입니다.

---

# 마무리

ESBuild는 "빌드 도구를 네이티브 언어로 만든다"는 패러다임을 처음 증명한 도구입니다.

* Go 기반 멀티스레드
* 자체 구현된 초고속 파서
* TypeScript/JSX 완전 내장
* 번들링/트랜스파일/압축 기능 올인원

Vite 8이 Oxc·Rolldown으로 갈아타면서 **대규모 앱의 기본 빌드 경로에서는 물러났지만**,
단발성 트랜스파일·테스트 러너·라이브러리 빌드 도구의 엔진으로는 지금도 널리 쓰입니다(7장 참고).
esbuild를 이해하는 것은 곧 그 뒤를 이은 Oxc·Rolldown·SWC를 이해하는 출발점입니다.
