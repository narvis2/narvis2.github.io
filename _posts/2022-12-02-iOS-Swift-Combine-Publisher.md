---
title: iOS SwiftUi Combine Publisher 종류
author: Narvis2
date: 2022-12-02 10:20:00 +0900
categories: [Swift, Combine]
tags: [iOS, Combine, SwiftUi, Publisher]
---

안녕하세요. narvis2 입니다.  
[지난시간](https://narvis2.github.io/posts/iOS-Swift-Combine-Basic/)에는 `iOS`에서 비동기를 처리할때 사용되는 `Combine`에 대하여 간단히 알아보았습니다.  
이번시간에는 지난 시간에 이어 간단한 예제를 통해 **_`Publisher`의 종류_** 에 대하여 알아보고자 합니다.

## 🍀 Publisher의 종류

---

### ☘️ Just

- 가장 단순한 형태의 `Publisher`
- 단일 `Event` 발생 후 종료되는 `Publisher`
- **_`Error` 타입은 항상 `Never`_**
  > **_예제_** 👇
  ```swift
  Just((0...5)).sink { value in
      print(value) // 0...5
  }
  ```

### ☘️ Future

- 일반적으로 `Publisher`의 처리를 `sink`라는 구독을 형태로 많이 처리하게 되는데 이 때 `Closure`를 전달하는 과정에서 **_`Callback` 기반의 `completion` 핸들러를 사용하게 되는데 `Future`를 통하여 더욱 깔끔한 코드 작성_** 이 가능
- 단일 `Event`와 종료 혹은 실패를 제공하는 `Publisher`

  > 1️⃣ **_예제_** ) 간단한 사용 👇

  ```swift
  let myFuture = Future<Int, Never> { promise in
      promise(.success(10))
  }
  myFuture.sink { value in
      print(value) // 10
  }
  ```

  > 2️⃣ **_예제_** ) `URLSession`이나 `Alamofire`등 `RestFul` 관련 `API`비동기 요청시에 해당 요청이 성공했는지, 실패 했는지에 대한 여부를 `return ` 해주는 예제 👇

  ```swift
  func isSuccessAPIRequest() -> AnyPublisher<Bool, Never> {
      Future<Bool, Never> { promise in
          urlRequestPublisher.sink(
              receiveCompletion: { completion in
                  switch completion {
                  case .finished:
                      print("finished")
                      promise(.success(true))
                  case .failure(let error): print(error.localizedDescription)
                      promise(.success(false))
                  }
              }, receiveValue: { value in
                  print(value)
              }
          )
      }
      .eraseToAnyPublisher()
  }

  // 사용
  isSuccessAPIRequest().sink {
      if $0 {
          // 성공 즉, true일 경우 Handling
      } else {
          // 실패 즉, false일 경우 Handling
      }
  }
  ```

### ☘️ Empty

- 값을 게시하지 않고 선택적으로 즉시 완료되는 `Publisher`
  > ✅ 즉, **_`Event` 없이 종료되는 `Publisher`_**
- 어떤 데이터도 발행하지 않는 `Publisher`로 주로 `Error`처리나, `Optional`값을 처리할 때 사용

  > **_예제_** 👇

  ```swift
  Empty<String, Never>().sink(
      receiveCompletion: {
          print($0) // finish
      },
      receiveValue: {
          print("receiveValue: \($0)") // 출력 안함
      }
  )
  ```

### ☘️ Fail

- 오류와 함께 종료되는 `Publisher`
  > **_예제_** 👇
  ```swift
  let failed = Fail<String, Error>(error: NSError(domain: "error", code: -1, userInfo: null))
  _ = failed.sink {
      print($0)
  } receiveValue: {
      print($0)
  }
  // 결과 👉 failure(Error Domain=error Code=-1 "(null)")
  ```

### ☘️ Deffered

- **_구독이 일어나기 전까지 대기상태로_** 있다가 구독이 일어 났을 때 `Publisher`가 결정이 됨
  > ✅ 즉, 구독(`Subscribers`)이 이루어질때 `publisher`가 만들어 짐
- `Closure` 안에는 지연 실행 할 `Publisher`를 반환함
  > **_예제_** 👇
  ```swift
  Deferred { Just(Void()) }.sink(receiveValue: { print("Diferred") })
  ```

### ☘️ Sequence

- 요소의 주어진 `Sequence`를 반환하는 `Publisher`
- `Publisher`가 `Sequence`에 있는 **_요소들을 하나 하나 제공_** 해주며, **_모든 요소들이 다 제공되었을 때 종료_**
  > **_예제_** 👇
  ```swift
  Publishers.Sequence<[Int], Never>(sequence: [1, 2, 3])
      .sink(receiveValue: {
          print("Sequence : \($0)")
      })
  ```

### ☘️ Record

- 입력과 완료를 기록해 후에 다른 `Subsciber`에서 반복될 수 있는 `Publisher`

  > **_예제_** 👇

  ```swift
  let record = Record<String, Error> { recoding in
      print("make recording")
      recording.receive("jack")
      recording.receive("tom")
      recording.receive(completion: .finished)
  }

  _ = record.sink {
      print($0)
  } receiveValue: {
      print($0)
  }

  _ = record.sink {
      print($0)
  } receiveValue: {
      print($0)
  }
  ```

  > **_결과_**
  >
  > - make recording
  > - jack
  > - tom
  > - finished
  > - jack
  > - tom
  > - finished
