
## 1️⃣ `droppableProps`의 성격부터 한 문장으로

> **`droppableProps`는
> “이 DOM이 droppable 영역임을 표시하고,
> 드롭 판정과 접근성을 가능하게 하기 위한 속성 묶음”** 입니다.

---

## 2️⃣ 실제로 들어 있는 것들 (역할 기준 분류) 🔍

### 🔹 ① 식별용 데이터 속성 (가장 핵심)

```html
data-rfd-droppable-id="list-1"
data-rfd-droppable-context-id="0"
```

* **droppableId**

  * 이 DOM이 어떤 Droppable인지 식별
* **contextId**

  * 하나의 DndContext(DragDropContext) 안에서의 구분자
  * 다중 보드 / 중첩 DnD 구조에서 충돌 방지용

👉 라이브러리는 드래그 중에
<strong> DOM 트리를 탐색하면서 이 data-* 속성으로 hit-test 를 합니다. </strong>

---

### 🔹 ② 이벤트 처리 관련 속성

보통은 직접 체감하지 못하지만, 내부적으로는 다음과 같은 목적을 가집니다.

* drag over 상태 추적
* drop 가능 여부 판정
* 포인터 이동 추적

> ⚠️ React에서는 이벤트가 props 형태로 연결되므로
> 이런 핸들러들이 `droppableProps` 안에 들어옵니다.

---

### 🔹 ③ 접근성(ARIA) 관련 속성 ♿

DnD 라이브러리는 **키보드 드래그**를 공식 지원합니다.

그래서 droppable root에는 다음과 같은 성격의 속성이 포함됩니다.

```html
role="list"
aria-describedby="..."
```

(정확한 값은 상황에 따라 다르며 내부 구현에 의존)

👉 이 덕분에:

* Tab 이동
* Space / Arrow 키 드래그
* Screen reader 안내

가 가능합니다.

---

### 🔹 ④ 내부 상태 연결용 메타 정보

이건 개발자가 직접 쓸 일은 없지만 매우 중요합니다.

* 드래그 중인 draggable이

  * “이 droppable 위에 있는가?”
  * “type이 맞는가?”
  * “지금 들어올 수 있는 상태인가?”

를 판단하기 위한 **메타 정보**들이 props로 연결됩니다.

이 정보가 있어야:

```js
snapshot.isDraggingOver === true
```

가 정확히 계산됩니다.

---

## 3️⃣ 왜 `style`은 없을까? 🤔 (중요한 차이점)

여기서 많은 분들이 헷갈립니다.

### ❓ draggableProps에는 `style`이 있는데

### ❓ droppableProps에는 왜 없지?

### ✅ 이유는 “누가 움직이느냐”입니다.

| 구분        | 누가 움직이나      | style 필요 여부    |
| --------- | ------------ | -------------- |
| Draggable | **아이템이 움직임** | ✅ transform 필요 |
| Droppable | 컨테이너는 고정     | ❌ 불필요          |

* 위치 변화는 **항상 Draggable 쪽**에서 발생
* Droppable은 **판정 기준점**일 뿐

그래서 droppableProps에는 `style`이 없습니다.

---

## 4️⃣ 실제 DOM에서 보면 이런 느낌입니다 👀

개발자 도구로 보면 보통 이런 식입니다:

```html
<div
  data-rfd-droppable-id="todo"
  data-rfd-droppable-context-id="0"
  role="list"
>
  <!-- draggable items -->
</div>
```

👉 이 DOM을 보고 라이브러리는 말합니다:

> “아, 여긴 id가 todo인 droppable이구나”

---

## 5️⃣ 절대 의존하면 안 되는 것 ⚠️

중요한 경고입니다.

### ❌ 이렇게 쓰면 안 됩니다

```js
if (provided.droppableProps['data-rfd-droppable-id'] === 'todo') {
  ...
}
```

이유:

* `data-rfd-*` 이름은 **내부 구현**
* 버전 업 시 바뀔 수 있음
* 공식 API 아님

👉 **우리는 ‘존재한다’는 사실만 믿고,
‘내용을 읽거나 수정하지 않는다’가 원칙**입니다.

---

## 6️⃣ 한 문장 요약 (핵심 정리) 🧩

> **`droppableProps`에는**
>
> * droppable 식별용 data 속성
> * 드롭 판정용 이벤트 연결
> * 접근성(ARIA) 속성
> * 내부 상태 추적용 메타 정보
>
> 가 들어 있으며,
> **모두 `@hello-pangea/dnd`가 생성하고
> 개발자는 DOM에 그대로 펼치기만 하면 됩니다.**

