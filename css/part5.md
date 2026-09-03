# PART 5. CSS Display & Normal Flow

## 1. 지금까지 무엇을 배웠는가?

지금까지의 흐름을 먼저 연결해 보겠습니다.

```text
PART 1
CSS Introduction
      ↓
PART 2
Selector & Cascade
      ↓
PART 3
Box Model
      ↓
PART 4
Size & Units
      ↓
PART 5
Display & Normal Flow
```

PART 3에서는 HTML Element가 CSS formatting 과정에서 Box를 생성한다는 것을 배웠습니다.

```text
HTML Element
      ↓
CSS 적용
      ↓
CSS Box
      ↓
┌─────────────────────┐
│ Margin              │
│ ┌─────────────────┐ │
│ │ Border          │ │
│ │ ┌─────────────┐ │ │
│ │ │ Padding     │ │ │
│ │ │ Content     │ │ │
│ │ └─────────────┘ │ │
│ └─────────────────┘ │
└─────────────────────┘
```

PART 4에서는 그 Box의 크기를 결정하는 방법을 배웠습니다.

```css
width: 300px;
width: 80%;
font-size: 2rem;
height: 100vh;
```

이제 중요한 질문이 생깁니다.

> **크기가 결정된 여러 Box는 화면에서 어떻게 배치되는가?**

여기서부터 **CSS Layout**이 본격적으로 시작됩니다.

---

# 2. CSS Layout이란?

HTML:

```html
<div class="box-a">A</div>
<div class="box-b">B</div>
<div class="box-c">C</div>
```

브라우저에는 여러 Box가 존재합니다.

```text
Box A
Box B
Box C
```

브라우저는 이 Box들에 대해 다음을 결정해야 합니다.

```text
크기는 얼마인가?
어디에 위치하는가?
다른 Box와 어떤 관계인가?
자식 Box는 어떻게 배치되는가?
```

이처럼 **Box의 크기와 위치, 그리고 Box들 사이의 배치 관계를 결정하는 과정**을 Layout이라고 이해할 수 있습니다.

```text
HTML Elements
      ↓
CSS Boxes
      ↓
Layout
      ↓
Box의 크기와 위치 결정
      ↓
Screen
```

---

# 3. CSS에는 디폴트 배치 방식이 있다

다음 HTML을 작성해 보겠습니다.

```html
<h1>CSS Layout</h1>

<p>Hello CSS</p>

<p>Normal Flow</p>
```

아무런 Layout CSS를 작성하지 않아도 브라우저는 Element들을 화면에 배치합니다.

대략:

```text
CSS Layout

Hello CSS

Normal Flow
```

처럼 표시됩니다.

왜 그럴까요?

브라우저에는 Element들을 배치하는 **디폴트 흐름**이 있기 때문입니다.

이것을 **Normal Flow**라고 합니다.

---

# 4. Normal Flow란?

**Normal Flow**는 특별한 Layout 방식이나 위치 조정을 적용하지 않았을 때 Box들이 문서 안에서 기본적으로 배치되는 방식입니다.

예를 들어:

```html
<div>A</div>
<div>B</div>
<div>C</div>
```

일반적인 `<div>`는 다음과 같이 배치됩니다.

```text
┌──────────────────────────────┐
│ A                            │
└──────────────────────────────┘

┌──────────────────────────────┐
│ B                            │
└──────────────────────────────┘

┌──────────────────────────────┐
│ C                            │
└──────────────────────────────┘
```

위에서 아래 방향으로 차례대로 배치됩니다.

반면:

```html
<span>A</span>
<span>B</span>
<span>C</span>
```

은 일반적으로:

```text
A B C
```

처럼 같은 줄에 배치됩니다.

이 차이를 이해하려면 `display`를 알아야 합니다.

---

# 5. `display`란?

`display`는 CSS Layout을 이해하는 핵심 Property입니다.

대표적으로:

```css
display: block;
display: inline;
display: inline-block;
display: none;
display: flex;
display: grid;
```

등이 있습니다.

초보 단계에서는 `display`를:

> **Element가 어떤 종류의 Box를 생성하고 Layout에 어떻게 참여할지를 결정하는 핵심 Property**

라고 이해하는 것이 좋습니다.

하지만 `display`에는 더 중요한 두 가지 관점이 있습니다.

```text
display
   │
   ├── Outer Display Type
   │
   │   주변 Box들과
   │   어떤 관계로 배치되는가?
   │
   └── Inner Display Type
       자식 Box들을
       어떤 방식으로 배치하는가?
```

이 개념은 뒤에서 다시 자세히 살펴보겠습니다.

---

# 6. `display: block`

가장 먼저 Block을 살펴보겠습니다.

```html
<div class="box">A</div>
<div class="box">B</div>
```

```css
.box {
  display: block;
}
```

Block Box는 Normal Flow에서 일반적으로 **새로운 줄에서 시작**합니다.

```text
Parent
┌──────────────────────────────┐
│                              │
│ ┌──────────────────────────┐ │
│ │ A                        │ │
│ └──────────────────────────┘ │
│                              │
│ ┌──────────────────────────┐ │
│ │ B                        │ │
│ └──────────────────────────┘ │
│                              │
└──────────────────────────────┘
```

즉 다음 Block Box는 아래쪽으로 배치됩니다.

```text
Block A
   ↓
Block B
   ↓
Block C
```

---

# 7. Block Box의 기본적인 특징

일반적인 Normal Flow의 Block Box는 다음과 같은 특징을 가집니다.

```text
1. 새로운 줄에서 시작

2. 기본 width가 auto

3. 일반적인 상황에서
   사용 가능한 가로 공간을 채움

4. 다음 Block Box는 아래에 배치
```

예:

```html
<div>A</div>
<div>B</div>
```

```text
Containing Block
┌────────────────────────────────┐
│ ┌────────────────────────────┐ │
│ │ A                          │ │
│ └────────────────────────────┘ │
│ ┌────────────────────────────┐ │
│ │ B                          │ │
│ └────────────────────────────┘ │
└────────────────────────────────┘
```

여기서 주의할 점은:

> `display: block`이라고 해서 `width: 100%`가 자동으로 설정되는 것은 아닙니다.

디폴트 `width`는:

```css
width: auto;
```

입니다.

Normal Flow의 일반적인 Block Formatting 상황에서 `auto`가 사용 가능한 가로 공간을 채우는 것처럼 동작하는 것입니다.

---

# 8. 대표적인 Block Element

브라우저의 User Agent Stylesheet에서 일반적으로 Block으로 표시되는 Element에는 다음과 같은 것들이 있습니다.

```html
<div>
<p>
<h1>
<h2>
<section>
<article>
<header>
<footer>
```

예:

```html
<p>첫 번째 문단</p>
<p>두 번째 문단</p>
```

별도의 `display`를 지정하지 않아도 일반적으로:

```text
첫 번째 문단

두 번째 문단
```

처럼 서로 다른 줄에 나타납니다.

중요한 것은:

> **HTML Element 자체가 본질적으로 영원히 Block인 것이 아니라 브라우저의 디폴트 CSS에서 해당 Element의 `display` 값이 그렇게 설정되어 있는 것**

입니다.

필요하다면 변경할 수 있습니다.

```css
p {
  display: inline;
}
```

---

# 9. `display: inline`

Inline Box는 Block과 다르게 일반적인 텍스트 흐름 안에서 배치됩니다.

HTML:

```html
<span>A</span>
<span>B</span>
<span>C</span>
```

일반적으로:

```text
A B C
```

처럼 표시됩니다.

```text
Text Line
────────────────────────────

[A] [B] [C]

────────────────────────────
```

즉 Inline Box는 **새로운 줄을 강제로 시작하지 않고 현재 Line 안에서 흐릅니다.**

---

# 10. Inline은 텍스트처럼 흐른다

다음 HTML을 보겠습니다.

```html
<p>
  가격은
  <span class="price">50,000원</span>
  입니다.
</p>
```

`<span>`은 일반적으로 Inline입니다.

화면에서는:

```text
가격은 50,000원 입니다.
       └──────┘
         span
```

처럼 문장 흐름의 일부로 배치됩니다.

그래서 `<span>`은 특정 텍스트 영역에 스타일을 적용할 때 매우 유용합니다.

```css
.price {
  color: red;
  font-weight: bold;
}
```

---

# 11. 대표적인 Inline Element

일반적으로 Inline으로 표시되는 대표적인 Element:

```html
<span>
<a>
<strong>
<em>
```

예:

```html
<p>
  이것은
  <strong>중요한</strong>
  내용입니다.
</p>
```

화면에서는 하나의 문장 흐름으로 표시됩니다.

```text
이것은 중요한 내용입니다.
```

---

# 12. Inline Box와 `width`, `height`

Inline Box에서 초보자가 가장 많이 혼동하는 부분입니다.

```html
<span class="box">Hello</span>
```

```css
.box {
  width: 300px;
  height: 100px;
}
```

일반적인 non-replaced Inline Box에서는 `width`와 `height`가 Block Box처럼 직접 적용되지 않습니다.

즉:

```text
display: inline

width: 300px
height: 100px

        ↓

Block Box와 같은 방식으로
크기가 결정되지 않음
```

Inline Box의 크기는 주로 Content와 Font Metrics, Line Formatting에 의해 결정됩니다.

이것이 `inline-block`이 필요한 이유 중 하나입니다.

---

# 13. Inline의 Padding과 Margin

Inline Element에도 Padding과 Margin을 지정할 수 있습니다.

```css
span {
  padding: 10px;
  margin: 10px;
}
```

하지만 Inline Formatting에서는 세로 방향 공간이 Block Layout에서 기대하는 방식과 다르게 동작할 수 있습니다.

특히:

```text
margin-left
margin-right
```

같은 수평 Margin은 흐름에 명확하게 영향을 주지만, 위아래 Margin은 일반적인 Block Box의 Margin처럼 Line Box의 배치를 밀어내는 방식으로 작동하지 않습니다.

따라서:

> **크기를 가진 독립적인 UI Box를 만들고 싶다면 `inline`보다 `inline-block`, Flexbox 등을 사용하는 경우가 많습니다.**

---

# 14. Block과 Inline 비교

핵심을 비교하면:

| 특징                 | `block`    | `inline`                              |
| ------------------ | ---------- | ------------------------------------- |
| 새로운 줄 시작           | O          | X                                     |
| Normal Flow 방향     | 주로 위 → 아래  | Line 안에서 흐름                           |
| `width` / `height` | 적용 가능      | 일반적인 non-replaced inline에는 직접 적용되지 않음 |
| 텍스트 흐름 내부 사용       | 일반적이지 않음   | 적합                                    |
| 대표 Element         | `div`, `p` | `span`, `a`                           |

개념적으로:

```text
BLOCK

┌──────────────────────┐
│ A                    │
└──────────────────────┘
┌──────────────────────┐
│ B                    │
└──────────────────────┘


INLINE

[A] [B] [C]
```

---

# 15. `display: inline-block`

`inline-block`은 이름 그대로 Inline과 Block의 특징을 함께 가집니다.

```css
.box {
  display: inline-block;
}
```

Outer Layout에서는 Inline처럼 배치됩니다.

```text
[A] [B] [C]
```

하지만 Box 자체에는 Block Container처럼 크기를 지정할 수 있습니다.

```css
.box {
  display: inline-block;

  width: 100px;
  height: 50px;
}
```

결과:

```text
┌───────┐ ┌───────┐ ┌───────┐
│   A   │ │   B   │ │   C   │
└───────┘ └───────┘ └───────┘
```

즉:

```text
바깥쪽
→ Inline처럼 배치

Box 자체
→ width / height 설정 가능
```

이라고 이해할 수 있습니다.

---

# 16. `inline-block`은 언제 사용하는가?

예를 들어:

```html
<a class="button">로그인</a>
<a class="button">회원가입</a>
```

```css
.button {
  display: inline-block;

  width: 120px;
  padding: 10px;

  text-align: center;
}
```

처럼 사용할 수 있습니다.

```text
┌────────────┐  ┌────────────┐
│   로그인   │  │  회원가입  │
└────────────┘  └────────────┘
```

현대 Layout에서는 Flexbox가 많은 역할을 대신하지만 `inline-block` 자체의 동작 원리는 여전히 알아둘 필요가 있습니다.

---

# 17. `block`, `inline`, `inline-block` 비교

세 가지를 한 번에 비교해 보겠습니다.

```text
block
──────────────────────────

┌──────────────────────┐
│ A                    │
└──────────────────────┘

┌──────────────────────┐
│ B                    │
└──────────────────────┘


inline
──────────────────────────

A B C


inline-block
──────────────────────────

┌─────┐ ┌─────┐ ┌─────┐
│  A  │ │  B  │ │  C  │
└─────┘ └─────┘ └─────┘
```

핵심:

```text
block
→ Box 단위로 위 → 아래

inline
→ Text Line 안에서 흐름

inline-block
→ Inline처럼 나란히
  + Box 크기 지정 가능
```

---

# 18. `display: none`

Element를 Layout에서 제거하고 싶다면:

```css
.menu {
  display: none;
}
```

을 사용할 수 있습니다.

HTML:

```html
<div>A</div>
<div class="menu">Menu</div>
<div>B</div>
```

`display: none`이 없다면:

```text
A
Menu
B
```

적용하면:

```text
A
B
```

가 됩니다.

중요한 것은 단순히 눈에 보이지 않는 것이 아닙니다.

> **`display: none`인 Element는 일반적인 Box를 생성하지 않으므로 Layout 공간도 차지하지 않습니다.**

---

# 19. `display: none`과 `visibility: hidden`

둘은 다릅니다.

```css
.box {
  display: none;
}
```

```text
A
B
```

Box가 Layout에서 제거됩니다.

반면:

```css
.box {
  visibility: hidden;
}
```

은 일반적으로:

```text
A

[공간은 존재]

B
```

처럼 Box가 차지하던 Layout 공간은 유지하면서 내용이 보이지 않게 됩니다.

비교:

| Property             | 보이는가? | Layout 공간 |
| -------------------- | ----: | --------: |
| `display: none`      |     X |         X |
| `visibility: hidden` |     X |         O |

---

# 20. Normal Flow에서 Block과 Inline은 어떻게 연결되는가?

Normal Flow는 단순히:

```text
모든 Element를 위에서 아래로 배치
```

하는 것이 아닙니다.

크게 보면 Block Formatting과 Inline Formatting이 함께 존재합니다.

예:

```html
<div>
  Hello
  <span>CSS</span>
  World
</div>

<div>Next</div>
```

개념적으로:

```text
Block Flow
│
├── Block Box
│    │
│    └── Inline Formatting
│         Hello [CSS] World
│
└── Block Box
     Next
```

즉:

```text
Block Box들
→ 주로 세로 방향으로 배치

그 안의 Inline Content
→ Line Box 안에서 흐름
```

으로 이해할 수 있습니다.

---

# 21. Line Box란?

Inline Content는 단순히 화면 아무 곳에 배치되는 것이 아닙니다.

브라우저는 Inline Content를 **Line Box** 안에 배치합니다.

예:

```html
<p>
  CSS layout is very important.
</p>
```

공간이 충분하면:

```text
┌────────────────────────────────┐
│ CSS layout is very important.  │ ← Line Box
└────────────────────────────────┘
```

공간이 부족하면 줄바꿈됩니다.

```text
┌──────────────────────┐
│ CSS layout is very   │ ← Line Box
├──────────────────────┤
│ important.           │ ← Line Box
└──────────────────────┘
```

따라서 Inline Layout은 **Line Box를 기반으로 흐른다**고 이해할 수 있습니다.

---

# 22. Normal Flow의 핵심 구조

전체를 정리하면:

```text
Normal Flow
│
├── Block Formatting
│
│    Block
│      ↓
│    Block
│      ↓
│    Block
│
└── Inline Formatting
     
     Line Box
     [Text] [span] [a]
     
     Line Box
     [Text] [strong]
```

이것이 Flexbox나 Grid를 사용하기 전 브라우저의 기본적인 Layout 세계입니다.

---

# 23. `display`는 자식만 배치하는 Property인가?

아닙니다.

이 부분이 매우 중요합니다.

다음 CSS를 보겠습니다.

```css
.container {
  display: block;
}
```

또는:

```css
.container {
  display: flex;
}
```

`display`는 단순히:

> "자식 Element의 Layout을 설정하는 Property"

라고 정의하면 부족합니다.

`display`에는 크게 두 가지 측면이 있기 때문입니다.

```text
display
   │
   ├── Outer Display Type
   │
   │   이 Element의 Box가
   │   부모의 Layout 안에서
   │   어떻게 참여하는가?
   │
   └── Inner Display Type
       이 Element가
       자신의 자식들을
       어떻게 배치하는가?
```

이 개념을 이해하면 `block`, `inline`, `flex`, `grid`의 관계가 훨씬 명확해집니다.

---

# 24. Outer Display Type

Outer Display Type은:

> **Element 자신이 주변 Sibling Box들과 어떤 관계로 배치되는가**

를 나타냅니다.

대표적으로:

```text
block
inline
```

을 생각할 수 있습니다.

예:

```text
block

Parent
│
├── Box A
│
├── Box B
│
└── Box C

주로 위 → 아래
```

반면:

```text
inline

Line Box
│
├── A
├── B
└── C

한 줄의 흐름
```

즉 **자기 자신이 부모 Layout에 어떻게 참여하는가**에 대한 개념입니다.

---

# 25. Inner Display Type

Inner Display Type은:

> **Element가 자신의 자식 Box들을 어떤 Layout 방식으로 배치하는가**

를 나타냅니다.

대표적으로:

```text
flow
flex
grid
```

를 생각할 수 있습니다.

예를 들어:

```css
.container {
  display: flex;
}
```

를 적용하면 Container의 자식들은 **Flex Formatting Context**에 참여합니다.

```text
Container
┌─────────────────────────────┐
│                             │
│ [Item A] [Item B] [Item C] │
│                             │
└─────────────────────────────┘
```

즉 `flex`가 자식 Layout에 영향을 주는 것은 맞지만, 이것이 `display` 전체 의미의 전부는 아닙니다.

---

# 26. `display`의 Outer + Inner 개념

현대 CSS Display 명세의 개념으로 보면 다음처럼 이해할 수 있습니다.

```text
display
│
├── Outer
│    ├── block
│    └── inline
│
└── Inner
     ├── flow
     ├── flow-root
     ├── flex
     └── grid
```

예를 들어 개념적으로:

```css
display: block;
```

은 일반적으로:

```text
Outer → block
Inner → flow
```

의 성격을 가집니다.

그리고:

```css
display: flex;
```

는 일반적으로:

```text
Outer → block
Inner → flex
```

인 Flex Container를 만듭니다.

---

# 27. `inline-flex`를 보면 더 명확해진다

Outer와 Inner를 이해하는 가장 좋은 예입니다.

```css
.container {
  display: flex;
}
```

와:

```css
.container {
  display: inline-flex;
}
```

를 비교해 보겠습니다.

둘 다 자식은:

```text
Flex Layout
```

으로 배치됩니다.

하지만 Container 자신이 부모 Layout에 참여하는 방식은 다릅니다.

```text
display: flex

Outer
→ block

Inner
→ flex
```

반면:

```text
display: inline-flex

Outer
→ inline

Inner
→ flex
```

입니다.

즉:

```text
          Outer      Inner

flex      block      flex

inline-
flex      inline     flex
```

입니다.

이것만 이해해도 `display`가 단순히 **자식 Layout만 설정하는 Property가 아니라는 것**을 알 수 있습니다.

---

# 28. `grid`도 같은 원리다

```css
.container {
  display: grid;
}
```

개념적으로:

```text
Outer → block
Inner → grid
```

입니다.

```css
.container {
  display: inline-grid;
}
```

라면:

```text
Outer → inline
Inner → grid
```

입니다.

따라서:

```text
display
        Outer       Inner

block   block       flow

inline  inline      flow

flex    block       flex

inline-flex
        inline      flex

grid    block       grid

inline-grid
        inline      grid
```

처럼 큰 그림을 이해할 수 있습니다.

---

# 29. Formatting Context란?

앞에서 다음 용어가 등장했습니다.

```text
Block Formatting
Inline Formatting
Flex Formatting
Grid Formatting
```

브라우저는 Box를 아무 규칙 없이 배치하지 않습니다.

특정 Layout 규칙이 적용되는 **Formatting Context** 안에서 Box를 배치합니다.

입문 단계에서는:

> **Formatting Context = 자식 Box들이 어떤 Layout 규칙에 따라 배치되는지를 결정하는 환경**

이라고 이해하면 충분합니다.

```text
Formatting Context
│
├── Block Formatting Context
├── Inline Formatting Context
├── Flex Formatting Context
└── Grid Formatting Context
```

---

# 30. Block Formatting Context

Block Formatting Context, 흔히 **BFC**라고 부릅니다.

BFC에서는 Block Box들이 특정 Block Layout 규칙에 따라 배치됩니다.

일반적인 흐름을 단순화하면:

```text
BFC

┌───────────────────────────────┐
│ ┌───────────────────────────┐ │
│ │ Block A                   │ │
│ └───────────────────────────┘ │
│              ↓                │
│ ┌───────────────────────────┐ │
│ │ Block B                   │ │
│ └───────────────────────────┘ │
│              ↓                │
│ ┌───────────────────────────┐ │
│ │ Block C                   │ │
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

BFC는 Float, Margin Collapsing 등과도 관련이 있습니다.

PART 5에서는 **“Block Layout이 동작하는 독립적인 Layout 환경”** 정도로 이해하면 충분합니다.

---

# 31. `display: flow-root`

BFC를 명시적으로 생성하고 싶을 때 사용할 수 있는 현대적인 방법이:

```css
.container {
  display: flow-root;
}
```

입니다.

`flow-root`는 Element가 새로운 Block Formatting Context를 만들도록 합니다.

```text
Parent Layout
      │
      ▼
┌─────────────────────────┐
│ display: flow-root      │
│                         │
│   새로운 BFC           │
│   ┌─────────────────┐   │
│   │ Child Boxes     │   │
│   └─────────────────┘   │
│                         │
└─────────────────────────┘
```

Float를 포함하는 Container 문제 등을 해결할 때 사용할 수 있습니다.

Float를 본격적으로 다루지 않는 입문 강의라면 개념 소개 정도면 충분합니다.

---

# 32. Normal Flow에서 벗어나는 경우

지금까지는 **Normal Flow**를 중심으로 설명했습니다.

하지만 모든 Box가 항상 Normal Flow 안에 있는 것은 아닙니다.

대표적으로:

```css
position: absolute;
```

같은 Positioning을 사용하면 Element가 일반적인 Normal Flow에서 벗어날 수 있습니다.

또한 Float도 Normal Flow와 다른 배치 특성을 가집니다.

```text
CSS Layout
│
├── Normal Flow
│    ├── Block
│    └── Inline
│
├── Float
│
├── Positioning
│
├── Flexbox
│
└── Grid
```

Position은 다음 PART에서 자세히 다룹니다.

---

# 33. `display`와 Position의 차이

두 Property는 서로 다른 역할을 합니다.

`display`:

```text
Box가 어떤 종류의
Layout에 참여하고

자식들에게 어떤
Formatting Context를 제공할 것인가?
```

Position:

```text
Box의 위치를
어떤 방식으로 결정할 것인가?
```

예:

```css
.box {
  display: block;
  position: relative;
}
```

두 Property는 동시에 존재할 수 있습니다.

따라서:

```text
display
≠
position
```

입니다.

---

# 34. 실전 예제 1 — `span`

HTML:

```html
<p>
  상품 가격은
  <span class="price">50,000원</span>
  입니다.
</p>
```

CSS:

```css
.price {
  color: red;
  font-weight: bold;
}
```

구조:

```text
Block Box
<p>
│
└── Inline Formatting
     │
     ├── "상품 가격은"
     ├── span.price
     └── "입니다."
```

화면:

```text
상품 가격은 50,000원 입니다.
           └───────┘
              span
```

이것이 `<span>`이 텍스트 일부를 스타일링할 때 적합한 이유입니다.

---

# 35. 실전 예제 2 — `div`

HTML:

```html
<div class="card">Keyboard</div>
<div class="card">Mouse</div>
```

CSS:

```css
.card {
  width: 300px;
  padding: 20px;
  border: 1px solid gray;
}
```

일반적인 `<div>`는 Block이므로:

```text
┌──────────────────────┐
│ Keyboard             │
└──────────────────────┘

┌──────────────────────┐
│ Mouse                │
└──────────────────────┘
```

처럼 위아래로 배치됩니다.

---

# 36. 실전 예제 3 — `inline-block`

두 Box를 나란히 놓고 크기도 지정하고 싶다고 하겠습니다.

```html
<div class="item">A</div>
<div class="item">B</div>
<div class="item">C</div>
```

```css
.item {
  display: inline-block;

  width: 100px;
  height: 100px;
}
```

결과:

```text
┌──────┐ ┌──────┐ ┌──────┐
│  A   │ │  B   │ │  C   │
└──────┘ └──────┘ └──────┘
```

하지만 여러 Item을 정렬하고 간격을 제어하는 Layout에서는 현대 CSS의 Flexbox가 더 편리합니다.

---

# 37. 왜 Flexbox가 필요한가?

예를 들어 다음 UI를 만든다고 하겠습니다.

```text
┌──────────────────────────────────┐
│ Logo       Menu       Login      │
└──────────────────────────────────┘
```

`inline-block`만으로도 어느 정도 만들 수 있지만 다음과 같은 요구가 생기면 복잡해집니다.

```text
수평 정렬
수직 중앙 정렬
Item 간격
남는 공간 분배
순서 변경
줄바꿈
```

그래서 현대 CSS에서는:

```css
.header {
  display: flex;
}
```

를 사용합니다.

```text
Container
┌───────────────────────────────────┐
│ Logo       Menu         Login     │
└───────────────────────────────────┘
           Flex Layout
```

즉:

```text
Normal Flow 이해
      ↓
block / inline 이해
      ↓
기존 Layout의 한계 이해
      ↓
Flexbox 필요성 이해
```

로 연결됩니다.

---

# 38. 왜 Grid가 필요한가?

이번에는 다음과 같은 Layout을 생각해 보겠습니다.

```text
┌────────┬────────┬────────┐
│ Card 1 │ Card 2 │ Card 3 │
├────────┼────────┼────────┤
│ Card 4 │ Card 5 │ Card 6 │
└────────┴────────┴────────┘
```

행과 열을 동시에 제어하고 싶습니다.

이때:

```css
.container {
  display: grid;
}
```

를 사용할 수 있습니다.

```text
Normal Flow
    ↓
Block / Inline
    ↓
1차원 Layout
Flexbox
    ↓
2차원 Layout
Grid
```

이러한 흐름으로 이해하면 CSS Layout 전체 구조가 자연스럽게 연결됩니다.

---

# 39. 자주 하는 오해 1

### "`display`는 자식 Element의 Layout을 설정한다."

절반만 맞습니다.

`display: flex`, `display: grid`에서는 Container의 자식 Layout이 크게 바뀌기 때문에 그렇게 보입니다.

하지만 더 정확하게는:

```text
display
│
├── Element 자신의
│   Outer Display Type
│
└── 자식들을 위한
    Inner Display Type
```

을 결정합니다.

따라서 `display`는 **자기 자신과 자식 Layout 양쪽에 관계하는 Property**입니다.

---

# 40. 자주 하는 오해 2

### "`div`는 Block Element이고 `span`은 Inline Element이다."

입문 설명으로는 사용할 수 있지만 엄밀하게는 조금 다듬어야 합니다.

브라우저의 디폴트 스타일이 일반적으로:

```css
div {
  display: block;
}

span {
  display: inline;
}
```

처럼 동작하도록 설정되어 있는 것입니다.

CSS로 변경할 수 있습니다.

```css
div {
  display: inline;
}

span {
  display: block;
}
```

따라서:

> **HTML Element의 의미(Semantics)와 CSS Display Type은 별개의 개념**

입니다.

---

# 41. 자주 하는 오해 3

### "`block`은 width가 항상 100%이다."

아닙니다.

일반적인 Block Box의 디폴트 width는:

```css
width: auto;
```

입니다.

Normal Flow의 일반적인 상황에서 사용 가능한 가로 공간을 채우도록 계산되기 때문에 `100%`처럼 보일 뿐입니다.

```text
width: auto
≠
width: 100%
```

입니다.

특히 Margin, Padding, Border, Box Sizing 등을 함께 사용하면 둘의 차이가 중요해질 수 있습니다.

---

# 42. 자주 하는 오해 4

### "`inline`에는 Box Model이 없다."

아닙니다.

Inline Element도 Box를 생성하며 Padding, Border 등의 Box Model 특성을 가질 수 있습니다.

다만 **Inline Formatting Context 안에서 동작하는 방식이 Block Box와 다릅니다.**

따라서:

```text
inline
→ Box가 없다
```

가 아니라:

```text
inline
→ Inline Formatting 규칙에 따라
  Box가 배치된다
```

라고 이해해야 합니다.

---

# 43. 자주 하는 오해 5

### "`display: none`은 투명하게 만드는 것이다."

아닙니다.

```css
display: none;
```

은 Box 생성을 억제하여 Layout 공간에서도 제거합니다.

반면:

```css
opacity: 0;
```

은 투명하게 만들지만 Box 자체는 Layout에 남아 있습니다.

그리고:

```css
visibility: hidden;
```

도 보이지 않지만 일반적으로 Layout 공간은 유지됩니다.

단순화해서 비교하면:

```text
display: none
→ 보이지 않음
→ Layout 공간 없음


visibility: hidden
→ 보이지 않음
→ Layout 공간 있음


opacity: 0
→ 투명함
→ Layout 공간 있음
```

각 속성은 이벤트/접근성 측면에서도 차이가 있으므로 단순히 “세 가지가 모두 숨기기”라고만 이해해서는 안 됩니다.

---

# 44. PART 5 전체 핵심 구조

이번 PART에서 가장 중요한 구조입니다.

```text
HTML Element
      ↓
CSS 적용
      ↓
Box 생성
      ↓
display
      │
      ├──────────────┐
      ↓              ↓
Outer Display     Inner Display
      │              │
      │              │
block / inline    flow / flex / grid
      │              │
      └──────┬───────┘
             ↓
      Formatting Context
             ↓
           Layout
             ↓
           Screen
```

---

# 45. Normal Flow 전체 정리

CSS를 특별히 변경하지 않았을 때 기본적으로 Normal Flow가 적용됩니다.

```text
Normal Flow
│
├── Block Formatting
│
│    ┌───────────────┐
│    │ Block A       │
│    └───────────────┘
│            ↓
│    ┌───────────────┐
│    │ Block B       │
│    └───────────────┘
│
└── Inline Formatting

     Line Box
     ┌───────────────────────┐
     │ Text [span] [a] Text  │
     └───────────────────────┘
```

그리고 `display`를 변경하면 다른 Layout Context를 만들 수 있습니다.

```text
display: flex
      ↓
Flex Formatting Context


display: grid
      ↓
Grid Formatting Context
```

---

# 46. PART 5 핵심 정리

**Normal Flow**

특별한 Layout 방식을 지정하지 않았을 때 Box가 기본적으로 배치되는 흐름입니다.

```text
Normal Flow
│
├── Block Flow
└── Inline Flow
```

**Block**

```text
┌───────────────────────┐
│ A                     │
└───────────────────────┘
┌───────────────────────┐
│ B                     │
└───────────────────────┘
```

일반적으로 새로운 줄에서 시작하고 Block Flow에서 위에서 아래로 배치됩니다.

**Inline**

```text
A B C
```

Line Box 안에서 텍스트처럼 흐릅니다.

**Inline Block**

```text
┌─────┐ ┌─────┐ ┌─────┐
│  A  │ │  B  │ │  C  │
└─────┘ └─────┘ └─────┘
```

Inline처럼 배치되면서 Box의 크기를 지정할 수 있습니다.

**Display의 핵심**

```text
display
│
├── Outer
│   자기 자신이
│   부모 Layout에 참여하는 방식
│
└── Inner
    자식들을 배치하는 방식
```

따라서:

```text
block
→ block + flow

inline
→ inline + flow

flex
→ block + flex

inline-flex
→ inline + flex

grid
→ block + grid

inline-grid
→ inline + grid
```

라는 큰 구조로 이해할 수 있습니다.

---

# 47. 지금까지의 CSS Layout 흐름

PART 1부터 현재까지 연결하면:

```text
HTML Element
      ↓
PART 1
CSS 적용
      ↓
PART 2
Selector & Cascade
      ↓
Computed Style
      ↓
PART 3
CSS Box / Box Model
      ↓
PART 4
Size & Units
      ↓
PART 5
Display
      ↓
Formatting Context
      ↓
Normal Flow
      ↓
┌─────────────┬─────────────┐
│             │             │
Block       Inline      다른 Layout
                            │
                     ┌──────┴──────┐
                     │             │
                    Flex          Grid
```

이제 우리는 **Box가 무엇이고, 크기가 어떻게 결정되며, 디폴트 상태에서 어떻게 배치되는지**까지 알게 되었습니다.

다음으로 필요한 것은:

> **Normal Flow의 기본 위치가 아니라 원하는 위치에 Element를 배치하려면 어떻게 해야 하는가?**

입니다.

다음 PART에서는 이를 담당하는 **`position`**을 배웁니다.

```text
PART 5
Display & Normal Flow
      ↓
PART 6
Position
      ↓
PART 7
Flexbox
      ↓
PART 8
Grid
```

---

# PART 5의 가장 중요한 한 문장

> **`display`는 단순히 자식 Element의 배치 방법을 정하는 Property가 아니라, Element 자신이 부모 Layout에 어떻게 참여할지와 자신의 자식들에게 어떤 Layout 방식을 제공할지를 결정하는 CSS Layout의 핵심 Property이다.**


