# ✅ `<a>` (Anchor) 태그: 완전 정복

## 1. 🔹 개요 (Definition)

* `<a>` 태그는 HTML 문서 내에서 \*\*하이퍼링크(hyperlink)\*\*를 생성하는 데 사용됩니다.
* "anchor"는 "닻"이라는 의미로, 웹 문서에서 **참조 지점 또는 연결 고리 역할**을 합니다.
* 문서 내부나 외부 리소스로 이동할 수 있습니다.

```html
<a href="https://example.com">링크 텍스트</a>
```

---

## 2. 🔧 주요 속성 (Attributes)

| 속성               | 설명                                                                  |
| ---------------- | ------------------------------------------------------------------- |
| `href`           | 이동할 URL 또는 문서 내 id(anchor). 생략하면 클릭 가능한 요소는 되지만 링크는 없음              |
| `target`         | 링크 클릭 시 열리는 위치를 지정 (`_blank`, `_self`, `_parent`, `_top`, 프레임 이름 등) |
| `rel`            | 링크된 리소스와의 관계를 명시 (보안 및 SEO 중요)                                      |
| `download`       | 링크를 클릭할 때 다운로드하게 함 (브라우저 지원 필수)                                     |
| `type`           | MIME 타입 힌트. 대부분 생략 가능                                               |
| `hreflang`       | 링크 리소스의 언어를 명시 (`en`, `ko`, `fr` 등)                                 |
| `referrerpolicy` | 링크 클릭 시 referrer(참조자) 전송 정책 설정                                      |
| `title`          | 마우스 오버 시 표시되는 툴팁 텍스트                                                |

---

## 3. 🔍 속성 상세 설명

### ✅ `href` (Hypertext REFerence)

* 링크의 목적지를 지정합니다.
* 절대 경로, 상대 경로, fragment(`"#id"`), 전화(`tel:`), 이메일(`mailto:`) 등도 가능

예시:

```html
<a href="/about">내부 링크</a>
<a href="https://google.com">외부 링크</a>
<a href="#section1">문서 내 링크</a>
<a href="tel:+821012345678">전화 걸기</a>
```

---

### ✅ `target`

| 값             | 설명                          |
| ------------- | --------------------------- |
| `_self` (기본값) | 현재 탭에서 열림                   |
| `_blank`      | 새 탭에서 열림 (**보안 주의**)        |
| `_parent`     | 부모 프레임에서 열림 (frameset 사용 시) |
| `_top`        | 최상위 창에서 열림 (frameset 제거용)   |
| `이름`          | 이름을 가진 iframe이나 창에서 열림      |

```html
<a href="https://vite.dev" target="_blank">Vite</a>
```

---

### ✅ `rel`

* 링크된 문서와의 **관계를 설명**하거나, **보안 목적**으로 사용
* `target="_blank"` 사용 시 보안상 `rel="noopener noreferrer"` **반드시 사용해야 안전함**

| 값            | 설명                                 |
| ------------ | ---------------------------------- |
| `noopener`   | 새 탭에서 opener 객체 제거 (XSS 방지)        |
| `noreferrer` | 참조자(referrer) 정보를 보내지 않음           |
| `nofollow`   | 검색엔진이 이 링크를 따라가지 않음                |
| `external`   | 외부 리소스임을 명시                        |
| `ugc`        | User Generated Content (예: 댓글의 링크) |
| `sponsored`  | 광고성 링크임을 명시                        |

**보안 모범 예시:**

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">외부 링크</a>
```

---

### ✅ `download`

* 클릭 시 브라우저가 리소스를 **다운로드하도록 유도**
* 다운로드 파일명을 직접 지정할 수도 있음

```html
<a href="/files/manual.pdf" download="user-manual.pdf">설명서 다운로드</a>
```

---

### ✅ `hreflang`

* 링크된 문서의 **언어 정보** 제공 (SEO에 도움)

```html
<a href="https://example.com/fr" hreflang="fr">French version</a>
```

---

### ✅ `type`

* 리소스의 MIME 타입 힌트 제공

```html
<a href="resume.pdf" type="application/pdf">이력서</a>
```

---

## 4. 🌐 내부 링크 vs 외부 링크

| 구분    | 예시                                        | 설명             |
| ----- | ----------------------------------------- | -------------- |
| 내부 링크 | `<a href="/about">About</a>`              | 동일 도메인 내 이동    |
| 외부 링크 | `<a href="https://google.com">Google</a>` | 외부 도메인 이동      |
| 해시 링크 | `<a href="#top">맨 위로</a>`                 | 문서 내 특정 id로 이동 |

---

## 5. 🧠 동작 방식

* 브라우저는 `<a>`를 렌더링할 때 해당 텍스트를 **하이퍼링크로 스타일링**합니다.
* 클릭 시:

  1. `href` 주소로 이동하거나 다운로드를 실행
  2. `target` 위치에 따라 새 탭/현재 탭 이동
  3. `rel` 설정에 따라 보안/SEO 영향을 받음

---

## 6. ♿ 접근성과 SEO

### ✅ 접근성 (Accessibility)

* 스크린 리더는 `<a>` 텍스트를 읽습니다. 텍스트가 명확해야 합니다.
* 아이콘만 있는 경우 `aria-label` 사용이 권장됩니다.

```html
<a href="https://twitter.com" aria-label="Twitter">
  <img src="twitter.svg" alt="" />
</a>
```

### ✅ SEO (검색 엔진 최적화)

* 검색 엔진은 `<a>` 태그의 텍스트를 **링크의 의미/키워드**로 분석합니다.
* `nofollow`/`ugc`/`sponsored` 등 rel 속성은 **링크 신뢰도** 평가에 영향을 줍니다.

---

## 7. 🔒 보안 이슈: `target="_blank"`

```html
<a href="https://malicious.com" target="_blank">Click me</a>
```

* 아무런 `rel` 속성 없이 `_blank`만 사용하면,

  * 새 탭이 원래 탭의 `window.opener` 객체를 통해 제어 가능
  * 피싱, 탭 재지정(Tab-nabbing) 공격 가능

✅ 해결:

```html
<a href="https://safe.com" target="_blank" rel="noopener noreferrer">안전한 링크</a>
```

---

## 8. 🧩 React와의 관계

### ✅ JSX에서는 `<a>`도 HTML처럼 사용하지만 다음에 유의:

| HTML               | JSX                     |
| ------------------ | ----------------------- |
| `class`            | `className`             |
| `onclick`          | `onClick`               |
| `href="#section1"` | 그대로 사용 가능               |
| `ref`              | React에서 DOM 참조용으로 별도 의미 |

---

## ✅ 정리 요약

| 항목      | 설명                                                  |
| ------- | --------------------------------------------------- |
| 용도      | 하이퍼링크 생성, 문서 내외부 이동                                 |
| 필수 속성   | `href`, `target`, `rel` (보안 시)                      |
| 접근성     | 의미 있는 텍스트, `aria-label` 보조                          |
| 보안      | `target="_blank"`에는 반드시 `rel="noopener noreferrer"` |
| SEO     | 텍스트 링크 + `rel` 속성 최적화 필요                            |
| React에서 | `<a>` 그대로 사용, 이벤트나 스타일은 JSX 방식 적용                   |

