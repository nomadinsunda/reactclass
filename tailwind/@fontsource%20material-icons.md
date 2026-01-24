`@fontsource/material-icons`는 **Google Material Icons를 NPM 패키지로 자기 서버(또는 번들)에서 직접 서빙하게 해주는 라이브러리**입니다. 아이콘 CDN 링크를 `<link>`로 붙이는 대신, 그냥 `npm install` 해서 폰트를 프로젝트 안에 집어넣는 방식이죠. 😎

아래에서 개념 → 설치 → Vite + React 예시 → 심화 사용법 순으로 정리해볼게요.

---

## 1. `@fontsource/material-icons`가 정확히 뭔가요? 🤔

### 1-1. Fontsource란?

**Fontsource**는 Google Fonts 같은 웹폰트들을 “CDN이 아니라 내 앱 번들에서 직접 서빙하도록” NPM 패키지로 제공하는 프로젝트입니다. 폰트마다 `@fontsource/폰트이름` 식의 패키지를 제공하고, 이것들을 import 하면 `@font-face`와 `.woff2`/`.woff` 파일이 전부 프로젝트 안으로 들어옵니다. ([fontsource.org][1])

### 1-2. `@fontsource/material-icons`의 역할

`@fontsource/material-icons`는 그 중에서 **Google Material Icons 아이콘 폰트**만 따로 빼서 제공하는 패키지입니다. 설명 그대로:

> “Self-host the Material Icons font in a neatly bundled NPM package.” ([unpkg.com][2])

즉:

* Google CDN에서 `https://fonts.googleapis.com/icon?family=Material+Icons` 가져오던 걸
* `npm install @fontsource/material-icons` + `import '@fontsource/material-icons';` 로 **로컬(자기 서버)에서 제공**하는 형태로 바꿔주는 역할입니다. ([fontsource.org][3])

---

## 2. 왜 굳이 이걸 써야 하나요? 💡

CDN `<link>` 쓰면 되는데 굳이 패키지를 쓰는 이유는:

1. **자기 호스팅(Self-host)**

   * 내 앱이 뜨는 도메인에서 폰트가 서빙되므로
   * **네트워크 단절/내부망/사내망 환경**에서도 잘 동작합니다. ([Stack Overflow][4])

2. **프라이버시 & 정책 문제 회피**

   * 어떤 회사/기관은 외부 CDN을 막거나 최소화하라고 요구합니다.
   * NPM 패키지로 가져오면 이런 정책을 지키기 좋습니다.

3. **빌드 도구와 잘 통합 (Vite, Webpack 등)**

   * `@fontsource` 패키지는 `@font-face`+폰트 파일 구조가 이미 빌드 도구 친화적으로 세팅되어 있어서,
   * Vite 번들 시에도 자동으로 폰트 파일을 asset으로 잘 처리해줍니다. ([fontsource.org][3])

4. **퍼포먼스와 캐싱에 대한 완전한 통제**

   * 필요하면 `preload`, `cache-control` 헤더, split 등 **내 서버 기준으로 최적화** 가능.
   * 패키지 자체도 JS 번들에 거의 부담이 없는 수준(몇백 byte 수준의 css import 코드만)입니다. ([bundlephobia.com][5])

---

## 3. Material Icons vs Material Symbols vs SVG 패키지 🧩

헷갈리기 쉬운 포인트라 정리해두면 강의 자료로도 좋습니다.

### 3-1. Material Icons (클래식 아이콘 폰트)

* 예전부터 있던 **클래식 아이콘 세트**

* 폰트 기반이고, 보통

  ```html
  <span class="material-icons">home</span>
  ```

  이런 식으로 사용합니다. ([Google for Developers][6])

* 굵기/스타일은 사실상 고정(Regular), variable font 아님. ([무찌르자 단팥빵][7])

### 3-2. Material Symbols (새로운 variable 아이콘 폰트)

* 2022년 이후 나온 **새로운 variable 아이콘 폰트 세트**
* `FILL`, `wght`, `GRAD`, `opsz` 같은 **폰트 variation 설정**으로 채우기/두께/크기 등을 조절 가능. ([fontsource.org][8])
* Fontsource도 `@fontsource/material-symbols-*` 패키지 형태로 지원합니다. ([fontsource.org][8])

### 3-3. SVG 기반 아이콘(@mui/icons-material 등)과의 차이

* `@mui/icons-material` 처럼 **아이콘당 React 컴포넌트**가 있는 패키지는 **SVG 요소**를 렌더링합니다.
* `@fontsource/material-icons`는 **“폰트 하나 + CSS”만 제공**합니다.
* 즉:

  * SVG 패키지: `<HomeIcon />` 같은 컴포넌트
  * Fontsource: `<span className="material-icons">home</span>`처럼 텍스트 기반

---

## 4. 설치 방법 🛠 (Vite + React 기준)

### 4-1. 패키지 설치

```bash
npm install @fontsource/material-icons
# 또는
yarn add @fontsource/material-icons
pnpm add @fontsource/material-icons
```

Fontsource 공식 문서에 위와 같은 설치 예시가 그대로 있습니다. ([fontsource.org][3])

### 4-2. 엔트리에서 import

Vite + React라면 보통 `src/main.tsx` 또는 `src/main.jsx`에서:

```ts
// main.tsx 또는 main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// 👇 아이콘 폰트 import
import '@fontsource/material-icons';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

이 한 줄로:

* `@font-face` 규칙이 글로벌 CSS에 추가되고
* 실제 `.woff2`/`.woff` 파일은 Vite가 asset으로 번들에 포함시킵니다. ([fontsource.org][3])

---

## 5. CSS 클래스 세팅하기 (중요한 함정 ⚠️)

Google의 공식 CSS에는 보통 이런 식의 `.material-icons` 클래스가 정의돼 있습니다 (요약): ([Google for Developers][6])

```css
.material-icons {
  font-family: 'Material Icons';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;
  line-height: 1;
  letter-spacing: normal;
  text-transform: none;
  display: inline-block;
  white-space: nowrap;
  word-wrap: normal;
  direction: ltr;
  -webkit-font-feature-settings: 'liga';
  -webkit-font-smoothing: antialiased;
}
```

하지만 `@fontsource/material-icons` 패키지에는 과거/버전에 따라 이 클래스가 **자동으로 포함되지 않는 이슈**가 있었습니다. 그래서 다음과 같은 GitHub 이슈에서 “`<span class="material-icons">face</span>`가 안 나온다”는 얘기가 나왔죠. ([GitHub][9])

> 👉 **실무 팁**: Fontsource를 쓸 때는 **global CSS에 위와 비슷한 `.material-icons` 클래스를 직접 정의**해 주는 걸 추천합니다.
> (폰트 이름 `font-family: 'Material Icons';`만 정확히 맞추면 됩니다.)

Vite + React 기준으로 `src/index.css` 또는 `src/global.css` 같은 곳에 위 클래스를 추가하면 됩니다.

---

## 6. 실제 사용 예시 (React 컴포넌트) 🧪

### 6-1. 가장 단순한 사용

```tsx
// App.tsx
function App() {
  return (
    <div style={{ padding: 24 }}>
      <h1>Material Icons Demo</h1>

      <p>
        기본 아이콘: <span className="material-icons">home</span>
      </p>
      <p>
        알림: <span className="material-icons">notifications</span>
      </p>
      <p>
        즐겨찾기: <span className="material-icons">favorite</span>
      </p>
    </div>
  );
}

export default App;
```

한국 블로그 예시에서도 똑같이 `className="material-icons"` + 아이콘 이름으로 사용하는 걸 보여줍니다. ([Release Center][10])

### 6-2. 재사용 가능한 `Icon` 컴포넌트로 감싸기

```tsx
// src/components/Icon.tsx
import type { FC } from 'react';

interface IconProps {
  name: string;
  size?: number;
  color?: string;
}

export const Icon: FC<IconProps> = ({ name, size = 24, color = 'inherit' }) => {
  return (
    <span
      className="material-icons"
      style={{ fontSize: size, color }}
      aria-hidden="true"
    >
      {name}
    </span>
  );
};
```

```tsx
// App.tsx
import { Icon } from './components/Icon';

function App() {
  return (
    <div style={{ padding: 24 }}>
      <h1>Material Icons Wrapper</h1>
      <Icon name="home" />
      <Icon name="search" size={30} color="#2563eb" />
      <Icon name="favorite" size={32} color="crimson" />
    </div>
  );
}
```

이 패턴은 실제 블로그 예시에서도 거의 동일하게 사용되고 있습니다. ([Release Center][10])

---

## 7. Round / Outlined / Sharp / Two-Tone 아이콘 사용 🎨

Material Icons는 기본 세트 외에도 여러 변형 세트가 있습니다: ([fontsource.org][11])

* `@fontsource/material-icons-round`
* `@fontsource/material-icons-outlined`
* `@fontsource/material-icons-sharp`
* `@fontsource/material-icons-two-tone`

각 패키지는 각각의 아이콘 스타일을 위한 폰트를 제공합니다.

예를 들어 **Outlined 버전**을 쓰고 싶다면:

```bash
npm install @fontsource/material-icons-outlined
```

```ts
// main.tsx
import '@fontsource/material-icons-outlined';
```

그리고 CSS:

```css
.material-icons-outlined {
  font-family: 'Material Icons Outlined';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;
  line-height: 1;
  display: inline-block;
  /* ... 필요시 기타 속성 추가 */
}
```

사용:

```tsx
<span className="material-icons-outlined">home</span>
```

이렇게 하면 기존 `home` 아이콘이 **둥근 외곽선(outlined)** 스타일로 렌더링됩니다.

---

## 8. 고급 사용법: 직접 `@font-face` 제어하기 🧬

Fontsource 문서를 보면, 단순 import 외에 **Advanced** 섹션에서 `@font-face`를 직접 컨트롤하는 방식도 소개합니다. ([fontsource.org][3])

예를 들어, 글로벌 CSS에 직접:

```css
/* material-icons-latin-400-normal (요약 버전) */
@font-face {
  font-family: 'Material Icons';
  font-style: normal;
  font-display: swap;
  font-weight: 400;
  src:
    url('@fontsource/material-icons/files/material-icons-latin-400-normal.woff2')
      format('woff2'),
    url('@fontsource/material-icons/files/material-icons-latin-400-normal.woff')
      format('woff');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+2000-206F, U+20AC, U+FFFD;
}
```

그리고 `.material-icons` 클래스 정의를 붙이면 됩니다. 이 방식은:

* **Vite 기반 프레임워크에서 추천**되는 고급 패턴으로,
* `@font-face`를 더 세밀하게 제어하고 싶을 때 유용합니다 (예: `unicode-range` 조절, 여러 weight, custom font-display 등). ([fontsource.org][3])

---

## 9. 자주 하는 실수 & 디버깅 포인트 🧯

1. **`font-family` 이름 오타**

   * 반드시 `'Material Icons'`와 정확히 일치해야 합니다. (중간 공백 포함)
2. **`.material-icons` 클래스를 안 만들었을 때**

   * Fontsource가 자동으로 넣어줄 거라고 기대했다가 아이콘이 안 보이는 경우가 많습니다. 직접 CSS에 정의해 주세요. ([GitHub][9])
3. **아이콘 이름 오타**

   * `<span class="material-icons">home</span>`처럼 정확한 아이콘 이름을 넣어야 합니다.
   * 전체 리스트는 Google Icons 페이지(검색/필터 지원)에서 확인할 수 있습니다. ([Google for Developers][6])
4. **CDN + Fontsource 혼용**

   * `<link href="https://fonts.googleapis.com/icon?family=Material+Icons" ...>`랑
   * `@fontsource/material-icons`를 같이 쓰면, 어떤 쪽이 실제로 로드되는지 헷갈리니 하나만 쓰는 게 좋습니다.

---

## 10. 정리 📝

짧게 정리하면:

* `@fontsource/material-icons` = **Google Material Icons를 자기 서버에서 서빙할 수 있게 해주는 NPM 패키지**
* Vite + React에서는:

  1. `npm install @fontsource/material-icons`
  2. `main.tsx`에서 `import '@fontsource/material-icons';`
  3. 글로벌 CSS에 `.material-icons` 클래스(폰트 패밀리 포함) 정의
  4. `<span className="material-icons">home</span>` 식으로 사용
* Round/Outlined/Sharp/Two-Tone 등은 각각 별도 패키지 (`@fontsource/material-icons-outlined` 등)로 제공
* SVG 기반 아이콘 패키지와는 달리 **텍스트/폰트 기반**이라서, 레거시 코드나 폰트 아이콘 스타일의 UI에 특히 잘 맞음

---



[1]: https://fontsource.org/fonts/material-icons?utm_source=chatgpt.com "Material Icons"
[2]: https://unpkg.com/%40fontsource/material-icons/?utm_source=chatgpt.com "fontsource/material-icons"
[3]: https://fontsource.org/fonts/material-icons/install?utm_source=chatgpt.com "Material Icons | Install"
[4]: https://stackoverflow.com/questions/38412991/how-to-use-material-fonts-and-icons-locally-with-material-ui?utm_source=chatgpt.com "How to use material fonts and icons locally with material-ui?"
[5]: https://bundlephobia.com/package/%40fontsource/material-icons?utm_source=chatgpt.com "@fontsource/material-icons v5.2.7 ❘ ..."
[6]: https://developers.google.com/fonts/docs/material_icons?utm_source=chatgpt.com "Material Icons Guide | Google Fonts"
[7]: https://dimorin.tistory.com/m/79?utm_source=chatgpt.com "Material Symbols, Material Icons 사용 방법 - 무찌르자 단팥빵"
[8]: https://fontsource.org/docs/getting-started/material-symbols?utm_source=chatgpt.com "Material Symbols | Documentation"
[9]: https://github.com/fontsource/fontsource/issues/976?utm_source=chatgpt.com "material-icons is missing the additional CSS class again"
[10]: https://taisou.tistory.com/887?utm_source=chatgpt.com "[TS]material-Icon 추가하여 사용하기"
[11]: https://fontsource.org/docs/getting-started/material-icons?utm_source=chatgpt.com "Material Icons | Documentation"
