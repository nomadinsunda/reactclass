# 🧾 `<form>` 태그란?

## ✅ 정의

`<form>` 태그는 **사용자 입력 데이터를 서버로 전송**하기 위한 HTML 요소입니다.
텍스트 입력, 비밀번호, 체크박스, 드롭다운, 제출 버튼 등을 포함하며,
폼 내부의 데이터를 `submit` 이벤트를 통해 서버에 전송합니다.

```html
<form action="/submit" method="post">
  <!-- 입력 필드들 -->
</form>
```

---

## ✅ 주요 속성

| 속성             | 설명                                            |
| -------------- | --------------------------------------------- |
| `action`       | 폼 데이터를 전송할 **서버 주소(URL)**                     |
| `method`       | 전송 방식: `GET` 또는 `POST`                        |
| `target`       | 응답 받을 창의 이름 (`_self`, `_blank` 등)             |
| `enctype`      | 폼 데이터 인코딩 방식 (파일 업로드 시 `multipart/form-data`) |
| `autocomplete` | 자동완성 허용 여부 (`on` / `off`)                     |
| `novalidate`   | HTML 기본 유효성 검사 비활성화                           |

---

## ✅ form 구성 기본 예제

```html
<form action="/signup" method="post">
  <label for="username">사용자명:</label>
  <input type="text" id="username" name="username" required />
  <br />

  <label for="email">이메일:</label>
  <input type="email" id="email" name="email" required />
  <br />

  <label for="pw">비밀번호:</label>
  <input type="password" id="pw" name="password" required />
  <br />

  <input type="submit" value="회원가입" />
</form>
```

* `name` 속성은 **서버로 보낼 key 값**
* `required`는 **필수 입력 검사**
* `type="submit"`인 버튼을 누르면 form이 제출됨

---

## ✅ 전송 방식 차이: GET vs POST

| 방식     | 특징        | URL에 데이터 노출          | 보안       |
| ------ | --------- | -------------------- | -------- |
| `GET`  | 데이터 조회    | 예: `/search?q=hello` | 낮음       |
| `POST` | 데이터 등록/수정 | 노출되지 않음              | 상대적으로 안전 |

---

## ✅ 자바스크립트로 form 제어하기

```html
<form id="myForm">
  <input type="text" name="msg" />
  <button type="submit">전송</button>
</form>

<script>
  const form = document.getElementById("myForm");
  form.addEventListener("submit", function (e) {
    e.preventDefault(); // 페이지 새로고침 방지
    const formData = new FormData(form);
    const msg = formData.get("msg");
    alert("입력값: " + msg);
  });
</script>
```

---

## ✅ 폼 전송 없이 자바스크립트로 데이터 수집

```javascript
const data = new FormData(document.querySelector("form"));
console.log(data.get("username")); // name="username"인 값
```

---

## ✅ React에서 `<form>` 사용 예시

```jsx
import React, { useState } from "react";

function LoginForm() {
  const [form, setForm] = useState({ email: "", password: "" });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`이메일: ${form.email}, 비밀번호: ${form.password}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" onChange={handleChange} />
      <input name="password" type="password" onChange={handleChange} />
      <button type="submit">로그인</button>
    </form>
  );
}
```

---

## ✅ `<form>` 내부에서 자주 쓰는 태그

| 태그                      | 역할                  |
| ----------------------- | ------------------- |
| `<input>`               | 단일 텍스트, 체크박스, 라디오 등 |
| `<textarea>`            | 여러 줄 입력             |
| `<select>` + `<option>` | 드롭다운                |
| `<label>`               | 입력 요소와 연결           |
| `<button>`              | 제출, 초기화 등 실행 버튼     |

---

## ✅ `enctype` 종류 (파일 업로드 등)

| `enctype` 값                         | 설명                   |
| ----------------------------------- | -------------------- |
| `application/x-www-form-urlencoded` | 기본값 (모든 문자를 URL 인코딩) |
| `multipart/form-data`               | 파일 업로드용              |
| `text/plain`                        | 디버깅용 (거의 안 씀)        |

---

## ✅ 실습 퀴즈

1. `<form>`의 `action` 속성은 무엇을 지정하나요?
2. `type="submit"` 버튼을 누르면 어떤 이벤트가 발생하나요?
3. `enctype="multipart/form-data"`는 어떤 상황에 필요한가요?
4. React에서는 `form`을 어떻게 제어하나요?

---

## ✅ 요약

| 항목    | 설명                                       |
| ----- | ---------------------------------------- |
| 목적    | 사용자 입력을 서버로 전송                           |
| 필수 속성 | `action`, `method`                       |
| 주의    | `<form>`은 기본적으로 페이지 새로고침이 동반됨 (`submit`) |
| JS 활용 | `e.preventDefault()`, `FormData`로 제어 가능  |


