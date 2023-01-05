---
title: iOS SwiftUi ViewModel
author: Narvis2
date: 2022-11-21 16:02:00 +0900
categories: [Swift, SwiftUi]
tags: [ObservableObject, ObservedObject, Published, StateObject, ViewModel]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Swift` 에서 `ViewModel` 을 사용하여 `MVVM` 패턴을 사용하는 방법에 대하여 알아보도록 하겠습니다.

## 🍀 MVVM (Model - View - ViewModel)

---

- `Model` 👉 `Application` 에서 사용되는 데이터와 그 데이터를 처리하는 부분
  - 서버에서 들어오는 데이터, 디바이스에 저장되는 Local 데이터
- `View` 👉 사용자에게 보여지는 `UI` 부분
  - `ViewModel`의 데이터가 변경된 것을 감지해 `UI`를 `Update`함
- `ViewModel` 👉 `View`를 표현하기 위해 만든 `View`를 위한 `Model`
  - `View`와 `Model` 을 연결 시켜 주는 곳
  - 데이터의 변경사항을 알려주는 `LiveData`를 가지고 있음
  - `Model` 이 변경된 것을 감지하여 `UI`를 위한 데이터를 변경해야함
- `View`는 `Model`을 모르지만 `ViewModel`을 알고 있음
- `ViewModel`은 `View`를 모르지만 `Model`을 알고 있음
- `View`를 통해 들어온 사용자 인터렉션은 `ViewModel`에게 전달되어 특정 로직이 실행됨
- **_✅ 즉, 의존성 분리_**
  - `View`와 `Model` 사이의 의존성이 없음 (`View`의 부담을 줄여줌)
  - `ViewModel`은 `Data Binding`을 통해 `View`와 `Model`을 이어줌
  - 각각의 부분은 **_독립적_** 이기 때문에 모듈화 하여 개발할 수 있음.
- **_✅ 장점_**
  - `View` 와 `data` 가 완전 분리된다.
  - `View`가 `data`관리를 할 필요가 없으므로 `UI update`에만 집중할 수 있음
  - 각 `View` 간에 `data` 공유가 훨씬 쉬워진다.
  - 유지보수의 장점 👉 의존성이 분리되므로, `View`의 코드를 변경할 때 다른 부분의 코드를 변경할 필요가 없음
- **_✅ 동작 순서_**
  - 1️⃣ 사용자의 `action`이 `View`를 통해 들어옴
  - 2️⃣ `command` 패턴을 사용하여 `ViewModel`에 `Action`을 전달함
  - 3️⃣ `ViewModel`이 `Model`에서 `data`를 요청하고, `Model`은 `ViewModel`에서 요청받은 데이터를 `ViewModel`에 전달함
  - 4️⃣ `ViewModel`은 응답받은 데이터를 가공, 저장함
  - 5️⃣ `View`는 `ViewModel`과의 `Data Binding`을 이용해 화면을 갱신함 (`Observer 패턴`)

## 🍀 Swift with ViewModel

---

- `ObservableObject`를 이용하여 `Data Model`과 `View`를 쉽게 `Binding`할 수 있음
- `ObservableObject`로 `ViewModel`을 설정하고, `View` 단에서는 `ViewModel`에 `@ObservedObject`를 붙여줌으로서 `ViewModel`의 **_변화를 관찰_** 할 수 있고, 그 변화에 반응할 수 있음

### ☘️ @ObservableOjbect

- 해당 클래스의 인스턴스를 관찰하고 있다가 값이 변경되면 `View`를 `Update`한다.

  > **_예제_** 👇

  ```swift
  class MainviewModel: ObservableObject {

  }
  ```

### ☘️ @Published

- `ObservableOject` 에서 **_속성을 선언할 때 사용_** 하는 `PropertyWrapper`
- 해당 속성이 업데이트 될 때마다 `View`를 `Update`, `$` `operator`를 붙여 게시자에 엑세스 가능
- `@Published`가 붙어있는 변수가 **_변경_** 되면 이를 **_지켜보고 있는 `View`에게 `ViewModel`이 변경되었음을 알려주고, `View`는 새로운 객체를 바탕으로 `View`를 `Refresh` 한다._**
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

### ☘️ @ObservedObject

- `ObservableObject`를 `구독`하고 **_값이 `Update`될 때 마다 `View`를 갱신하는 `PropertyWrapper`_**
- `View`를 만료시키고 새로 그림

  > ⚠️ `View`가 새로 그려질 때 마다 `인스턴스`가 새로 초기회됨

  > **_예제_** 👇

  ```swift
  struct ContentView: View {
    @ObservedObject var viewModel = MainViewModel()
  }
  ```

### ☘️ @StateObject

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

### ☘️ @State

- `DataBinding`을 위해 사용
  > - `$` 키워드를 붙혀 `DataBinding`
  > - `$`가 붙으면 **_값을 수정가능한_** `Binding`타입 참조
- `@State`는 `View` 외부로는 사용할 수가 없고 **_`private` 형태로 내부에서만 사용_**

  > **_예제_** 👇

  ```swift
  struct ContentView: View {
    @State var isToggleOn: Bool = true

    var body: some View {
      VStrck {
        Toggle(isOn: $isToggleOn) {
          Text("글자를 가립니다.")
        }.padding()

        if isToggleOn {
          Text("그으으을자!")
        }
      }
    }
  }
  ```

### ☘️ @Binding

- **_2개의 `View`가 동시에 하나의 `State`를 참조_** 해야하는 경우 사용

  > **_예제_** 👇

  ```swift
  struct Toggole: View {
    @Binding var isOn: Bool

    var body: some View {
      // TODO..
    }
  }
  ```
