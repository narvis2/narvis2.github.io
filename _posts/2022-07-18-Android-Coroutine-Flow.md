---
title: Android Coroutine Flow
author: Narvis2
date: 2022-07-18 15:28:00 +0900
categories: [Android, Coroutine]
tags: [android, coroutine, flow]
---

안녕하세요. Narvis2 입니다.  
오늘은 Kotlin Coroutines Flow에 대하여 알아보도록 하겠습니다.  
suspend 함수는 비동기로 단일 값을 리턴합니다.  
Flow는 비동기로 계산된 여러값을 리턴합니다. 연속적으로 값을 가져와야하는 상황(시시각각 변화하는 온도를 가져올 때)에서 주로 사용합니다. 즉, 비동기로 계산된 값을 실시간으로 나타낼 수 있습니다.

## Flow
---
- **Coroutine의 Flow는 Data Stream이며, Coroutine 상에서 Reactive Programming을 지원하기 위한 구성요소입니다.**
> **_참고_** Reactive Programming(반응형 프로그래밍) : Data가 변경 될 때 이벤트를 발생시켜 데이터를 계속해서 전달하도록 하는 프로그래밍 방식 입니다.  
- Clod Stream 형식으로, 외부에서 설정하는 데이터가 아닌 생성 시 해당 데이터를 정의해야 합니다. 또한 단 한명의 구독자가 존재하며, 데이터 발생 시점의 주체가 구독자 입니다.
- flow {} 를 통해 빌드합니다. 코드 내부가 일시 중단 될 수 있으며, collect{}가 호출될 때까지 코드가 실행되지 않습니다.
- suspend 수정자를 붙히지 않아도 됩니다. flow는 자체적으로 취소할 수 없습니다.
- emit()을 통해 값을 내보내고, Coroutine Builder 내에서 collect {} 를 통하여 값을 받습니다.
- flow {} 빌더의 코드는 Context Preservation(컨텍스트 보존) 특성을 준수해야 하며, 다른 Context에서 emit 할 수 없습니다. 즉 withContext를 사용하지 못합니다.
- flowOf() : 고정 값을 외부로 보내는 flow 입니다. 👇
``` kotlin
fun main() = runBlocking {
    sendNumbers().collect {
        Timber.e("Received $it")
    }
}
// Flow 생성
fun sendNumbers() = flowOf("ONE", "TWO", "THREE")
```
- asFlow() : 변수의 Collection, 시퀸스를 변경할 수 있습니다.
             Collection을 flow로 직접 변환하며 emit()을 할 필요가 없습니다.
             suspend 함수나 Coroutine Builder 안에서 사용합니다. 👇
``` kotlin
fun main() = runBlocking<Unit> {
    (1..3).asFlow().collect { response: Int ->
        Timber.e("response -> $response")
    }
}
```

## Flow Operator
---
입력 Flow를 가져와서 **_변환_**하고 출력 Flow를 **_반환_**하며 반환하는 Flow는 동기 입니다.  
Collect {} 함수를 호출하지 않으면 실행되지 않습니다.
### 1. transform
- transform{} 을 통해 여러개의 값을 매핑(하나의 값을 다른 값으로 대응 시키는 것)시켜 전달할 수 있습니다.
- take() : 리스트의 갯수를 제한합니다. 첫 번째부터 지정된 Count의 수 만큼의 요소를 포함하는 Flow를 반환합니다.
- 다음은 예제 코드 입니다. 👇
``` kotlin
fun main() = runBlocking<Unit> {
    (1..3).asFlow()
        .take(2)
        .transform { request: Int ->
            emit("Making request -> $request")
            emit(performRequest(request))
        }
        .collect { response: String -> 
            Timber.e("Response -> $response")
        }
}
suspend fun performRequest(request: Int): String {
    delay(1000)
    return "response -> $request"
}
```
> **결과** -> Making request 1, response 1, Making request 2, response 2

### 2. flowOn 
- flow{} 빌더의 코드는 다른 Context에서 emit() 사용은 불가합니다. 즉, flow{} 안에서 withContext 를 사용하여 다른 Thread로 바꿀 수 없습니다.  
이럴 때 사용하는 것이 flowOn 입니다.  
- flow {} 빌더 코드를 다른 Thread에서 emit() 시키고 싶을 때 즉, Flow가 방출되는 Context를 전환할 때 flowOn() 함수를 사용합니다. 
- 다음은 예제 코드 입니다. 👇
``` kotlin
fun main() = runBlocking<Unit> {
    foo().collect { value: Int -> // 기본 Thread 즉 MainThread에서 실행됩니다.
        Timber.e("[${Thread.currentThread().name} Collected : $value]")
    }
}
private fun foo(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(1000)
        Timber.e("[${Thread.currentThread().name} Emitting : $i]")
        emit(i)
    }
}.flowOn(Dispatchers.Default)
```
>**결과**   
 [DefaultDispatcher-worker-1 Emitting : 1]  
 [main Collected : 1]  
 [DefaultDispatcher-worker-1 Emitting : 2]  
 [main Collected : 2]  
 [DefaultDispatcher-worker-1 Emitting : 3]  
 [main Collected : 3]  
 **Main Thread에서 Collect 되는 동안 foo() 함수에서는 다른 Thread에서 진행됩니다.**
  
### 3. buffer
 - flow {} 가 구현된 함수를 호출할 때 속도를 빠르게 할 수 있습니다.
 - Flow 처리에 오랜 시간이 걸리는 경우 buffer()는 나중에 처리할 수 있는 Flow값을 축적하는데 유용합니다.
 - 다음은 예제 코드 입니다. 👇
``` kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        generate().buffer().collect { it: Int
            delay(300L)
            Timber.e("collect -> $it")
        }
    }
    Timber.e("Collect in $time ms")
}
fun generate() = flow {
    for (i in 1..3) {
        delay(100L)
        emit(i)
    }
}
```
> **결과** : buffer 사용 = 약 1,000 ms, buffer 사용 x = 약 1,250 ms

### 4. map 
map 을 이용한 Mapping 예제입니다. 문자열에 매핑합니다.
``` kotlin
fun main() { runBlocking{ mapOperator() } }
suspend fun mapOperator() {
   (1..10).asFlow().map { it: Int 
       delay(500L)
       "mapping $it"
   }.collect { it: String
       Timber.e("mapped values -> $it")
   }
}
```
### 5. Filter
filter 를 이용한 예제 입니다. 2의 배수만 출력하겠습니다.(짝수만 출력)
``` kotlin
fun main() { runBlocking{ filterOpertaor() } }
suspend fun filterOperator() {
   (1..10).asFlow().filter { it: Int
          it%2 == 0
   }.collect { it: Int
       Timber.e("filtered value -> $it")
   }
}
``` 
### 6. zip
2개의 Flow를 결합하여 하나의 Flow로 만듭니다.  
2개의 Flow를 결합시켜 각 값을 다른 flow의 해당 값과 일치시킵니다.
``` kotlin
fun main() {
    runBlocking<Unit> {
        val nums: Flow<Int> = (1..3).asFlow()
        val strs: Flow<String> flowOf("one","two")
        nums.zip(strs) { t1: Int, t2: String ->
            "$t1 -> $t2"
        }.collect { it: String
            Timber.e("response -> $it")
        }
    }
}
```
> **결과** 1 -> one, 2 -> two  
nums 와 strs 의 사이즈가 다르기 때문에 사이즈가 작은 strs 를 기준으로 출력됩니다. 즉, nums 의 3은 삭제됩니다.

### 7. flatternConcat, flatMapConcat, flatMapMerge
return 타입이 Flow<Flow<T>> 일때 이 타입을 Flow<T> 로 만들 수 있는 flattern(평탄화) 작업이 필요합니다.  
이때 사용하는 것이 flatternConcat, flatMapConcat, flatMapMerge 입니다.
``` kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i : first")
    delay(500)
    emit("$i : second")
}
```
- 다음은 flattenConcat() 사용 예제 입니다.
``` kotlin
fun main() = runBlocking<Unit> {
    val beforeFlattening: Flow<Flow<String>> = (1..3).asFlow().onEach { 
        delay(1000)
        }.map { 
            requestFlow(it)
        }
            
    val flatten: Flow<String> = beforeFlattening.flattenConcat()
    flatten.collect {
        Timber.e("flatten response -> $it")
    }
}
```
> **결과**  
1 : first  
1 : second  
2 : first  
2 : second  
3 : first  
3 : second  

- 다음은 flatMapConcat 사용 예제 입니다.
``` kotlin
fun main() = runBlocking<Unit> {
    (1..3).asFlow().onEach { 
        elay(1000)
    }.flatMapConcat { 
        requestFlow(it)
    }.collect { 
        Timber.e("flatten response : $it")
    }
}
```
> **결과**  
1 : first  
1 : second  
2 : first  
2 : second  
3 : first  
3 : second  

- 다음은 flatMapMerge 를 사용한 예제 입니다.(flatMapMerge는 들어오는 Flow를 동시에 실행시켜 단일 Flow로 가능한 빨리 값을 전달합니다.)
``` kotlin
fun main() = runBlocking<Unit> {
    (1..3).asFlow().onEach { 
        elay(1000)
    }.flatMapMerge { 
        requestFlow(it)
    }.collect { 
        Timber.e("flatten response : $it")
    }
}
```
> **결과**  
1 : first  
2 : first  
3 : first  
1 : second  
2 : second  
3 : second  

### 8. Combine
- 한 Flow의 최신 값을 다른 Flow의 최신 값과 결합합니다.
- Combine() 연산자의 출력은 기본적으로 모든 단일 Flow의 최신값 입니다.
``` kotlin
fun main() = runBlocking<Unit> {
    combine()
} 
private suspend fun combine() {
    val numbers = (1..5).asFlow().onEach { 
        delay(300L) 
    }

    val values = flowOf("One", "Two", "Three", "Four", "Five").onEach {
        delay(400L)
    }

    numbers.combine(values) { a: Int, b: String ->
        "$a -> $b"
    }.collect { it: String
        Timber.e("combine response -> $it")
    }
}
```
> **결과**  
1 -> One  
2 -> One  
2 -> Two  
3 -> Two  
4 -> Two  
4 -> Three  
5 -> Three  
5 -> Four  
5 -> Five  
combine() 대신 zip()을 사용했을 경우 👇  
1 -> One  
2 -> Two  
3 -> Three  
4 -> Four  
5 -> Five  

### 9. Exception handler Flow
- try / catch로 둘러 싸기, emit() 과 collect{}를 모두 try/catch 문으로 둘러싸서 지금 발생하는 모든 Exception을 표착합니다.
- catch {} : 예외 Operator, collect{}를 하기전에 사용하며, catch {} 이후에 다른 Exception이 발생하면 예외 처리가 불가합니다.
- onCompletion {} : catch{} 연산자와 같이 사용합니다. Flow의 성공 실패 여부를 담당하며 finally {} 구문과 비슷합니다.
- 다음은 try / catch 문을 사용한 예제입니다. 👇
``` kotlin
fun main() {
    runBlocking {
        tryCatch()
    }
}
private suspend fun tryCatch() {
    try {
        (1..3).asFlow().onEach {
            check(it != 2) // 값이 false이면 IllegalStateException을 던집니다. check는 onEach에서 사용하셔야 합니다.
        }.collect { it: Int
            Timber.e("response -> $it")
        }
    } catch (e: Exception) {
        Timber.e("caught exception -> $e")
    }
}
``` 
- 다음은 catch {} 을 사용한 예제입니다.
``` kotlin
fun main() {
    runBlocking {
        catched()
    }
}
private suspend fun catched() {
    (1..3).asFlow().onEach {
        check(it != 2) // 값이 false이면 IllegalStateException을 던집니다. check는 onEach에서 사용하셔야 합니다.
    }.catch { e: Throwable ->
        Timber.e("Caught exception -> $e")
    }.collect { it: Int
        Timber.e("response -> $it")
    }
}
```
- 다음은 onCompletion {} + catch {} 를 사용한 예제입니다.
``` kotlin
fun main() {
    runBlocking {
        onCompletion()
    }
}
private suspend fun onCompletion() {
    (1..3).asFlow().onEach {
        check(it != 2) // 값이 false이면 IllegalStateException을 던집니다. check는 onEach에서 사용하셔야 합니다.
    }.onCompletion { cause: Throwable? ->
        if (cause != null) {
            // TODO: 실패 처리
            Timber.e("실패")
        } else {
            // TODO: 성공 처리
            Timber.e("성공")
        }
    }.catch { e: Throwable ->
        Timber.e("Caught exception -> $e")
    }.collect { it: Int
        Timber.e("response -> $it")
    }
}
```
> **결과**  
response -> 1  
실패  
Caught exception -> java.lang.IllegalStateException: Check failed.  
**처음 1은 2와 다름 따라서 IllegalStateException 발생 이후의 값은 실행되지 않음**

## 마치며
이번 포스팅에서는 Coroutine Flow 와 Flow Operator 에 대하여 알아보았습니다.  
저는 실무에서 map 은 자주 사용중이나 나머지 부분은 잘 사용하지 않습니다.  
Coroutin Flow는 emit 을 통해 값을 보내며 collect를 통해 값을 전달받고 또한 Coroutine Flow는 **_Cold Stream이기에 값을 공유할 수 없다._** 정도만 기억해주시면 되겠습니다.  
Cold Stream 이기에 값을 공유할 수 없다는 말의 의미는 예를 들어 하나의 flow를 2개의 activity나 fragment에서 각각 collect 할때 flow 의 값이 서로 다를 수 있다는 이야기 입니다.  
다음 포스팅에서는 이런 문제를 해결하기 위해 나온 Hot Stream 인 SharedFlow, StateFlow, 또한 나아가 Coroutine Channel 에 대하여 알아보겠습니다.  