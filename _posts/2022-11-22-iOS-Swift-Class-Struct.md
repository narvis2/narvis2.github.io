---
title: iOS Swift 문법 Class / Struct
author: Narvis2
date: 2022-11-22 09:02:00 +0900
categories: [Swift, Grammar]
tags: [class, struct]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `class`와 `struct`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Class

---

- `Swift`에서는 `class`를 정의하여 `객체(Object)`를 만들고 사용할 수 있음
- `class` 하나를 만든다면 `class`에서 생성된 객체인 **_instance_** 를 만들어 실제 작업에 쓰일 수 있게 해야함
  > - `instance` 👉 클래스 초기화, (타입 캐스팅 허용)
- `상속` 가능
- ✅ **_참조 타입_**
  > - 변수나 상수에 할당하거나 함수에 넘길 때 복사하지 않음
  > - 복사 대신에 기존에 같은 `instance`에 참조가 사용됨
  > - 즉, 값이 복사되는 것이 아닌 **_메모리를 참조하는 것_**
- `속성(property)` 👉 클래스 안의 `변수`
- `메서드(method)` 👉 클래스 안의 `함수`
- ✅ `init` 👉 `instance`를 만들때 생성자에 `인수`를 넣을 때 사용
  > - `self` 👉 Kotlin 의 `this` 와 유사

> **_예제_** 👇

```swift
class Name {
    var name = "Choi"
    var age: Int

    init(name: String, age: Int) { // 초기화
        self.name = name
        self.age = age
    }

    func myName() {
        print("my name is \(name) and \(age) year's old")
    }
}

// instance 화
let name1: Name = Name(name: "choi", age: 29)
let name2: Name = Name(name: "kim", age: 31)

// 프로퍼티 호출
print(name1.name)
// 메서드 호출
name1.myName()

print(name2.name)
name2.myName()
```

## 🍀 Struct

---

- `instance`의 `값(프로퍼티)`를 `저장`하거나 `기능(메서드)`를 제공하고 이를 `캡슐화`할 수 있는 `Swift`가 제공하는 `Type`임
  > - `class` 처럼 `instance`화를 하여 실제 작업에도 쓸 수 있음 (타입 캐스팅 허용 안함)
- `속성(property)` 👉 구조체(`Struct`) 안의 변수
- `메소드(method)` 👉 구조체(`Stuct`) 안의 함수
- ✅ **_값 타입_**
  > - 상수나 변수에 할당하거나 함수에 넘겨질 때 복사가 됨
- ✅ **_`Stuct`에는 상속할 수 없음_**
- ✅ `class` 처럼 `init()`메소드를 사용할 필요 없이 **_자동으로 초기화 코드를 만들어줌_**
  > - 프로퍼티 속성값을 선언하지 않고 instance 선언시 매개변수로 넣어줌
  > - 구조체 멤버를 패러미터 네임으로하여 **_`Swift`가 자동으로 초기화 코드를 만들어 줌_**
- ✅ **_사용 기준_** 하나라도 해당되면 `Stuct` 사용
  - 1️⃣ 몇몇 **_단순 데이터 값을 `캡슐화`하는 경우_**
  - 2️⃣ `캡슐화`한 값을 `참조`하는 것보다 `복사`하는 것이 합당할 때
  - 3️⃣ `Stuct`에 저장된 `프로퍼티`가 `값 타입`이며, `참조`하는 것보다 `복사`하는 것이 합당할 때
  - 4️⃣ **_다른 `기존 타입`으로부터 `상속`받거나 자신을 `상속`할 필요가 없을 때_**

> **_예제_** 👇

```swift
struct Name {
    // 프로퍼티 속성값을 선언하지 않고 instance 선언시 매개변수로 넣어줌
    var name: String

    func myName() {
        print("my name is \(name)")
    }
}

// instance 생성
var choi: Name = Name(name: "choi")

print(choi.name) // 프로퍼티 호출
choi.myName() // 메서드 호출
```

## 🍀 Class / Struct 공통점 및 차이점

---

### ☘️ 공통점

- 여러 변수(`property`)와 함수(`method`)를 담을 수 있는 하나의 집합
- 데이터를 용도에 맞게 묶어서 사용할때 편리하고 가독성을 높여줌
- 초기화(`init()`)를 정의하여 여러 매개변수에 대해 다양한 `instance`를 생성 가능
- 기본적인 구현을 넘어선 기능을 확장시킬 수 있도록 확장 가능
- `.`을 사용하여 `instance`생성이 가능하고 생성 방법이 같음
- 특정 종류의 표준 기능을 제공하는 `protocal`을 사용 가능
- 새로운 데이터 타입을 만들어 주는 것

### ☘️ 차이점

- `Struct`에는 `init()` `method` 없이 **_자동으로 초기화 함수를 만들어 줌_**
- `Class` 👉 상속 가능, `Struct` 👉 상속 불가
- `Class` 👉 **_값 타입_** , `Struct` 👉 **_참조 타입_**
- `TypeCasting(타입캐스팅)`은 `Class`의 `instance` 에만 허용
- `Deinitializer`는 `class`의 `instance`에만 활용 가능
- 참조 횟수 계산(Reference Counting)은 `Class`의 `instance`에만 적용

### ☘️ 값 타입 / 참조 타입

- `값` 타입
  > - `Stuct`, `enum` 에 해당
  > - **_`상수`나 `변수`에 `할당`하거나 `함수`에 넘겨질 때 `복사`가 됨_**
- `참조` 타입
  > - `Class` 에 해당
  > - **_`변수`나 `상수`에 `할당`하거나 `함수`에 넘길 때 `복사`하지 않음_**
  > - 복사 대신에 기존에 같은 `instance`에 참조가 사용됨
  > - 즉, 값이 복사되는 것이 아닌 **_메모리를 참조하는 것_**

> **_예제_** 👇

```swift
struct A { // 구조체 (상속이 안됨)
    var a = 10
}

class B { // 클래스 (상속 가능)
    var a = 10
}

// instance 선언
var str_1: A = A() // 값을 복사 - a 값을 바꿔도 struct 안의 a는 변화 없음
var cls_1: B = B() // 값을 참조 - a 값을 바꾸면 class 안의 a도 바뀜

var str_2 = str_1
var cls_2 = cls_1

str_1.a = 20
cls_1.a = 20

// 구조체에는 변화 없지만 클래스에는 변화가 있음
print("\(str_2.a) \(cls_2.a)") // 결과 : 10 20
```

> **_설명_**
>
> - `Struct(구조체)`는 그저 `복사`를 하여 **_`str_1`과 `str_2`의 관계는 참조되지 않고 각각의 `개별`의 관계_**
> - `Class(클래스)`는 서로 `참조`하는 관계를 가지므로 **_`cls_1`과 `cls_2`는 `참조 관계`가 되어 서로에게 영향을 줌_**
