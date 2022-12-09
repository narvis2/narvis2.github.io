---
title: JavaScript 비동기 작업 Promise, async, await 알아보기
author: Narvis2
date: 2022-12-09 17:19:00 +0900
categories: [React-Native, JavaScript]
tags: [javasciprt, async, await, promise]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `JavaScript`에서 비동기 처리에 사용되는 `promise`, `async`, `await`에 대하여 알아보도록 하겠습니다.

## 🍀 Promise

---

- 비동기 작업의 단위
- `new Promise`를 사용하여 생성
- 첫 번째 인수 `resolve` 👇
  - 비동기 작업이 성공했을 경우 호출됨
- 두 번째 인수 `reject`
  - 비동기 작업이 실패했을 경우 호출됨
- ✅ **_`new Promise(...)`의 인스턴스를 생성하는 순간 여기에 할당된 비동기 작업은 바로 실행됨_**
  > 즉, `new Promise`를 하는 순간 비동기 작업이 시작
- `then` 👉 해당 `Promise`가 성공했을 때의 동작을 지정, 함수를 받음
- `chatch` 👉 해당 `Promise`가 실패했을 때의 동작을 지정, 함수를 받음

  > **_예제_**👇

  ```javascript
  const promise1 = new Promise((resolve, reject) => {
    resolve();
  });

  promise
    .then(() => {
      console.log("then!");
    })
    .catch(() => {
      console.log("catch!");
    });
  ```

- ✅ **_재사용_** 👉 비동기 작업이 있을때 마다 `new Promise`를 할 필요 없이 `return`에 `new Promise`를 하게되면 재사용이 가능함

  > **_예제_** ) 재사용 👇

  ```javascript
  function startAsync(age) {
    return new Promise((resolve, reject) => {
      if (age > 20) resolve();
      else reject();
    });
  }

  const promise1 = startAsync(25);
  promise1
    .then(() => {
      console.log("1 then!");
    })
    .catch(() => {
      console.log("1 catch!");
    });

  const promise2 = startAsync(15);
  promise2
    .then(() => {
      console.log("2 then!");
    })
    .catch(() => {
      console.log("2 then!");
    });
  ```

- ✅ **_작업 결과 전달_** 👉 `resolve`, `reject`함수에 인자를 전달함으로써 `then` 및 `catch`함수에서 비동기 작업으로부터 정보를 얻을 수 있음

  > **_예제_** ) 작업 결과 전달 👇

  ```javascript
  function startAsync(age) {
    return new Promise((resolve, reject) => {
      if (age > 20) resolve(`${age} success`);
      else reject(new Error(`${age} is not over 20`));
    });
  }

  const promise1 = startAsync(25);
  promise1
    .then((value) => {
      console.log(value);
    })
    .catch((error) => {
      console.error(error);
    });

  const promise2 = startAsync(15);
  promise2
    .then((value) => {
      console.log(value);
    })
    .catch((error) => {
      console.error(error);
    });
  ```

## 🍀 Async

---

- 비동기 작업을 만드는 손쉬운 방법
- 함수앞에 `async` 키워드를 붙힘
- ✅ **_`async`함수의 `return`값은 무조건 `Promise`_**

  > **_예제_** `new Promise`를 `async`로 변경 👇

  ```javascript
  async function startAsync(age) {
    if (age > 20) return `${age} success`;
    else throw new Error(`${age} is not over 20`);
  }

  const promise1 = startAsync(25);
  promise1
    .then((value) => {
      console.log(value);
    })
    .catch((error) => {
      console.error(error);
    });

  const promise2 = startAsync(15);
  promise2
    .then((value) => {
      console.log(value);
    })
    .catch((error) => {
      console.error(error);
    });
  ```

## 🍀 Await

---

- ✅ **_`Promise`가 끝날 때까지 기다림_**
- `Promise`가 `resolve`되든지 `reject`되든지 끝날 때까지 기다리는 함수
- **_`await`는 `async` 함수 내부에서만 사용할 수 있음_**

  > **_예제_** 👇

  ```javascript
  function setTimeoutPromise(delay) {
    return new Promise((resolve) => setTimeout(resolve, delay));
  }

  async function startAsync(age) {
    if (age > 20) return `${age} success`;
    else throw new Error(`${age} is not over 20`);
  }

  async function startAsyncJobs() {
    await setTimeoutPromise(1000);
    const promise1 = startAsync(25);

    try {
      const value = await promise1;
      console.log(value);
    } catch (e) {
      console.error(e);
    }

    const promise2 = startAsync(15);

    try {
      const value = await promise2;
      console.log(value);
    } catch (e) {
      console.error(e);
    }
  }

  startAsyncJobs();
  ```
