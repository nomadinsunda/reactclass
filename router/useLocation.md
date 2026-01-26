

# 🌍 React Router v6 – `useLocation()`

**“현재 URL 정보를 가장 정확하게 읽어오는 훅”**

React Router v6의 `useLocation()`은
**현재 라우트가 가진 모든 URL 정보를 담고 있는 객체**를 반환하는 훅입니다.

SPA 라우팅에서 URL은 단순 주소가 아니라 “현재 뷰(View)의 상태”를 표현하는 중요한 요소입니다. `useLocation()`은 이 URL 상태를 그대로 받아와 컴포넌트 UI 렌더링에 활용할 수 있게 해줍니다.

---

# 🎯 1. useLocation이란?

React Router가 관리하는 **현재 위치 객체(location object)**를 읽어오는 훅입니다.

즉, 브라우저의 주소창이 바뀌면 React Router 내부에서 “location” 객체를 새로 만들고
→ 이에 반응하여 컴포넌트가 다시 렌더링됩니다.

---

# 📦 2. useLocation이 반환하는 값

```ts
{
  pathname: string;    // 예: "/users/10"
  search: string;      // 예: "?page=2&sort=latest"
  hash: string;        // 예: "#section-1"
  state: any;          // navigate(state)로 전달한 사용자 정의 데이터
  key: string;         // location을 식별하는 고유 키 (React Router가 내부에서 사용)
}
```

각 항목을 좀 더 상세하게 보겠습니다.

---

# 🔍 3. 주요 필드 상세 설명

## 🛣 pathname

현재 URL의 경로입니다.

* `/`
* `/about`
* `/users/10`
* `/products/watch/detail`

이 값이 변하면 해당 라우트에 매핑된 컴포넌트가 렌더링됩니다.

---

## 🔎 search (쿼리스트링)

URL의 **? 뒤에 붙는 문자열**입니다.

예:

```
?page=3&keyword=react
```

이 값은 그대로 문자열로 들어갑니다.
쿼리 파싱은 `useSearchParams()`로 하는 것이 표준입니다.

---

## 🧩 hash

URL의 **#** 뒤의 값입니다.

예:

```
http://example.com/#section2
```

해당 페이지 내 특정 지점 스크롤 이동 등에 사용됩니다.

---

## 📦 state

`navigate(path, { state: {...} })` 를 사용할 때
URL이 아닌 **내부 히스토리 상태에 저장되는 데이터**입니다.

예:

```jsx
navigate('/detail/10', {
  state: { from: 'home', scrollY: 220 }
});
```

`useLocation()`으로 읽을 수 있습니다:

```jsx
const location = useLocation();
console.log(location.state);
```

SPA에서 URL에 넣기 애매한 **메타 정보**를 전달할 때 유용합니다.

---

## 🔑 key

각 location을 고유하게 식별하는 문자열입니다.
React Router가 내부적으로 **히스토리 스택 관리**할 때 사용합니다.

예:

```
key: "d34j9d"
```

직접 사용할 일은 거의 없습니다.

---

# 🧠 4. 내부 동작 메커니즘 (History API 기반)

React Router는 브라우저의 **History API**를 감싸서 관리합니다.

## 1️⃣ 브라우저에서 URL이 변경됨

* 사용자가 `<Link>` 클릭
* `navigate()` 호출
* 브라우저의 뒤로가기/앞으로가기
* setSearchParams로 쿼리 변경

## 2️⃣ React Router 내부에서 새로운 location 객체 생성

새로운 `location`을 만들고 React 전역 Context에 저장합니다.

## 3️⃣ useLocation 훅이 이 값을 구독

location이 변경되면 useLocation을 사용하는 컴포넌트가 **리렌더링**됩니다.

➡ 즉, URL 자체가 React의 상태처럼 동작합니다.

---

# 🧪 5. 기본 예시

```jsx
import { useLocation } from "react-router-dom";

export default function LocationDemo() {
  const location = useLocation();

  return (
    <div>
      <p>현재 경로: {location.pathname}</p>
      <p>검색값: {location.search}</p>
      <p>해시: {location.hash}</p>
      <p>state: {JSON.stringify(location.state)}</p>
    </div>
  );
}
```

---

# 🧭 6. useLocation 실전 활용 패턴

---

## ✔ 패턴 1) 경로에 따라 UI 조건부 렌더링

```jsx
const { pathname } = useLocation();

return (
  <div>
    {pathname.startsWith('/admin') ? <AdminNav /> : <MainNav />}
  </div>
);
```

---

## ✔ 패턴 2) `state`로 전달받은 정보 활용

```jsx
navigate('/product/10', {
  state: { from: 'search', keyword: 'shoes' }
});
```

```jsx
const location = useLocation();
console.log(location.state.keyword); // shoes
```

검색 페이지로부터 넘어온 메타 데이터 등을 저장할 때 매우 유용합니다.

---

## ✔ 패턴 3) 뒤로 가기 시 특정 스크롤 복원

상세 페이지 진입 전 스크롤 위치 저장:

```jsx
navigate(`/detail/${id}`, {
  state: { scrollY: window.scrollY }
});
```

상세 페이지에서 “뒤로 가기” 후 목록 스크롤 복원:

```jsx
const { state } = useLocation();

useEffect(() => {
  if (state?.scrollY != null) {
    window.scrollTo(0, state.scrollY);
  }
}, []);
```

---

## ✔ 패턴 4) 라우팅 기반 애니메이션

페이지 전환 애니메이션 시 key를 이용:

```jsx
const location = useLocation();

// AnimatePresence key 기준 변경
<AnimatePresence mode="wait">
  <motion.div key={location.pathname}>
    <Routes location={location}>
      ...
    </Routes>
  </motion.div>
</AnimatePresence>
```

Framer Motion 조합에서 매우 중요한 패턴입니다.

---

# ⚠ 7. 주의할 점

## ❌ 1) location 객체는 “읽기 전용”

직접 수정해도 의미 없습니다.

```js
location.pathname = "/abc"; // 작동 안함❌
```

수정은 반드시 `navigate()` 또는 `<Link>`로 해야 합니다.

---

## ❌ 2) location 변경 = 컴포넌트 리렌더링

URL이 변하면 useLocation을 사용하는 컴포넌트는 무조건 렌더링됩니다.
무거운 컴포넌트라면 메모이제이션(useMemo) 또는 분리 필요.

---

## ❌ 3) state는 URL이 아닌 history stack에 저장됨

새로고침(F5) 하면 **state가 사라집니다**.

그래서 “중요한 데이터 저장용”으로는 쓰면 안 됩니다.
(로컬스토리지나 서버 URL Query에 저장해야 함)

---

# 🎉 8. 요약

| 항목          | 의미                   |
| ----------- | -------------------- |
| useLocation | 현재 URL 객체를 반환        |
| pathname    | 경로 (/users/10)       |
| search      | 쿼리 (?page=2)         |
| hash        | 앵커 (#section3)       |
| state       | navigate로 전달한 임시 데이터 |
| key         | location 고유 키 (내부용)  |

---

# 📚 9. useSearchParams와 비교 (강사님 강의용 표)

| 기능          | useLocation   | useSearchParams   |
| ----------- | ------------- | ----------------- |
| 전체 URL 정보   | ✔             | ❌ (쿼리만)           |
| pathname 읽기 | ✔             | ❌                 |
| hash 읽기     | ✔             | ❌                 |
| state 읽기    | ✔             | ❌                 |
| 쿼리 읽기       | ✔ (문자열)       | ✔ (객체 형태)         |
| 쿼리 수정       | ❌             | ✔                 |
| 리렌더 트리거     | location 변경 시 | searchParams 변경 시 |


