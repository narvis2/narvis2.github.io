---
title: Android Jetpack Compose DropDownMenu
author: Narvis2
date: 2022-11-02 15:41:00 +0900
categories: [Android, Jetpack-Compose]
tags: [android, jetpack, compose, snackbar]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `Compose`에서 지원하는 펼쳐지는 메뉴인 `DropDownMenu`에 대하여 간다한 예제로 알아보도록 하겠습니다.

## 🍎 DropDownMenu

- 버튼을 눌렀을 때 선택지를 보여주는 `Menu` item
- 구성요소
  - `expanded` 👉 `DropDownMenu`가 펼쳐졌는지 여부
  - `onDismissRequest` 👉 `DropDownMenu`를 닫으라는 명령이 떨어졌을 때의 Callback
  - `offset` 👉 `DropDownMenu`를 호출하는 `Composable`의 기준점으로부터의 거리(`offset`)을 설정
    > ⚠️**_주의_**⚠️
    >
    > - `DropDownMenu`는 내부에서 `DropDownMenuPositionProvider`에 의해 자동으로 위치가 조정되어 화면 상에 표시됨
  - `properties` 👉 `Back Button`을 눌렀을 때 `DropDownMenu`를 `Dismiss`할 것인지, `DropDownMenu`의 바깥쪽을 눌렀을 때 `Dismiss`할 것인지 등의 `DropDownMenu`의 기본 동작을 정의
  - `content` 👉 `DropDownMenu`안에 들어갈 `Menu Item`을 넣는 공간, `@Composable` 넣기, 람다 형식

### 🍀 DropDownMenuItem

- `onClick` 👉 `Menu Item`을 눌렀을 때 Callback
- `enabled` 👉 `Menu Item`을 클릭 가능하게 할 것인지 여부
- `contentPadding` 👉 `DropDownMenuItem`에 적용할 `Padding` 값
- `interactionSource` 👉 `DropDownMenuItem`과 사용자와의 `Interaction`에 대한 `Event`를 관리.
- `content` 👉 `DropDownMenuItem`에 표기할 요소 관리, `@Composable`넣기, 람다 형식

> **_예제_**
>
> - `DropDownMenu` + `DropDownMenuItem` 을 사용
> - 필수 3가지 요소
>
> 1. `DropDownMenu`가 펼처지는지 제어할 수 있는 변수 필요
> 2. `DropDownMenu`의 펼처짐 상태를 제어하기 위한 버튼 혹은 onClick `Event`를 포함한 `Composable` 필요
> 3. `DropDownMenu` 정의

```kotlin
@Composable
fun ButtonWithDropDownMenu() {
    // 1. DropDownMenu 펼처짐 여부 상태
    val isExpanded = remember { mutableStateOf(false) }

    // 2. DropDownMenu 를 Controller 할 버튼 및 Event 생성
    Button(onClick = {
        isExpanded.value = true
    }) {
        Text(text = "Show Menu")
    }

    // 3. DropDownMenu 정의
    DropdownMenu(
        modifier = Modifier.wrapContentSize(),
        expanded = isExpanded.value,
        offset = DpOffset(0.dp, 12.dp),
        onDismissRequest = { isExpanded.value = false }
    ) {
        // 4. DropDownMenuItem 을 정의
        DropdownMenuItem(onClick = {
            println("Hello")
        }) {
            Text(text = "Print Hello")
        }

        DropdownMenuItem(onClick = {
            println("Compose Dropdown")
        }) {
            Text(text = "Print Compose Dropdown")
        }
    }
}
```
