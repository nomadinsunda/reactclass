
```
import React from 'react'
import {Droppable} from '@hello-pangea/dnd'

export const CardDroppable = ({ droppableId, children }) => {
  return (
    <Droppable droppableId={droppableId} type="card">
      {(provided, snapshot) => (
        <div
          {...provided.droppableProps}
          ref={provided.innerRef}          
          className={`flex flex-col p-2 min-h-[150px] transition-colors duration-200 rounded-lg ${
            snapshot.isDraggingOver ? 'bg-blue-100/50' : 'bg-transparent'
          }`}
        >
          {children}
          {provided.placeholder}
        </div>
      )}
    </Droppable>
  )
}
```

이 코드는 **`@hello-pangea/dnd`의 `<Droppable>`을 “카드가 떨어질 수 있는 영역(리스트/컬럼)”으로 감싸는 래퍼 컴포넌트**입니다. <br>
핵심은 <br>
**(1) provided로 droppable을 DOM에 “연결(wire)”** 하고, <br>
**(2) snapshot으로 드래그 상태에 따른 UI 피드백**을 주는 구조입니다.

---

## 1) 전체 구조 한눈에 보기 🧠

* `CardDroppable`은 `droppableId`를 받아서
* 내부에서 `<Droppable droppableId=... type="card">`를 만들고
* **render props 패턴**으로 `(provided, snapshot) => (...)` 함수를 통해

  * Droppable이 요구하는 props/ref를 DOM에 주입하고
  * 드래그 상태(snapshot)를 기반으로 스타일을 바꿉니다.

---

## 2) 코드 라인별로 “역할” 해부 🔧

### ✅ `Droppable` 선언부

```jsx
<Droppable droppableId={droppableId} type="card">
```

* **`droppableId`**

  * 이 Droppable 영역의 “고유 ID”입니다.
  * DnD 엔진이 “지금 어떤 드롭 존 위에 있는지”를 판단할 때 키로 씁니다.
  * 보통 리스트/컬럼 id를 넣습니다. (예: `"list-1"`, `"todo"`, UUID 등)

* **`type="card"`**

  * **드롭 가능한 Draggable의 종류를 제한/구분**하는 값입니다.
  * 같은 `type`끼리만 이동이 가능해집니다.
  * 예를 들어:

    * 리스트 자체를 드래그하는 Droppable은 `type="list"`
    * 카드 드래그는 `type="card"`
  * 이렇게 분리하면 “카드는 카드 영역끼리만”, “리스트는 리스트 영역끼리만” 같은 룰이 깔끔해집니다.

---

## 3) `(provided, snapshot)`이 의미하는 것

```jsx
{(provided, snapshot) => ( ... )}
```

### A) `provided` = “Droppable을 DOM에 붙이는 도구 세트”

여기서 **꼭** 해야 하는 3가지가 있습니다.

#### 1) `...provided.droppableProps`
Droppable 컴포넌트가 DOM에 반드시 붙여 달라고 요구하는 속성 묶음

```jsx
{...provided.droppableProps}
```

는 결국 아래와 같은 의미입니다 👇

```
<div
  data-rfd-droppable-id="list-1"
  data-rfd-droppable-context-id="0"
  onDragEnter={...}
  onDragOver={...}
  onDrop={...}
  /* 기타 내부적으로 필요한 속성들 */
>

```
👉 DnD 엔진이 “이 div가 drop zone이다”라고 인식하기 위한 신호들입니다.

* 드롭 영역이 작동하려면 필요한 **필수 props(이벤트/속성)** 들을 붙여줍니다.
* 이걸 빼면 드롭 판정이 제대로 안 되거나 경고/오작동이 납니다.

#### 2) `ref={provided.innerRef}`

```jsx
ref={provided.innerRef}
```

* 라이브러리가 **해당 DOM 노드의 크기/위치/스크롤** 정보를 읽어야 합니다.
* 그래서 “이 div가 droppable root야”라고 ref로 연결해주는 겁니다.
* 빼면 “droppable root를 찾을 수 없음” 류의 문제가 납니다.

#### 3) `provided.placeholder`

```jsx
{provided.placeholder}
```

* 이게 매우 중요합니다.
* 드래그로 아이템(카드)을 끌어올리면 원래 자리에서 카드가 빠지죠?
* 그러면 리스트 높이가 갑자기 줄어들어서 레이아웃이 흔들립니다.
* **placeholder는 “빠진 카드 자리만큼의 빈 공간”을 DOM에 유지**해서

  * 리스트 높이 유지
  * 다른 아이템들의 위치 계산 안정화
  * 드래그 중 레이아웃 점프 방지
    를 해줍니다.

> 결론: `droppableProps + innerRef + placeholder` 3종 세트는 사실상 “필수”입니다.

---

### B) `snapshot` = “현재 드래그 상태 스냅샷”

```jsx
snapshot.isDraggingOver
```

* **지금 드래그 중인 Draggable이 이 Droppable 위에 올라와 있으면 true**
* 그래서 아래처럼 “드롭 가능 상태”를 사용자에게 시각적으로 보여줍니다.

```jsx
snapshot.isDraggingOver ? 'bg-blue-100/50' : 'bg-transparent'
```

---

## 4) Tailwind 클래스 설계 의도 🎨

```jsx
className={`flex flex-col p-2 min-h-[150px] transition-colors duration-200 rounded-lg ${
  snapshot.isDraggingOver ? 'bg-blue-100/50' : 'bg-transparent'
}`}
```

* `flex flex-col`: 카드들이 세로로 쌓이도록
* `p-2`: 내부 패딩
* `min-h-[150px]`: **카드가 하나도 없을 때도 드롭 영역이 “보이게”**

  * 이거 없으면 빈 리스트는 높이가 0에 가까워져서 드롭하기 어려워집니다.
* `transition-colors duration-200`: 배경색 변화가 부드럽게
* `rounded-lg`: 모서리 둥글게
* `bg-blue-100/50`: 올라왔을 때 “하이라이트”
* `bg-transparent`: 평소에는 투명

---

## 5) `children`이 들어가는 위치의 의미

```jsx
{children}
{provided.placeholder}
```

* children이 **카드 리스트**일 가능성이 높고
* placeholder는 **항상 children 뒤**에 둡니다.

  * 그래야 “리스트 끝에 드롭” 같은 케이스에서 공간 계산이 안정적입니다.

---

## 6) 이 컴포넌트가 “정확히” 하는 일 요약

✅ `CardDroppable`은 “카드가 들어갈 수 있는 리스트 영역”을 만들고
✅ DnD 엔진이 필요로 하는 **필수 바인딩(Props/Ref/Placeholder)** 을 처리하며
✅ 드래그 오버 상태를 **snapshot** 으로 감지해 UI 피드백(배경색)을 줍니다.

---

## 7) 실무에서 자주 나는 실수 체크리스트 ✅

* [ ] `provided.innerRef`를 droppable root에 걸었는가?
* [ ] `...provided.droppableProps`를 root에 펼쳤는가?
* [ ] `{provided.placeholder}`를 렌더링했는가?
* [ ] `droppableId`가 리스트마다 **유일**한가?
* [ ] 같은 이동 그룹끼리 `type`이 맞는가?



