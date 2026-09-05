# PART 6. daisyUI Data Display Component

## 1. 이번 PART에서 배울 내용

웹 애플리케이션은 서버에서 가져온 데이터를 사용자에게 보여줘야 합니다.

예를 들어 쇼핑몰이라면:

```text
상품
가격
재고
주문 상태
카테고리
사용자 정보
```

관리자 Dashboard라면:

```text
매출
주문 수
회원 수
재고 현황
최근 주문
통계 데이터
```

등을 화면에 표현해야 합니다.

이번 PART에서는 이런 데이터를 표현하는 daisyUI Component를 다룹니다.

```text
Card
Stat
Table
Badge
Avatar
Timeline
Progress
Radial Progress
Kbd
Diff
```

핵심은 다음입니다.

> **daisyUI는 데이터를 관리하는 것이 아니라 데이터를 사용자에게 보여주는 UI 구조를 제공합니다.**

---

# 2. Data Display와 Data Fetching은 다르다

먼저 역할을 구분해야 합니다.

예를 들어 서버에서 상품 목록을 가져온다고 하겠습니다.

```text
Server
  ↓
GET /products
  ↓
RTK Query / TanStack Query
  ↓
React Component
  ↓
daisyUI
  ↓
Card / Table / Badge
```

여기서:

```text
RTK Query / TanStack Query
→ Server State 관리

React
→ 데이터 반복 렌더링

daisyUI
→ 데이터를 화면에 표현

Tailwind CSS
→ Layout / 간격 / Responsive 조정
```

입니다.

---

# 3. Card

Card는 관련된 정보를 하나의 시각적 단위로 묶어 보여주는 Component입니다.

```html
<div class="card bg-base-100 shadow-sm">
  <div class="card-body">
    <h2 class="card-title">
      Mechanical Keyboard
    </h2>

    <p>
      ₩129,000
    </p>

    <div class="card-actions justify-end">
      <button class="btn btn-primary">
        구매하기
      </button>
    </div>
  </div>
</div>
```

구조:

```text
card
│
└─ card-body
    │
    ├─ card-title
    ├─ Content
    └─ card-actions
```

Card는 다음처럼 사용할 수 있습니다.

```text
상품
게시글
프로필
공지사항
Dashboard Widget
결제 정보
```

---

# 4. Card에 Image 추가

상품 Card를 만들어보겠습니다.

```html
<div class="card bg-base-100 shadow-sm">
  <figure>
    <img
      src="/keyboard.jpg"
      alt="Mechanical Keyboard"
    />
  </figure>

  <div class="card-body">
    <h2 class="card-title">
      Mechanical Keyboard
    </h2>

    <p>
      ₩129,000
    </p>

    <button class="btn btn-primary">
      구매하기
    </button>
  </div>
</div>
```

구조:

```text
Card
│
├─ figure
│   └─ Image
│
└─ card-body
    ├─ Title
    ├─ Price
    └─ Button
```

여기서 `figure`와 `img`는 HTML Element입니다.

```text
card / card-body
→ daisyUI

figure / img
→ HTML

object-cover / h-48 등
→ Tailwind CSS
```

로 역할을 구분해야 합니다.

---

# 5. Card와 Tailwind Layout

실전에서는 Card Component 자체보다 **Card를 여러 개 배치하는 Layout**이 중요합니다.

```html
<div
  class="
    grid
    grid-cols-1
    md:grid-cols-2
    lg:grid-cols-3
    gap-6
  "
>
  <div class="card">...</div>
  <div class="card">...</div>
  <div class="card">...</div>
</div>
```

여기서:

```text
card
→ daisyUI

grid
grid-cols-1
md:grid-cols-2
lg:grid-cols-3
gap-6
→ Tailwind CSS
```

입니다.

즉:

```text
daisyUI
→ 개별 Card 디자인

Tailwind
→ 여러 Card 배치
```

입니다.

---

# 6. React에서 Card 반복 렌더링

상품 데이터가 있다고 하겠습니다.

```js
const products = [
  {
    id: 1,
    name: 'Keyboard',
    price: 129000,
  },
  {
    id: 2,
    name: 'Mouse',
    price: 59000,
  },
]
```

React에서는:

```jsx
<div className="grid grid-cols-1 md:grid-cols-2 gap-6">
  {products.map((product) => (
    <div
      key={product.id}
      className="card bg-base-100 shadow-sm"
    >
      <div className="card-body">
        <h2 className="card-title">
          {product.name}
        </h2>

        <p>
          {product.price.toLocaleString()}원
        </p>
      </div>
    </div>
  ))}
</div>
```

흐름:

```text
products[]
   ↓
map()
   ↓
Card Component 반복 생성
   ↓
Final UI
```

입니다.

daisyUI가 반복 렌더링을 하는 것이 아닙니다.

```text
React
→ map()

daisyUI
→ Card Style
```

입니다.

---

# 7. Badge

Badge는 짧은 상태나 Label을 표시할 때 사용합니다.

```html
<span class="badge badge-primary">
  NEW
</span>
```

예:

```text
NEW
SALE
BEST
재고 있음
배송 중
완료
```

Semantic Color와 함께 사용할 수 있습니다.

```html
<span class="badge badge-success">
  완료
</span>

<span class="badge badge-warning">
  대기
</span>

<span class="badge badge-error">
  실패
</span>
```

PART 3에서 배운 Theme 시스템과 연결됩니다.

---

# 8. Badge는 상태 그 자체가 아니다

중요한 구분입니다.

다음 코드에서:

```jsx
<span className="badge badge-success">
  완료
</span>
```

`badge-success`가 주문 상태를 관리하는 것은 아닙니다.

실제 상태는 데이터에 있습니다.

```js
order.status === 'completed'
```

그리고 React가:

```text
status
   ↓
조건 판단
   ↓
badge-success
   ↓
UI 표현
```

을 결정합니다.

예:

```jsx
const statusClass = {
  pending: 'badge-warning',
  completed: 'badge-success',
  failed: 'badge-error',
}

<span
  className={`badge ${statusClass[order.status]}`}
>
  {order.status}
</span>
```

---

# 9. Stat

Dashboard에서 숫자 정보를 강조할 때 `Stat` Component를 사용할 수 있습니다.

```html
<div class="stats shadow">
  <div class="stat">
    <div class="stat-title">
      Total Orders
    </div>

    <div class="stat-value">
      1,248
    </div>

    <div class="stat-desc">
      +12% from last month
    </div>
  </div>
</div>
```

구조:

```text
stats
│
└─ stat
    │
    ├─ stat-title
    ├─ stat-value
    └─ stat-desc
```

Dashboard KPI에 매우 유용합니다.

---

# 10. 여러 Stat 조합

```html
<div class="stats shadow">
  <div class="stat">
    <div class="stat-title">
      Revenue
    </div>
    <div class="stat-value">
      ₩28M
    </div>
  </div>

  <div class="stat">
    <div class="stat-title">
      Orders
    </div>
    <div class="stat-value">
      1,248
    </div>
  </div>

  <div class="stat">
    <div class="stat-title">
      Users
    </div>
    <div class="stat-value">
      8,420
    </div>
  </div>
</div>
```

화면 개념:

```text
┌─────────────┬─────────────┬─────────────┐
│ Revenue     │ Orders      │ Users       │
│ ₩28M        │ 1,248       │ 8,420       │
└─────────────┴─────────────┴─────────────┘
```

---

# 11. Stat와 Responsive

Mobile에서는 Stat를 세로로 배치할 수도 있습니다.

```html
<div class="stats stats-vertical lg:stats-horizontal shadow">
```

개념:

```text
Mobile
↓
Stat
Stat
Stat


Desktop
↓
Stat | Stat | Stat
```

여기에서도:

```text
stats-vertical
stats-horizontal
→ daisyUI

lg:
→ Tailwind Responsive Variant
```

로 이해하면 됩니다.

---

# 12. Table

많은 데이터를 행과 열로 보여줄 때 사용합니다.

```html
<div class="overflow-x-auto">
  <table class="table">
    <thead>
      <tr>
        <th>상품</th>
        <th>가격</th>
        <th>재고</th>
      </tr>
    </thead>

    <tbody>
      <tr>
        <td>Keyboard</td>
        <td>129,000원</td>
        <td>12</td>
      </tr>

      <tr>
        <td>Mouse</td>
        <td>59,000원</td>
        <td>24</td>
      </tr>
    </tbody>
  </table>
</div>
```

구조:

```text
table
│
├─ thead
│   └─ tr
│       └─ th
│
└─ tbody
    └─ tr
        └─ td
```

`table`, `thead`, `tbody`, `tr`, `th`, `td`는 HTML Table 구조입니다.

`table` Class가 daisyUI Style을 적용합니다.

---

# 13. Table과 Responsive

Table은 Mobile에서 가로 폭이 부족한 경우가 많습니다.

그래서 흔히:

```html
<div class="overflow-x-auto">
  <table class="table">
    ...
  </table>
</div>
```

처럼 감쌉니다.

```text
Table Width
    >
Viewport Width

       ↓

overflow-x-auto

       ↓

가로 Scroll
```

`overflow-x-auto`는 Tailwind Utility입니다.

---

# 14. React에서 Table 렌더링

```jsx
<tbody>
  {products.map((product) => (
    <tr key={product.id}>
      <td>{product.name}</td>
      <td>
        {product.price.toLocaleString()}원
      </td>
      <td>
        <span
          className={
            product.stock > 0
              ? 'badge badge-success'
              : 'badge badge-error'
          }
        >
          {product.stock > 0
            ? '재고 있음'
            : '품절'}
        </span>
      </td>
    </tr>
  ))}
</tbody>
```

흐름:

```text
products
   ↓
map()
   ↓
<tr>
   ↓
<td>
   ↓
Badge
```

입니다.

---

# 15. Table에 Action 추가

관리자 페이지에서는 각 Row마다 Action Button을 넣는 경우가 많습니다.

```html
<tr>
  <td>Keyboard</td>
  <td>129,000원</td>

  <td>
    <button class="btn btn-sm">
      수정
    </button>

    <button class="btn btn-error btn-sm">
      삭제
    </button>
  </td>
</tr>
```

즉:

```text
Table
│
└─ Row
    │
    ├─ Data
    └─ Actions
         ├─ Edit
         └─ Delete
```

가 됩니다.

---

# 16. Avatar

사용자의 Profile Image를 보여줄 때 사용합니다.

```html
<div class="avatar">
  <div class="w-12 rounded-full">
    <img
      src="/user.jpg"
      alt="User"
    />
  </div>
</div>
```

구조:

```text
avatar
│
└─ Image Container
    │
    └─ img
```

Tailwind Utility:

```text
w-12
rounded-full
```

등을 함께 사용할 수 있습니다.

---

# 17. Avatar + User Information

```html
<div class="flex items-center gap-3">
  <div class="avatar">
    <div class="w-10 rounded-full">
      <img src="/user.jpg" alt="User" />
    </div>
  </div>

  <div>
    <div class="font-bold">
      Kim
    </div>

    <div class="text-sm opacity-70">
      kim@example.com
    </div>
  </div>
</div>
```

이 패턴은:

```text
Navbar
Comment
Table
Chat
Profile
```

등에서 자주 사용됩니다.

---

# 18. Avatar Group

여러 사용자를 겹쳐 보여줄 수도 있습니다.

개념적으로:

```text
◯◯◯ +12
```

예:

```html
<div class="avatar-group -space-x-6">
  <div class="avatar">
    ...
  </div>

  <div class="avatar">
    ...
  </div>

  <div class="avatar placeholder">
    <div class="bg-neutral text-neutral-content">
      +12
    </div>
  </div>
</div>
```

프로젝트 참여자, 팀원, 구매자 등을 요약해서 보여주기 좋습니다.

---

# 19. Progress

작업 진행률을 보여줄 때 사용할 수 있습니다.

```html
<progress
  class="progress progress-primary"
  value="70"
  max="100"
></progress>
```

구조:

```text
value = 70
max = 100

      ↓

70%
```

여기서:

```text
value / max
→ HTML Attribute

progress
→ daisyUI

progress-primary
→ Theme Semantic Color
```

입니다.

---

# 20. React에서 Progress

```jsx
<progress
  className="progress progress-primary"
  value={progress}
  max="100"
/>
```

React State:

```text
progress = 30
    ↓
30%

progress = 70
    ↓
70%

progress = 100
    ↓
100%
```

즉 daisyUI는 값을 계산하지 않습니다.

```text
Application State
→ progress 값

HTML
→ value / max

daisyUI
→ Progress UI
```

---

# 21. Radial Progress

원형 Progress UI도 사용할 수 있습니다.

```html
<div
  class="radial-progress text-primary"
  style="--value:70;"
  role="progressbar"
>
  70%
</div>
```

개념:

```text
     70%
   ◜─────◝
  │       │
   ◟─────◞
```

Dashboard나 Achievement UI에서 유용합니다.

---

# 22. Timeline

시간 순서대로 Event를 보여줄 때 사용합니다.

예:

```text
주문 생성
   ↓
결제 완료
   ↓
배송 시작
   ↓
배송 완료
```

다음과 같은 데이터를 표현하기 좋습니다.

```text
주문 처리 과정
프로젝트 이력
사용자 활동
회사 연혁
배포 History
```

---

# 23. Timeline의 핵심

Timeline에서 중요한 것은 Event의 순서입니다.

```text
Event A
   │
   ▼
Event B
   │
   ▼
Event C
```

React라면:

```js
const events = [
  {
    id: 1,
    title: '주문 생성',
  },
  {
    id: 2,
    title: '결제 완료',
  },
]
```

을:

```text
events[]
   ↓
map()
   ↓
Timeline Item
```

으로 표현할 수 있습니다.

---

# 24. Kbd

Keyboard Shortcut이나 Key 입력을 표시할 때 사용합니다.

```html
<kbd class="kbd">
  Ctrl
</kbd>

+

<kbd class="kbd">
  K
</kbd>
```

결과:

```text
[ Ctrl ] + [ K ]
```

다음과 같은 UI에서 유용합니다.

```text
검색 Shortcut
Command Palette
Editor
Help
Keyboard Guide
```

---

# 25. Diff

두 이미지를 비교해서 Before / After를 보여주는 UI가 필요할 수도 있습니다.

개념:

```text
Before          After
───────|────────────
       ↑
    Slider
```

예:

```text
이미지 보정 전 / 후
디자인 변경 전 / 후
상품 Before / After
```

등에 사용할 수 있습니다.

---

# 26. 어떤 Component를 선택할까?

| Component       | 적합한 데이터             |
| --------------- | ------------------- |
| Card            | 개별 Entity / Summary |
| Badge           | 짧은 상태 / Label       |
| Stat            | KPI / 핵심 숫자         |
| Table           | 행/열 구조의 많은 데이터      |
| Avatar          | 사용자 / Profile       |
| Progress        | 진행률                 |
| Radial Progress | 강조된 진행률             |
| Timeline        | 시간 순서 Event         |
| Kbd             | Keyboard Shortcut   |
| Diff            | Before / After 비교   |

중요한 것은:

> **데이터 모양이 아니라 사용자가 무엇을 이해해야 하는지에 따라 Component를 선택하는 것**입니다.

---

# 27. Card vs Table

같은 상품 데이터도 상황에 따라 표현 방법이 달라집니다.

쇼핑몰 고객 화면:

```text
[ Image ]
Keyboard
129,000원
[ 구매하기 ]
```

→ Card가 자연스럽습니다.

관리자 화면:

```text
상품      가격       재고      상태
Keyboard 129,000    12        판매중
Mouse     59,000     0        품절
```

→ Table이 자연스럽습니다.

즉:

```text
Card
→ 개별 항목을 시각적으로 강조

Table
→ 여러 항목을 빠르게 비교
```

입니다.

---

# 28. Stat vs Card

Stat 역시 Card와 비슷해 보이지만 목적이 다릅니다.

```text
Card
→ 하나의 Entity나 Content 묶음

Stat
→ 하나의 핵심 Metric 강조
```

예:

```text
Product Card
Keyboard
129,000원
```

vs:

```text
Total Revenue
₩28,400,000
+12%
```

입니다.

---

# 29. Data Loading 상태

실제 서버 데이터를 표시할 때는 Loading 상태도 고려해야 합니다.

```text
Request 시작
   ↓
Loading
   ↓
Success / Error
```

예:

```jsx
if (isLoading) {
  return (
    <span className="loading loading-spinner" />
  )
}
```

데이터가 도착하면:

```jsx
return (
  <div className="card">
    ...
  </div>
)
```

이 됩니다.

---

# 30. Empty State

데이터가 없는 경우도 고려해야 합니다.

예:

```text
주문 내역이 없습니다.
```

단순히 빈 Table을 보여주는 것보다:

```text
┌─────────────────────────┐
│                         │
│ 주문 내역이 없습니다.   │
│                         │
│ [ 상품 보러가기 ]       │
│                         │
└─────────────────────────┘
```

처럼 안내하는 것이 좋습니다.

daisyUI Component를 조합해서 만들 수 있습니다.

```text
Card
Text
Button
```

---

# 31. Error State

API 요청이 실패했을 수도 있습니다.

```jsx
if (isError) {
  return (
    <div className="alert alert-error">
      데이터를 불러오지 못했습니다.
    </div>
  )
}
```

즉 실제 Data Display는 단순히 데이터를 보여주는 것만이 아닙니다.

```text
Loading
Empty
Error
Success
```

네 가지 상태를 함께 생각해야 합니다.

---

# 32. Data Display의 전체 상태

```text
             API Request
                  │
                  ▼
               Loading
                  │
           ┌──────┴──────┐
           │             │
           ▼             ▼
        Success         Error
           │
     ┌─────┴─────┐
     ▼           ▼
  데이터 있음   데이터 없음
     │           │
     ▼           ▼
 Card/Table    Empty UI
```

실제 프로젝트에서는 이 흐름을 항상 고려해야 합니다.

---

# 33. 실전 Dashboard 예제

관리자 Dashboard를 구성해보겠습니다.

```text
┌────────────────────────────────────────────┐
│ Navbar                                     │
├────────────┬───────────────────────────────┤
│ Sidebar    │                               │
│            │ [ Revenue ] [ Orders ]        │
│            │ [ Users   ] [ Products ]      │
│            │                               │
│            │ Recent Orders                 │
│            │ ┌───────────────────────────┐ │
│            │ │ Table                     │ │
│            │ └───────────────────────────┘ │
│            │                               │
└────────────┴───────────────────────────────┘
```

여기서:

```text
Stat
→ Revenue / Orders / Users

Table
→ Recent Orders

Badge
→ Order Status

Avatar
→ User

Progress
→ 목표 달성률
```

을 사용할 수 있습니다.

---

# 34. 실전 Product Page

쇼핑몰에서는:

```text
Product List
│
├─ Card
│   ├─ Image
│   ├─ Badge
│   ├─ Title
│   ├─ Price
│   └─ Button
│
├─ Card
│
└─ Card
```

상품 상세에서는:

```text
Breadcrumbs
│
├─ Product Image
├─ Title
├─ Badge
├─ Price
├─ Rating
├─ Progress
└─ Button
```

처럼 조합할 수 있습니다.

---

# 35. React에서 Component 분리

실전 프로젝트에서는 Data Display Component를 분리하는 것이 좋습니다.

```text
components/
│
├─ ProductCard.jsx
├─ OrderTable.jsx
├─ StatusBadge.jsx
├─ UserAvatar.jsx
├─ DashboardStat.jsx
└─ LoadingState.jsx
```

예:

```jsx
function StatusBadge({ status }) {
  const classMap = {
    pending: 'badge-warning',
    completed: 'badge-success',
    failed: 'badge-error',
  }

  return (
    <span
      className={`badge ${classMap[status]}`}
    >
      {status}
    </span>
  )
}
```

이렇게 하면 UI 규칙을 재사용할 수 있습니다.

---

# 36. Data Display와 Server State

서버 데이터 기반 화면은 다음처럼 생각하면 됩니다.

```text
RTK Query / TanStack Query
           │
           ▼
       Server State
           │
           ▼
         React
           │
     ┌─────┼─────┐
     ▼     ▼     ▼
 Loading Error   Data
                 │
                 ▼
              daisyUI
           ┌─────┼─────┐
           ▼     ▼     ▼
          Card  Table  Stat
```

daisyUI가 Server State를 관리하지 않는다는 점이 중요합니다.

---

# 37. 역할을 정확하게 구분하자

```text
Server / API
→ 실제 데이터

RTK Query / TanStack Query
→ Server State

React
→ 조건부 Rendering / 반복 Rendering

daisyUI
→ Data Display Component

Tailwind CSS
→ Layout / Responsive / 세부 Style
```

이 구조가 PART 6에서 가장 중요합니다.

---

# 38. PART 6 전체 구조

```text
                         Data
                          │
              ┌───────────┼───────────┐
              │           │           │
              ▼           ▼           ▼
          Entity        Metric       Status
              │           │           │
          Card/Table      Stat       Badge
              │           │           │
              └───────────┼───────────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
            User        Progress     History
              │           │           │
           Avatar       Progress    Timeline
                          │
                          ▼
                       Final UI
```

---

# 39. PART 6 핵심 정리

대표 Component:

```text
Card
→ 개별 데이터

Table
→ 많은 데이터 비교

Stat
→ 핵심 Metric

Badge
→ 상태

Avatar
→ 사용자

Progress
→ 진행률

Timeline
→ 시간 흐름
```

그리고 실제 애플리케이션은 반드시 다음 상태를 고려합니다.

```text
Loading
Error
Empty
Success
```

최종 역할 분담:

```text
Server
→ Data

Query Library
→ Server State

React
→ Rendering Logic

daisyUI
→ Data Display UI

Tailwind CSS
→ Layout
```

> **PART 6의 핵심은 “어떤 데이터를 가지고 있는가?”보다 “사용자가 이 데이터를 어떤 방식으로 이해해야 하는가?”를 먼저 생각하고 적절한 Data Display Component를 선택하는 것입니다.**

다음 PART에서는 이렇게 표시된 데이터를 기반으로 **Modal, Dialog, Toast, Dropdown 등 사용자의 Action과 Feedback을 처리하는 Overlay / Feedback Component**를 다루는 흐름이 좋습니다.
