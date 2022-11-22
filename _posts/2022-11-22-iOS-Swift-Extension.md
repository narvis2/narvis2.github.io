---
title: iOS Swift 문법 - Extension
author: Narvis2
date: 2022-11-22 13:18:00 +0900
categories: [Swift, Grammar]
tags: [Extension]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Extension`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Extension

---

- Kotlin의 확장 함수와 유사
- **_기존 `Class`, `Struct`, `Enum`, `Protocol`에 새 기능을 추가_**
- 하위 `Class`를 **_생성하거나 참조하지 않고_** 그저 기능을 추가하기 위해 사용
- ✅ 특징

  - 계산된 `instance property` 및 계산 유형 속성 추가
  - `instance method` 및 `type method` 정의
  - 새 `initializer` 제공
  - `Subscript` 정의
  - 새 중첩 타입 정의 및 사용
  - 기존 타입을 `Protocol`에 맞게 설정

- ✅ Extention 선언 👇

  ```text
  extension 기존 타입이름 {
      //새로운 기능
  }

  // 익스텐션은 기존에 존재하는 타입이 추가적으로 다른 프로토콜을 채택할 수 있도록 확장할 수도 있음
  extension 확장 타입이름: 프로토콜1, 프로토콜2, 프로토콜3 {
      //새로운 기능
  }
  ```

> Double Type에 Extension 적용 **_예제_** 👇

```swift
extension Double {
    var squared: Double {
        return self * self
    }
}

let myValue:Double = 3.5
print(myValue.squared)
print(3.5.squared)
```

> **_설명_** 👇
>
> - `Double` 타입의 변수나 상수를 생성하면 `squared`라는 `property`의 기능을 사용할 수 있게 됨
> - ⚠️ 기존 타입에 새로운 기능을 추가하기 위해 확장을 정의한다면, 새로운 기능은 기존 타입의 `instnace`에서만 가능

### ☘️ Protocol을 채택할 때 사용하는 Extenstion

- `Extenstion`은 `Protocol` 채택할 때 많이 사용됨

> 하나의 `ViewController` 클래스를 만들고 이 클래스는 `UIViewController`, `UIPickerViewDelegate`, `UIPickerViewDataSource` `Protocol`을 채택 받는 **_예제_** 👇

```swift
class ViewController: UIViewController, UIPickerViewDelegate, UIPickerViewDataSource {

}
```

> **_설명_** 👇
>
> - 위의 코드와 같이 소스를 작성하게 된다면 `Class`가 너무 **_비대해진다는 단점_** 이 생김.
> - 이때 `Extention`을 이용하여 `Class`를 나누게 되면 소스가 깔끔하고 이해하기 쉬워짐

> ✅ 위 코드 개선 **_예제_** 👇

```swift
class ViewController: UIViewController{}
extension ViewController: UIPickerViewDelegate{}
extension ViewController: UIPickerViewDataSource{}
```
