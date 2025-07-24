
# 🔧 JSX의 Self-closing 태그란?

## ✅ 1. 정의

> **Self-closing tag**란
> **자식 요소(children)가 없는 태그**를
> `</태그>` 없이 **닫는 슬래시(`/`)를 사용하여 한 줄로 종료**하는 JSX 문법입니다.

```jsx
<br />
<img src="..." />
<UserList />
```

JSX에서는 **XML과 같은 문법 규칙을 따르기 때문에**
모든 태그는 반드시 **명시적으로 닫혀야** 합니다.

---

## 🧩 2. 일반 태그와의 비교

| 형식              | 의미                   | 사용 예                         |
| --------------- | -------------------- | ---------------------------- |
| `<Comp></Comp>` | 열고 닫는 태그 (자식 포함 가능)  | `<Button>확인</Button>`        |
| `<Comp />`      | Self-closing (자식 없음) | `<Button />`, `<UserList />` |

---

## 📌 3. 사용 가능한 대상

### ✅ HTML 태그

```jsx
<input type="text" />
<img src="logo.png" alt="로고" />
<br />
<hr />
```

> HTML에서는 `<br>` 같이 닫지 않아도 되지만,
> **JSX에서는 반드시 `<br />`처럼 닫는 슬래시가 있어야** 문법 오류가 나지 않습니다.

---

### ✅ React 컴포넌트

```jsx
function Header() {
  return <h1>Welcome</h1>;
}

<Header />  // ✅ Self-closing 사용 가능
```

컴포넌트에 자식이 필요 없는 경우, `<Header />`처럼 셀프 클로징으로 쓰는 것이 일반적입니다.

---

## ⚠️ 4. JSX의 문법 특징

* **JSX는 HTML이 아니라 JavaScript 문법입니다.**
* JSX는 Babel을 통해 `React.createElement(...)`로 변환되기 때문에 **XML처럼 모든 태그를 명확히 닫아야 합니다.**
* HTML과 달리 `<br>`, `<input>`과 같은 태그도 JSX에서는 반드시 `<br />`, `<input />`처럼 써야 오류가 나지 않습니다.

```jsx
// ❌ 오류
<img src="logo.png">

// ✅ 올바른 문법
<img src="logo.png" />
```

---

## ✅ 5. 요약

| 항목       | 내용                                         |
| -------- | ------------------------------------------ |
| 이름       | Self-closing 태그                            |
| 문법       | `<태그명 />`                                  |
| 용도       | 자식 요소가 없을 때 간단히 닫기                         |
| HTML 예시  | `<br />`, `<img />`, `<input />`           |
| React 예시 | `<Header />`, `<Footer />`, `<UserList />` |
| 필수 이유    | JSX는 XML 기반 → **모든 태그는 닫혀야 함**             |

---

## 💬 결론

> **JSX에서 Self-closing 태그는**
>
> * 자식 요소가 없는 태그를 짧게 표현할 수 있게 해주는 문법이며,
> * JSX의 **XML-like 문법 규칙** 때문에 반드시 **닫는 슬래시(`/`)가 필수**입니다.

