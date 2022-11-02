---
title: Android Jetpack Compose Tab + ViewPager
author: Narvis2
date: 2022-11-02 13:47:00 +0900
categories: [Android, Compose]
tags: [android, jetpack, compose, viewpager, tab]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 Android Jetpack Compose 에서 `Tab + ViewPager` 를 사용하는 방법에 대하여 예제를 통해 알아보고자 합니다.

## 🍎 Tab + ViewPager

- 기존 TabLayout + ViewPager2 를 `Compose` 에서 사용하기

### 🍀 1. Dependency 추가 (build.gradle)

```gradle
implementation "com.google.accompanist:accompanist-pager:0.20.1"
implementation "com.google.accompanist:accompanist-pager-indicators:0.20.1"
```

### 🍀 2. rememberPagerState

- `rememberPagerState` 👉 `TabRow` 와 `Pager` 객체에서 공유할 데이터
  - 현재 몇 페이지에 있는지 저장
- `rememberCoroutineScrope` 👉 `Composable`에서 페이지를 이동하는 동작은 `CoroutineScope`에서 수행되어야 함
- `TabRow` 👉 기존 `TabLayout` 과 같음
  - `selectedTabIndex` 👉 현재 선택된 페이지 Index
  - `indicator` 👉 `indicator` 설정
- `HorizontalPager` 👉 기존 `ViewPager2` 와 같음

```kotlin
@Composable
fun TabViewPagerScreen(
    pages: List<String> = listOf("페이지1", "페이지2", "페이지3")
) {
    val pagerState = rememberPagerState()
    val coroutineScope = rememberCoroutineScope()

    Surface(
        modifier = Modifier.fillMaxSize(),
        color = MaterialTheme.colors.background,
    ) {
        Column(
            modifier = Modifier.fillMaxSize(),
            verticalArrangement = Arrangement.Center,
        ) {
            TabRow(
                selectedTabIndex = pagerState.currentPage,
                indicator = { tabPositions: List<TabPosition> ->
                    TabRowDefaults.Indicator(
                        Modifier.pagerTabIndicatoroffset(pagerState, tabPositions)
                    )
                }
            ) {
                // pages 개수 만큼 Tab 만들기
                pages.forEachIndexed { index, title ->
                    Tab(
                        text = { Text(text = title) },
                        selected = pagerState.currentPosition == index,
                        onClick = {
                            // Tab 클릭 시 해당 페이지로 스크롤링
                            coroutineScope.launch {
                                pagerState.scrollToPage(index)
                            }
                        }
                    )
                }
            }
        }

        // ViewPager2
        HorizontalPager(count = pagers.size, state = pagerState) { page ->
            Text(
                modifier = Modifier.wrapContentSize(),
                text = page.toString(),
                textAlign = TextAlign.Center,
                fontSize = 30.sp
            )
        }
    }
}
```
