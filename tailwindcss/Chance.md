

# 🎲 **chance는 “무작위 데이터(Random Data)”를 생성하는 JavaScript 라이브러리입니다.**

Luxon이 날짜/시간을 다루는 라이브러리인 것처럼,
**Chance.js**는 다양한 종류의 랜덤 데이터를 손쉽게 만들어주는 **유틸리티 라이브러리**입니다.

---

# ✔ Chance.js 한 줄 정의

> **이메일, 이름, 주소, 번호, GUID, 시드값 기반 난수 등 각종 랜덤 데이터를 생성하는 라이브러리.**

테스트용 mock 데이터를 만들어야 할 때 매우 유용합니다.

---

# ✔ Chance.js가 제공하는 기능

| 범주       | 예시                                            |
| -------- | --------------------------------------------- |
| 🔢 숫자    | 랜덤 int, float, dice(주사위), normal distribution |
| 🧑 이름 관련 | firstname, lastname, full name                |
| 🌐 인터넷   | email, url, ip, domain                        |
| 🏙 주소    | city, country, zip, street                    |
| 📝 문자열   | character, string, paragraph                  |
| 🔐 ID    | guid, hash, nanoid 비슷한 unique ID              |
| 📱 전화번호  | phone                                         |
| 📆 날짜    | birthday 등 날짜 관련                              |
| 🎲 기타    | color, animal, company, profession 등          |

---

# ✔ 설치

`npm i chance` 로 설치하면 됩니다.

---

# ✔ 사용 예제(기본)

```js
import Chance from 'chance';

const chance = new Chance();

console.log(chance.name());       // 랜덤 이름
console.log(chance.email());      // 랜덤 이메일
console.log(chance.age());        // 나이
console.log(chance.guid());       // GUID
console.log(chance.integer({ min: 1, max: 100 }));
```

---

# ✔ React + Vite 환경에서 자주 쓰는 예

## mock 데이터 만드는 데 최고

```js
import Chance from 'chance';

const chance = new Chance();

const users = Array.from({ length: 10 }).map(() => ({
  id: chance.guid(),
  name: chance.name(),
  email: chance.email(),
  age: chance.age(),
}));

console.log(users);
```

→ **더미 리스트, 테이블 테스트 데이터, Chart.js 더미 데이터 등을 만들 때 훨씬 편합니다.**

---

# ✔ 왜 chance를 luxon과 같이 많이 쓰나?

Luxon으로 날짜 처리 + Chance로 랜덤 데이터를 생성하여
**"랜덤 날짜 + 랜덤 이름 + 랜덤 숫자"가 필요한 mock database**를 만들기 좋기 때문입니다.

예:

```js
import { DateTime } from 'luxon';
import Chance from 'chance';

const chance = new Chance();

const todos = Array.from({ length: 5 }).map(() => ({
  id: chance.guid(),
  title: chance.sentence(),
  createdAt: DateTime.now().minus({ days: chance.integer({ min: 1, max: 30 }) }).toISO(),
}));
```

→ 실제 서비스 같은 더미 데이터를 쉽게 구성할 수 있습니다.

---

# ✔ 결론

### 👉 **chance = Random Data Generator**

* 테스트용 mock data 생성
* 랜덤 문자열/숫자/날짜/이름/주소 등 생성
* React/Vite 개발에서 더미 데이터 만들 때 매우 자주 사용됨


