---
title: Android Process 및 Thread
author: Narvis2
date: 2022-07-14 15:20:00 +0900
categories: [Android]
tags: [android, process, thread]
---

이번 포스팅에서는 Android의 Process와 Thread에 대해서 알아보고자 합니다.  
이번에는 설명이 많이 필요하여 지루하실 수 있습니다.  
빠르게 시작해보겠습니다.  

## Process 와 Thread
---
애플리케이션 구성 요소가 시작되고, 앱에 실행중인 다른 구성 요소가 없으면 하나의 실행 Thread로 앱의 Linux Process를 시작합니다.  
기본적으로 같은 애플리케이션의 모든 구성 요소는 같은 Process와 Thread에서 실행됩니다. 하지만 구성 요소가 각자 별도의 Process에서 실행 되도록 할 수 있고, 어느 Process든 추가 Thread를 만들 수 있습니다.

## Process (프로세스)
---
- 기본적으로 같은 앱의 모든 구성 요소는 Process와 Thread에서 실행되고 이를 바꾸면 안됩니다. 그러나 어느 Process가 특성 구성 요소에 속하는지 확인해야 할 경우 AndroidManifest 파일에서 확인 가능합니다.(android:process 속성)  
- AndroidManifest에서 android:process 속성을 설정하여 구성요소가 실행되는 Process를 지정할 수 있습니다.  
  지원하는 요소 : application, activity, service, receiver, provider
- Android System 에서는 어느 Process를 삭제할지 결정할때, 이들의 상대적 중요성을 가늠하여 우선 순위가 낮은 Process부터 삭제합니다. (메모리가 부족할 때 종료해야 하는 Process를 결정하기 위해 Android는 중요도에 따라 Process들을 유형별로 계층 구조에 배치합니다.)

## Process Lifecycle / 중요도가 높은 순으로 정리
---
- 안드로이드 시스템은 Process에서 실행되는 구성요소와 해당 구성요소의 상태에 기초하여 각 Process에 "중요계층"을 보여합니다.  
- 중요도가 낮은 Process가 먼저 제거됩니다.
  
### 1. Foreground Process (포그라운드 프로세스) 
- 사용자가 현재 진행하는 작업에 필요한 Process입니다.(Activity 나 Foreground Service)
- 사용자가 상호작용하는 Activity를 호스팅할 경우
- 사용자와 상호작용하는 Activity에 Bind 된 Service를 호스팅할 경우 (bindService())
- Foreground 에서 실행되는 Service를 호스팅할 경우 (ForegroundService, startForeground())
- Lifecycle Call back 중 하나를 실행하는 Service를 실행할 경우 (onCreate(), onstart() 또는 onDestory())
> 아래의 하나라도 해당되면 Process가 Foreground 에 있는 것으로 간주합니다.  
1. Process가 사용자와 상호작용하고 있는 화면에서 Activity 실행중인 경우
2. Process가 현재 실행중인 BroadcastReceiver가 있는 경우
3. Process에 Call back 중 하나에서 현재 코드를 실행중인 Service 가 있을 경우

### 2. Visible Process (가시적 프로세스)
- Foreground 구성 요소는 없지만 사용자가 화면에서 보는 것에 영향을 미칠 수 있는 Process 입니다.
- Process가 화면상으로는 사용자에게 표시되지만, Foreground에 있지 않은 Activity가 실행 중(onPause() 메서드가 호출된 상태)  
예를 들어 Foreground Activity 대화상자로 표시되고, 이 대화상자에서 Forground Activity 뒤에 이전 Activity가 보이는 때
- Process에 Service.startForground()를 통해 Foreground Service로 실행중인 Service가 있는 경우
- Process가 System에서 라이브 배경화면, 입력 방법 서비스 등과 같이 사용자가 알고 있는 특정 기능에 사용하는 서비스를 호스팅할 경우

### 3. Service Process (서비스 프로세스)
- startService() 메서드로 시작된 Service를 유지하는 Process 입니다.
- 사용자에게 직접 표시되지는 않지만 일반적으로 사용자가 관심을 가진 작업을 실행합니다.
- System은 모든 Foreground Process 및 Visible Process를 유지할 메모리가 부족하지 않다면 항상 이 Process의 실행 상태를 유지합니다.

### 4. Cached Process (빈 프로세스)
- 현재 필요하지 않는 Process 입니다.
- System은 다른 곳에서 메모리가 필요할 때 언제든 원하는 대로 이 Process를 종료합니다.
- 정상적으로 작용하는 System에서 Chaned Process는 메모리 관리와 관련된 유일한 Process 입니다.
- 현재 사용자에게 표시되지 않는 하나 이상의 Activity 인스턴스를 포함합니다.
- Activity Lifecycle 을 올바르게 구현하면 System이 이런 Process를 종료해도 앱으로 돌아갈때 사용자 환경에 영향을 주지 않습니다.
- LRU 목록으로 유지되며, 메모리 회수 시 목록의 마지막 Process가 제일 처음 종료됩니다.

## Thread (쓰레드)
---
- 앱이 시작되면 앱에 대한 실행의 Thread를 생성하며, 이를 Main Thread 혹은 UI Thread라고 합니다.
- 이 Thread는 이벤트를 포함한 UI 위젯에 이벤트를 발송하는 역할을 맡기도하여 UI Thread라고도 합니다. (UI 조작 작업은 UI Thread에서만 이루어져야 합니다.)
- 같은 Process에서 실행되는 모든 구성 요소는 UI Thread에서 시작되고, 구성 요소를 호출하는 System이 해당 Thread에서 발송됩니다.
- **Network Access나 Database Query 등의 긴 작업을 UI Thread에서 수행할 경우 전체 UI 이벤트가 차단되어 사용자에게는 애플리케이션이 중단된 것 처럼 보입니다.(해당 상태를 ANR이라고 합니다.)**
> Android System은 UI Thread가 몇 초 이상 차단되면(현재는 약 5초) 사용자에게 Application Not Response(ANR) 이라는 대화상자가 표시되고, Application이 종료되거나 블만족에 의해 제거될 수 있습니다.
  
- Android Ui Tool kit은 Thread Safe 하지 않기 떄문에 UI를 Worker Thread에서 조작하면 안됩니다.(IOException 발생)

## Worker Thread
---
- UI 반응성을 위해 UI Thread를 차단하지 않는 것이 매우 중요하며, UI Update를 제외하고 시간이 소요되는 작업은(Network, Database) 반드시 별도의 Thread 에서 수행해야 합니다. 이 별도의 Thread를 Background Thread 또는 Worker Thread라고 합니다.
> 다른 Thread 에서 UI Thread에 Access 하기 위해 제공하는 방식
1. Activity.runOnUiThread(Runnable)
2. View.post(Runnable)
3. View.postDelayed(Runnable, long)
4. 위의 방법을 사용하기에 복잡한 작업은 Handler를 사용하여 UI Thread에서 전달받은 메시지를 처리하는 방안을 고려해야 합니다.  
[Handler, Looper 정리](https://narvis2.github.io/posts/Android-handler-looper/) 참조
