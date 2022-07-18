---
title: Android Coroutine
author: Narvis2
date: 2022-07-18 15:12:00 +0900
categories: [Android, Coroutine]
tags: [android, coroutine]
---

안녕하세요. Narvis2 입니다.  
오늘은 Kotlin Coroutines Flow에 대하여 알아보기 전 Coroutine에 대하여 알아보고자 합니다.  
우선 Coroutine에 대하여 알기전에 동기, 비동기에 대하여 아셔야합니다.  
간단하게 설명 드리면  
**_동기_**란 순차적으로 작업을 처리하는 모델(어떤 작업이 처리 중이면 다음 작업은 대기)을 의미합니다.  
**_비동기_**란 병렬적으로 작업을 처리하며, 어떤 작업을 처리중 이더라도 처리 될때까지 기다리지 않고 즉시 다음 작업을 처리합니다.(작업이 종료되는 시점에 처리 결과를 받습니다.)
이제 Coroutine에 대하여 알아보도록 하겠습니다.

## Coroutine
---
- 비동기 라이브러리 입니다.
- 경량 Thread : Thread간 Context 전환 비용이 발생하지 않으며, 개발자가 직접 중지 시점을 선택 가능하고 Thread를 재사용 가능합니다.
- 특징 : CoroutinScope 컨텍스트의 launch 빌더를 통하여 실행될 수 있으며, 이 빌더 블록안의 코드는 Thread처럼 비동기로 실행됩니다.
- Coroutine은 비동기 작업을 수행하면서 중지 상태일 때 Thread를 블로킹하지 않고 그 Thread를 재사용할 수 있기 때문에 더욱 효율적이고 빠르게 동작할 수 있습니다.

## suspend point
---
- Coroutine 을 사용하시려면 suspend 함수에 대하여 알아야 합니다.
- 현재 실행중인 작업을 중지하고 다른 작업을 수행시키는 함수 입니다.
- Coroutine이 사용중인 Thread를 블로킹하지 않으면서 실행중인 Coroutine을 잠시 중단 시킬 수 있는 중단 지점 함수 입니다.
- suspend 함수를 호출하는 시점에 현재 실행중인 Coroutine 은 잠시 중단되며, 그 때 남게되는 Thread는 다른 Coroutine에 할당될 수 있으며 Suspend 함수의 로직이 끝났을 때에 중단되었던 Coroutine은 다시 실행됩니다.
- suspend 함수는 Coroutine 내부에서 실행되거나 suspend 함수 내부에서 실행되어야 합니다. suspend 함수 호출 또한 다른 Coroutine에서 일어나거나 suspend함수 내부에서 호출되어야 합니다. 즉, **suspend 함수를 호출하기 위해서는 최소 하나의 Coroutine Builder 블록이 필요하게 됩니다.**
- 간단한 코드를 통해 알아보기 👇
``` kotlin
    runBlocking {
        launch { uniFuction() }
    }

    suspend fun somNetworkCall(): String {
        delay(1000)
        return "data from network"
    }

    suspend fun uniFuction() {
        val data = somNetworkCall()
        println(data)
        println("uniFuction is done")
    }
```
> 결과 : "data from network"
         "uniFuction is done"
> - 해설 : uniFuction() 함수를 Coroutine으로 호출하게 되면 내부적으로 somNetworkCall() 호출이 일어나게 되고, 이 지점에서 실행되고 있던 unifuction() 함수는 someNetworkCall() 함수의 호출이 끝날 때 까지 대기하다가 somNetwork() 함수 호출이 끝나면 다시 실행됩니다.  
즉, uiFuction()을 실행하고 있는 Thread는 대기하는 것이 아니라 다른 Coroutine에 할당될 수 있는 상태가 된다.

## Coroutine Scope
---
### 1. GlobalScope 
- 앱의 시작부터 종료까지 장시간 실행되어야 할 필요가 있을 경우 사용합니다.
- Application의 생명주기와 함께 동작합니다.(앱이 실행되는 동안에는 별도의 생명주기가 필요없습니다.) 즉, 앱 프로세스의 생명주기를 따라갑니다.

### 2. CoroutineScope 
- 작업 필요할 때만 실행하고 완료되면 종료됩니다.
- Coroutine의 기본 Scope로 다른 Scope는 Coroutine Scope를 상속 받아 처리하고 있습니다.
> **참고** : 주로 버튼을 클릭해서 서버의 정보를 가져오거나 파일을 열때 사용합니다.

### 3. MainScope
- UI 관련 작업을 처리하는 용도로 사용합니다.
- 이 Scope 안에서 만들어진 모든 Coroutine 을 Main Thread에서 실행합니다.

### 4. coroutineScope (소문자)
- 반환전에 제공되는 자식 범위 내의 모든 작업의 완료를 보장합니다.
- 구조화된 동기성, suspend 함수가 반환되기 전에 자식 범위 내에서 Coroutine에 의해 시작된 모든 작업을 완료하도록 보장합니다. 즉, coroutineScope 블록안의 suspend 함수가 완료 되면 반환합니다.

### 5. viewModelScope
- ViewModel 에 연결된 Coroutine Scope 입니다.
- viewModel이 활성 상태인 경우에만 실행해야 할 작업이 있을 경우 사용합니다.
- 이 범위에서 시작되는 모든 Coroutine은 ViewModel 이 삭제되면 자동으로 취소 됩니다.(onCleared()가 호출되면 자동으로 취소됨) 즉, 수동으로 onCleared()에서 Job Cancel을 할 필요가 없습니다.

### 6. lifecycleScope 
- Activity/Fragment 의 lifecycle 에 연결되 Coroutine Scope 입니다.
- 이 범위에서 시작된 모든 Coroutine 은 Lifecycle이 파괴되면 자동으로 취소됩니다.
- Activity/Fragment 와 같은 수명주기가 있는 객체에 Coroutine을 만들 때 사용합니다.

### 7. LiveData + Coroutine (LiveData Builder)
- LiveData를 사용할 때 값을 비동기적으로 계산해야 할 때 사용합니다.
- 사용자의 환경 설정을 검색하여 UI에 제공할 때 이런 경우 liveData { } 를 사용해 suspend 함수를 호출하여 결과를 LiveData 객체로 제공(emit) 합니다.
- emit() 을 통해 결과를 내보냅니다.
- LiveData가 활성화되면 실행을 시작하고 LiveData가 비활성화가 되면 구성 가능한 제한 시간 후 자동으로 취소 됩니다.
- 간단한 코드를 통해 알아보기 👇
``` kotlin
val isMobileDataOk: LiveData<Boolean> = liveData { // LiveData Builder 입니다.
    dataStore.getUserMobileData.collect { // DataStore에 저장된 값 가져오기
        emit(it) // 결과를 LiveData에 보내기
    }
}
```
  
## Coroutine Builder
---
### 1. runBlocking 
- 현재 Thread를 블록킹하는 Coroutine Builder 입니다.
- 일반 함수 내에서 suspend 함수를 호출하기 위해 사용할 수 있는 가장 단순한 형태의 Coroutine Builder 입니다.
- 내부 suspend 함수들도 모두 현재 Thread를 블로킹하게 됩니다.
- 주로 Test시 Top Level 함수로 사용되며, 주어진 블록이 완료될때 까지 현재 Thread를 멈추는 새로운 Coroutine을 생성하여 실행하는 Coroutine Builder 입니다.
- 잘 사용하지 않으나 Test 코드를 짤때 주로 사용합니다.
> **_주의!!_** 실무에서는 잘 사용하지 않습니다. 그 이유는 Context가 Main Thread일 때 runBlocking 을 넣으면 오류를 유발할 수 있습니다.  
여기서 오류란 Main Thread를 장기간 블럭킹하여 ANR 을 유발할 수 있습니다. 따라서 runBlocking 은 Unit Test에서 주로 사용합니다.  

### 2. launch
- 가장 많이 사용하는 Coroutine Builder 입니다.
- 현재 Thread를 블록킹 하지 않고 새로운 비동기 작업을 시작합니다. 
- Coroutine이 시작되었다는 의미의 Job 객체를 반환합니다. Job 객체는 Coroutine의 종료를 기다리거나 취소를 기다리기 위해서 사용됩니다.
- isCancelled 프로퍼티를 이용하여 작업이 성공인지, 실패인지 확인할 수 있습니다.
- 결과 값을 반환받을 수 없기 때문에 파이어 앤드 포켓 방식의 UseCase에서 많이 사용합니다. 
> **참고** : 예외가 전파되지 않기 때문에 블록 내부에서 CoroutineExceptionHandler 와 함께 try-catch 가 필요할 수 있습니다. 
- CoroutineExceptionHandler 를 사용한 Sample Code 입니다. 👇
``` kotlin
fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { coroutineContext, throwable ->
        println("Caught:${coroutineContext[CoroutineName]}, ${throwable.message?.substring(0..28)}")
    }

    try {
        val airportCodes = listOf("LAX", "SF-", "PD-", "SEA")
        val jobs: List<Job> = airportCodes.map { onAirportCode ->
            // SupervisorJob -> 단 방향 취소가 가능하게 만들어줍니다. (부모에서 자식으로만 단방향으로 취소가 가능하게 만들어줍니다.)
            launch(Dispatchers.IO + CoroutineName(onAirportCode) + handler + SupervisorJob()) {
                val airport = Airport.getAirportData(onAirportCode) // Network 요청
                println("${airport.code}")
            }
        }

        jobs.forEach { it.join() }
        jobs.forEach { println("cancelled : ${it.isCancelled}") }
    } catch (e:Exception) {
        println("ERROR: ${e.message}")
    }
}
```
  
### 3. async
- 현재 Thread를 블록킹하지 않고 새로운 비동기 작업을 시작합니다.
- Deferred<T> 타입의 객체를 반환하며 await()를 호출하여 결과 값을 반환 받을 수 있습니다.
- await()는 susepnd 함수 이기에 Coroutine 내부나 또 다른 susepnd 함수 내부에서 호출되어야 합니다.
> **참고** : 예외가 전파되기 때문에 블록 외부에서 try-catch 가 가능합니다.

### 4. withContext
- 현재 Thread를 블록킹하지 않고 새로운 Coroutine을 실행할 수 있습니다.
- async 처럼 결과값을 반환하는 빌더입니다. async는 반환하는 Deferred<T> 객체로 결과값을 원하는 시점에 await()함수를 통해 결과값을 얻지만, withContext()는 Deferred<T>객체로 반환하지 않고, 결과(T)를 그 자리에서 반납합니다.
- 코드의 한 부분을 Coroutine의 다른 코드들과 완전히 다른 Context에서 실행할 수 있습니다.(Coroutine을 한 Context에서 실행하다가 중간에 Context를 바꾸고 싶을 때 사용합니다.) 

## Coroutine Context
---
### 1. Dispatchers.Default
- Coroutine에게 DefaultDispatchers 풀(pool)의 Thread 안에서 실행을 시작하라고 지시합니다.
- 풀(Pool) 안의 Thread 숫자는 2개 이거나 시스템의 코어 숫자 중 높은 것을 사용합니다. 
- 계산한 일이 많은 작업을 위한 풀(Pool)입니다. (cpu 에서 처리하는 대부분의 작업들에 사용합니다.)
> **참고** : 데이터 처리, 이미지 처리 등에 사용합니다.
  
### 2. Dispatchers.IO
- IO 작업 실행을 위한 풀(Pool)안에 Coroutine을 실행시키는 데 사용됩니다. 
- Local, Network 에서 데이터를 읽을 때 사용합니다.
> **참고** : 네트워크 작업, 이미지 다운로드, 파일 입출력 등의 입출력에 최적화 되어있습니다.  
> - IO에서 UI 변경 시 IOException 이 발생 하므로 UI 변경은 Main Thread에서 하셔야 합니다.
  
### 3. Dispatchers.Main
- Main Thread 에서만 사용되는 UI 업데이트 기능이 필요할 때 사용합니다.
- Android Main Thread UI 작업에 주로 사용됩니다.
> **참고** : 항상 Main Thread로 Coroutine을 시작한 다음 Background Thread로 전환하는 것이 좋습니다.

### 4. Custom Pool 에서 실행 시키기
- 풀(Pool)안에 Thread가 있기 때문에 이 Context를 사용하는 Coroutine은 **_병렬_** 진행이 아닌 **_동시_** 진행으로 진행됩니다.
- 작업을 Coroutine으로 진행시킬 때 작업들 간에 자원 경쟁에 대해서 고려할 때 사용합니다.
- 싱글 Thread 생성자 만들기 👇
``` kotlin
Executors.newSingleThreadExecutor()
```
- 실행자로 부터 CoroutineContext 가져오기 👇 (.asCoroutineDispatcher() 확장함수 사용)
``` kotlin
Executors.newSingleThreadExecutor().asCoroutineDispatcher().use { context ->
    runBlocking {
        // launch 에 context를 전달하면 해당 블럭에서 실행되는 Coroutine은 Single Thread Pool에서 동작합니다.
        launch(context) {
            // TODO: Coroutine 에서 실행할 함수 및 동작
        }
    }
}
```
> **_주의!!_** 실행자를 닫지 않으면 프로그램이 영원히 멈추지 않습니다. 그 이유는 실행자의 풀(Pool)에는 Main Thread 외에도 Active Thraed가 있고, Active Thread가 JVM 을 계속 살려두게 되기 때문입니다.  
**_해결!!_** 위의 상황을 해결하기 위해 .use{context -> }를 사용합니다.
- 다음은 Multi Thread 를 가지는 풀(Pool) 사용 코드 입니다. 즉 시스템 코어의 숫자 만큼 Thread를 이용하고 싶을 때 사용합니다. 👇
``` kotlin
Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors())
            .asCoroutineDispatcher().use { context ->
                runBlocking {
                    launch(context) {
                        // TODO: Coroutine 에서 실행할 함수 및 동작 
                    }
                }
            }
```
> **참고** 이 context를 사용하는 Coroutine은 코드가 동작하는 시스템의 코어 숫자와 동일한 Thread 숫자를 가지는 커스텀 Pool에서 실행됩니다.

## Mutex (상호배제)
- 상호 배제란 한번에 하나의 Coroutine만 코드 블럭을 실행할 수 있도록하는 동기화 메커니즘 입니다. 즉, 모든 공유되는 상태의 변경들이 절대 동시 실행되지 않도록 합니다.
- 동시에 실행되면 안되는 부분을 lock()/unlock() 으로 보호합니다. (List의 값을 변경하는 부분에 사용될 수 있습니다.)
- 해당 Thread는 Block의 작업이 다 처리될 때 까지 다른 작업을 수행할 수 없습니다. (차단 역할)
``` kotlin
private val mutex = Mutex()
private suspend fun test() {
    mutex.withLock {
        // TODO: something..
    }
}
```

## 마무리
이번 포스팅에서는 Coroutine의 기본 개념에 대하여 알아보았습니다. Coroutine을 사용하면 매우 간결하게 비동기 처리가 됩니다.  
Coroutine 은 경량 Thread로 비동기 작업을 수행하면서 중지 상태일 때 Thread를 블로킹하지 않고 그 Thread를 재사용할 수 있기 때문에 더욱 효율적이고 빠르게 동작할 수 있습니다.  
다음 포스팅에서는 Coroutine Flow에 대하여 알아보도록 하겠습니다.