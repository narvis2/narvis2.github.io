---
title: iOS SwiftUi ViewModel
author: Narvis2
date: 2022-11-21 16:02:00 +0900
categories: [SwiftUi]
tags: [ObservableObject, ObservedObject, Published, StateObject, ViewModel]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Swift` 에서 `ViewModel` 을 사용하여 `MVVM` 패턴을 사용하는 방법에 대하여 알아보도록 하겠습니다.

## 🍀 MVVM (Model - View - ViewModel)

---

- `Model` 👉 `Application` 에서 사용되는 데이터와 그 데이터를 처리하는 부분
- `View` 👉 사용자에게 보여지는 `UI` 부분
- `ViewModel` 👉 `View`를 표현하기 위해 만든 `View`를 위한 `Model`
  > `View`와 `Model` 을 연결 시켜 주는 곳
- `View`는 `ViewModel` 을 알고 있지만 `Model`을 알지 못함
- `View` 와 `Model`은 서로 알지 못함
- **_즉, 의존성 분리_**
  > - `View`와 `Model` 사이의 의존성이 없음 (`View`의 부담을 줄여줌)
  > - `ViewModel`은 `Data Binding`을 통해 `View`와 `Model`을 이어줌
  > - 각각의 부분은 **_독립적_** 이기 때문에 모듈화 하여 개발할 수 있음.

## 🍀 Swift with ViewModel

---

- `ObservableObject`를 이용하여 `Data Model`과 `View`를 쉽게 `Binding`할 수 있음

### ☘️ ObservableOjbect

- 해당 클래스의 인스턴스를 관찰하고 있다가 값이 변경되면 `View`를 `Update`한다.

  > **_예제_** 👇

  ```swift
  class MainviewModel: ObservableObject {

  }
  ```

### ☘️ Published

- `ObservableOject` 에서 속성을 선언할 때 사용하는 `PropertyWrapper`
- 해당 속성이 업데이트 될 때마다 `View`를 `Update`, `$` `operator`를 붙여 게시자에 엑세스 가능
- Android 의 `LiveData`와 유사

  > **_예제_** 👇

  ```swift
  class MainviewModel: ObservableObject {
    @Published var youngjun = Person(name: "영준", birthday: Date())

    var name: String {
        youngjun.name
    }

    var age: String {
        return "29"
    }

    func changeName(_ name: String) {
        youngjun.name = name
    }
  }
  ```

### ☘️ ObservedObject

- `ObservableObject`를 `구독`하고 **_값이 `Update`될 때 마다 `View`를 갱신하는 `PropertyWrapper`_**
- `View`를 만료시키고 새로 그림

  > ⚠️ `View`가 새로 그려질 때 마다 `인스턴스`가 새로 초기회됨

  > **_예제_** 👇

  ```swift
  struct ContentView: View {
    @ObservedObject var viewModel = MainViewModel()
  }
  ```

### ☘️ StateObject

- **_단 한 번 인스턴스가 생성_**
- `View`를 처음부터 새로 그리지 않고, `ObservableObject`에서의 데이터가 변할 때, 그 **_`ObservableObject`의 데이터가 들어간 부분만 `View`를 다시 그 림_**
- `View`가 얼마나 다시 그려지든 상관없이 **_별개의 객체로 관리_**
  > **_예제_** 👇
  ```swift
  struct ContentView: View {
    @StateObject var viewModel = MainViewModel()
  }
  ```

### ☘️ SharedViewModel 구현

- ✅ 추천
  - `Observable Object`를 **_처음 초기화_** 할 때는 `StateObject` 를 사용하고, 이미 **_객체화된 것을 넘겨 받을 때는 `ObservedObject`를 사용_**
- **_`StateObject` 와 `ObservedObject`를 사용하여 `SharedViewModel`을 사용가능_**
- 상위 `View`에서 객체로 만들어서 따로 저장해두고, 하위 `View`도 이 `Observable Object`의 변화를 감지하고, 같은 정보에 **_접근_** 할 수 있도록 할 수 있음

  > **_예제_** 👇

  ```swift
  struct UpperView: View {
    @StateObject var viewModel: MainViewModel = MainViewModel()
    var body: some View {
        LowerView(viewModel: viewModel)
    }
  }

  struct LowerView: View {
    @ObservedObject var viewModel: ViewModel
    var body: some View {
        Text("Hello")
    }
  }
  ```
