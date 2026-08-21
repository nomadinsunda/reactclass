# Rollup이란?

**— ES Module 기반의 “정적 분석(Static Analysis)”에 최적화된 고품질 번들러**

Rollup은 **JavaScript 모듈(특히 ES Modules)을 기반으로 한 오픈소스 번들러**로,
라이브러리 제작에 가장 널리 사용되는 도구 중 하나입니다.

Webpack처럼 범용적인 번들러가 아니라, **“불필요한 코드 없이 가장 깨끗한 출력물을 만드는 데 특화된 번들러”** 라는 점이 Rollup의 가장 큰 특징입니다.

---

# Rollup의 핵심 특징 요약

| 기능                         | 설명                                   |
| -------------------------- | ------------------------------------ |
| **정적 분석 기반의 Tree-shaking** | ES Module 구조를 정적으로 분석하여 Dead code 제거 |
| **아주 가벼운 출력물**             | 라이브러리 배포에 적합                         |
| **플러그인 기반의 확장성**           | ESBuild/SWC와도 연계 가능                  |
| **Rollup 공식 플러그인 생태계**     | CommonJS, JSON, TypeScript 등         |
| **Vite의 내부 빌드 엔진**         | Vite 7까지의 build 단계를 Rollup이 수행 (Vite 8부터는 후계자 Rolldown) |

즉, **“번들 크기 최적화”** 에 있어서는 아직도 업계 최고의 품질을 보장합니다.

---

# Rollup의 설계 철학: 정적 분석 중심

Webpack과 달리 Rollup은 **ES Module(ESM) 문법만을 기반으로 설계**되었습니다.

### ES Module의 장점

Rollup은 다음과 같은 ESM 특성을 최대한 활용합니다:

* 정적 `import`, `export`
* 코드 실행 전에 의존성 그래프 완전 파악 가능
* 사이드 이펙트(side effect)를 정적으로 판단 가능

즉…

### “정적 분석을 극대화하여 Dead code를 안전하게 제거”

Webpack의 tree-shaking보다 안정적이고 단순합니다.

---

# Rollup이 인기 있는 이유

## O 1. 라이브러리 제작에 최적화

React, Vue, Redux 등 주요 라이브러리도 Rollup을 사용해 빌드합니다.

라이브러리는 다음이 중요합니다:

* 작은 번들 크기
* 모듈 시스템 지원 (ESM, CommonJS, UMD)
* 사이드 이펙트 없는 클린 코드

Rollup은 이 점에서 Webpack보다 훨씬 적합합니다.

---

## O 2. 매우 강력한 Tree-shaking

Rollup의 tree-shaking은 업계에서 가장 정교합니다.

* 사용되지 않는 import 제거
* 선언된 함수/변수 중 실제 호출되지 않는 코드 제거
* 모듈별 사이드 이펙트 감지

> “라이브러리 번들링 품질”만 놓고 보면 Rollup은 Webpack보다 더 우수합니다.

---

## O 3. Plugin 기반 아키텍처

Rollup은 플러그인으로 모든 기능을 확장합니다.

대표적인 플러그인:

* @rollup/plugin-node-resolve
* @rollup/plugin-commonjs
* @rollup/plugin-typescript
* @rollup/plugin-terser (압축)

> 예전 자료에 자주 나오는 `rollup-plugin-terser`는 **더 이상 유지보수되지 않는 폐기된 패키지**입니다.
> 공식 스코프로 이관된 **`@rollup/plugin-terser`** 를 쓰세요. 이름만 다른 게 아니라 Rollup 3+ 호환성이 다릅니다.

Rollup은 플러그인 구조가 단순하고 강력하여 **커뮤니티 플러그인 생태계가 매우 풍부합니다.**

---

## O 4. Vite Build 엔진

**Vite 5\~7**은 dev 서버는 esbuild, 빌드 단계는 **Rollup**을 사용했습니다.

왜 그럴까요?

* Dev 서버: 빠른 HMR → esbuild (파일 1개를 최대한 빨리 변환)
* Production 빌드: 고품질 번들 → Rollup (전체 그래프를 보고 트리셰이킹·청크 분할)

즉, Rollup은 “최종 출력”의 품질을 높이기 위한 핵심 엔진으로 선택되었습니다.

> **단, Vite 8(2026년 3월)부터는 Rollup이 아니라 Rolldown을 씁니다.** 아래 절에서 설명합니다.

---

# Rollup 기본 사용 예제

### 1) 설치

```bash
npm install rollup --save-dev
```

### 2) 설정 파일 작성: rollup.config.js

```js
export default {
  input: "src/index.js",
  output: {
    file: "dist/bundle.js",
    format: "esm",
  },
};
```

### 3) 실행

```bash
npx rollup -c
```

---

# Rollup 출력 포맷(format) 종류

| format   | 설명                           |
| -------- | ---------------------------- |
| **esm**  | ES Module                    |
| **cjs**  | Node.js CommonJS (`require`) |
| **umd**  | 브라우저 & Node 모두 가능            |
| **iife** | 즉시 실행 함수 형태 (브라우저용)          |

라이브러리 배포 시 `esm + cjs + umd` 세 가지를 모두 내놓는 경우가 많습니다.

---

# Rollup과 Webpack의 차이 (핵심 비교)

| 항목           | Rollup             | Webpack                 |
| ------------ | ------------------ | ----------------------- |
| 목적           | 라이브러리 번들링          | 웹 앱 번들링                 |
| Tree-shaking | **최고 수준 (ESM 기반)** | 중간                      |
| Dev server   | 없음                 | 있음 (Webpack Dev Server) |
| 빌드 속도        | 보통                 | 느림                      |
| 코드 분할        | 지원                 | **매우 잘 지원**             |
| 사용 분야        | 라이브러리              | SPA, 웹 애플리케이션           |

→ **‘웹앱’은 Webpack / ‘라이브러리’는 Rollup**이라는 공식이 생길 만큼 역할이 확실하게 다릅니다.

---

# 미래와 현재의 Rollup: Vite와 함께 부활

Rollup은 Webpack에 밀려 한동안 주류에서 멀어졌지만,
**Vite를 통해 다시 핵심 빌드 엔진으로 자리 잡았습니다.**

* 빠른 Dev → esbuild
* 고품질 번들 → Rollup

그리고 Rollup 3.x/4.x에서 빌드 속도가 크게 향상되어
**대규모 SPA에서도 Rollup을 직접 사용하는 사례**가 늘었습니다.

---

## 그다음 이야기: Rolldown

Rollup의 한계는 **JavaScript로 작성되어 있다는 점**이었습니다. 아무리 알고리즘을 다듬어도 대규모 프로젝트의 빌드 시간은 언어 성능의 벽에 부딪힙니다.

그래서 Vite 팀(Evan You)이 만든 것이 **Rolldown**입니다.

| 항목    | Rollup             | Rolldown                    |
| ----- | ------------------ | --------------------------- |
| 구현 언어 | JavaScript         | **Rust**                    |
| 목표    | 최고 품질의 번들 출력       | **Rollup과 호환되면서 훨씬 빠른 빌드**  |
| 플러그인  | Rollup 플러그인 생태계    | Rollup 플러그인 API를 **의도적으로 호환** |

* **Vite 8부터 Rolldown이 기본 번들러**가 되어 Rollup과 esbuild를 모두 대체했습니다. 옵션 이름도 `build.rollupOptions` → `build.rolldownOptions`로 바뀌었습니다(옛 이름은 자동 변환).
* Rolldown이 Rollup의 **플러그인 API를 그대로 흉내 낸 이유**가 중요합니다. 그래야 수년간 쌓인 Rollup 플러그인 생태계를 버리지 않고 그대로 옮겨올 수 있기 때문입니다. 즉 **Rollup은 사라진 것이 아니라, 그 설계가 Rust로 계승된 것**입니다.

> 그래서 **이 문서에서 배우는 Rollup의 개념(정적 분석, 트리셰이킹, 출력 포맷, 플러그인 훅)은 그대로 유효합니다.** 실행 엔진만 바뀌었을 뿐입니다.
> 또한 Rollup 자체도 **라이브러리 빌드용으로는 여전히 현역**입니다.

---

# 마무리

Rollup은 다음과 같은 상황에서 최고의 선택입니다.

* JavaScript/TypeScript 라이브러리 제작
* 모듈 크기가 중요한 상황
* Tree-shaking이 매우 중요한 상황
* 다양한 모듈 시스템으로 배포해야 할 때
* Vite 7 이하 기반 프로젝트의 Production 빌드 (Vite 8부터는 같은 역할을 Rolldown이 맡습니다)

즉, Rollup은 **“깔끔한 결과물을 우선하는 번들러”**이며,
그 설계 철학은 Rolldown에 그대로 계승되어 지금도 유효합니다.

