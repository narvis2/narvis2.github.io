---
title: iOS Swift 문법 - Protocol
author: Narvis2
date: 2022-11-22 10:02:00 +0900
categories: [Swift, Grammar]
tags: [protocol]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Protocol`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Protocol

---

- Kotlin 의 `interface` 와 비슷합니다.
- **_특정 작업 또는 기능에 적합한 메서드, 속성 및 기타 요구 사항의 `Blueprint`를 정의한다_**
- `Protocol` 👉 요구사항
- **_`Protocol`에는 구현 내용은 들어가 있지 않고, 채용한 타입이 직접 구현_**
- `Protocol`을 채용한 형식은 **_`요구사항`을 반드시 모두 구현_**
- `Protocol`은 `Protocol`간 상속을 지원, `class`와 달리 **_다중 상속_** 도 지원
- ℹ️ **_사용 이유_**
  - `swift`는 `protocol` 지향 프로그래밍임
  - `protocol` 초기 구현이 `protocol` 지향 프로그래밍의 핵심
  - **_`swift`에서는 `class`만 `상속`이 가능하고 `class`는 `참조 타입`이므로 참조 추적에 비용이 많이 발생_**
    > - 따라서 비교적 비용이 적은 `값 타입`을 활용하고 싶어도, 상속을 할 수 없으므로 때마다 기능을 다시 구현해 주어야 한다는 불편함이 있음
  - `protocol`을 사용하면 `상속`이라는 한계점을 탈피할 수 있음
    > - 초기에 구현해 놓은 많은 `property`나 `method`를 우리가 쉽게 채택하여서 사용을 할 수 있다는 큰 장점이 있음

### ☘️ Protocol property

- `Protocol`에서 `property(속성)`를 정의할 때에는 `get` 과 `set` 키워드를 사용해 `property(속성)`가 `읽기 전용 property(속성)`인지 `쓰기 property(속성)`인지를 **_반드시 명시_** 를 해줘야 함
  > - {get, set} 👉 읽기 / 쓰기 `property`
  > - { get } 👉 읽기 전용 `property`

> **_예제_** 👇

```swift
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}
```

### ☘️ Protocol method

- `method` 의 본문은 포함하지 않음

> **_예제_** 👇

```swift
protocol RandomNumberGenerator {
    func random() -> Double
}

class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0

    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))

        return lastRandom / m
    }
}
```
