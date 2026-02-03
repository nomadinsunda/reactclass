# 🔬 React Render Prop 패턴

---

## 0️⃣ 결론부터 (이 한 문장)

> **render prop 패턴은
> “컴포넌트가 가진 내부 로직/상태를
> JSX를 그리는 ‘함수’에게 위임하는 구조”입니다.**

즉,

* **로직은 컴포넌트가 갖고**
* **모양(JSX)은 사용자가 정한다**

이 분리가 목적입니다.

---

## 1️⃣ React에서 원래 props는 뭐였나?

우리가 익숙한 props는 이 구조입니다.

```jsx
<MyComponent title="안녕하세요" />
```

```jsx
function MyComponent(props) {
  return <h1>{props.title}</h1>;
}
```

👉 **데이터를 내려준다**
👉 **컴포넌트가 JSX를 고정적으로 그린다**

이 방식의 한계는:

> ❌ “로직은 재사용하고 싶은데
> ❌ UI는 상황마다 다르게 그리고 싶다”

---

## 2️⃣ 그 한계를 정면으로 해결하려고 나온 게 Render Prop

### 핵심 발상

> “JSX를 값으로 넘기지 말고
> **JSX를 만드는 ‘함수’를 넘기자**”

---

## 3️⃣ children이 원래 뭐냐부터 정확히 짚기

React에서 `children`은 원래 이런 거였습니다.

```jsx
<Box>
  <p>안녕하세요</p>
</Box>
```

```jsx
function Box({ children }) {
  return <div>{children}</div>;
}
```

👉 `children`은 **ReactNode(결과물)**

---

## 4️⃣ Render Prop의 첫 번째 전환점

children을 **값이 아니라 함수로** 쓰자.

```jsx
<Box>
  {(data) => <p>{data}</p>}
</Box>
```

```jsx
function Box({ children }) {
  const data = "Hello";
  return children(data);
}
```

여기서 무슨 일이 일어났냐면:

* ❌ Box가 JSX를 결정하지 않음
* ✅ Box는 **데이터만 계산**
* ✅ JSX는 **개발자가 결정**

---

## 5️⃣ 이게 바로 Render Prop 패턴입니다

### 정의를 다시 정확히 쓰면

> **Render Prop 패턴 =
> 컴포넌트가 내부 상태/로직을 계산한 뒤
> “그걸 아규먼트로 전달하면서
> 개발자가 구현한 함수(render 함수)를 호출하는 패턴”**

---

## 6️⃣ 흐름을 그림처럼 정리하면

```
[ Box 컴포넌트 ]
   ├─ 상태/로직 계산
   ├─ data 준비
   └─ children(data) 호출
                ↓
        [ 개발자가 정의한 함수 ]
                ↓
              JSX 리턴
```

👉 **JSX의 주도권이 개발자에게 넘어감**

---

## 7️⃣ render prop vs 일반 컴포넌트 차이

### 일반 컴포넌트

```jsx
<Tooltip text="설명" />
```

```jsx
function Tooltip({ text }) {
  return <div className="tooltip">{text}</div>;
}
```

* UI 고정
* 재사용성 낮음

---

### Render Prop 컴포넌트

```jsx
<Tooltip>
  {(isOpen) => (
    isOpen ? <div>열림</div> : null
  )}
</Tooltip>
```

```jsx
function Tooltip({ children }) {
  const [isOpen] = useState(true);
  return children(isOpen);
}
```

* 로직 재사용
* UI 완전 자유

---

## 8️⃣ “왜 굳이 이렇게 복잡하게?”에 대한 정확한 답

### React에는 근본 제약이 하나 있습니다

> ❗ **부모 컴포넌트는
> 자식 컴포넌트의 JSX 내부를
> 직접 바꿀 수 없다**

그래서:

* props → 값만 전달 가능
* children → JSX 덩어리만 전달 가능

👉 **“JSX를 만드는 방법” 자체를 넘기려면?**

➡️ **함수밖에 없습니다**

---

## 9️⃣ 그래서 DnD 코드가 저렇게 생긴 겁니다

```jsx
<Droppable>
  {(provided) => (
    <div ref={provided.innerRef}>
      {provided.placeholder}
    </div>
  )}
</Droppable>
```

이건 말 그대로:

> “Droppable이 계산한 내부 결과(provided)를
> 인자로 받아서
> JSX를 어떻게 그릴지는
> 내가 결정하겠다”

---

## 🔑 여기서 핵심 포인트 (이해 체크)

* ❌ render prop은 JSX 문법이 아닙니다
* ❌ 특수 React 기능도 아닙니다
* ✅ **그냥 함수입니다**
* ✅ **children을 함수로 쓰는 패턴**일 뿐입니다

---

## 10️⃣ 왜 요즘엔 덜 쓰일까? (중요한 역사)

Render Prop은 강력했지만 문제가 있었습니다.

```jsx
<A>
  {(a) => (
    <B>
      {(b) => (
        <C>
          {(c) => (
            <D a={a} b={b} c={c} />
          )}
        </C>
      )}
    </B>
  )}
</A>
```

👉 **지옥의 중첩**

그래서 등장한 게:

* React Hooks
* Custom Hooks
* Context + Hook 조합

👉 **하지만 DnD 같은 “DOM 제어 라이브러리”는
Hook만으로 해결이 안 되는 영역**이라
render prop이 아직 살아 있습니다.




