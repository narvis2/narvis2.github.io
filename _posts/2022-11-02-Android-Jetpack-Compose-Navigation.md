---
title: Android Jetpack Compose Navigation
author: Narvis2
date: 2022-11-02 13:06:00 +0900
categories: [Android, Jetpack-Compose]
tags: [android, jetpack, compose, navigation]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 `Compose` 의 화면전화에 사용되는 `Compose Navigation`에 대하여 알아보겠습니다.

## 🍎 Compose Navigation

### 🍀 1. Dependency 추가 (build.gradle)

```gradle
implementation "androidx.navigation:navigation-compose:2.5.3"
```

### 🍀 2. rememberNavController

- `navigation` 구성요소의 중심 API
- `Stateful` 이며, 앱의 화면과 각 화면 상태를 구성하는 `Composable Back Stack`을 추적함
- **_모든 `Composable`이 접근할 수 있는 곳에 만들어야 함_** 즉, 최상위 함수

> **_예제_** 👇
>
> - 처음 시작 시 `HomeScreen` 을 보여주고 Click 시 `DetailScreen` 으로 이동
> - `NavGraph` 설정

```kotlin
@Composable
fun MovieNavigation() {
    val navController = rememberNavController()

    // Navigation Graph 를 그림
    NavHost(
        navController = navController,
        startDestination = MovieScreen.HomeScreen.name // 처음 시작화면 설정
    ) {
        composable(route = MovieScreen.HomeScreen.name) {
            HomeScreen(navController = navConttoller)
        }

        // "/{movie}" -> DetailScreen 에서 사용할 파라미터 이름 (데이터를 넘김)
        // 하나의 DetailScreen 을 여러군데에서 재사용하므로 Unique Key 필요
        composable(
            route = MovieScreen.DetailScreen.name + "/{movie}",
            argument = listOf(navArgument(name = "movie") { type = NavType.StringType })
        ) {
            DetailScreen(
                navController = navController,
                it.argument?.getString("movie")
            )
        }
    }
}

enum class MovieScreen {
    HomeScreen,
    DetailScreen
}
```

> **_예제_** 👇
>
> - `Navigation`을 활용하여 `HomeScreen -> DetailScreen 이동`

```kotlin
navController.navigate(route = MovieScreens.DetailScreen.name + "/$보내는 값")
```

> **_예제_** 👇
>
> - `Navigation` 뒤로가기

```kotlin
navController.popBackStack()
```

### 🍀 3. Navigation + Dialog

> **_예제_** 👇 `Custom Dialog` 를 생성하여 `Navigation` 으로 띄우기
>
> - `NavGraph`에 `Custom Dialog` 등록

```kotlin
@Composable
fun NoteNavigation(viewModel: NoteViewModel) {
    val navController = rememberNavController()
    val coroutineScope = rememberCoroutineScope()

    // CustomDialog
    val customDialogTitle = viewModel.customDialogTitle.collectAsState()
    val customDialogConfirmText = viewModel.customDialogConfirmText.collectAsState()
    val customDialogCancelText = viewModel.customDialogCancelText.collectAsState()

    NavHost(
        navController = navController,
        startDestination = NoteNavigation.HomeScreen.name
    ) {
        composable(route = NoteNavigation.HomeScreen.name) {
            NoteScreen(navController = navController)
        }

        dialog(
            route = NoteNavigation.CustomDialog.name,
            dialogProperies = DialogProperties(
                dismissOnBackPress = true,
                dismissOnClickOutside = true,
            )
        ) {
            CustomDialog(
                navController = navController,
                value = customDialogTitle.value.first,
                valueRes= customDialogTitle.value.second,
                confirmText = customDialogConfirmText.value,
                cancelText = customDialogCancelText.value,
            ) {
                coroutineScope.launch {
                    if (customDialogTitle.value.second == R.string.dialog_all_remove_title) {
                        viewModel.removeAllNote()
                        return@launch
                    }

                    viewModel.currentNote.value?.let {
                        if (customDialogTitle.value.second == R.string.dialog_modify_title) {
                            viewModel.updateNote(it)
                            scaffoldState.snackbarHostState.showSnackbar("${it.title}를 수정하였습니다.")
                            return@launch
                        }

                        viewModel.removeNote(it)
                        scaffoldState.snackbarHostState.showSnackbar("${it.title}를 삭제하였습니다.")
                    }
                }
            }
        }
    }
}
```

> **_예제_** 👇
>
> - `Custom Dialog` 생성
> - [참고](https://github.com/narvis2/ComposeTodoApp/blob/73e489e4fb256d9edaa043a342d2bc0104b4a9a6/app/src/main/java/com/example/composetodoapp/presentation/components/CustomDialog.kt)

```kotlin
@Composable
fun CustomDialog(
    navController: NavController,
    value: String,
    @StringRes valueRes: Int? = null,
    @StringRes confirmText: Int,
    @StringRes cancelText: Int,
    onConfirmClick: () -> Unit,
) {
    Surface(
        shape = RoundedCornerShape(16.dp), color = Color.White
    ) {
        Box(contentAlignment = Alignment.Center) {
            Column {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(10.dp),
                    horizontalArrangement = Arrangement.End,
                ) {
                    Icon(imageVector = Icons.Filled.Cancel,
                        contentDescription = "",
                        modifier = Modifier
                            .width(30.dp)
                            .height(30.dp)
                            .clickable { navController.popBackStack() })
                }
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(10.dp),
                    horizontalArrangement = Arrangement.Center,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text(
                        text = if (valueRes != null) {
                            stringResource(id = valueRes, value)
                        } else {
                            value
                        }, style = TextStyle(
                            fontSize = 20.sp,
                            fontFamily = FontFamily.Default,
                            fontWeight = FontWeight.Bold
                        )
                    )
                }

                Spacer(modifier = Modifier.height(20.dp))

                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(20.dp)
                ) {
                    Button(
                        onClick = {
                            navController.popBackStack()
                            /**
                             * navController.currentBackStackEntry?.destination?.route
                             * -> 현재 Navigation Route 이름 가져오기
                             */
                            if (navController.currentBackStackEntry?.destination?.route == NavigationType.DETAIL_SCREEN.name) {
                                navController.popBackStack()
                            }

                            onConfirmClick()
                        },
                        shape = RoundedCornerShape(50.dp),
                        modifier = Modifier
                            .height(50.dp)
                            .padding(end = 10.dp)
                            .weight(1f)
                    ) {
                        Text(text = stringResource(id = confirmText))
                    }
                    Button(
                        onClick = {
                            navController.popBackStack()
                        },
                        shape = RoundedCornerShape(50.dp),
                        modifier = Modifier
                            .height(50.dp)
                            .padding(start = 10.dp)
                            .weight(1f),
                        colors = ButtonDefaults.buttonColors(
                            backgroundColor = Color.White,
                            contentColor = Color.Black,
                        )
                    ) {
                        Text(
                            text = stringResource(id = cancelText),
                            style = TextStyle(color = Color.Black)
                        )
                    }
                }
            }
        }
    }
}
```

## 🍎 Compose BottomSheet Navigation

### 🍀 1. Dependency 추가 (build.gradle)

- `Jetpack Navigation Compose Material` 사용
  > - [공식 홈페이지](https://google.github.io/accompanist/navigation-material/)
  > - [자세한 설명](https://jossiwolf.medium.com/introducing-navigation-material-%EF%B8%8F-a19ed5cc33fd)

```gradle
implementation "com.google.accompanist:accompanist-navigation-material:0.27.0"
```

### 🍀 2. rememberNavController, rememberBottomSheetNavigator

- `rememberBottomSheetNavigator`, `rememberNavController`를 사용하여 `NavGraph` 작성

```kotlin
@Composable
fun MovieNavigation(mainViewModel: MainViewModel) {
    val bottomSheetNavigator = rememberBottomSheetNavigator()
    val navController = rememberNavController(bottomSheetNavigator)

    ModalBottomSheetLayout(
        bottomSheetNavigator = bottomSheetNavigator,
        sheetShape = RoundedCornerShape(topEnd = 16.dp, topStart = 16.dp),
    ) {
        NavHost(
            navController = navController,
            startDestination = NavigationType.HomeScreen.name
        ) {
            composable(route = NavigationType.HomeScreen.name) {
                HomeScreen(navController = navController)
            }

            bottomSheet(route = NavigationType.BottomSheet.name) {
                Text("This is a cool bottom sheet!")
            }
        }
    }
}
```

> **_예제_** 👇 화면 전환

```kotlin
navController.navigate(route = NavigationType.BottomSheet.name)
```

> **_예제_** 👇 뒤로가기

```kotlin
navController.popBackStack()
```

### 🍀 3. Navigating with Arguments

- `bottomSheet` 에 데이터 전달

> **_예제_** 👇

```kotlin
@Composable
fun MyNavigation(mainViewModel: MainViewModel) {
    val navController = rememberNavController()
    val bottomSheetNavigator = rememberBottomSheetNavigator()

    navController.navigatorProvider += bottomSheetNavigator

    ModalBottomSheetLayout(bottomSheetNavigator) {
        NavHost(navController, startDestination = "home") {
            composable(route = "home") {
                Button(onClick = {
                    navController.navigate("sheet?message=hellow_world")
                }) {
                    Text("Click me to see something cool!")
                }
            }

            bottomSheet(route = "sheet?message={message}") { backStackEntry ->
                val message = backStackEntry.arguments?.getString("message")
                Text("This is a cool bottom Sheet")
                if (message != null) {
                    Text("Magic message: $message")
                }

                Button(onClick = { navController.navigate("home") }) {
                    Text("Take me back, please")
                }
            }
        }
    }
}
```
