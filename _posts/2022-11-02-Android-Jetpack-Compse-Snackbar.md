---
title: Android Jetpack Compose Snackbar
author: Narvis2
date: 2022-11-02 14:09:00 +0900
categories: [Android, Compose]
tags: [android, jetpack, compose, snackbar]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 Android Jetpack Compose 에서 `snackbar` 를 사용하는 방법에 대하여 간단한 예제를 통해 알아보고자 합니다.

## 🍎 Scaffold 를 사용하여 Snackbar 만들기

- `Compose` 에서 `Snackbar`를 기존 `Snackbar`의 동작대로 이용하기 위해서는 `Scaffold State`로 감싸야 함
  > ⚠️ 주의 ⚠️
  >
  > - 최상단 `Component`를 `Scaffold`로 감싸줘야 `rememberScaffoldState()`를 사용하여 `snackbar` 사용가능
- 만약 `Scaffold`로 감싸지 않으면 보통의 `Composable`과 같이 동작함
- `Coroutine Builder` 내부에서 `Snackbar`가 불려야함
- `rememberScaffoldState()` 👉 `Scaffold`를 사용

> **_예제_** 👇

```kotlin
@Composable
fun NoteScreen(
    navController: NavController,
    scaffoldState: ScaffoldState,
    onActionButtonClick: () -> Unit
) {
    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(text = stringResource(id = R.string.app_name))
                },
                actions = {
                    IconButton(
                        onClick = { navController.navigate(route = NavigationType.SEARCH_SCREEN.name) }
                    ) {
                        Icon(imageVector = Icons.Default.Search, contentDescription = "Search")
                    }
                },
                backgroundColor = Color.White
            )
        },
        scaffoldState = scaffoldState, // state 연결
        floatingActionButton = {
            onActionButtonClick()
        },
        floatingActionButtonPosition = FabPosition.End,
    ) {
        // TODO:: do something...
    }
}

fun NoteNavigation(viewModel: NoteViewModel) {
    val navController = rememberNavController()
    // SnackBar
    val scaffoldState = rememberScaffoldState()
    val coroutineScope = rememberCoroutineScope()

    NavHost(
        navController = navController,
        startDestination = NavigationType.HOME_SCREEN.name
    ) {
        composable (route = NavigationType.HOME_SCREEN.name) {
            NoteScreen(
                navController = navController,
                scaffoldState = scaffoldState,
            ) {
                scaffoldState.snackbarHostState.showSnackbar("Floating Action Button Click")
            }
        }
    }
}
```
