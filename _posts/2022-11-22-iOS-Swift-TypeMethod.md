---
title: iOS Swift 문법 - Type Method
author: Narvis2
date: 2022-11-22 11:13:00 +0900
categories: [Swift, Grammar]
tags: [Type Method]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Type Method`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Type Method

---

- Type 자체에서 호출되는 `Method`
- **_`instance`를 생성하지 않고도 타입 자체에서 `Method`를 호출할 수 있음_**
  > - kotlin 의 `singleton` 인 `object`/`companion object`와 비슷
- **_`method` 앞에 `static` 이나 `class`의 키워드가 붙음_**

| static func                           | class func                    |
| ------------------------------------- | ----------------------------- |
| 서브 클래스에서 `override` 할 수 없음 | 서브 클래에서 `override` 가능 |

### ☘️ class func

- `class`는 상속이 가능하나 **_`Struct`/`Enum`에서는 상속이 불가능하여 `class func`을 사용할 수 없음_**
  > ❗️ `Struct`/`Enum`에서는 오로지 `Static func`만 사용 가능
- `override` 가능

> **_예제_**

```swift
class Print {

    class func printMessage() {
        print("Hello!!")
    }
}

// Type Method로 instance를 생성해줄 필요 없이 타입 자체에서 호출할 수 있음
Print.printMessage()
```

### ☘️ static func

- `override` 불가

> **_예제_**

```swift
class Print {

    static func printMessage() {
        print("Hello!!")
    }
}

Print.printMessage()
```
