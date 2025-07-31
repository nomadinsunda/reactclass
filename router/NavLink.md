# 🌐 `<NavLink>`란?

> `<NavLink>`는 `react-router-dom`에서 제공하는 `<Link>`의 확장 버전으로,
> \*\*현재 URL과 경로가 일치하는지(active)\*\*를 자동으로 감지하여 **활성화 스타일을 적용할 수 있는 컴포넌트**입니다.

```jsx
import { NavLink } from 'react-router-dom';

<NavLink to="/about">About</NavLink>
```

---

## ✅ 언제 사용하나요?

| 용도       | 설명                              |
| -------- | ------------------------------- |
| 네비게이션 메뉴 | "현재 위치가 이 링크에 해당하는가?"를 감지해야 할 때 |
| 탭 메뉴     | 현재 탭이 어떤 탭인지 스타일로 구분해야 할 때      |
| 사이드바     | 현재 경로와 일치하는 메뉴 항목 강조            |

---

# 🧪 기본 사용 예시

```jsx
<NavLink to="/home" className="nav-link">
  Home
</NavLink>
```

* 위 예시는 기본적으로 `<a href="/home">`로 렌더링되며,
* 현재 브라우저의 URL이 `/home`일 경우 자동으로 `"active"` 클래스를 포함합니다.

---

# 🎨 고급 스타일링: `className` or `style` as function

## 1. `className` 함수

```jsx
<NavLink
  to="/about"
  className={({ isActive }) => (isActive ? 'nav-link active' : 'nav-link')}
>
  About
</NavLink>
```

## 2. `style` 함수

```jsx
<NavLink
  to="/about"
  style={({ isActive }) => ({
    fontWeight: isActive ? 'bold' : 'normal',
    color: isActive ? 'red' : 'black',
  })}
>
  About
</NavLink>
```

| 함수 인자       | 설명                                       |
| ----------- | ---------------------------------------- |
| `isActive`  | 현재 경로와 이 링크가 일치하는가? (boolean)            |
| `isPending` | 로딩 중인 경로로 변경될 예정인가? (React Router v6.4+) |

---

# 🧠 내부 동작 방식

1. React Router는 현재 URL과 `<NavLink>`의 `to`를 비교
2. 일치할 경우 `isActive: true`
3. `className` 또는 `style` 함수가 호출되어 해당 속성이 적용됨
4. `<a>` 태그로 렌더링됨 (접근성 유지)

---

# 🔀 `end` 속성

기본적으로 `<NavLink to="/about">`는 `/about`, `/about/team`, `/about/us` 모두에서 active 상태입니다.

```jsx
<NavLink to="/about" end>
  About
</NavLink>
```

| 옵션       | 설명                              |
| -------- | ------------------------------- |
| 없음       | `/about`, `/about/xxx` 모두 일치 처리 |
| `end` 사용 | **정확하게** `/about`만 일치로 간주함      |

---

# ✅ `NavLink` vs `Link`

| 항목        | `<Link>`    | `<NavLink>`         |
| --------- | ----------- | ------------------- |
| 기본 목적     | 경로 이동       | 경로 이동 + 활성 상태 감지    |
| active 상태 | 수동 처리 필요    | 자동 감지 가능            |
| 스타일링      | 수동 class 설정 | `isActive` 기반 동적 처리 |
| UI 용도     | 일반 링크, 버튼   | 네비게이션, 메뉴, 탭        |

---

## 📦 실전 예시: 네비게이션 메뉴

```jsx
<nav>
  <NavLink to="/" end className={({ isActive }) => isActive ? 'active' : ''}>Home</NavLink>
  <NavLink to="/about" className={({ isActive }) => isActive ? 'active' : ''}>About</NavLink>
  <NavLink to="/contact" className={({ isActive }) => isActive ? 'active' : ''}>Contact</NavLink>
</nav>
```

* `isActive`에 따라 `.active` 클래스를 자동으로 적용
* 각 메뉴가 현재 경로와 일치할 때만 강조 표시됨

---

## 🧾 요약

| 항목    | 설명                                                  |
| ----- | --------------------------------------------------- |
| 정의    | 경로 일치 여부에 따라 동적 스타일을 적용하는 라우터 링크                    |
| 용도    | 네비게이션 바, 탭 메뉴 등에서 "현재 위치"를 표시할 때                    |
| 주요 기능 | `className`, `style` 속성에 함수 전달 → `isActive`에 따라 변화  |
| 비교    | `<Link>`는 단순 경로 이동, `<NavLink>`는 스타일링까지 포함          |
| 조합    | `end`, `replace`, `state` 속성 등 `<Link>`와 동일하게 사용 가능 |


