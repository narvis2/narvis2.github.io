---
title: iOS Swift 직렬화, 역직렬화 Codable / Decodable
author: Narvis2
date: 2022-11-25 15:26:00 +0900
categories: [Swift, Grammar]
tags: [codable, decodable, encodable]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `Swift` 에서 직렬화, 역직렬화를 하는 방법에 대하여 알아보고자 합니다.

`Android` 에서는 다음 방식으로 진행합니다.  
✅ 직렬화 : `data class` 👉 `json`  
✅ 역질렬화 : `json` 👉 `data class`

`Swift` 에서는 다음 방식으로 진행합니다.
✅ 직렬화(`Encodable`) : `struct` 👉 `json`  
✅ 역직렬화(`Decodable`) : `json` 👉 `struct`  
자세한 사항은 밑에서 알아보도록 하겠습니다.

## 🍀 Data

---

- `Codable`, `Decodable`, `Encodable`을 알아보기전 필수 개념
- `Memory` 안의 `Byte`가 저장될 수 있는 `Byte Buffer`
  > - `Byte Buffer` 👉 운영체제의 커널이 관리하는 시스템 메모리를 직접 사용할 수 있기 때문에 **_데이터의 저장, 로드가 가능_**
- `Swift`는 `URLSession`으로 `dataTask`를 만들어, `Network` 호출(`request`)을 하면 응답(`response`)으로 `Data`형을 받는데, 이는 저장, 로드, 변환이 쉽기 떄문에 `Data`로 받음
- ✅ 즉, 자주 사용되는 것은 `json` 데이터를 `struct`형으로 변경하거나, 반대로 `struct`형에서 `json`으로 변경할 때 먼저 **_`Data`형으로 변경한 다음 원하는 데이터형으로 변경하여_** 사용

## 🍀 Encodable

---

- `Data`형으로 변형할 수 있는 타입(`JSON 형`)
- ✅ 즉, `Model`을 `json` 형으로 변경( **_직렬화_** )

  > 1️⃣ **_예제_** ) `Encodable`을 준수하고 있는 `Sample` `struct` 생성 👇

  ```swift
  struct Sample: Encodable {
      let a: Int
      let b: Int
  }
  ```

  > **_✅ 설명_**
  >
  > - `Sample`은 `Encodable`을 준수하고 있으므로, `Sample`객체는 `Data`형의 객체로 변형 가능

  > 2️⃣ **_예제_** ) `struct` 👉 `json` 으로 변환 (직렬화) 👇

  ```swift
  let sample = Sample(a: 0, b: 0)

  // JSON 타입으로 인코딩할 수 있는 객체
  let jsonEncoder = JSONEncoder()

  jsonEncoder.outputFormatting = .prettyPrinted
  let data = try jsonEncoder.encode(sample)

  print(data) // 13 bytes
  print(String(data: data, encoding: .utf8)) // "{\n  \"a\" : 0,\n  \"b\" : 0\n}"
  ```

  > **_✅ 설명_**
  >
  > - `JSONEncoder` 👉 JSON 타입으로 인코딩할 수 있는 객체

## 🍀 Decodable

- `Data`형을 `sturct`와 같은 것으로 변환할 수 있는 타입
- **_주로 서버에서 받아온 `response`를 `struct`에 매칭 시킬 때 많이 사용_**
- ✅ 즉, `Json`을 `Model` 형으로 변경( **_역직렬화_** )

  > 1️⃣ **_예제_** ) `Decodable`을 준수하고 있는 `Sample2` `struct` 생성 👇

  ```swift
  struct Sample2: Decodable {
      let aVal: Int
      let bVal: Int

      enum CodingKeys: String, CodingKey {
          case aVal = "a"
          case bVal = "b"
      }
  }
  ```

  > **_✅ 설명_**
  >
  > - `Sample2`는 `Decodable`을 준수하고 있기 때문에, `Data`형을 `Sample2`에서 정의한 **_프로퍼티에 매핑이 가능_**
  > - `CodingKey` 👉 `json key`가 아닌 내가 원하는 이름으로 지정해줄 수 있게 해주는 프로토콜
  >   > - 즉, `Android` `Gson` `@SeriallizeName`과 역할이 같음
  >   > - 서버에서 내려오는 `response` 의 각 이름이 마음에 안들때 내가 원하는 이름으로 매핑시켜 주기 위해 사용
  >   > - 여기서는 `aVal` 이 `a`의 이름을 가진 `response` 와 매칭되고, `bVal`은 `b`의 이름을 가진 `response`와 매칭됨.

  > 2️⃣ **_예제_** ) `json` 👉 `struct`로 변환 (역직렬화) 👇

  ```swift
  let data = try jsonEncoder.encode(smaple)

  let jsonDecoder = JSONDecoder()
  let sample2 = try jsonDecoder.decode(Sample2.self, from: data)
  print(sample2) // Sample2(aVal: 0, bVal: 0)
  ```

## 🍀 Codable

- `Decodable`과 `Encodable`을 동시에 가지고 있는 타입
- ✅ 즉, `Decodable`과 `Encodable`이 합쳐진 것
- `struct`, `enum`, `class` 전부 채택 가능
