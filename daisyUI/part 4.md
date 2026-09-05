# PART 4. daisyUI Form & 사용자 입력 UI

## 1. 이번 PART의 목표

웹 애플리케이션에서 사용자가 서버로 데이터를 보내는 대표적인 출발점은 Form입니다.

```text
사용자
  ↓
Input / Select / Checkbox ...
  ↓
Form
  ↓
JavaScript / React
  ↓
Validation
  ↓
API Request
  ↓
Server
```

daisyUI는 Form 자체의 데이터 처리 기능을 제공하는 라이브러리가 아닙니다.

핵심 역할은 다음과 같습니다.

```text
HTML / React
→ 입력과 상태 관리

daisyUI
→ Form UI 표현

Tailwind CSS
→ 세부적인 Layout / Style 조정
```

즉,

> **daisyUI는 Form의 동작을 관리하는 것이 아니라 Form을 구성하는 UI Component를 제공합니다.**

---

# 2. HTML Form에서 daisyUI Form으로

일반 HTML Input부터 보겠습니다.

```html
<input
  type="text"
  placeholder="이름을 입력하세요"
/>
```

브라우저에서는 기본 Input UI가 나타납니다.

daisyUI를 사용하면:

```html
<input
  type="text"
  placeholder="이름을 입력하세요"
  class="input"
/>
```

처럼 Component Class를 추가할 수 있습니다.

핵심은:

```text
<input>
   +
input
   ↓
daisyUI Input UI
```

입니다.

HTML Element 자체가 바뀌는 것은 아닙니다.

---

# 3. Input

가장 많이 사용하는 Form Component입니다.

```html
<input
  type="text"
  class="input"
  placeholder="이름"
/>
```

`input`은 daisyUI의 Input Component Class입니다.

Tailwind Utility와 함께 사용할 수도 있습니다.

```html
<input
  type="email"
  class="input w-full"
  placeholder="email@example.com"
/>
```

구조를 보면:

```text
input
→ daisyUI Component

w-full
→ Tailwind Utility
```

입니다.

---

# 4. Input의 다양한 상태

Form에서는 단순히 Input을 보여주는 것보다 **현재 상태를 사용자에게 전달하는 것**이 중요합니다.

예를 들어 정상 상태:

```html
<input class="input" />
```

성공 상태:

```html
<input class="input input-success" />
```

경고 상태:

```html
<input class="input input-warning" />
```

오류 상태:

```html
<input class="input input-error" />
```

개념적으로:

```text
Input
  │
  ├─ Normal
  ├─ Success
  ├─ Warning
  └─ Error
```

처럼 표현할 수 있습니다.

이러한 상태 표현은 PART 3에서 배운 Semantic Color와 연결됩니다.

```text
input-success
      ↓
success
      ↓
현재 Theme
      ↓
실제 색상
```

---

# 5. Label과 Input

실전 Form에서는 Input만 배치하지 않습니다.

사용자가 **무엇을 입력해야 하는지** 설명하는 Label이 필요합니다.

HTML의 기본 구조는:

```html
<label for="email">
  이메일
</label>

<input
  id="email"
  type="email"
  class="input"
/>
```

입니다.

여기서 중요한 것은:

```text
<label for="email">
        │
        │ 연결
        ▼
<input id="email">
```

입니다.

Label을 클릭하면 해당 Input에 Focus가 이동합니다.

따라서 Label은 단순한 디자인 요소가 아니라 **Form 접근성에서 중요한 HTML Element**입니다.

---

# 6. Fieldset과 Legend

관련된 Form Control들을 하나의 그룹으로 묶을 때는 HTML의 `fieldset`과 `legend`를 사용할 수 있습니다.

```html
<fieldset class="fieldset">
  <legend class="fieldset-legend">
    계정 정보
  </legend>

  <label for="email">
    이메일
  </label>

  <input
    id="email"
    type="email"
    class="input"
  />
</fieldset>
```

개념적으로:

```text
fieldset
│
├─ legend
│    └─ 그룹 제목
│
├─ label
│
└─ input
```

입니다.

여기에서도 중요한 것은 **HTML Semantic 구조와 daisyUI Style을 구분하는 것**입니다.

```text
fieldset / legend
→ HTML Semantic Element

fieldset / fieldset-legend
→ daisyUI Component Class
```

---

# 7. Select

여러 선택지 중 하나를 선택할 때 사용합니다.

```html
<select class="select">
  <option disabled selected>
    카테고리 선택
  </option>

  <option>Frontend</option>
  <option>Backend</option>
  <option>DevOps</option>
</select>
```

구조:

```text
select
  │
  ├─ option
  ├─ option
  └─ option
```

daisyUI에서는:

```text
<select>
   +
select
   ↓
daisyUI Select UI
```

가 됩니다.

---

# 8. Textarea

여러 줄의 텍스트를 입력할 때 사용합니다.

```html
<textarea
  class="textarea"
  placeholder="내용을 입력하세요"
></textarea>
```

게시글 작성, 문의 내용, 자기소개 등에 많이 사용합니다.

예:

```html
<textarea
  class="textarea w-full"
  rows="5"
  placeholder="상품 후기를 작성하세요"
></textarea>
```

여기에서도:

```text
textarea
→ daisyUI Component

w-full
→ Tailwind Utility

rows
→ HTML Attribute
```

라는 역할 구분이 중요합니다.

---

# 9. Checkbox

여러 옵션을 독립적으로 선택할 때 사용합니다.

```html
<input
  type="checkbox"
  class="checkbox"
/>
```

Label과 함께 사용하면:

```html
<label class="flex items-center gap-2">
  <input
    type="checkbox"
    class="checkbox"
  />

  이용약관에 동의합니다.
</label>
```

구조는:

```text
Label
│
├─ Checkbox
│
└─ 설명 Text
```

입니다.

---

# 10. Radio

여러 선택지 중 **하나만 선택**하도록 만들 때 사용합니다.

```html
<label>
  <input
    type="radio"
    name="gender"
    class="radio"
  />
  남성
</label>

<label>
  <input
    type="radio"
    name="gender"
    class="radio"
  />
  여성
</label>
```

여기서 중요한 것은 `name`입니다.

```text
name="gender"
      │
      ├─ Radio A
      ├─ Radio B
      └─ Radio C
```

같은 `name`을 가지는 Radio Button들이 하나의 선택 그룹을 형성합니다.

이것은 daisyUI 기능이 아니라 **HTML Radio의 동작**입니다.

---

# 11. Checkbox와 Radio의 차이

두 Component를 확실하게 구분해야 합니다.

| Checkbox            | Radio         |
| ------------------- | ------------- |
| 여러 항목 선택 가능         | 그룹에서 하나 선택    |
| 독립적인 Boolean 상태에 적합 | 상호 배타적 선택에 적합 |
| `checkbox`          | `radio`       |

예를 들어:

```text
관심 분야

☑ Java
☑ React
☑ Spring

→ Checkbox
```

반면:

```text
배송 방법

● 일반 배송
○ 새벽 배송
○ 직접 수령

→ Radio
```

입니다.

---

# 12. Toggle

ON/OFF 형태의 설정 UI에는 Toggle을 사용할 수 있습니다.

```html
<input
  type="checkbox"
  class="toggle"
/>
```

HTML 관점에서는 여전히:

```html
type="checkbox"
```

입니다.

하지만 daisyUI의:

```text
toggle
```

Class가 시각적으로 Switch 형태로 표현합니다.

즉:

```text
HTML
checkbox

    +

daisyUI
toggle

    ↓

Switch 형태 UI
```

입니다.

이 구조는 daisyUI의 철학을 이해하는 데 상당히 중요합니다.

---

# 13. Range

숫자 범위를 시각적으로 선택할 때 사용합니다.

```html
<input
  type="range"
  min="0"
  max="100"
  value="50"
  class="range"
/>
```

예:

```text
볼륨

0 ───────●──────── 100
```

HTML Attribute:

```text
min
max
value
```

가 실제 범위를 정의하고,

```text
range
```

가 UI 표현을 담당합니다.

---

# 14. Rating

별점 UI가 필요할 때 사용할 수 있습니다.

예를 들어:

```html
<div class="rating">
  <input
    type="radio"
    name="rating"
    class="mask mask-star"
  />

  <input
    type="radio"
    name="rating"
    class="mask mask-star"
  />

  <input
    type="radio"
    name="rating"
    class="mask mask-star"
  />
</div>
```

중요한 점은 Rating 역시 내부적으로 **Radio Input을 활용할 수 있다는 것**입니다.

```text
Rating UI

☆ ☆ ☆ ☆ ☆
│ │ │ │ │
└─ Radio Input Group
```

즉 시각적인 별점 Component와 실제 Form Control의 관계를 함께 이해해야 합니다.

---

# 15. File Input

파일 업로드 UI도 제공합니다.

```html
<input
  type="file"
  class="file-input"
/>
```

예:

```html
<input
  type="file"
  class="file-input w-full"
/>
```

하지만 중요한 점이 있습니다.

daisyUI가 파일을 서버에 업로드해주는 것은 아닙니다.

```text
daisyUI
→ File Input UI

Browser
→ File 선택

JavaScript / React
→ File 객체 처리

HTTP Request
→ 실제 Upload

Server
→ 파일 저장
```

입니다.

---

# 16. Form Validation과 daisyUI

여기서 중요한 경계를 하나 잡아야 합니다.

daisyUI는:

```text
"이 이메일이 올바른가?"
```

를 판단하는 Validation Library가 아닙니다.

예를 들어:

```html
<input
  type="email"
  required
  class="input"
/>
```

에서:

```text
type="email"
required
```

는 HTML Validation이고,

```text
input
```

은 daisyUI입니다.

React에서는 Validation 결과에 따라 Class를 변경할 수도 있습니다.

```jsx
<input
  className={`input ${
    error ? 'input-error' : ''
  }`}
/>
```

흐름은:

```text
사용자 입력
    ↓
Validation
    ↓
error ?
 ┌──┴──┐
Yes    No
 │      │
 ▼      ▼
input-  Normal
error   Input
```

입니다.

---

# 17. React Controlled Component와 연결

React에서는 Form Element의 값을 State와 연결하는 경우가 많습니다.

```jsx
const [email, setEmail] = useState('')

return (
  <input
    type="email"
    className="input w-full"
    value={email}
    onChange={(e) =>
      setEmail(e.target.value)
    }
  />
)
```

구조를 보면:

```text
React State
    │
    ▼
  value
    │
    ▼
 <input>
    │
    │ onChange
    ▼
setEmail()
    │
    └──────────→ React State
```

여기서 daisyUI는:

```text
className="input w-full"
```

부분의 **UI 표현**을 담당합니다.

React가:

```text
value
onChange
State
```

를 담당합니다.

---

# 18. 실전 로그인 Form

지금까지 배운 내용을 합쳐보겠습니다.

```jsx
<form className="space-y-4">
  <fieldset className="fieldset">
    <legend className="fieldset-legend">
      이메일
    </legend>

    <input
      type="email"
      className="input w-full"
      placeholder="email@example.com"
    />
  </fieldset>

  <fieldset className="fieldset">
    <legend className="fieldset-legend">
      비밀번호
    </legend>

    <input
      type="password"
      className="input w-full"
      placeholder="비밀번호"
    />
  </fieldset>

  <label className="flex items-center gap-2">
    <input
      type="checkbox"
      className="checkbox"
    />
    로그인 상태 유지
  </label>

  <button
    type="submit"
    className="btn btn-primary w-full"
  >
    로그인
  </button>
</form>
```

구조적으로 보면:

```text
Login Form
│
├─ Email Field
│   ├─ Label
│   └─ Input
│
├─ Password Field
│   ├─ Label
│   └─ Input
│
├─ Checkbox
│
└─ Submit Button
```

입니다.

---

# 19. 실전 회원가입 Form

조금 더 복잡한 Form은 다음과 같은 구조가 됩니다.

```text
회원가입
│
├─ 이름
│   └─ Input
│
├─ 이메일
│   ├─ Input
│   └─ Validation Message
│
├─ 비밀번호
│   ├─ Input
│   └─ Validation Message
│
├─ 역할
│   └─ Select
│
├─ 관심 분야
│   ├─ Checkbox
│   ├─ Checkbox
│   └─ Checkbox
│
├─ 알림 수신
│   └─ Toggle
│
├─ 이용약관
│   └─ Checkbox
│
└─ 가입하기
    └─ Button
```

여기서부터 daisyUI가 실제 프로젝트의 Form UI를 얼마나 빠르게 구성할 수 있는지 체감할 수 있습니다.

---

# 20. 역할을 정확하게 구분하자

Form을 만들 때 다음 네 계층을 혼동하면 안 됩니다.

```text
HTML
│
├─ input
├─ select
├─ textarea
├─ label
└─ fieldset
        │
        ▼
Form의 의미와 기본 동작


daisyUI
│
├─ input
├─ select
├─ checkbox
├─ radio
├─ toggle
└─ range
        │
        ▼
Component UI


Tailwind CSS
│
├─ w-full
├─ gap-4
├─ grid
├─ flex
└─ mt-4
        │
        ▼
Layout / 세부 Style


React
│
├─ State
├─ onChange
├─ Validation
└─ Submit
        │
        ▼
Application Logic
```

이 구분이 **PART 4의 가장 중요한 개념**입니다.

---

# 21. PART 4 핵심 정리

daisyUI Form을 이해할 때는 Component 이름을 외우는 것보다 **각 기술이 어디까지 담당하는지** 이해하는 것이 중요합니다.

```text
HTML
→ Form Control의 의미와 기본 동작

daisyUI
→ Form Component의 시각적 표현

Tailwind CSS
→ Layout과 세부 Style

React
→ State와 Event 처리

Validation
→ 입력값 검증

API
→ 서버와 데이터 교환
```

그리고 daisyUI Form Component의 전체 관계는 다음과 같이 정리할 수 있습니다.

```text
                  Form
                    │
       ┌────────────┼─────────────┐
       │            │             │
       ▼            ▼             ▼
    Text 입력      선택 입력       설정/범위
       │            │             │
   ┌───┼───┐    ┌───┼────┐    ┌───┼───┐
   ▼   ▼   ▼    ▼   ▼    ▼    ▼   ▼   ▼
Input Textarea   Select Checkbox Radio Toggle Range
       │
       └──────────────┬──────────────┘
                      ▼
               Validation / State
                      ▼
                   Submit
                      ▼
                     API
```

**PART 4의 한 문장 정리:**

> **HTML이 Form의 의미와 동작을 만들고, daisyUI가 UI를 표현하며, Tailwind CSS가 Layout을 조정하고, React가 State와 Event를 관리한다.**

이 구분을 확실하게 잡아두면 이후 로그인, 회원가입, 검색/필터, 상품 등록, 관리자 페이지 같은 실제 Form UI를 훨씬 쉽게 설계할 수 있습니다.
