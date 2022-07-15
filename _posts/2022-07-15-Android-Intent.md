---
title: Android Intent 에 관하여 
author: Narvis2
date: 2022-07-15 14:48:00 +0900
categories: [Android, Intent]
tags: [android, intent]
---

안녕하세요. Narvis2 입니다.
지난번 포스팅에서는 Android 4대 Component 에 대하여 쭉 알아 보았습니다.  
오늘은 Android 4대 Component 간에 통신에 사용되는 Intent에 대하여 알아보도록 하겠습니다.  
Intent는 Application Component 간에 작업 수행을 위한 정보를 전달하는 역할을 합니다. 즉, **_Component 간의 "통신수단"_**이라고 이해하시면 되겠습니다.
Intent에는 명시적 인텐트, 암시적 인텐트가 존재합니다.  
각각 무엇인지 알아보도록 하겠습니다.

## Intent 에 대한 간단 설명
- Android Component 간 통신에 이용됩니다.
- Component에 Action, Data 등을 전달합니다.
- Intent를 통하여 다른 Application의 Component를 활성화 시킬 수 있습니다.
- 가장 많이 사용하는 예로는 Activity간의 화면 전환이 있습니다.

## Explicit Intent(명시적 인텐트)
- 명시적 인텐트는 일반적으로 Activity, Service같은 Component를 시작할 때 사용됩니다.
> **참고** : 실행하고자 하는 Activity 또는 Service의 class name을 사용하는 형태를 의미합니다.
``` kotlin
private startMainActivity() {
    val intent = Intent(this@IntroActivity, MainActivity::class.java)
    startActivity(intent)
}
```

## Implicit Intent(암시적 인텐트)
- 특정 구성 요소의 class name을 알아야 할 필요는 없지만, 수행할 작업을 선언하여 다른 앱의 구성 요소가 이를 처리할 수 있도록 할 때 사용하는 Intent 입니다.
``` kotlin
binding.startGoogleWeb.setOnClickListener {
    // Google 브라우저를 띄워줍니다.
    val intent = Intent(Intent.ACTION_VIEW, Uri.parse("http://www.google.com"))
    startActivity(intent)
}
```