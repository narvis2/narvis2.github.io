---
title: Android Jetpack Compose Remember, State<T>
author: Narvis2
date: 2022-11-02 10:05:00 +0900
categories: [Android, Compose]
tags: [android, jetpack, compose, remember, state]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 Jetpack Compose 에서 사용되는 `remember` 와 `state` 에 대하여 알아보도록 하겠습니다.

## 🍎 State / Remember

### 🍀 1. State

- 상태 변경에 대응하는 것은 `Compose`의 핵심
- `Compose`앱은 `Composable`함수를 호출하여 데이터를 UI로 변환하는데, **_데이터가 변경되면 새 데이터로 이러한 기능을 호출하여 Update 된 UI를 만든다_**
- `recomposing`
  - `Compose`는 기존의 `Observer` 패턴과 같이 앱 데이터의 변경 사항을 관찰하기 위한 도구를 제공한다.
  - `Compose`는 **_데이터가 변경된 구성 요소만 재구성_** 하고, 영향을 받지 않은 구성을 건너 뛸 수 있도록 개별 구성에 필요한 데이터를 확인한다.
    > ⚠️ **_주의_** ⚠️
    >
    > - `@Composable` 함수는 `recompose`되기 때문에 변수를 함수 내부에서 선언하면 안됨 (함수가 재시작될때마다 초기화되기 때문)
- `Compose` 는 `State` 값이 변경되면 해당 요소가 다시 그려짐(`Recomposing`)
- 하나의 `Component`에서 `State` 변화가 일어나면 해당 `Component`는 `recomposing` 을 거치고 이때 `remember` 키워드를 통해 `state` 를 기억함으로써 성공적으로 UI를 업데이트 한다.

### 🍀 2. Remember

- 이 변수는 `initial composition`에서 메모리에 저장되어, `recompose`때에 값을 반환받아 사용 가능
- `recompose`로 인한 함수의 재호출과 상관없이 변숫 값이 유지될 수 있다.
- `Composition` 이 유지되는 동안에만 적용
  > 즉, `Composition` 이 그려지는 밑바닥의 `activity/fragment`의 `Lifecycle`이 변경되면 `State` 또한 초기화됨
- 해당 `Composable function`이 `composition` 에서 제걸될 때 마다 같이 제거된다.
  > ⚠️ **_주의_** ⚠️
  >
  > - `Configuration change` 발생 시 값이 유지되지 않음. (기기 회전, 다크 모드적용 등등..)
- `rememberSaveable` 👉 `Configuration Change`에도 `State`가 살아 있도록 해줌

### 🍀 3. MutableState<T>

- `mutableStateOf(defaultValue)` 를 사용하여 만듬
- `State<T>` 타입의 변수를 **_Runtime 에 Observing 할 수 있다_**
- `State`의 `value` 가 변경되면 이 값을 읽어가는 `Compose` 들은 `Recompose` 대상이 된다.

> **_예제_** 👇 `MutableState` 객체를 `Composable function` 내부에 선언하는 3가지 방법

```kotlin
val mutableState = remember { mutableStateOf(기본값) }
var value by remember { mutableStateOf(기본값) }
val (value, setValue) = remember { mutableStateOf(기본값) }
```

> **_예제_** 👇

```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("android", "compose")) {
    val counterState = remember { mutableStateOf(0) }

    Column {
        names.forEach {
            Greeting(name = it)
            Divder(color = Color.Black)
        }

        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter(counterState.value) {
            counterState.value = it
        }
    }
}

@Composable
fun Counter(counte: Int, updateCount: (Int) -> Unit) {
    Button(onClick = { updateCount(count + 1) }) {
        Text(text = "$count 번 클릭하셨어요!")
    }
}
```

### 🍀 4. ViewModel 에 State 사용

- `Compose`에서는 `State` 변화시에 해당 `State`와 연결된 `Composable`이 `Recomposition`을 거치기 때문에 `ViewModel`에서 `State`와 이를 `Update`하는 함수를 직접 선언하고 `Composable` 함수 매개변수로 `ViewModel`을 받아 사용
- `LiveData, Flow` -> `State` 변환
  - `LiveData` 👉 LiveData<T>.observeAsState()
  - `Flow` 👉 Flow<T>.collectAsState()

> **_예제_** 👇

```kotlin
class TodoViewModel : ViewModel() {
    var todoItems = mutableStateListOf<TodoItem>()
        private set

    fun addItem(item: TodoItem) {
        todoItems.add(item)
    }

    fun removeItem(item: TodoItem) {
        todoItems.remove(item)
    }
}
```

> **_예제_** 👇 위 `ViewModel` 사용
> ✅ state holder 설정
>
> - 만약 실제 UI를 그리는 TodoScreen 에 직접 ViewModel 을 전달하면 `Preview` 기능을 사용하기 어려워지고 테스트 가능성이 떨어짐
> - 따라서, UI 를 직접 그리는 `Composable`에는 **_ViewModel 을 직접 주입하지 않고_** `State Holder` 를 담당하는 `Composable` 을 한 단계 거치도록 함

```kotlin
// MyScreen -> state holder 를 담당하는 @Composable
@Composable
fun MyScreen(todoViewModel: TodoViewModel) {
    TodoScreen(
        items = todoViewModel.todoItems,
        onAddItem = todoViewModel::addItem,
        onRemoveItem = todoViewModel::removeItem,
    )
}
```

> **_예제_** 👇 `Flow<T>.collectAsState()` 사용
>
> - ViewModel 에서 `StateFlow` 등록 후 `.collectAsState()`를 사용 시 `Configuration Change` 발생 시에도 값을 유지할 수 있음

```kotlin
class NoteViewModel : ViewModel() {
    private val _searchValue = MutableStateFlow("")
    val searchValue = _searchValue.asStateFlow()

    fun setSearchValue(query: String) {
        _searchValue.value = query
    }
}

@Composable
fun NoteNavigation(viewModel: NoteViewModel) {
    val navController = rememberNavController()

    val searchValue = viewModel.searchValue.collectAsState()

    // TODO:: something do..
}
```

## 🍎 Stateless 를 위한 State Hoisting

- `stateful` - `state`를 갖고 있으며 이를 직접 변경할 수 없는 `Composable`
- `stateless` - `state`를 갖고있지 않고, `state`를 선언하는 것이 아닌 주입받는 방식
- `state hoisting`
  - 개발자는 최대한 `stateful composable`을 줄이고, 이들을 `stateless`로 변경하는 것이 이상적
  - 하위의 `Composable`에 선언된 `State`들을 이들의 공통 조상인 상위 `Composable`로 옮기는 방식으로 수행
  - 장점
    > - `state`를 한 곳에서만 관리함으로써 버그 방지에 도움이됨
    > - `Hoisting` 한 `State`를 여러 `Composable`과 공유할 수 있음

### 🍀 State 를 어느 수준의 Composable 까지 끌어올려야 할지 쉽게 파악할 수 있는 규칙 3가지

1. 읽기 👉 `State`는 적어도 이를 사용하는 모든 `Composable`의 가장 낮은 공통 상위 요소로 끌어 올려야함
2. 쓰기 👉 `State`는 최소한 이를 변경할 수 있는 `Composable`중 가장 상위 `Composable`로 끌어올려야 함
3. 동일한 `Event`에 대한 응답으로 두 `State`가 변경되는 경우 두 `State`를 같이 끌어 올려야함

> **_예제_** 👇 Button을 통해 Component를 숨기고 표시

```kotlin
@Composable
fun MyScreen() {
    val visible = remember { mutableStateOf(true) }

    Column() {
        if (visible.value) MyCard()
        MyButton(onClick = { visible.value = !visible.value }, visible = visible.value)
    }
}

@Composable
fun MyButton(onClic: () -> Unit, visible: Boolean) {
    Button(onClick = onClick) {
        Text(if(visible) "show" else "hide")
    }
}
```

> **_🍬 참고 🍬_**
>
> - MyButton에 `State`를 선언할 필요없이, 적당한 상단 `Composable`에 `State`를 선언한 뒤 `State`의 `Value`나 `State`를 변경하는 함수만 전달해주면 `stateless`를 유지한 채 구현할 수 있음

## 🍎 Composable 에서 올바른 CoroutinScope

- `Composable` 내부에서 `Coroutine`을 수행할 경우 `Composable`에 대한 `Recomposition`이 일어날 때 정리되어야 하는 `Coroutine`이 정리가 안된 상태로 계속해서 `Coroutine`이 쌓일 수 있음
- `Composable` 에서 `Coroutine`을 생성한다면 `Recomposition`이 일어날 때 취소되어야 함
- `rememberCoroutineScope`
  - `Composable`의 `Lifecycle`을 따르는 `CoroutineScope` 를 반환
  - `Composable` 이 파괴되면 자동으로 `Coroutine Job` 파괴
  - `Composable` 이 파괴될 때 파괴되는 `Coroutine`을 생성해야될 때에는 `rememberCoroutineScope` 사용
