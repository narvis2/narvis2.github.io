---
title: Android StateFlow, SharedFlow, Coroutine Channel 에 대하여
author: Narvis2
date: 2022-07-21 11:11:00 +0900
categories: [Android, Coroutine]
tags: [android, stateFlow, sharedFlow, channel, coroutine]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `StateFlow`, `SharedFlow`, `Coroutine Channel` 에 대하여 알아보고자 합니다.  
아직 `Flow`에 대하여 잘 모르시면 `Flow` 포스팅을 참고하시길 바랍니다. 👉 [Flow에 관하여](http://127.0.0.1:4000/posts/Android-Coroutine-Flow/)  
`Flow`는 `Cold Stream`입니다. 즉, **_`collector` 를 공유할 수 없어 `collect`를 할때마다 각기 다른 데이터가 수집됩니다._**  
`StateFlow`, `SharedFlow`는 `Flow`의 **_단점을 극복하기 위해_** 나왔습니다.  
`StateFlow`, `SharedFlow`는 `Hot Stream`으로 **_`collector`를 공유_** 합니다. 자세한건 밑에서 알아보도록 하겠습니다.

## 🍀 Flow의 단점

- `Flow`는 스스로 `Android`의 `Lifecycle`에 대해 알지 못합니다.
  > - ✅ 즉, **_`Lifecycle`에 따른 중지나 재개가 어렵습니다._**
- `Flow`는 상태가 없어 값이 할당된 것인지, 현재 값은 무엇인지 알기 어렵습니다.
- `Flow`는 `Cold Stream` 방식으로, 연속해서 계속 들어오는 데이터를 처리할 수 없으며, **_`Collect` 되었을 때만 생성되고 값을 반환_** 합니다.
  > **_참고_** 👉 만약, 하나의 `flow builder` 에 대해 다수의 `Collector` 가 있다면 **_`Collector` 하나마다 하나씩 데이터를 호출_** 하기 때문에 `UpSteam`이 비싼 비용을 요구하는 `DB 접근`이나 `서버 통신` 등이라면 **_여러 번 리소스 요청_** 을 하게 될 수 있습니다.

## 🍀 StateFlow

- `Hot Stream` 방식입니다. 즉, **_`collect`가 공유 되어 오직 하나의 `Flow`만을 실행_** 하게 합니다.
  > **_✅ 참고_** 👉 `Flow`와 다르게 하나의 `StateFlow`를 통해 `DB에 접근`할 때 **_여러개의 `Collect`를 쓰더라도 `DB`에는 한번만 접근_** 합니다.
- **_초기 데이터(기본 값)이 항상 존재_** 해야 합니다.
- **_마지막 값의 개념이 있으며_** 생성하자 마자 활성화 됩니다.
- 값이 업데이트 된 경우에만 반환하고 **_동일한 값을 반환하지 않습니다._**
  > `Flow` 의 `distinctUtilChanged()`가 항상 포함되어 있다고 생각하시면 됩니다.
- `.value`를 사용하여 **_현재 값에 접근_** 할 수 있습니다.
- `SharedFlow` 의 `replay` 값이 `1`로 고정된 경우와 같습니다.

  > **_참고_** 👉 새로운 `subscriber`가 등록될 때 **_바로 최신의 값을 가져_** 옵니다.

  > **_1️⃣ 예제_** ) `StateFlow` 생성 👇🏾

  ```kotlin
  @HiltViewModel
  class MainViewModel @Inject constructor() : ViewModel() {
      // StateFlow 생성
      private val _isAdult = MutableStateFlow(false)
      val isAdult: StateFlow<Boolean> = _isAudult.asStateFlow()

      // StateFlow에 값 넣기
      fun onIsAdultClick() {
          _isAdult.value = !isAdult.value
      }
  }
  ```

  > **_2️⃣ 예제_** ) `StateFlow` `Collect` 👇🏾

  ```kotlin
  @AndroidEntryPoint
  class MainActivity : AppCompatActivity() {
      private lateinit var binding: ActivityMainBinding
      private val mainViewModel: MainViewModel by viewModels()

      override fun onCreate(saveInstanceState: Bundle?) {
          binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
          binding.lifecycleOwner = this

          lifecycleScope.launchWhenStarted {
              mainViewModel.isAdult.collect { it: Boolean
                  // TODO : true, false 동작 넣기
              }
          }
      }
  }
  ```

## 🍀 SharedFlow

- `Hot Stream` 방식으로 여러개의 `Collector`를 공유하여 오직 하나의 `Flow`만을 실행하게 합니다.( **_아무리 많은 `Collector`가 존재해도 오직 하나의 `Flow`만 실행_** )
- `firstOrNull`을 통해 **_현재 값에 접근 가능_** 합니다.
- **_초기 값이 필요없으며, 중복된 값이 필요한 경우 사용_**
  > **_✅ 참고_** 👉 중복 값이 필요한 경우란 `Event`를 말하는데 예를들어 API를 호출하였을 때 **_실패의 경우 연속 적으로 실패할 수 있으므로_** 이럴때는 `SharedFlow`를 사용합니다.
- 만약, `SharedFlow` 한 개를 정의하고 `Database` 결과값을 공유하게 한 뒤, **_여러 개의 `Collector`를 달아 준다면 `Database` 접근은 오직 한 번만 일어납니다._**
- `SharedFlow` 파라미터 👇
  - `replay` : `Collect` 시 전달받을 이전 데이터의 개수 지정, `replay`가 `0`이면 `Collect` 시점에 담겨있던 데이터부터 전달 받습니다. `replay`가 `1`이면 `collect` 시점 직전의 데이터부터 전달 받으며 시작(최신 데이터)합니다.
    - `0` : `collect` 이후의 데이터를 전달 받습니다. (새로운 구독자에게 이전 이벤트를 전달하지 않습니다.)
    - `1` : `collect` 시점 직전의 데이터부터 전달받으며 시작합니다.( **_최신 데이터_** )
    - `2` : 현재 데이터 이전 두개의 데이터 부터 전달받으면서 시작합니다.
  - `extraBufferCapacity` : `buffer`의 개수를 설정합니다. **_`flow`의 `emit`이 빠르고 `collect`가 느릴 때_** 지정된 개수만큼 `buffer`에 저장되고, **_저장된 개수가 넘어가면 `'onBufferOverFlow'`에 설정된 정책에 따라 동작_** 합니다.
    - `onBufferOverFlow` : `Buffer` 가 꽉 찼을 때 동작을 정의합니다.
      - `SUSPEND` : `buffer`가 꽉 찼을 때 `emit`을 수행하면 `emit` 코드가 `blocking` 됩니다. 즉, **_`buffer`의 빈자리가 생겨야 `emit`코드 이후의 코드가 수행 될 수 있습니다._**
      - `DROP_OLDSET` : `buffer`가 꽉 찼을 때 `emit`을 수행하면 **_오래된 데이터부터 삭제하면서_** 새로운 데이터를 넣습니다.
      - `DROP_LATEST` : `buffer` 가 꽉 찼을 때 `emit`을 수행하면 **_최근 데이터를 삭제하고_** 새로운 데이터를 넣습니다.

## 🍀 StateIn / SharedIn

- `Flow Builder` 로 만든 **_`Clod Flow`를 `Hot Flow`로 변경_** 할 수 있는 확장함수 입니다.
- **_하나의 `Flow` 에서 방출된 값을 여러개의 `Collector` 에서 받아야할 경우_** 에 유용하게 사용됩니다.
- `stateIn()` / `sharedIn()` 파라미터

  - `scope` 👉 공유가 시작되는 `Coroutine Scope`를 설정합니다.
  - `started` 👉 공유가 시작 및 중지되는 **_시기를 제어_** 하는 전략을 설정합니다.
    - `Eagerly` 👇
      > - `Collector` 가 존재하지 않더라도 바로 `Sharing`이 시작되며, 중간에 중지되지 않습니다.
      > - 값이 `replay` 크기 보다 많이 들어오면 바로 삭제됩니다. 즉, **_즉시 시작되며 `Scope`가 취소되면 중지됩니다._**
    - `Lazily` 👇
      > - `Collector` 가 등록된 이후부터 `Sharing`이 시작되며 중간에 중지되지 않습니다.
      > - 첫 번째 `Collector`는 그 동안 `emit`된 모든 값들을 얻으며, 이후에 `Collector`는 `replay` 개수 만큼 값을 얻어갑니다.
      > - ✅ 즉, **_`Collector`가 모두 없어지더라도 `Sharing` 동작을 유지되며 `replay` 개수만큼 `Cache`하고 `Sopce`가 취소되면 중지됩니다._**
    - `WhileSubscribed` 👇
      - **_`Collector` 가 등록되면 바로 `Sharing`을 시작하며, `Collector` 가 전부 없어지면 바로 `Sharing`을 중지_** 합니다.
      - 이때 `replay` 개수만큼 `Cache` 처리 됩니다.
        - `stopTimeOutMillis` 👇
          - `collector`가 모두 사라진 이후에 정지할 `delay`를 넣습니다. 즉, **_`Collect`가 사라지고 몇 초 후에 `Sharing`을 중지할지 설정_** 합니다.
          - `0`이면 `Collector`가 모두 사라지는 순간에 바로 정지합니다.
          - **_`5,000`을 사용하면 `Configuration(구성요소)` 변경과 같은 특정 상황에서 이득을 볼 수 있습니다._**
        - `replayExpirationMillis` 👇
          - **_`cache` 한 값을 유지할 시간을_** 정합니다.
          - 시간이 지나면 `stateIn`의 **_초기 값으로 복원_** 됩니다.
          - 기본 값 👉 `replay cache`를 **_영구적으로 유지하며_** `buffer`를 재 생성하지 않습니다.
          - `0`을 사용 👉 **_`cache`를 즉각적으로 만료 시킬 수 있습니다._**
  - `initialValue` 👉 `stataIn` 사용 시 **_초기 값을 설정_** 합니다.
  - `replay` 👇

    - `sharedIn` 사용 시 사용되며( **_`StateFlow`는 `replace` 값이 `1`로 고정되어 있습니다._** ), **_`Collect` 시 전달받을 이전 데이터의 개수 지정_** 지정
    - `replay`가 `0` 👉 `Collect` 시점에 담겨있던 데이터부터 전달 받습니다.
    - `replay`가 `1` 👉 `collect` 시점 직전의 데이터부터 전달 받으며 시작합니다.( **_최신 데이터_** )

      > - **_❗️주의❗️_** 👉 `stateIn`, `sharedIn`을 **_함수로 만들면 매번 재 사용되지 않는 새로운 `instance`를 만들게 됩니다._**
      > - 아래는 해당 코드의 샘플 **_예제_** 입니다. 👇🏾

      ```kotlin
      @HiltViewModel
      class MainViewModel @Inject constructor(
          private val getCastInfoUseCase: GetCastInfoUseCase
      ) : ViewModel() {
          // 해당 Result Sealed Class 는 보다 이해를 돕기위해 ViewModel에서 생성하였습니다.
          sealed class Result<T>(
              val data: T? = null,
              val message: String? = null
          ) {
              class Loading<T>: Result<T>()
              class Success<T>(data: T) : Result<T>(data)
              class NetworkError<T>(message: String?, data: T? = null) : Result<T>(data, message)
              class Error<T>(message: String?, data: T? = null) : Result<T>(data, message)
          }
          private val getCastInfo: StateFlow<Result<CastInfoResponseModel>> = getCastInfoUseCase()
                  .stateIn(
                      scope = viewModelScope,
                      started = SharingStarted.WhileSubscribed(5000),
                      initialValue = Result.Loading()
                  )
          fun requestCastInfo() = viewModelScope.launch {
              getCastInfo().collect { result ->
                  when (result) {
                      is Result.Loading -> {
                          showLoadingDialog()
                      }
                      is Result.Success -> {
                          hideLoadingDialog()
                          // TODO : API 요청 성공일 때 동작 넣기
                      }
                      is Result.NetworkError -> {
                          hideLoadingDialog()
                          // TODO : Network 오류 발생 때 동작 넣기
                      }
                      is Result.Error -> {
                          hideLoadingDialog()
                          // TODO : API ERROR 발생 때 동작 넣기
                      }
                  }
              }
          }
      }
      ```

## 🍀 Coroutine Channel

- **_`Channel`은 단방향 `Observing`_** 이라고 생각하시면 쉽습니다.
  > **_✅ 참고_**
  >
  > - `LiveData`에서는 단방향 옵저빙을 위해 `SingleLiveEvent` 혹은 `Event Wrapper` 를 사용합니다. `Coroutine Channel` 은 `Flow`의 **_단방향 옵저빙_** 이라고 생각하시면 됩니다.
- `Channel`은 **_정확히 한 번만 처리해야하는 이벤트_** 를 처리하는 데 사용됩니다. 이는 일반적으로 단일 구독자 가있는 이벤트 유형의 설계에서 사용됩니다.
  > **_✅ 참고_**
  >
  > - 예를 들면 **_Click Event Listener_** 가 이에 해당합니다.
  > - `Click Event Listener`는 항상 `Observing` 할 필요없이 **_눌렀을 때만 `Observing` 하면 되므로_** 이럴 때 `Channel` 을 사용합니다.
- **_`send` 값이 오지않으면 `Observing`을 하지않고, `send` 값이 들어오면 `Observing` 시작합니다._**
- `Channel`의 `Buffer Type`

  - `Rendezvous` (Unbuffered) : 기본 타입으로 **_버퍼가 없습니다._**
  - `Conflated` : **_크기가 1인 고정 버퍼가 있는 채널이 생성_** 됩니다. 만약에 수신하는 `Coroutine`이 송신하는 `Coroutine`을 따라잡지 못했다면, 송신하는 쪽은 새로운 값을 `Buffer`의 마지막 아이템에 덮어씌웁니다. 즉, **_최신 값을 받습니다._**
  - `Buffered` : 이 모드는 **_고정된 크기의 버퍼를 생성_** (`Buffer`는 `Array` 형식)합니다. `송신` `Coroutine`은 **_`Buffer`가 꽉 차있으면 새로운 값을 보내는 걸 중단_** 합니다. `수신 Coroutine` 은 `Buffer`가 빌때까지 계속해서 꺼내서 수행합니다.
  - `Unlimited` : **_제한 없는 크기의 버퍼를 생성_** (`Buffer`는 `LinkedList` 형식)합니다. 만약에 `Buffer`가 소비되지 않았다면 메모리가 힘들어할때까지 계속해서 아이템을 착착 채우고 **_결국엔 `OutOfMemeoryException`을 일으키게_** 됩니다.

  > 1️⃣ 다음은 `Coroutine Channel` **_예제_** 입니다. 👇🏾

  ```kotlin
  @HiltViewModel
  class MainViewModel @Inject constructor() : ViewModel() {
      private val _actionLogin = Channel<Unit>(Channel.CONFLATED)
      val actionLogin = _actionLogin.receiveAsFlow()

      fun onLoginClick() = viewModelScope.launch {
          _actionLogin.send(Unit)
      }
  }
  ```

  > 2️⃣ 다음은 `수신`을 받아서 사용하는 쪽 **_예제_** 입니다.

  ```kotlin
  @AndroidEntryPoint
  class MainActivity : AppCompatActivity() {
      private lateinit var binding: ActivityMainBinding
      private val mainViewModel: MainViewModel by viewModels()

      override fun onCreate(saveInstanceState: Bundle?) {
          binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
          binding.lifecycleOwner = this

          lifecycleScope.launchWhenStarted {
              mainViewModel.actionLogin.collect {
                  // TODO: 로그인 버튼 클릭 시 동작할 코드 넣기
              }
          }
      }
  }
  ```

## 🍀 Flow Observer

- `Flow`는 스스로 `Android`의 `Lifecycle`에 대해 알지 못합니다. ( **_`Lifecycle`에 따른 중지나 재개가 어렵습니다._** ) 따라서 해당 문제를 해결하기 위해 `Custom Class`를 만듭니다.
- 해당 코드는 **_`lifecycle` 이 `onStart`가 되면 구독을 `시작`하고, `onStop`이 되면 구독 `취소`_** 합니다.

  > 1️⃣ 다음은 `FlowObserver` **_예제_** 코드 입니다. 👇

  ```kotlin
  class FlowObserverInStop<T>(
      lifecycleOwner: LifecycleOwner,
      private val flow: Flow<T>,
      private val collector: suspend (T) -> Unit
  ) {
      private var job: Job? = null
      init {
          lifecycleOwner.lifecycle.addObserver(LifecycleEventObserver { source, event ->
              when (event) {
                  Lifecycle.Event.ON_START -> {
                      job = source.lifecycleScope.launch {
                          flow.collect {
                              collector(it)
                          }
                      }
                  }
                  Lifecycle.Event.ON_STOP -> {
                      job?.cancel()
                      job = null
                  }
                  else -> {}
              }
          })
      }
  }

  inline fun <reified T> Flow<T>.observeOnLifecycleStop(
      lifecycleOwner: LifecycleOwner,
      noinline collector: suspend (T) -> Unit
  ) = FlowObserverInStop(lifecycleOwner, this, collector)

  // .onEach{ } 사용할때 사용
  inline fun <reified T> Flow<T>.observeInLifecycleStop(
      lifecycleOwner: LifecycleOwner
  ) = FlowObserverInStop(lifecycleOwner, this) {}
  ```

  > **_✅ 설명_** 👇
  >
  > - `observeOnLifecycleStop()` 의 매개변수에는 `lifecycle` 을 넣습니다. `Activity`의 경우 `this` 이고, `Fragment`인 경우 `viewLifecycleOwner`를 넣어주시면 됩니다.

  > 2️⃣ `FlowObserver` 사용 **_예제_** 입니다. 👇🏾

  ```kotlin
  @AndroidEntryPoint
  class MainActivity : AppCompatActivity() {
      private lateinit var binding: ActivityMainBinding
      private val mainViewModel: MainViewModel by viewModels()

      override fun onCreate(saveInstanceState: Bundle?) {
          binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
          binding.lifecycleOwner = this

          // StateFlow Observing
          mainViewModel.actionLogin.observeOnLifecycleStop(this) {
              // TODO : true, false 동작 넣기
          }

          // Coroutine Channel Observing
          mainViewModel.actionLogin.onEach {
              // TODO: 로그인 버튼 클릭 시 동작할 코드 넣기
          }.observeInLifecycleStop(this)
      }
  }
  ```

## 🍀 마치며

이번 포스팅에서는 `StateFlow`, `SharedFlow`, `Coroutine Channel` 에 대하여 알아보았습니다.  
`LiveData`를 사용하고 `Event Wrapper`를 통해 `단방향 Observing`을 처리해도 되지만 문제는 `Clean Architecture`에 있습니다.

**_`LiveData`는 `UI`에 밀접하게 연관되어 있기 때문에_** `Clean Architecture`로 `Project`를 구성하면 `Domain`, `Data layer`에서 **_비동기 방식으로 데이터를 처리하기에 자연스러운 방법이 없습니다._**

또한 `LiveData` 는 **_안드로이드 플랫폼에 속해 있기 때문에_** 순수 `Java` / `Kotlin` 을 사용해야 하는 **_`Domain Layer` 에서 사용하기에 적합하지 않습니다._**  
이럴 경우 `LiveData`를 대체하여 **_`StateFlow` 나 `SharedFlow` 또는 `Coroutine Channel`을 사용_** 할 수 있습니다.
