---
title: Android Context
author: Narvis2
date: 2022-07-22 13:29:00 +0900
categories: [Android, Context]
tags: [android, context, applicationContext, activityContext]
---

안녕하세요. Narvis2 입니다.
오늘은 Android의 Context에 대하여 알아보고자 합니다.  
Context는 Application 환경에 대한 글로벌 정보를 갖는 인터페이스 입니다.  
Android에서는 대표적으로 Application의 Context와, Activity의 Context가 존재합니다.  
밑에서 자세히 알아보도록 하겠습니다.

## Context
---
- Application 환경에 대한 글로벌 정보를 갖는 인터페이스입니다.
- Android System에서 구현체를 제공하는 추상 클래스입니다.
- Application 별 Resource 및 클래스 접근에 사용되며, Activity 실행, BroadcastReceiver, Intent 수신과 같은 Application 수준 작업에 사용됩니다.
- Application의 현재 상태를 나타냅니다.
> **_참고_** 👇
> - Context는 Application의 현재 상태의 맥락(Context)를 의미합니다. Context는 새로 생긴 객체가 지금 어떤일이 일어나고 있는지 알 수 있도록 합니다. 따라서 Activity와 Application에 대한 정보를 얻기 위해서는 Context를 사용하면 됩니다.
- Activity Application의 정보를 얻기 위해 사용할 수 있습니다.
- Activity 와 Application은 Context 클래스를 확장한 클래스입니다.
> **_참고_** 👉 Context를 사용하는 경우  
> Resource, Database, SharedPreference 등에 접근하기 위해 사용합니다. 

## Application Context
---
- Singleton 인스턴스입니다. 
- Activity에서 getApplicationContext()를 통해 접근할 수 있습니다.
- Application의 Lifecycle에 묶여 있습니다.
> **_참고_** 👉 현재 Context가 종료된 이후에도 Context가 필요한 작업이나 Activity Scope를 벗어난 Context가 필요한 작업에 적합합니다.
- Singleton Object를 생성하고 해당 Object가 Context가 필요하면 ApplicationContext를 사용합니다.
> **_주의_**❗️ Singleton에 Activity의 Context를 전달할 경우, 해당 Object가 Activity를 항상 참조하므로 Activity가 화면에 표시되지 않는 순간에도 Garbage Collection(가비지 컬렉션)이 진행되지 않아 메모리 누수가 발생할 수 있습니다.
- Application 전체에서 사용할 라이브러리를 특정 Activity에 초기화 한다면 ApplicationContext를 사용합니다.
> **_참고_** 👇
> - Room, SharedPreference, Sinlgeton을 사용할 때는 ApplicationContext를 사용합니다.
> - ContentProvide 의 getContext() 는 Application Context입니다.  
  

## Activity Context
---
- Activity 내에서 유요한 Context입니다.
- Activity의 Lifecycle과 연결되어 있습니다.
- Activity와 함께 소멸해야하는 경우에 사용합니다. 즉, Activity와 Lifecycle이 같은 Object를 생성할 때 사용합니다.
> **_참고_** 👇
> - Toast, Dialog 등의 UI Option에서 Context가 필요한 경우 Activity의 Context를 사용합니다.
> - resources 를 사용하는 경우 Activity의 Context를 사용합니다.