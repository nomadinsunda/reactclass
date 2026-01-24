# 🚀 PostCSS는 왜 `postcss.config.js` 파일을 필요로 할까?

Vite 기반 React 프로젝트에서 Tailwind CSS를 사용할 때 종종 이런 파일을 보시죠:

```
postcss.config.js
tailwind.config.js
```

많은 분들이 “Vite가 CSS를 잘 처리해주는데, PostCSS 설정 파일을 꼭 둬야 하나?”라고 의문을 가집니다.
정답은 **Yes, 필요합니다.**

그 이유는 **PostCSS가 동작하는 구조적 원리**와 **Vite가 PostCSS를 내부적으로 어떻게 호출하는가**를 이해하면 명확해집니다.

---

# ✅ 1. PostCSS는 ‘플러그인 실행기(Plugin Runner)’ 이다

PostCSS 자체는 단순한 CSS 파서(parser) + 트랜스포머(transformer)입니다.
**아무 기능도 하지 않습니다.**
Tailwind, Autoprefixer 같은 기능은 모두 **PostCSS 플러그인**입니다.

즉:

```
PostCSS = 엔진
plugin = 실제 기능(Tailwind, Autoprefixer 등)
```

❗ 그렇기 때문에 **PostCSS에게 어떤 플러그인을 어떤 순서로 실행할지 알려줘야 합니다.**
그 역할이 바로 **postcss.config.js**입니다.

---

# ✅ 2. Tailwind CSS는 PostCSS 플러그인이다

Tailwind는 PostCSS 플러그인으로 구현되어 있습니다:

```js
// postcss 플러그인 형태
module.exports = function tailwindcss() { ... }
```

따라서 Tailwind를 사용한다면 반드시 PostCSS가 필요하며, Tailwind를 PostCSS 플러그인 목록에 등록해줘야 합니다.

즉,

* Tailwind 사용 = PostCSS 필요
* PostCSS 실행 = `postcss.config.js` 필요

---

# ✅ 3. Vite는 PostCSS를 자동으로 실행하지만, 설정은 사용자 몫이다

Vite는 다음과 같은 역할을 합니다:

* `index.css` → PostCSS 로더에 자동 전달
* PostCSS 설정 파일(`postcss.config.js`)이 있으면 읽어서 플러그인 적용
* 없으면 내부 기본 설정만 사용

📌 **핵심**
Vite는 PostCSS를 “실행”만 해줄 뿐,
“어떤 플러그인을 적용할지”는 **사용자가 지정**해야 합니다.

따라서 Tailwind를 쓰려면 반드시:

```
postcss.config.js → tailwindcss, autoprefixer 등록
```

이 구조가 필요합니다.

---

# ✅ 4. Tailwind는 반드시 PostCSS를 통해 컴파일된다

Tailwind의 핵심 기능은 다음과 같은 PostCSS 트랜스폼입니다.

* `@tailwind base;`
* `@tailwind components;`
* `@tailwind utilities;`
* JIT 모드로 class 스캔 후 CSS 자동 생성
* 필요없는 CSS 정리(purge)

이러한 기능은 **PostCSS 파이프라인을 거쳐야 실행됩니다.**

즉 Tailwind가 동작하려면:

```
postcss → tailwindcss plugin → autoprefixer → 최종 CSS 출력
```

이 파이프라인이 필요하며
그 파이프라인을 **구성하는 곳이 postcss.config.js입니다.**

---

# ✅ 5. autoprefixer 도 PostCSS 플러그인이다

브라우저 벤더 프리픽스를 자동으로 붙여주는 `autoprefixer`도 PostCSS 플러그인입니다.

예:

```css
display: flex;
```

↓ autoprefixer ↓

```css
display: -webkit-box;
display: -ms-flexbox;
display: flex;
```

Tailwind 공식 템플릿이 autoprefixer를 기본 포함하는 이유도 같습니다.

---

# 📌 결론: postcss.config.js가 필요한 최종 이유

정말 한 문장으로 요약하면 다음과 같습니다.

> **Tailwind CSS는 PostCSS 플러그인이므로,
> PostCSS가 Tailwind(및 autoprefixer)를 적용하도록 알려주기 위해
> 반드시 `postcss.config.js` 파일이 필요하다.**

이게 없으면:

* Tailwind 지시자(`@tailwind`)를 컴파일하지 못하고
* Tailwind 유틸리티 CSS도 생성되지 않으며
* autoprefixer 같은 기능도 작동하지 않습니다.

---

# 📁 실제 작성해야 하는 기본 파일

### 📌 `postcss.config.js`

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### 📌 `tailwind.config.js`

```js
export default {
  content: ["./index.html", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

---

# ✨ 부가 설명:

## ❓ “Vite는 이미 CSS를 잘 처리하는데 PostCSS 파일이 꼭 필요해요?”

→ Vite가 CSS를 해석하는 것은 **CSS Modules / CSS import** 처리이고,
CSS ‘변환(Transformation)’은 PostCSS가 담당합니다.

따라서 Tailwind처럼 변환이 필요한 CSS 프레임워크는
반드시 PostCSS 설정이 필요합니다.

