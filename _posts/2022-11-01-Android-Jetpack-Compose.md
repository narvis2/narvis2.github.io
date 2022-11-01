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
