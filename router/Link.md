
# 🔗 `<Link>`란?

> `<Link>`는 React Router에서 **클라이언트 측 라우팅을 수행하기 위한 기본 컴포넌트**입니다.
> HTML의 `<a>` 태그와 유사하지만, **전체 페이지 새로고침 없이** 경로를 이동시킵니다.

```jsx
import { Link } from 'react-router-dom';

<Link to="/about">About</Link>
```

---

## 🚀 목적

| 기능             | 설명                                      |
| -------------- | --------------------------------------- |
| 클라이언트 라우팅      | 브라우저 URL을 바꾸고 컴포넌트를 재렌더링                |
| 전체 페이지 리로드 방지  | HTML `<a>`와는 달리 서버로 HTTP 요청을 보내지 않음     |
| history API 활용 | `pushState`를 통해 브라우저 기록 스택을 조작함         |
| 접근성 유지         | `<a>` 태그로 렌더링되기 때문에 키보드 내비게이션/스크린리더와 호환 |

---

# 🧪 기본 사용법

```jsx
<Link to="/products">Product List</Link>
```

* 브라우저 주소: `/products`로 변경
* React Router는 이에 해당하는 `<Route path="/products">`를 탐색해 렌더링

---

## 📦 `to` 속성

`<Link>`에서 가장 중요한 속성은 `to`입니다.
이 값은 이동할 **경로(path)** 또는 \*\*위치 객체(location object)\*\*를 의미합니다.

### 1️⃣ 문자열 경로

```jsx
<Link to="/contact">Contact</Link>
```

### 2️⃣ 상대 경로

```jsx
<Link to="settings">Settings</Link> // 현재 경로 기준 상대 이동
```

### 3️⃣ 객체 경로 (state 전달 포함)

```jsx
<Link to={{
  pathname: "/product/123",
  search: "?ref=home",
  hash: "#details",
  state: { fromDashboard: true }
}}>View Product</Link>
```

| 속성         | 설명                               |
| ---------- | -------------------------------- |
| `pathname` | 절대 or 상대 경로                      |
| `search`   | 쿼리 스트링 (예: `?q=react`)           |
| `hash`     | 앵커 (예: `#section1`)              |
| `state`    | 라우팅에 추가 데이터를 전달 (브라우저 히스토리에 저장됨) |

---

# ⚙️ 내부 동작 원리

1. `<Link>` 클릭 시:

   * `event.preventDefault()`로 기본 동작(페이지 새로고침)을 막고
   * React Router가 `history.push()` 또는 `navigate()`를 호출
2. URL이 변경됨
3. `Routes` 컴포넌트가 새 경로에 맞는 `<Route>`를 찾고 컴포넌트를 렌더링

---

## 🔁 `<Link>` vs `<a>` 차이

| 항목                | `<Link>` | `<a>`                 |
| ----------------- | -------- | --------------------- |
| 페이지 리로드           | ❌ 없음     | ✅ 있음                  |
| 클라이언트 라우팅         | ✅ 지원     | ❌ 비지원                 |
| SPA 방식            | ✅        | ❌                     |
| 브라우저 요청 발생        | ❌        | ✅ (서버 요청 발생)          |
| React Router에서 사용 | ✅ 필수     | ❌ 권장되지 않음 (단 예외 있음)\* |

> \*외부 링크나 파일 다운로드는 여전히 `<a href>` 사용

---

## 🧠 실전 팁

### 1. 조건부 활성화 링크

```jsx
<Link
  to="/profile"
  className={({ isActive }) => isActive ? 'active' : ''}
>
  Profile
</Link>
```

> `react-router-dom@6` 부터 `<NavLink>` 대신 `<Link>`도 className 함수 지원

---

### 2. 버튼처럼 보이는 링크

```jsx
<Link to="/next" className="btn btn-primary">Next</Link>
```

* `<Link>`는 시멘틱하게 `<a>`를 반환하므로 접근성도 유지됨
* 다만, 진짜 버튼 기능 (`submit`, `type="button"` 등)이 필요하다면 `<button onClick={navigate}>` 형태 추천

---

### 3. 객체 전달 시 `useLocation()`으로 state 접근

```jsx
<Link to="/confirm" state={{ from: "settings" }}>Go</Link>
```

```jsx
// Confirm.jsx
import { useLocation } from 'react-router-dom';

const location = useLocation();
console.log(location.state.from); // "settings"
```

---

### 4. `replace` 속성

```jsx
<Link to="/login" replace>Login</Link>
```

* 기본 동작은 `history.push()` → 뒤로 가기 가능
* `replace`는 `history.replace()` → 현재 히스토리 항목을 덮어쓰기 (뒤로 가기 불가능)

---

# ⚠️ 주의할 점

| 항목                          | 설명                                      |
| --------------------------- | --------------------------------------- |
| `to`는 필수                    | 생략 시 경로가 `undefined`가 되어 오류 발생          |
| 외부 링크엔 `<a>` 사용             | `target="_blank"`로 열거나 PDF 등은 `<a>`가 필요 |
| SSR에서는 `Link` → `<a>` 대체 필요 | 서버 렌더링 시에는 JS가 없을 수 있음                  |

---

## ✅ 요약

| 항목    | 설명                                      |
| ----- | --------------------------------------- |
| 역할    | React Router에서 경로 이동을 처리하는 컴포넌트         |
| 핵심 속성 | `to`, `replace`, `state`, `className` 등 |
| 동작 방식 | 페이지 리로드 없이 클라이언트 측 이동 수행                |
| 차이점   | `<a>`와 달리 SPA 구조 유지                     |
| 조합    | `useLocation`, `useNavigate`와 함께 자주 사용  |


