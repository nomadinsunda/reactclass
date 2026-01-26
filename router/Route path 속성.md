`<Route path="/users/:userId">`에서 `:`(콜론)은 **URL에서 “동적 파라미터(dynamic parameter)”를 선언하는 문법**입니다.
즉, `:userId`는 **변하는 값(동적 값)**을 의미합니다.


---

# 🔥 React Router에서 `:`(콜론)은 “동적 세그먼트(Parameter)” 선언 문법이다!

React Router는 URL을 **정적인 문자열 그대로 일치**시키는 것뿐만 아니라,
URL 일부를 **변수처럼 표현할 수 있는 기능**을 제공합니다.

그때 사용하는 문법이 바로 **`:`(콜론)**입니다.

예시:

```jsx
<Route path="/users/:userId" element={<UserDetail />} />
```

여기서 의미는 다음과 같습니다:

> 📌 `/users/뒤에 어떤 값이 오든 허용하고, 그 값을 userId라는 이름으로 읽어오겠다.`

---

# 🧩 예시로 완전히 이해해보자!

## 예) URL이 다음과 같다면…

* `/users/1`
* `/users/42`
* `/users/abc123`

모두 아래 라우트에 **매칭됩니다**:

```jsx
<Route path="/users/:userId" element={<UserDetail />} />
```

그리고 `:userId`에 실제 URL 값이 들어갑니다:

| 실제 URL          | 매칭 값                |
| --------------- | ------------------- |
| `/users/1`      | userId = `"1"`      |
| `/users/42`     | userId = `"42"`     |
| `/users/abc123` | userId = `"abc123"` |

---

# 🎣 useParams()로 동적 파라미터 읽기

React Router에서는 동적 파라미터 값을 **`useParams()` 훅**으로 읽습니다.

```jsx
import { useParams } from 'react-router-dom'

export default function UserDetail() {
  const { userId } = useParams()

  return (
    <div>
      <h1>User ID: {userId}</h1>
    </div>
  )
}
```

URL이 `/users/50`이면 화면에:

```
User ID: 50
```

이 출력됩니다.

---

# 🗺️ 라우팅 규칙 정리

React Router는 path를 다음처럼 인식합니다:

```
/users/:userId
```

* `/users/` → 문자열(정적)
* `:userId` → 매칭 가능한 모든 문자열(동적)
* userId라는 이름으로 값을 저장

### ✔️ 한 라우트에 여러 파라미터도 가능

```jsx
<Route path="/products/:category/:productId" />
```

URL 예시:

* `/products/electronics/100`
* `/products/books/19`

받는 값:

```json
{
  "category": "electronics",
  "productId": "100"
}
```

---

# 💡 왜 동적 파라미터가 필요할까?

동적 파라미터는 주로 다음을 위해 사용됩니다:

| 목적        | 설명                                      |
| --------- | --------------------------------------- |
| 🔍 상세 페이지 | `/users/1`, `/users/2` 같은 “id 기반 상세페이지” |
| 📁 카테고리   | `/products/food`, `/products/digital`   |
| 🔄 필터링/검색 | `/search/react`, `/search/vue`          |
| 📄 페이징    | `/articles/page/3`                      |

즉, **하나의 라우트로 여러 URL을 처리**할 수 있게 됩니다.

---

# 🧠 정리

`<Route path="/users/:userId">`에서 `:`는 다음 의미입니다:

> **“여기에 어떤 값이 들어와도 매칭하겠다. 그리고 그 값을 userId라는 이름으로 읽을 수 있게 하겠다.”**

React Router에서 `:`로 시작하는 부분은
**Dynamic Segment(동적 세그먼트)**, **Route Parameter(라우트 파라미터)** 라고 부릅니다.

