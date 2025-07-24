`<script>` 태그는 HTML 문서에서 **JavaScript를 삽입하거나 외부 스크립트를 불러오는 데 사용되는 매우 핵심적인 태그**입니다. 
---

# 🔧 `<script>` 태그란?

## ✅ 1. 정의

```html
<script>
  alert("Hello, world!");
</script>
```

또는 외부 파일 로드:

```html
<script src="main.js"></script>
```

* `<script>` 태그는 HTML 문서에 **JavaScript 코드 또는 외부 스크립트를 삽입**하는 용도로 사용됩니다.
* HTML 어디에서든 사용할 수 있지만, **위치와 속성에 따라 실행 타이밍과 성능에 큰 영향을 미칩니다.**

---

## 🧠 2. 기본 문법

```html
<script src="파일 경로" [속성들]>내부 스크립트</script>
```

* `src` 속성이 있으면 외부 스크립트를 가져옴
* `src`가 없으면 내부에 작성된 스크립트를 실행

---

## 🔧 3. 주요 속성

| 속성               | 설명                                          | 비고                             |
| ---------------- | ------------------------------------------- | ------------------------------ |
| `src`            | 외부 자바스크립트 파일 경로                             | **절대경로 / 상대경로 / CDN 등 가능**     |
| `type`           | MIME 타입 (`text/javascript`, `module` 등)     | 디폴트값: `text/javascript` (생략 가능) |
| `async`          | 외부 스크립트 **비동기 로드 + 병렬 실행**                  | HTML 파싱과 병렬, 실행 순서 제어 어려움      |
| `defer`          | 외부 스크립트 **비동기 로드 + HTML 파싱 완료 후 실행**        | 문서 구조 유지 필요 시 권장               |
| `crossorigin`    | CORS 정책 설정 (`anonymous`, `use-credentials`) | 외부 CDN 로딩 시 유용                 |
| `nomodule`       | 모듈을 지원하지 않는 브라우저에만 실행                       | `<script nomodule>`            |
| `referrerpolicy` | 스크립트 요청 시 Referrer 헤더 제어                    | 보안 및 프라이버시 제어 가능               |
| `integrity`      | Subresource Integrity (SRI) 검증을 위한 해시값      | CDN 보안 검증용                     |

---

## ⚙️ 4. 실행 방식에 따른 비교

| 위치/속성            | HTML 파싱 차단 | 실행 시점               | 특징                          |
| ---------------- | ---------- | ------------------- | --------------------------- |
| `<script>` (기본)  | ✅ 예        | 파싱 중단 후 즉시 실행       | 가장 기본적 방식                   |
| `<script defer>` | ❌ 아니요      | DOM 파싱 완료 후 순서대로 실행 | **HTML 구조 보존 + 순서 유지**      |
| `<script async>` | ❌ 아니요      | 다운로드 완료 즉시 실행       | **순서 보장 불가**, 독립적인 스크립트에 적합 |

---

## 📌 5. 모듈 스크립트 (`type="module"`)

```html
<script type="module" src="app.js"></script>
```

* ES6 모듈 시스템을 활용 가능
* 모듈 간 `import` / `export` 가능
* 자동으로 `defer` 동작 포함됨
* 모듈 스코프(전역 변수 공유 X)

---

## 📦 6. 예시

### ✅ 내부 스크립트

```html
<script>
  function sayHi() {
    console.log("Hi!");
  }
  sayHi();
</script>
```

### ✅ 외부 스크립트

```html
<script src="/js/app.js" defer></script>
```

### ✅ 모듈 + nomodule 병행 지원 (호환성 대응)

```html
<script type="module" src="/main.module.js"></script>
<script nomodule src="/main.legacy.js"></script>
```

---

## 🧠 7. HTML 내 사용 위치

| 위치           | 설명                                 |
| ------------ | ---------------------------------- |
| `<head>`     | 일반적으로 `defer` 또는 `async` 속성과 함께 사용 |
| `<body>` 끝부분 | `defer` 없이 사용해도 HTML 파싱 완료 후 실행됨   |
| `<noscript>` | 자바스크립트를 지원하지 않는 브라우저용 대체 콘텐츠 제공    |

---

## ❗️ 8. 주의 사항 및 모범 사례

* `src`가 있는 `<script>`는 **비워두거나 `type="module"`과 함께 defer 권장**
* 가능하면 외부 스크립트로 분리 → **코드 관리, 캐싱, CDN 적용 유리**
* 모듈(`type="module"`)을 쓸 경우 **중복 실행/스코프 충돌 없음**
* `<script>`는 기본적으로 **HTML 파싱을 차단**하므로 렌더링 차단을 피하고 싶으면 `defer` 또는 `async` 사용

---

## ✅ 요약

| 항목     | 내용                                        |
| ------ | ----------------------------------------- |
| 태그명    | `<script>`                                |
| 목적     | JavaScript 코드 실행 또는 외부 파일 로딩              |
| 사용 위치  | `<head>` 또는 `<body>` 내부                   |
| 내부/외부  | 직접 작성 또는 `src`로 외부 연결 가능                  |
| 중요한 속성 | `src`, `defer`, `async`, `type`, `module` |
| 기본 동작  | HTML 파싱 중단 → 즉시 실행                        |
| 모범 사용  | `defer` + 외부 파일 or `type="module"` 활용     |

