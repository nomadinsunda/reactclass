
# 🧭 `<header>` 태그: 완전 정복

---

## ✅ 1. 정의 (Definition)

```html
<header>
  <h1>사이트 제목</h1>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
```

* `<header>`는 HTML5에서 도입된 **시맨틱 태그(semantic tag)** 로,
* **페이지 전체** 또는 **개별 섹션/컴포넌트**의 **머리말(헤더) 역할**을 명시적으로 정의합니다.
* 내부에 제목, 부제목, 로고, 네비게이션 메뉴 등이 포함됩니다.

---

## 🧠 2. 의미와 목적

* `<header>`는 **문서 또는 섹션의 서론/정보 도입부**로 해석됩니다.
* 웹 페이지 전체의 **글로벌 헤더**뿐만 아니라, `<article>`, `<section>` 같은 **로컬 구조에도 반복적으로 사용 가능**합니다.

```html
<article>
  <header>
    <h2>블로그 제목</h2>
    <p>작성자: 홍길동</p>
  </header>
  <p>본문 내용...</p>
</article>
```

> ✅ 즉, `<header>`는 문서 전체의 헤더뿐 아니라 **각 콘텐츠 블록의 시작점**도 의미합니다.

---

## 📦 3. 포함할 수 있는 내용

| 가능 요소          | 설명                   |
| -------------- | -------------------- |
| `<h1>`\~`<h6>` | 제목 또는 로고 텍스트         |
| `<nav>`        | 내비게이션 메뉴 (자주 같이 사용됨) |
| `<p>`, `<img>` | 부제목, 로고 이미지          |
| `<form>`       | 검색창 등                |

❌ **주의**: `<header>`는 다른 **시맨틱 요소(`<footer>`, `<main>`, `<article>` 등)** 내부에는 중첩 가능하지만, `<header>` 안에 `<header>`는 중첩하지 말아야 합니다.

---

## 🔧 4. 주요 속성

| 속성      | 설명                                          |
| ------- | ------------------------------------------- |
| 전역 속성   | `id`, `class`, `style`, `title`, `data-*` 등 |
| ARIA 속성 | `role="banner"`로 암묵적으로 처리됨                  |
| 자주 사용   | `class="site-header"` 등 스타일링 목적             |

---

## 🧩 5. 예시 코드

### ✅ 글로벌 헤더 (사이트 전체)

```html
<header>
  <img src="logo.png" alt="MySite 로고">
  <h1>MySite</h1>
  <nav>
    <a href="/">홈</a>
    <a href="/about">소개</a>
  </nav>
</header>
```

### ✅ 로컬 헤더 (기사나 섹션 내부)

```html
<section>
  <header>
    <h2>공지사항</h2>
    <p>2025년 7월 24일 업데이트</p>
  </header>
  <p>본 내용은...</p>
</section>
```

---

## 🧠 6. `<header>` vs `<head>` 차이

| 항목       | `<header>`        | `<head>`                                    |
| -------- | ----------------- | ------------------------------------------- |
| 위치       | `<body>` 내부       | `<html>` 내부                                 |
| 용도       | 시각적 콘텐츠의 시작부      | 메타데이터, 스크립트, 스타일                            |
| 포함 요소    | 텍스트, 이미지, 내비게이션 등 | `<meta>`, `<title>`, `<link>`, `<script>` 등 |
| 사용자에게 보임 | ✅ 예               | ❌ 보이지 않음                                    |

---

## ♿ 7. 접근성 (Accessibility)

* `<header>`는 브라우저와 스크린 리더가 **시맨틱 구조**로 인식
* 페이지 최상단의 `<header>`는 자동으로 `role="banner"` 로 해석됩니다

  * 단, 여러 개의 `<header>`가 있을 경우 **페이지 레벨은 오직 하나만** `role="banner"` 로 인식됨

> ✅ `role="banner"`는 페이지의 대표적인 소개/제목/로고에 부여되는 역할입니다.

---

## 🔍 8. SEO 관점에서

* `<header>`는 SEO에 긍정적인 영향을 줍니다.
* `<header>` 내의 `<h1>`, `<nav>`, `<a>` 링크들은 검색 엔진이 **페이지 구조, 주요 콘텐츠**를 이해하는 데 도움을 줍니다.

---

## 🎨 9. 스타일링 예시 (CSS)

```css
header {
  background-color: #333;
  color: white;
  padding: 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
header h1 {
  font-size: 1.5rem;
}
```

---

## 🧩 10. React(JSX)에서의 사용

```jsx
function Header() {
  return (
    <header className="site-header">
      <h1>My App</h1>
      <nav>
        <a href="/">Home</a>
        <a href="/docs">Docs</a>
      </nav>
    </header>
  );
}
```

---

## ✅ 요약

| 항목         | 설명                                       |
| ---------- | ---------------------------------------- |
| 태그명        | `<header>`                               |
| 의미         | 문서 또는 섹션의 머리말, 소개부                       |
| 포함 가능한 요소  | 제목, 내비게이션, 이미지, 서브타이틀 등                  |
| 위치         | `<body>` 어디든 사용 가능, 반복 가능                |
| 접근성        | 자동으로 `role="banner"` 역할                  |
| SEO        | 검색 엔진이 주요 구조로 인식                         |
| React 사용 시 | JSX 문법 그대로 사용 (`className`, `onClick` 등) |


