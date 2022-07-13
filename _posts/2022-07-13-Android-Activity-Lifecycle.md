---
title: Android Activity Lifecycle 생명주기 정리
author: Narvis2
date: 2022-07-13 09:28:00 +0900
categories: [Android]
tags: [android, activity, lifecycle]
---
## Android Activity Lifecycle 동작 순서
---
[구글 공식 홈페이지](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko)에 나와있는 이미지를 참고하시면 이해가 쉽습니다.  
해당 이미지를 참고하여 Activity Lifecycle 의 순서를 보면 Activity 생성 시 onCreate() -> onStart() -> onResume() 의 순서로 콜백이 실행되며 Activity의 구성요소 혹은 일부가 사라지면 onPause() -> onStop() -> onDestory() 순으로 콜백이 실행되며 onDestory() 이후에 Activity가 종료됩니다.  
> **주의!!** <u>Activity가 중지(onStop())상태에서 다시 화면에 보여지면 Lifecycycle은 어떻게 될까요??</u>  
onRestart() -> onStart() -> onResume() -> ActivityRunning 순으로 실행됩니다.  
Activity가 onDestory()를 타지않고 onStop()에서 다시 보여진다면 onCreate()가 호출되지 않고 onRestart()가 호출된다는 점 꼭 기억해주세요!!  

![Desktop View](/assets/img/lifecycle/activity_lifecycle.png){: width="100%" }

## Activity Lifecycle Callback Method 역할
---
**각 Callback Method 의 역할에 관하여 살펴보겠습니다.**
### 1. onCreate()
- Activity가 실행되면 제일 처음 호출되는 메서드입니다.
- View 관련 처리는 해당 메서드에서 진행합니다. (click Listenr, data binding 등등..)
- onCreate 에는 saveInstanceState 즉 bundle를 매개변수로 가지고 있습니다. 해당 프로퍼티는 처음 생성된 경우 null을 담고있고, onSaveInstanceState() 함수를 통해 인스턴스 상태를 저장한 경우 saveInstanceState를 통해 상태값을 받아 처리할 수 있습니다.
``` kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding 

    override fun onCreate(saveInstanceState: Bundle?) {
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this

        // todo View 세팅 (Adapter 세팅, ClickListener 연결 등등..)
    }
}
```  
### 2. onStart()
- Activity 가 onCreate()를 호출한 뒤 Activity가 **_'STARTED'_** 상태에 진입하게 되면  
즉, Activity가 화면에 보이기 바로 직전에 호출됩니다.
- 이 메서드가 호출되면 Activity가 사용자와 상호작용할 수 있도록 준비합니다.

### 3. onResume()
- Activity가 onStart()를 호출한 뒤 Activity가 **_'RESUME'_** 상태에 진입하게 되면 호출됩니다.
- 사용자와 상호작용을 할 수 있는 상태가 됩니다. 
> **주의!!** 전화가 오거나, 화면을 슬립하면 Activity의 lifecycle은 onPause()로 넘어갑니다.  
이때 다시 해당 Activity로 돌아오면 onCreate()가 호출되지 않고 onResume()이 다시 한번 호출됩니다.  
이런 경우 Activity가 다시 보여질때마다 필요한 코드를 입력하시면 됩니다.

### 4. onPause()
- 사용자가 잠시 Activity를 떠났을 때 즉, 또 다른 Activity에 Focus를 뒀을 때 호출됩니다.  
- 전화가 오는 경우, 멀티 윈도우 상 다른 앱에 Focus를 두는 경우 호출
- 이 메서드가 Return 하기 전에는 다음 Activity가 시작될 수 없습니다. 따라서 이 작업은 매우 빨리 수행된 후 return 되어야 합니다.
- Activity가 onPause() 상태에 들어가면 System은 Activity를 강제 종료할 수 있습니다.
- Activity가 다시 시작되거나, 사용자에게 완전히 보여지지 않는 이상 Activity는 **_'PAUSED'_** 상태에 머무르게 된다. 
- onPause() 상태에서 다시 Activity가 보여지면 onCreate()를 타지않고 onRestart()를 호출합니다. 
> **주의!!** onPause()는 아주 잠깐 실행됩니다. 따라서 사용자 데이터 저장, 네트워크 호출, DB Transaction 등 시간이 오래걸리는 코드를 실행해서는 안 됩니다.

### 5. onStop()
- Activity가 사용자에게 더 이상 보이지 않을 때 Activity는 **_'STOPPED'_** 상태에 진입하고 onStop()을 호출합니다.
- Activity가 소멸하거나 다른 Activity가 화면을 가릴 때 호출됩니다.
- 이 메서드에서는 필요하지 않는 리소스를 해제해야하고(애니메이션 일시중지 등등..), 사용자의 데이터 저장, 네트워크 호출, DB Transaction 등 시간이 오래걸리는 작업 또한 가능합니다. 
- Activity가 onStop() 상태에 들어가면 System은 Activity를 강제 종료할 수 있습니다.
> **주의!** 만약 onStop() 상태에서 Activity가 다시 보여진다면 onCreate()는 호출되지 않고 onRestart() -> onStart() -> onResume() 순으로 호출됩니다.  
예를 들어 화면을 잠그면 Activity는 onResume() 상태에서 onPause() -> onStop()을 호출하고 잠금을 해제하면 onRestart() -> onStart() -> onResume() 순으로 호출됩니다.

### 6. onDestory()
- Activity가 완전히 소멸되기 전에 호출됩니다. 이 메서드는 Activity가 받는 마지막 메서드입니다.
- Activity가 앱에 의해 종료되거나(finish 메서드 호출), 화면 구성요소 변경(기기 회전 등..), System이 강제로 종료를 시키는 경우 호출될 수 있습니다.
- Activity가 onDestory() 상태에 들어가면 System 은 Activity를 강제로 종료할 수 있습니다.
> **주의!** 만약 onDestory()가 호출되기 까지 해제되지 않는 리소스가 있다면, 모두 여기서 해제해줘야 합니다. 그렇지 않으면 Memory Leak 즉, 메모리 누수의 위험이 있습니다.

## 구성요소 변경시(화면 회전 등..) Activity 의 생명주기는??
--- 
- 화면 회전 등 구성요소가 변경되면 Activity는 다음 순서로 lifecycle Callback Method를 호출합니다.  
onPause() -> onStop() -> onDestory() -> onStart() -> onResumse()  
즉, 현재 Activity를 Destory 하고 새로 생성하기 때문에 기존 데이터를 유지하기 위해서는 onSaveInstanceState()에서 Bundle에 데이터를 저장하도록 해야합니다.  
이러한 점 때문에 요즘 Android 개발에서는 AAC ViewModel 즉, Android Architecture Components ViewModel 을 사용하여 구성요소 변경시에도 데이터가 살아있도록 합니다. AAC ViewModel에 대한 개념은 다음에 따로 작성하겠습니다.

---
## 'A' Activity 에서 'B' Activity를 실행하는 경우 Activity의 생명주기는??
---
- A -> B Activity를 실행할 경우 Activity의 생명주기는 다음과 같습니다.  
'A' onPause() -> 'B' onCreate() -> 'B' onStart() -> 'B' onResume() -> 'A' onStop() -> 'B'는 사용자에게 보여짐.
- 위의 상태에서 B Activity를 종료 즉, finish() 함수를 통해 종료하면 생명주기는 어떻게 될까요??  
  'B' onPause() -> 'A' onRestart() -> 'A' onStart() -> 'A' onResume() -> 'B' onStop() -> 'B' onDestroy() -> 'A'는 사용자에게 보여짐.

## 마지며..
처음 작성하는 포스팅이다보니 다소 서툴렀던 점 양해부탁드립니다.  
Activity Lifecycle 포스팅은 쓸데없는 설명은 최대한 자제하며 실용적인 의미의 내용만 담을려 노력하였습니다.  
긴 글 읽어주셔서 감사드리고 여러분께 꼭 도움이 되셨으면 합니다.  
다음 포스팅은 Fragment 의 Lifecycle 에 대하여 알아보고 정리하는 시간을 가지도록 하겠습니다.  