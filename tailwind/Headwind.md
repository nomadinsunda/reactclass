# 🌀 Headwind(헤드윈드)란 무엇인가?

**Headwind**는 Tailwind CSS 개발자들이 가장 불편해하는 문제,
바로 **클래스 순서의 혼돈(messy class order)** 를 자동으로 해결해주는 **VS Code 확장 프로그램**입니다.

Tailwind의 유틸리티 클래스는 한 줄에 수십 개가 붙을 수 있습니다:

```html
<div class="mt-3 bg-blue-500 text-white rounded px-4 py-2 flex items-center justify-between shadow hover:bg-blue-600"></div>
```

문제는…

* 개발자마다 쓰는 순서가 다르고
* 협업 시 리뷰가 힘들고
* 스태틱 분석도 어렵고
* 자동 정렬 스타일이 없으면 계속 “지저분함”이 누적됩니다.

이 문제를 해결하기 위해 등장한 것이 바로 **Headwind**입니다.

---

# 🎯 Headwind의 목적

Headwind는 두 가지 핵심 목적을 가집니다.

## 1) **Tailwind 클래스 자동 정렬**

클래스들을 **논리적 그룹**(Layout, Flex, Spacing, Colors 등) 기준으로
자동으로 정렬해줍니다.

예:

```html
<!-- Before -->
<div class="text-white px-4 py-2 bg-blue-500 rounded mt-3 flex items-center justify-between shadow hover:bg-blue-600"></div>

<!-- After (Headwind 정렬) -->
<div class="mt-3 flex items-center justify-between px-4 py-2 rounded bg-blue-500 text-white shadow hover:bg-blue-600"></div>
```

## 2) **중복 클래스 제거**

불필요한 중복을 자동으로 제거합니다.

```html
<!-- Before -->
<div class="p-4 p-4 mt-2 bg-blue-500 bg-blue-500 text-sm"></div>

<!-- After -->
<div class="mt-2 p-4 bg-blue-500 text-sm"></div>
```

---

# 🔧 Headwind는 어떻게 동작하는가?

Headwind는 Tailwind CSS 공식 문서를 바탕으로
**유틸리티 그룹 + 우선순위 체계**를 정의하고 있습니다.

그룹 예시:

1. Layout (block, inline, hidden)
2. Flexbox/Grid (flex, items-*, justify-*, grid-cols-*)
3. Spacing (m-*, p-*)
4. Sizing (w-*, h-*)
5. Typography (text-*, font-*)
6. Backgrounds (bg-*)
7. Borders (border-*, rounded-*)
8. Effects (shadow, opacity)
9. Transitions/Transforms
10. Interactivity (cursor-pointer, select-none)
11. SVG/Accessibility
12. Tailwind Functions (apply 등)
13. Arbitrary Values

그리고 클래스 하나 하나를 파싱해서
이 그룹 우선순위에 따라 정렬합니다.

이 방식은 Tailwind 공식 가이드라인에 매우 충실하기 때문에
**전 세계 Tailwind 개발자들이 사실상 표준처럼 사용**하고 있습니다.

---

# 📦 Headwind의 주요 기능 상세 설명

## ✔ 1. 자동 정렬 (Auto-sort on Save)

파일을 저장하면 자동으로 정렬됩니다.

VS Code 설정:

```json
"editor.codeActionsOnSave": {
  "source.fixAll": true
}
```

Headwind가 설치되어 있으면 클래스 정렬이 자동 적용됩니다.

---

## ✔ 2. 중복 Tailwind 클래스 제거

클래스가 2번 이상 있을 경우 정렬 과정에서 자동으로 중복 제거합니다.

---

## ✔ 3. Prettier와 완전 호환

Tailwind용 Prettier 플러그인도 있지만
Headwind는 **Prettier와 충돌 없이** 정렬 기능만 제공하기 때문에
대부분의 팀에서 같이 사용합니다.

---

## ✔ 4. Svelte, React, Vue, Angular, HTML 모두 지원

클래스를 가진 태그를 인식하여 정렬합니다:

* JSX: `<div className="..." />`
* Vue: `<div class="..." />`
* Svelte: `<div class="..." />`
* HTML: `<div class="..." />`

---

## ✔ 5. Tailwind v3 / v4 모든 버전 지원

Arbitrary values까지 완벽하게 정렬합니다.

예:

```html
<div class="bg-[#1da1f2] p-[10px] w-[100px]"></div>
```

---

# ⚙ Headwind의 VS Code 설정 예시

`.vscode/settings.json`:

```json
{
  "headwind.runOnSave": true,
  "headwind.classRegex": "(class|className)=",
  "headwind.sortTailwindClasses": true,
  "headwind.removeDuplicates": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  }
}
```

Tailwind 관련 속성만 정렬하고
정렬 후 Prettier가 문서 포맷팅을 이어서 담당하는 구조입니다.

---

# 🚀 Headwind vs Prettier-plugin-tailwindcss 비교

| 기능             | Headwind                    | Prettier Tailwind Plugin |
| -------------- | --------------------------- | ------------------------ |
| 자동 정렬          | ✔ yes                       | ✔ yes                    |
| Prettier 필요 여부 | ❌ 독립적으로 동작                  | ✔ Prettier 필요            |
| 중복 클래스 제거      | ✔                           | ❌                        |
| 다중 파일 확장자 지원   | ✔ HTML/JSX/TSX/Vue/Svelte 등 | ✔                        |
| CI에서 사용        | ❌ VSCode 확장 기반              | ✔ Prettier CLI 기반        |
| ESLint와 통합     | ❌                           | ✔ (Prettier plugin)      |

요약:

* **개발 중에는 Headwind가 가장 빠르고 편함**
* **CI·빌드 단계에서는 Prettier Tailwind Plugin이 더 적합**

현업에서는 대부분:

> 개발 중 Headwind → commit 전에 Prettier Tailwind Plugin 적용

이렇게 병행합니다.

---

# 🎓 강의용 포인트 요약

Headwind를 학생들에게 설명할 때 이렇게 말하면 가장 효과적입니다:

> “Tailwind는 클래스가 많아서 반드시 자동 정렬이 필요하다.
> Headwind는 VS Code에서 실시간으로 정렬해주는 도구고,
> Prettier Tailwind Plugin은 저장소 수준에서 정렬을 강제하는 도구다.”

학생들은 Headwind를 쓰면 Tailwind 클래스가 과감해져서
**코드 리뷰 시 부담이 크게 줄어듭니다.**

---

# 📌 결론: 왜 Headwind를 꼭 써야 하는가?

Tailwind를 본격적으로 쓰는 순간부터
클래스 수십 개가 한 줄에 몰리기 시작합니다.

이때 Headwind가 없으면:

* 정렬 기준 제각각 → 가독성 저하
* 코드리뷰 어려움
* 충돌·중복 증가
* 유지보수 난이도 상승

Headwind는 그 모든 문제를 해결하는 **Tailwind 개발자 필수 도구**입니다.

