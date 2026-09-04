Tailwind CSS에서 **content**는 한마디로 **"Tailwind가 감시할 파일들의 경로 리스트"** 를 의미합니다.

JIT 엔진은 똑똑하지만 모든 파일을 뒤져볼 만큼 한가하진 않거든요. 그래서 설정 파일(`tailwind.config.js`)의 `content` 섹션을 통해 **"여기 있는 파일들만 읽어서 내가 어떤 클래스를 CSS로 만들지 결정해줘"** 라고 길을 안내해 주는 것이라 보시면 됩니다.

---

### 🛠️ 어떻게 작동하나요?

JIT 엔진은 `content`에 설정된 경로의 파일들을 스캔하여, 텍스트 뭉치 안에서 Tailwind 클래스처럼 보이는 문자열(예: `bg-blue-500`, `flex`)을 찾아냅니다.

1.  **파일 스캔:** `content` 경로에 있는 모든 HTML, JS, TSX 파일을 읽습니다.
2.  **클래스 추출:** 파일 내부 코드에서 `text-center`, `p-4` 같은 클래스명을 추출합니다.
3.  **CSS 생성:** 추출된 클래스에 해당하는 실제 CSS 코드만 생성하여 최종 결과물에 넣습니다.



---

### 📝 설정 예시 (`tailwind.config.js`)

보통 프로젝트의 루트 폴더에 있는 설정 파일에서 다음과 같이 작성합니다.

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}", // src 폴더 안의 모든 JS, TS, React 파일들
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

---

### ⚠️ 주의할 점: "동적 클래스명"
JIT 엔진은 파일을 **정적으로 분석**합니다. 즉, 소스 코드 안에 클래스 이름이 **완전한 문자열**로 적혀 있어야 합니다.

* **✅ 좋은 예 (작동함):**
    `<div className="text-red-500"></div>`
* **❌ 안 좋은 예 (작동 안 함):**
    `<div className={"text-" + colorName + "-500"}></div>`
    *(JIT 엔진은 `colorName`이 나중에 무엇이 될지 모르기 때문에 해당 CSS를 생성하지 않습니다.)*

