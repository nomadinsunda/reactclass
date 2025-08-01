
# 🧩 코드 스플리팅(Code Splitting)이란?

**코드 스플리팅**이란 웹 애플리케이션의 JavaScript 번들을 **여러 개의 작은 청크(chunk)** 로 분할하여,
**필요할 때만 로드**할 수 있도록 하는 기술입니다.

> ✅ 핵심 목적:
>
> * 초기 로딩 속도 개선
> * 렌더링 성능 최적화
> * 사용자 경험(UX) 향상

---

## 🔥 왜 필요한가?

### 💥 문제: "모놀리식 번들(monolithic bundle)"

* React, 앱 코드, 페이지 컴포넌트, 유틸, 아이콘 등 모든 JS를 **한 번들로 번역**하면:

  * `bundle.js`가 **수백 KB \~ MB**
  * 첫 페이지 진입에 **오래 걸림**
  * 사용하지도 않은 페이지 코드까지 로드됨

---

## 🧪 코드 스플리팅의 유형

| 유형                 | 설명                    | 예시                          |
| ------------------ | --------------------- | --------------------------- |
| ✅ Entry-based      | SPA 진입점을 나눔           | 다중 페이지 앱 (MPA)              |
| ✅ Route-based      | 페이지 단위 분할             | `React.lazy()` + `Suspense` |
| ✅ Component-based  | 컴포넌트 단위 분할            | 버튼 클릭 → 다이얼로그 import        |
| ❌ Manual Splitting | Webpack 설정으로 수동 청크 분리 | SplitChunksPlugin 등         |

---

## 📦 React에서 코드 스플리팅 적용 방법

### 1. ✅ Route-based Splitting (가장 흔함)

```jsx
const Dashboard = React.lazy(() => import('./pages/Dashboard'));

<Suspense fallback={<Loading />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
  </Routes>
</Suspense>
```

* `Dashboard` 페이지에 접근할 때만 해당 JS 번들이 네트워크 요청됨
* 초기 로딩 시 이 페이지는 제외됨

---

### 2. ✅ Component-based Splitting

```jsx
const ChatWidget = React.lazy(() => import('./components/ChatWidget'));

function App() {
  const [showChat, setShowChat] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChat(true)}>Open Chat</button>
      {showChat && (
        <Suspense fallback={<div>Loading chat...</div>}>
          <ChatWidget />
        </Suspense>
      )}
    </div>
  );
}
```

* 버튼 클릭 시에만 **ChatWidget** 번들 로딩
* 불필요한 코드까지 처음부터 로딩하지 않음

---

## 🎯 코드 스플리팅이 UX에 주는 이점

| 항목           | 영향                   |
| ------------ | -------------------- |
| 🚀 초기 로딩 속도  | 빨라짐 (핵심 기능만 먼저 다운로드) |
| 🧠 사용자의 인지   | 로딩 대기 시간 단축, 빠른 반응   |
| 📶 네트워크 효율   | 필요한 순간에 필요한 코드만 전송   |
| 📱 모바일 UX 개선 | 트래픽 절감, 성능 개선        |

---



## ❗️주의할 점

| 항목             | 설명                                     |
| -------------- | -------------------------------------- |
| fallback 필수    | Lazy 컴포넌트는 `Suspense`와 함께 써야 함         |
| default export | `React.lazy()`는 default export만 지원     |
| SSR 제한         | React 18 이전은 SSR에서 lazy 사용 불가          |
| 번들 전략 혼용 금지    | Static import와 lazy import가 섞이면 관리 어려움 |

---

## 📘 실무 적용 시 체크리스트

* ✅ 라우터 페이지는 반드시 lazy + Suspense 처리
* ✅ 무거운 컴포넌트(모달, 차트, 에디터)는 동적 로딩 처리
* ✅ Webpack + Bundle Analyzer로 청크 확인
* ✅ Suspense fallback UI는 사용자 컨텍스트에 맞게

---

## 🔚 요약

| 항목    | 요약                               |
| ----- | -------------------------------- |
| 정의    | 필요할 때만 JS 코드를 로드하는 기술            |
| 도구    | `React.lazy()` + `Suspense`      |
| 목적    | 초기 로딩 성능 향상, UX 개선               |
| 대상    | 페이지, 모달, 다이얼로그, 무거운 위젯 등         |
| 도구 보조 | Webpack, Vite, Bundle Analyzer 등 |

