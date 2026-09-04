# PART 5. State & Variant

## 사용자 상태에 따라 스타일 변경하기

PART 4에서는 viewport width에 따라 Utility를 적용했습니다.

```jsx
<div className="text-xl md:text-3xl">
  Tailwind CSS
</div>
```

```text
md:text-3xl
│       │
│       └─ Utility
└───────── Responsive Variant
```

PART 5에서는 같은 구조를 사용자 인터랙션과 UI 상태에 적용합니다.

```jsx
<button className="bg-blue-500 hover:bg-blue-600">
  저장
</button>
```

```text
hover:bg-blue-600
│          │
│          └─ Utility
└──────────── State Variant
```

핵심은 하나입니다.

> **Variant는 “언제 적용할 것인가?”, Utility는 “무엇을 적용할 것인가?”를 표현합니다.**

---

# 1. State Variant란?

일반적인 CSS에서는 상태를 pseudo-class로 표현합니다.

```css
.button {
  background-color: blue;
}

.button:hover {
  background-color: darkblue;
}
```

Tailwind에서는:

```jsx
<button className="
  bg-blue-500
  hover:bg-blue-600
">
  저장
</button>
```

처럼 작성합니다.

개념적으로:

```text
CSS

.button:hover
      │
      ▼
특정 상태에서 스타일 적용
```

Tailwind:

```text
hover:bg-blue-600
  │         │
  │         └─ Utility
  └─────────── Variant
```

즉:

```text
Variant : Utility
```

구조입니다.

---

# 2. PART 4의 Responsive Variant와 같은 원리

PART 4:

```jsx
md:text-3xl
```

```text
md
→ viewport가 md 이상일 때

text-3xl
→ font-size 변경
```

PART 5:

```jsx
hover:bg-blue-600
```

```text
hover
→ pointer hover 상태일 때

bg-blue-600
→ background-color 변경
```

따라서 Tailwind를 다음처럼 이해할 수 있습니다.

```text
조건
 :
스타일
```

예:

```text
md:text-3xl
hover:bg-blue-600
focus:ring-2
disabled:opacity-50
```

---

# 3. `hover:`

마우스 또는 hover가 가능한 pointer가 요소 위에 위치하는 상태입니다.

```jsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  text-white
  px-4
  py-2
  rounded-lg
">
  저장
</button>
```

동작:

```text
기본
┌──────────┐
│   저장   │  bg-blue-500
└──────────┘

       ↓ hover

┌──────────┐
│   저장   │  bg-blue-600
└──────────┘
```

일반 CSS의:

```css
button:hover {
  background-color: ...;
}
```

에 대응합니다.

`hover:`는 버튼뿐 아니라 다양한 Utility와 조합할 수 있습니다.

```text
hover:text-blue-600
hover:bg-gray-100
hover:border-blue-500
hover:shadow-md
hover:scale-105
```

다만 touch 환경에는 전통적인 mouse hover와 동일한 인터랙션이 없을 수 있으므로 **중요한 기능을 hover에만 의존하면 안 됩니다.**

---

# 4. `focus:`

요소가 focus 상태일 때 적용됩니다.

```jsx
<input
  className="
    border
    border-gray-300
    focus:border-blue-500
  "
/>
```

사용자가 input을 클릭하거나 keyboard navigation으로 focus하면:

```text
기본
border-gray-300

       ↓ focus

focus:border-blue-500
```

일반 CSS:

```css
input:focus {
  border-color: ...;
}
```

와 같은 개념입니다.

---

# 5. `focus-visible:`

실전에서는 `focus:`와 함께 반드시 알아야 하는 Variant입니다.

```jsx
<button className="
  px-4
  py-2
  rounded-lg

  focus-visible:outline-none
  focus-visible:ring-2
  focus-visible:ring-blue-500
">
  저장
</button>
```

`focus-visible:`은 브라우저가 **focus indicator를 표시하는 것이 적절하다고 판단하는 상황**에 대응합니다.

특히 keyboard navigation의 focus 표시를 구현할 때 유용합니다.

개념적으로:

```text
Tab 키로 이동
     ↓
Button focus
     ↓
focus-visible
     ↓
ring 표시
```

중요:

> **Focus indicator는 keyboard 사용자가 현재 위치를 알 수 있게 해주는 중요한 접근성 정보입니다.**

따라서 단순히:

```jsx
focus:outline-none
```

만 사용해서 focus 표시를 없애는 것은 피하는 것이 좋습니다.

없앴다면 다른 명확한 focus indicator를 제공해야 합니다.

---

# 6. `active:`

요소가 활성화되는 순간의 상태입니다.

버튼에서는 일반적으로 누르고 있는 순간을 생각하면 쉽습니다.

```jsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  active:bg-blue-700
">
  저장
</button>
```

흐름:

```text
기본
bg-blue-500

   ↓ pointer hover

hover
bg-blue-600

   ↓ press

active
bg-blue-700
```

크기 변화도 줄 수 있습니다.

```jsx
active:scale-95
```

예:

```jsx
<button className="
  px-4
  py-2
  bg-blue-500
  hover:bg-blue-600
  active:bg-blue-700
  active:scale-95
  text-white
  rounded-lg
">
  저장
</button>
```

---

# 7. `disabled:`

HTML form element가 disabled 상태일 때 적용합니다.

```jsx
<button
  disabled
  className="
    bg-blue-500
    text-white
    px-4
    py-2

    disabled:bg-gray-300
    disabled:text-gray-500
    disabled:cursor-not-allowed
    disabled:opacity-50
  "
>
  저장
</button>
```

동작:

```text
enabled

┌──────────┐
│   저장   │
└──────────┘


disabled

┌──────────┐
│   저장   │
└──────────┘
opacity ↓
cursor → not-allowed
```

중요한 점은:

```jsx
disabled:opacity-50
```

가 버튼을 disabled 상태로 **만드는 것이 아니라는 것**입니다.

실제 상태는:

```jsx
disabled
```

HTML attribute가 결정합니다.

Tailwind는 그 상태를 보고 스타일을 적용합니다.

즉:

```text
HTML / React
상태 결정

disabled={true}
       ↓
DOM
disabled attribute
       ↓
Tailwind
disabled:* 적용
```

---

# 8. React state와 Tailwind Variant의 차이

이 부분은 매우 중요합니다.

Tailwind의:

```jsx
hover:
focus:
disabled:
checked:
```

는 해당 CSS 상태에 따라 스타일을 적용합니다.

하지만 다음과 같은 애플리케이션 상태:

```text
isOpen
isLoading
isSelected
isLoggedIn
```

자체를 Tailwind가 관리하는 것은 아닙니다.

예:

```jsx
const [isOpen, setIsOpen] = useState(false)
```

이 state는 React가 관리합니다.

```text
React
→ 상태 관리

Tailwind
→ 상태에 따른 스타일 표현
```

예:

```jsx
<button
  className={
    isOpen
      ? 'bg-blue-600 text-white'
      : 'bg-gray-100 text-gray-900'
  }
>
  Menu
</button>
```

여기서는 React state에 따라 className 자체를 결정하고 있습니다.

---

# 9. `checked:`

Checkbox 또는 Radio의 checked 상태에 대응합니다.

```jsx
<input
  type="checkbox"
  className="
    size-5
    accent-blue-600
  "
/>
```

상태 기반 스타일이 필요한 경우 `checked:`도 사용할 수 있습니다.

```jsx
<input
  type="checkbox"
  className="
    appearance-none
    size-5
    border
    border-gray-300
    rounded

    checked:bg-blue-600
    checked:border-blue-600
  "
/>
```

개념:

```text
unchecked
border-gray-300

      ↓ click

checked
bg-blue-600
border-blue-600
```

`checked:`는 CSS의:

```css
:checked
```

에 대응합니다.

---

# 10. `required:`, `invalid:`, `valid:`

Form 상태도 Variant로 표현할 수 있습니다.

예:

```jsx
<input
  type="email"
  required
  className="
    border
    border-gray-300

    invalid:border-red-500
    valid:border-green-500
  "
/>
```

구조:

```text
HTML validation state
        ↓
:invalid / :valid
        ↓
Tailwind Variant
        ↓
invalid:border-red-500
valid:border-green-500
```

또:

```jsx
required:border-blue-500
```

처럼 `required:`를 사용할 수도 있습니다.

단, validation UX에서는 색상만으로 상태를 전달하지 말고 **텍스트 메시지나 아이콘 등 추가 정보도 함께 제공**하는 것이 좋습니다.

---

# 11. Structural Variant

State Variant와 함께 자주 사용하는 것이 요소의 위치나 구조를 기준으로 한 Variant입니다.

대표적으로:

```text
first:
last:
odd:
even:
```

가 있습니다.

예:

```jsx
<ul>
  {items.map((item) => (
    <li
      key={item.id}
      className="
        py-3
        border-b
        last:border-b-0
      "
    >
      {item.name}
    </li>
  ))}
</ul>
```

마지막 item:

```text
last:border-b-0
```

이 적용됩니다.

---

# 12. `first:`와 `last:`

예:

```jsx
<div className="
  border-t
  first:border-t-0
">
```

또는:

```jsx
<li className="
  py-3
  border-b
  last:border-b-0
">
```

개념:

```text
Item 1  ─────────
Item 2  ─────────
Item 3
        ↑
        last:border-b-0
```

리스트 UI에서 매우 자주 사용합니다.

---

# 13. `odd:`와 `even:`

Table이나 List의 alternating row 스타일에 사용할 수 있습니다.

```jsx
{users.map((user) => (
  <div
    key={user.id}
    className="
      px-4
      py-3
      odd:bg-white
      even:bg-gray-50
    "
  >
    {user.name}
  </div>
))}
```

결과:

```text
Row 1   white
Row 2   gray
Row 3   white
Row 4   gray
```

일반 CSS의:

```css
:nth-child(odd)
:nth-child(even)
```

과 연결해서 이해하면 됩니다.

---

# 14. `group`과 `group-hover:`

지금까지는 **자기 자신의 상태**를 이용했습니다.

```text
button:hover
input:focus
input:checked
```

하지만 부모의 상태에 따라 자식 스타일을 변경하고 싶을 때가 있습니다.

예:

```jsx
<article className="group">
  <h2 className="
    text-gray-900
    group-hover:text-blue-600
  ">
    Product
  </h2>
</article>
```

구조:

```text
article
class="group"
     │
     │ hover
     ▼
┌─────────────────────┐
│ Product             │
│       │             │
│       ▼             │
│ group-hover:        │
│ text-blue-600       │
└─────────────────────┘
```

즉:

```text
부모 상태
     ↓
자식 스타일 변경
```

입니다.

---

# 15. Product Card에서 `group-hover:`

실전에서는 Card 전체에 hover했을 때 내부 이미지나 제목을 변경할 수 있습니다.

```jsx
<article
  className="
    group
    overflow-hidden
    rounded-xl
    border
    border-gray-200
  "
>
  <img
    src={product.image}
    alt={product.name}
    className="
      w-full
      aspect-[4/3]
      object-cover
      transition-transform
      group-hover:scale-105
    "
  />

  <div className="p-4">
    <h2
      className="
        font-semibold
        text-gray-900
        group-hover:text-blue-600
      "
    >
      {product.name}
    </h2>
  </div>
</article>
```

흐름:

```text
Card hover
     │
     ├── Image
     │     └─ scale-105
     │
     └── Title
           └─ text-blue-600
```

---

# 16. `peer`와 `peer-*`

`group`이 부모의 상태를 자식에게 전달하는 패턴이라면, `peer`는 **형제 요소의 상태를 기준으로 다른 형제의 스타일을 변경**할 때 유용합니다.

예:

```jsx
<label>
  <input
    type="checkbox"
    className="peer"
  />

  <span className="
    text-gray-500
    peer-checked:text-blue-600
  ">
    이메일 알림 받기
  </span>
</label>
```

구조:

```text
input.peer
     │
     │ checked
     ▼
span
peer-checked:text-blue-600
```

즉:

```text
형제의 상태
     ↓
다른 요소 스타일 변경
```

입니다.

---

# 17. `peer-invalid:`

Form에서 특히 유용합니다.

```jsx
<div>
  <input
    type="email"
    required
    className="
      peer
      border
      border-gray-300
      invalid:border-red-500
    "
  />

  <p
    className="
      invisible
      mt-1
      text-sm
      text-red-600
      peer-invalid:visible
    "
  >
    올바른 이메일을 입력하세요.
  </p>
</div>
```

흐름:

```text
input
     ↓ invalid

input.peer:invalid
     ↓

message
peer-invalid:visible
```

여기에서도 JavaScript 없이 CSS 상태를 이용할 수 있습니다.

---

# 18. Variant를 여러 개 조합하기

Variant는 하나만 사용할 필요가 없습니다.

PART 4의 Responsive Variant와도 조합할 수 있습니다.

```jsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  md:hover:bg-blue-700
">
  저장
</button>
```

또:

```jsx
<input className="
  focus:border-blue-500
  disabled:opacity-50
  md:focus:ring-2
">
```

구조:

```text
md : hover : bg-blue-700
│      │           │
│      │           └─ Utility
│      └───────────── State Variant
└──────────────────── Responsive Variant
```

여러 조건을 단계적으로 조합할 수 있습니다.

---

# 19. Variant Chain 읽는 법

예:

```text
md:hover:bg-blue-700
```

왼쪽부터 조건을 읽으면 됩니다.

```text
md
↓
md breakpoint 이상

hover
↓
hover 상태

bg-blue-700
↓
background color 적용
```

즉:

> **md 이상이면서 hover 상태일 때 `bg-blue-700`을 적용합니다.**

다른 예:

```text
group-hover:text-blue-600
```

```text
group-hover
↓
group으로 지정된 관련 부모가 hover 상태

text-blue-600
↓
자식 text color 변경
```

---

# 20. Transition과 함께 사용하기

State Variant와 Transition을 함께 사용하면 상태 변화가 자연스럽게 보입니다.

```jsx
<button className="
  bg-blue-500
  hover:bg-blue-600
  transition-colors
  duration-200
">
  저장
</button>
```

개념:

```text
기본
bg-blue-500

      ↓ hover

즉시 변경
X

부드럽게 변화
O
```

`transition-*`은 **상태를 만드는 Utility가 아닙니다.**

상태 변화 과정의 animation/transition을 설정합니다.

예:

```text
transition-colors
transition-transform
transition-shadow
transition-all
```

---

# 21. Button 상태 전체 조합

실전 Button을 만들어 보겠습니다.

```jsx
<button
  className="
    px-5
    py-2.5

    bg-blue-600
    text-white
    font-medium
    rounded-lg

    hover:bg-blue-700
    active:bg-blue-800
    active:scale-95

    focus-visible:outline-none
    focus-visible:ring-2
    focus-visible:ring-blue-500
    focus-visible:ring-offset-2

    disabled:opacity-50
    disabled:cursor-not-allowed

    transition
  "
>
  저장
</button>
```

상태:

```text
Default
   ↓
Hover
   ↓
Active

Focus-visible
   ↓
Focus Ring

Disabled
   ↓
Opacity + Cursor
```

하나의 Component 안에서 여러 상태를 모두 표현할 수 있습니다.

---

# 22. Input 상태 전체 조합

```jsx
<label className="block">
  <span className="text-sm font-medium text-gray-700">
    이메일
  </span>

  <input
    type="email"
    required
    placeholder="user@example.com"
    className="
      mt-2
      w-full
      rounded-lg
      border
      border-gray-300
      px-3
      py-2

      focus:border-blue-500
      focus:outline-none
      focus:ring-2
      focus:ring-blue-200

      invalid:border-red-500

      disabled:bg-gray-100
      disabled:text-gray-500
    "
  />
</label>
```

여기에는:

```text
Default
Focus
Invalid
Disabled
```

상태가 함께 들어 있습니다.

---

# 23. State Variant와 React 조건부 className

모든 상태를 CSS Variant만으로 표현할 수 있는 것은 아닙니다.

예를 들어 선택된 Tab:

```jsx
const [selected, setSelected] = useState('profile')
```

```jsx
<button
  className={
    selected === 'profile'
      ? 'bg-blue-600 text-white'
      : 'bg-gray-100 text-gray-700'
  }
>
  Profile
</button>
```

이 상태는:

```text
:hover
:focus
:checked
```

같은 CSS pseudo-class가 아니라 **애플리케이션 state**입니다.

따라서 React가 className을 결정합니다.

구분:

```text
CSS/DOM 상태
hover
focus
checked
disabled
invalid
    ↓
Tailwind Variant


Application 상태
isOpen
isLoading
selectedTab
isLoggedIn
    ↓
React state
    ↓
className 결정
```

이 차이를 이해하는 것이 중요합니다.

---

# 24. 자주 하는 실수 ① `disabled:`가 상태를 만든다고 생각하기

잘못된 생각:

```jsx
<button className="disabled:opacity-50">
```

이것만 작성하면 버튼이 disabled가 된다고 생각할 수 있습니다.

아닙니다.

```jsx
<button
  disabled={isLoading}
  className="disabled:opacity-50"
>
```

처럼 상태 자체가 먼저 존재해야 합니다.

```text
disabled={true}
       ↓
DOM disabled
       ↓
disabled:* 활성화
```

입니다.

---

# 25. 자주 하는 실수 ② focus indicator 제거

다음처럼 작성하면:

```jsx
focus:outline-none
```

브라우저의 focus outline을 제거할 수 있습니다.

하지만 대체 표시가 없다면 keyboard 사용자가 현재 focus 위치를 확인하기 어려워집니다.

따라서:

```jsx
focus-visible:outline-none
focus-visible:ring-2
focus-visible:ring-blue-500
```

처럼 명확한 대체 indicator를 제공하는 것이 좋습니다.

---

# 26. 자주 하는 실수 ③ hover에만 정보 의존

예를 들어 중요한 버튼을 hover해야만 발견할 수 있도록 만들면 touch 사용자나 keyboard 사용자에게 문제가 될 수 있습니다.

```text
중요 정보
    ↓
hover에서만 표시
    ↓
피하는 것이 좋음
```

`hover:`는 **시각적 feedback을 강화하는 용도**로 사용하는 것이 좋습니다.

---

# 27. 자주 하는 실수 ④ Variant를 너무 복잡하게 연결

다음처럼 Variant가 지나치게 길어지면:

```text
md:group-hover:focus-visible:...
```

읽고 유지보수하기 어려워질 수 있습니다.

조건이 복잡해지면:

```text
Component 분리
React state
className 구성
```

등을 고려합니다.

Tailwind Variant는 강력하지만 **모든 상태 로직을 Utility 하나에 몰아넣는 도구는 아닙니다.**

---

# 28. 실전 Product Card

PART 4에서 만들었던 Responsive Product Grid에 State를 추가해 보겠습니다.

```jsx
<article
  className="
    group
    overflow-hidden
    rounded-xl
    border
    border-gray-200
    bg-white

    hover:shadow-lg
    transition-shadow
  "
>
  <img
    src={product.image}
    alt={product.name}
    className="
      w-full
      aspect-[4/3]
      object-cover

      transition-transform
      duration-300
      group-hover:scale-105
    "
  />

  <div className="p-4">
    <h2
      className="
        font-semibold
        text-gray-900
        group-hover:text-blue-600
      "
    >
      {product.name}
    </h2>

    <button
      className="
        mt-4
        w-full
        rounded-lg
        bg-blue-600
        px-4
        py-2
        text-white

        hover:bg-blue-700
        active:bg-blue-800

        focus-visible:outline-none
        focus-visible:ring-2
        focus-visible:ring-blue-500

        disabled:opacity-50

        transition
      "
    >
      장바구니
    </button>
  </div>
</article>
```

이 하나의 Card에는:

```text
Card
└─ hover:shadow-lg

Image
└─ group-hover:scale-105

Title
└─ group-hover:text-blue-600

Button
├─ hover:
├─ active:
├─ focus-visible:
└─ disabled:
```

가 모두 들어 있습니다.

---

# 29. PART 5 핵심 구조

지금까지 배운 Variant를 관계로 정리하면:

```text
Variant
│
├─ Responsive
│   ├─ sm:
│   ├─ md:
│   └─ lg:
│
├─ Interaction / State
│   ├─ hover:
│   ├─ focus:
│   ├─ focus-visible:
│   ├─ active:
│   ├─ disabled:
│   ├─ checked:
│   ├─ invalid:
│   └─ valid:
│
├─ Structural
│   ├─ first:
│   ├─ last:
│   ├─ odd:
│   └─ even:
│
└─ Relationship
    ├─ group-hover:
    └─ peer-checked:
```

공통 구조는:

```text
Variant : Utility
```

입니다.

---

# 30. PART 5 핵심 정리

가장 중요한 것은 Variant를 단순히 prefix 목록으로 외우는 것이 아닙니다.

```text
언제?
  ↓
Variant

무엇을?
  ↓
Utility
```

예:

```text
hover:bg-blue-600
```

```text
hover 상태일 때
        ↓
background color 변경
```

그리고:

```text
md:hover:bg-blue-700
```

은:

```text
md 이상
   AND
hover 상태
   ↓
bg-blue-700
```

입니다.

마지막으로 반드시 구분합니다.

```text
DOM / CSS 상태
hover / focus / disabled / checked
              ↓
       Tailwind Variant


Application 상태
isOpen / isLoading / selected
              ↓
          React state
              ↓
      className / rendering
```

> **Tailwind의 State Variant는 상태를 만드는 기능이 아니라, 이미 존재하는 상태에 따라 스타일을 적용하는 기능입니다.**

---

# 다음 PART

## PART 6. Layout — Flexbox & Grid

다음 PART에서는 PART 4에서 Responsive 예제로 사용했던 Flexbox와 Grid 자체를 본격적으로 학습합니다.

```text
Flexbox
├─ flex
├─ flex-row / flex-col
├─ justify-*
├─ items-*
├─ grow / shrink
├─ basis-*
└─ wrap

Grid
├─ grid
├─ grid-cols-*
├─ gap-*
├─ col-span-*
├─ row-span-*
├─ place-items-*
└─ auto-flow
```

지금까지 배운 Responsive와 State Variant도 Layout Utility에 그대로 적용할 수 있습니다.

```jsx
<div className="
  flex
  flex-col
  md:flex-row
">
```

```jsx
<div className="
  grid
  grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
">
```

즉 앞으로도 계속 사용하는 핵심 공식은 같습니다.

```text
Variant : Utility
```
