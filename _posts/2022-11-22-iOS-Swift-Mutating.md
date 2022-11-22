---
title: iOS Swift 문법 - Mutating
author: Narvis2
date: 2022-11-22 10:21:00 +0900
categories: [Swift, Grammar]
tags: [Mutating]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Mutating`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Mutating

---

- `값 타입`인 `Struct`에서는 **_`instance` `method` 내에서 `property`들을 수정할 수 없게 되어 있음_**
  > ❗️ Error ❗️
  >
  > - Cannot assign property 👉 `값 타입`인 `Struct`에서는 `method`안의 값을 수정할 수 없기 때문에 발생
- 이러한 **_`property`들을 `Struct`안의 `method`에서 `수정`을 해주기 위해_** `mutating`이라는 키워드를 사용
- ✅ `mutating` 👉 특정 `method`내에서 `Struct` 또는 `Enum` `property`를 수정해야 하는 경우 해당 `method`의 동작을 변경하도록 하는 것
- `Struct`나 `Enum`을 쓸때는 정말 중요한 키워드

> **_예제_** 👇

```swift
struct Person {
    let name: String
    var age: Int

    init (name: String, age: Int) {
        self.name = name.uppercased()
        self.age = age
    }

    // mutating 키워드를 사용하지 않으면 Cannot assign property Error 발생
    mutating func changeAge() {
        age = 10
    }
}

var choi = Person(name: "choi", age: 29)
choi.changeAge()
print(choi.age)
```
