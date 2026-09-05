# PART 3. daisyUI Color & Theme 시스템

## 1. 이번 PART에서 배울 내용

PART 2에서 다음과 같은 Class를 사용했습니다.

```html
<button class="btn btn-primary">
  저장
</button>

<div class="alert alert-success">
  저장되었습니다.
</div>
```

여기서 중요한 질문이 있습니다.

```text
primary는 무슨 색인가?

success는 왜 초록색처럼 보이는가?

dark Theme으로 변경하면
같은 btn-primary의 색상은 왜 달라지는가?
```

이 질문을 이해하려면 daisyUI의 **Theme 시스템**을 이해해야 합니다.

이번 PART에서는 다음 내용을 학습합니다.

* Semantic Color란?
* `primary`, `secondary`, `accent`의 의미
* `base-*` Color의 역할
* Content Color란?
* Component와 Theme의 연결
* Built-in Theme
* `data-theme`
* Theme 전환
* Custom Theme
* React에서 Theme 변경
* Tailwind CSS Utility와 Theme Color 조합

이번 PART의 핵심은 다음과 같습니다.

> **daisyUI에서 `primary`는 특정 색상 자체가 아니라 Theme이 정의하는 의미적 역할(Semantic Role)입니다.**

---

# 2. `primary`는 파란색인가?

다음 Button을 보겠습니다.

```html
<button class="btn btn-primary">
  저장
</button>
```

많은 초보자가 다음처럼 생각합니다.

```text
btn-primary
     ↓
파란색 Button
```

하지만 정확하지 않습니다.

`primary`의 핵심 의미는:

```text
primary
     ↓
이 UI에서 가장 중요한
대표 색상 역할
```

입니다.

실제 색상은 **현재 적용된 Theme이 결정합니다.**

```text
              btn-primary
                    │
                    ▼
                 primary
                    │
                    ▼
                  Theme
             ┌──────┴──────┐
             │             │
           light         다른 Theme
             │             │
          #xxxxxx       #yyyyyy
```

따라서 같은:

```html
<button class="btn btn-primary">
```

라도 Theme에 따라 실제 표현 색상이 달라질 수 있습니다.

---

# 3. Semantic Color란?

Semantic은 **의미**를 뜻합니다.

일반적인 Tailwind CSS에서는 다음처럼 실제 색상 계열을 직접 지정할 수 있습니다.

```html
<button class="bg-blue-600">
  저장
</button>
```

여기서는:

```text
blue
```

라는 실제 색상 계열을 직접 선택했습니다.

반면 daisyUI에서는:

```html
<button class="btn btn-primary">
  저장
</button>
```

처럼 작성합니다.

여기서:

```text
primary
```

는 색상 이름이라기보다 **역할의 이름**입니다.

비교하면:

```text
고정 Palette 중심

blue-600
red-500
green-600

        vs

Semantic Role 중심

primary
secondary
success
error
```

Semantic Color를 사용하면 UI 코드가 특정 색상에 덜 의존합니다.

---

# 4. 왜 Semantic Color를 사용하는가?

예를 들어 쇼핑몰의 대표 색상이 파란색이라고 하겠습니다.

```text
Brand Color
    ↓
Blue
```

여러 곳에서 직접:

```html
bg-blue-600
text-blue-600
border-blue-600
```

를 사용했다고 하겠습니다.

나중에 브랜드 색상을 보라색으로 바꾸려면 많은 코드를 수정해야 할 수 있습니다.

반면:

```text
Brand Color
    ↓
primary
```

라는 의미를 사용하면:

```html
<button class="btn btn-primary">
```

코드는 그대로 두고 Theme의 `primary` 정의를 변경할 수 있습니다.

```text
Before

primary → Blue


After

primary → Purple
```

Component 코드는:

```html
btn-primary
```

그대로입니다.

이것이 Semantic Color를 사용하는 가장 큰 이유 중 하나입니다.

---

# 5. daisyUI의 주요 Semantic Color

daisyUI Theme에서는 여러 의미 기반 Color를 사용합니다.

대표적인 Color는 다음과 같습니다.

```text
primary
secondary
accent
neutral

info
success
warning
error
```

역할별로 나누어 생각하면 이해하기 쉽습니다.

```text
Brand / UI Color
────────────────────
primary
secondary
accent
neutral


State / Feedback Color
────────────────────
info
success
warning
error
```

---

# 6. `primary`

`primary`는 UI에서 가장 중요한 대표 색상입니다.

예:

```html
<button class="btn btn-primary">
  구매하기
</button>
```

일반적으로:

```text
주요 Action
CTA
브랜드 대표 요소
선택된 주요 기능
```

등에 사용할 수 있습니다.

예:

```text
[ 구매하기 ]
[ 가입하기 ]
[ 저장하기 ]
```

하지만 모든 Button에 `primary`를 사용하면 무엇이 중요한지 구분하기 어려워집니다.

---

# 7. `secondary`

`secondary`는 `primary`를 보조하는 두 번째 주요 색상 역할입니다.

```html
<button class="btn btn-secondary">
  자세히 보기
</button>
```

개념적으로:

```text
Primary
   ↓
가장 중요한 Action

Secondary
   ↓
보조 Action
```

처럼 사용할 수 있습니다.

다만 실제 디자인에서 `primary`와 `secondary`의 사용 기준은 프로젝트의 Design System에 따라 결정해야 합니다.

---

# 8. `accent`

`accent`는 특정 요소를 눈에 띄게 강조하고 싶을 때 사용할 수 있는 의미적 색상입니다.

```html
<span class="badge badge-accent">
  NEW
</span>
```

예:

```text
NEW
HOT
추천
특별 기능
```

등을 강조할 때 활용할 수 있습니다.

하지만 `accent`의 정확한 사용 목적 역시 프로젝트의 디자인 규칙에 따라 정하는 것이 좋습니다.

---

# 9. `neutral`

`neutral`은 강한 의미 색상이 필요하지 않은 중립적인 UI에 활용할 수 있습니다.

```html
<button class="btn btn-neutral">
  취소
</button>
```

예:

```text
보조 UI
중립적 Action
Navigation
강조가 필요하지 않은 영역
```

등에 사용할 수 있습니다.

---

# 10. 상태와 Feedback Color

다음 네 가지는 사용자에게 상태나 결과를 전달할 때 매우 유용합니다.

```text
info
success
warning
error
```

예:

```html
<div class="alert alert-info">
  새로운 업데이트가 있습니다.
</div>

<div class="alert alert-success">
  저장되었습니다.
</div>

<div class="alert alert-warning">
  입력 내용을 확인하세요.
</div>

<div class="alert alert-error">
  저장에 실패했습니다.
</div>
```

개념적으로:

```text
info
→ 정보

success
→ 성공

warning
→ 주의

error
→ 오류
```

입니다.

여기서 중요한 것은 이것들도 **고정 색상 이름이 아니라 Semantic Role**이라는 것입니다.

---

# 11. Base Color

애플리케이션에는 Button 같은 강조 색상뿐 아니라 **화면의 기본 표면(Surface)**을 표현하는 색상이 필요합니다.

daisyUI에서는 대표적으로:

```text
base-100
base-200
base-300
base-content
```

를 사용합니다.

개념적으로:

```text
base-100
→ 주요 Surface

base-200
→ 한 단계 구분되는 Surface

base-300
→ 더 강하게 구분되는 Surface

base-content
→ Base Surface 위의 Content
```

로 이해할 수 있습니다.

예:

```html
<div class="bg-base-100 text-base-content">
  Content
</div>
```

---

# 12. `base-100`, `base-200`, `base-300`

예를 들어 화면을 다음처럼 구성할 수 있습니다.

```text
Application Background
────────────────────────────
base-200

    Card
    ┌──────────────────┐
    │    base-100      │
    │                  │
    │   Content        │
    └──────────────────┘

Border / 구분 영역
────────────────────────────
base-300
```

예:

```html
<body class="bg-base-200">

  <div class="card bg-base-100">
    ...
  </div>

</body>
```

Theme을 변경하면 이 Surface 계층의 색상도 Theme에 맞게 변경됩니다.

---

# 13. Content Color란?

배경색만 결정하면 충분하지 않습니다.

예를 들어:

```text
Primary Background
```

위에는 읽을 수 있는 Text Color가 필요합니다.

그래서 Theme에는 Color와 함께 Content Color 개념이 있습니다.

대표적으로:

```text
primary
primary-content

secondary
secondary-content

accent
accent-content

neutral
neutral-content

base-100
base-content
```

처럼 생각할 수 있습니다.

관계는:

```text
primary
     ↓
Background / Surface

primary-content
     ↓
그 위에 표시되는 Content
```

입니다.

---

# 14. Content Color가 중요한 이유

예를 들어 배경이 어두운 보라색이라고 하겠습니다.

```text
████████████████
   Primary
████████████████
```

Text도 어두우면 읽기 어렵습니다.

Theme은:

```text
primary
        +
primary-content
```

를 함께 정의하여 적절한 대비를 만들 수 있습니다.

개념적으로:

```text
┌─────────────────────┐
│      primary        │
│                     │
│  primary-content    │
│                     │
└─────────────────────┘
```

따라서 Custom Theme을 만들 때는 단순히 예쁜 색상을 선택하는 것뿐 아니라 **Content의 가독성과 Contrast**도 중요합니다.

---

# 15. Theme이란?

Theme은 애플리케이션의 Semantic Color와 여러 디자인 값을 하나의 체계로 정의한 것입니다.

쉽게 생각하면:

```text
Theme
  │
  ├─ primary
  ├─ secondary
  ├─ accent
  ├─ neutral
  │
  ├─ base-100
  ├─ base-200
  ├─ base-300
  ├─ base-content
  │
  ├─ info
  ├─ success
  ├─ warning
  └─ error
```

그리고 Component가 이 값을 사용합니다.

```text
Theme
   ↓
Semantic Color
   ↓
Component
   ↓
Final UI
```

예:

```text
Theme
  ↓
primary
  ↓
btn-primary
  ↓
Primary Button
```

---

# 16. Component와 Theme의 관계

다음 코드를 보겠습니다.

```html
<button class="btn btn-primary">
  저장
</button>
```

구조는 다음과 같습니다.

```text
btn
│
└─ Button Component

btn-primary
│
└─ primary Semantic Color 사용
             │
             ▼
           Theme
             │
             ▼
       실제 Color 결정
```

즉 `btn-primary`가 자체적으로 특정 HEX 색상을 의미한다고 생각하면 안 됩니다.

핵심 관계는:

```text
Component
     ↓
Modifier / Variant
     ↓
Semantic Color
     ↓
Theme
     ↓
실제 화면 표현
```

입니다.

---

# 17. Theme을 사용하면 무엇이 좋은가?

Theme을 사용하면 Component 코드를 크게 변경하지 않고 전체 UI 분위기를 바꿀 수 있습니다.

예:

```html
<button class="btn btn-primary">
  저장
</button>

<div class="alert alert-success">
  저장되었습니다.
</div>
```

코드는 그대로 두고 Theme만 변경합니다.

```text
Light Theme

btn-primary
alert-success

        ↓ Theme 변경

Dark Theme

btn-primary
alert-success
```

Component의 의미는 유지하면서 실제 색상 표현이 Theme에 맞게 변경됩니다.

---

# 18. Built-in Theme

daisyUI는 여러 Built-in Theme을 제공합니다.

예:

```text
light
dark
cupcake
bumblebee
emerald
corporate
synthwave
retro
```

각 Theme은 동일한 Semantic Color 이름을 서로 다른 실제 색상 값으로 정의할 수 있습니다.

예:

```text
                  primary

light                │
                     ├── Color A

dark                 │
                     ├── Color B

cupcake              │
                     └── Color C
```

하지만 Component 코드는 계속:

```html
<button class="btn btn-primary">
```

입니다.

---

# 19. `data-theme`

Theme은 `data-theme` 속성을 통해 적용할 수 있습니다.

예:

```html
<html data-theme="light">
```

Dark Theme:

```html
<html data-theme="dark">
```

Cupcake Theme:

```html
<html data-theme="cupcake">
```

구조는:

```text
data-theme="dark"
        ↓
daisyUI Theme 선택
        ↓
Semantic Color 변경
        ↓
Theme 기반 Component 표현 변경
```

입니다.

---

# 20. Theme은 특정 영역에도 적용할 수 있다

`data-theme`을 반드시 `<html>`에만 지정해야 하는 것은 아닙니다.

예:

```html
<div data-theme="light">
  <button class="btn btn-primary">
    Light
  </button>
</div>

<div data-theme="dark">
  <button class="btn btn-primary">
    Dark
  </button>
</div>
```

같은 페이지 안에서도 서로 다른 Theme 영역을 구성할 수 있습니다.

```text
Page

┌───────────────────────┐
│ data-theme="light"    │
│                       │
│ [ Primary Button ]    │
└───────────────────────┘

┌───────────────────────┐
│ data-theme="dark"     │
│                       │
│ [ Primary Button ]    │
└───────────────────────┘
```

Theme은 해당 Element와 그 하위 영역에 영향을 줄 수 있습니다.

---

# 21. `data-theme="dark"`와 Tailwind `dark:`는 다르다

이 부분은 매우 중요합니다.

daisyUI:

```html
<html data-theme="dark">
```

Tailwind CSS:

```html
<div class="dark:bg-gray-900">
```

둘은 같은 기능이라고 생각하면 안 됩니다.

```text
data-theme="dark"
────────────────────
daisyUI Theme 선택


dark:
────────────────────
Tailwind CSS의
dark mode Variant
```

따라서:

> **daisyUI Theme 시스템과 Tailwind CSS의 `dark:` Variant는 구분해서 이해해야 합니다.**

---

# 22. Theme 기반 Utility Class

Theme의 Semantic Color는 Component에서만 사용하는 것이 아닙니다.

예:

```html
<div class="bg-primary text-primary-content">
  Primary Area
</div>
```

또는:

```html
<div class="bg-base-100 text-base-content">
  Content
</div>
```

처럼 사용할 수 있습니다.

즉:

```text
Semantic Color
       │
       ├─ Component
       │    btn-primary
       │
       └─ Utility
            bg-primary
            text-primary-content
```

라는 두 가지 활용 방식이 있습니다.

---

# 23. 고정 Palette와 Theme Color 비교

다음 두 코드를 비교해 보겠습니다.

### 고정 Palette 사용

```html
<div class="bg-blue-600 text-white">
  Save
</div>
```

### Theme Semantic Color 사용

```html
<div class="bg-primary text-primary-content">
  Save
</div>
```

차이는:

```text
bg-blue-600
     ↓
특정 Palette Color


bg-primary
     ↓
Theme의 primary
```

입니다.

Theme을 적극적으로 사용하는 애플리케이션에서는 **디자인 시스템에 속하는 색상은 Semantic Color 중심으로 사용하는 것**이 유지보수에 유리합니다.

반대로 모든 색상을 반드시 Semantic Color로 바꿔야 한다는 뜻은 아닙니다.

일회성 장식이나 Theme과 독립적인 표현에는 Tailwind의 일반 Palette가 더 적절할 수도 있습니다.

---

# 24. Custom Theme이 필요한 이유

Built-in Theme만으로 프로젝트의 브랜드 디자인을 모두 표현할 수 있는 것은 아닙니다.

예를 들어 회사의 Design System이:

```text
Brand Primary
→ #2563EB

Brand Secondary
→ #7C3AED

Brand Accent
→ #10B981
```

라고 하겠습니다.

이 값을 daisyUI의 Semantic Color에 연결하면:

```text
Brand Design System
        ↓
daisyUI Custom Theme
        ↓
primary
secondary
accent
        ↓
모든 Theme 기반 Component
```

로 연결할 수 있습니다.

---

# 25. Custom Theme 정의

Tailwind CSS v4 + 최신 daisyUI 구성에서는 CSS에서 Custom Theme을 정의할 수 있습니다.

예:

```css
@plugin "daisyui/theme" {
  name: "myapp";
  default: true;
  color-scheme: light;

  --color-primary: #2563eb;
  --color-primary-content: #ffffff;

  --color-secondary: #7c3aed;
  --color-secondary-content: #ffffff;

  --color-accent: #10b981;
  --color-accent-content: #ffffff;

  --color-neutral: #1f2937;
  --color-neutral-content: #ffffff;

  --color-base-100: #ffffff;
  --color-base-200: #f3f4f6;
  --color-base-300: #e5e7eb;
  --color-base-content: #111827;
}
```

그리고:

```html
<html data-theme="myapp">
```

으로 적용할 수 있습니다.

---

# 26. Custom Theme의 핵심 구조

Custom Theme의 핵심은 단순합니다.

```text
myapp Theme
     │
     ├─ primary
     │    └─ #2563EB
     │
     ├─ secondary
     │    └─ #7C3AED
     │
     ├─ accent
     │    └─ #10B981
     │
     └─ base-100
          └─ #FFFFFF
```

그리고 Component는:

```html
<button class="btn btn-primary">
```

그대로 사용합니다.

즉:

```text
Theme 값 변경
      ↓
Component 코드 유지
      ↓
전체 디자인 변경
```

이라는 구조입니다.

---

# 27. `default`

Custom Theme에서:

```css
default: true;
```

를 지정하면 해당 Theme을 디폴트 Theme으로 사용할 수 있습니다.

예:

```css
@plugin "daisyui/theme" {
  name: "myapp";
  default: true;
}
```

개념적으로:

```text
Application 시작
       ↓
myapp Theme
       ↓
디폴트 Theme
```

입니다.

---

# 28. `color-scheme`

다음 설정도 볼 수 있습니다.

```css
color-scheme: light;
```

또는:

```css
color-scheme: dark;
```

`color-scheme`은 브라우저가 제공하는 Form Control, Scrollbar 등의 기본 UI를 어떤 색상 체계에 맞춰 표현할지 판단하는 데 영향을 줄 수 있습니다.

즉 단순히:

```text
color-scheme: dark
=
daisyUI Dark Theme
```

라고 생각하면 안 됩니다.

Theme 이름과 `color-scheme`은 서로 다른 개념입니다.

---

# 29. Custom Theme에서 Content Color도 함께 생각해야 한다

다음처럼 `primary`만 결정하면 끝이라고 생각하기 쉽습니다.

```css
--color-primary: #2563eb;
```

하지만 실제 Component에는 그 위에 Text나 Icon이 올라갑니다.

따라서:

```css
--color-primary: #2563eb;
--color-primary-content: #ffffff;
```

처럼 함께 설계하는 것이 중요합니다.

```text
Primary Surface
██████████████████
    Save
██████████████████
        ↑
primary-content
```

Theme 설계는 **색상 선택 + Contrast 설계**입니다.

---

# 30. React에서 Theme 변경하기

React에서는 State를 이용해 Theme을 변경할 수 있습니다.

```jsx
import { useEffect, useState } from 'react'

function ThemeSwitcher() {
  const [theme, setTheme] = useState('light')

  useEffect(() => {
    document.documentElement.setAttribute(
      'data-theme',
      theme
    )
  }, [theme])

  return (
    <select
      className="select"
      value={theme}
      onChange={(e) => setTheme(e.target.value)}
    >
      <option value="light">Light</option>
      <option value="dark">Dark</option>
      <option value="cupcake">Cupcake</option>
    </select>
  )
}
```

흐름은:

```text
<select>
    ↓
onChange
    ↓
setTheme("dark")
    ↓
React State 변경
    ↓
useEffect
    ↓
<html data-theme="dark">
    ↓
daisyUI Theme 변경
```

입니다.

---

# 31. Theme Toggle 예제

Button으로 Theme을 전환할 수도 있습니다.

```jsx
function ThemeToggle() {
  const [theme, setTheme] = useState('light')

  const toggleTheme = () => {
    setTheme(
      theme === 'light'
        ? 'dark'
        : 'light'
    )
  }

  useEffect(() => {
    document.documentElement.dataset.theme = theme
  }, [theme])

  return (
    <button
      className="btn btn-ghost"
      onClick={toggleTheme}
    >
      {theme === 'light' ? 'Dark' : 'Light'}
    </button>
  )
}
```

핵심은 Button의 색상을 직접 하나씩 바꾸는 것이 아닙니다.

```text
잘못된 사고방식

Button 색 변경
Card 색 변경
Navbar 색 변경
Text 색 변경
...


Theme 방식

data-theme 변경
       ↓
Semantic Color 변경
       ↓
Theme 기반 UI가 함께 변경
```

입니다.

---

# 32. Theme 선택을 저장하려면?

사용자가 선택한 Theme을 새로고침 후에도 유지하고 싶을 수 있습니다.

이 경우 브라우저의 `localStorage` 등을 사용할 수 있습니다.

개념적인 흐름은:

```text
사용자 Theme 선택
       ↓
React State
       ↓
data-theme 변경
       ↓
localStorage 저장
       ↓
새로고침
       ↓
저장된 Theme 복원
```

입니다.

예:

```jsx
const [theme, setTheme] = useState(() => {
  return localStorage.getItem('theme') ?? 'light'
})

useEffect(() => {
  document.documentElement.dataset.theme = theme
  localStorage.setItem('theme', theme)
}, [theme])
```

---

# 33. 실전 예제 — Theme 기반 Dashboard

다음과 같은 Dashboard를 생각해 보겠습니다.

```jsx
function Dashboard() {
  return (
    <div className="min-h-screen bg-base-200 text-base-content">
      <nav className="navbar bg-base-100 shadow-sm">
        <div className="flex-1">
          <span className="text-xl font-bold">
            MyApp
          </span>
        </div>

        <button className="btn btn-primary">
          새 프로젝트
        </button>
      </nav>

      <main className="p-6">
        <div className="card bg-base-100 shadow-lg">
          <div className="card-body">
            <h2 className="card-title">
              Dashboard
            </h2>

            <div className="alert alert-success">
              정상적으로 연결되었습니다.
            </div>
          </div>
        </div>
      </main>
    </div>
  )
}
```

여기서 Theme 관련 Class를 찾아보면:

```text
bg-base-200
text-base-content

bg-base-100

btn-primary

alert-success
```

입니다.

이들은 모두 Theme의 Semantic Color와 연결되어 있습니다.

---

# 34. Dashboard에서 Theme을 바꾸면?

Component 코드는 그대로 둡니다.

```text
bg-base-200
bg-base-100
btn-primary
alert-success
```

그리고:

```html
<html data-theme="light">
```

를:

```html
<html data-theme="dark">
```

로 변경합니다.

흐름:

```text
              Application
                    │
                    ▼
              data-theme
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        light                dark
          │                   │
          └─────────┬─────────┘
                    ▼
             Semantic Color
                    ▼
        ┌───────────┼───────────┐
        ▼           ▼           ▼
    base-100     primary     success
        │           │           │
        └───────────┼───────────┘
                    ▼
               Component
                    ▼
                 Final UI
```

이것이 daisyUI Theme 시스템의 핵심입니다.

---

# 35. Theme 설계 시 흔한 실수

### 실수 1. `primary = blue`라고 외운다

잘못된 이해:

```text
primary = blue
success = green
error = red
```

더 정확한 이해:

```text
primary
→ 대표 역할

success
→ 성공 역할

error
→ 오류 역할

실제 색상
→ Theme이 결정
```

---

### 실수 2. 모든 색상을 Tailwind Palette로 직접 지정한다

예:

```html
<div class="bg-white">
<button class="bg-blue-600">
<span class="text-gray-900">
```

이 방식이 항상 잘못된 것은 아닙니다.

하지만 Theme 전환이 중요한 UI에서 모든 핵심 색상을 고정 Palette로 작성하면 Theme 시스템의 장점을 살리기 어렵습니다.

Theme 기반 영역에서는:

```html
bg-base-100
text-base-content
btn-primary
```

같은 Semantic Color를 우선 고려합니다.

---

### 실수 3. `data-theme="dark"`와 `dark:`를 같은 것으로 생각한다

다시 정리하면:

```text
data-theme="dark"
→ daisyUI Theme

dark:
→ Tailwind CSS Variant
```

입니다.

---

### 실수 4. Custom Theme에서 Contrast를 고려하지 않는다

예쁜 `primary` 색상을 만들었더라도 `primary-content`가 잘 보이지 않으면 좋은 Theme이 아닙니다.

```text
Color
+
Content Color
+
Contrast
=
사용 가능한 Theme
```

으로 생각해야 합니다.

---

# 36. Theme을 사용할 때 추천하는 사고방식

UI를 만들 때 바로 실제 색상부터 생각하지 않는 것이 좋습니다.

예를 들어:

```text
"이 Button을 파란색으로 해야지"
```

보다:

```text
"이 Button은 가장 중요한 Action이다"
        ↓
primary
```

라고 생각합니다.

오류 메시지도:

```text
"빨간색으로 해야지"
```

보다:

```text
"이것은 Error Feedback이다"
        ↓
error
```

라고 생각합니다.

즉:

```text
색상부터 결정
      X

의미부터 결정
      ↓
Semantic Color 선택
      ↓
Theme이 실제 색상 결정
```

이라는 사고방식이 중요합니다.

---

# 37. 전체 구조 한눈에 보기

```text
                     Application
                          │
                          ▼
                        Theme
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       ▼                  ▼                  ▼
   Brand Color        Base Color        State Color
       │                  │                  │
 primary            base-100             info
 secondary          base-200             success
 accent             base-300             warning
 neutral            base-content         error
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ▼
                  Semantic Color
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        Component       Utility      Content
             │            │            │
       btn-primary    bg-primary   text-base-content
             │            │            │
             └────────────┼────────────┘
                          ▼
                       Final UI
```

---

# 38. PART 3 핵심 정리

가장 중요한 관계는 다음입니다.

```text
Theme
   ↓
Semantic Color
   ↓
Component / Utility
   ↓
Final UI
```

`primary`는 특정 색상이 아닙니다.

```text
primary
≠ blue

primary
= UI에서 대표 역할을 담당하는 Semantic Color
```

그리고:

```text
btn-primary
       ↓
primary
       ↓
현재 Theme
       ↓
실제 Color
```

입니다.

Theme을 변경하면:

```text
data-theme 변경
       ↓
Semantic Color 값 변경
       ↓
Theme 기반 Component / Utility 변경
       ↓
Application UI 변경
```

이 일어납니다.

Custom Theme을 사용하면:

```text
Brand Design System
        ↓
Custom Theme
        ↓
Semantic Color
        ↓
daisyUI Component
        ↓
일관된 Application UI
```

로 연결할 수 있습니다.

> **daisyUI Theme의 핵심은 “색상을 직접 선택하는 것”보다 “색상에 의미를 부여하고 Theme이 실제 표현을 결정하게 하는 것”입니다.**

---

# 다음 PART

## PART 4. Button Component 완벽 이해

다음 PART에서는 지금까지 배운:

```text
Component Class
Modifier
Color
Size
Style
Shape
State
Theme
Tailwind Utility
```

를 실제 `Button` Component 하나에 집중해서 적용합니다.

```text
btn
 │
 ├─ Color
 ├─ Size
 ├─ Style
 ├─ Shape
 ├─ State
 └─ Tailwind Utility
        ↓
실전 Button Design
```

Button을 통해 daisyUI Component를 **읽고, 조합하고, 커스터마이징하는 전체 패턴**을 완성합니다.
