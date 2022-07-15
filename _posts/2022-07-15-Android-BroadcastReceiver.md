---
title: Android 4대 컴포넌트 중 BroadcastReceiver 에 대하여 
author: Narvis2
date: 2022-07-15 8:34:00 +0900
categories: [Android]
tags: [broadcastReceiver, android]
---

안녕하세요. Narvis2입니다.  
지난번에 Android4 대 Component 중 하나인 [Service에 대하여](https://narvis2.github.io/posts/Android-BroadcastReceiever/) 알아보았습니다.  
이번 시간에는 Android 4대 Component(Activity, ContentProvider, BroadcastReceiver, Service) 중 하나인 BroadcastReceiver에 대하여 알아보겠습니다.  
BroadcastReceiver 는 이름에서 알 수 있듯이 **_"방송"_**이라고 생각하시면 됩니다.  
예를 들어 전화, 문자 등 어떤 행위가 왔다는 알림을 받고 처리할 수 있습니다. 밑에서 더욱 자세한 내용을 알아보도록 하겠습니다.

## BroadcastReceiver
---
- 전화, 문자 등 어떤 행위가 왔다는 알림을 받고 이것을 알려주는 기능입니다.  
  예를 들어 Android System은 System **"부팅"**, **"기기 충전"** 시작과 같은 다양한 System Event가 발생할 때 Broadcast 를 전송합니다.  
  이 Broadcast는 Intent를 통해 발송하게 되고, 이렇게 발송된 Broadcast는 BroadcastReceiver 객체가 수신하게 됩니다.
  
- BroadcastReceiver는 Broadcast가 발생하면 Intent-Filter를 통해 원하는 Intent만 수신할 수 있습니다.
> **참고** : AndroidManifest에 Receiver를 등록하고 Intent-Filter를 통해 원하는 Intent에 대해서만 알림을 받을 수 있습니다. 즉, 수많은 Broadcast 중에서 어떤 것을 수신할 것인지 등록하는 과정이라 할 수 있습니다.
- BroadcastReceiver는 복잡성이 낮고 Process간의 가장 쉬운 통신 방법입니다.  
- One to All 통신으로 모든 수신자에게 동시에 메시지를 전송합니다.(Android OS 기반 응용 프로그램 구성 요소간의 통신) 따라서 보안 위협이 발생 가능합니다. 그 이유는 민감한 데이터는 방송되어서는 안되기 때문입니다.
> **참고** : Process 간 통신을 위해 BroadcastReceiver를 사용하면 이러한 보안 위협이 발생할 수 있습니다. 이런 경우 BroadcastReceiver를 대신하여 AIDL을 사용합니다.
- 비동기 통신입니다.

## onReceive()
---
- 간단하게 "내가 원하는 Broadcast를 고르는 곳" 이라고 생각하시면 됩니다. 즉, 원하는 Intent.action에 대한 처리를 담당하는 곳입니다.
- 항상 Main Thread 에서 실행합니다.
- Intent를 통해 Data를 보낼 때 Data 크기를 몇 KB로 제한 하도록 주의해야 합니다.
> **주의!!** : 너무 많은 데이터를 보낼 경우 TransactionTooLargeException 이 발생할 수 있습니다.  
Intent는 최대 500Kb 크기의 Data를 전송할 수 있습니다.

## 대표적인 Broadcast 종류
---
- ACTION_BOOT_COMPLERED : System 부팅이 끝났을 때(RECEIVE_BOOT_COMPLETED 권한 등록 필요
- ACTION_CAMERA_BUTTON : 카메라 버튼을 눌렀을 때 
- ACTION_DATE_CHANGED, ACTION_TIME_CHANGED : 핸드폰의 날짜, 시간이 수동으로 변했을 때 즉, 설정에서 수정했을 때 
- ACTION_SCREEN_OFF, ACTION_SCREEN_ON : 화면 on, off
- ACTION_AIRPLANE_MODE_CHANGED : 비행기 모드
- ACTION_BATTERY_CHANGED, ACTION_BATTERY_LOW, ACTION_BATTERY_OKAY : 베터리 상태 변화
- ACTION_PACKAGE_ADDED, ACTION_PACKAGE_CHANGED, ACTION_PACKAGE_DATA_CLEARED, ACTION_PACKAGE_INSTALL, ACTION_PACKAGE_REMOVED, ACTION_PACKAGE_REPLACED, ACTION_PACKAGE_RESTARTED : 어플 설치 / 제거
- ACTION_POWER_CONNECTED, ACTION_POWER_DISCONNECTED : 충전 관련
- ACTION_REBOOT, ACTION_SHUTDOWN : 재부팅 / 종료
- ACTION_TIME_TICK : 매 분마다 수신
- android.provider.Telephony.SMS_RECEIVED : SMS 수신 (RECEIVE_SMS 권한 필요)

## BroadcastReceiver의 종류
---
- Global Broadcast : 일반적으로 이야기하는 Broadcast 이며, Process 간의 경계를 무시하고 Android System 상에 등록된 모든 Receiver 들에게 전달됩니다.
  
- Local Broadcast : 현재 Process 안에만 유효한 Broadcast 입니다. 주로 Process 간의 통신에 사용됩니다.  
예를 들어 bindService를 통하여 Service -> Activity/Fragment 간의 통신에 LocalBroadcastReceiver를 사용할 수 있습니다.

## LocalBroadcastManager를 이용한 Process 통신에 대해 간단한 예제로 알아보겠습니다.
---
### 1. BroadcastReceiver 등록
``` kotlin
private fun registerLocalBroadcastManager() {
    LocalBroadcastManager.getInstance(context).registerReceiver(broadcastReceiver, createIntentFilter())
}
```
Intent-Filter
``` kotlin
private fun createIntentFilter(): IntentFilter {
    // Action Type을 지정해 줍니다. 이 Type을 바탕으로 onReceive() 에서 action을 분류하여 처리합니다.
    return IntentFilter().apply { addAction("Test") }
}
```

### 2. BroadcastReceiver 해제
``` kotlin
private fun unRegisterLocalBroadcastManager() {
    LocalBroadcastManager.getInstance(context).unregisterReceiver(broadcastReceiver)
}
```

### 3. BroadcastReceiver에 Intent 전달
``` kotlin
private fun sendTestBroadcast() {
    val message = "test 입니다."
    val intent = Intent("Test")
    intent.putExtra("TestData", message)
    LocalBroadcastManager.getInstance(context).sendBroadcast(intent)
}
```

### 4. Broadcast 를 수신하는 쪽 onReceive()
Activity/Fragment 의 상태 공유를 위해 AndroidViewModel() 을 사용하여 AndroidViewModel 안에 해당 코드 작성
``` kotlin
// 객체 지향을 지켜 내부에서는 값을 넣을 수 있고, 외부에서는 값을 못넣고 가져올수만 있도록 합니다.
private val _testEvent = MutableLiveData<String>()
val testEvent: LiveData<String>
    get() = _testEvent

// BroadcastReceiver 생성
private val broadcastReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        // intent가 null 이 아닐때만 밑에 로직이 실행됩니다. null 일 경우 Timber를 이용하여 Error Log를 남깁니다.
        intent?.let { it: Intent
            // intent 내부의 data가 null이 아닐때만 밑의 로직을 실행합니다. null 일 경우 Timber를 이용하여 Error Log를 남깁니다.
            it.getStringExtra("TestData")?.let { data: String ->
                when (it.action) {
                    "Test" -> {
                        // TODO : 값을 StateFlow, LiveData에 넣고 적절한 곳에서 Observing 혹은 직접 처리
                        _testEvent.postValue(data)
                    }
                }
            } ?: Timber.e("onReceive data is null.")
        } ?: Timber.e("onReceive intent is null.")
    }
}
```

### 5. Activity, Fragment 에서 해당 Event를 Observing
1. Activity에서 Observing
``` kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding 
    // activityX 를 이용한 ViewModel 생성
    // 해당 by viewModels()를 사용하기 위해서는 build.gradle(app)에 androidX 의존성을 추가하셔야 합니다.
    private mainViewModel: MainViewModel by viewModels() 

    override fun onCreate(saveInstanceState: Bundle?) {
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this

        mainViewModel.testEvent.observe(this) { it: String
            Toast.makeText(this, it, Toast.LENGTH_SHORT).show()
        }
    }
}
```

2. Fragment에서 Observing
``` kotlin
class HomeFragment : Fragment() {
    private lateinit var binding: FragmentHomeBinding
    // fragmentX 를 이용한 ViewModel 생성 (Activity에 있는 MainViewModel 공유)
    // 해당 by activityViewModels()를 사용하기 위해서는 build.gradle(app)에 fragmentX 의존성을 추가하셔야 합니다.
    private mainViewModel: MainViewModel by activityViewModels()
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.lifecycleOwner = viewLifecycleOwner

        mainViewModel.testEvent.observe(viewLifecycleOwner) { it: String
            Toast.makeText(requireContext(), it, Toast.LENGTH_SHORT).show()
        }
    }
}
```

### 6. LocalBroadcastManager 관련 전체 코드
``` kotlin
class MainViewModel(application: Application) : AndroidViewModel(application) {
    private val context = application

    // StateFlow, SharedFlow 를 사용하셔도 무방합니다. 
    // 저는 요즘 StateFlow, SharedFlow, Coroutine Channel 을 사용하지만 여기서는 간단한 예를 위해 LiveData를 사용하였습니다.
    private val _testEvent = MutableLiveData<String>()
    val testEvent: LiveData<String>
        get() = _testEvent

    // LocalBroadcastManager 등록
    private fun registerLocalBroadcastManager() {
        LocalBroadcastManager.getInstance(context.applicationContext).registerReceiver(broadcastReceiver, createIntentFilter())
    }

    // Intent-Filter 생성
    // Kotlin Type 추론에 의하여 반환값을 적어주지 않으셔도 되지만 이해를 돕기위해 반환값을 명시적으로 적었습니다.
    private fun createIntentFilter(): IntentFilter = IntentFilter().apply { 
        // 여기서는 action Type 하나 뿐이라 하드 코딩을 하였지만,
        // type이 여러개일 경우 enum class를 활용하는 것이 좋습니다.
        addAction("Test") 
    }

    // LocalBroadcastManager 해제
    private fun unRegisterLocalBroadcastManager() {
        LocalBroadcastManager.getInstance(context.applicationContext).unregisterReceiver(broadcastReceiver)
    }

    // BroadcastReceiver에 Intent 전달하는 함수
    private fun sendTestBroadcast() {
        // 여기서는 간단한 예를 들기 위해 String 값을 넣었으나 
        // 직렬화를 통해 객체를 String으로 바꿔서 넘기고 onReceive() 에서 역직렬화를 통해 String을 객체로 바꿔 사용할 수 있습니다. 
        val message = "test 입니다."
        val intent = Intent("Test")
        intent.putExtra("TestData", message)
        LocalBroadcastManager.getInstance(context.applicationContext).sendBroadcast(intent)
    }

    // BroadcastReceiver 생성
    private val broadcastReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context?, intent: Intent?) {
            // intent가 null 이 아닐때만 밑에 로직이 실행됩니다. null 일 경우 Timber를 이용하여 Error Log를 남깁니다.
            intent?.let { it: Intent
                // intent 내부의 data가 null이 아닐때만 밑의 로직을 실행합니다. null 일 경우 Timber를 이용하여 Error Log를 남깁니다.
                it.getStringExtra("TestData")?.let { data: String ->
                    when (it.action) {
                        "Test" -> {
                            // TODO : 값을 StateFlow, LiveData에 넣고 적절한 곳에서 Observing 혹은 직접 처리
                            _testEvent.postValue(data)
                        }
                    }
                } ?: Timber.e("onReceive data is null.")
            } ?: Timber.e("onReceive intent is null.")
        }
    }
}
``` 

## 마치며
---
이번 사간에는 BroadcastReceiver에 대하여 알아보았습니다.  
BroadcastReciver는 Android 4대 컴포넌트 중 하나이며 핸드폰의 각가지 Event와 Process간의 통신에도 사용됩니다.  
핸드폰의 각가지 Event를 수신할 때는 Global BroadcastReceiver 즉, 일반 BroadcastReceiver를 사용하고 Process간 통신에 사용할 경우 LocalBroadcastManager를 사용합니다.  
저는 Service -> Fragment/Activity 통신에 있어 LocalBroadcastManager 를 사용 하였으며, 요즘에는 BroadcastReceiver의 혹시 모를 보안상 문제를 해결하기 위해 AIDL 을 사용하여 Process 통신을 하고 있습니다.  
최대한 이해를 돕기위해 실무적인 코드를 보여드리고 설명 드렸습니다.  
도움이 되셨길 바라며, 다음에는 **_Android 4대 컴포넌트 중 하나인 "ContentProvider"_**에 관한 포스팅으로 돌아오겠습니다. 

Android 4대 Component
- [Service](https://narvis2.github.io/posts/Android-Service/) 참고
- [Activity의 Lifecycle](https://narvis2.github.io/posts/Android-Activity-Lifecycle/) 참고