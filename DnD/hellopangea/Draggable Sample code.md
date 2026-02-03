```
import React from 'react'
import {Draggable} from '@hello-pangea/dnd'

export const CardDraggable = ({ draggableId, index, children }) => {
  return (
    <Draggable draggableId={draggableId} index={index}>
      {(provided, snapshot) => {
        return (
          <div
            ref={provided.innerRef}
            {...provided.draggableProps}
            {...provided.dragHandleProps}
            // 1. Snapshot을 이용해 드래그 중일 때의 스타일을 제어할 수 있습니다.
            style={{
              ...provided.draggableProps.style,
              cursor: 'grab', // 마우스 커서를 잡기 모양으로 변경
              opacity: snapshot.isDragging ? 0.8 : 1 // 드래그 중일 때 살짝 투명하게
            }}
          >
            {children}
          </div>
        )
      }}
    </Draggable>
  )
}
```

이 코드는 `@hello-pangea/dnd`에서 **“카드 하나”를 드래그 가능하게 만드는 래퍼 컴포넌트**이고, 핵심은 4가지입니다.

1. **`draggableId` / `index`로 드래그 대상의 정체성과 순서를 정의**
2. `provided`로 **DOM에 “드래그 기능을 연결(wiring)”**
3. `snapshot`으로 **드래그 중 UI 상태를 제어**
4. `provided.draggableProps.style`를 **반드시 style에 합쳐서** 라이브러리의 transform/positioning을 보존

아래에서 한 줄씩 “왜 이게 필요한지”까지 해부해드리겠습니다.

---

## 0) Draggable은 “아이템”이고 Droppable은 “컨테이너”입니다

* **Droppable**: 카드가 놓일 수 있는 리스트/컬럼(드롭 존)
* **Draggable**: 실제로 끌고 다니는 카드 하나

즉, 이 컴포넌트는 “카드 한 장”을 드래그 대상으로 만드는 역할입니다.

---

## 1) `<Draggable draggableId index>`의 의미 🎯

```jsx
<Draggable draggableId={draggableId} index={index}>
```

### ✅ `draggableId`

* 드래그 가능한 아이템의 **고유 ID**입니다.
* DnD 엔진이 “지금 잡고 있는 게 누구냐?”를 추적하는 키입니다.
* **Droppable의 `droppableId`와는 다른 개념**이고, Draggable마다 유일해야 합니다.

> 실무 팁: DB id/UUID 등을 그대로 쓰는 게 가장 안전합니다.

### ✅ `index`

* 현재 Droppable(리스트) 안에서 이 카드가 **몇 번째 위치인지(0부터)** 입니다.
* 라이브러리가 “정렬 변경 / placeholder 위치 / 드롭 후 재배치”를 계산할 때 필수입니다.

> 실무에서 가장 흔한 버그: `index`가 UI 렌더 순서와 실제 배열 순서가 안 맞으면 드래그가 튀거나 순서가 꼬입니다.

---

## 2) Render Props: `(provided, snapshot)`이 핵심 구조 🧠

```jsx
{(provided, snapshot) => {
  return (
    <div ...>...</div>
  )
}}
```

### A) `provided` = “드래그 기능을 DOM에 붙이는 도구 묶음”

여기에는 반드시 써야 하는 3종 세트가 있습니다:

1. `ref={provided.innerRef}`
2. `{...provided.draggableProps}`
3. `{...provided.dragHandleProps}` (핸들을 분리하지 않는다면)

### B) `snapshot` = “현재 드래그 상태”

* `snapshot.isDragging` : 지금 이 카드가 드래그 중인지 여부
* 이 값으로 opacity 같은 시각 효과를 줍니다.

---

## 3) `ref={provided.innerRef}`: 이게 없으면 위치 계산이 깨집니다

```jsx
ref={provided.innerRef}
```

DnD 엔진은 드래그 중에 계속 이런 걸 계산합니다:

* 카드의 원래 위치(좌표)
* 현재 마우스/터치 위치
* Droppable의 스크롤 상태
* 충돌(어느 위치에 들어갈지)

이를 위해 **해당 DOM 노드를 직접 참조해야** 하므로 ref 연결이 필수입니다.

---

## 4) `{...provided.draggableProps}`: “드래그 가능한 요소”임을 표시하는 속성들

```jsx
{...provided.draggableProps}
```

이 객체에는 보통 아래 같은 것들이 포함됩니다(라이브러리 내부 구현에 따라 다를 수 있음):

* `data-rfd-draggable-id` 같은 데이터 속성
* 드래그 중 transform/transition 관련 설정
* 접근성(ARIA) 속성 등

즉, “이 div는 draggable root야”라는 **필수 신호**입니다.

---

## 5) `{...provided.dragHandleProps}`: “잡는 손잡이(handle)” 역할

```jsx
{...provided.dragHandleProps}
```

이건 “어디를 잡아야 드래그가 시작되는지”를 결정합니다.

### 지금 코드의 의미

* draggable root(div) 전체가 **핸들**입니다.
* 즉, 카드 어느 부분을 잡아도 드래그가 시작됩니다.

### 핸들을 분리하고 싶다면?

예를 들어 카드 상단 바만 드래그 핸들로 쓰고 싶으면:

```jsx
<div ref={provided.innerRef} {...provided.draggableProps}>
  <div {...provided.dragHandleProps}>여기만 잡아서 드래그</div>
  <div>본문</div>
</div>
```

> 실무 팁: 버튼/인풋이 카드 안에 많으면 “전체가 핸들”일 때 클릭 UX가 나빠질 수 있어 핸들을 분리하는 편이 좋습니다.

---

## 6) `style`에서 가장 중요한 포인트: `provided.draggableProps.style`를 반드시 합쳐라 ⚠️

```jsx
style={{
  ...provided.draggableProps.style,
  cursor: 'grab',
  opacity: snapshot.isDragging ? 0.8 : 1
}}
```

이 부분이 이 코드의 “생명줄”입니다.

### ✅ 왜 필수인가?

드래그 중에 카드가 움직이는 효과는 보통:

* `transform: translate(...)`
* `transition`
* `position` 관련 값

을 라이브러리가 `provided.draggableProps.style`에 넣어서 제어합니다.

만약 여러분이 이렇게 해버리면:

```jsx
style={{ opacity: 0.8 }}
```

→ 라이브러리가 넣어둔 transform이 날아가서
**카드가 안 움직이거나, 이상한 위치로 튀거나, 애니메이션이 깨집니다.**

그래서 반드시:

* 먼저 라이브러리 style을 펼치고
* 그 위에 내 커스텀 스타일을 덮어씌우는

순서가 정석입니다.

---

## 7) `cursor: 'grab'` / `opacity`의 UX 의미 🎨

### cursor

```js
cursor: 'grab'
```

* 사용자에게 “이건 잡아서 끌 수 있는 요소다”라는 힌트를 줍니다.
* 드래그 중에는 `grabbing`으로 바꾸는 패턴도 많습니다.

예:

```js
cursor: snapshot.isDragging ? 'grabbing' : 'grab'
```

### opacity

```js
opacity: snapshot.isDragging ? 0.8 : 1
```

* 드래그 중인 카드가 “떠 있는 상태”임을 시각적으로 표현합니다.
* 너무 낮추면 가독성이 떨어지니 0.8 정도는 무난합니다.

---

## 8) 이 컴포넌트가 보장해야 하는 불변 조건 ✅

Draggable이 “정상 작동”하려면 아래가 충족돼야 합니다.

* [ ] `draggableId`는 **고유**하다.
* [ ] `index`는 **렌더링 순서와 일치**한다.
* [ ] root 요소에 `ref`, `draggableProps`가 붙었다.
* [ ] 드래그 핸들을 쓸 거면 `dragHandleProps`가 어딘가에 붙었다.
* [ ] `provided.draggableProps.style`를 style에 **합쳤다**.
* [ ] (Droppable 쪽) placeholder도 렌더링 중이다.

---

## 9) 한 문장 요약 🧩

> 이 코드는 `Draggable`이 제공하는 `provided`(필수 props/ref/style)로 DOM을 드래그 가능 상태로 연결하고, `snapshot`으로 드래그 중 UI(투명도/커서)를 제어하는 “카드 드래그 래퍼”입니다.

