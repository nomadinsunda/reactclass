# 🧩 React Router v6 — `useParams()`

**“URL 경로에 포함된 동적 파라미터를 읽는 훅”**

SPA 라우팅에서 **URL 자체가 데이터의 일부**가 되면서
동적 라우팅(dynamic routing)은 매우 중요한 패턴이 되었습니다.

React Router의 `useParams()`는 URL 경로에 포함된
**`:paramName` 형태의 값**을 읽기 위한 핵심 훅입니다.

예:

```
/users/10
/products/abc123
/articles/react-router/hooks
```

여기서 `/users/:userId`에서 `userId = 10`
이 값을 가져오는 것이 `useParams()`입니다.

---

# 🎯 1. useParams란?

> routing path에 정의된 “동적 세그먼트(:param)”의 값을
> 객체 형태로 반환하는 React Router 훅

예:

```jsx
const { userId } = useParams();
```

---

# 📦 2. 기본 형태

```ts
const params = useParams();
// params: Record<string, string | undefined>
```

반환되는 값 예:

```json
{
  "userId": "10",
  "postId": "34"
}
```

각 value는 "문자열"입니다. 숫자라 하더라도 변환을 직접 해야 합니다.

---

# 🛣️ 3. params는 어디서 오는가? (중요)

Route 설정이 다음과 같다고 해보겠습니다:

```jsx
<Route path="/users/:userId" element={<UserDetail />} />
```

URL이:

```
/users/10
```

이면 React Router는 다음처럼 파싱합니다:

```
{ userId: "10" }
```

그리고 해당 컴포넌트에서 `useParams()`로 읽을 수 있습니다.

---

# 🔍 4. 가장 기본적인 예제

```jsx
import { useParams } from "react-router-dom";

export default function User() {
  const { userId } = useParams();

  return <h1>사용자 ID: {userId}</h1>;
}
```

URL:

```
/users/88
```

출력:

```
사용자 ID: 88
```

---

# 🧪 5. 여러 파라미터 사용

Route:

```jsx
<Route path="/users/:userId/posts/:postId" element={<Post />} />
```

컴포넌트:

```jsx
const { userId, postId } = useParams();
```

URL:

```
/users/88/posts/100
```

출력:

```
{ userId: "88", postId: "100" }
```

---

# 🔥 6. Nested Route에서 useParams는 부모 + 자식 params를 모두 제공

중첩 라우팅 구조 예:

```jsx
<Route path="users">
  <Route path=":userId">
    <Route path="posts/:postId" element={<UserPost />} />
  </Route>
</Route>
```

여기서 `/users/10/posts/333`이면:

```js
useParams() → { userId: "10", postId: "333" }
```

➡ 부모 라우트의 params도 그대로 전달받습니다.

---

# 🧠 7. 내부 동작 메커니즘 (정말 중요!)

React Router는 다음 과정으로 params를 계산합니다:

1. 현재 URL과 Route Tree를 비교(match)
2. path pattern에 포함된 모든 “:key”를 추출
3. URL 실제 segment 값과 매핑
4. 최종적으로 **params 객체** 생성
5. useParams() 훅 사용 컴포넌트가 재렌더링

➡ 즉, URL 변경 → match 변경 → params 변경 → 렌더링

---

# 🧨 8. useParams 실전 활용 패턴

---

## ✔ 패턴 1) 상세 페이지 로딩

```jsx
const { id } = useParams();

useEffect(() => {
  fetch(`/api/items/${id}`).then(...);
}, [id]);
```

---

## ✔ 패턴 2) URL 파라미터 변경에 반응하는 컴포넌트

`/products/10`에서 `/products/11`으로 이동 시
새로운 데이터를 다시 로드합니다.

---

## ✔ 패턴 3) params 기반 조건부 UI

```jsx
if (params.type === "admin") {
  return <AdminPage />;
}
return <UserPage />;
```

---

## ✔ 패턴 4) params 값 변환

문자열 → 숫자 변환이 필요할 때:

```js
const productId = Number(params.productId);
```

---

## ✔ 패턴 5) params + useNavigate

```jsx
const { userId } = useParams();
const navigate = useNavigate();

navigate(`/users/${userId}/edit`);
```

---

# 🧩 9. 다른 훅들과 비교 (강의용 표)

| 훅                   | 역할                               |
| ------------------- | -------------------------------- |
| **useParams**       | URL path의 동적 파라미터 읽기             |
| **useSearchParams** | URL 쿼리스트링 읽기/쓰기                  |
| **useLocation**     | pathname + search + hash + state |
| **useNavigate**     | 페이지 이동                           |
| **useResolvedPath** | 상대 경로 → 절대 경로 계산                 |

---

# 🚫 10. 주의할 점 (실무에서 많이 실수)

---

## ⚠ 1) params는 반드시 string

숫자라고 생각하면 안 됩니다.

예:

```
/users/10 → "10"
```

---

## ⚠ 2) params는 route에 정의된 name과 동일해야 함

```jsx
<Route path="/product/:id" />   // id
<Route path="/product/:productId" />  // productId
```

이름이 다르면 값이 나오지 않습니다.

---

## ⚠ 3) 없는 params는 undefined

```jsx
const { userId, postId } = useParams();
// postId가 없는 상황이라면 undefined
```

---

## ⚠ 4) useParams는 "search"를 못 읽음

쿼리스트링은 `useSearchParams` 사용해야 합니다.

---

# 🎉 11. 요약

| 항목     | 설명                       |
| ------ | ------------------------ |
| 기능     | URL 경로의 `:param` 값을 읽는 훅 |
| 반환     | `{ key: value }` 형태 객체   |
| value  | 항상 string                |
| 사용 위치  | 해당 라우트 하위 컴포넌트           |
| 중첩 라우팅 | 부모 params 자동 상속          |


