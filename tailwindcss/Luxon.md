# 📘 Luxon: 현대적 JavaScript 날짜/시간 라이브러리

Luxon은 **JavaScript의 Date API 한계를 극복하기 위해 만들어진 현대적 DateTime 라이브러리**입니다.
특히 Moment.js 팀 멤버가 개발했고, 다음 특징 때문에 가장 현대적인 선택지로 평가받습니다.

---

# 1️⃣ Luxon이 필요한 이유 (왜 Date 대신 Luxon인가?)

JavaScript `Date`는 다음과 같은 문제를 갖습니다:

| 문제             | 설명                                           |
| -------------- | -------------------------------------------- |
| ❌ 불변성 없음       | Date 객체는 mutable → 함수 호출 시 값이 바뀌어서 버그가 많이 발생 |
| ❌ 타임존 처리 약함    | 시스템 로컬 타임존에 의존, 명시적 타임존 지정이 어려움              |
| ❌ 포매팅 불편       | 복잡한 문자열 조합 필요                                |
| ❌ DST/로캘 처리 난해 | Summer Time 변화, locale 포매팅 지원이 약함            |

Luxon은 이를 다음 방식으로 해결합니다:

✔ **모든 객체가 immutable**
✔ **타임존을 명시적으로 설정**
✔ **intl API 기반의 강력한 로캘 포매팅**
✔ **명확한 API (DateTime.fromISO, toFormat 등)**
✔ **Duration, Interval 제공** (시간 간격 계산)

---

# 2️⃣ Luxon 핵심 클래스 구조

Luxon은 크게 3개 클래스로 구성됩니다:

| 클래스          | 기능                |
| ------------ | ----------------- |
| **DateTime** | 날짜·시간 객체 (핵심)     |
| **Duration** | 시간 길이(초・분・시간 단위)  |
| **Interval** | 두 DateTime 사이의 구간 |

---

# 3️⃣ DateTime 생성 방법

Luxon의 가장 중요한 기능은 **다양한 포맷으로 DateTime을 안전하게 생성**할 수 있다는 점입니다.

### ✔ 3-1. 현재 시간

```js
const now = DateTime.now();
```

### ✔ 3-2. ISO 문자열로 생성

```js
const dt = DateTime.fromISO("2024-01-10T13:45:00");
```

### ✔ 3-3. 특정 타임존에서 생성

```js
const seoul = DateTime.now().setZone("Asia/Seoul");
```

### ✔ 3-4. 객체로 생성

```js
const birthday = DateTime.fromObject({
  year: 1990,
  month: 5,
  day: 20,
  hour: 10,
  zone: "Asia/Seoul"
});
```

---

# 4️⃣ Luxon은 Immutable (불변 객체)

Luxon은 **모든 변경 메서드가 새로운 객체를 반환**합니다.

```js
const dt1 = DateTime.now();
const dt2 = dt1.plus({ days: 1 });

console.log(dt1.toISO()); // 원본 유지
console.log(dt2.toISO()); // 새로운 값
```

React에서 매우 중요합니다.
`useState`로 관리할 때 Date 객체처럼 “값이 바뀌어 버리는” 문제가 없습니다.

---

# 5️⃣ Luxon의 Format (출력 형식)

### ✔ 템플릿 기반 포매팅

```js
dt.toFormat("yyyy-MM-dd HH:mm");
```

### ✔ Locale 기반 포매팅

```js
dt.setLocale("ko").toLocaleString(DateTime.DATE_FULL);
// → 2026년 1월 24일 토요일
```

### ✔ DateTime 제공 프리셋

```js
dt.toLocaleString(DateTime.DATETIME_SHORT);
dt.toLocaleString(DateTime.DATE_MED);
dt.toLocaleString(DateTime.TIME_SIMPLE);
```

---

# 6️⃣ Duration (기간 객체)

`plus`, `minus`에 사용되는 시간 길이 객체입니다.

```js
const dur = Duration.fromObject({ hours: 2, minutes: 30 });
console.log(dur.toISO()); // "PT2H30M"

DateTime.now().plus(dur);
```

타이머, 경과 시간 표시 시 유용합니다.

---

# 7️⃣ Interval (두 날짜 사이의 구간)

```js
const start = DateTime.fromISO("2024-01-01");
const end = DateTime.fromISO("2024-01-10");

const interval = Interval.fromDateTimes(start, end);
interval.length('days'); // → 9
interval.contains(DateTime.now()); // true/false
```

React 강의에서 종종 질문받는 “기간 계산”에 매우 좋습니다.

---

# 8️⃣ 타임존 처리 강력함

Luxon은 Intl API 기반으로 타임존을 명시적으로 관리합니다.

```js
DateTime.now().setZone("Asia/Tokyo");
DateTime.fromISO("2024-03-15T12:00", { zone: "UTC" });
```

### 시스템 로컬 영향을 받지 않음 ✔

서버/클라이언트 환경에서 동일한 결과를 유지합니다.

---

# 9️⃣ Luxon 실전 예제 (React에서 자주 쓰는 패턴)

## ✔ 9-1. useEffect 안에서 Luxon 사용

```jsx
import { DateTime } from "luxon";
import { useEffect, useState } from "react";

export default function Clock() {
  const [time, setTime] = useState(DateTime.now());

  useEffect(() => {
    const id = setInterval(() => {
      setTime(DateTime.now());
    }, 1000);

    return () => clearInterval(id);
  }, []);

  return <div>{time.toFormat("HH:mm:ss")}</div>;
}
```

### → Luxon은 immutable이어서 stale closure 문제도 없고, 안전하게 쓰입니다.

---

# 1️⃣0️⃣ Luxon이 Moment.js보다 우위에 있는 이유

| 항목             | Luxon | Moment.js      |
| -------------- | ----- | -------------- |
| 불변성            | ✔     | ❌ mutable      |
| Modern Intl 기반 | ✔     | ❌              |
| 타임존 처리         | 강력    | 약함             |
| 패키지 크기         | 작음    | 큼              |
| 공식 유지보수        | 활발    | Maintenance 모드 |
| API 설계         | 모던    | 구식             |

현재 Moment.js 팀도 Luxon을 공식 대안으로 추천합니다.

---

# 1️⃣1️⃣ Luxon 사용 시 흔한 실수(Pitfalls)

| 실수                           | 설명                                                |
| ---------------------------- | ------------------------------------------------- |
| `new DateTime()` 호출          | ❌ Luxon은 생성자를 쓰지 않습니다. 항상 `fromXXX()`, `now()` 사용 |
| zone을 set하지 않음               | 기본 로컬 zone 사용 → 서버/브라우저 환경에서 다른 값 발생              |
| toJSDate()로 변환 후 Date API 혼용 | Date는 mutable → Luxon의 장점 상실                      |

---

# 1️⃣2️⃣ 가장 많이 사용하는 API 요약 (강의 슬라이드용)

### 생성

* `DateTime.now()`
* `DateTime.fromISO()`
* `DateTime.fromJSDate()`
* `DateTime.fromObject()`

### 조작

* `plus({ days: 1 })`
* `minus({ hours: 2 })`
* `set({ month: 3 })`
* `setZone("Asia/Seoul")`

### 출력

* `toISO()`
* `toFormat("yyyy-MM-dd HH:mm")`
* `toLocaleString(DateTime.DATETIME_FULL)`




요청해 주시면 바로 제작해 드리겠습니다!
