# PART 7. daisyUI Overlay & Feedback Component

## 1. 이번 PART에서 배울 내용

실제 웹 애플리케이션에서는 데이터를 보여주는 것만으로 충분하지 않습니다.

사용자는 다음과 같은 Action을 수행합니다.

```text
삭제
수정
로그인
상품 추가
메뉴 열기
상세 정보 확인
설정 변경
```

그리고 애플리케이션은 사용자에게 결과를 알려줘야 합니다.

```text
정말 삭제하시겠습니까?
저장되었습니다.
로그인에 실패했습니다.
메뉴가 열렸습니다.
추가 정보를 확인하세요.
```

이때 사용하는 Component가 Overlay와 Feedback Component입니다.

이번 PART에서는 다음을 다룹니다.

```text
Modal
Dropdown
Tooltip
Toast
Alert
Loading
Collapse
Accordion
```

핵심은 다음입니다.

> **Overlay는 기존 화면 위에 추가 UI를 표시하고, Feedback Component는 사용자의 Action 결과나 상태를 전달합니다.**

---

# 2. Overlay란?

Overlay는 현재 화면의 흐름 위에 임시 UI를 표시하는 방식입니다.

예:

```text
기존 화면
────────────────────────────
상품 목록
상품 목록
상품 목록

        ↓ Modal Open

────────────────────────────
상품 목록   ┌───────────────┐
상품 목록   │ 삭제 확인     │
상품 목록   │               │
            │ 취소   삭제   │
            └───────────────┘
```

대표적인 Overlay Component는:

```text
Modal
Dropdown
Tooltip
```

입니다.

다만 세 Component의 목적은 다릅니다.

```text
Modal
→ 중요한 Action / 집중이 필요한 내용

Dropdown
→ 선택지 / Menu

Tooltip
→ 짧은 보조 설명
```

---

# 3. Modal

Modal은 사용자가 특정 작업에 집중하도록 현재 화면 위에 별도의 UI를 표시합니다.

예:

```html
<dialog id="delete-modal" class="modal">
  <div class="modal-box">
    <h3 class="text-lg font-bold">
      상품 삭제
    </h3>

    <p class="py-4">
      정말 삭제하시겠습니까?
    </p>

    <div class="modal-action">
      <button class="btn">
        취소
      </button>

      <button class="btn btn-error">
        삭제
      </button>
    </div>
  </div>
</dialog>
```

구조:

```text
modal
│
└─ modal-box
    │
    ├─ Title
    ├─ Content
    └─ modal-action
         ├─ Cancel
         └─ Confirm
```

---

# 4. `dialog` Element와 daisyUI Modal

여기서 중요한 점이 있습니다.

```html
<dialog>
```

는 HTML Element입니다.

```text
dialog
→ HTML의 Dialog API

modal
modal-box
modal-action
→ daisyUI UI
```

즉 daisyUI가 Modal이라는 개념 자체를 새로 만든 것이 아닙니다.

HTML의 Dialog 기능 위에 스타일과 구조를 제공합니다.

---

# 5. Modal 열기

HTML `dialog`는 JavaScript로 열 수 있습니다.

```html
<button
  class="btn btn-primary"
  onclick="delete_modal.showModal()"
>
  삭제
</button>
```

```html
<dialog
  id="delete_modal"
  class="modal"
>
  ...
</dialog>
```

흐름:

```text
Button 클릭
    ↓
showModal()
    ↓
HTMLDialogElement
    ↓
Dialog Open
    ↓
daisyUI Modal 표시
```

여기서:

```text
showModal()
→ Browser API

modal
→ daisyUI
```

입니다.

---

# 6. Modal 닫기

Dialog 내부에서:

```html
<form method="dialog">
  <button class="btn">
    닫기
  </button>
</form>
```

처럼 사용할 수 있습니다.

`method="dialog"`는 HTML Dialog 동작입니다.

```text
<form method="dialog">
       ↓
Button 클릭
       ↓
Dialog Close
```

JavaScript를 사용할 수도 있습니다.

```js
delete_modal.close()
```

---

# 7. React에서 Modal 제어

React에서는 `ref`를 이용해 Dialog를 제어할 수 있습니다.

```jsx
import { useRef } from 'react'

function DeleteModal() {
  const modalRef = useRef(null)

  const openModal = () => {
    modalRef.current?.showModal()
  }

  const closeModal = () => {
    modalRef.current?.close()
  }

  return (
    <>
      <button
        className="btn btn-error"
        onClick={openModal}
      >
        삭제
      </button>

      <dialog
        ref={modalRef}
        className="modal"
      >
        <div className="modal-box">
          <h3 className="font-bold text-lg">
            삭제 확인
          </h3>

          <p className="py-4">
            정말 삭제하시겠습니까?
          </p>

          <div className="modal-action">
            <button
              className="btn"
              onClick={closeModal}
            >
              취소
            </button>

            <button
              className="btn btn-error"
            >
              삭제
            </button>
          </div>
        </div>
      </dialog>
    </>
  )
}
```

역할:

```text
React
→ Event / ref

HTML Dialog API
→ showModal() / close()

daisyUI
→ Modal UI
```

---

# 8. Modal을 언제 사용해야 하는가?

Modal은 중요한 Action에 적합합니다.

예:

```text
삭제 확인
결제 확인
로그인
회원 정보 수정
상세 설정
중요한 경고
```

하지만 모든 정보를 Modal로 띄우는 것은 좋지 않습니다.

```text
단순 안내
짧은 설명
일반적인 Navigation
```

에는 다른 Component가 더 적절할 수 있습니다.

---

# 9. Dropdown

Dropdown은 Button이나 특정 Trigger를 눌렀을 때 작은 Menu나 Action 목록을 표시합니다.

예:

```html
<div class="dropdown">
  <div
    tabindex="0"
    role="button"
    class="btn"
  >
    메뉴
  </div>

  <ul
    tabindex="-1"
    class="
      dropdown-content
      menu
      bg-base-100
      rounded-box
      shadow
      w-52
      p-2
    "
  >
    <li><a>프로필</a></li>
    <li><a>설정</a></li>
    <li><a>로그아웃</a></li>
  </ul>
</div>
```

구조:

```text
dropdown
│
├─ Trigger
│
└─ dropdown-content
     └─ menu
```

---

# 10. Dropdown의 대표 사용 위치

```text
사용자 Profile Menu
상품 Action Menu
정렬 기준 선택
더보기 메뉴
Navbar 메뉴
```

예:

```text
[ 홍길동 ▼ ]
      │
      ▼
┌──────────────┐
│ 프로필       │
│ 설정         │
│ 로그아웃     │
└──────────────┘
```

---

# 11. Dropdown vs Modal

두 Component를 혼동하면 안 됩니다.

```text
Dropdown
→ 짧은 선택 / Action 목록
→ 현재 Context 유지

Modal
→ 중요한 작업
→ 사용자 집중 필요
```

예:

```text
프로필 메뉴
→ Dropdown

계정 삭제 확인
→ Modal
```

---

# 12. Tooltip

Tooltip은 짧은 설명을 제공하는 Component입니다.

```html
<div
  class="tooltip"
  data-tip="상품을 장바구니에 추가합니다"
>
  <button class="btn">
    Cart
  </button>
</div>
```

화면:

```text
상품을 장바구니에 추가합니다
              ↓
          [ Cart ]
```

Tooltip은 다음과 같은 경우에 유용합니다.

```text
Icon Button 설명
줄임말 설명
짧은 도움말
추가 Context
```

---

# 13. Tooltip을 남용하면 안 되는 이유

Tooltip은 짧은 보조 정보에 적합합니다.

다음처럼 중요한 정보를 Tooltip에만 넣으면 좋지 않습니다.

```text
이 버튼을 누르면 계정이 즉시 삭제됩니다.
```

이런 중요한 내용은:

```text
Alert
Modal
본문 Text
```

처럼 더 명확한 UI가 적절합니다.

또 Tooltip만으로 필수 정보를 전달하면 Touch Device나 접근성 측면에서도 문제가 될 수 있습니다.

---

# 14. Feedback이란?

Feedback Component는 사용자의 Action 결과를 전달합니다.

예:

```text
저장 성공
삭제 실패
네트워크 오류
로그인 중
파일 업로드 중
```

대표 Component:

```text
Alert
Toast
Loading
Progress
```

PART 6에서는 Progress를 다뤘으므로 이번 PART에서는 Alert, Toast, Loading 중심으로 설명합니다.

---

# 15. Alert

Alert는 화면 안에서 비교적 지속적으로 상태나 중요한 정보를 전달합니다.

```html
<div class="alert alert-success">
  저장되었습니다.
</div>
```

예:

```html
<div class="alert alert-info">
  새로운 업데이트가 있습니다.
</div>

<div class="alert alert-warning">
  입력값을 확인해주세요.
</div>

<div class="alert alert-error">
  서버 연결에 실패했습니다.
</div>
```

PART 3에서 배운 Feedback Semantic Color와 연결됩니다.

```text
info
success
warning
error
```

---

# 16. Alert는 상태를 관리하지 않는다

다음 코드:

```html
<div class="alert alert-error">
  로그인에 실패했습니다.
</div>
```

에서 daisyUI는 실패 여부를 판단하지 않습니다.

실제 Logic:

```text
Login Request
      ↓
Success ?
   ┌──┴──┐
  Yes    No
   │      │
   ▼      ▼
페이지    Error State
이동         │
             ▼
       alert-error
```

즉:

```text
React / Query Library
→ 상태 판단

daisyUI
→ Alert UI
```

입니다.

---

# 17. React에서 Alert 조건부 렌더링

```jsx
function LoginMessage({ error }) {
  if (!error) {
    return null
  }

  return (
    <div className="alert alert-error">
      로그인에 실패했습니다.
    </div>
  )
}
```

흐름:

```text
error State
    ↓
true ?
 ┌──┴──┐
Yes    No
 │      │
 ▼      ▼
Alert  표시 안 함
```

---

# 18. Toast

Toast는 사용자의 Action 결과를 **짧은 시간 동안 화면 한쪽에 표시하는 Feedback UI 패턴**입니다.

개념:

```text
                     ┌──────────────────────┐
                     │ 저장되었습니다.     │
                     └──────────────────────┘

Application
```

예:

```text
상품이 장바구니에 추가되었습니다.
저장되었습니다.
삭제되었습니다.
링크를 복사했습니다.
```

처럼 짧은 결과 안내에 적합합니다.

---

# 19. daisyUI에서 Toast 구성

Toast Container와 Alert를 조합할 수 있습니다.

```html
<div class="toast toast-end">
  <div class="alert alert-success">
    <span>
      저장되었습니다.
    </span>
  </div>
</div>
```

구조:

```text
toast
│
└─ alert
```

즉:

```text
toast
→ 화면 위치

alert
→ Feedback 표현
```

입니다.

---

# 20. Toast 위치

위치를 조정할 수 있습니다.

개념적으로:

```text
toast-start
toast-center
toast-end
```

등을 조합해 위치를 결정합니다.

예:

```text
Top Left       Top Center       Top Right

Bottom Left    Bottom Center    Bottom Right
```

실전에서는 우측 상단이나 우측 하단을 많이 사용합니다.

---

# 21. Toast의 중요한 한계

daisyUI의 `toast`가 자동으로:

```text
3초 후 사라짐
중복 Toast 관리
Queue 관리
Animation Lifecycle
```

를 전부 처리하는 것은 아닙니다.

이런 동작은 애플리케이션 Logic이나 Toast Library가 담당합니다.

즉:

```text
daisyUI
→ Toast UI / Position

React / JavaScript
→ 생성 / 삭제 / Timer

Toast Library
→ Queue / Lifecycle / 편의 기능
```

입니다.

---

# 22. React에서 간단한 Toast 구현

```jsx
import { useState } from 'react'

function SaveButton() {
  const [message, setMessage] = useState('')

  const handleSave = () => {
    setMessage('저장되었습니다.')

    setTimeout(() => {
      setMessage('')
    }, 3000)
  }

  return (
    <>
      <button
        className="btn btn-primary"
        onClick={handleSave}
      >
        저장
      </button>

      {message && (
        <div className="toast toast-end">
          <div className="alert alert-success">
            {message}
          </div>
        </div>
      )}
    </>
  )
}
```

흐름:

```text
Save 클릭
   ↓
setMessage()
   ↓
Toast 표시
   ↓
3초
   ↓
setMessage("")
   ↓
Toast 제거
```

---

# 23. Alert vs Toast

둘의 차이를 명확하게 구분해야 합니다.

```text
Alert
→ 화면 안에 지속적으로 표시
→ 사용자가 반드시 확인해야 할 정보

Toast
→ 일시적으로 표시
→ 작업 결과 같은 짧은 Feedback
```

예:

```text
회원가입 Form Validation 오류
→ Alert 또는 Field Error

상품이 장바구니에 추가됨
→ Toast
```

---

# 24. Loading

비동기 작업 중에는 사용자에게 진행 중임을 알려야 합니다.

```html
<span class="loading loading-spinner"></span>
```

다양한 형태의 Loading Component를 사용할 수 있습니다.

```text
loading-spinner
loading-dots
loading-ring
loading-ball
loading-bars
loading-infinity
```

---

# 25. Loading + Button

실전에서 자주 사용하는 패턴입니다.

```jsx
<button
  className="btn btn-primary"
  disabled={saving}
>
  {saving && (
    <span
      className="
        loading
        loading-spinner
        loading-sm
      "
    />
  )}

  {saving
    ? '저장 중...'
    : '저장'}
</button>
```

흐름:

```text
saving = false
     ↓
[ 저장 ]

클릭

saving = true
     ↓
disabled
+
spinner
+
저장 중...
```

---

# 26. Loading의 역할 분담

```text
React / Query Library
→ isLoading / isFetching / saving

HTML
→ disabled

daisyUI
→ loading-spinner / Button UI
```

즉 Loading 상태 자체를 daisyUI가 만드는 것이 아닙니다.

---

# 27. Skeleton

데이터가 로딩 중일 때 화면 구조를 미리 보여주기 위해 Skeleton UI를 사용할 수 있습니다.

개념:

```text
Loading 전

┌────────────────────────┐
│ ███████████████        │
│ █████████              │
│                        │
│ ████████               │
└────────────────────────┘

        ↓

Data Load 완료

┌────────────────────────┐
│ Mechanical Keyboard    │
│ ₩129,000               │
│                        │
│ [ 구매하기 ]           │
└────────────────────────┘
```

Skeleton은 사용자가 페이지 구조를 미리 예측할 수 있게 합니다.

---

# 28. Spinner vs Skeleton

둘은 목적이 다릅니다.

```text
Spinner
→ 작업이 진행 중임을 알려줌

Skeleton
→ 로딩 후 나타날 Layout을 미리 표현
```

추천:

```text
작은 Button Action
→ Spinner

Page / Card Loading
→ Skeleton
```

---

# 29. Collapse

Collapse는 필요할 때 내용을 펼치고 접는 Component입니다.

```html
<div class="collapse bg-base-100 border">
  <input type="checkbox" />

  <div class="collapse-title font-semibold">
    배송 정보
  </div>

  <div class="collapse-content">
    기본 배송은 2~3일이 소요됩니다.
  </div>
</div>
```

구조:

```text
collapse
│
├─ Trigger State
├─ collapse-title
└─ collapse-content
```

---

# 30. Collapse 활용

```text
FAQ
상세 설명
필터 옵션
설정 영역
배송 정보
상품 상세 정보
```

에 사용할 수 있습니다.

기본적인 Open/Close State는 HTML/CSS와 daisyUI 구조를 활용할 수 있습니다.

---

# 31. Accordion

여러 Collapse를 그룹화해 한 번에 하나의 항목만 열리도록 구성하면 Accordion UI를 만들 수 있습니다.

개념:

```text
FAQ

▼ 배송 기간
  2~3일 소요됩니다.

▶ 환불 정책

▶ 교환 방법
```

Accordion은:

```text
FAQ
설정 Panel
도움말
Category Detail
```

등에 많이 사용됩니다.

---

# 32. Collapse vs Modal

둘 다 내용을 숨겼다가 보여주지만 목적이 다릅니다.

```text
Collapse
→ 현재 Layout 안에서 펼침
→ Context 유지

Modal
→ 화면 위에 Overlay
→ 사용자 집중
```

예:

```text
FAQ 답변
→ Collapse

삭제 확인
→ Modal
```

---

# 33. Dropdown vs Collapse

이 둘도 구분해야 합니다.

```text
Dropdown
→ 작은 선택/Action 목록
→ Trigger 근처 Overlay

Collapse
→ 현재 Layout 안의 내용 확장
```

예:

```text
프로필 메뉴
→ Dropdown

상품 상세 설명
→ Collapse
```

---

# 34. 실제 사용자 Action 흐름

상품 삭제를 예로 들어보겠습니다.

```text
[ 삭제 ]
   ↓
Modal Open
   ↓
"정말 삭제하시겠습니까?"
   ↓
[ 삭제 확인 ]
   ↓
API Request
   ↓
Loading
   ↓
┌───────────────┐
│ Success       │
│ 또는 Error    │
└───────────────┘
   ↓
Toast
```

여러 Component가 하나의 흐름에서 함께 사용됩니다.

---

# 35. React + Query Library와 연결

예를 들어 Mutation을 사용한다고 하겠습니다.

```text
사용자 클릭
    ↓
React Event
    ↓
Mutation 실행
    ↓
isLoading
    ↓
Loading Button
    ↓
Success?
 ┌───┴───┐
Yes      No
 │        │
 ▼        ▼
Toast   Alert/Toast
Success    Error
```

daisyUI는 여기에서 UI를 담당합니다.

---

# 36. RTK Query 예시

개념적으로:

```jsx
const [
  deleteProduct,
  {
    isLoading,
    isError,
    isSuccess,
  },
] = useDeleteProductMutation()
```

그리고:

```jsx
<button
  className="btn btn-error"
  disabled={isLoading}
>
  {isLoading && (
    <span className="loading loading-spinner" />
  )}

  삭제
</button>
```

결과에 따라 Toast를 표시할 수 있습니다.

```text
RTK Query
→ Mutation State

React
→ 조건부 Rendering

daisyUI
→ Modal / Loading / Toast
```

---

# 37. 실전 삭제 Modal 흐름

```text
Product Card
    │
    ▼
[ 삭제 ]
    │
    ▼
┌──────────────────────────┐
│ Modal                    │
│ 정말 삭제하시겠습니까?  │
│                          │
│ [취소]     [삭제]        │
└──────────────────────────┘
                 │
                 ▼
             Loading
                 │
                 ▼
               API
                 │
          ┌──────┴──────┐
          ▼             ▼
       Success         Error
          │             │
          ▼             ▼
   Success Toast    Error Toast
```

이것이 Overlay와 Feedback Component를 실제 애플리케이션에서 사용하는 대표 패턴입니다.

---

# 38. 언제 어떤 Component를 사용할까?

| Component | 주요 목적                |
| --------- | -------------------- |
| Modal     | 중요한 Action / Dialog  |
| Dropdown  | 짧은 Menu / Action 목록  |
| Tooltip   | 짧은 보조 설명             |
| Alert     | 지속적인 Feedback / 상태   |
| Toast     | 일시적인 Action 결과       |
| Loading   | 비동기 작업 진행 상태         |
| Skeleton  | 데이터 로딩 Layout        |
| Collapse  | Layout 내부 Content 펼침 |
| Accordion | 여러 Collapse 그룹       |

---

# 39. 선택 기준

Component의 모양보다 **사용자 경험의 목적**을 기준으로 선택합니다.

```text
사용자가 반드시 집중해야 하는가?
→ Modal

짧은 Action 목록인가?
→ Dropdown

보조 설명인가?
→ Tooltip

계속 보여줘야 하는 정보인가?
→ Alert

잠깐 알려주면 되는가?
→ Toast

작업이 진행 중인가?
→ Loading

데이터 Layout을 기다리는가?
→ Skeleton

현재 화면 안에서 내용을 펼칠 것인가?
→ Collapse
```

---

# 40. PART 7 전체 역할 분담

```text
Application Logic
│
├─ React State
├─ Mutation State
├─ Timer
└─ Event
       │
       ▼
     상태 결정
       │
       ▼
daisyUI Component
│
├─ Modal
├─ Dropdown
├─ Alert
├─ Toast
├─ Loading
└─ Collapse
       │
       ▼
사용자에게 Action / Feedback 표현
```

---

# 41. 접근성도 함께 생각해야 한다

Overlay Component는 접근성에 특히 주의해야 합니다.

예:

```text
Modal
→ Keyboard Focus
→ ESC / Close
→ 적절한 Label

Tooltip
→ Hover만 의존하지 않기

Dropdown
→ Keyboard Navigation 고려

Loading
→ 작업 중임을 사용자에게 전달
```

daisyUI를 사용한다고 해서 모든 접근성 요구사항이 자동으로 완성되는 것은 아닙니다.

HTML Semantic 구조와 실제 Interaction을 함께 확인해야 합니다.

---

# 42. PART 7 핵심 정리

Overlay:

```text
Modal
Dropdown
Tooltip
```

Feedback:

```text
Alert
Toast
Loading
Skeleton
```

Expandable UI:

```text
Collapse
Accordion
```

실전 역할 분담:

```text
React / Query Library
→ 상태와 동작

HTML / Browser API
→ Dialog 등의 기본 동작

daisyUI
→ UI 표현

Tailwind CSS
→ Layout / Position / 세부 Style
```

최종 흐름:

```text
사용자 Action
     ↓
Application Logic
     ↓
State 변경
     ↓
┌──────────────┬───────────────┐
▼              ▼               ▼
Overlay      Feedback       Loading
Modal        Alert/Toast    Spinner
Dropdown                    Skeleton
     │              │               │
     └──────────────┼───────────────┘
                    ▼
             사용자 경험 완성
```

> **PART 7의 핵심은 Component 이름을 외우는 것이 아니라, 사용자의 Action과 현재 상태에 따라 어떤 Overlay 또는 Feedback UI가 가장 적절한지 선택하는 것입니다.**

다음 PART에서는 지금까지 배운 Component들을 조합해서 **실전 쇼핑몰 / 관리자 Dashboard / 게시판 UI를 구성하는 방법**으로 넘어가면 좋습니다.
