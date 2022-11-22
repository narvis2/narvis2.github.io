---
title: iOS Swift 문법 - Closure
author: Narvis2
date: 2022-11-22 11:13:00 +0900
categories: [Swift, Grammar]
tags: [closure]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 문법에 중에서도 `Closure`에 대하여 알아보는 시간을 가지도록 하겠습니다.

## 🍀 Closure

---

- 이름 없는 함수 (Kotlin 의 `람다`와 비슷, 즉 익명 함수)
  > - 일반 함수 👉 이름이 있는 `Closure` 함수
- `인자`들을 넣을 수 있고, `반환타입`까지 설정을 해줄 수 있음
- 함수의 내용 👉 `in` 다음에 수행 내용 적기

> 일반함수 **_예제_** 👇

```swift
func add(x:Int, y:Int) -> Int {
    return (x+y)
}

print(add(x:10, y:20))
```

> `Closure` 함수 **_예제_** 👇

```swift
let add1 = {(x: Int, y:Int) -> Int in
    return(x+y)
}

print(add(10, 20))
```

### ☘️ 후행 클로저(trailing closure)

- `closure`가 _함수의 마지막 `argument`라면_ **_마지막 매개변수 이름을 생략한 후_** 함수 소괄호 외부에 `closure`를 구현할 수 있게
  > - ✅ Kotlin 의 `람다`와 같음

> **_예제_** 👇

```swift
//후행 클로저 미사용
let onAction = UIAlertAction(title: "On", style: UIAlertAction.Style.default, handler: {
    //실행 코드
})

//후행 클로저 사용
let onAction = UIAlertAction(title: "On", style: UIAlertAction.Style.default) {
    //실행 코드
})

```

### ☘️ Closure 의 축약 표현

- 1️⃣ 타입 생략
  - 👉 `closure`는 `method`에서 요구하는 형태로 전달해야 함
  - 👉 `swift`는 이러한 문맥을 이용해 타입을 유추할 수 있음(타입 추론)
  - 👉 `매개변수`의 `타입`이나 `반환 타입`을 **_생략해서_** `closure`를 사용할 수 있게 됨
- 2️⃣ `return` 생략 👉 `return` 도 생략 가능
- 3️⃣ 매개변수 생략
  - 👉 매개변수의 이름도 생략 가능
  - 👉 매개변수의 이름을 명시하지 않아도 `$`와 `숫자`의 조합으로 단축 인자 이름을 사용할 수 있음
  - 👉 `$0` 첫 번째 매개변수
  - 👉 `$1` 두 번째 매개변수
  - 👉 `in` 키워드도 생략 가능
- 4️⃣ 연산자만 표기
  - 👉 매개변수의 타입과 반환 타입이 연산자를 구현한 함수의 모양과 동일하다면, 연산자만 표기하더라도 알아서 연산하고 반환해줌

> **_예제_** 👇

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]

// 일반 closure
let reversed = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})

// 1️⃣ 매개 변수 타입, 반환 타입 생략
let reversed2 = names.sorted(by: { (s1, s2) in
    return s1 > s2
})

// 2️⃣ Return 생략
let reversed3 = names.sorted(by: { (s1, s2) in
   	s1 > s2
})

// 3️⃣ 매개변수, in 키워드 생략
let reversed4 = names.sorted(by: {
    return $0 > $1
})

// 4️⃣ 연산자만 표기
let reversed5 = names.sorted(by: >)
```
