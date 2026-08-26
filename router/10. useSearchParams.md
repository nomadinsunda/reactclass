# React Router v6의 `useSearchParams()`

`useSearchParams()`는 React Router에서 현재 URL의 Query Parameter를 읽고 변경할 때 사용하는 Hook입니다.

예를 들어 다음 URL을 보겠습니다.

```text id="1r5roh"
/products?keyword=react&page=2&sort=latest
```

이 URL에서:

```text id="puaf82"
?keyword=react&page=2&sort=latest
```

부분이 Query String이고, 그 안의:

```text id="lctxof"
keyword=react
page=2
sort=latest
```

가 각각 Query Parameter입니다.

`useSearchParams()`를 사용하면 이런 값들을 컴포넌트에서 읽을 수 있고, 변경된 Query Parameter를 URL에 다시 반영할 수도 있습니다.

---

# 1. `useSearchParams()`의 핵심 역할

`useSearchParams()`는 두 값을 반환합니다.

```jsx id="d98k76"
const [searchParams, setSearchParams] =
  useSearchParams();
```

첫 번째 값인 `searchParams`는 현재 Query Parameter를 읽기 위한 `URLSearchParams` 객체입니다.

두 번째 값인 `setSearchParams`는 Query Parameter를 변경하고 새로운 URL 상태로 네비게이션하기 위한 함수입니다.

```text id="wyo3ea"
useSearchParams()
       │
       ├── searchParams
       │      │
       │      └── 현재 Query Parameter 읽기
       │
       └── setSearchParams
              │
              └── Query Parameter 변경
```

한 문장으로 정리하면 다음과 같습니다.

> `useSearchParams()`는 현재 Location의 `search` 값을 `URLSearchParams` 형태로 다룰 수 있게 하고, Query Parameter 변경을 React Router 네비게이션으로 연결해주는 Hook입니다.

---

# 2. URL에서 Query Parameter는 어디에 있는가?

URL은 다음처럼 여러 영역으로 나눌 수 있습니다.

```text id="192pwu"
/products/10?page=2&sort=latest#review
│           │                   │
│           │                   └── hash
│           │
│           └────────────────────── search
│
└────────────────────────────────── pathname
```

여기서:

```text id="sfcuf1"
?page=2&sort=latest
```

가 `search` 영역입니다.

그 안에는:

```text id="762vju"
page = 2
sort = latest
```

라는 Query Parameter들이 들어 있습니다.

`useSearchParams()`는 바로 이 `search` 영역을 다루는 Hook입니다.

---

# 3. Query Parameter는 언제 사용하는가?

Query Parameter는 대체로 같은 페이지 안에서 화면 상태를 표현할 때 사용합니다.

예를 들어 상품 목록 페이지가 있다고 하겠습니다.

```text id="ujuk5c"
/products
```

검색어를 추가하면:

```text id="7b9d7u"
/products?keyword=keyboard
```

정렬 조건까지 추가하면:

```text id="rt2163"
/products?keyword=keyboard&sort=price
```

페이지 번호까지 추가하면:

```text id="7hng9j"
/products?keyword=keyboard&sort=price&page=3
```

가 됩니다.

Path는 계속 `/products`이지만, 현재 어떤 상품 목록을 보여줄지는 Query Parameter가 표현합니다.

| 용도     | 예                 |
| ------ | ----------------- |
| 검색어    | `?keyword=react`  |
| 페이지 번호 | `?page=3`         |
| 정렬     | `?sort=latest`    |
| 필터     | `?category=shoes` |
| 표시 방식  | `?view=grid`      |

---

# 4. Query Parameter를 URL 상태로 사용하는 이유

컴포넌트 내부 상태로 페이지 번호를 관리하면 다음과 같이 작성할 수 있습니다.

```jsx id="j78aim"
const [page, setPage] = useState(1);
```

하지만 이 값은 주소창에 나타나지 않습니다.

반면 Query Parameter를 사용하면:

```text id="8tiagl"
/products?page=3
```

처럼 현재 상태가 URL 자체에 남습니다.

그 결과 다음과 같은 장점이 생깁니다.

```text id="cp1mqj"
URL에 상태 표현
      │
      ├── 새로고침 후에도 같은 조건 복원 가능
      ├── Bookmark 가능
      ├── URL 공유 가능
      ├── 뒤로 가기 / 앞으로 가기 활용 가능
      └── 검색 결과 링크 공유 가능
```

그래서 검색, 정렬, 필터, Pagination처럼 공유하거나 다시 방문했을 때 복원할 가치가 있는 상태에 잘 어울립니다.

---

# 5. 기본 사용법

```jsx id="lhypa7"
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] =
    useSearchParams();

  return <div>Products</div>;
}
```

여기서:

```jsx id="ppwrao"
searchParams
```

는 현재 Query Parameter를 읽기 위한 객체이고,

```jsx id="is0avm"
setSearchParams
```

는 Query Parameter를 변경하기 위한 함수입니다.

---

# 6. Query Parameter 읽기

현재 URL이 다음과 같다고 하겠습니다.

```text id="o8vxbj"
/products?page=2&keyword=react
```

그러면:

```jsx id="ly12ss"
const [searchParams] = useSearchParams();

const page = searchParams.get('page');
const keyword = searchParams.get('keyword');
```

결과는:

```text id="2gksqg"
page     = "2"
keyword  = "react"
```

입니다.

여기서 중요한 점은 Query Parameter 값도 문자열이라는 것입니다.

```js id="iou9yh"
typeof page
```

결과:

```text id="21aal8"
string
```

숫자로 사용하려면 직접 변환해야 합니다.

```jsx id="u9v3jg"
const page =
  Number(searchParams.get('page') ?? 1);
```

---

# 7. `get()`으로 값 하나 읽기

가장 자주 사용하는 메서드는 `get()`입니다.

```jsx id="76ngw4"
searchParams.get('page');
```

현재 URL이:

```text id="olmjsy"
/products?page=3
```

이라면 결과는:

```text id="zpcuuv"
"3"
```

입니다.

해당 Parameter가 없으면:

```js id="0a130b"
null
```

을 반환합니다.

그래서 다음과 같은 패턴을 많이 사용합니다.

```jsx id="j8fhza"
const page =
  Number(searchParams.get('page') ?? 1);
```

또는:

```jsx id="mosw5n"
const keyword =
  searchParams.get('keyword') ?? '';
```

---

# 8. 같은 key가 여러 번 등장하는 경우

Query String에는 같은 key가 반복될 수 있습니다.

```text id="t1resz"
/products?category=shoes&category=coat
```

이때:

```jsx id="huas2u"
searchParams.get('category');
```

을 호출하면 첫 번째 값인:

```text id="7lvy2o"
"shoes"
```

만 반환됩니다.

모든 값을 읽고 싶다면:

```jsx id="9mw36a"
searchParams.getAll('category');
```

을 사용합니다.

결과:

```js id="pt7lop"
[
  'shoes',
  'coat'
]
```

---

# 9. `searchParams`에서 자주 쓰는 메서드

`searchParams`는 표준 `URLSearchParams` 객체입니다.

| 메서드                  | 역할                  |
| -------------------- | ------------------- |
| `get(key)`           | 첫 번째 값 반환           |
| `getAll(key)`        | 같은 key의 모든 값 반환     |
| `has(key)`           | Parameter 존재 여부 확인  |
| `set(key, value)`    | 해당 key의 값 설정        |
| `append(key, value)` | 같은 key에 값 추가        |
| `delete(key)`        | 해당 Parameter 삭제     |
| `toString()`         | Query String 문자열 생성 |

예:

```jsx id="ycgw7s"
searchParams.get('page');

searchParams.getAll('category');

searchParams.has('keyword');
```

---

# 10. Query Parameter 변경

Query Parameter를 바꾸려면 `setSearchParams()`를 사용합니다.

```jsx id="az5xm7"
const [searchParams, setSearchParams] =
  useSearchParams();
```

예를 들어:

```jsx id="pldpmr"
setSearchParams({
  page: '3',
  sort: 'latest'
});
```

를 실행하면 결과 URL은:

```text id="2rr62g"
?page=3&sort=latest
```

가 됩니다.

흐름을 보면:

```text id="71522j"
setSearchParams(...)
        │
        ▼
React Router Navigation
        │
        ▼
Location.search 변경
        │
        ▼
URL 변경
```

입니다.

---

# 11. 기존 Query Parameter는 자동으로 유지되지 않는다

현재 URL이:

```text id="6gglhi"
/products?page=2&keyword=react
```

이라고 하겠습니다.

이 상태에서:

```jsx id="ufvn53"
setSearchParams({
  page: '3'
});
```

를 호출하면 기존 `keyword`가 자동으로 유지된다고 생각하면 안 됩니다.

새로운 Query Parameter 집합을 설정한 것이므로 결과는 일반적으로:

```text id="imlm33"
/products?page=3
```

가 됩니다.

따라서 일부 값만 수정하고 나머지는 그대로 유지하려면 기존 Query Parameter를 복사한 뒤 원하는 값만 변경해야 합니다.

---

# 12. 기존 값을 유지하면서 일부만 변경하기

현재 URL:

```text id="i21w9v"
/products?page=2&keyword=react&sort=latest
```

여기서 `page`만 변경하고 싶다면 다음처럼 작성할 수 있습니다.

```jsx id="4226nq"
const updatePage = () => {
  const next =
    new URLSearchParams(searchParams);

  next.set('page', '3');

  setSearchParams(next);
};
```

결과:

```text id="nrx8p4"
/products?page=3&keyword=react&sort=latest
```

흐름은 다음과 같습니다.

```text id="cmrhb7"
현재 searchParams
       │
       ▼
URLSearchParams 복사
       │
       ▼
page 수정
       │
       ▼
setSearchParams()
       │
       ▼
새 Query String으로 Navigation
```

---

# 13. 함수형 업데이트

현재 Query Parameter를 기준으로 변경해야 할 때 함수 형태도 사용할 수 있습니다.

```jsx id="mhlzxx"
setSearchParams(prev => {
  const next =
    new URLSearchParams(prev);

  next.set('page', '3');

  return next;
});
```

개념적으로 보면:

```text id="hgbd0x"
현재 Params
    │
    ▼
Updater Function
    │
    ▼
새 Params 생성
    │
    ▼
setSearchParams
```

입니다.

---

# 14. `searchParams`는 실제로 수정 가능한 객체다

`searchParams`를 읽기 전용 객체라고 표현하면 정확하지 않습니다.

다음 코드는 JavaScript 차원에서는 가능합니다.

```jsx id="q884ub"
searchParams.set('page', '5');
```

즉, 객체 자체에는 변경 메서드가 있습니다.

하지만:

```jsx id="uaskds"
searchParams.set('page', '5');
```

만 실행해서는 React Router Navigation이 발생하지 않습니다.

URL을 바꾸려면 결국 `setSearchParams()`를 호출해야 합니다.

그래서 교육용 코드에서는 다음 패턴이 가장 명확합니다.

```jsx id="ezho8u"
const next =
  new URLSearchParams(searchParams);

next.set('page', '5');

setSearchParams(next);
```

즉:

```text id="98yxmp"
searchParams
    │
    │ 복사
    ▼
새 URLSearchParams
    │
    │ 변경
    ▼
setSearchParams()
    │
    ▼
URL 변경
```

입니다.

---

# 15. `set()`과 `append()`는 다르다

현재 Query Parameter가:

```text id="k0w9nn"
category=shoes
```

라고 하겠습니다.

`set()`을 사용하면:

```jsx id="baq17r"
params.set('category', 'coat');
```

결과:

```text id="5zgil0"
category=coat
```

가 됩니다.

기존 값을 바꾸는 동작입니다.

반면:

```jsx id="cr89nk"
params.append('category', 'coat');
```

을 사용하면:

```text id="3q73ax"
category=shoes&category=coat
```

가 됩니다.

즉:

```text id="izl6ia"
set()
 ↓
기존 값 교체


append()
 ↓
추가 값 삽입
```

입니다.

---

# 16. 동일한 key에 여러 값 넣기

다음과 같은 Query String을 만들고 싶다고 하겠습니다.

```text id="awqzbz"
?category=shoes&category=coat
```

이 경우 `URLSearchParams`를 직접 사용하면 명확합니다.

```jsx id="9q549g"
const params = new URLSearchParams();

params.append('category', 'shoes');
params.append('category', 'coat');

setSearchParams(params);
```

결과:

```text id="zf17ei"
?category=shoes&category=coat
```

읽을 때는:

```jsx id="8n0vc0"
searchParams.getAll('category');
```

를 사용합니다.

결과:

```js id="5iuxyt"
[
  'shoes',
  'coat'
]
```

---

# 17. Query Parameter 삭제하기

현재 URL이:

```text id="8jktjp"
/products?page=2&keyword=react
```

이라고 하겠습니다.

`keyword`를 제거하려면:

```jsx id="gzr6ii"
const next =
  new URLSearchParams(searchParams);

next.delete('keyword');

setSearchParams(next);
```

결과는:

```text id="ubyerz"
/products?page=2
```

입니다.

---

# 18. `setSearchParams()`는 단순 State Setter가 아니다

`setSearchParams()`를 `useState()`의 setter와 완전히 같은 개념으로 보면 안 됩니다.

다음 코드는:

```jsx id="k9uqqh"
setSearchParams({
  page: '3'
});
```

현재 Route의 Query String을 바꿉니다.

즉:

```text id="r5pvoi"
/products?page=2
       │
       │ setSearchParams(...)
       ▼
/products?page=3
```

처럼 Location이 변경됩니다.

전체 흐름은 다음과 같습니다.

```text id="501ewx"
setSearchParams()
       │
       ▼
새 search 생성
       │
       ▼
React Router Navigation
       │
       ▼
Location 변경
       │
       ▼
관련 React UI 업데이트
```

따라서 `setSearchParams()`는 Query Parameter를 변경하는 동시에 React Router Navigation을 발생시키는 API라고 이해하는 것이 좋습니다.

---

# 19. 기본 동작은 새로운 History Entry를 만든다

일반적인 `setSearchParams()` 호출은 새로운 Location으로 이동합니다.

현재:

```text id="vcp007"
/products?page=1
```

에서:

```jsx id="leoh0x"
setSearchParams({
  page: '2'
});
```

를 실행하면 History는 개념적으로:

```text id="oolwvg"
/products?page=1
/products?page=2   ← 현재
```

처럼 됩니다.

그래서 브라우저의 뒤로 가기를 이용하면 이전 Query 상태로 돌아갈 수 있습니다.

---

# 20. `replace: true`

History Entry를 새로 추가하지 않고 현재 Entry를 교체할 수도 있습니다.

```jsx id="2y27rx"
setSearchParams(
  {
    page: '2'
  },
  {
    replace: true
  }
);
```

개념적으로:

```text id="njkdlw"
Before

/products?page=1
        │
        │ replace
        ▼

After

/products?page=2
```

입니다.

검색어 입력처럼 값이 매우 자주 변경되는 경우에는 History가 과도하게 쌓이지 않도록 `replace`를 고려할 수 있습니다.

---

# 21. `useLocation()`과 무엇이 다른가?

현재 URL:

```text id="69lxyf"
/products?page=2&sort=latest
```

에서 `useLocation()`을 사용하면:

```jsx id="4wgrtm"
const location = useLocation();

console.log(location.search);
```

결과는:

```text id="qiv52t"
?page=2&sort=latest
```

입니다.

즉, `useLocation()`은 전체 Query String을 문자열로 제공합니다.

반면 `useSearchParams()`에서는:

```jsx id="tzx9j5"
const [searchParams] =
  useSearchParams();
```

이후:

```jsx id="binfxo"
searchParams.get('page');
```

처럼 개별 값을 바로 다룰 수 있습니다.

```text id="fzo0gd"
useLocation()
     │
     ▼
"?page=2&sort=latest"


useSearchParams()
     │
     ├── page → "2"
     └── sort → "latest"
```

입니다.

---

# 22. `useParams()`와 무엇이 다른가?

현재 URL이:

```text id="07gxbb"
/products/10?page=2
```

이고 Route가:

```jsx id="vcrlxb"
<Route
  path="/products/:productId"
  element={<Product />}
/>
```

라고 하겠습니다.

구조를 나누면:

```text id="twrksk"
/products/10?page=2
          │      │
          │      └── Query Parameter
          │
          └───────── Path Parameter
```

`useParams()`:

```jsx id="xilntm"
const { productId } = useParams();
```

결과:

```text id="asp6xl"
"10"
```

`useSearchParams()`:

```jsx id="rma5y3"
const [searchParams] =
  useSearchParams();

searchParams.get('page');
```

결과:

```text id="mfroey"
"2"
```

따라서:

```text id="cjk4vm"
Path Parameter
      ↓
 useParams()


Query Parameter
      ↓
useSearchParams()
```

라고 구분하면 됩니다.

---

# 23. 검색 UI와 연결하기

예를 들어 검색어를 URL에 저장하고 싶다고 하겠습니다.

```text id="6eb2au"
/products?keyword=react
```

다음처럼 작성할 수 있습니다.

```jsx id="7btceh"
import { useSearchParams } from 'react-router-dom';

function SearchBox() {
  const [searchParams, setSearchParams] =
    useSearchParams();

  const keyword =
    searchParams.get('keyword') ?? '';

  const handleSearch = value => {
    const next =
      new URLSearchParams(searchParams);

    next.set('keyword', value);

    setSearchParams(next);
  };

  return (
    <input
      value={keyword}
      onChange={e =>
        handleSearch(e.target.value)
      }
    />
  );
}
```

흐름은 다음과 같습니다.

```text id="x39ko9"
사용자 입력
    │
    ▼
keyword 변경
    │
    ▼
setSearchParams()
    │
    ▼
URL 변경
    │
    ▼
?keyword=react
```

이 방식에서는 검색어 상태가 컴포넌트 내부에만 머무르지 않고 URL에 반영됩니다.

---

# 24. Pagination에 적용하기

현재 URL:

```text id="pi0348"
/products?page=2
```

다음과 같이 작성할 수 있습니다.

```jsx id="juifli"
function Products() {
  const [searchParams, setSearchParams] =
    useSearchParams();

  const page =
    Number(searchParams.get('page') ?? 1);

  const nextPage = () => {
    const next =
      new URLSearchParams(searchParams);

    next.set(
      'page',
      String(page + 1)
    );

    setSearchParams(next);
  };

  return (
    <>
      <p>현재 페이지: {page}</p>

      <button onClick={nextPage}>
        다음 페이지
      </button>
    </>
  );
}
```

동작은:

```text id="z99d1q"
/products?page=2
       │
       │ 다음 페이지 클릭
       ▼
page + 1
       │
       ▼
setSearchParams()
       │
       ▼
/products?page=3
```

처럼 이어집니다.

---

# 25. 검색, 정렬, 필터를 함께 관리하기

현재 URL:

```text id="cd0ycf"
/products?category=shoes&sort=latest&page=2
```

각 값은 다음처럼 읽을 수 있습니다.

```jsx id="5mh2dv"
const category =
  searchParams.get('category') ?? 'all';

const sort =
  searchParams.get('sort') ?? 'latest';

const page =
  Number(searchParams.get('page') ?? 1);
```

정렬만 변경하고 싶다면:

```jsx id="z6rnlw"
const changeSort = sort => {
  const next =
    new URLSearchParams(searchParams);

  next.set('sort', sort);

  setSearchParams(next);
};
```

처럼 기존 Parameter를 유지한 채 일부만 바꿀 수 있습니다.

---

# 26. 검색 + 정렬 + Pagination 전체 예제

```jsx id="jkgmsh"
import { useSearchParams } from 'react-router-dom';

function Products() {
  const [searchParams, setSearchParams] =
    useSearchParams();

  const keyword =
    searchParams.get('keyword') ?? '';

  const sort =
    searchParams.get('sort') ?? 'latest';

  const page =
    Number(searchParams.get('page') ?? 1);

  const updateParam = (key, value) => {
    setSearchParams(prev => {
      const next =
        new URLSearchParams(prev);

      next.set(key, String(value));

      return next;
    });
  };

  const nextPage = () => {
    updateParam('page', page + 1);
  };

  const changeSort = value => {
    updateParam('sort', value);
  };

  return (
    <div>
      <h2>상품 목록</h2>

      <p>검색어: {keyword}</p>
      <p>정렬: {sort}</p>
      <p>현재 페이지: {page}</p>

      <button
        onClick={() =>
          changeSort('latest')
        }
      >
        최신순
      </button>

      <button
        onClick={() =>
          changeSort('popular')
        }
      >
        인기순
      </button>

      <button onClick={nextPage}>
        다음 페이지
      </button>
    </div>
  );
}
```

현재 URL이:

```text id="li373w"
/products?keyword=shoes&page=2&sort=latest
```

이고 인기순 버튼을 누르면:

```text id="msdbls"
/products?keyword=shoes&page=2&sort=popular
```

로 바뀝니다.

---

# 27. URL 상태와 Local State는 구분해야 한다

모든 UI 상태를 Query Parameter에 넣는 것은 적절하지 않습니다.

예를 들어:

```text id="e5mtd1"
검색어
페이지 번호
정렬
필터
```

처럼 새로고침이나 URL 공유 후에도 같은 화면 상태가 유지되어야 하는 값은 Query Parameter와 잘 맞습니다.

반면:

```text id="kpw21s"
Modal 열림 여부
버튼 Hover 여부
현재 Input Focus
임시 UI Animation 상태
```

같은 값은 일반적으로 Component State가 더 적합합니다.

판단 기준을 단순화하면 다음과 같습니다.

```text id="cffhnt"
URL을 공유했을 때
같은 화면 상태가 재현되어야 하는가?
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
Query      useState
Parameter  등 Local State
```

---

# 28. Query Parameter도 검증이 필요하다

URL은 사용자가 직접 바꿀 수 있습니다.

예를 들어:

```text id="putr5w"
/products?page=abc
/products?page=-10
/products?sort=unknown
```

같은 값도 들어올 수 있습니다.

따라서:

```jsx id="ijqhc1"
const page =
  Number(searchParams.get('page') ?? 1);
```

로 변환한 뒤 필요하다면 유효성 검사를 해야 합니다.

예:

```jsx id="y6810o"
const rawPage =
  Number(searchParams.get('page'));

const page =
  Number.isInteger(rawPage) &&
  rawPage > 0
    ? rawPage
    : 1;
```

흐름은 다음처럼 생각할 수 있습니다.

```text id="h4tkhq"
Query Parameter
       │
       ▼
문자열
       │
       ▼
Parsing
       │
       ▼
Validation
       │
       ▼
Application Logic
```

---

# 29. 내부 동작을 개념적으로 보면

현재 URL이:

```text id="fdwpjj"
/products?page=2
```

라고 하겠습니다.

다음 코드를 실행합니다.

```jsx id="mrhc4d"
setSearchParams({
  page: '3'
});
```

전체 흐름은 개념적으로:

```text id="sjuot9"
React Component
      │
      │ setSearchParams(...)
      ▼
React Router
      │
      ▼
새 Search 생성
      │
      ▼
?page=3
      │
      ▼
Navigation
      │
      ▼
Location 변경
      │
      ▼
새 Router 상태
      │
      ▼
관련 UI 렌더링
```

입니다.

그래서 `setSearchParams()`는 단순한 문자열 편집 함수라기보다 Query String 변경을 React Router Navigation으로 연결하는 API라고 이해하는 편이 정확합니다.

---

# 30. `window.location.search`와의 차이

브라우저에는 원래:

```js id="bf8e0p"
window.location.search
```

가 있습니다.

예를 들어:

```js id="ygvuaf"
console.log(window.location.search);
```

결과:

```text id="9w5mxe"
?page=2&sort=latest
```

를 얻을 수 있습니다.

하지만 React Router를 사용하는 컴포넌트에서는 일반적으로 `useSearchParams()`를 사용합니다.

이유는 Query Parameter를 단순히 읽는 데서 끝나는 것이 아니라 React Router의 Location과 Navigation 흐름에 연결해서 다룰 수 있기 때문입니다.

```text id="niluqw"
Browser URL
      │
      ▼
React Router
      │
      ▼
Location.search
      │
      ▼
useSearchParams()
      │
      ▼
React Component
```

---

# 31. 관련 Hook 비교

| Hook                | 역할                          |
| ------------------- | --------------------------- |
| `useSearchParams()` | Query Parameter 읽기/변경       |
| `useParams()`       | 현재 Route의 Path Parameter 읽기 |
| `useLocation()`     | 현재 Location 객체 읽기           |
| `useNavigate()`     | 코드에서 Navigation 실행          |
| `useMatch()`        | 현재 pathname과 Pattern 비교     |
| `useResolvedPath()` | 상대적인 `to` 값을 Path로 해석       |

질문으로 바꾸면 더 쉽게 구분할 수 있습니다.

```text id="4m0u6z"
/users/10?page=2
        │      │
        │      └── "page는 무엇인가?"
        │             ↓
        │       useSearchParams()
        │
        └────────── "userId는 무엇인가?"
                      ↓
                  useParams()


현재 URL 전체 상태는?
        ↓
   useLocation()


코드로 다른 곳으로 이동?
        ↓
   useNavigate()
```

---

# 32. 언제 사용하면 좋은가?

| 상황                | 적합성       |
| ----------------- | --------- |
| 검색어               | 매우 적합     |
| Pagination        | 매우 적합     |
| 정렬 기준             | 매우 적합     |
| 필터 조건             | 매우 적합     |
| Tab 상태            | 경우에 따라 적합 |
| 공유 가능한 화면 상태      | 적합        |
| 새로고침 후 복원할 상태     | 적합        |
| 단순 Modal 표시       | 보통 부적합    |
| Hover 상태          | 부적합       |
| Component 내부 임시 값 | 보통 부적합    |

판단 기준은 다음 질문으로 정리할 수 있습니다.

> 이 상태가 URL에 들어가 있을 때 사용자에게 의미가 있는가?

---

# 33. 전체 흐름 정리

`useSearchParams()`의 역할을 전체 구조로 보면 다음과 같습니다.

```text id="cl1o3l"
Browser URL
/products?page=2&sort=latest
          │
          ▼
     React Router
          │
          ▼
       Location
          │
          └── search
               │
               ▼
        useSearchParams()
               │
       ┌───────┴──────────┐
       │                  │
       ▼                  ▼
 searchParams       setSearchParams
       │                  │
       │                  ▼
       │              Navigation
       │                  │
       ▼                  ▼
 Query 읽기         Query 변경
```

---

# 34. 핵심 정리

`useSearchParams()`를 단순히 “Query String을 읽는 Hook”이라고만 기억하면 기능의 절반만 이해한 것입니다.

더 정확하게 표현하면:

> `useSearchParams()`는 React Router가 관리하는 현재 Location의 Query Parameter를 `URLSearchParams` 형태로 읽게 해주고, 새로운 Query Parameter를 설정하여 네비게이션까지 수행할 수 있게 해주는 Hook입니다.

가장 짧게 정리하면 다음과 같습니다.

```text id="2fqifa"
현재 URL
   │
   ▼
Location.search
   │
   ▼
useSearchParams()
   │
   ├── searchParams
   │       ↓
   │     읽기
   │
   └── setSearchParams()
           ↓
        변경 + Navigation
```

즉, `useSearchParams()`의 핵심은 Query Parameter를 단순 문자열이 아니라 **React Router가 관리하는 URL 상태의 일부로 다룬다는 점**입니다.
