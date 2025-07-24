HTML의 `<nav>` 태그는 웹 문서 구조에서 **내비게이션(navigation)** 역할을 시맨틱하게 나타내는 매우 중요한 태그입니다. 

---

# 🧭 `<nav>` 태그: 시맨틱 내비게이션 완전 정복

---

## ✅ 1. 정의 (Definition)

```html
<nav>
  <a href="/">홈</a>
  <a href="/about">소개</a>
  <a href="/contact">문의</a>
</nav>
```

* `<nav>`는 HTML5에서 도입된 **시맨틱 블록 요소 (semantic block-level element)** 입니다.
* 웹사이트 내에서 **주요 탐색 경로(내비게이션 링크)** 를 묶는 용도로 사용됩니다.
* 예: 상단 메뉴, 사이드바, 바닥글 메뉴, 탭 목록 등

---

## 🎯 2. 사용 목적

* 사용자와 검색 엔진에게 "이 영역은 문서의 주요 탐색 링크를 포함하고 있다"는 의미를 명시적으로 전달
* 내비게이션 관련 UI 구조를 명확히 하여 **시맨틱 구조 강화 + 접근성 향상**

---

## 📌 3. 사용 위치

| 위치             | 예시                                          |
| -------------- | ------------------------------------------- |
| 페이지 상단         | 글로벌 메뉴 (예: 홈, 소개, 로그인)                      |
| 사이드바           | 카테고리, 문서 메뉴                                 |
| `<header>` 내부  | 보통 메인 내비게이션으로 사용                            |
| `<footer>` 내부  | 사이트맵, 정책, 하단 링크                             |
| `<article>` 내부 | 문서 내 목차용 링크도 가능 (단, 너무 사소하면 `<nav>` 쓰지 말 것) |

---

## 🔧 4. 포함 가능한 요소

* `<a>`: 필수 링크
* `<ul>`, `<ol>`, `<li>`: 리스트 형태로 정리 가능
* `<button>`: 모바일 메뉴 토글용
* `<svg>`, `<img>`: 아이콘 등

```html
<nav>
  <ul>
    <li><a href="/">홈</a></li>
    <li><a href="/about">소개</a></li>
    <li><a href="/products">제품</a></li>
  </ul>
</nav>
```

---

## 🧠 5. 시맨틱 의미와 다른 태그 비교

| 태그         | 의미       | 차이점                          |
| ---------- | -------- | ---------------------------- |
| `<nav>`    | 탐색 링크 그룹 | 시맨틱 강조, role="navigation" 암시 |
| `<div>`    | 단순 그룹    | 의미 없음                        |
| `<aside>`  | 보조 콘텐츠   | 보통 본문 외 링크/광고 등              |
| `<header>` | 제목/소개    | 종종 `<nav>` 포함됨               |

---

## ♿ 6. 접근성과 ARIA

* `<nav>`는 **자동으로 `role="navigation"`** 을 가집니다.
* **여러 개의 `<nav>`** 태그가 있을 경우 `aria-label` 또는 `aria-labelledby`로 역할 설명을 명확히 해야 합니다.

```html
<nav aria-label="메인 내비게이션">
  <a href="/">홈</a>
  <a href="/products">제품</a>
</nav>

<nav aria-label="푸터 링크">
  <a href="/terms">이용약관</a>
  <a href="/privacy">개인정보처리방침</a>
</nav>
```

---


## 🎨 7. 스타일링 예시 (CSS)

```css
nav {
  background-color: #333;
  padding: 1rem;
}
nav a {
  color: white;
  margin-right: 1rem;
  text-decoration: none;
}
nav a:hover {
  text-decoration: underline;
}
```

---

## 🧩 8. React(JSX) 사용 예시

```jsx
function Navigation() {
  return (
    <nav aria-label="메인 메뉴" className="main-nav">
      <a href="/">홈</a>
      <a href="/about">소개</a>
      <a href="/contact">문의</a>
    </nav>
  );
}
```

* JSX에서는 `className`, `aria-*`, `onClick` 등을 함께 사용
* React Router 사용 시 `<Link>`로 대체 가능

---

## ✅ 9. 요약 정리

| 항목         | 내용                                               |
| ---------- | ------------------------------------------------ |
| 태그명        | `<nav>`                                          |
| 의미         | 탐색 링크를 그룹화한 시맨틱 블록                               |
| 포함 요소      | `<a>`, `<ul><li>`, `<button>`, 아이콘 등             |
| 위치         | `header`, `footer`, `aside`, `main` 어디든 사용 가능    |
| 접근성        | 기본적으로 `role="navigation"` 포함, `aria-label` 사용 권장 |
| SEO        | 검색 엔진이 중요한 링크 영역으로 인식                            |
| React 사용 시 | JSX에서 `className`, `aria-label` 활용               |

