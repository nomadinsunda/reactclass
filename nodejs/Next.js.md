# 🚀 Next.js란?

**Next.js**는 **React 기반의 풀스택 웹 프레임워크**로, Vercel에서 개발한 오픈소스입니다.
단순한 React 라이브러리가 아니라 **웹 애플리케이션을 서버와 클라이언트 모두에서 효율적으로 실행하도록 설계된 프레임워크**입니다.

React가 **UI 라이브러리**라면, Next.js는 다음을 포함한 **애플리케이션 구조 전체를 제공**합니다.

---

# 🧱 1. Next.js를 왜 쓰는가? (React의 한계를 보완)

| 기능      | React                          | Next.js                       |
| ------- | ------------------------------ | ----------------------------- |
| 렌더링 방식  | CSR(Client Side Rendering)만 가능 | CSR + SSR + SSG + ISR 모두 가능   |
| 라우팅     | 직접 Router 설치 필요                | **파일 기반 라우팅 자동 지원**           |
| SEO     | CSR만으로는 SEO 불리                 | SSR/SSG로 **SEO 최적화 강함**       |
| API 서버  | 없음                             | **API Routes 제공 → 풀스택 개발 가능** |
| 이미지 최적화 | 없음                             | 빌트인 이미지 최적화 기능                |
| 번들링/빌드  | 설정 직접해야 함                      | Webpack·SWC 모두 자동 구성          |

즉, **Next.js = React의 불편함을 모두 해결하는 상위 프레임워크**라고 보시면 됩니다.

---

# ⚙️ 2. Next.js의 핵심 특징

## ✅ 1) 다중 렌더링 방식 지원

Next.js의 가장 큰 매력입니다.

### ● **SSR (Server-Side Rendering)**

요청이 올 때마다 서버에서 HTML 생성
→ SEO, 초기 로딩 속도 매우優

### ● **SSG (Static Site Generation)**

빌드 타임에 HTML을 미리 생성
→ 매우 빠른 속도, CDN 배포 최적화

### ● **ISR (Incremental Static Regeneration)**

정적 페이지를 일정 주기마다 재생성
→ SSG + 동적 데이터의 중간 형태

### ● **CSR (Client Rendering)**

React 기본 방식도 사용 가능

➡️ **4가지 방식 중 필요에 따라 유연하게 선택 가능**

---

# 📁 3. 파일 기반 라우팅 (Routing)

Next.js는 **pages/ 디렉터리 기반의 라우팅 시스템**을 제공합니다.

예)

```
pages/
 ├── index.js        → /
 ├── about.js        → /about
 └── products/
        └── [id].js  → /products/:id (동적 라우팅)
```

React Router를 직접 설치하거나 설정할 필요가 없습니다.

---

# 🔌 4. API Routes로 서버까지 가능 (Full-stack)

React는 프론트엔드만 담당하지만
Next.js는 다음과 같은 API 서버도 만들 수 있습니다.

```
/pages/api/users.js
```

이 파일 하나로 `/api/users` API가 자동 생성됩니다.

즉, **Next.js 하나로 프론트 + 서버 API 모두 개발 가능**합니다.

---

# 🖼️ 5. 이미지 최적화(Image Optimization)

`next/image` 컴포넌트가 자동으로:

* WebP 변환
* 사이즈 최적화
* 지연 로딩(lazy loading)
* CDN 캐싱

등을 수행합니다.

→ 이미지가 많은 웹사이트에서 **성능 향상 효과가 매우 큼**

---

# 📦 6. 번들링/트랜스파일링 자동 처리

Next.js는 아래 작업을 자동으로 해 줍니다.

* Webpack/SWC 기반 번들링
* 코드 스플리팅
* 트리 쉐이킹
* 폴리필 자동 관리

개발자 입장에서 “환경 설정 지옥”에서 해방됩니다.

---

# 🧭 7. Next.js 13+의 App Router (최신)

Next.js 13부터는 **App Router**라는 새로운 구조를 제공합니다.

* React Server Components 기반
* 폴더 구조가 라우터
* 서버/클라이언트 컴포넌트 분리
* 로딩 UI, 에러 UI 구조화
* Layout 시스템 강화

예)

```
app/
 ├─ layout.js
 ├─ page.js
 ├─ loading.js
 ├─ error.js
 └─ dashboard/
        ├─ page.js
        └─ layout.js
```

완전히 새로운 패러다임으로 **프론트엔드와 백엔드 경계가 희미해짐**.

---

# 🎯 결론: Next.js는 어떤 프레임워크인가?

> **Next.js는 React 개발을 위한 “최적화된 풀스택 프레임워크”이며
> SEO, 성능, 개발 편의성을 극대화한 현대적 웹 개발 표준입니다.**

특히:

✔ 서버 렌더링(SSR)
✔ 정적 페이지 생성(SSG/ISR)
✔ 쉬운 라우팅
✔ API 서버 가능
✔ 성능 최적화 자동 제공

이런 이유 때문에 대규모 서비스들도 Next.js를 적극적으로 사용하고 있습니다.
(Netflix, TikTok, GitHub Copilot 사이트, Nike, Uber 등)
