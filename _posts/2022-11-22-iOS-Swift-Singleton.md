---
title: iOS Swift 에서 Singleton 사용하기
author: Narvis2
date: 2022-11-22 13:14:00 +0900
categories: [Swift, SwiftUi]
tags: [singleton, static]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift`에서 `Singleton` 패턴을 사용하는 법에 대하여 알아보도록 하겠습니다.

## 🍀 Singleton

---

- Kotlin의 `compainion object`와 유사함
- **_👍 장점_**
  - 단 한번의 `instance`만 새성하므로 메모리 낭비를 방지할 수 있음
  - `Singleton instance`는 전역 `instance`로 다른 클래스들과 **_자원 공유_** 가 쉬움
  - `DBCP(Database Connection Pool)`처럼 공통된 객체를 여러개 생성해서 사용해야하는 상황에서 많이 사용
    > - `Thread Pool`, `Cache`, `대화상자`, `사용자 설정`, `Registry 설정`, `Log 기록 객체` 등..
- **_👎 단점_**
  - `Singleton Instance`가 너무 많은 일을 하거나, 많은 데이터를 공유시킬 경우 다른 클래스의 `Instance`들 간 결합도가 높아져 `개방=폐쇄` 원칙을 위배함(**_객체 지향 설계 원칙에 어긋남_**)
  - 따라서 유지 보수 측면에서 어려워짐
    > - 수정과 Test가 어려워짐

### ☘️ Swift 에서 Singleton 사용

- `Static`을 사용해 `Type Property`로 `Instance`를 생성하면, **_사용 시점에 초기화(`Lazy`)_** 가 되기 때문에 `Singleton Instance`가 최소 생성되기 전까진 메모리에 올라가지 않고, `Dispatch_once`도 자동 적용됨
  > - ✅ **_별도의 코드 없이 `Instance`가 여러 개 생성되지 않는, `Thread-Safe`한 방법이 됨_**
- 혹시라도 `init()`을 호출하여 `Instance`를 또 생성하는 것을 막기 위해 `init()` 함수 접근 제어자를 `private`로 지정

> **_예제_** 👇

```swift
class UserInfo {
    static let shared = UserInfo()

    var id: String?
    var password: String?
    var name: String?

    private init() {}
}

let userInfo = UserInfo.shared
userInfo.id = "YoungJun"
```
