# 🎯 React Router v6 — `useMatch()`

**“현재 URL이 특정 패턴과 일치하는지(match) 확인하는 고급 훅”**

React Router의 `useMatch()`는

> **현재 URL이 특정 path 패턴과 일치하는지 검사하고, 매칭된 params까지 반환**
> 하는 훅입니다.

쉽게 말하면,

* “이 URL 패턴이 맞는지?”
* “맞다면 params 값은 무엇인지?”

를 확인해주는 **라우트 매칭 엔진**에 직접 접근하는 기능입니다.

SPA에서 “특정 컴포넌트가 어떤 URL에서 활성화(active) 상태인지” 판단할 때 매우 유용합니다.

---

# 🌟 1. useMatch란?

`useMatch(pattern)` 은 현재 URL이 pattern과 match될 경우:

* **매칭 정보 객체(match object)** 를 반환
* 매칭되지 않으면 `null` 반환

즉,
**URL 검사기(URL matcher)** 역할을 수행하는 훅입니다.

예:

```jsx
const match = useMatch("/users/:id");
```

현재 URL이 `/users/10` 이라면:

```js
{
  params: { id: "10" },
  pathname: "/users/10",
  pattern: { path: "/users/:id", ... }
}
```

---

# 🔍 2. useMatch() 기본 사용법

```jsx
import { useMatch } from "react-router-dom";

const match = useMatch("/products/:productId");
```

URL이 `/products/3`이면:

* match 객체 반환
* `params.productId === "3"`

URL이 `/products`이면:

* match = `null`

---

# 📦 3. useMatch가 반환하는 객체 구조

```ts
{
  params: Record<string, string>;
  pathname: string;
  pathnameBase: string;
  pattern: {
    path: string;
    caseSensitive?: boolean;
    end?: boolean;
  }
}
```

각 항목 설명:

### 🧩 params

`path`에 정의한 `:param` 값들

### 🛣 pathname

매칭된 전체 경로

### 🔧 pathnameBase

매칭된 영역의 기초 경로 (중첩 처리에 사용)

### 🔖 pattern

요청한 패턴(path) 정보

---

# 🧠 4. 왜 useMatch가 필요한가?

`useParams()`는 “현재 라우트에서” 정의된 param만 가져올 수 있습니다.
하지만 `useMatch()`는:

### ✔ **현재 라우트 트리 구조와 무관하게**

### ✔ **원하는 패턴만 주면 매칭 여부와 params를 직접 계산**

합니다.

즉, **라우팅 구조와 상관없이 URL만 보고 매칭**하기 때문에 더 자유롭습니다.

---

# 🧪 5. 실전 예제

## ✔ 예제 1) URL에 따라 메뉴 활성화 상태 표시

```jsx
import { useMatch } from "react-router-dom";

function NavItem() {
  const isActive = useMatch("/dashboard/*");

  return (
    <div className={isActive ? "active" : ""}>
      Dashboard
    </div>
  );
}
```

현재 URL이 `/dashboard`, `/dashboard/stats`, `/dashboard/users`
등이면 `active` 적용됨.

---

## ✔ 예제 2) 특정 패턴이 맞는지 확인하고 로직 실행

```jsx
const match = useMatch("/users/:id");

if (match) {
  console.log("User page", match.params.id);
}
```

---

## ✔ 예제 3) useMatch로 params 가져오기 (useParams 없이)

```jsx
const match = useMatch("/posts/:postId");

if (match) {
  console.log(match.params.postId);
}
```

➡ “현재 라우트와 무관하게” URL에 대한 매칭 가능.

---

## ✔ 예제 4) end 옵션 (정확히 일치해야 할 때)

```jsx
const exactMatch = useMatch({ path: "/about", end: true });
```

URL이 `/about/team`이면 일치하지 않음.

---

# 🧨 6. 중첩 라우팅보다 자유로운 이유

예:

현재 라우터 트리:

```
/app
   ├─ /products
   │      └─ :productId
```

현재 URL이:

```
/app/products/3
```

이 상황에서:

```jsx
useParams(); 
```

→ `productId: "3"` **가능**
(해당 라우트 안이라면)

하지만 컴포넌트가 이 트리 밖에 있다면?

→ `useParams()`는 값 못 가져옴 ❌

---

하지만:

```jsx
useMatch("/app/products/:productId");
```

→ 라우트 위치와 관계없이 매칭 가능 ⭕
→ `productId: "3"` 바로 가져옴

➡ **useMatch는 위치와 관계없는 범용 URL 매처**

---

# 🧬 7. 내부 동작 방식 (Match Algorithm)

React Router는 매칭 시 다음 과정을 수행합니다:

1. 패턴을 정규화
   `/users/:id` → 정규 패턴으로 변환
2. 현재 URL에서 pathname 추출
3. 경로 세그먼트 비교
4. 각 세그먼트가 `:파라미터`면 params로 저장
5. 모든 세그먼트가 매칭되면 match 객체 반환
6. 실패하면 null 반환

➡ 완전히 `path-to-regexp` 스타일의 매칭 엔진

---

# ⚠ 8. 실무에서 주의해야 할 점

---

### ❌ 1) useMatch는 “라우트 안에 있어야” 작동

React Router 컨텍스트 내부가 아니면 동작하지 않습니다.

---

### ❌ 2) useMatch는 “쿼리스트링(search)” 검사하지 않음

검색 파라미터는 무조건 `useSearchParams()` 사용해야 합니다.

---

### ❌ 3) URL 전체가 아닌 pathname만 검사

예:

```
/users/10?page=3
```

`useMatch("/users/:id")`는 매칭됨
쿼리는 무시됨.

---

# 🎉 9. 요약 정리

| 항목           | 내용                          |
| ------------ | --------------------------- |
| 역할           | URL이 특정 패턴과 일치하는지(match) 검사 |
| 반환           | match 객체 또는 null            |
| match.params | `:param` 값들                 |
| 장점           | 라우트 구조와 무관하게 URL 매칭 가능      |
| 주로 사용        | active UI, 조건부 렌더링, URL 캐치  |

---

# 📚 10. 관련 훅들과 비교 (강의용 표)

| 훅                   | 설명                     |
| ------------------- | ---------------------- |
| **useMatch**        | URL이 주어진 패턴과 맞는지 검사    |
| **useParams**       | 현재 라우터에서 정의된 param만 읽기 |
| **useSearchParams** | 쿼리스트링 읽기/쓰기            |
| **useLocation**     | 현재 URL 전체 정보           |
| **useNavigate**     | 이동/redirect            |
| **useResolvedPath** | 상대경로 → 절대경로 변환         |

