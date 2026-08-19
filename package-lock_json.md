

## 📦 `package-lock.json`이란?

`package-lock.json`은 **프로젝트에서 설치된 정확한 npm 패키지 버전과 의존성 트리**를 기록하는 **잠금(lock) 파일**입니다.
`npm install`을 실행하면 자동으로 생성되며, **패키지 설치의 "정확한 버전"을 고정**해줍니다.

---

## ✅ 왜 필요한가요?

### 1. **버전 일관성 보장**

* `package.json`의 버전은 범위 기반(`^1.2.3`, `~2.0.0`)이기 때문에,
* 팀원이 같은 `package.json`으로 설치해도 실제 설치된 버전은 다를 수 있음

> 🔐 그래서 `package-lock.json`은 "실제로 설치된 정확한 버전"을 고정시켜줍니다.

### 2. **빠른 설치**

* npm은 `package-lock.json`을 분석해 **캐싱된 패키지**를 빠르게 설치합니다.

### 3. **보안 및 감사 도구 연동**

* `npm audit` 등의 보안 검사 도구는 `package-lock.json`을 기반으로 취약점을 검사함

---

## 🧩 구성 예시 (요약)

```json
{
  "name": "my-app",
  "lockfileVersion": 3,
  "requires": true,
  "packages": {
    "": {
      "name": "my-app",
      "version": "1.0.0",
      "dependencies": {
        "lodash": "^4.17.21"
      }
    },
    "node_modules/lodash": {
      "version": "4.17.21",
      "resolved": "https://registry.npmjs.org/lodash/-/lodash-4.17.21.tgz",
      "integrity": "sha512-..."
    }
  }
}
```

---

## 📌 요약 비교

| 항목                 | `package.json`     | `package-lock.json` |
| ------------------ | ------------------ | ------------------- |
| 작성 위치              | 직접 작성하거나 자동 생성     | 자동 생성               |
| 목적                 | 프로젝트 정보, 의존성 범위 정의 | 실제 설치된 패키지 버전 고정    |
| 사람 눈에 보기 쉬움        | O                  | △ (기계가 주로 읽음)       |
| Git에 포함해야 하나?      | O                  | ✅ 반드시 포함해야 함        |
| `npm install`에 영향? | 버전 범위만 참고함         | ✅ 실제로 설치되는 버전 결정    |

---

## 🚫 실무 실수 예시

* `package-lock.json`을 `.gitignore`에 추가하면 팀원마다 설치된 버전이 달라져 **버그 재현이 어려움**
* 직접 수정하거나 삭제하지 말 것! → `npm install` 또는 `npm ci`가 관리해야 함

---

## 🧪 고급 명령어

| 명령어             | 설명                                                                   |
| --------------- | -------------------------------------------------------------------- |
| `npm install`   | `package.json`을 기준으로 설치, `package-lock.json`도 갱신                     |
| `npm ci`        | **CI/CD 환경용**: `package-lock.json`만 사용해서 빠르게 설치, `node_modules` 초기화함 |
| `npm audit fix` | 보안 취약점 자동 수정 (lock 파일까지 수정)                                          |


