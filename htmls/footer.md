
# 🧱 `<footer>` 태그

---

## ✅ 1. 정의 (Definition)

```html
<footer>
  <p>&copy; 2025 MyCompany. All rights reserved.</p>
</footer>
```

* `<footer>`는 HTML5에서 도입된 **시맨틱 블록 요소 (semantic block-level element)** 입니다.
* 페이지나 섹션(section)의 **하단 정보를 시각적 + 구조적으로** 구분하는 데 사용됩니다.

---

## 🧩 2. 역할 및 의미

* **전체 페이지의 바닥글**
* 또는 개별 `<article>`, `<section>`, `<aside>`, `<main>` 요소의 **로컬 푸터(local footer)**

```html
<article>
  <h2>블로그 글</h2>
  <p>내용...</p>
  <footer>작성자: 홍길동, 2025.07.24</footer>
</article>
```

> ✅ 즉, `<footer>`는 "문서의 끝"이 아니라 "문맥의 끝"을 의미할 수 있습니다.

---

## 🔧 3. 사용 가능한 위치

| 위치                                     | 설명                        |
| -------------------------------------- | ------------------------- |
| `<body>` 바로 아래                         | 전체 페이지의 글로벌 footer        |
| `<section>`, `<article>`, `<aside>` 내부 | 로컬 footer (작성자, 출처, 태그 등) |
| `<main>` 내부 또는 외부                      | 보통 내부 사용 권장 (접근성 향상)      |

---

## 📦 4. 포함 가능한 콘텐츠

`<footer>`에는 다음과 같은 요소들을 포함할 수 있습니다:

* 저작권 정보 (`&copy;`, 라이선스)
* 제작자 정보 / 연락처
* 사이트 맵 링크
* 하단 내비게이션 (`<nav>`)
* 소셜 링크
* 법적 고지사항
* 언어 설정
* 테마/접근성 설정

예:

```html
<footer>
  <p>&copy; 2025 DevSchool. Contact: <a href="mailto:info@devschool.com">info@devschool.com</a></p>
  <nav>
    <a href="/privacy">개인정보처리방침</a> |
    <a href="/terms">이용약관</a>
  </nav>
</footer>
```

---

## 🧠 5. 시맨틱 의미 및 SEO

| 항목         | 설명                                                     |
| ---------- | ------------------------------------------------------ |
| **시맨틱 구조** | `<footer>`는 해당 블록의 "보조 정보" 또는 "마무리" 의미                 |
| **SEO**    | 검색엔진은 `<footer>` 영역을 **부가정보 또는 사이트 전역 정보**로 인식         |
| **중복 정보**  | 동일한 footer가 여러 페이지에 반복되어도 SEO 패널티 없음 (common practice) |

---

## ♿ 6. 접근성과 `<footer>`

* `<footer>`는 시멘틱 구조에 기반하므로 **스크린 리더와 보조 기술이 인식**합니다.
* `<main>`, `<header>`, `<nav>`, `<footer>` 구조를 제대로 사용하면 **웹 접근성(ARIA 레벨)** 이 향상됩니다.

> ✅ `<footer>`는 자동으로 `role="contentinfo"` 역할을 가집니다.

---

## 🎨 7. 기본 스타일 및 커스터마이징

기본 스타일은 없으며, CSS로 완전히 제어합니다.

```css
footer {
  background-color: #f0f0f0;
  padding: 1em;
  text-align: center;
  font-size: 0.9rem;
  color: #666;
}
```

---

## 🔁 8. React(JSX)에서의 사용

```jsx
function Footer() {
  return (
    <footer className="footer">
      <p>&copy; 2025 DevSchool. All rights reserved.</p>
    </footer>
  );
}
```

* JSX에서도 동일하게 사용 가능
* `className`을 통해 CSS 클래스 지정

---

## ⚠️ 9. `<footer>`와 `<div id="footer">`의 차이

| 항목  | `<footer>`       | `<div id="footer">`      |
| --- | ---------------- | ------------------------ |
| 의미  | 시맨틱 HTML, 접근성 우수 | 의미 없음 (의미 부여하려면 ARIA 필요) |
| SEO | 구조 인식 가능         | 의미 없음                    |
| 추천  | ✅ 사용 권장          | ❌ 되도록 피함 (구식 방법)         |

---

## ✅ 요약

| 항목    | 내용                                        |
| ----- | ----------------------------------------- |
| 태그    | `<footer>`                                |
| 의미    | 페이지 또는 섹션의 마무리 정보                         |
| 위치    | `<body>` 또는 `<section>`, `<article>` 등 내부 |
| 포함 내용 | 저작권, 연락처, 하단 내비, 소셜 링크 등                  |
| 시맨틱   | **예** (role="contentinfo")                |
| 접근성   | 스크린 리더가 명확히 인식                            |
| SEO   | 사이트 구조 파악에 도움                             |
| React | JSX에서도 그대로 사용 가능                          |


