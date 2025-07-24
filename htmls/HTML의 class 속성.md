`class`는 HTML에서 **스타일링과 스크립팅에 핵심적인 역할을 하는 속성**입니다.

---

# 🎯 HTML의 `class` 속성

## ✅ 1. 정의

HTML에서 `class` 속성은 요소에 **하나 이상의 클래스 이름을 부여**하여:

* CSS 스타일을 적용하거나,
* JavaScript로 요소를 선택하거나 조작할 수 있도록

하는 **범용적인 식별자 역할**을 합니다.

```html
<p class="notice">공지사항입니다.</p>
```

> 여기서 `"notice"`는 클래스 이름이며, CSS에서 `.notice { ... }`로 스타일을 지정할 수 있습니다.

---

## 🧩 2. 주요 특징

| 항목    | 설명                                          |
| ----- | ------------------------------------------- |
| 유형    | 전역 속성(Global attribute) — 모든 HTML 요소에 사용 가능 |
| 값     | 공백으로 구분된 **하나 이상의 클래스 이름**                  |
| 용도    | CSS 스타일 지정, JavaScript DOM 조작, 접근성 관리 등     |
| 중복 사용 | 동일한 클래스 이름을 여러 요소에 부여 가능                    |
| 우선순위  | CSS에서 class 선택자는 `.`으로 접근하며, ID보다 낮은 우선순위   |

---

## 🖌️ 3. CSS와의 연결

```html
<style>
  .warning {
    color: red;
    font-weight: bold;
  }
</style>

<p class="warning">주의! 이 문장은 빨간색입니다.</p>
```

* `.warning`: class가 `warning`인 모든 요소에 스타일 적용
* 여러 개의 요소에 동일한 class를 적용하면 **스타일을 일괄 적용**할 수 있어 효율적

---

## 🧠 4. JavaScript와의 연계

```html
<div class="box">Hello</div>

<script>
  const box = document.querySelector('.box');
  box.classList.add('highlight');   // 새로운 클래스 추가
  box.classList.remove('box');      // 클래스 제거
  box.classList.toggle('active');   // 토글링
</script>
```

* `document.querySelector('.box')`: class 이름으로 요소 탐색
* `classList`: DOMTokenList 객체로 클래스 조작 가능

---

## ✅ 5. 여러 클래스 지정하기

```html
<div class="card highlight shadow"></div>
```

* **클래스 이름은 공백으로 구분하여 여러 개 지정** 가능
* CSS에서 각각의 클래스가 개별적으로 적용됨

```css
.card { border: 1px solid #ccc; }
.highlight { background-color: yellow; }
.shadow { box-shadow: 2px 2px 5px rgba(0,0,0,0.3); }
```

---

## 🔍 6. `id`와의 차이점

| 속성      | `class`          | `id`                    |
| ------- | ---------------- | ----------------------- |
| 대상 수    | 여러 요소에 동일 클래스 가능 | 문서 내 유일한 값만 허용          |
| CSS 선택자 | `.` (예: `.btn`)  | `#` (예: `#header`)      |
| 용도      | 그룹 스타일링 및 DOM 접근 | 고유 식별자, 앵커, 라벨 연결 등     |
| 우선순위    | 낮음               | 높음 (CSS specificity 기준) |

---

## 📌 7. class 이름 규칙

* **알파벳/숫자/대시(-)/언더스코어(\_)** 사용 가능
* 숫자로 시작하면 안 됨
* 띄어쓰기 불가 (띄어쓰기는 **다중 클래스 지정용**으로 사용됨)
* 일반적으로 **케밥 케이스(kebab-case)** 사용

```html
<div class="user-profile-card"></div>
```

---

## 🧩 8. HTML과 React(JSX) 차이

| 항목     | HTML                | React (JSX)                                       |
| ------ | ------------------- | ------------------------------------------------- |
| 클래스 지정 | `<div class="box">` | `<div className="box">`                           |
| 이유     | `class`는 HTML 표준 속성 | JavaScript의 `class` 예약어와 충돌 피하기 위해 `className` 사용 |

---

## 🧠 실무 팁

* ✅ **공통 UI 요소**에는 `class`를 적극적으로 활용하여 스타일 재사용을 극대화
* ✅ BEM(Block Element Modifier) 명명법 사용 시 구조적인 class 네이밍 가능
* ✅ JavaScript 동적 UI에서는 `classList` API로 상태 제어

---

## ✅ 요약

| 항목      | 설명                                  |
| ------- | ----------------------------------- |
| 역할      | CSS 스타일링, JavaScript DOM 조작을 위한 식별자 |
| 값       | 공백으로 구분된 여러 개의 클래스명                 |
| 사용 위치   | 모든 HTML 태그 가능                       |
| CSS 접근  | `.클래스명 { ... }`                     |
| React에서 | `className` 속성 사용                   |

