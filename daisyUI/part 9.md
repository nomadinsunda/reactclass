# PART 9. daisyUI Design System & 재사용 가능한 Component

## 1. 이번 PART의 목표

PART 8에서는 다음과 같은 실제 화면을 만들었습니다.

```text
쇼핑몰
관리자 Dashboard
게시판
```

프로젝트가 작을 때는 다음처럼 직접 daisyUI Class를 작성해도 문제가 없습니다.

```jsx
<button className="btn btn-primary">
  저장
</button>
```

하지만 애플리케이션이 커지면 같은 UI 규칙이 여러 곳에 반복됩니다.

```text
회원가입
상품 등록
주문 관리
게시글 작성
관리자 페이지
설정 화면
```

그리고 비슷한 Button이 수십 개 생깁니다.

```jsx
<button className="btn btn-primary">
  저장
</button>

<button className="btn btn-primary">
  등록
</button>

<button className="btn btn-primary">
  확인
</button>
```

이때 중요한 질문이 생깁니다.

> **어디까지 daisyUI Class를 직접 사용하고, 언제 우리 프로젝트의 공통 Component로 추상화해야 하는가?**

PART 9에서는 이 문제를 다룹니다.

---

# 2. Design System이란?

Design System은 단순히 색상 모음이나 Component Library를 의미하지 않습니다.

애플리케이션 전체에서 사용할:

```text
Color
Typography
Spacing
Component
Interaction
State
Accessibility
```

등의 **일관된 UI 규칙**을 정의하는 시스템입니다.

개념적으로:

```text
Design System
│
├─ Theme
│   ├─ primary
│   ├─ secondary
│   └─ base
│
├─ Component
│   ├─ Button
│   ├─ Input
│   ├─ Badge
│   └─ Modal
│
├─ Interaction
│   ├─ Loading
│   ├─ Disabled
│   └─ Error
│
└─ Layout Rules
    ├─ Spacing
    ├─ Responsive
    └─ Container
```

입니다.

---

# 3. daisyUI와 Design System의 관계

daisyUI는 이미 많은 기본 규칙을 제공합니다.

```text
daisyUI
│
├─ Theme
├─ Semantic Color
├─ Button
├─ Card
├─ Input
├─ Modal
├─ Badge
└─ ...
```

따라서 처음부터 모든 UI Component를 직접 만들 필요가 없습니다.

하지만 실제 프로젝트에서는 여기에 **우리 프로젝트의 규칙**을 추가하게 됩니다.

```text
daisyUI
      ↓
Base Component System
      ↓
Project Rule
      ↓
MyApp Design System
```

예:

```text
daisyUI
btn

       +

우리 프로젝트 규칙
"Primary Button에는 loading 지원"
"Delete Button은 error Variant"
"Button Size는 sm / md / lg만 사용"

       ↓

AppButton
```

---

# 4. 가장 먼저 Theme을 Design Token으로 사용한다

PART 3에서 Custom Theme을 만들었습니다.

```css
@plugin "daisyui/theme" {
  name: "myapp";
  default: true;
  color-scheme: light;

  --color-primary: ...;
  --color-secondary: ...;
  --color-accent: ...;
  --color-base-100: ...;
}
```

이것이 Design System의 첫 번째 기반입니다.

애플리케이션에서 직접:

```text
#2563eb
#7c3aed
#10b981
```

를 반복하기보다:

```text
primary
secondary
accent
```

라는 Semantic Role을 사용합니다.

즉:

```text
Brand Color
    ↓
Theme Token
    ↓
Component
```

입니다.

---

# 5. 공통 Button Component가 필요한가?

다음 코드가 있다고 하겠습니다.

```jsx
<button className="btn btn-primary">
  저장
</button>
```

단순히 Class가 반복된다는 이유만으로 반드시 Component화해야 하는 것은 아닙니다.

예를 들어:

```jsx
<button className="btn btn-primary">
  저장
</button>
```

이 코드가 충분히 명확하다면 그대로 사용하는 것도 좋습니다.

그러나 다음 요구가 반복된다면 이야기가 달라집니다.

```text
Loading 지원
Disabled 처리
Variant 제한
Size 제한
공통 Icon 처리
접근성 규칙
공통 Event 처리
```

이때는 재사용 Component의 가치가 커집니다.

---

# 6. AppButton 만들기

예:

```jsx
function AppButton({
  children,
  variant = 'primary',
  size = 'md',
  loading = false,
  disabled = false,
  className = '',
  ...props
}) {
  const variantClass = {
    primary: 'btn-primary',
    secondary: 'btn-secondary',
    error: 'btn-error',
    neutral: 'btn-neutral',
  }

  const sizeClass = {
    sm: 'btn-sm',
    md: 'btn-md',
    lg: 'btn-lg',
  }

  return (
    <button
      className={`
        btn
        ${variantClass[variant]}
        ${sizeClass[size]}
        ${className}
      `}
      disabled={loading || disabled}
      {...props}
    >
      {loading && (
        <span className="loading loading-spinner loading-sm" />
      )}

      {children}
    </button>
  )
}
```

사용:

```jsx
<AppButton>
  저장
</AppButton>
```

```jsx
<AppButton variant="error">
  삭제
</AppButton>
```

```jsx
<AppButton loading>
  저장
</AppButton>
```

---

# 7. AppButton의 역할

이제 Project 전체에 다음 규칙이 생겼습니다.

```text
AppButton
│
├─ daisyUI btn 사용
├─ 지원 Variant 제한
├─ 지원 Size 제한
├─ Loading 처리
└─ Disabled 처리
```

즉:

```text
daisyUI Button
      ↓
Project 규칙 추가
      ↓
AppButton
```

입니다.

---

# 8. 모든 Button을 AppButton으로 바꿔야 하는가?

아닙니다.

다음처럼 단순한 Button은:

```jsx
<button className="btn btn-ghost">
  닫기
</button>
```

그대로 사용하는 것이 더 간단할 수도 있습니다.

추상화 기준은:

```text
단순히 Class가 반복되는가?
       ↓
그것만으로는 부족

공통 규칙이 존재하는가?
       ↓
Yes
       ↓
Component 추상화 고려
```

입니다.

---

# 9. 좋은 추상화와 나쁜 추상화

좋은 추상화:

```text
Loading 규칙
Variant 규칙
공통 접근성
공통 Interaction
```

나쁜 추상화:

```text
Button 한 줄을 감추기 위해
무조건 Component 생성
```

예:

```jsx
function BlueButton({ children }) {
  return (
    <button className="btn btn-primary">
      {children}
    </button>
  )
}
```

프로젝트에서 `BlueButton`이라는 이름은 좋지 않습니다.

왜냐하면 PART 3에서 배웠듯:

```text
primary ≠ blue
```

이기 때문입니다.

추천:

```text
PrimaryButton
AppButton variant="primary"
```

처럼 **역할 기반 이름**을 사용합니다.

---

# 10. Variant 중심으로 Component를 설계한다

UI Component의 Props는 실제 색상보다 역할을 표현하는 것이 좋습니다.

좋지 않은 예:

```jsx
<Button blue />
<Button red />
<Button green />
```

더 나은 예:

```jsx
<Button variant="primary" />
<Button variant="danger" />
<Button variant="success" />
```

왜냐하면:

```text
blue
red
green
→ 실제 Color 중심

primary
danger
success
→ Semantic Role 중심
```

이기 때문입니다.

Theme을 변경해도 Component의 의미는 유지됩니다.

---

# 11. Input Component 추상화

Form에서도 비슷한 문제가 발생합니다.

반복 코드:

```jsx
<label className="form-control">
  <span>Email</span>

  <input
    className="input w-full"
    type="email"
  />
</label>
```

Error가 추가되면:

```jsx
<label>
  <span>Email</span>

  <input
    className="input input-error w-full"
  />

  <span className="text-error text-sm">
    이메일을 입력하세요.
  </span>
</label>
```

이 패턴이 계속 반복될 수 있습니다.

---

# 12. FormField 만들기

예:

```jsx
function FormField({
  label,
  error,
  children,
}) {
  return (
    <fieldset className="fieldset">
      <legend className="fieldset-legend">
        {label}
      </legend>

      {children}

      {error && (
        <p className="text-error text-sm">
          {error}
        </p>
      )}
    </fieldset>
  )
}
```

사용:

```jsx
<FormField
  label="이메일"
  error={errors.email}
>
  <input
    className={`
      input w-full
      ${errors.email ? 'input-error' : ''}
    `}
  />
</FormField>
```

---

# 13. AppInput까지 만들 것인가?

다음처럼 만들 수도 있습니다.

```jsx
function AppInput({
  label,
  error,
  className = '',
  ...props
}) {
  return (
    <fieldset className="fieldset">
      <legend className="fieldset-legend">
        {label}
      </legend>

      <input
        className={`
          input w-full
          ${error ? 'input-error' : ''}
          ${className}
        `}
        {...props}
      />

      {error && (
        <p className="text-error text-sm">
          {error}
        </p>
      )}
    </fieldset>
  )
}
```

사용:

```jsx
<AppInput
  label="이메일"
  type="email"
  error={errors.email}
/>
```

하지만 여기에서도 너무 많은 기능을 한 Component에 몰아넣지 않도록 주의해야 합니다.

---

# 14. Composition을 우선한다

UI Component를 만들 때 가능한 한 **Composition**을 활용합니다.

예:

```jsx
<Card>
  <CardTitle>
    상품
  </CardTitle>

  <ProductInfo />

  <CardActions>
    <AppButton>
      구매하기
    </AppButton>
  </CardActions>
</Card>
```

하나의 거대한 Component에 수많은 Props를 넣는 방식:

```jsx
<ProductCard
  showBadge
  showPrice
  showRating
  showStock
  showAction
  showDescription
  ...
/>
```

보다 역할을 나눌 수 있습니다.

---

# 15. Props가 지나치게 많아지는 문제

예:

```jsx
<Button
  primary
  secondary
  outline
  ghost
  small
  large
  rounded
  circle
  loading
/>
```

이런 API는 문제가 많습니다.

```text
primary + secondary ?
small + large ?
outline + ghost ?
```

상호 충돌하는 Props가 생깁니다.

더 좋은 방식:

```jsx
<Button
  variant="primary"
  size="lg"
  appearance="outline"
  loading
/>
```

즉 **하나의 축에는 하나의 값**을 사용합니다.

---

# 16. Variant 설계

예:

```text
Button
│
├─ variant
│   ├─ primary
│   ├─ secondary
│   ├─ neutral
│   └─ danger
│
├─ size
│   ├─ sm
│   ├─ md
│   └─ lg
│
└─ appearance
    ├─ solid
    ├─ outline
    └─ ghost
```

이런 구조는 Component 사용자가 이해하기 쉽습니다.

---

# 17. Class Mapping

실제 daisyUI Class로 변환합니다.

```jsx
const variantMap = {
  primary: 'btn-primary',
  secondary: 'btn-secondary',
  neutral: 'btn-neutral',
  danger: 'btn-error',
}
```

```jsx
const sizeMap = {
  sm: 'btn-sm',
  md: 'btn-md',
  lg: 'btn-lg',
}
```

```jsx
const appearanceMap = {
  solid: '',
  outline: 'btn-outline',
  ghost: 'btn-ghost',
}
```

최종:

```jsx
className={`
  btn
  ${variantMap[variant]}
  ${sizeMap[size]}
  ${appearanceMap[appearance]}
`}
```

---

# 18. 문자열을 직접 만드는 방식의 한계

프로젝트가 커지면:

```jsx
className={`
  btn
  ${variant === 'primary' ? 'btn-primary' : ''}
  ${size === 'lg' ? 'btn-lg' : ''}
  ${loading ? 'opacity-70' : ''}
`}
```

같은 코드가 복잡해질 수 있습니다.

이때 Class 조합 Utility를 사용할 수 있습니다.

대표적인 예:

```text
clsx
classnames
```

---

# 19. `clsx` 사용 예

```jsx
import clsx from 'clsx'

function AppButton({
  variant = 'primary',
  size = 'md',
  loading,
  className,
  ...props
}) {
  return (
    <button
      className={clsx(
        'btn',
        {
          'btn-primary': variant === 'primary',
          'btn-error': variant === 'danger',
          'btn-sm': size === 'sm',
          'btn-lg': size === 'lg',
        },
        className,
      )}
      {...props}
    />
  )
}
```

`clsx`는 daisyUI 기능이 아닙니다.

```text
daisyUI
→ UI Class

clsx
→ Class 문자열 조합
```

입니다.

---

# 20. Tailwind Class 충돌 문제

예:

```jsx
<AppButton className="w-full mt-4" />
```

좋습니다.

하지만 Component 내부에서:

```text
w-auto
```

를 사용하고 외부에서:

```text
w-full
```

을 넣으면 어느 Class가 실제 적용될지 복잡해질 수 있습니다.

이것은 Tailwind Class 추상화에서 자주 발생하는 문제입니다.

따라서 공통 Component에서는:

> **어떤 Style을 Component가 책임지고, 어떤 Style을 외부에서 허용할 것인지**

를 정해야 합니다.

---

# 21. Component가 담당할 것과 외부에 맡길 것

예:

```text
AppButton 내부
────────────────────
Button Variant
Size
Loading
Disabled


사용하는 쪽
────────────────────
width
margin
position
Layout
```

예:

```jsx
<AppButton
  className="w-full mt-4"
>
  로그인
</AppButton>
```

이 구조가 자연스럽습니다.

```text
AppButton
→ Button 자체

w-full / mt-4
→ 주변 Layout
```

---

# 22. 공통 StatusBadge

PART 6에서도 사용했던 예입니다.

```jsx
const statusMap = {
  pending: {
    label: '대기',
    className: 'badge-warning',
  },

  shipping: {
    label: '배송 중',
    className: 'badge-info',
  },

  completed: {
    label: '완료',
    className: 'badge-success',
  },

  cancelled: {
    label: '취소',
    className: 'badge-error',
  },
}
```

```jsx
function StatusBadge({ status }) {
  const config = statusMap[status]

  return (
    <span className={`badge ${config.className}`}>
      {config.label}
    </span>
  )
}
```

사용:

```jsx
<StatusBadge status={order.status} />
```

---

# 23. StatusBadge의 장점

각 페이지에서 다음 코드를 반복하지 않아도 됩니다.

```jsx
status === 'completed'
  ? 'badge-success'
  : ...
```

UI 규칙이 한곳에 모입니다.

```text
Server Status
      ↓
StatusBadge
      ↓
UI Mapping
      ↓
일관된 Badge
```

상태 표현 규칙을 변경해도 한 곳만 수정하면 됩니다.

---

# 24. LoadingState Component

여러 페이지에서:

```jsx
if (isLoading) {
  return (
    <span className="loading loading-spinner" />
  )
}
```

를 반복할 수 있습니다.

공통 Component:

```jsx
function LoadingState({
  message = '불러오는 중...',
}) {
  return (
    <div className="flex flex-col items-center gap-3 p-10">
      <span className="loading loading-spinner loading-lg" />

      <p>
        {message}
      </p>
    </div>
  )
}
```

사용:

```jsx
if (isLoading) {
  return <LoadingState />
}
```

---

# 25. EmptyState Component

```jsx
function EmptyState({
  title,
  description,
  action,
}) {
  return (
    <div className="card bg-base-100">
      <div className="card-body items-center text-center">
        <h2 className="card-title">
          {title}
        </h2>

        {description && (
          <p className="opacity-70">
            {description}
          </p>
        )}

        {action}
      </div>
    </div>
  )
}
```

사용:

```jsx
<EmptyState
  title="상품이 없습니다."
  description="새로운 상품을 등록해보세요."
  action={
    <AppButton>
      상품 등록
    </AppButton>
  }
/>
```

---

# 26. ErrorState Component

```jsx
function ErrorState({
  message = '오류가 발생했습니다.',
  onRetry,
}) {
  return (
    <div className="alert alert-error">
      <span>
        {message}
      </span>

      {onRetry && (
        <button
          className="btn btn-sm"
          onClick={onRetry}
        >
          다시 시도
        </button>
      )}
    </div>
  )
}
```

실전에서는:

```text
LoadingState
EmptyState
ErrorState
```

가 거의 모든 데이터 화면에서 반복됩니다.

---

# 27. Async State UI 통일

이제 서버 데이터 화면을 다음처럼 만들 수 있습니다.

```jsx
if (isLoading) {
  return <LoadingState />
}

if (isError) {
  return (
    <ErrorState
      onRetry={refetch}
    />
  )
}

if (products.length === 0) {
  return (
    <EmptyState
      title="상품이 없습니다."
    />
  )
}

return (
  <ProductGrid products={products} />
)
```

구조:

```text
Server State
│
├─ Loading
│   └─ LoadingState
│
├─ Error
│   └─ ErrorState
│
├─ Empty
│   └─ EmptyState
│
└─ Success
    └─ Data UI
```

---

# 28. ConfirmModal 공통화

삭제 확인 Modal도 여러 곳에서 반복될 수 있습니다.

```text
상품 삭제
게시글 삭제
댓글 삭제
회원 삭제
```

공통 Component:

```jsx
function ConfirmModal({
  open,
  title,
  message,
  confirmText = '확인',
  cancelText = '취소',
  danger = false,
  loading = false,
  onConfirm,
  onCancel,
}) {
  if (!open) {
    return null
  }

  return (
    <dialog className="modal" open>
      <div className="modal-box">
        <h3 className="font-bold text-lg">
          {title}
        </h3>

        <p className="py-4">
          {message}
        </p>

        <div className="modal-action">
          <button
            className="btn"
            onClick={onCancel}
          >
            {cancelText}
          </button>

          <button
            className={
              danger
                ? 'btn btn-error'
                : 'btn btn-primary'
            }
            disabled={loading}
            onClick={onConfirm}
          >
            {loading && (
              <span className="loading loading-spinner loading-sm" />
            )}

            {confirmText}
          </button>
        </div>
      </div>
    </dialog>
  )
}
```

---

# 29. 공통 Modal의 장점

이제:

```jsx
<ConfirmModal
  open={open}
  title="상품 삭제"
  message="정말 삭제하시겠습니까?"
  danger
  onConfirm={handleDelete}
  onCancel={close}
/>
```

처럼 사용할 수 있습니다.

규칙:

```text
Modal Layout
Button 위치
Loading
Danger 스타일
```

이 모두 일관됩니다.

---

# 30. Toast도 중앙화할 수 있다

프로젝트 곳곳에서:

```js
setMessage(...)
setTimeout(...)
```

을 반복하는 것은 관리하기 어렵습니다.

프로젝트가 커지면:

```text
Toast Provider
Toast Store
Toast Library
```

등으로 중앙화할 수 있습니다.

개념:

```text
Feature Component
       ↓
toast.success("저장되었습니다.")
       ↓
Toast System
       ↓
daisyUI toast + alert
```

입니다.

daisyUI가 Toast Lifecycle을 관리하는 것은 아닙니다.

---

# 31. UI Component Directory

React 프로젝트에서는 다음처럼 구성할 수 있습니다.

```text
src/
│
├─ components/
│   │
│   ├─ ui/
│   │   ├─ AppButton.jsx
│   │   ├─ AppInput.jsx
│   │   ├─ StatusBadge.jsx
│   │   ├─ ConfirmModal.jsx
│   │   ├─ LoadingState.jsx
│   │   ├─ ErrorState.jsx
│   │   └─ EmptyState.jsx
│   │
│   └─ layout/
│       ├─ Navbar.jsx
│       ├─ Sidebar.jsx
│       └─ Footer.jsx
│
└─ features/
    ├─ product/
    ├─ order/
    └─ user/
```

---

# 32. `ui`와 `feature` Component의 차이

`ui/`:

```text
AppButton
StatusBadge
ConfirmModal
LoadingState
```

특정 비즈니스 Domain을 몰라도 됩니다.

반면:

```text
ProductCard
OrderTable
UserProfile
```

은 Domain 지식을 가지고 있습니다.

따라서:

```text
ui
→ 범용 UI

feature
→ 비즈니스 기능
```

으로 생각할 수 있습니다.

---

# 33. 좋은 UI Component의 특징

좋은 공통 Component는:

```text
역할이 명확하다
Props가 예측 가능하다
Style 규칙이 일관된다
접근성을 고려한다
필요한 확장이 가능하다
```

반대로 좋지 않은 Component:

```text
모든 기능을 하나에 넣음
Props가 수십 개
특정 화면에 지나치게 종속
내부 Style 변경이 어려움
```

입니다.

---

# 34. Semantic Naming

이름도 Design System의 일부입니다.

좋은 이름:

```text
Primary
Secondary
Danger
Success
Muted
```

좋지 않은 이름:

```text
Blue
Purple
RedButton
GrayText
```

이유:

```text
Color 중심 이름
→ Theme 변화에 취약

Role 중심 이름
→ Theme과 독립적
```

입니다.

---

# 35. spacing도 규칙을 만든다

실전 프로젝트에서 다음이 모두 섞이면:

```text
gap-2
gap-3
gap-5
gap-7

p-3
p-5
p-9
```

화면의 리듬이 불규칙해질 수 있습니다.

프로젝트에서 자주 사용하는 간격 규칙을 정할 수 있습니다.

예:

```text
작은 간격
gap-2

일반 간격
gap-4

Section 간격
gap-6 / gap-8
```

이런 규칙은 반드시 코드로 강제할 필요는 없지만 **팀의 UI 규칙**으로 정할 수 있습니다.

---

# 36. Responsive 규칙도 통일한다

예:

```text
Mobile
1 Column

Tablet
2 Columns

Desktop
3~4 Columns
```

상품 Grid:

```jsx
<div className="
  grid
  grid-cols-1
  sm:grid-cols-2
  lg:grid-cols-3
  xl:grid-cols-4
  gap-6
">
```

같은 패턴을 프로젝트 전체에서 반복적으로 사용할 수 있습니다.

---

# 37. 접근성 규칙도 Design System에 포함된다

공통 Component를 만드는 중요한 이유 중 하나입니다.

예:

```text
Icon Button
→ aria-label 필수

Dialog
→ Focus 관리

Form
→ Label 연결

Loading
→ 상태 전달

Error
→ Text만으로도 의미 전달
```

이 규칙을 공통 Component 안에 넣으면 여러 화면에서 실수를 줄일 수 있습니다.

---

# 38. `className` 확장 허용

공통 Component를 너무 닫아두면 실전에서 사용하기 어렵습니다.

예:

```jsx
function AppButton({
  className = '',
  ...props
}) {
  return (
    <button
      className={`btn btn-primary ${className}`}
      {...props}
    />
  )
}
```

사용:

```jsx
<AppButton className="w-full mt-4">
  로그인
</AppButton>
```

하지만 `className`으로 내부 핵심 규칙까지 계속 덮어써야 한다면 추상화 설계를 다시 검토해야 합니다.

---

# 39. 공통 Component의 목표

목표는 Class를 감추는 것이 아닙니다.

```text
잘못된 목표

daisyUI Class가 안 보이게 만들자
```

가 아니라:

```text
좋은 목표

프로젝트의 반복되는 UI 규칙을
한 곳에서 일관되게 관리하자
```

입니다.

---

# 40. daisyUI를 그대로 사용할 영역

다음처럼 단순한 경우:

```jsx
<div className="divider" />

<span className="loading loading-spinner" />

<div className="badge badge-info">
  INFO
</div>
```

굳이 모두 공통 Component로 감쌀 필요는 없습니다.

daisyUI 자체가 이미 Component Class를 제공하고 있기 때문입니다.

---

# 41. 프로젝트 Component로 승격할 영역

반복되는 **Project Rule**이 생기면 승격을 고려합니다.

예:

```text
Button
+
Loading
+
Variant 제한
+
공통 접근성

        ↓

AppButton
```

또:

```text
Badge
+
Server Status Mapping
+
Label Mapping

        ↓

StatusBadge
```

입니다.

---

# 42. 추상화 단계

처음부터 Design System을 크게 만들 필요는 없습니다.

추천 흐름:

```text
1단계
daisyUI 직접 사용
        ↓
2단계
반복되는 패턴 발견
        ↓
3단계
공통 Component 추출
        ↓
4단계
Variant / Props 정리
        ↓
5단계
Design System 규칙화
```

즉:

> **먼저 사용하고, 반복되는 규칙이 보일 때 추상화합니다.**

---

# 43. 너무 이른 추상화의 문제

프로젝트 시작부터:

```text
Button System
Input System
Modal System
Card System
Grid System
Typography System
```

을 완벽하게 만들려고 하면 실제 요구사항을 알기 전에 복잡한 구조를 만들 수 있습니다.

이를 흔히 **Premature Abstraction** 문제로 볼 수 있습니다.

실제 UI가 만들어지고 반복 패턴이 보인 후 추상화하는 것이 더 현실적입니다.

---

# 44. 프로젝트가 커지는 과정

```text
처음

<button className="btn btn-primary">

          ↓

반복 발생

Button + Loading
Button + Variant
Button + Size

          ↓

AppButton

          ↓

여러 Feature에서 사용

          ↓

Project Design System
```

이 흐름이 자연스럽습니다.

---

# 45. 실제 프로젝트 구조 예

```text
src/
│
├─ components/
│   │
│   ├─ ui/
│   │   ├─ AppButton.jsx
│   │   ├─ AppInput.jsx
│   │   ├─ StatusBadge.jsx
│   │   ├─ ConfirmModal.jsx
│   │   ├─ LoadingState.jsx
│   │   ├─ EmptyState.jsx
│   │   └─ ErrorState.jsx
│   │
│   └─ layout/
│       ├─ AppNavbar.jsx
│       ├─ AppSidebar.jsx
│       └─ AppFooter.jsx
│
├─ features/
│   ├─ product/
│   │   ├─ ProductCard.jsx
│   │   └─ ProductList.jsx
│   │
│   ├─ order/
│   │   ├─ OrderTable.jsx
│   │   └─ OrderFilter.jsx
│   │
│   └─ auth/
│       └─ LoginForm.jsx
│
└─ layouts/
    └─ AppLayout.jsx
```

---

# 46. 전체 UI 계층

```text
                    Application
                         │
                         ▼
                   Design System
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
      Theme         UI Component       Rules
        │                │                │
   Semantic Color     Button          Spacing
   Base Color         Input           Responsive
   Content Color      Modal           Accessibility
        │             Badge              │
        └────────────────┼────────────────┘
                         ▼
                 Feature Components
                         │
          ┌──────────────┼───────────────┐
          ▼              ▼               ▼
       Product          Order           User
          │              │               │
          └──────────────┼───────────────┘
                         ▼
                     Application
```

---

# 47. PART 9 실전 체크리스트

공통 Component를 만들기 전에 확인합니다.

```text
□ 정말 반복되는 UI 규칙인가?

□ 단순한 daisyUI Class만 감추고 있지는 않은가?

□ Project의 Semantic Role을 표현하는가?

□ Props 간 충돌 가능성은 없는가?

□ Loading / Disabled 같은 공통 State가 있는가?

□ Layout은 사용하는 쪽에서 조절할 수 있는가?

□ className 확장이 필요한가?

□ 접근성 규칙을 포함할 수 있는가?

□ Domain Component와 범용 UI Component가 구분되는가?

□ Theme 변경에도 의미가 유지되는가?
```

---

# 48. PART 9 핵심 정리

daisyUI를 사용하는 초기 단계:

```text
HTML / JSX
   +
daisyUI Class
   +
Tailwind Utility
```

프로젝트가 커지면:

```text
daisyUI
   ↓
반복되는 UI 패턴
   ↓
Project Rule
   ↓
Reusable Component
   ↓
Design System
```

가 됩니다.

대표적인 공통 Component:

```text
AppButton
AppInput
StatusBadge
ConfirmModal
LoadingState
EmptyState
ErrorState
```

가장 중요한 원칙은:

```text
Class를 감추기 위해 추상화
        X

Project의 반복되는 규칙을
관리하기 위해 추상화
        O
```

입니다.

그리고:

```text
daisyUI Theme
      +
Reusable UI Component
      +
Tailwind Layout Rule
      +
Accessibility Rule
      ↓
Project Design System
```

으로 발전할 수 있습니다.

> **PART 9의 핵심은 daisyUI를 다시 만드는 것이 아니라, daisyUI를 기반으로 우리 프로젝트에서 반복되는 UI 규칙을 하나의 일관된 Design System으로 발전시키는 것입니다.**

다음 PART에서는 PART 1~9 전체 내용을 종합해서 **daisyUI를 실제 React 프로젝트에서 어떻게 설계하고 사용하는지 전체 아키텍처와 Best Practice를 최종 정리**하면 좋습니다.
