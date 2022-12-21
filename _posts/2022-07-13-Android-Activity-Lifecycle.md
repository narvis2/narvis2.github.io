---
title: Android Activity Lifecycle 생명주기 정리
author: Narvis2
date: 2022-07-13 09:28:00 +0900
categories: [Android, Lifecycle]
tags: [android, activity, lifecycle]
---

## Android Activity Lifecycle 동작 순서

---

[구글 공식 홈페이지](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)에 나와있는 이미지를 참고하시면 이해가 쉽습니다.  
해당 이미지를 참고하여 `Activity` `Lifecycle` 의 순서를 보면 **_`Activity` 생성 시 `onCreate()` -> `onStart()` -> `onResume()` 의 순서로 콜백이 실행_** 되며 **_`Activity`의 구성요소 혹은 일부가 사라지면 `onPause()` -> `onStop()` -> `onDestory()` 순으로 콜백이 실행되며_** `onDestory()` 이후에 `Activity`가 종료됩니다.

> **_참고_** 👉 `Activity`가 중지(`onStop()`)상태에서 **_다시 화면에 보여지면_** `Lifecycycle`은 어떻게 될까요??
>
> - ✅ `onRestart()` -> `onStart()` -> `onResume()` -> `ActivityRunning` 순으로 실행됩니다.
> - `Activity`가 `onDestory()`를 타지않고 **_`onStop()`에서 다시 보여진다면 `onCreate()`가 호출되지 않고 `onRestart()`가 호출_** 된다는 점 꼭 기억해주세요!!

![Desktop View](/assets/img/lifecycle/activity_lifecycle.png){: width="100%" }

## 🍀 Activity Lifecycle Callback Method 역할

---

**각 `Callback Method` 의 역할에 관하여 살펴보겠습니다.**

### ☘️ 1. onCreate()

- `Activity`가 실행되면 제일 처음 호출되는 메서드입니다.
- `View` 관련 처리는 해당 메서드에서 진행합니다. (`click Listenr`, `data binding` 등등..)
- `onCreate` 에는 `saveInstanceState` 즉 `bundle`를 매개변수로 가지고 있습니다. 해당 프로퍼티는 처음 생성된 경우 `null`을 담고있고, `onSaveInstanceState()` 함수를 통해 인스턴스 상태를 저장한 경우 `saveInstanceState`를 통해 상태값을 받아 처리할 수 있습니다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(saveInstanceState: Bundle?) {
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this

        // todo View 세팅 (Adapter 세팅, ClickListener 연결 등등..)
    }
}
```

### ☘️ 2. onStart()

- `Activity` 가 `onCreate()`를 호출한 뒤 `Activity`가 **_'STARTED'_** 상태에 진입하게 되면  
  즉, `Activity`가 화면에 보이기 바로 직전에 호출됩니다.
- 이 메서드가 호출되면 `Activity`가 사용자와 상호작용할 수 있도록 준비합니다.

### ☘️ 3. onResume()

- `Activity`가 `onStart()`를 호출한 뒤 `Activity`가 **_'RESUME'_** 상태에 진입하게 되면 호출됩니다.
- 사용자와 상호작용을 할 수 있는 상태가 됩니다.
  > **_⚠️주의⚠️_**
  >
  > - 전화가 오거나, 화면을 슬립하면 `Activity`의 `lifecycle`은 `onPause()`로 넘어갑니다.  
  >   이때 다시 해당 `Activity`로 돌아오면 `onCreate()`가 호출되지 않고 `onResume()`이 다시 한번 호출됩니다.  
  >   이런 경우 `Activity`가 다시 보여질때마다 필요한 코드를 입력하시면 됩니다.

### ☘️ 4. onPause()

- 사용자가 잠시 `Activity`를 떠났을 때 즉, 또 다른 `Activity`에 `Focus`를 뒀을 때 호출됩니다.
- ✅ **_전화가 오는 경우, 멀티 윈도우 상 다른 앱에 `Focus`를 두는 경우_** 호출
- 이 메서드가 `Return` 하기 전에는 다음 `Activity`가 시작될 수 없습니다. 따라서 **_이 작업은 매우 빨리 수행된 후 `return` 되어야_** 합니다.
- `Activity`가 `onPause()` 상태에 들어가면 `System`은 `Activity`를 강제 종료할 수 있습니다.
- `Activity`가 다시 시작되거나, 사용자에게 완전히 보여지지 않는 이상 `Activity`는 **_'PAUSED'_** 상태에 머무르게 된다.
- ✅ `onPause()` 상태에서 **_다시 `Activity`가 보여지면 `onCreate()`를 타지않고 `onRestart()`를 호출_** 합니다.
  > **_⚠️주의⚠️_**
  >
  > - `onPause()`는 아주 잠깐 실행됩니다. 따라서 **_사용자 데이터 저장, 네트워크 호출, `DB Transaction`_** 등 시간이 오래걸리는 코드를 실행해서는 안 됩니다.

### ☘️ 5. onStop()

- `Activity`가 사용자에게 더 이상 보이지 않을 때 `Activity`는 **_'STOPPED'_** 상태에 진입하고 `onStop()`을 호출합니다.
- ✅ `Activity`가 소멸하거나 다른 `Activity`가 화면을 가릴 때 호출됩니다.
- 이 메서드에서는 **_필요하지 않는 리소스를 해제해야_** 하고(애니메이션 일시중지 등등..), **_사용자의 데이터 저장, 네트워크 호출, `DB Transaction` 등 시간이 오래걸리는 작업 또한 가능_** 합니다.
- Activity가 onStop() 상태에 들어가면 System은 Activity를 강제 종료할 수 있습니다.
  > **_⚠️주의⚠️_**
  >
  > - 만약 **_`onStop()` 상태에서 `Activity`가 다시 보여진다면_** `onCreate()`는 호출되지 않고 **_`onRestart()` -> `onStart()` -> `onResume()`_** 순으로 호출됩니다.
  > - 예를 들어 화면을 잠그면 `Activity`는 `onResume()` 상태에서 `onPause()` -> `onStop()`을 호출하고 **_잠금을 해제하면 `onRestart()` -> `onStart()` -> `onResume()`_** 순으로 호출됩니다.

### ☘️ 6. onDestory()

- `Activity`가 완전히 소멸되기 전에 호출됩니다. 이 메서드는 `Activity`가 받는 **_마지막 메서드_** 입니다.
- `Activity`가 앱에 의해 종료되거나(`finish` 메서드 호출), **_화면 구성요소 변경(기기 회전 등..)_**, `System`이 강제로 종료를 시키는 경우 호출될 수 있습니다.
- `Activity`가 `onDestory()` 상태에 들어가면 `System` 은 `Activity`를 강제로 종료할 수 있습니다.
  > **_⚠️주의⚠️_**
  >
  > - 만약 `onDestory()`가 호출되기까지 해제되지 않는 리소스가 있다면, **_모두 여기서 해제해줘야_** 합니다. 그렇지 않으면 `Memory Leak` 즉, **_메모리 누수의 위험이_** 있습니다.

## 🍀 번외

---

### ☘️ 구성요소 변경시(화면 회전 등..) Activity 의 생명주기는??

- 화면 회전 등 **_구성요소가 변경되면_** `Activity`는 다음 순서로 `lifecycle` `Callback` `Method`를 호출합니다. 👇
  - **_`onPause()` -> `onStop()` -> `onDestory()` -> `onCreate()` -> `onStart()` -> `onResumse()`_**
  - 🛠 즉, **_현재 `Activity`를 `Destory` 하고 새로 생성_** 하기 때문에 **_기존 데이터를 유지하기 위해서는 `onSaveInstanceState()`에서 `Bundle`에 데이터를 저장하도록_** 해야합니다.  
    이러한 점 때문에 요즘 `Android` 개발에서는 `AAC ViewModel` 즉, `Android Architecture Components ViewModel` 을 사용하여 **_구성요소 변경시에도 데이터가 살아있도록_** 합니다. `AAC ViewModel`에 대한 개념은 다음에 따로 작성하겠습니다.

### ☘️ 'A' Activity 에서 'B' Activity를 실행하는 경우 Activity의 생명주기는??

- `A` -> `B` `Activity`를 실행할 경우 `Activity`의 생명주기는 다음과 같습니다. 👇
  > `'A' onPause()` -> `'B' onCreate()` -> `'B' onStart()` -> `'B' onResume()` -> `'A' onStop() -> 'B'`는 사용자에게 보여짐.
- 위의 상태에서 `B Activity`를 종료 즉, `finish()` 함수를 통해 종료하면 생명주기는 어떻게 될까요?? 👇
  > `'B' onPause()` -> `'A' onRestart()` -> `'A' onStart()` -> `'A' onResume()` -> `'B' onStop()` -> `'B' onDestroy()` -> `'A'`는 사용자에게 보여짐.

## 🍀 마치며..

`Activity` `Lifecycle` 포스팅은 쓸데없는 설명은 최대한 자제하며 실용적인 의미의 내용만 담을려 노력하였습니다.  
긴 글 읽어주셔서 감사드리고 여러분께 꼭 도움이 되셨으면 합니다.  
[다음 포스팅](https://narvis2.github.io/posts/Android-Fragment-Lifecycle/)은 `Fragment` 의 `Lifecycle` 에 대하여 알아보고 정리하는 시간을 가지도록 하겠습니다.
