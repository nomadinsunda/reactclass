# 🔗 React Router v6 — `useResolvedPath()`

**“상대 경로를 절대 경로로 변환하는 훅”**

SPA 라우팅에서 **상대 경로(relative path)** 문제는 컴포넌트 구조가 깊어질수록 까다로워집니다.
`useResolvedPath()`는 이러한 상대 경로를 **현재 라우트 기준의 절대 경로로 계산해주는 훅**입니다.

---

# 🎯 1. useResolvedPath란?

React Router의 `useResolvedPath()`는 아래를 수행합니다:

> **현재 라우트의 위치(context)를 기준으로**
> **주어진 경로(path)를 “절대 경로(absolute path)”로 계산해주는 훅**

즉,
`"../users"` → `"/admin/users"`
`"profile"` → `"/users/10/profile"`
처럼 **상대 경로 → 완전한 절대 경로**로 만들어줍니다.

---

# 🧭 2. 왜 필요한가?

### ⭕ 중첩 라우팅이 많아질 때

예를 들어 `/users/:id/posts` 안에서
`../settings` 경로를 계산하려면 직접 문자열 조합하기 어렵습니다.

### ⭕ `<Link to="relative">` 동작을 컴포넌트 외부에서 계산할 때

React Router 내부는 `<Link>`가 자동으로 상대경로를 처리하지만,
컴포넌트에서 “이 버튼 클릭 시 이동할 URL을 미리 계산”할 때 필요합니다.

### ⭕ 네비게이션 메뉴에서 동적 경로 생성

메뉴가 깊은 중첩 안에 있을 때도 정상적으로 동작하게 하려면
항상 절대 경로가 필요합니다.

---

# 📦 3. useResolvedPath 반환값 구조

```ts
{
  pathname: string;
  search: string;
  hash: string;
}
```

예:

```
{
  pathname: "/users/10/profile",
  search: "",
  hash: ""
}
```

즉, `useLocation()`의 일부와 같은 구조입니다.

---

# 🧪 4. 기본 예시

현재 URL이:

```
/users/10
```

이 상태에서:

```jsx
const resolved = useResolvedPath("profile");
console.log(resolved.pathname);
```

결과:

```
/users/10/profile
```

---

# 📍 5. 중첩 라우팅에서의 강력함

현재 라우트:

```
/dashboard/settings/profile
```

코드:

```jsx
const resolved = useResolvedPath("../security");

console.log(resolved.pathname);
```

결과:

```
/dashboard/settings/security
```

➡ 자동으로 “한 단계 위로 올라간 후” 경로 계산합니다.

---

# 🔥 6. 실전 예제 — UI에서 절대 URL 계산하기

예를 들어, 메뉴에서 “현재 라우트와 관계없이”
항상 정확한 이동 경로를 생성하고 싶을 때:

```jsx
import { useResolvedPath, useNavigate } from "react-router-dom";

export default function MenuButton() {
  const target = useResolvedPath("../edit");
  const navigate = useNavigate();

  return (
    <button onClick={() => navigate(target.pathname)}>
      Edit Page 이동
    </button>
  );
}
```

현재 위치가 `/users/10/profile`이라면:

```
../edit → /users/10/edit
```

정확하게 계산됩니다.

---

# 🔍 7. useResolvedPath vs useLocation vs useMatch 비교

| 기능             | useResolvedPath | useLocation | useMatch |
| -------------- | --------------- | ----------- | -------- |
| 상대 경로 계산       | ✔               | ❌           | ❌        |
| 현재 URL 정보      | ❌               | ✔           | ❌        |
| 특정 패턴 매칭       | ❌               | ❌           | ✔        |
| 쿼리/해시 포함       | ✔               | ✔           | ❌        |
| 동적 이동 경로 미리 계산 | ✔               | ❌           | ❌        |

강의하시면서 비교 시각자료로 쓰기 좋은 표입니다.

---

# 🛠 8. 내부 동작 메커니즘 (깊이 설명)

React Router는 “라우트 트리(Route Tree)”를 구성합니다.

예를 들어:

```
/users
  └── :id
        └── profile
```

각 라우트는 **parent route의 경로 정보를 상속**합니다.
`useResolvedPath()`는 이 트리를 역추적하여:

1. 현재 라우트 객체(location 기반)
2. 부모 라우트들의 base 경로
3. 전달된 상대경로(../, ./, 경로 조합)

이를 모두 조합해 **정확한 절대 경로를 계산**합니다.

➡ 결국, Node.js의 `path.resolve()`와 비슷한 개념입니다.

---

# 🧨 9. 복잡한 예제 — 다단계 네스팅

현재 URL:

```
/app/accounts/42/details/profile
```

코드:

```jsx
const a = useResolvedPath("../../settings/security");
```

계산 과정:

1. `../../` → `details` → `42`로 두 단계 위로
2. 그 뒤 `settings/security` 붙이기

최종 결과:

```
/app/accounts/42/settings/security
```

➡ 문자열로 직접 계산하기 매우 복잡한 상황도 자동 처리!

---

# 📌 10. 주의사항 (실무에서 자주 실수하는 부분)

### ⚠ 1) `useResolvedPath`는 실제 라우팅을 변경하지 않음

단지 경로를 계산하는 역할입니다.

### ⚠ 2) navigation 시 반드시 navigate 또는 Link가 필요

```jsx
navigate(resolved.pathname)  // OK
```

### ⚠ 3) 상대 경로의 기준은 “현재 라우트 아님 → 현재 URL”

중첩 <Route> 구조가 아니라 “현재 URL”이 기준입니다.

---

# 🎉 11. 요약

| 항목     | 설명                          |
| ------ | --------------------------- |
| 역할     | 상대 경로를 절대 경로로 변환            |
| 기반     | 현재 URL (useLocation)        |
| 리턴     | pathname, search, hash 객체   |
| 용도     | 메뉴, 버튼, 동적 컴포넌트 내부에서 URL 계산 |
| 중첩 라우팅 | 매우 강력                       |


