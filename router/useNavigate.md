# 🚀 React Router v6 — `useNavigate()`

**“SPA에서 페이지 이동을 제어하는 핵심 네비게이션 훅”**

React Router v6에서 `useNavigate()`는
React 앱에서 페이지 이동을 프로그래밍적으로 제어하기 위한 **가장 중요한 훅**입니다.

`<Link>`는 “사용자가 클릭했을 때”만 이동되지만,
`useNavigate()`는 **이벤트, 조건, 비동기 작업, API 응답 등 원하는 순간에**
코드로 직접 라우팅을 변경할 수 있게 해줍니다.

---

# 🎯 1. useNavigate란?

React Router에서 제공하는 훅으로,

> **현재 라우터 컨텍스트에서 특정 URL로 이동(push/replace)하는 함수**를 반환합니다.

즉:

### ✔ 버튼 클릭 → 페이지 이동

### ✔ API 응답 후 → 페이지 이동

### ✔ 로그인 성공 시 → 홈으로 이동

### ✔ 특정 조건 만족 시 → redirect

### ✔ 뒤로가기/앞으로가기 로직도 직접 제어

이 모든 것을 `useNavigate()` 하나로 처리합니다.

---

# 📦 2. 기본 사용법

```jsx
import { useNavigate } from "react-router-dom";

export default function Example() {
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate("/about")}>
      소개 페이지로 이동
    </button>
  );
}
```

이제 버튼 클릭 시
➡ 브라우저 URL이 `/about`으로 변경
➡ 해당 라우트 컴포넌트 렌더링

---

# 🔥 3. navigate 함수의 시그니처

```ts
navigate(
  to: string | number,
  options?: {
    replace?: boolean;
    state?: any;
  }
)
```

### ✔ 1) `to: string`

이동할 경로
`"/about"`, `"../settings"`, `"profile"` 가능

### ✔ 2) `to: number`

히스토리 이동도 가능
`navigate(-1)` → 뒤로가기
`navigate(1)` → 앞으로가기

### ✔ 3) options.replace

`replace: true` → 히스토리 스택 교체 (push 아님)

### ✔ 4) options.state

URL에 보이지 않는 내부 데이터 전달

---

# 🧠 4. push vs replace (매우 중요한 개념)

React Router는 기본적으로 **push** 동작입니다.

```jsx
navigate("/home"); // pushState
```

즉, 브라우저 히스토리 스택에 새로운 엔트리가 추가됩니다.

---

## 🔁 replace (이전 기록을 덮어쓰기)

```jsx
navigate("/login", { replace: true });
```

예:
로그인 성공 후 `/login` 페이지로 다시 돌아가는 것을 막기 위해 replace 사용

➡ 실제 실무에서 매우 많이 사용됩니다.

---

# 🔎 5. state 옵션 — URL에 노출되지 않는 데이터 전달

```jsx
navigate("/detail/10", {
  state: { from: "product-list", scrollY: 320 }
});
```

읽기:

```jsx
import { useLocation } from "react-router-dom";

const location = useLocation();
console.log(location.state);
```

> ⛔ 단, state는 브라우저 새로고침 시 사라집니다.
> 절대 중요한 값 저장용으로 쓰지 않습니다!

---

# 🧭 6. 상대 경로(relative path) 이동도 가능

현재 경로가 `/users/10/profile`이라면,

```jsx
navigate("../settings");
```

최종 URL:

```
/users/10/settings
```

➡ 내부적으로 `useResolvedPath()`가 동작합니다.

---

# 🧪 7. 실전 예제 모음

---

## ✔ 예제 1) 로그인 성공 후 홈으로 이동

```jsx
const navigate = useNavigate();

async function handleLogin() {
  const ok = await loginAPI();
  if (ok) navigate("/", { replace: true });
}
```

---

## ✔ 예제 2) 폼 제출 후 상세 페이지로 이동

```jsx
navigate(`/item/${itemId}`);
```

---

## ✔ 예제 3) 뒤로 가기 구현

```jsx
navigate(-1);
```

---

## ✔ 예제 4) 조건부 리디렉션

```jsx
if (!user) {
  navigate("/login");
}
```

---

## ✔ 예제 5) state 전달로 메타 정보 저장

```jsx
navigate("/checkout", {
  state: { fromCart: true, couponCode }
});
```

---

## ✔ 예제 6) 현재 URL에서 상대 이동

현재 `/dashboard/settings/profile`

```jsx
navigate("../security");
```

결과:

```
/dashboard/settings/security
```

---

# 🧨 8. useNavigate + 비동기 조합 (실무 필수 패턴)

```jsx
const navigate = useNavigate();

async function onSubmit(data) {
  const result = await saveData(data);
  if (result.success) {
    navigate(`/result/${result.id}`);
  }
}
```

---

# 🧩 9. 사용 빈도 높은 패턴 모음

### 🔹 인증/인가(Authorization) Redirect

### 🔹 장바구니 → 주문서 이동

### 🔹 검색 → 결과 페이지 이동

### 🔹 목록 → 상세 페이지 이동

### 🔹 작업 성공 후 특정 화면으로 이동

SPA에서 “버튼 도움 없이 이동해야 하는 상황” 대부분을 해결합니다.

---

# 🧠 10. 내부 동작 메커니즘 (History API 기반)

`useNavigate()`는 내부적으로 다음 작업을 수행합니다:

1. React Router가 브라우저의 history 객체를 감싸서 관리
2. navigate()가 호출되면 history.pushState() 또는 history.replaceState() 실행
3. React Router가 새로운 location 객체 생성
4. 해당 location을 사용하는 컴포넌트가 리렌더링됨
5. 화면이 업데이트됨

즉, navigate는 “URL → location → 렌더링” 전체 흐름을 트리거합니다.

---

# 🎉 11. 요약

| 항목    | 설명                                       |
| ----- | ---------------------------------------- |
| 역할    | 프로그래밍 방식 페이지 이동                          |
| 지원    | push, replace, relative path, history 이동 |
| state | navigate 간 메타 데이터 전달                     |
| 내부 기반 | History API                              |
| 대표 패턴 | 로그인, 폼 제출, 비동기 후 이동                      |

---

# 📚 12. 관련 훅과의 비교 (강의용 표)

| 훅               | 역할             |
| --------------- | -------------- |
| useNavigate     | 이동(redirect)   |
| useLocation     | 현재 위치 정보       |
| useParams       | URL 파라미터       |
| useSearchParams | 쿼리스트링 읽기/쓰기    |
| useResolvedPath | 상대경로 → 절대경로 계산 |


