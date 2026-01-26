

# 🌟 React Router의 `useSearchParams`

URL 쿼리스트링(검색 파라미터)을 다루는 가장 쉬운 방법!

React Router v6에서 제공하는 `useSearchParams()`는 URL의 **쿼리 파라미터(Query String)**를 읽고 쓰기 위한 훅입니다.

---

# 📌 1. `useSearchParams`란 무엇인가?

`useSearchParams`는 아래 두 가지를 반환합니다:

1️⃣ **searchParams**
→ JavaScript의 `URLSearchParams` 인스턴스 (읽기 전용)

2️⃣ **setSearchParams**
→ URL의 쿼리 파라미터를 변경하는 함수

즉, URL의 `?key=value` 형태의 파라미터를 **React Router 방식으로 읽고 수정**할 수 있는 훅입니다.

---

# 🔎 2. 왜 필요한가?

### 🎯 1) URL이 상태(state)가 되는 시대

React SPA에서 URL은 단순 주소가 아니라 **상태를 저장하는 저장소** 역할도 합니다.

예:

* 검색 페이지: `?keyword=react&sort=latest`
* 페이지네이션: `?page=3`
* 필터링: `?category=shoes&price=10000`

이런 값들을 *전역 상태로 관리할 필요 없이*
URL 자체가 상태가 되므로 **Bookmark·새로고침·링크 공유**에 강합니다.

---

# 🧩 3. 기본 사용법 예제

```jsx
import { useSearchParams } from "react-router-dom";

export default function UserList() {
  const [searchParams, setSearchParams] = useSearchParams();

  const page = searchParams.get("page") || 1;
  const keyword = searchParams.get("keyword") || "";

  const updatePage = () => {
    setSearchParams({ page: Number(page) + 1, keyword });
  };

  return (
    <div>
      <p>현재 페이지: {page}</p>
      <p>검색어: {keyword}</p>

      <button onClick={updatePage}>다음 페이지</button>
    </div>
  );
}
```

---

# 📍 4. 내부에서 어떻게 동작하는가?

React Router는 다음 순서로 동작합니다:

### 1️⃣ 현재 URL을 분석

브라우저의 `window.location.search` 읽음
예: `?page=2&keyword=react`

### 2️⃣ 이를 `URLSearchParams` 객체로 파싱

```ts
new URLSearchParams("?page=2&keyword=react")
```

### 3️⃣ 컴포넌트는 이 값을 읽고 렌더링

### 4️⃣ `setSearchParams()` 호출 시:

* 쿼리 스트링을 새로 구성
* React Router가 History API의 `pushState()`를 호출
* URL이 바뀌면 해당 라우트가 다시 렌더링

즉, **useState + useEffect 없이 URL만 변경해도 리렌더링**됩니다.

---

# 🔧 5. setSearchParams 사용법 3가지

## ✔ 5-1) 객체로 설정하기 (가장 일반적)

```jsx
setSearchParams({ page: 3, sort: "latest" });
```

결과 URL:

```
?page=3&sort=latest
```

---

## ✔ 5-2) 기존 searchParams 복사 후 추가/수정

URL 유지하면서 특정 파라미터만 변경하고 싶을 때 유용합니다.

```jsx
const newParams = new URLSearchParams(searchParams);
newParams.set("page", 5);
setSearchParams(newParams);
```

---

## ✔ 5-3) 배열 값 사용 (여러 개의 동일 key)

```jsx
setSearchParams({ category: ["shoes", "coat"] });
```

결과:

```
?category=shoes&category=coat
```

---

# 📑 6. searchParams 주요 메서드

| 메서드                  | 설명           |
| -------------------- | ------------ |
| `get(key)`           | 첫 번째 값을 반환   |
| `getAll(key)`        | 모든 값을 배열로 반환 |
| `has(key)`           | key 존재 여부    |
| `set(key, value)`    | 값 설정 (덮어씌움)  |
| `append(key, value)` | 값 추가         |
| `delete(key)`        | 특정 key 삭제    |

예:

```js
searchParams.get("page")
searchParams.getAll("category")
```

---

# 🛠 7. 복잡한 필터/검색 UI에서의 활용 패턴

## ✔ 패턴 1) 검색 필터 상태 = URL 상태

예:

```
/products?category=shoes&sort=popular&minPrice=10000
```

## ✔ 패턴 2) 검색 인풋 변화 → URL 반영

```jsx
setSearchParams(prev => {
  const params = new URLSearchParams(prev);
  params.set("keyword", inputValue);
  return params;
});
```

## ✔ 패턴 3) 페이지 이동 시 페이지 번호 변경

React Router는 URL만 바꿔도 리렌더링되므로 매우 단순해짐.

---

# 🚨 8. 주의할 점 (실무에서 실수 많이 하는 부분)

### ⚠ 1. `searchParams`는 불변 객체처럼 취급해야 함

React Router는 변경을 감지해야 하므로
**항상 새 URLSearchParams 객체를 만들어서 setSearchParams에 넣어야 합니다.**

잘못된 코드 ❌

```js
searchParams.set("page", 5)
setSearchParams(searchParams)
```

올바른 코드 ⭕

```js
const p = new URLSearchParams(searchParams);
p.set("page", 5);
setSearchParams(p);
```

---

### ⚠ 2. setSearchParams는 replace와 push 모두 가능

기본은 **push** 동작입니다.
(히스토리 스택에 쌓임)

```jsx
setSearchParams({ page: 1 }, { replace: true });
```

---

### ⚠ 3. 배열 형태 전송 시 순서 보장 X

쿼리스트링 순서에 의존하지 않는 것이 좋습니다.

---

# 🌈 9. 완전한 실습 예제: 검색 + 필터 + 페이지네이션

아래와 같은 URL을 만들어봅니다:

```
/products?keyword=shoes&page=2&sort=latest
```

```jsx
import { useSearchParams } from "react-router-dom";

export default function Products() {
  const [searchParams, setSearchParams] = useSearchParams();

  const keyword = searchParams.get("keyword") || "";
  const page = Number(searchParams.get("page") || 1);
  const sort = searchParams.get("sort") || "latest";

  const updateSort = (newSort) => {
    const p = new URLSearchParams(searchParams);
    p.set("sort", newSort);
    setSearchParams(p);
  };

  const nextPage = () => {
    const p = new URLSearchParams(searchParams);
    p.set("page", page + 1);
    setSearchParams(p);
  };

  return (
    <div>
      <h2>상품 목록</h2>

      <p>검색어: {keyword}</p>
      <p>정렬 방법: {sort}</p>
      <p>현재 페이지: {page}</p>

      <button onClick={() => updateSort("popular")}>인기순</button>
      <button onClick={nextPage}>다음 페이지</button>
    </div>
  );
}
```

---

# 🎉 결론: 언제 `useSearchParams`를 사용해야 할까?

| 상황                  | 사용 여부                     |
| ------------------- | ------------------------- |
| 검색 결과 URL 공유        | ✔ 강력 추천                   |
| 페이지네이션              | ✔ 추천                      |
| 필터링/정렬              | ✔ 추천                      |
| 전역 상태처럼 URL에 상태를 저장 | ✔ 매우 추천                   |
| 컴포넌트 로컬 상태          | ❌ useState, useReducer 사용 |

