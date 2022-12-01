---
title: iOS SwiftUi Combine 기초
author: Narvis2
date: 2022-12-01 17:20:00 +0900
categories: [Swift, Combine]
tags: [iOS, Combine, SwiftUi]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `iOS`에서 비동기를 처리할때 사용되는 `Combine`에 대하여 알아보고자 합니다.

## 🍀 Combine

---

- `Event` 처리를 위한 선언적 접근을 합니다.
- `Delegate`나 `Completion Handler` 구현 대신 `Event` 소스에 대한 `Single processing chain`을 만들 수 있습니다.
- `Apple` 버전의 `RxSwift`
- 시간의 흐름에 따라 값을 처리하기 위한 `Declarative Swift API`를 제공하는 프레임워크
- ✅ `Combine`으로 할 수 있는 대표적인 작업
  - 1️⃣ 필드에 입력한 값이 유요한 경우에만 `Submit`버튼이 활성화되도록 설정
  - 2️⃣ 비동기작업을 수행하고 반환된 값을 사용하여 `View`를 `Update`할 방법과 대상을 선택할 수 있음
  - 3️⃣ 사용자가 텍스트필드에 동적으로 입력하고 입력한 내용을 기반으로 사용자 인터페이스 `View`를 `Update`함
- ✅ 장점
  - 1️⃣ `System Level`에 통합되어 있음.
  - 2️⃣ `delegate`, `closure`를 만들 필요 없음.
  - 3️⃣ 동일한 `interface`를 쓰기 때문에 재사용성이 좋음.
  - 4️⃣ `operator`를 조합하기 좋음.
  - 5️⃣ 비동기 코드에서도 비즈니스 로직에 집중할 수 있음.
- ✅ `Combine`의 3가지 주요 부분
  - `Publisher`
  - `Operator`
  - `Subscriber`
- ✅ `Key points`
  - `Combine`은 비동기 `Event`를 위한 선언적, 반응형 프레임워크
  - 비동기 프로그래밍의 기존 문제를 해결하는 것이 목표
  - 주요 3 타입 흐름 : `publisher`(`Event` 발행) 👉 `operator`(`Event`처리, 조작) 👉 `subscriber`(결과물 소비)

### ☘️ Publisher

- **_`value`들을 내보내는(`emit`) 역할_**
- `Publisher`가 `emit`할 수 있는 `Event` 종류
  - 1️⃣ `Output`
  - 2️⃣ `Completion` : `successful completion`
  - 3️⃣ `Failure` : `completion with an error`
- ✅ **_참고_**

  > - `Publisher`는 `Output`을 안보내고 있거나 여러번 보낼 수 있으며, `Completion`이나 `Failure`를 한번 보내고 나면 더 이상의 `Event`를 보낼 수 없음
  > - 구독이 없는 경우 `Publisher`는 데이터를 제공하지 않음

- ✅ **_특징_**
  - 1️⃣ **_3가지 `Event`로_** 모든 종류의 동적 데이터를 표현 가능
  - 2️⃣ `delegate`를 추가하거나 `completion callback` 주입이 필요 없음
  - 3️⃣ `Publisher`는 `error handling`이 내장
  - 4️⃣ `Publisher`는 2개의 `Generic`을 기반으로 구성
    > - `Generic` 첫 번째 `Publisher.Ouput` 👉 `output value`
    > - `Generic` 두 번째 `Publisher.Failure` 👉 `Error`전달, `Error`가 발생할 일이 없으면, `Never`라는 `type`으로 정의하면 됨

### ☘️ Operator

- **_`Event`를 처리하고, 조작하는 역할_**
- `Publisher` `Protocol`에 선언되어 있음
- **_같거나 새로운 `Publisher`를 반환하는 메소드_**
- `Operator`를 체이닝해서 사용할 수 있기 때문에 유용함
- ✅ **_장점_**
  - 1️⃣ **_`Operator`들은 독립적이고 조합가능_** 하기 때문에, 복잡한 로직을 구현하는데 조합(`Combine`) 가능.
  - 2️⃣ 항상 `Input & Output(Upstream & DownStream)`을 가지기 때문에 `shared state`를 피할 수 있음.
    > 즉, 동시성 이슈로 인해 **_비동기 코드가 끼어들어 데이터를 중간에 변경할 일이 없음_**

### ☘️ Subscribers

- **_결과물을 소비하는 역할_**
- **_전달받은 `value`나 `completion event`로 작업을 수행_**
- 모든 구독은 `subscriber`로 끝남
- ✅ 2개의 내장된 `subscriber`
  - 1️⃣ `sink` 👉 `output value`와 `completion`을 받을 수 있는 `클로저`를 제공할 수 있음.
  - 2️⃣ `assign` 👉 `output`을 `key path`를 통해 `data model`의 `property`나 `UI control`에 바로 `binding`할 수 있음.

### ☘️ Subscriptions

- **_`publisher`, `operator`, `subcriber`의 전체 `chain`_**
- ✅ 중요
  - `subscription`의 끝에 `subscriber`를 추가 👉 `chainning`의 맨 앞에 있는 `publisher`를 활성화
  - **_`output`을 수신해줄 `subscriber`가 없으면 `publisher`는 어떤 `value`도 전달하지 않음_**
- ✅ 장점
  - 1️⃣ `Subscription`은 비동기 `Event`들의 `Chain`을 `Custom` 코드와 `Error handling`과 함께 한번에 선언 가능
  - 2️⃣ `Full-Combine`이면, 앱 전체의 로직을 `subscription`들로 표현 가능
  - 3️⃣ `Subscription`이 한번 선언되고 나면 `Callback`을 호출할 필요 없이 `System`이 다 알아서 해줌

### ☘️ 메모리 관리

- **_`Cancellable` `Protocol`을 사용해서 메모리 관리_**
- `Subscriber`들은 `Cancellable`을 준수하고 있음
- `Object`를 메모리에서 해제 👉 모든 `subscription`은 취소 👉 리소스를 메모리로부터 해제
- ✅ `subscirber`가 더 이상 값을 받을 필요 없을 때 `cancel()` 사용
  > ❗️ `cancel()`을 직접 호출하지 않으면, `deinit`될 때까지 구독됨
- ✅ 장점
  - 1️⃣ `Subscription`의 수명을 `View Controller`같은 `Object`에 `binding` 기능
  - 2️⃣ 유저가 `View Controller`를 `View Stack`에서 `dismiss 👉 subscription` 취소 해줌
