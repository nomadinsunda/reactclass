`onMouseEnter`는 React에서 **마우스 이벤트**를 처리하기 위한 **합성 이벤트(Synthetic Event)** 중 하나입니다.

---

# 🖱️ `onMouseEnter`란?

## ✅ 정의

> **`onMouseEnter`** 는 사용자가 마우스를 어떤 요소 위로 **올렸을 때**(mouseenter 이벤트 발생 시)
> 해당 요소에 등록된 이벤트 핸들러를 실행하는 **React의 합성 이벤트**입니다.

---

## ✅ 기본 사용법 (React JSX에서)

```jsx
<div onMouseEnter={() => console.log('마우스 들어옴')}>
  마우스를 올려보세요
</div>
```

---

## 🔍 HTML vs React 이벤트 이름 비교

| 동작                       | DOM 이벤트 이름 (HTML/JS) | React 이벤트 이름   |
| ------------------------ | -------------------- | -------------- |
| 마우스가 요소 위로 진입할 때         | `mouseenter`         | `onMouseEnter` |
| 마우스가 요소에서 벗어날 때          | `mouseleave`         | `onMouseLeave` |
| 마우스가 올라갈 때 (자식 포함 모두 반응) | `mouseover`          | `onMouseOver`  |
| 마우스가 나갈 때 (자식 포함 모두 반응)  | `mouseout`           | `onMouseOut`   |

> 📌 React에서는 항상 **카멜 케이스**(`onMouseEnter`, `onClick`)로 작성합니다.
> 그리고 JSX에서는 `addEventListener`가 아니라 **속성처럼 핸들러를 등록**합니다.

---

## ✅ `onMouseEnter` vs `onMouseOver`

| 항목     | `onMouseEnter`            | `onMouseOver`             |
| ------ | ------------------------- | ------------------------- |
| 트리거 시점 | 마우스가 요소에 **처음 들어왔을 때만**   | 요소나 **자식 요소에 진입해도 계속 발생** |
| 버블링    | ❌ 버블링되지 않음 (자식 요소에 영향 없음) | ✅ 버블링됨 (자식 요소에서 계속 발생)    |
| 사용 용도  | 오직 해당 요소에 진입한 순간만 감지      | 자식 포함해서 마우스가 머무는 모든 순간 감지 |

### 🔸 예시

```jsx
<div
  onMouseEnter={() => console.log('Enter')}
  onMouseOver={() => console.log('Over')}
>
  <span>마우스를 여기에 올려보세요</span>
</div>
```

* 마우스를 `<span>`에 올리면:

  * `onMouseOver`: ✅ 실행됨 (자식도 포함)
  * `onMouseEnter`: ❌ 실행되지 않음 (부모 기준)

---

## ✅ 실전 예제

```jsx
function HoverBox() {
  const handleEnter = () => {
    console.log('마우스가 들어옴');
  };
  const handleLeave = () => {
    console.log('마우스가 나감');
  };

  return (
    <div
      onMouseEnter={handleEnter}
      onMouseLeave={handleLeave}
      style={{
        border: '1px solid gray',
        padding: '20px',
        width: '200px',
        textAlign: 'center',
        cursor: 'pointer',
      }}
    >
      🖱 여기에 마우스를 올려보세요
    </div>
  );
}
```

---

## ✅ 요약 정리

| 항목     | 설명                            |
| ------ | ----------------------------- |
| 이름     | `onMouseEnter`                |
| 종류     | React 합성 이벤트 (SyntheticEvent) |
| 동작     | 마우스가 요소에 **진입했을 때** 실행        |
| 버블링 여부 | ❌ 버블링되지 않음                    |
| 주 용도   | 마우스가 요소에 "딱 들어왔을 때" 효과 주기     |
| 비교     | `onMouseOver`는 자식 포함해서 계속 발생  |

