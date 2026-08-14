다음 포맷은 **Redux가 커지면서 반드시 필요해진 “이름 충돌 방지 + 구조적 의미 부여”를 위한 관례**입니다.

```js
'@cardEntities/add'
```

이 한 줄에 **의도, 역사, 아키텍처 철학**이 전부 들어 있습니다.

---

## 1⃣ Redux action의 본질부터 짚고 가겠습니다

Redux에서 action은 **전역 이벤트 이름**입니다.

```js
dispatch({ type: 'SOMETHING' })
```

 이 `type` 문자열은
O **전역에서 유일해야 하고**
O **어떤 도메인에서 무슨 일이 일어났는지**를 표현해야 합니다.

---

## 2⃣ 왜 그냥 `'add'` 가 아니라 이렇게 쓰는가?

### X 나쁜 예 (실무에서 바로 사고 납니다)

```js
type: 'add'
```

* card에서 add
* todo에서 add
* user에서 add
* cart에서 add

 **Reducer가 많아질수록 충돌 + 가독성 붕괴**

---

## 3⃣ 그래서 등장한 “namespace/action” 패턴

```js
'cardEntities/add'
```

이 형식은 사실상 다음 의미입니다:

```
[도메인]/[행동]
```

| 부분           | 의미                         |
| ------------ | -------------------------- |
| cardEntities | 어떤 상태 영역(slice / domain)인가 |
| add          | 그 영역에서 무슨 일이 일어났는가         |

 **URL, 파일 경로, Java 패키지명과 동일한 사고방식**입니다.

---

## 4⃣ 그럼 왜 `@`가 붙어 있을까?

```js
'@cardEntities/add'
```

이 `@`는 **Redux 공식 문법은 아닙니다. 관례(convention)** 입니다.

### `@`의 실제 의미

#### O 1. “이건 애플리케이션 내부 action이다”

* Redux 내부 액션: `@@redux/INIT`
* 사용자 정의 액션: `@cardEntities/add`

 **의도적으로 Redux 내부 액션과 시각적으로 구분**

---

#### O 2. 로그 & 디버깅 가독성

Redux DevTools에서 보면:

```txt
@cardEntities/add
@cardEntities/remove
@auth/login
@auth/logout
```

 **한눈에 도메인별로 묶여 보임**

---

#### O 3. “이 액션은 slice 소유다”라는 신호

```js
@cardEntities/add
```

 이 action은
X UI 이벤트가 아니라
X 네트워크 이벤트가 아니라
O **cardEntities 상태를 변경하기 위한 action**

---

## 5⃣ Redux Toolkit의 slice는 이걸 자동으로 해준다

RTK의 `createSlice`를 쓰면:

```js
createSlice({
  name: 'cardEntities',
  reducers: {
    add(state, action) {}
  }
})
```

자동으로 생성되는 action type:

```js
'cardEntities/add'
```

 즉,

```js
'@cardEntities/add'
```

은 **RTK 이전 시대에 개발자가 직접 흉내 낸 포맷**입니다.

### RTK를 쓰면 “액션 타입 상수 파일”이 사라집니다

```js
// ❌ 레거시: 상수 → action creator → reducer 3중 관리
export const ADD_CARD = '@cardEntities/add'
export const addCard = (card) => ({ type: ADD_CARD, payload: card })
// switch (action.type) { case ADD_CARD: ... }
```

```js
// ✅ RTK: createSlice 하나로 끝
const { add } = cardEntitiesSlice.actions

add.type        // 'cardEntities/add'  ← 문자열이 필요하면 여기서 꺼냅니다
String(add)     // 'cardEntities/add'  (toString()이 type을 반환)
add.match(action) // 타입 가드 (TS에서 action 타입 좁히기 / addMatcher에 활용)
```

* 오타로 인한 “아무 일도 안 일어나는” 버그가 구조적으로 사라집니다.
* `createAsyncThunk`도 같은 규칙을 따릅니다.

```txt
todos/fetchTodos/pending
todos/fetchTodos/fulfilled
todos/fetchTodos/rejected
```

* 미들웨어/로깅에서 도메인 단위로 잡고 싶다면 문자열 비교 대신 `action.type.startsWith('cardEntities/')`
  또는 `isAnyOf(add, remove)` / `addMatcher`를 쓰는 편이 안전합니다.

### 그래서 지금도 `@`를 붙여야 할까요?

* **RTK 기준으로는 붙일 이유가 거의 없습니다.** `name`이 이미 네임스페이스 역할을 하고, DevTools에서도 도메인별로 잘 묶입니다.
* `createSlice`의 `name`에 `'@cardEntities'`처럼 `@`를 넣는 것도 가능은 하지만, 팀 안에서 붙인 것/안 붙인 것이 섞이면 오히려 검색·필터가 어려워집니다.
* 결론: **레거시 코드에서 `@`를 보면 “RTK 이전 관례구나”로 읽고, 신규 코드는 `slice이름/리듀서이름`으로 통일**하세요.

---

## 6⃣ 왜 굳이 문자열로 이런 포맷을 유지하나?

Redux action은:

* 직렬화 가능해야 하고
* 로그/리플레이 가능해야 하고
* 네트워크/미들웨어에서 분석 가능해야 합니다

그래서:

```js
type: 'ADD_CARD' ❌
type: 'cardEntities/add' ✅
```

또한 action은 **직렬화 가능해야 한다**는 규칙 때문에,
`type`은 반드시 **문자열**이어야 하고 Symbol이나 함수는 쓸 수 없습니다.
(`configureStore`의 `serializableCheck` 미들웨어가 개발 모드에서 이를 검사해 줍니다.)

 **의미가 살아 있는 문자열**이 선호됩니다.

---

## 7⃣ 이 패턴의 진짜 목적 (중요)

```txt
@cardEntities/add
```

이건 단순한 문자열이 아니라:

> **“cardEntities라는 도메인에서 add라는 상태 변화 이벤트가 발생했다”**

라는 **도메인 이벤트 선언**입니다.

Redux는 사실상:

* 전역 Event Bus
* 상태 기반 Event Sourcing 모델

에 가깝습니다.

---

## 8⃣ 이름 짓기 실전 가이드

이벤트 로그라는 관점에서 보면, action 이름은 **“명령”이 아니라 “일어난 일”** 로 짓는 편이 좋습니다.

| 권장 O | 비권장 | 이유 |
| ------------------- | ------------------- | --------------------------- |
| `cart/itemAdded`    | `cart/addItem`      | reducer는 “요청 처리기”가 아니라 “사실 기록기” |
| `auth/loginSucceeded` | `auth/setUser`    | 무슨 일이 있었는지 드러남              |
| `todos/todoToggled` | `todos/setTodoDone` | 하나의 사건 = 하나의 액션             |

* 특히 여러 slice가 **같은 사건에 반응**해야 할 때(로그인 성공 → user, cart, ui가 각각 반응),
  “사실 중심” 이름이어야 `extraReducers`에서 자연스럽게 공유됩니다.
* 반대로 `setXxx` 스타일이 늘어나면 Redux가 그냥 **전역 setState**로 전락합니다.

