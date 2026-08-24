# React Router를 이해하기 위한 브라우저의 Location과 History

React Router를 처음 배우면 다음 코드부터 접하는 경우가 많습니다.

```jsx
<BrowserRouter>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/products" element={<Products />} />
    <Route path="/products/:id" element={<ProductDetail />} />
  </Routes>
</BrowserRouter>
```

그리고 다음 API들을 배웁니다.

```jsx
<Link to="/products">상품</Link>
```

```javascript
const navigate = useNavigate();
navigate("/products");
```

```javascript
const location = useLocation();
```

하지만 이것부터 배우면 이런 의문이 생깁니다.

> React Router는 어떻게 주소창의 URL을 변경하는가?

> URL이 변경됐는데 왜 페이지 전체가 새로 로드되지 않는가?

> 브라우저의 뒤로가기 버튼을 눌렀는데 React는 어떻게 그것을 아는가?

> `window.location`이 있는데 왜 `useLocation()`이 또 필요한가?

> `<a href>`와 `<Link>`는 무엇이 다른가?

> `navigate()`는 내부적으로 무엇을 하는가?

이 질문들에 답하려면 React가 아니라 먼저 **웹 브라우저의 navigation 시스템**을 이해해야 합니다.

---

# 1. 브라우저에서 URL은 단순한 문자열이 아니다

주소창에 다음 URL이 있다고 생각해봅시다.

```text
https://shop.example.com:8443/products/100?category=book&page=2#review
```

사람에게는 하나의 문자열처럼 보이지만 브라우저는 이것을 여러 부분으로 나누어 해석합니다.

```text
https://shop.example.com:8443/products/100?category=book&page=2#review
└─┬─┘   └───────┬───────┘└┬─┘└─────┬──────┘└──────────┬─────────┘└─┬──┘
scheme      hostname     port  pathname             query        fragment
```

JavaScript의 `window.location`에서는 대략 다음과 대응됩니다.

| URL 부분   | `location` 속성 | 값                         |
| -------- | ------------- | ------------------------- |
| scheme   | `protocol`    | `"https:"`                |
| hostname | `hostname`    | `"shop.example.com"`      |
| port     | `port`        | `"8443"`                  |
| path     | `pathname`    | `"/products/100"`         |
| query    | `search`      | `"?category=book&page=2"` |
| fragment | `hash`        | `"#review"`               |

그리고 전체 URL은:

```javascript
location.href
```

로 얻을 수 있습니다.

---

# 2. `window.location`은 정확히 무엇인가?

브라우저에는 최상위 객체인 `window`가 있습니다.

```text
window
│
├── document
├── location
├── history
├── navigator
├── localStorage
├── sessionStorage
└── ...
```

여기서 `window.location`은 현재 `Window`와 연결된 **현재 문서의 URL 정보를 제공하고 탐색을 시작할 수 있게 해주는 객체**입니다.

쉽게 표현하면:

> **location = 현재 브라우저가 어디를 가리키고 있는가**

입니다.

예를 들어:

```javascript
console.log(window.location.href);
console.log(window.location.pathname);
console.log(window.location.search);
console.log(window.location.hash);
```

현재 주소가

```text
https://shop.example.com/products/100?page=2#review
```

라면:

```javascript
location.href
// "https://shop.example.com/products/100?page=2#review"

location.origin
// "https://shop.example.com"

location.pathname
// "/products/100"

location.search
// "?page=2"

location.hash
// "#review"
```

입니다.

---

# 3. React Router에서 가장 중요한 것은 `pathname`

React Router를 공부한다면 URL의 여러 부분 중 특히 이것을 주목해야 합니다.

```javascript
location.pathname
```

예를 들어:

```text
https://shop.example.com/products/100
                         └──────┬──────┘
                              pathname
```

```javascript
location.pathname
// "/products/100"
```

React Router에서:

```jsx
<Route
  path="/products/:id"
  element={<ProductDetail />}
/>
```

라고 작성했다면 Router는 현재 URL 정보를 기반으로 해당 Route가 일치하는지 판단합니다.

개념적으로:

```text
Browser URL
      │
      ▼
https://shop.example.com/products/100
      │
      ▼
location
      │
      ▼
pathname
      │
      ▼
"/products/100"
      │
      ▼
React Router
      │
      ▼
Route Matching
      │
      ▼
"/products/:id"
      │
      ▼
<ProductDetail />
```

따라서 React Router는 **URL을 만드는 기술**이라기보다는 URL과 React UI를 연결하는 시스템이라고 보는 편이 정확합니다.

---

# 4. 그런데 `location`은 URL을 읽기만 하는 객체가 아니다

`location`을 이용하면 브라우저를 다른 URL로 이동시킬 수도 있습니다.

대표적으로:

```javascript
location.href = "/products";
```

또는:

```javascript
location.assign("/products");
```

가 있습니다.

이때 중요한 것은 단순히 "문자열이 변경된다"가 아니라 **navigation이 시작된다**는 것입니다.

현재:

```text
http://localhost:5173/
```

에서

```javascript
location.href = "/products";
```

를 실행했다고 해봅시다.

일반적인 흐름은:

```text
현재 Document

       │

location.href = "/products"

       ▼

Browser Navigation 시작

       ▼

GET /products

       ▼

Server

       ▼

HTML Response

       ▼

새 Document 로드

       ▼

CSS / JavaScript 등 로드

       ▼

React JavaScript 실행

       ▼

React Application 시작
```

입니다.

즉, `location.href`를 통한 일반적인 문서 탐색은 **React 컴포넌트만 교체하는 것이 아닙니다.**

브라우저 수준의 navigation입니다.

---

# 5. 여기서 반드시 `Document`를 생각해야 한다

React Router를 이해하는 데 굉장히 중요한 구분입니다.

브라우저는 현재 페이지의 DOM을:

```javascript
window.document
```

로 제공합니다.

전통적인 페이지 이동에서는:

```text
Document A
   │
   │ navigation
   ▼
Document B
```

처럼 **현재 Document 자체가 교체될 수 있습니다.**

예를 들어:

```text
/products
```

에서

```text
/cart
```

로 일반적인 문서 탐색을 한다면:

```text
/products

Document A
DOM Tree A
JS Context A

       │
       │ navigation
       ▼

/cart

Document B
DOM Tree B
JS Context B
```

라는 관점으로 이해할 수 있습니다.

SPA가 해결하려는 핵심 문제 중 하나가 바로 이것입니다.

---

# 6. SPA는 Document를 유지하고 싶다

React SPA에서는 보통 애플리케이션을 처음 한 번 로드합니다.

```text
index.html
    │
    ▼
JavaScript
    │
    ▼
React
    │
    ▼
<App />
```

그 이후:

```text
/home
/products
/products/100
/cart
```

등을 이동할 때마다 새로운 HTML Document를 서버에서 받아오는 것이 아니라,

**현재 Document와 실행 중인 React 애플리케이션을 유지하면서 화면만 바꾸고 싶습니다.**

즉:

```text
전통적인 navigation

/products
Document A

     ↓

/cart
Document B
```

가 아니라:

```text
SPA

하나의 Document
──────────────────────────────

URL: /
UI : <Home />

          ↓

URL: /products
UI : <Products />

          ↓

URL: /products/100
UI : <ProductDetail />

          ↓

URL: /cart
UI : <Cart />

──────────────────────────────

Document는 계속 유지
```

를 원하는 것입니다.

그런데 문제가 있습니다.

주소창도 같이 바뀌어야 합니다.

---

# 7. SPA라고 URL을 포기할 수는 없다

초기 SPA에서 생각할 수 있는 가장 단순한 방식은:

```text
http://example.com/
```

URL은 그대로 두고 React state만 변경하는 것입니다.

```javascript
setPage("products");
```

그러면 화면은:

```text
Home
 ↓
Products
 ↓
Cart
```

로 변경할 수 있습니다.

하지만 URL은 계속:

```text
http://example.com/
```

입니다.

이러면 웹의 중요한 기능들을 잃습니다.

예를 들어 사용자가 상품 100번 페이지를 보고 있는데:

```text
http://example.com/
```

밖에 없다면 해당 화면을 북마크하거나 URL을 복사해서 다른 사람에게 공유하기 어렵습니다.

우리가 원하는 것은:

```text
Home
/products

Products
/products

Product 100
/products/100

Cart
/cart
```

처럼 **UI 상태와 URL이 동기화되는 것**입니다.

---

# 8. 그런데 `location.href`를 사용하면 다시 Document navigation이다

그러면 이런 생각을 할 수 있습니다.

```javascript
location.href = "/products";
```

URL도 바뀌니까 해결된 것 아닌가?

아닙니다.

일반적으로:

```text
location.href
     ↓
Navigation
     ↓
Server Request
     ↓
Document 교체
```

가 발생합니다.

우리가 원하는 것은:

```text
URL 변경
     ↓
Document 유지
     ↓
React UI만 변경
```

입니다.

이 요구를 가능하게 해주는 중요한 브라우저 기능이 바로 **History API**입니다.

---

# 9. `window.history`란?

`window.history`는 현재 탭의 **세션 히스토리(session history)** 와 상호작용할 수 있도록 제공되는 객체입니다.

브라우저를 사용하면서:

```text
Google
 ↓
YouTube
 ↓
GitHub
 ↓
ChatGPT
```

처럼 페이지를 이동하면 브라우저는 탐색 기록을 관리합니다.

개념적으로:

```text
Session History

┌────────────────────┐
│ google.com         │
├────────────────────┤
│ youtube.com        │
├────────────────────┤
│ github.com         │
├────────────────────┤
│ chatgpt.com        │ ← 현재 Entry
└────────────────────┘
```

여기서 중요한 단어가 **History Entry**입니다.

---

# 10. History는 단순한 URL 배열이라고 생각하면 부족하다

입문 단계에서는:

```javascript
[
  "/",
  "/products",
  "/products/100",
  "/cart"
]
```

처럼 생각해도 됩니다.

하지만 실제 개념은 단순한 URL 문자열 배열보다 훨씬 복잡합니다.

각 항목은 **session history entry**입니다.

개념적으로:

```text
Session History

Entry
├── URL
├── state
├── 관련 탐색 정보
└── 기타 브라우저 관리 정보
```

따라서:

> **History는 URL 문자열 목록이라기보다 브라우저의 탐색 상태 기록입니다.**

라고 이해하는 것이 좋습니다.

---

# 11. History에는 "현재 위치"가 있다

다음처럼 방문했다고 합시다.

```text
/
 ↓
/products
 ↓
/products/100
 ↓
/cart
```

개념적으로:

```text
Session History

[ / ]
[ /products ]
[ /products/100 ]
[ /cart ]            ← 현재
```

뒤로가기를 누르면:

```text
[ / ]
[ /products ]
[ /products/100 ]    ← 현재
[ /cart ]
```

다시 뒤로가면:

```text
[ / ]
[ /products ]        ← 현재
[ /products/100 ]
[ /cart ]
```

앞으로가기를 누르면:

```text
[ / ]
[ /products ]
[ /products/100 ]    ← 현재
[ /cart ]
```

따라서 History는 단순히 기록을 저장하는 기능만 있는 것이 아니라 **그 기록 사이를 이동하는 navigation 기능**도 가지고 있습니다.

---

# 12. `back()`, `forward()`, `go()`

그래서 History API에는 다음이 있습니다.

```javascript
history.back();
```

뒤로 이동합니다.

```javascript
history.forward();
```

앞으로 이동합니다.

```javascript
history.go(-2);
```

두 단계 뒤로 이동합니다.

예를 들어:

```text
             -3         -2             -1          현재

            Home     Products     ProductDetail     Cart
              │          │              │            │
              ▼          ▼              ▼            ▼
             "/"    "/products"  "/products/100"   "/cart"
```

현재 `/cart`에서:

```javascript
history.go(-2);
```

하면 `/products` 방향으로 이동합니다.

React Router의:

```javascript
navigate(-1);
navigate(-2);
navigate(1);
```

가 갑자기 등장한 문법이 아니라 **브라우저의 History navigation 모델을 React Router API로 제공하는 것**임을 알 수 있습니다.

---

# 13. SPA의 핵심 기능 `pushState()`

이제 가장 중요한 API입니다.

```javascript
history.pushState(state, "", url);
```

예:

```javascript
history.pushState(null, "", "/products");
```

이 메서드가 특별한 이유는 **현재 Document를 새로 로드하지 않고 현재 세션 히스토리에 새로운 Entry를 추가하면서 URL을 변경할 수 있기 때문**입니다.

현재:

```text
http://localhost:5173/
```

에서:

```javascript
history.pushState(null, "", "/products");
```

를 실행하면 주소창은:

```text
http://localhost:5173/products
```

로 변경될 수 있습니다.

하지만 일반적인 새 문서 navigation을 일으키지 않습니다.

---

# 14. `location.href`와 `pushState()`의 차이

이 차이는 반드시 정확하게 잡아야 합니다.

### `location.href`

```javascript
location.href = "/products";
```

개념적으로:

```text
URL 변경 요청
     │
     ▼
Browser Navigation
     │
     ▼
Server Request
     │
     ▼
새 Document
     │
     ▼
Application 재시작 가능
```

### `history.pushState()`

```javascript
history.pushState(null, "", "/products");
```

```text
현재 Document
     │
     │ 그대로 유지
     │
     ├───────────────┐
     │               │
     ▼               ▼
History Entry      URL 변경
추가              /products
     │
     └───────┬───────┘
             ▼
        Document 유지
```

SPA의 클라이언트 라우팅이 가능한 핵심 기반입니다.

---

# 15. `pushState()`의 `state`는 무엇인가?

형식은:

```javascript
history.pushState(state, unused, url);
```

입니다.

예:

```javascript
history.pushState(
  { productId: 100 },
  "",
  "/products/100"
);
```

개념적으로 새로운 History Entry가:

```text
History Entry
│
├── URL
│    └── /products/100
│
└── state
     └── {
           productId: 100
         }
```

처럼 만들어집니다.

현재 Entry에 연결된 state는:

```javascript
history.state
```

로 접근할 수 있습니다.

```javascript
console.log(history.state);
```

```javascript
{
  productId: 100
}
```

React Router에서:

```javascript
navigate("/products/100", {
  state: {
    from: "/products"
  }
});
```

같은 기능을 이해할 때 이 배경지식이 도움이 됩니다.

---

# 16. `replaceState()`는 왜 필요한가?

`pushState()`는 새로운 Entry를 **추가**합니다.

```text
현재

Home
 ↓
Products


pushState("/cart")


Home
 ↓
Products
 ↓
Cart
```

반면:

```javascript
history.replaceState(null, "", "/cart");
```

는 현재 Entry를 **교체**합니다.

```text
Before

Home
 ↓
Products ← 현재


replaceState("/cart")


After

Home
 ↓
Cart ← 현재
```

이 차이는 로그인 처리 같은 곳에서 중요합니다.

예를 들어:

```text
/login
```

에서 로그인 성공 후:

```text
/dashboard
```

로 이동했는데 사용자가 뒤로가기를 눌렀을 때 다시 `/login`으로 돌아갈 필요가 없다면 replace 방식이 적절할 수 있습니다.

React Router에서는:

```javascript
navigate("/dashboard", {
  replace: true
});
```

같은 형태로 사용합니다.

---

# 17. 그런데 `pushState()`만으로 Router를 만들 수 있는 것은 아니다

여기서 굉장히 중요한 사실이 있습니다.

```javascript
history.pushState(null, "", "/products");
```

를 실행하면:

```text
URL
/ → /products
```

는 변경됩니다.

하지만 브라우저가 알아서:

```jsx
<Products />
```

를 렌더링해 주지는 않습니다.

왜냐하면 브라우저는 React를 모르기 때문입니다.

브라우저가 아는 것은:

```text
URL
History
Document
DOM
Navigation
```

정도입니다.

브라우저는:

```text
"/products"
        ↓
<Products />
```

라는 관계를 모릅니다.

이 관계를 만드는 것이 **Router**입니다.

---

# 18. 그래서 Router의 본질은 URL → UI 매핑이다

React Router에서:

```jsx
<Routes>
  <Route path="/" element={<Home />} />

  <Route
    path="/products"
    element={<Products />}
  />

  <Route
    path="/products/:id"
    element={<ProductDetail />}
  />
</Routes>
```

라고 작성하면 우리가 선언하는 것은 본질적으로:

```text
URL Pattern              UI

/                   →    <Home />

/products           →    <Products />

/products/:id       →    <ProductDetail />
```

라는 관계입니다.

따라서 Router의 핵심 역할 중 하나는:

```text
현재 Location
      ↓
URL 분석
      ↓
Route Matching
      ↓
일치하는 Route 결정
      ↓
React Element 결정
      ↓
React Rendering
```

입니다.

---

# 19. `popstate`가 필요한 이유

이제 사용자가 브라우저의 뒤로가기 버튼을 눌렀다고 합시다.

현재:

```text
Home
 ↓
Products
 ↓
ProductDetail ← 현재
```

뒤로가기:

```text
Home
 ↓
Products ← 현재
 ↓
ProductDetail
```

브라우저 입장에서는 History Entry가 변경되었습니다.

그런데 React Router도 이 사실을 알아야 합니다.

그렇지 않으면:

```text
주소창
/products

화면
<ProductDetail />
```

처럼 URL과 UI가 불일치할 수 있습니다.

브라우저는 History traversal과 관련된 변화를 알 수 있도록 `popstate` 이벤트를 제공합니다.

```javascript
window.addEventListener("popstate", event => {
  console.log(location.pathname);
});
```

개념적으로:

```text
사용자 Back 버튼
       │
       ▼
History Entry 이동
       │
       ▼
현재 Location 변경
       │
       ▼
popstate
       │
       ▼
Router가 변화 감지
       │
       ▼
새 Location 반영
       │
       ▼
Route Matching
       │
       ▼
UI 변경
```

---

# 20. 여기서 아주 중요한 함정

많이 헷갈리는 부분입니다.

```javascript
history.pushState(...)
```

자체는 `popstate` 이벤트를 발생시키지 않습니다.

즉:

```javascript
history.pushState(null, "", "/products");
```

했다고 해서:

```javascript
window.addEventListener("popstate", ...)
```

가 바로 호출되는 것은 아닙니다.

`popstate`는 주로 사용자가:

```text
Back
Forward
history.go(...)
```

등으로 기존 History Entry 사이를 이동할 때 중요합니다.

따라서 실제 Router는 단순히:

```javascript
window.addEventListener("popstate", ...)
```

하나만 작성한다고 완성되지 않습니다.

Router 자신이 시작한 navigation과 브라우저 History traversal 양쪽을 모두 관리해야 합니다.

---

# 21. 이제 `<a>`와 `<Link>`의 차이가 보인다

HTML:

```html
<a href="/products">Products</a>
```

일반적으로 브라우저의 문서 navigation을 수행합니다.

```text
<a href>

Click
  ↓
Browser Navigation
  ↓
Request
  ↓
Document Load
```

React Router:

```jsx
<Link to="/products">
  Products
</Link>
```

일반적인 내부 라우팅 상황에서는 React Router가 클릭을 처리하여 클라이언트 측 navigation을 수행합니다.

개념적으로:

```text
<Link>

Click
  ↓
React Router
  ↓
브라우저 기본 문서 navigation 대신
Client-side Navigation
  ↓
History 업데이트
  ↓
Location 업데이트
  ↓
Route Matching
  ↓
React Rendering
```

그래서 URL은 변경되지만 **현재 SPA의 Document와 JavaScript 실행 환경을 유지할 수 있습니다.**

---

# 22. 왜 결국 `<a>`를 기반으로 하는가?

여기서 한 단계 더 깊게 볼 필요가 있습니다.

`Link`는 단순한:

```jsx
<span onClick={...}>
```

같은 것이 아닙니다.

링크는 웹에서 중요한 의미를 가지고 있습니다.

예를 들어:

* 새 탭에서 열기
* 링크 주소 복사
* 접근성
* 키보드 탐색
* 브라우저의 링크 처리

등입니다.

따라서 React Router의 `Link`는 **웹의 링크 의미를 유지하면서 클라이언트 라우팅을 추가하는 것**이라고 이해하는 편이 좋습니다.

---

# 23. `useLocation()`은 왜 필요한가?

이제 처음의 질문으로 돌아옵니다.

브라우저에 이미:

```javascript
window.location
```

이 있는데 왜:

```javascript
useLocation()
```

이 필요할까요?

핵심은 **React의 렌더링 시스템과 연결되어 있느냐**입니다.

다음 코드를 작성한다고 생각해봅시다.

```jsx
function Header() {
  const pathname = window.location.pathname;

  return <h1>{pathname}</h1>;
}
```

`window.location`은 React state가 아닙니다.

React는:

```javascript
window.location.pathname
```

이 바뀌었다는 이유만으로 컴포넌트를 자동으로 리렌더링하지 않습니다.

React 입장에서 브라우저 객체의 프로퍼티가 바뀐 것일 뿐입니다.

반면:

```javascript
const location = useLocation();
```

은 Router가 관리하는 현재 location 정보를 **React 컴포넌트와 연결**합니다.

개념적으로:

```text
Browser
   │
   │ URL / History
   ▼
React Router
   │
   │ 현재 Location 관리
   ▼
Router State
   │
   ├───────────────┐
   │               │
   ▼               ▼
<Routes>       useLocation()
   │
   ▼
Route Matching
```

따라서 navigation이 발생하면 새로운 location을 바탕으로 관련 React 컴포넌트가 다시 렌더링될 수 있습니다.

---

# 24. `useNavigate()`도 이제 이해할 수 있다

다음 코드:

```javascript
const navigate = useNavigate();

navigate("/products");
```

를 단순히:

> "페이지 이동 함수"

라고 가르치면 내부 원리가 보이지 않습니다.

더 정확한 사고 방식은:

> **React Router에게 새로운 location으로 client-side navigation을 수행하도록 요청한다.**

입니다.

개념적으로:

```text
navigate("/products")

          │
          ▼

      React Router

          │
          ▼

Browser History와 Router 상태 업데이트

          │
          ▼

location 변경

          │
          ▼

Route Matching

          │
          ▼

<Products />

          │
          ▼

React Render
```

입니다.

---

# 25. React Router가 History API를 단순히 감싼 것만은 아니다

여기서 한 가지 주의할 점도 있습니다.

React Router를:

> "`history.pushState()`를 쉽게 쓰게 해주는 라이브러리"

정도로 정의하면 너무 좁습니다.

React Router는 History/navigation 위에서 훨씬 많은 기능을 제공합니다.

예를 들면:

```text
URL
 │
 ▼
Route Matching
 │
 ├── Dynamic Segment
 │     /products/:id
 │
 ├── Nested Routes
 │
 ├── Search Params
 │
 ├── Navigation
 │
 ├── Location
 │
 └── Data Router 기능
```

등입니다.

즉 History API는 **기반 인프라**이고 React Router는 그 위에 **React 애플리케이션을 위한 라우팅 모델**을 제공합니다.

---

# 26. 새로고침하면 왜 문제가 생길 수 있는가?

React Router를 이해했다면 반드시 이것도 알아야 합니다.

React 앱에서:

```text
http://localhost:5173/products/100
```

으로 client-side navigation했다고 합시다.

현재는:

```text
Document 유지
       +
React Router
       +
<ProductDetail />
```

이므로 정상적으로 보입니다.

그런데 여기서 사용자가 **새로고침**합니다.

그러면 이야기가 완전히 달라집니다.

브라우저가 서버에:

```http
GET /products/100
```

을 요청합니다.

즉:

```text
Client-side navigation

/products/100
      ↓
React Router가 처리
```

였던 것이 새로고침 순간:

```text
/products/100
      ↓
Browser Navigation
      ↓
GET /products/100
      ↓
Server
```

가 됩니다.

서버가 `/products/100`이라는 실제 파일이나 서버 Route를 찾으려고 한다면:

```text
404 Not Found
```

가 발생할 수도 있습니다.

그래서 SPA 서버 설정에서는 흔히:

```text
/products/100
/cart
/users/10
/settings

        ↓

모두 index.html 제공

        ↓

React Application 시작

        ↓

React Router가 현재 URL 확인

        ↓

올바른 화면 렌더링
```

같은 fallback 설정이 필요합니다.

이것은 `BrowserRouter`를 이해할 때 상당히 중요한 부분입니다.

---

# 27. 그래서 `HashRouter`라는 방식도 이해할 수 있다

URL의 `#` 뒤쪽인 fragment는 전통적으로 서버에 같은 방식으로 전달되는 path가 아닙니다.

예:

```text
https://example.com/#/products/100
```

서버 관점에서는 기본 문서를 요청하고:

```text
#/products/100
```

부분을 클라이언트에서 처리할 수 있습니다.

그래서 과거에는 서버 설정이 어려운 환경에서 hash 기반 routing이 많이 사용됐습니다.

React Router의:

```jsx
<HashRouter>
```

도 이 원리를 활용합니다.

반면:

```jsx
<BrowserRouter>
```

는:

```text
/products/100
```

처럼 자연스러운 URL을 사용할 수 있지만 서버도 SPA routing을 고려해 설정하는 것이 일반적입니다.

---

# 28. Location과 History의 관계를 정확하게 정리하면

둘은 비슷해 보이지만 역할이 다릅니다.

### Location

질문:

> **지금 어디에 있는가?**

```text
location

현재 URL
│
├── protocol
├── host
├── hostname
├── port
├── pathname
├── search
└── hash
```

### History

질문:

> **어떤 navigation 기록을 가지고 있고 그 기록을 어떻게 이동하거나 변경할 것인가?**

```text
history

Session History
│
├── back()
├── forward()
├── go()
├── pushState()
├── replaceState()
└── state
```

그리고 두 개가 연결됩니다.

```text
                  Browser
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼

      location                history

   현재 위치 표현          탐색 기록 관리

          │                     │
          └──────────┬──────────┘
                     │
                     ▼
                 Navigation
```

---

# 29. 이것이 React Router에서 어떻게 대응되는가

개념적으로 다음과 같이 연결해서 기억하면 좋습니다.

| Browser                | React Router                               |
| ---------------------- | ------------------------------------------ |
| 현재 URL / Location      | `useLocation()`                            |
| History navigation     | `useNavigate()`                            |
| `history.go(-1)`       | `navigate(-1)`                             |
| 새로운 navigation         | `navigate("/products")`                    |
| History Entry 교체       | `navigate("/products", { replace: true })` |
| 링크 navigation          | `<Link to="/products">`                    |
| URL pattern과 UI 연결     | `<Route>`                                  |
| Route 집합/매칭            | `<Routes>`                                 |
| 브라우저 History 기반 Router | `<BrowserRouter>`                          |

다만 이것은 **개념적 대응표**입니다. React Router의 각 API가 브라우저 API 하나를 그대로 1:1 포장했다는 의미는 아닙니다.

---

# 30. 전체 구조를 한 번에 연결해보자

이제 React Router를 다음 구조로 보면 됩니다.

```text
┌───────────────────────────────────────────┐
│                 Browser                   │
│                                           │
│   ┌──────────────┐   ┌────────────────┐   │
│   │   Location   │   │    History     │   │
│   │              │   │                │   │
│   │ pathname     │   │ pushState      │   │
│   │ search       │   │ replaceState   │   │
│   │ hash         │   │ back           │   │
│   │ href         │   │ forward        │   │
│   └──────┬───────┘   └───────┬────────┘   │
│          │                   │             │
└──────────┼───────────────────┼─────────────┘
           │                   │
           └─────────┬─────────┘
                     ▼
            ┌─────────────────┐
            │  BrowserRouter  │
            │                 │
            │ URL / History와 │
            │ React 연결      │
            └────────┬────────┘
                     │
         ┌───────────┴────────────┐
         │                        │
         ▼                        ▼
   Navigation                Route Matching
         │                        │
   ┌─────┴─────┐                  ▼
   │           │              <Routes>
 <Link>    useNavigate            │
                                  ▼
                               <Route>
                                  │
                                  ▼
                         React Component
```

---

# 31. 실제 클릭 한 번을 끝까지 추적해보자

마지막으로 이것을 이해하면 React Router의 기본 원리는 거의 잡힌 것입니다.

현재 화면:

```text
URL
/

UI
<Home />
```

사용자가:

```jsx
<Link to="/products/100">
  상품 보기
</Link>
```

를 클릭합니다.

전체 흐름은 개념적으로:

```text
① 사용자 Click

        ↓

② <Link>

        ↓

③ React Router가
   client-side navigation 수행

        ↓

④ Browser History 업데이트

        ↓

⑤ 새로운 URL

   /products/100

        ↓

⑥ Router가 새로운 Location 반영

        ↓

⑦ Route Matching

   /products/:id

        ↓

⑧ params

   id = "100"

        ↓

⑨ 선택되는 Element

   <ProductDetail />

        ↓

⑩ React Re-render

        ↓

⑪ DOM에 필요한 변경 Commit

        ↓

⑫ Browser Paint

        ↓

⑬ 화면

   Product Detail
```

이 과정에서 가장 중요한 것은:

```text
새 HTML Document를 서버에서 받아서
페이지 전체를 교체한 것이 아니다.
```

라는 점입니다.

현재 React 애플리케이션이 계속 실행되고 있고,

```text
URL 변화
    ↓
Router
    ↓
Route Matching
    ↓
React Element 변화
    ↓
Re-render
```

가 발생한 것입니다.

---

# React Router를 이해하는 핵심 사고 모델

결국 세 계층으로 나누면 가장 깔끔합니다.

```text
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Browser Layer
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Location
History
Navigation
popstate
URL

             ↓


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2. React Router Layer
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BrowserRouter
Routes
Route
Link
navigate
location
params

             ↓


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
3. React Layer
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

React Element
Component
Re-render
Reconciliation
Commit

             ↓

            DOM
             ↓
           Screen
```


