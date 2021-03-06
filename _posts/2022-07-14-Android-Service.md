---
title: Android 4대 컴포넌트 중 Service 에 대하여 
author: Narvis2
date: 2022-07-14 16:56:00 +0900
categories: [Android, 4 Component]
tags: [android, service, bind, foregorund]
---

안녕하세요. Narvis2 입니다.  
오늘은 Android 4대 Component(Activity, ContentProvider, BroadcastReceiver, Service) 중 하나인 Service에 대하여 알아볼까 합니다.  
안드로이드 Document에 따르면 Service의 주된 목적은 오래 걸리는 작업을 백그라운드에서 처리하라고 적혀있습니다.  
저는 현 직장에서 실시간 채팅 기능을 위한 Socket 통신에 Bind Service를 사용하고 있습니다.  
Service의 종류로는 Background, Bound, Foreground가 있으며 Background Service부터 알아보겠습니다. 

## Background Service
---
- 사용자에게 직접 보이지 않는 작업을 수행할때 사용합니다. 
- 작업을 수행하고 결과를 호출자에게 반환하지 않습니다.
- Application을 종료해도 Background Service는 계속 수행할 수 있습니다.
- 압축과 같은 작업을 할때 사용하며 Service가 Background 에서 장시간 실행 중이면 소멸 가능성이 높습니다.
> **_주의!!_** [API 26 부터는 Background Service에 대하여 여러가지 제약이 걸렸습니다.](https://developer.android.com/about/versions/oreo/background)  

## Bound Service (Bind Service)
---
- 바인딩을 위한 Service 입니다. 
- Service는 모든 바인딩이 해제되면 System에 의해 소멸됩니다.
- Activity / Fragment 와 Service 간에 데이터를 주고 받을 수 있고 Process간 통신에도 사용됩니다.
> Activity / Fragment -> Service 간의 통신에 보통 다음 3가지를 많이 사용합니다.  
1. BroadcastReciever
2. Handler
3. AIDL (AIDL 은 BroadcastReciever 보다 보안상 우위에 있습니다.)  
AIDL 에 관한 내용은 향후 따로 포스팅을 작성하도록 하겠습니다.

## Foreground Service
---
- 현재 서비스의 동작이나 상황을 유저에게 계속 알려주어야 하는 상황에서 유용합니다.
- 사용자가 다른 앱을 사용 중이라도 계속해서 실행 되도록 합니다.
- 소멸 가능성이 희박하며, **_notifictaion 알림을 10초안에 띄워야 합니다._** (그렇지 않으면 Android에서 Service를 죽입니다.)

## Service Lifecycle
---
- startService() 의 경우 -> onCreate() -> onStartCommand() -> Service가 스스로 혹은 Client에 의해 중단됨 -> onDestory()
- bindService() 의 경우 -> onCreate() -> onBind() -> onUnbind() -> onDestory()  
### 1. onCreate()
  - Service가 생성될때 최초로 호출되는 메서드 입니다.
  - 해당 메서드에서 초기 설정을 수행하시면 됩니다.
  - Service가 이미 실행중이면 onCreate()는 호출되지 않습니다.
  
  ### 2. onStartCommand()
  - startService()로 Service가 시작될 때 호출됩니다.
  - **startService()를 통해 전달한 Intent를 onStartCommand()에서 Service가 수신하게 됩니다.**
  - 작업이 완료되면 stopSelf()를 호출하여 스스로 중단 하거나, 다른 컴포넌트가 stopService()를 호출하여 중단 시킬 수 있습니다.
  - Binding만 제공하고자 하는 경우에는 onStartCommand()를 구현하지 않아도 됩니다.
  - onStartCommand() 호출을 한번이라도 받은 Service는 Lifecycle을 직접 관리 및 중단해야 합니다.
  > onStartCommand() 의 Return 값 -> Service가 System 에 의해 소멸된 경우, Service가 다시 시작할지 여부를 결정합니다.
  > - START_REDELIVER_INTENT : System이 Service를 중단하면 Service를 다시 생성합니다. 파일 다운로드와 같은 Service에 적합합니다.
  > - START_STICKY : 시스템이 Service를 중단하면 Service를 다시 생성합니다.
                     마지막 Intent를 전달하지 않고, null Intent로 onStartCommand()를 호출합니다.
                     명령을 실행하진 않지만 작업을 기다리는 Media Player와 같은 Service에 적합합니다.
  > - START_NOT_STICKY : System이 Service를 중단하면 Service를 재생성하지 않습니다.
                         다시 시작하려는 경우에 적합합니다.

  ### 3. onBind()
  - 다른 구성 요소(Activity/Fragment)와 Service를 바인딩 하려는 경우에 사용합니다. 즉, Service 와 Activity / Fragment 사이에서 데이터를 주고 받을 때 사용합니다.
  - bindService()로 시작합니다.
  - 구현할때 Interface를 제공해야 합니다. reutrn으로 IBinder를 반환하면 됩니다.
  > - 다른 구성요소가 Service에 Bind 하고자 하는 경우, Client 가 Service와 통신을 주고받기 위해 사용할 인터페이스를 제공해야 합니다.
  > - AIDL 사용시 Stub() 구현체를 Return 하면 된다.
  > - Handler 사용시 Messanger(Handler(mainLopper, this)).binder 를 반환합니다.
  - **이 메서드는 항상 구현해야 하며, 사용하지 않을 때는 return null 을 하시면 됩니다.**

  ### 4. onUnBind()
  - 모든 Client 들이 unBindService()를 호출하여 binding이 해지되었을 때 호출됩니다.

  ### 5. onReBind()
  - unBindService() 가 호출된 이후에, Client가 bindService()를 호출하여 Service에 binding 될 때 호출됩니다.
  
  ### 6. Destory()
  - Service가 해지될 때 호출 됩니다. Resource 해지를 수행합니다.

## 번외 - PendingIntent
- Notification 으로 작업을 수행할 때 Intent가 실행되도록 합니다.(Notification 으로 Intent 수행 시 PendingIntent 사용이 필수)
> **참고** 
> - 런처 바탕화면의 위젯으로 Intent 작업을 수행할때 PendingIntent를 사용합니다.  
> - Alarm Manager를 통해 지정된 시간에 Intent가 실행되도록 할때 PendingIntent를 사용합니다.  
- 
> **생성**
> - Activity를 시작하는 Intent -> PendingIntent.getActivity(context: Context, requestCode: Int, intent: Intent, flags: Int)
> - Service를 시작하는 Intent -> PendingIntent.getService(context: Context, requestCode: Int, intent: Intent, flags: Int)
> - BroadcastReciver를 시작하는 Intent -> PendingIntent.getBroadcast(context: Context, requestCode: Int, intent: Intent, flags: Int)  
-  
> **Flag**
> - FLAG_UPDATE_CURRENT : PendingIntent가 이미 존재하는 경우, Extra Data를 새로운 Intent에 있는 것으로 대체합니다.
> - FLAG_CANCEL_CURRENT : PendingIntent가 이미 존재하는 경우, 기존 PendingIntent를 Cancel 하고 다시 생성합니다.
> - FLAG_IMMUTABLE : 기존 PendingIntent는 변경되지 않고, 새로운 데이터를 추가한 PendingIntent를 보내도 무시합니다.
> - FLAG_NO_CREATE : PendingIntent가 기존에 존재하지 않으면 Null을 반환합니다.
> - FLAG_ONE_SHOT : 한번만 사용할 수 있는 PendingIntent
``` kotlin
    fun makePendingIntent(): PendingIntent {
        // RequestCode, Id를 고유값으로 지정하여 알람이 개별 표시되도록 설정
        val uniId: Int = (System.currentTimeMillis() / 1000).toInt()
        val intent = Intent(context, MainActivity::class.java)
        return PendingIntent.getActivity(
            context, 
            uniId, 
            intent,
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT
            } else {
                PendingIntent.FLAG_UPDATE_CURRENT
            }
        )   
    }
```


## 마무리
이번 시간에는 Service에 대하여 알아보았습니다.  
저는 Service를 Socket 연결을 위해 Bind Service를 주로 사용하였고, AIDL 을 통해 Process 통신을 했습니다.  
AIDL에 대한 설명은 추후 따로 포스팅을 올리겠습니다.  
다음 포스팅은 Android 4대 컴포넌트 중 하나인 BroadcastReceiver 에 대하여 알아보는 시간을 가지도록 하겠습니다.  

Android 4대 Component
- [BroadcastReceiver](https://narvis2.github.io/posts/Android-BroadcastReceiver/) 참고
- [Activity의 Lifecycle](https://narvis2.github.io/posts/Android-Activity-Lifecycle/) 참고