---
title: React-Native Dimensions Window / Screen 차이점
author: Narvis2
date: 2022-12-19 17:30:00 +0900
categories: [React-Native, View]
tags: [react-native, dimensions, window, screen]
---

안녕하세요. Narvis2 입니다.  
이번 시간에는 `React-Native`에서 모바일 기기의 해상도를 가져올 수 있는 `API`인 `Dimensions`에 대하여 알아보도록 하겠습니다.

## 🍀 Window / Screen 차이점

---

- `Dimensions`는 기본적으로 현재 기기의 화면 크기를 알기위해 사용
- **_`iOS`에서는 `window`와 `screen`둘 중 어떤 것을 쓰더라도 동일_** 하게 작동
- **_`Android`에서는 최상단 `statusBar`에서 차이가 남_** 👇

![Desktop View](/assets/img/reactnative/dimension.png){: width="100%" }

### ☘️ Window

- 위에 보이는 그림 1의 **_빨간색으로 표시된(`statusBar`) 부분을 `포함하지 않고`_** 영역을 추출

```javascript
const width = Dimensions.get("window").width;
```

### ☘️ Screen

- 위에 보이는 그림 1의 **_빨간색으로 표시된(`statusBar`) 부분을 `포함하고`_** 영역을 추출

```javascript
const height = Dimensions.get("screen").height;
```
