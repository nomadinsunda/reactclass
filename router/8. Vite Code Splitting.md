
# ⚡ Vite 관점의 코드 스플리팅

## 1. 🧠 기본 개념 요약

| 항목              | Vite                | Webpack              |
| --------------- | ------------------- | -------------------- |
| 번들러             | `Rollup` 기반 (빌드시)   | 자체 번들러               |
| dev 모드          | 번들 없음, ESM 직접 서빙    | 번들 있음 (HMR 필요)       |
| 코드 스플리팅         | ESM import 기준 자동 분리 | 수동 or SplitChunks 설정 |
| lazy + suspense | 동일하게 동작             | 동일하게 동작              |

---

## 2. 🔥 Vite의 코드 스플리팅 방식

Vite는 **ESM(import/export) 문법 자체를 코드 스플리팅의 기준점**으로 삼습니다.

```js
// Lazy import → 별도 chunk 생성
const Chat = React.lazy(() => import('./components/Chat'));
```

✅ 위 코드 한 줄만으로 Vite는 아래와 같이 청크를 분리합니다:

```
dist/assets/
├── index.html
├── index-xxxxx.js        ← main entry
├── Chat-xxxxx.js         ← 👈 lazy된 컴포넌트
└── vendor-xxxxx.js       ← 공통 라이브러리
```

📦 즉, `import()` 구문이 **lazy chunk 생성의 신호**가 됩니다.

---

## 3. 🚀 개발 환경(dev server)에서의 차이

| 구분      | 설명                                            |
| ------- | --------------------------------------------- |
| Vite    | 개발 중에는 **번들을 생성하지 않고**, 각 모듈을 **ESM으로 직접 서빙** |
| Webpack | 개발 중에도 모든 코드를 번들로 만들고 HMR로 핫 리로드              |

📌 Vite는 개발 서버에서도 실제 chunk를 나누는 대신, 브라우저가 필요한 모듈만 요청합니다.
따라서 lazy된 컴포넌트는 **실제로 `import()`가 실행될 때** `.jsx` 파일을 fetch합니다.

---

## 4. 📦 Vite에서의 코드 스플리팅 예시

```jsx
// App.jsx
const Settings = lazy(() => import('./pages/Settings'));

<Suspense fallback={<Spinner />}>
  <Settings />
</Suspense>
```

👉 `npm run build` 실행 시:

```bash
vite v5.0.0 building...
✓ 1 modules transformed.
dist/assets/
├── index.html
├── index-abcde.js
├── Settings-c3a7d.js       ✅ 이게 lazy chunk!
└── vendor-df87a.js
```

> ✅ chunk 이름은 자동 생성되며 `Rollup`이 내부적으로 dependency graph를 분석해서 코드 분할을 수행

---

## 5. ⚙️ Vite 빌드 최적화 전략

```ts
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            if (id.includes('react')) return 'vendor-react';
            if (id.includes('lodash')) return 'vendor-lodash';
            return 'vendor';
          }
        },
      },
    },
  },
});
```

| 목적                       | 효과                |
| ------------------------ | ----------------- |
| manualChunks 설정          | chunk 분리 전략 수동 제어 |
| `vendor-react`           | React 계열 분리       |
| `Settings` 등 lazy import | 페이지별 청크 분리 자동 수행  |

---

## 6. 🧪 코드 스플리팅 확인 방법

1. `npm run build`
2. `npm run preview` 실행
3. Chrome DevTools → Network 탭에서 JS 요청 확인

### ✅ 예시

| 경로           | 요청                    |
| ------------ | --------------------- |
| `/dashboard` | `Dashboard-xxxxx.js`  |
| `/chat`      | `ChatWidget-xxxxx.js` |
| `/settings`  | `Settings-xxxxx.js`   |

---

## 7. ❗ 주의할 점

| 항목                    | 설명                                 |
| --------------------- | ---------------------------------- |
| dynamic import만 분할 대상 | static import는 항상 main bundle에 포함됨 |
| SSR에서는 제한 있음          | Vite SSR은 lazy chunk를 처리하는 전략이 다름  |
| dev/preview 환경 차이 존재  | 개발 서버는 번들 없음, preview는 번들 기준       |

---

## 🧾 요약

| 항목           | 설명                               |
| ------------ | -------------------------------- |
| Vite의 기준     | `import()`가 코드 스플리팅 트리거          |
| lazy 적용 방법   | React.lazy(() => import(...))    |
| 빌드시 결과       | 자동으로 chunk 생성 (`xxx.chunk.js`)   |
| dev 서버       | 실시간 모듈 서빙 (ESM 기반, 번들 X)         |
| Webpack과 차이점 | Vite는 빌드시만 번들, Webpack은 dev부터 번들 |

