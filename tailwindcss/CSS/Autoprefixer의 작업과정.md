# 🎯 먼저 핵심 요약(이걸 이해하면 끝!)

1. **옛날 브라우저들은 “flex”라는 표준 문법을 몰랐다.**
2. 그래서 flex 기능을 각각 자기만의 문법으로 제공했다.

   * 옛날 Safari → `display: -webkit-box`
   * IE10 → `display: -ms-flexbox`
3. 최신 브라우저들은 **표준 문법인 `display: flex`만 이해한다.**
4. Autoprefixer는 “여러 브라우저에서 다 돌아가게 하려고”
   → **구형 + 최신 방식을 모두 넣어준다.**

즉,

```css
display: flex;
```

라고 **개발자가 단 한 줄만 썼는데**,
Autoprefixer가 **과거 호환성까지 챙겨서 3줄로 확장**해주는 것이다.

---

# 이제 하나씩 아주 천천히 설명드립니다.

## 🧩 1. “왜 flex가 3개의 다른 코드로 변환되지?”

### 👉 이유: 옛날 브라우저들은 표준이 나오기 전에 **각자 독자적인 문법으로 구현했기 때문**

과거 브라우저들은 flexbox가 표준이 되기 전에 이렇게 각각 **다른 이름**으로 구현했습니다:

| 브라우저            | flexbox를 뭐라고 불렀나? | 표기법                    |
| --------------- | ----------------- | ---------------------- |
| 옛 Safari/WebKit | `box` 모델          | `display: -webkit-box` |
| IE10            | 자체 flexbox 개념     | `display: -ms-flexbox` |
| 최신 브라우저         | 표준 flexbox        | `display: flex`        |

즉, flexbox의 공식 문법인 `display: flex`가 나오기 **전**에는:

* Safari 팀: “우리만의 방식으로 박스 레이아웃 만들어볼게!” → `-webkit-box`
* IE 팀: “우리도 flexbox 만들어봤는데 문법은 다름!” → `-ms-flexbox`

그래서 “진짜 표준 문법”인 `display: flex`가 세상에 나오기 전
브라우저들은 **각자 제멋대로 구현**하고 있었습니다.

---

## 🧩 2. 그럼 Autoprefixer는 왜 3개를 모두 넣는가?

Autoprefixer의 역할은 다음과 같습니다:

> “이 CSS가 옛날부터 최신 브라우저까지 다 돌아가게 해줄게요!”

즉, **과거와 현재를 모두 고려해서 변환**합니다.

그래서 개발자가 이렇게 썼는데:

```css
.box {
  display: flex;
}
```

Autoprefixer는 자동으로 이렇게 바꿉니다:

```css
.box {
  display: -webkit-box;   /* 옛날 Safari/WebKit */
  display: -ms-flexbox;   /* 옛날 IE10 */
  display: flex;          /* 최신 표준 */
}
```

---

## 🧩 3. “그럼 브라우저는 저 3줄을 어떻게 해석하나요?”

브라우저는 CSS를 읽을 때 이런 규칙을 따릅니다:

### ✔ 규칙 1 — **모르는 속성은 그냥 무시한다.**

예를 들어 Chrome 최신 버전은:

* `display: -webkit-box` → “이거 옛날 방식인데, 난 이걸 안 써!” → 무시
* `display: -ms-flexbox` → “이건 IE 용인데? 몰라!” → 무시
* `display: flex` → “이건 내가 아는 표준 문법!” → 적용

**최종적으로 `display: flex`가 적용됨.**

---

### ✔ 규칙 2 — **같은 속성이 여러 번 있으면, 나중 줄이 이긴다.**

옛날 Safari 같은 경우는:

* `display: -webkit-box` → “오! 내가 아는 거네!” → 적용
* `display: -ms-flexbox` → “이건 IE 용이네? 무시!”
* `display: flex` → “이건 나 아직 몰라…” → 무시

**그래서 `-webkit-box`가 최종 적용.**

---

## 🔥 아주 중요: “3줄이 아니라 1줄만 적용되는 것”

항상 마지막까지 모두 적용되는 게 아닙니다.
**브라우저는 자기에게 맞는 한 줄만 적용하고, 나머지는 무시합니다.**

즉,

| 브라우저      | 실제로 적용되는 줄             |
| --------- | ---------------------- |
| 최신 Chrome | `display: flex`        |
| 사파리 구버전   | `display: -webkit-box` |
| IE10      | `display: -ms-flexbox` |

나머지는 그냥 무시.
에러도 없음. 문제 없음.

그래서 CSS 파일에는 3줄이 들어가지만
브라우저는 **딱 1줄만 사용**합니다.

---

# 🔍 지금까지 설명을 초간단 요약

### 개발자가 쓴 것

```css
display: flex;
```

### Autoprefixer가 과거 브라우저 호환 위해 추가

```css
display: -webkit-box;
display: -ms-flexbox;
display: flex;
```

### 브라우저가 실제로 적용

* 최신 Chrome → `display: flex`만 사용
* 구 Safari → `display: -webkit-box`만 사용
* IE10 → `display: -ms-flexbox`만 사용

❗ 결국 브라우저는 “내가 이해할 수 있는 한 줄만 적용”
❗ 나머지는 무시

---

# 🎯 사용자님이 가장 막히신 핵심 문장

**“왜 내가 flex 한 줄만 썼는데, Autoprefixer는 -webkit-box, -ms-flexbox까지 생성할까?”**

정답은:

> flexbox가 표준이 되기 전, 브라우저들이 제각각 다른 방식으로 flex를 구현했기 때문에
> “모든 브라우저에서 돌아가게 하려고” Autoprefixer가 과거 버전의 문법까지 자동으로 넣어주는 것이다.

