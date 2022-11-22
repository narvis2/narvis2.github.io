---
title: iOS Swift 문법 - Optional Binding
author: Narvis2
date: 2022-11-22 10:21:00 +0900
categories: [Swift, Grammar]
tags: [Optional Binding, Optional, if let, guard let]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Optional Binding`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Optional Binding

---

- 강제로 `Optional`을 여는 방식(`!`)가 아닌 안전하게 확인을 해보고 `unwrapping`을 하는 방법
- **_`if`문을 이용하여 `Optional`에 할당된 값을 임시 변수 또는 상수에 할당을 해주는 방식_**

| Force unwrapping                 | Optional Binding                            |
| -------------------------------- | ------------------------------------------- |
| `!`를 써서 상제로 `Optional`추출 | `if let`, `guard let`을 써서 `Optional`추출 |

- 만약 `Optional`에 값이 있다면 `if`문 안으로 들어가게 되고, `nil`이라면 그냥 통과하게 되는 방식

### ☘️ if let (지역 변수로만 사용가능)

- **_지역 변수로만 사용 가능_**
- 즉, `if` 문 블럭 내부에서만 사용 가능
  - **_할당 상수를 if문 안에서만 사용_**

> **_예제_** 👇

```swift
let x: Int? = 10
let y: Int? = nil

if let xx = x {
    print("x = \(xx)")
} else {
    print("x is Optional")
}

if let yy = y {
    print("y = \(yy)")
}
```

> **_설명_** 👇
>
> - 값이 있는 `x`는 `xx`에 대입되어 `if`문을 수행하게 되고, 값이 `nil`인 `y`는`yy`에 대입하지 못하고 그냥 통과함

### ☘️ 여러 Optional Binding

- `Optional Binding` 할때는 `여러 변수`나 `상수` 또한 같이 할 수 있음
  > `,`로 구분지어서 사용

> **_예제_** 👇

```swift
let name1: String?
let name2: String?

name1 = "choi"
name2 = "kim"

if let nameFirst = name1, let nameSecond = name2 {
    // name1, name2 가 Optinal이 아닌 경우 해당 로직 실행
    print(nameFirst, nameSecond)
}
```

### ☘️ guard let (전역 변수로 사용가능)

- `guard let` 에서는 **_`else`인 부분만 작성_** 이 가능
  - 즉, 값이 `nil`이여서 `Optional` 추출이 되지 않을때만 어떠한 행동을 취할 수 있음
  - 만약 `nil`값이 아닐 걸 확인하고 `Optional`을 성공적으로 추출했다면 `guard let`문을 통과
- ✅ **_전역변수로 사용 가능_**
  - 즉, **_`guard let`문 밖에서도 할당 상수를 자유롭게 사용 가능_**
- `guard let`의 `else`문
  > - ❗️항상 `return` 이나 `throw`문이 와야함

> **_예제_** 👇

```swift
let x: Int? = 10
let y: Int? = nil

func opbinding() {
    guard let xx = x else {
        return print("x is Optional")
    }
    print(xx)

    guard let yy = y else {
        return print("y is Optional")
    }
    print(yy)
}

opbinding()
```
