> ✅ **Next.js는 React 기반 애플리케이션에서 SSR(Server Side Rendering)을 매우 쉽게 구현할 수 있도록 만들어진 프레임워크**입니다.

하지만 **단순히 SSR만 지원하는 게 아니라**,
**CSR, SSG, ISR, API Routes, 라우팅, 번들링, 코드 스플리팅, 이미지 최적화 등**을 모두 포함한 **React의 풀스택 프레임워크**입니다.

---

## ✅ Next.js의 핵심 목적

| 목적                 | 설명                                                  |
| ------------------ | --------------------------------------------------- |
| 🔹 **SSR 지원**      | React 앱을 서버에서 렌더링 가능하게 함 (SEO, 성능 개선)               |
| 🔹 **파일 기반 라우팅**   | `pages/index.js` → `/`, `pages/about.js` → `/about` |
| 🔹 **SSG & ISR**   | 정적 사이트 생성 (Static Site Generation), 점진적 재생성도 지원     |
| 🔹 **API 라우트**     | Express 없이도 `pages/api/*.js`로 API 엔드포인트 생성          |
| 🔹 **풀스택 개발 지원**   | React + Node 기능을 모두 한 프로젝트에서 구현 가능                  |
| 🔹 **자동 번들링, 최적화** | Webpack, Babel 설정 없이도 바로 사용 가능                      |

* Express : React/Vue 프론트엔드의 백엔드 API 서버
---

## ✅ 핵심 기능별 설명

### 1. ✅ SSR (Server Side Rendering)

* `getServerSideProps()`를 사용
* 매 요청 시 서버에서 HTML을 생성하여 브라우저에 전달

```js
// pages/index.js
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  return { props: { posts } };
}
```

### 2. ✅ SSG (Static Site Generation)

* `getStaticProps()` 사용
* **빌드 시점에 HTML 생성 → CDN에 배포**

```js
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();
  return { props: { posts } };
}
```

### 3. ✅ ISR (Incremental Static Regeneration)

* **정적 페이지를 주기적으로 자동 갱신**
* 예: `revalidate: 60` → 60초마다 새로 빌드

```js
export async function getStaticProps() {
  return {
    props: { ... },
    revalidate: 60, // 60초마다 새로 빌드
  };
}
```

### 4. ✅ CSR (Client Side Rendering)

* 컴포넌트 내부에서 `useEffect()` 등으로 브라우저에서 데이터 fetch

```js
function Page() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch('/api/hello').then(res => res.json()).then(setData);
  }, []);
  return <div>{data ? data.message : '로딩 중...'}</div>;
}
```

---

## ✅ Next.js는 SSR에 "특화"된 것이 맞는가?

### ✔️ 그렇습니다.

Next.js는 **SSR을 React로 구현할 수 있도록 만든 최초의 실전 프레임워크** 중 하나이며, 다음과 같은 점에서 SSR에 특화되어 있습니다:

| 특화된 점         | 설명                                    |
| ------------- | ------------------------------------- |
| SSR 라우트 지원    | `getServerSideProps()`로 요청 시마다 서버 렌더링 |
| 자동 라우팅        | `pages/*.js`로 자동 라우팅, SSR 가능          |
| 개발 서버 포함      | Node.js 기반 서버 내장 (SSR 결과 렌더링 서버로 작동)  |
| Hydration 자동화 | 서버에서 만든 HTML에 클라이언트 JS 자동 연결          |
| SEO 대응 최적화    | `<Head>`, `<meta>` 자동 관리로 검색 노출 유리    |

---

## ✅ 요약: Next.js란?

| 항목         | 설명                                     |
| ---------- | -------------------------------------- |
| 기반         | React                                  |
| SSR 지원     | ✅ 매우 잘 됨                               |
| CSR 지원     | ✅ 기본 React처럼 가능                        |
| SSG 지원     | ✅ Gatsby처럼 정적 페이지 생성                   |
| ISR 지원     | ✅ 정적 + 동적 재생성 하이브리드                    |
| Node.js 필요 | ✅ 서버 측 렌더링을 위해 필요 (Express 없이도 동작)     |
| API 서버 내장  | ✅ `/pages/api/*.js`로 Node.js API 작성 가능 |
| 프레임워크 역할   | React의 풀스택 프레임워크 (Vue의 Nuxt와 대응)       |

---

## ✅ 결론

> ✔️ **Next.js는 React의 SSR 기능을 공식적으로, 쉽게 구현할 수 있도록 만든 프레임워크입니다.**
> ✔️ 하지만 SSR만 하는 게 아니라 **CSR, SSG, ISR, API까지 포함한 종합 풀스택 도구입니다.**
> ✔️ React + Node.js 기반으로 고급 웹앱 개발 시 거의 표준처럼 사용됩니다.


