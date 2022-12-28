---
title: iOS Swfit UIKit ViewController Lifecycle
author: Narvis2
date: 2022-12-02 10:20:00 +0900
categories: [Swift, UIKit]
tags: [iOS, UIKit, ViewController, Lifecycle]
---

안녕하세요. narvis2 입니다.  
이번시간에는 `iOS` `UIKit` `ViewController`의 `Lifecycle` 에 대하여 알아보도록 하겠습니다.

## 🍀 ViewController Lifecycle

---

![Desktop View](/assets/img/lifecycle/view_controller_life_cycle.png){: width="100%" }

### ☘️ Init

- `ViewController`의 초기화를 진행하면 내부의 객체들을 초기화하는 작업이 수행됨
  > - ⚠️ 아직 내부의 `View`들이 생성된 것은 아니기에 **_내부 `View` 요소에는 접근할 수 없음_**
- `init(coder:)` 👉 스토리보드를 기반으로 `ViewController`를 만들 경우 사용
- `init(nibName: bundle:)`: `nib` 파일을 기반으로 `ViewController`를 만들 경우 사용
  > **_✅ 참고_**
  >
  > - `nib` 👉 `NeXT Interface Builder (binary 기반)`, 인터페이스 빌더에서 생성한 객체들을 저장하는 파일, `UI`를 구성하는 객체들과 이들의 세부설정, 각 객체들간의 관계등을 포함합니다.

### ☘️ loadView

- `ViewController`가 사용자가 설정한 `View`를 자신의 최상위 `View`로 설정하는 과정
- `ViewController`가 무엇을 기반으로 호출되었는가(`Code`, `스토리보드`, `nib`)와 관계없이 호출됨
- 내부적으로 `instantiateWithOwner()`라는 메서드가 실행되면서 필요한 `subView`들을 `nib`으로부터 `unarchiving`하고 **_`subView` 객체들이 `ViewController`와 연결_** 되게 됩니다.
  > - 이 때 `subView` 객체들은 로드가 끝나자마자 `awakeFromNib()`을 호출합니다.
  > - **_`subView`들은 `ViewController`가 아니기 때문에 바로 내부 객체들에 접근할 수 있음._**
- **_`subView`들과 연결이 완료되면 `ViewController`는 `viewDidLoad()`를 호출_** 하여 `view` 들에 대한 `load`가 완전히 종료되었음을 알림

### ☘️ viewDidLoad

- 필요한 **_`View`의 정보들이 모두 메모리에 위치되면 호춛됨_**
- **_필요한 데이터를 갱신하는 코드를 작성_**
- `ViewController`의 생애중 단 한번만 호출됨

### ☘️ viewWillAppear

- `ViewController`의 **_`View`가 화면에 나타나기 직전에_** 호출됨
- **_화면이 나타날때마다 호출_** 되므로 **_다른 `View`에서 돌아올 때 수행하고 싶은 행위_** 들에 대해 처리하기 좋음

### ☘️ viewDidAppear

- `ViewController`의 **_`View`가 모두 화면에 나타나면_** 호출됨
- 주로 **_`UI`의 `Animation`을 실행_** 시키거나 **_비디오 및 소리를 재생_** 시키거나 **_`data`의 `update`를 수행_**
- **_`API` `data`를 받아와 화면을 `update`하는 로직을 위치시키기에 적당_**

### ☘️ viewWillLayoutSubviews

- `view`의 `bounds`를 정하는 단계
- `UIView`의 `layoutSubviews()` 메서드가 트리거되기 직전에 호출
- **_가로모드 혹은 세로모드가 되면서 `screen`의 방향이 변화될 때도 호출_**
- **_`View`의 `bounds`에 대한 재계산이 필요할때마다 호출_**

### ☘️ viewDidLayoutSubviews

- `layoutSubviews()` 메서드 호출 후에 호출
- `sub View`들의 `size`와 `position`, `constraint`들이 적용이 완료된 상태
- **_가로모드 혹은 세로모드가 되면서 `screen`의 방향이 변화될 때도 호출_**
- **_`View`의 `bounds`에 대한 재계산이 필요할때마다 호출_**

### ☘️ viewWillDisappear

- **_다른 `ViewController`로 화면이 전환되면서 원래 `ViewController`가 사라질 때_** 호출
- 일반적으로 이 함수를 `override` 해서 처리해야 할 작업은 거의 없음

### ☘️ viewDidDisappear

- `ViewController`가 화면에 **_사라지고나면 호출_**
- `Container` `View` `Controller`를 사용하다보면 여러 `ViewController`가 메모리에 유지
  > ⚠️ 즉, 화면에서 사라진 `ViewController`도 여전히 `notification` 등의 이벤트를 받을 수 있다는 것
- **_`notification`, `observing` 취소 및 디바이스 점검 등을 수행_**

### ☘️ deinit

- `ViewController` 객체가 **_메모리에 사라지기 전_** 호출
- **_주로 할당받은 자원중 `ARC`에 의해 해제가 되지 않는 자원을 해제하기 위해 `override`_**
- **_화면에서 사라진다고 해당 메서드가 호출되는 것은 아님_**

### ☘️ didReceiveMemoryWarning

- **_메모리가 부족해지면_** 호출
- 이를 관리해 **_필요없는 메모리를 해제_** 하는 작업을 할 수 있음
- `iOS`가 **_강제로 `Application`을 종료하는 것을 방지_** 하기 위해 필요
