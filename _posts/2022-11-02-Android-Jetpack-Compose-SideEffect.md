---
title: Android Jetpack Compose Side Effect
author: Narvis2
date: 2022-11-02 11:16:00 +0900
categories: [Android, Compose]
tags: [android, jetpack, compose, side-effect]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `Side-Effect`에 대하여 알아보도록 하겠습니다.

## 🍎 Side Effect

- `Composable` 범위 밖에서 발생하는 앱 상태에 대한 변경
- `Composable`은 각각의 `Lifecycle`을 가지고 있음
- `Composable`은 단방향으로만 `State`를 전달
- `Composable`을 사용할 떄 여러 `Composable`들을 겹쳐서 사용함. 그러면 `System`은 각 `Composable`에 대한 `Lifecycle`을 만들고 `Composable`별로 재구성이 필요할때만 재구성 시킨다.
- **_`Composable`은 기본적으로 바깥쪽 `Composable`이 안쪽 `Composable`로 `State`를 내려줌. 이로 인해 단방향으로만 의존성이 생김_**
  > ⚠️ 하지만, 만약 안쪽에 있는 `Composable`에서 바깥쪽에 있는 `Composable`의 상태에 대한 변경을 준다면??
  > 혹은 `Composable`에서 `Composable` 이 아닌 앱 `State`에 대한 변화를 준다면?? 👇
  >
  > - **_양방향 의존성으로 인해 예측할 수 없는 `Effect`가 생긴다. 이를 `Side Effect`라 부름_**

## 🍎 Side Effect 처리하기

- `LaunchedEffect` 👉 `Composable Lifecycle Scope`에서 `suspend fun`을 실행하기 위해 사용
- `DisposableEffect` 👉 `Composable`이 `Dispose`될 때 정리되어야 할 `Side Effect`를 정의하기 위해 사용
- `SideEffect` 👉 `Composable`의 `State`를 `Compose`에서 관리하지 않는 객체와 공유하기 위해 사용
- **_`Compose`는 위 3가지와 함께 사용할 수 있는 여러 `CoroutineScope`와 `State`관련 함수를 제공_**
  > - `rememberCoroutineScope` 👉 `Composable`의 `CoroutineScope`를 참조하여 외부에서 실행할 수 있도록 해줌
  > - `rememberUpdatedState` 👉 `Launded Effect`는 `Composable`의 `State`가 변경되면 재실행되는데 재실행되지 않아도 되는 `State`를 정의하기 위해 사용
  > - `produceState` 👉 `Compose State`가 아닌 것을 `Compose`의 `State`로 변환
  > - `derivedStateOf` 👉 `State`를 다른 `State`로 변환하기 위해 사용, `Composable`은 변환된 `State`에만 영향을 미침
  > - `snapshotFlow` 👉 `Composable`의 `State`를 `Flow`로 변환

### 🍀 1. Launched Effect

- `Composable` 에서 `Composition`이 일어날 때 `suspend fun`을 실행해주는 `Composable` 임
- ⚠️ `Recomposition`은 `Composable`의 `State`가 바뀔때마다 일어나므로 `Recomposition`이 일어날때마다 이전 `Launched Effect`가 취소되고 다시 수행된다면 매우 비효율적
  > ✅ 이를 해결하기 위해 `LaunchedEffect`는 `key`라 불리는 기준값을 두어 `key`가 바뀔때만 `LaunchedEffect`의 `suspend fun`을 취소하고 재실행함

> **_예제_** 👇 `LaunchedEffect` 에서 **_한번만 실행되어야_** 하는 동작 처리
>
> - 한번만 실행해야 하는 경우 `key`값에 `true`나 `Unit`을 넘겨주는 방향으로 설계

```kotlin
@Composable
fun KotlinWorldScreen(oneTimeEffect: () -> String) {
    LaunchedEffect(true) {
        onTimeEffect()
    }
}
```

> **_예제_** 👇 `LaunchedEffect` 에서 **_한번만 실행되어야_** 하는데 **_동작이 길때_**
>
> - 긴 동작의 람다식을 처리할 때 👉 `rememberUpdatedState` 를 사용하여 `launch`를 기억해야 함

```kotlin
fun KotlinWorldScreen(longTimeJob: suspend () -> String) {
    val rememberLongTimeJob by rememberUpdatedState(longTimeJob)

    LaunchedEffect(true) {
        println(rememberLongTimeJob())
    }
}
```

### 🍀 2. Disposable Effect

- `Composable`이 `Dispose`된 후에 정리해야 할 `Side Effect`가 있는 경우에 사용되는 `Effect`
- **_`Composable`의 `Lifecycle`에 맞춰 정리되어야 하는 `Listener`나 작업이 있는 경우에 `Listener`나 작업을 제거하기 위해 사용되는 `Effect`_**
- ⚠️ `Lifecycle`에 따라 `Side Effect`를 발생시킨 다음 정리되어야 하는 부분이 많을 경우 **_제대로 `Side Effect`에 대한 정리를 하지 않으면 `Memory Leak(메모리 누수)`가 발생할 수 있음_**
- 첫 번쨰 인자 `key` 👉 `key` 값이 바뀔때 마다 `effect` 호출
- 두 번째 인자 `effect`
  - `Effect` 블럭은 처음에는 초기화 로직만 수행하고 이후에는 `key` 값이 바뀔때마다 `onDispose` 블록을 호출한 후 초기화 로직을 다시 호출함
  - `onDispose` 블록의 `return`값이 바로 `DisposableEffect` 여서 `onDispose`블록은 `effect`람다식의 맨 마지막에 무조건 와야함

> **_예제_** 👇

```kotlin
DisposableEffect(key) {
    // Composable 이 제거될 때 Dispose 되어야 하는 효과 초기화
    onDispose {
        // Composable 이 Dispose 될 때 호출되어 Dispose 되어야 하는 효과 제거
    }
}
```

> **_예제_** 👇
>
> - 사용자의 사용 패턴 분석을 위한 로깅 (`Activity` 의 `onStart()`에서 시작되어 `onStop()`에서 끝나야 함)
> - `Lifecycle` 이 바뀔 때 새로운 `Observer`가 `Lifecycle`에 붙어 변화를 구독하고 `Composable`이 제거될 떄 `Observer` 또한 제거되도록 구현

```kotlin
@Composable
fun HomeScreen(
    lifecycleOwner: LifecycleOwner = LocalLifecycleOwner.current,
    _onStartLogging: () -> Unit,
    _onStopLogging: () -> Unit,
) {
    val startLoggingOnStart by rememberUpdatedState(_onStartLogging)
    val stopLoggingOnStop by rememberUpdatedState(_onStopLogging)

    DisposableEffect(lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_START) {
                startLoggingOnStart()
            } else {
                stopLoggingOnStop()
            }
        }

        lifecycleOwner.lifecycle.addObserver(observer)

        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

### 🍀 3. SideEffect

- `Composable`의 `Composition`이 성공적으로 되었을 때 발생하는 `Effect`
- `Compose`에서 관리하지 않는 객체와 `Compose` 내부의 데이터를 공유하기 위해 사용
- ⚠️ `SideEffect` 의 한계점
  - `SideEffect`로 수행하는 `Effect`는 `Composable`이 `Dispose`될 때 정리가 불가능
  - `SideEffect`는 `LaunchedEffect` or `DisposableEffect`로 충분히 대체 가능

> **_예제_** 👇
>
> - `FocusRequester` 의 `requestFocus`는 `Composable`이 아닌 `System`의 `Event`이므로 `Composable` 이 관리하는 `Event` 가 아님
> - 따라서, `Composable`의 구성이 완료된 이후에 `requestFocus`가 호출되도록 보장하려면 `SideEffect`를 사용해야 함

```kotlin
@Composable
fun HomeScreen() {
    var isVisible by remember { mutableStateOf(false) }
    // Composable 이 아닌 System의 Event
    val focusRequester = remember { FocusRequester() }

    Column (modifier = Modifier.fillMaxSize()) {
        Button(onClick = { isVisible = true }) {
            Text(text = "버튼 클릭")
        }

        if (isVisible) {
            OutlinedTextField(
                modifier = Modifier.fillMaxSize()
                                .focusRequester(focusRequester),
                value = "",
                onValueChange={}
            )
        }
    }

    SideEffect {
        if (isVisible) {
            focusRequester.requestFocus()
        }
    }
}
```
