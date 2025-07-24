# 🖱️ `<button>` 태그

## ✅ 1. 정의 (Definition)

```html
<button type="submit">로그인</button>
```

* `<button>` 태그는 사용자가 **클릭 가능한 버튼**을 생성합니다.
* HTML5에서 버튼은 **폼(form) 제어용**, **자바스크립트 인터랙션**, **접근성 강화**, **동적 UI 컨트롤** 등에 사용됩니다.
* **클릭, 포커스, 키보드 이벤트 처리**가 기본적으로 내장되어 있습니다.

---

## 🔧 2. 주요 속성 (Attributes)

| 속성                                          | 필수 | 설명                                     |
| ------------------------------------------- | -- | -------------------------------------- |
| `type`                                      | ✅  | 버튼의 동작 타입: `submit`, `reset`, `button` |
| `disabled`                                  | ❌  | 버튼 비활성화                                |
| `name`                                      | ❌  | 폼 필드 이름으로 제출됨 (submit/reset 시)         |
| `value`                                     | ❌  | 폼 제출 시 전달할 값                           |
| `autofocus`                                 | ❌  | 페이지 로드시 자동 포커스                         |
| `form`                                      | ❌  | 버튼이 속하지 않은 외부 폼의 ID를 지정                |
| `formaction`, `formenctype`, `formmethod` 등 | ❌  | `<form>` 속성과 동일하지만 버튼에서 override 가능    |

---

## 🔍 3. `type` 속성 — 버튼의 동작을 결정

```html
<button type="submit">제출</button>
<button type="reset">초기화</button>
<button type="button">클릭</button>
```

| 타입       | 설명                   | 기본 동작               |
| -------- | -------------------- | ------------------- |
| `submit` | 폼을 서버로 제출            | `<form>` 내부 버튼의 기본값 |
| `reset`  | 폼의 모든 필드 초기화         | 입력값 원상복구            |
| `button` | 아무 동작 없음 (JS 이벤트 필요) | UI 제어용              |

🚨 `type`을 명시하지 않으면 `submit`이 기본값이기 때문에 **form 내부에서 의도치 않은 제출 문제가 발생**할 수 있습니다.

---

## 🧠 4. 기본 동작 (with `<form>`)

```html
<form action="/login" method="POST">
  <input type="text" name="username" />
  <button type="submit">로그인</button>
</form>
```

* 버튼 클릭 시 브라우저는 해당 `<form>`의 `action`으로 `method`에 따라 HTTP 요청을 전송합니다.

---

## 🖌️ 5. CSS 스타일링

```css
button {
  background-color: royalblue;
  color: white;
  border-radius: 4px;
  padding: 0.5em 1em;
  cursor: pointer;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

* `button`은 기본적으로 OS 스타일을 따릅니다.
* 다양한 상태(pseudo-classes) 스타일링 필요:

  * `:hover`, `:focus`, `:active`, `:disabled` 등

---

## 📱 6. 접근성 (Accessibility)

* `<button>`은 스크린 리더와 키보드 접근성이 **기본적으로 우수**
* 클릭 외에 `Space`, `Enter` 키로도 활성화됨
* 반드시 **의미 있는 텍스트 또는 `aria-label`** 필요

```html
<button aria-label="삭제">
  <svg aria-hidden="true">🗑️</svg>
</button>
```

---

## ⚙️ 7. JavaScript 이벤트와 함께 사용

```html
<button type="button" id="clickBtn">눌러보세요</button>

<script>
  document.getElementById('clickBtn').addEventListener('click', function () {
    alert('버튼이 클릭됨!');
  });
</script>
```

* `click` 이벤트는 가장 일반적인 UI 인터랙션
* 필요 시 `mousedown`, `mouseup`, `keydown` 등도 사용 가능

---

## 🧩 8. React(JSX)에서의 사용

```jsx
<button type="button" onClick={handleClick} className="btn-primary">
  제출
</button>
```

* `onClick` 등의 이벤트 핸들러는 camelCase로 작성
* `class` → `className`
* 내부에 텍스트, 아이콘, 컴포넌트 등 자유롭게 렌더링 가능

---

## 🔐 9. 보안 고려

| 이슈          | 설명                                          | 예시             |
| ----------- | ------------------------------------------- | -------------- |
| CSRF        | `submit` 버튼 사용 시 서버는 POST 요청에 대해 CSRF 방어 필요 | CSRF 토큰 포함     |
| disabled 우회 | 클라이언트에서 `disabled` 제거 가능 → 서버에서 재검증 필수      |                |
| XSS         | 버튼을 통한 HTML 주입 방지                           | `innerHTML` 주의 |

---

## 🎯 10. 실무 팁

| 팁                           | 설명                  |
| --------------------------- | ------------------- |
| `type="button"`을 기본값으로 사용   | form 내에서 submit 방지용 |
| `disabled` + loading 표시     | 중복 클릭 방지 및 UX 개선    |
| 아이콘 버튼은 반드시 `aria-label` 사용 | 접근성 보장              |
| 상태에 따라 `class` 제어           | JS로 동적 스타일링         |

---

## ✅ 요약

| 항목         | 설명                                        |
| ---------- | ----------------------------------------- |
| 태그 이름      | `<button>`                                |
| 목적         | 클릭 가능한 UI 요소 생성                           |
| 핵심 속성      | `type`, `disabled`, `name`, `value`       |
| 기본 동작      | `submit`, `reset`, `button` (JS 이벤트로만 동작) |
| 접근성        | 우수 (키보드 사용 가능, 스크린리더 호환)                  |
| React 사용 시 | `onClick`, `className`, JSX 표현식 활용        |

