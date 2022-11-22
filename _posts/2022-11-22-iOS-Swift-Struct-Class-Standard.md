---
title: iOS Swift 문법 - Struct / Class 선택 기준
author: Narvis2
date: 2022-11-22 17:43:00 +0900
categories: [Swift, Grammar]
tags: [struct, class]
---

안녕하세요. narvis2 입니다.  
이번시간에는 지난 `Struct` / `Class` 강의에 이어 `Struct`과 `Class`의 선택 기준을 알아보도록 하겠습니다.

## 🍀 Struct / Class 선택기준

---

- ✅ 기준
  - 1️⃣ 기본적으로 `Struct`를 사용함
    > - 안전성 측면에서 **_가능한 한 `struct`를 사용_**
  - 2️⃣ `Object-C`코드와 연계가 필요하면 `class`를 사용
  - 3️⃣ `Data`의 `identity`를 다뤄야 한다면 `class`를 사용 (`참조방식`)
  - 4️⃣ `상속`이 필요할 경우 👉 `struct` + `protocol` 조합을 사용
    > - `protocol`은 `class`와 달리, `sturct`, `class`, `enum` 모두와 상속 관계를 지을 수 있어 선호됨
  - 5️⃣ 상속이 필요하지 않고 모델이 크지 않으면 `struct` 사용
  - 6️⃣ `DB`연결 처럼 `App` 전체 코드에서 상황을 공유해야 할 때 `class` 사용
  - 7️⃣ `json`파싱할 경우 `struct`사용
  - 8️⃣ `serialize`해서 전송하거나 파일로 저장할 일이 있다면 `class` 사용

### ☘️ 기본적으로 struct 사용

- 일반적인 종류의 `Data`를 보여줄 때 `struct`를 사용하면 됨
- `Swift`에서 `struct`는 다른 언어에서 `class`에게만 허용하는 여러 `feature`들을 사용할 수 있음
- `struct`를 사용하면 `app`의 전체 상태를 고려할 필요없이 코드의 일부에 대해 더 쉽게 추론할 수 있음
  > - `struct`는 **_값 타입_** 이기 때문에 `struct`의 지역적인 변화는 의도적으로 알리지 않는 한 `app`의 다른 부분에서는 보이지 않음.
  > - ✅ 결과 적으로 `code section`에서는 인스턴스에 변화를 줄 수 없기에 **_code의 한 부분에만 집중할 수 있음_**
- `참조 타입`인 `class`를 사용하는데 만약 `system`이 크고 복잡하다면, 서로를 뒤죽박죽 참조해댈 수도 있고 **_아무곳에서나 `Instance`를 변경할 수 있기에 코드 전체에 신경을 써야함_**

### ☘️ Object-C 코드와 연계가 필요할 경우 class 사용

- 만약 `data`를 처리하기 위해 `Objective-C API`를 사용해야 한다거나, `Objective-C` `Framework`에 "`class`로 정의된" 타입으로 `data modeling`을 해야 한다면, `class`와 `class` 상속을 사용

### ☘️ Data의 identity를 다뤄야 한다면 class를 사용 (참조방식)

- `app`간 `class instance`를 공유할 때, 이 `instance`를 변경하면 이를 참조하는 코드의 다른 모든 부분에서도 이 변화가 보이길 원한다면 `class`사용

  > - `file handles`, `network connections`, `shared hardware intermediaries` 등등..
  > - 만약 `local DB`연결을 나타내는 타입을 가지고 있다면, `DB`접근을 관리하는 코드는 `DB`의 상태를 완전히 제어해야함 이 경우 `class`를 사용하는게 적절함

- **_`identity`를 가지더라도 제어할 필요가 없으면 `struct`를 사용_**
  > - `modeling` 중인 `data`가 `entity`에 대한 정보이며, 이 `entity`에 `identity`가 들어있지만 `control`이 필요없다면 `struct`를 사용

### ☘️ 상속이 필요할 경우 struct + protocol 조합을 사용

- `protocol` 상속 + `struct` 조합으로 상속계층을 만들 수 있음
  > - `protocol`은 `class`, `struct`, `enum` 모두가 상속에 참여할 수 있도록 허용
  > - `class` 상속은 `class` 끼리만 가능
- ✅ **_`data`를 어떻게 `modeling`할지 고르고 있다면, 우선 `protocol` 상속을 쓰고 `struct`가 채택하도록 설정_**
