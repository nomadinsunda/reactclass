# React Router의 Dynamic Segment와 Route Parameter  `:` 문법 이해하기

React Router의 Route Path에서 `:`(콜론)은 **Dynamic Segment(동적 세그먼트)** 를 선언하는 문법입니다.

예를 들어:

```jsx id="a1"
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

여기서:

```text id="a2"
:userId
```

는 고정된 문자열이 아니라 **URL에 따라 달라질 수 있는 값**을 의미합니다.

즉:

```text id="a3"
/users/1
/users/42
/users/abc123
```

처럼 `/users/` 뒤에 서로 다른 값이 들어와도 같은 Route Pattern으로 처리할 수 있습니다.

---

# 1. 한 문장으로 정의하면

> React Router에서 `:`는 URL Path의 한 세그먼트를 동적 값으로 선언하고, 그 값을 지정한 이름으로 Route Parameter에 저장하도록 만드는 문법입니다.

예를 들어:

```text id="a4"
/users/:userId
```

는 다음과 같이 해석할 수 있습니다.

```text id="a5"
/users/       :userId
   │             │
   │             └── Dynamic Segment
   │
   └──────────────── Static Segment
```

현재 URL이:

```text id="a6"
/users/50
```

이라면:

```text id="a7"
:userId
   │
   ▼
 "50"
```

으로 매칭됩니다.

---

# 2. Static Segment와 Dynamic Segment

다음 Route를 보겠습니다.

```jsx id="a8"
<Route path="/users/:userId" />
```

이 Path에는 두 종류의 세그먼트가 있습니다.

```text id="a9"
/users/:userId
│      │
│      └── Dynamic Segment
│
└───────── Static Segment
```

`users`는 정확하게 같은 문자열이어야 합니다.

반면 `:userId`는 해당 위치에 들어오는 값을 받아들입니다.

예를 들어:

```text id="a10"
/users/1
/users/100
/users/kim
```

은 모두 매칭될 수 있습니다.

하지만:

```text id="a11"
/products/1
```

은 `/users/` 부분이 다르기 때문에 매칭되지 않습니다.

---

# 3. `:userId`에서 `userId`는 무엇인가?

`userId`는 **Parameter의 이름**입니다.

```text id="a12"
:userId
 │
 └── Parameter 이름
```

현재 URL이:

```text id="a13"
/users/42
```

라면 React Router는 개념적으로 다음과 같은 값을 만듭니다.

```js id="a14"
{
  userId: '42'
}
```

즉:

```text id="a15"
/users/:userId
        │
        ▼

/users/42
        │
        ▼

params.userId = "42"
```

입니다.

---

# 4. Dynamic Segment는 URL 한 세그먼트를 매칭한다

다음 Route를 보겠습니다.

```jsx id="a16"
<Route path="/users/:userId" />
```

여기서 `:userId`는 보통 `/`로 구분되는 **하나의 Path Segment**를 매칭합니다.

예:

```text id="a17"
/users/10
       └── userId
```

또는:

```text id="a18"
/users/abc123
       └────┘
        userId
```

처럼 하나의 세그먼트 값을 받습니다.

따라서 `:userId`를 단순히:

> 뒤에 어떤 문자열이 와도 전부 매칭된다.

라고 표현하는 것보다는:

> **해당 Path Segment 위치에 들어오는 값을 동적으로 매칭한다.**

라고 설명하는 것이 더 정확합니다.

---

# 5. 실제 매칭 예제

Route:

```jsx id="a19"
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

다음 URL들은 모두 이 Route와 매칭될 수 있습니다.

| URL             | Parameter           |
| --------------- | ------------------- |
| `/users/1`      | `userId = "1"`      |
| `/users/42`     | `userId = "42"`     |
| `/users/abc123` | `userId = "abc123"` |
| `/users/kim`    | `userId = "kim"`    |

즉 하나의 Route Pattern으로 여러 URL을 처리할 수 있습니다.

```text id="a20"
/users/1
/users/42
/users/abc123
     │
     ▼
/users/:userId
     │
     ▼
<UserDetail />
```

---

# 6. `useParams()`로 값 읽기

React Router가 Dynamic Segment를 매칭하면 그 값은 Route Parameter가 됩니다.

컴포넌트에서는 `useParams()`를 사용하여 읽을 수 있습니다.

```jsx id="a21"
import { useParams } from 'react-router-dom';

function UserDetail() {
  const { userId } = useParams();

  return (
    <h1>
      User ID: {userId}
    </h1>
  );
}

export default UserDetail;
```

현재 URL:

```text id="a22"
/users/50
```

이라면:

```jsx id="a23"
const { userId } = useParams();
```

에서:

```text id="a24"
userId = "50"
```

이 됩니다.

화면에는:

```text id="a25"
User ID: 50
```

이 출력됩니다.

---

# 7. 전체 동작 흐름

다음 Route가 있다고 하겠습니다.

```jsx id="a26"
<Route
  path="/users/:userId"
  element={<UserDetail />}
/>
```

현재 URL:

```text id="a27"
/users/88
```

React Router의 동작을 개념적으로 보면:

```text id="a28"
현재 URL
/users/88
     │
     ▼
Route Pattern
/users/:userId
     │
     ▼
Route Matching
     │
     ├── users ↔ users
     │
     └── :userId ↔ 88
                    │
                    ▼
             params 생성
                    │
                    ▼
          { userId: "88" }
                    │
                    ▼
              useParams()
```

입니다.

핵심은 `useParams()`가 URL 문자열을 직접 분석하는 것이 아니라는 점입니다.

먼저 React Router가 Route Matching을 수행하고:

```text id="a29"
:userId = "88"
```

이라는 Parameter를 만든 뒤, `useParams()`가 그 값을 읽습니다.

---

# 8. 여러 개의 Dynamic Segment도 사용할 수 있다

Route Path에 여러 개의 Dynamic Segment를 선언할 수도 있습니다.

```jsx id="a30"
<Route
  path="/products/:category/:productId"
  element={<ProductDetail />}
/>
```

현재 URL:

```text id="a31"
/products/electronics/100
```

이라면:

```text id="a32"
/products/:category/:productId
          │         │
          ▼         ▼
   electronics     100
```

React Router는 다음과 같은 Parameter를 만듭니다.

```js id="a33"
{
  category: 'electronics',
  productId: '100'
}
```

컴포넌트에서는:

```jsx id="a34"
const {
  category,
  productId
} = useParams();
```

처럼 읽을 수 있습니다.

---

# 9. Parameter 이름은 개발자가 정한다

다음 두 Route는 Parameter 이름이 다릅니다.

```jsx id="a35"
<Route path="/users/:id" />

<Route path="/users/:userId" />
```

첫 번째 Route:

```text id="a36"
:id
```

이면:

```jsx id="a37"
const { id } = useParams();
```

로 읽어야 합니다.

두 번째 Route:

```text id="a38"
:userId
```

이면:

```jsx id="a39"
const { userId } = useParams();
```

로 읽습니다.

즉:

```text id="a40"
: 뒤의 이름
      │
      ▼
params 객체의 key
```

가 됩니다.

---

# 10. Parameter 값은 문자열이다

현재 URL이:

```text id="a41"
/users/100
```

이라고 해도:

```jsx id="a42"
const { userId } = useParams();
```

의 `userId`는 숫자 `100`이 아니라 문자열입니다.

```js id="a43"
userId === '100'
```

숫자가 필요하다면 직접 변환해야 합니다.

```jsx id="a44"
const id = Number(userId);
```

즉:

```text id="a45"
URL
/users/100
        │
        ▼
params.userId
        │
        ▼
      "100"
        │
        ▼
   Number(...)
        │
        ▼
       100
```

입니다.

---

# 11. Dynamic Segment를 사용하는 이유

Dynamic Segment가 없다면 사용자마다 Route를 따로 정의해야 할 수도 있습니다.

예를 들어:

```jsx id="a46"
<Route path="/users/1" />
<Route path="/users/2" />
<Route path="/users/3" />
```

이 방식은 현실적으로 사용할 수 없습니다.

대신:

```jsx id="a47"
<Route path="/users/:userId" />
```

하나만 정의합니다.

그러면:

```text id="a48"
/users/1
/users/2
/users/3
/users/1000
```

등을 모두 같은 Route로 처리할 수 있습니다.

즉:

> Dynamic Segment의 가장 큰 목적은 하나의 Route Pattern으로 여러 실제 URL을 처리하는 것입니다.

---

# 12. 대표적인 활용 사례

Dynamic Segment는 특히 상세 페이지에서 많이 사용됩니다.

## 사용자 상세

```text id="a49"
/users/:userId
```

예:

```text id="a50"
/users/10
/users/20
```

## 상품 상세

```text id="a51"
/products/:productId
```

예:

```text id="a52"
/products/100
/products/200
```

## 게시글 상세

```text id="a53"
/articles/:articleId
```

## 카테고리

```text id="a54"
/categories/:category
```

## 여러 단계의 리소스

```text id="a55"
/users/:userId/posts/:postId
```

이처럼 **리소스의 식별값을 URL Path에 포함시켜야 할 때** Dynamic Segment가 특히 적합합니다.

---

# 13. Path Parameter와 Query Parameter는 다르다

다음 URL을 보겠습니다.

```text id="a56"
/users/10?page=2
```

여기에는 서로 다른 두 종류의 Parameter가 있습니다.

```text id="a57"
/users/10?page=2
       │      │
       │      └── Query Parameter
       │
       └───────── Path Parameter
```

Route가:

```jsx id="a58"
<Route path="/users/:userId" />
```

라면:

```text id="a59"
10
```

은 Path Parameter입니다.

`useParams()`로 읽습니다.

```jsx id="a60"
const { userId } = useParams();
```

반면:

```text id="a61"
?page=2
```

는 Query Parameter입니다.

`useSearchParams()`로 읽습니다.

```jsx id="a62"
const [searchParams] =
  useSearchParams();

searchParams.get('page');
```

정리하면:

```text id="a63"
:parameter
    │
    ▼
Path Parameter
    │
    ▼
useParams()


?key=value
    │
    ▼
Query Parameter
    │
    ▼
useSearchParams()
```

입니다.

---

# 14. `:` 자체가 값에 포함되는 것은 아니다

다음 Route에서:

```text id="a64"
/users/:userId
```

콜론 `:`은 **Route Pattern을 정의하기 위한 문법**입니다.

실제 URL에는 콜론이 들어가지 않습니다.

올바른 URL:

```text id="a65"
/users/10
```

Route Pattern:

```text id="a66"
/users/:userId
```

즉:

```text id="a67"
Route 정의
/users/:userId
        │
        └── ":"는 문법


실제 URL
/users/10
        │
        └── 실제 값
```

입니다.

이 구분이 중요합니다.

---

# 15. `:`와 `*`는 역할이 다르다

React Router의 Path Pattern에는 `*`도 사용할 수 있습니다.

예:

```jsx id="a68"
<Route path="/files/*" />
```

`:`와 `*`는 모두 동적인 경로를 처리하지만 의미는 다릅니다.

```text id="a69"
:userId
   │
   ▼
하나의 동적 Path Segment


*
│
▼
남은 여러 Path Segment를
포괄적으로 매칭
```

예를 들어:

```text id="a70"
/users/:userId
```

는:

```text id="a71"
/users/10
```

처럼 하나의 세그먼트를 Parameter로 받습니다.

반면:

```text id="a72"
/files/*
```

은:

```text id="a73"
/files/images/icons/logo.png
```

처럼 뒤쪽의 여러 세그먼트를 처리하는 데 사용할 수 있습니다.

---

# 16. 내부적으로 보면

Dynamic Segment는 React Router의 Route Matching 과정에서 처리됩니다.

다음 Route:

```jsx id="a74"
<Route path="/users/:userId" />
```

와 현재 URL:

```text id="a75"
/users/50
```

이 있다면:

```text id="a76"
Route Pattern
/users/:userId
        │
        ▼
React Router
Path Matching
        │
        ▼
현재 pathname
/users/50
        │
        ▼
:userId ↔ "50"
        │
        ▼
params
{
  userId: "50"
}
```

가 됩니다.

따라서 `:`는 단순한 문자열 표시가 아니라 **React Router의 Path Matching 엔진에게 “이 위치는 동적으로 매칭하라”고 알려주는 문법**입니다.

---

# 17. 가장 정확한 표현

다음과 같이 설명하면 조금 과도합니다.

> `/users/` 뒤에 어떤 값이 오든 모두 허용한다.

보다 정확한 설명은:

> **`/users/` 다음 Path Segment를 동적으로 매칭하고, 그 값을 `userId`라는 이름의 Route Parameter로 저장한다.**

입니다.

즉:

```text id="a77"
/users/:userId
        │
        ▼
해당 위치의
Path Segment를 매칭
        │
        ▼
userId라는 이름으로 저장
```

입니다.

---

# 18. 전체 구조

지금까지의 내용을 하나의 흐름으로 보면:

```text id="a78"
<Route
  path="/users/:userId"
/>
        │
        │ Route Pattern
        ▼
/users/:userId
        │
        │
현재 URL
/users/10
        │
        ▼
React Router
Route Matching
        │
        ▼
Dynamic Segment 매칭
        │
        ▼
:userId = "10"
        │
        ▼
params
{
  userId: "10"
}
        │
        ▼
useParams()
        │
        ▼
React Component
```

---

# 19. 핵심 정리

| 항목          | 설명                               |
| ----------- | -------------------------------- |
| `:`         | Dynamic Segment 선언 문법            |
| `:userId`   | `userId`라는 이름의 동적 Path Parameter |
| 실제 URL      | `/users/10`처럼 콜론 없이 실제 값 사용      |
| Parameter 값 | 문자열                              |
| 읽는 방법       | `useParams()`                    |
| 주요 목적       | 하나의 Route로 여러 URL 처리             |
| 대표 사례       | 사용자, 상품, 게시글 상세 페이지              |

가장 핵심만 압축하면:

```text id="a79"
Route Pattern

/users/:userId
        │
        ▼

실제 URL

/users/42
        │
        ▼

params

{
  userId: "42"
}
```

따라서 React Router에서 `:`의 본질은:

> **“이 Path Segment는 고정된 문자열이 아니라 동적으로 매칭하고, 그 값을 이름 있는 Parameter로 저장하라.”**

라는 의미입니다.
