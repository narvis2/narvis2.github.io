---
title: Android Compose 기초
author: Narvis2
date: 2022-11-01 16:19:00 +0900
categories: [Android, Compose]
tags: [android, jetpack, compose, surface, modifier, row, column]
---

녕하세요. Narvis2 입니다.  
이번 시간에는 Android jetpack **_Compose_** 에 대하여 알아보겠습니다.

## 🍎 Jetpack Compose

- 선언형 UI, **_특정 상태에 따라 UI가 무엇을 보여주면 되는지_**
- 상태에 따른 UI를 작성하기 위해 훨씬 적은 코드를 사용
- 유지보수 측면에서 유리
- View의 속성등의 구현을 할 경우 경우에 따라 상세하게 작성하지 않아도 되므로 재사용, 확장성에 용이
- `Fragment`를 더 이상 사용하지 않아도 됩니다.

### 🍀 1. **_@Composable_**

- 데이터를 받아서 UI요소로 Emit 하는 함수
- 컴파일러에게 데이터가 UI를 변한하기 위한 함수임을 알림
- 파라미터로 데이터를 받아서 사용할 수 있음
- **_아무것도 return 하지 않는다_**
- 명등원이며, `Side-Effect`가 없음
  > 🍬참고🍬 명등성 👉 연산을 여러번 적용하더라도 결과가 달라지지 않는 성질
- `Composable` 함수는 순서와 관계없이 실행 가능하다.
  > 즉, `@Composable` 이 붙어있는 함수 내부에 다른 `@Composable` 함수 호출이 포함되어 있는 경우 `Composable` 함수는 순서와 관계없이 실행 가능
- 재구성은 최대한 많은 수의 `Composable` 함수 및 람다를 건너뜀
- 재구성은 낙관적이며, 취소가 될 수 있음
- `Composable` 함수는 Animation 의 모든 프레임과 같은 빈도로 매우 자주 실행될 수 있음
- **_데이터 변경 시 데이터가 변경된 위젯만 새로 그려짐_**
- **_`SharedPreference`에서 값을 읽어오는 것과 같이 비용이 많이드는 작업은 `Background Thread`에서 실행하여 결과값을 `Composable` 함수의 파라미터로 전달해야함_**
  > `Composable` 외부에서 `Background Thread`에서 실행하고 `LiveData` 나 `StateFlow` 등을 사용하여 결과 받기

### 🍀 2. Surface

- 요소를 감싸는 `Container` 와 같은 역할을 하는 요소, 배경의 색상을 변경할 수 있음

> **_예제_** 👇

```kotlin
@Composable
fun Greeting(name: String) {
    Surface(color = Purple200) {
        Text(text="Hello $name")
    }
}
```

### 🍀 3. Modifiers

- 구성요소의 크기, 마진등을 변경하거나 클릭이나 스크롤등의 이벤트를 제어할 수 있도록 함
- UI를 꾸미고 상호작용
- `Surface`와 `Text`와 같은 대부분의 `UI Component`의 위치를 지정할 수 있음(위치, 색상, 스타일 수정)
- **_`Modifier`의 순서는 결과에 영향을 주므로 중요_**

> **_예제_** 👇

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text="Hello $name", modifier = Modifier.Padding(2.dp))
}
```

### 🍀 4. Divider

- 수평 구분선 역할

### 🍀 5. Column

- 내부 `Component`들을 **_수평방향_**으로 배치

> **_예제_** 👇

```kotlin
@Composable
fun MyScreenContent(name: List<String> = listOf("안드로이드", "컴포즈")) {
    Column {
        names.forEach {
            Greeting(name = it)
            Divider(color = Color.Black)
        }
    }
}
```

### 🍀 6. Row

- 내부 `Component`들을 **_수직방향_**으로 배치

> **_예제_** 👇

```kotlin
@Composable
fun ComposeRow() {
    Row {
        Text(text= "Text1")
        Text(text= "Text2")
    }
}
```

### 🍀 7. Box

- 구성요소를 다른 구성요소 위에 배치

```kotlin
fun ComposeBox() {
    Box {
        Text(text="Text1")
        Text(text="Text2")
    }
}
```

### 🍀 8. LazyColumn, LazyRow

- `RecyclerView`와 동일한 기능
  > - `LazyColumn` 👉 RecyclerView orientation Vertical
  > - `LazyRow` 👉 RecyclerView orientation Horizontal
- `View`를 재활용하지는 않음, 스크롤할때 새로운 `Composable`을 내보냄
  > **_참고_** 👉 새로운 `Composable`을 내보내는 것이 기존의 방법인 `View`를 `instance`하는 것에 비해 상대적으로 효율적임

> **_예제_** 👇 `LazyColumn`

```kotlin
@Composable
fun MyScreenContent(names: List<String> = List(100) {"안드로이드 $it"}) {
    val counterState = remember { mutableStateOf(0) }

    Column(modifier = Modifier.fillMaxHeight()) {
        NameList(names = names, Modifier.weight(1f))
        Counter(counterState.value) {
            counterState.value = it
        }
    }
}

@Composable
fun NameList(names: List<String>, modifier: Modifier = Modifier) {
    LazyColumn(modifier = modifier) {
        items(items = names) { name ->
            Greeting(name = name)
            Divider(color = Color.Black)
        }
    }
}
```

> **_예제_** 👇 `LazyColumn` `Scroll Controll`
>
> - `rememberLazyListState()` 사용
> - `LazyColum`, `LazyRow` 의 스크롤 위치를 기억
> - 맨위로 끌어당기기, 맨 아래로 넘기기 등에 사용

```kotlin
@Composable
fun SimpleLazyColumn() {
    val listSize = 100
    val scrollState = rememberLazyListState()
    val scrollCoroutineScope = rememberCoroutineScope()

    Column {
        Row {
            Button(onClick = {
                scrollCoroutineScope.launch {
                    scrollState.animateScrollToItem(index = 0)
                }
            }) {
                Text(text = "Scroll to Top")
            }
            Button(onClick = {
                scrollCoroutineScope.launch {
                    scrollState.animateScrollToItem(index = listSize - 1)
                }
            }) {
                Text(text = "Scroll to Bottom")
            }
        }
    }

    LazyColumn(
        state = scrollState,
        contentPadding = PaddingValues(horizantal = 16.dp, vertical = 8.dp),
        verticalArrangment = Arrangement.spaceBy(4.dp) // 각 Item 사이의 간격
    ) {
        items(listSize) { index ->
            ImageListItem(index)
        }
    }
}
```

### 🍀 9. Weight Modifier

- 특정 아이템의 위치를 지정
- 아이템의 `Weight`를 지정하는 방법 👉 전체를 감싸는 `Colum`의 `Modifier.fillMaxHeight()`를 사용해야 함
  > `Modifier.fillMaxHeight()` 👉 `Colum` 이 최대의 높이를 가지도록 함

> **_예제_** 👇

```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "Compose")) {
    val countState = remember { mutableStateOf(0) }

    Column(modifier = Modifier.fillMaxHeight()) {
        Column(modifier = Modifier.weight(1f)) {
            names.forEach {
                Greeting(name = it)
                Divider(color = Color.Black)
            }
        }

        Counter(counterState.value) {
            counterState.value = it
        }
    }
}
```

> **_예제_** 버튼 색상 변경 👇

```kotlin
@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(
        onClick = { updateCount(count + 1) },
        colors = ButtonDefaults.buttonColor(
            backgroundColor = if (count > 5) Color.Cyan else Color.White
        )
    ) {
        Text(text = "$count 번 클릭")
    }
}
```

### 🍀 10. Scaffold

- **_기본으로 제공되는 `Material Component Composable`중 가장 높은 레벨의 `Composable`_**
- `TopAppBar`, `BottomBar`, `SnackBar`, `FloatingActionButton` 및 `Drawer` 용 `slot`이 있음
  > `slot` 👉 여러가지 속성들을 개발자가 정의하여 사용하도록 하기위해 제공하는 `빈공간`
- 뼈대 역할
  > 주로 Main 화면 만들때 사용

> **_예제_** 앱바 + 앱바 오른쪽 끝 Icon 👇

```kotlin
@Composable
fun ScaffoldEx() {
    Scaffold(topBar = {
        TopAppBar(
            title = { Text(text = "앱 바 타이틀") },
            actions = {
                IconButton(onClick = {}) {
                    Icon(Icons.Filled.Favorite, contentDescription = null)
                }
            }
        )
    }) { innerPadding ->
        BodyContent(Modifier.padding(innerPadding).padding(8.dp))
    }
}
```

## 🍎 번외

### 🍀 1. Custom Button

```kotlin
@Composable
fun CreateButton() {
    Button(onClick = {}) {
        Row() {
            Image(painter = painterResource(R.drawable.something), contentDescription = "")
            Text(
                "버튼",
                fontWeight = FontWeight.Bold,
                modifier = Modifier.padding(4.dp).align(Alignment.CenterVerticall)
            )
        }
    }
}
```

### 🍀 2. UI 꾸미기

- padding 으로 클릭 영역 늘리기 👉 `clickable`전에 `padding`을 하는 경우 클릭 영역은 늘러나지 않음

```kotlin
@Composable
fun PhotographerCard(modifier: Modifier = Modifier) {
    Row(
        modifier.padding(8.dp)
            .clip(RoundedCornerShape(4.dp)) // Row 에 코너 추가 (Radius)
            .backgroundColor = MaterialTheme.colors.surface
            .clickable {}
            .padding(16.dp)
    ) {
        // size 가 50인 원 만들기
        Surface(
            modifier = Modifier.size(50.dp),
            shape = CircleShape,
            color = MaterialTheme.colors.onSurface.copy(alpha = 0.2f),
            border = BorderStroke(width = 2.dp, color = Color.LightGray) // 테두리
        ) {
            // TODO :: something view...
        }
    }
}
```
