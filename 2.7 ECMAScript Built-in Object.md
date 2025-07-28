
# 📘 ECMAScript 내장 객체란?

> ECMAScript 내장 객체는 **JavaScript 언어 자체**가 정의하고 제공하는 표준 객체들입니다.
> 즉, 어떤 실행 환경(브라우저, Node.js, Deno 등)에서도 **항상 존재하며**,
> JavaScript 엔진이 초기 실행 시 자동으로 메모리에 로드합니다.

이들은 JavaScript의 **기초 데이터 타입**, **언어 기능**, **비동기 처리**, **에러 처리**, **수학 연산**, **객체 조작** 등을 담당하는 핵심 구성 요소입니다.

---

## 📑 1. 정의 출처

| 항목       | 내용                                                                       |
| -------- | ------------------------------------------------------------------------ |
| 📘 정의 문서 | [ECMAScript Language Specification (ECMA-262)](https://tc39.es/ecma262/) |
| 🧠 정의 주체 | TC39 (ECMAScript 표준 위원회)                                                 |
| 🎯 목표    | JavaScript 언어의 표준화, 환경 독립성 보장                                            |

---

## 📚 2. 종류와 분류

ECMAScript 내장 객체는 다음처럼 **카테고리별로** 분류됩니다:

### ✅ 기본 객체

| 객체         | 설명               |
| ---------- | ---------------- |
| `Object`   | 모든 객체의 부모        |
| `Function` | 함수 정의            |
| `Boolean`  | true/false 값을 다룸 |
| `Symbol`   | 유일한 식별자 생성       |
| `BigInt`   | 정수 한계를 넘는 큰 수 표현 |

---

### ✅ 텍스트 및 정규식

| 객체       | 설명     |
| -------- | ------ |
| `String` | 문자열 처리 |
| `RegExp` | 정규 표현식 |

---

### ✅ 수학, 날짜

| 객체     | 설명         |
| ------ | ---------- |
| `Math` | 수학 연산 유틸리티 |
| `Date` | 날짜 및 시간 조작 |

---

### ✅ 숫자 및 배열 관련

| 객체              | 설명                                        |
| --------------- | ----------------------------------------- |
| `Number`        | 숫자 처리                                     |
| `Array`         | 배열 자료형                                    |
| `TypedArray` 계열 | `Int8Array`, `Float32Array` 등: 이진 데이터 처리용 |

---

### ✅ 에러 처리

| 객체                                             | 설명               |
| ---------------------------------------------- | ---------------- |
| `Error`                                        | 기본 오류            |
| `TypeError`, `ReferenceError`, `SyntaxError` 등 | 다양한 오류 타입별 객체 제공 |

---

### ✅ 구조화 데이터

| 객체                    | 설명                   |
| --------------------- | -------------------- |
| `Map` / `Set`         | 키-값 및 유일값 컬렉션        |
| `WeakMap` / `WeakSet` | 약한 참조 기반 컬렉션 (GC 고려) |

---

### ✅ 비동기/동시성

| 객체              | 설명                      |
| --------------- | ----------------------- |
| `Promise`       | 비동기 처리                  |
| `AsyncFunction` | `async` 키워드로 생성되는 함수 타입 |

---

### ✅ 기타

| 객체                        | 설명               |
| ------------------------- | ---------------- |
| `JSON`                    | 직렬화/역직렬화         |
| `Reflect`                 | 객체 조작을 함수형 API로  |
| `Proxy`                   | 객체에 대한 트랩/캡슐화 제공 |
| `eval()`                  | 문자열을 코드로 실행      |
| `parseInt()`, `isNaN()` 등 | 전역 함수로 포함됨       |

---

## 🧠 3. JavaScript가 실행될 때 어떻게 로딩되나?

1. JS 엔진(V8 등)이 실행되면
2. **ECMAScript 사양에 정의된 모든 내장 객체를 전역으로 초기화**합니다
3. 사용자 코드가 실행되기 전에 `Object`, `Array`, `Function`, `Math`, `Promise` 등은 모두 이미 메모리에 존재합니다

---

## ❗ 4. 브라우저의 Web API와 구분해야 하는 이유

| 항목            | ECMAScript 내장 객체                     | Web API (브라우저 제공)                             |
| ------------- | ------------------------------------ | --------------------------------------------- |
| 정의 위치         | ECMAScript (TC39)                    | WHATWG / W3C (DOM, Fetch 등)                   |
| 실행 환경         | JS 엔진이 존재하는 모든 환경                    | 브라우저, 일부 플랫폼                                  |
| 예시            | `Array`, `Math`, `Promise`, `Object` | `document`, `window`, `fetch`, `localStorage` |
| Node.js 사용 여부 | ✅ 사용 가능                              | ❌ 기본적으로 없음 (`fetch`는 최근 일부 환경에 포함)            |

---

## 🔍 5. 예제 코드

```js
// ECMAScript 내장 객체들
const arr = new Array(1, 2, 3); // Array
const now = new Date();         // Date
const max = Math.max(10, 20);   // Math
const json = JSON.stringify({ a: 1 }); // JSON
const promise = new Promise((resolve) => resolve("OK")); // Promise
```

```js
// Web API (브라우저 환경에서만 가능)
document.querySelector('h1');
localStorage.setItem('key', 'value');
fetch('/api/data');
```

---

## ✅ 요약 정리

| 구분                  | 설명                                          |
| ------------------- | ------------------------------------------- |
| 📌 ECMAScript 내장 객체 | JavaScript 언어 자체에 내장된 전역 객체들 (표준)           |
| ✅ 항상 사용 가능          | 브라우저, Node.js, 어디서든 기본 탑재                   |
| ❌ Web API 아님        | DOM, `document`, `window`, `fetch`는 포함되지 않음 |
| 🎯 목적               | 언어의 핵심 기능 지원 (배열, 함수, 비동기, 에러 등)            |

---
<br/>
---

# ✅ ECMAScript 내장 객체나 Web API들이 필요한 이유는???

> **ECMAScript 내장 객체나 Web API는 컴파일 타임에 존재하지 않기 때문에,
> 런타임에 JS 엔진이 이를 동적으로 제공해야 합니다.**
> 따라서 이런 내장 객체들이 **디폴트 전역 객체(Global Object)** 로 설계된 것입니다.

---

## 1. JavaScript는 컴파일 타임이 "고정되어 있지 않다"

* JavaScript는 전통적으로 **인터프리터 언어**였고,
* 최신 JS 엔진(V8 등)은 **JIT (Just-In-Time)** 컴파일을 사용하긴 하지만,
* 여전히 **정적인 타입 정보나 객체 구조를 컴파일 시점에 알 수 없습니다.**

### 예시

```js
const result = JSON.stringify({ x: 1 });
const el = document.createElement('div');
```

* 이 코드의 의미는 **런타임에 `JSON`과 `document`가 존재한다는 가정 하에서 실행**됩니다.
* 컴파일 시점에는 `document.createElement()`가 실제로 무엇을 반환할지 알 수 없습니다.

---

## 2. 런타임에 제공되어야 할 이유

### ✔️ JavaScript는 동적 언어

* 객체의 속성, 타입, 구조를 **런타임에 변경할 수 있음**
* 따라서 내장 객체(`Object`, `Array`, `Promise`)도 런타임에 만들어져야 함

### ✔️ 실행 환경에 따라 존재 여부가 달라짐

| 환경      | `Array`, `Math`, `Date` | `document`, `fetch`, `localStorage` |
| ------- | ----------------------- | ----------------------------------- |
| 브라우저    | ✅ 있음                    | ✅ 있음                                |
| Node.js | ✅ 있음                    | ❌ 없음 (DOM 없음)                       |
| Deno    | ✅ 있음                    | ⭕ 있음 (Web API 일부 지원)                |

> 즉, 브라우저는 자체적으로 DOM API 객체들을 **런타임에 window/document 아래 등록**해서 사용 가능하게 만듭니다.

---

## 3. ECMAScript 내장 객체는 “엔진이 부팅될 때 메모리에 올려짐”

### 예시: `Array`, `Math` 등은 다음처럼 초기화됩니다.

```js
globalThis.Array = ... // Array 생성자
globalThis.Math = ...  // Math 객체
globalThis.Promise = ... // Promise 함수
```

→ 즉, ECMAScript 내장 객체들은 **런타임 시작 시점에 전역 객체(globalThis)에 등록**되므로
→ JS 코드가 실행되기 전부터 존재하게 됩니다.

---

## 4. 타입스크립트와 비교해 보면 더 명확해짐

### TypeScript는 정적 타입 정보만 제공할 뿐,

* 컴파일 타임에는 실제 `document`나 `window` 같은 객체는 존재하지 않음
* 런타임에는 여전히 **브라우저가 그것을 메모리에 등록해줘야 함**

```ts
// TypeScript는 단지 여기에 대해 타입만 제공할 뿐(자바의 제너릭이 사실상 컴파일타임 때 타입 검사 역할을 하는것처럼):
declare const document: Document;
```

---

## ✅ 요약 정리

| 항목                       | 이유                                        |
| ------------------------ | ----------------------------------------- |
| ECMAScript 내장 객체가 필요한 이유 | JS 엔진이 **기본 언어 기능**을 제공하기 위해              |
| Web API가 런타임에 존재해야 하는 이유 | 환경에 따라 DOM, Fetch 등 API가 달라지므로            |
| 컴파일 타임에 존재하지 않는 이유       | JS는 정적 타입/정적 구조가 없기 때문                    |
| 결국                       | **모든 전역 객체는 런타임에 엔진 또는 호스트 환경이 메모리에 등록**함 |



