---
title: Android Compose Animation 기초
author: Narvis2
date: 2022-11-02 09:19:00 +0900
categories: [Android, Jetpack-Compose]
tags: [android, jetpack, compose, animation, alpha]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `Compose`에서 Animation 을 사용하는 법에 대하여 알아보고자 합니다.  
`Animation`에 대한 이해가 아직 부족하여 보다 자세히 알고 싶으시면 밑의 링크를 참고하세요.  
[Simple Animation](https://origogi.github.io/android/compose-animation/)  
[Deep Animation](https://sungbin.land/jetpack-compose-%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98-%EB%A7%88%EC%8A%A4%ED%84%B0-efbe5b074e03)

### 🍎 1. animateColorAsState

- UI 변경시의 `Animation` 사용
  > `animatedColorAsState(Color 값 넣기)`

> **_예제_** `animateColorAsState` 를 사용하여 텍스트가 클릭된 상태면 빨간색, 아니라면 투명색 배경 지정 👇

```kotlin
@Composable
fun Greeting(name: String) {
    var isSelected by remember { mutableStateOf(false) }

    val backgroundColor by animatedColorAsState(if (isSelected) Color.Red else Color.Transparent)

    Text(
        text = "안녕하세요 $name",
        modifier = Modifier
            .padding(24.dp)
            .backgroundColor(color = backgroundColor)
            .clickable(onClick = { isSelected = !isSelected })
    )
}
```

### 🍎 2. CompositionLocalProvider

- `alpha` 값을 설정하기 위해 사용
- 해당 블럭안에 있는 모든 하위 계층의 `Composable` 들에게 영향을 줌
- `LocalContentAlpha`
  - `Material Theme Level` 에서 사용하는 `Text`, `Icon` 을 위한 `alpha` 값
  - 기본값 👉 1f, 투명도가 없이 보여짐

> **_예제_** 👇

```kotlin
@Composable
fun PhotoGrapherCard() {
    Coloumn {
        Text(text = "안드로이드", fontWeight = FontWeight.Bold)
        CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
            Text(text = "3 minutes ago", style = MaterialTheme.typography.body2)
        }
    }
}
```

### 🍎 3. Animatable

- `animateTo` 를 통해 **_값이 변경될 때 자동으로 값에 Animation 을 적용_**하는 값 홀더
- 첫 번째 인자 `initialValue` 👉 초기 값
- 두 번째 인자 `typeConverter` 👉 Animation 객체에 사용할 AnimationVector
- 세 번째 인자 `visibilityThreshold` 👉 가시성 임계값
- 내부 필드
  - `value` 👉 Animation 의 현재 값
  - `targetValue` 👉 현재 Animation 의 대상
  - `animateTo` 👉 `targetValue` 값을 향해 Animation 을 시작
    - `animateTo` 인자
      > - `targetValue`, `animationSpec`, `initialVelocity`, `block` 이 있음
      > - `targetValue` 👉 Animation 도달 값, 이 값이 작을 수록 Animation 이 작아짐
      > - `initialVelocity` 👉 Animation 의 시작 값을 의미
      > - `block` 👉 매 Animation 프레임 마다 호출됨

> **_예제_** Splash 화면👇
> 원이 커졌다가 작아지는 Animation, Animation 이 끝나면 다음 화면으로 진입

```kotlin
@Composable
fun WeatherSplashScreen(navController: NavController) {
    val scale = remember { Animatable(initialValue = 0f) }

    // LaunchedEffect : key1 이 바뀔때마다 블록내의 동작을 중지하고 다시 시작함
    // LaunchedEffect 에서 한 번만 실행되어야 하는 동작을 처리하기 위해 true를 넣음
    LaunchedEffect(key1 = true) {
        scale.animateTo(
            targetValue = 0.9f, // 이 값이 작을 수록 Animation 이 작아짐
            animationSpec = tween(
                durationMillis = 800, // 이 값이 커질수록 Animation 이 늦게 끝남
                easing = {
                    OvershootInterpolator(8f).getInterpolation(it)
                }
            )
        )

        // Animation 이 끝나고 2초 뒤 MainScreen 으로 이동
        delay(2000L)
        navController.navigate(WeatherScreens.MainScreen.name)
    }

    Surface(
        modifier = Modifier
            .padding(15.dp)
            .size(330.dp)
            .scale(scale.value),
        shape = CircleShape,
        color = Color.White,
        border = BorderStroke(width = 2.dp, color = Color.LightGray)
    ) {
        Column(
            modifier = Modifier.padding(1.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Image(
                painter = painterResource(id = R.drawable.sun),
                contentDescription = "sunny icon",
                contentScale = ContentScale.Fit,
                modifier = Modifier.size(95.dp)
            )
            Text(
                text = "Find the Sun?", style = MaterialTheme.typography.h5, color = Color.LightGray
            )
        }
    }
}
```
