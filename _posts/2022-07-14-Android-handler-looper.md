---
title: Android Handler, Looper 에 관하여
author: Narvis2
date: 2022-07-14 11:44:00 +0900
categories: [Android]
tags: [android, thread, handler, looper]
---

이번에는 Handler, Looper 에 대해서 알아보도록 하겠습니다.  
Android의 UI 처리는 Single Thread Model로 동작합니다. 즉, mainThread가 아닌 다른 Thread에서 UI 를 Update하는 등의 행위를 하면 안됩니다. 따라서 mainThread를 UI Thread라고 부르기도 합니다.  
동작의 무결성을 보장하기 위해 타 Thread에서는 UI를 건드릴 수 없고, 오로지 Main Thread에서만 UI 관련 동작을 할 수 있게끔 합니다.(MainThread를 제외한 다른 Thread에서 UI Update시 **_IOException_**이 발생합니다.)  
이때 시간이 오래걸리는 작업을 MainThread에서 하게되면 UI가 멈추게 되는데(ANR) 이를 방지하기 위해 시간이 오래걸리는 작업은 다른 Thread에서 하고, 해당 결과를 Main Thread에 넘겨줘서 Main Thread에서 해당 결과 값을 바탕으로 UI를 Update합니다.  
> Android에서는 UI Thread가 몇 초 이상 차단되면(현재는 약 5초) 사용자에게 Application Not Response(ANR) 이라는 대화 상자가 표시되고, 어플리케이션이 종료 되거나 불만족에 의해 제거 될 수 있습니다.  
  
여기서 다른 Thread -> Ui Thread 간에 결과값을 넘겨줄 때 통신을 위해서 즉, Thread 간의 통신을 위해 사용하는 것이 Handler 와 Looper 입니다. 
> UI 반응성을 위해 UI Thread를 차단하지 않는 것이 매우 중요하며, UI Update를 제외하고 시간이 소요되는 작업은 반드시 별도의 Thread에서 수행해야 하는데 이 별도의 Thread를 Background Thread 또는 Worker Thread라고 합니다.  
   
이번에도 실용적인 측면에 초점을 두고 다루겠습니다.  
  
## 간단한 용어 설명 Thread, Main Thread
---
Thread에 대해 간단히 설명드리면 "하나의 독립된 실행 흐름" 이라고 생각하시면 됩니다.  
mainThread는 하나의 프로그램이 실행될때 최초로 실행되는 Thread라고 생각하시면 됩니다.

## 다른 Thread -> UI Thread에 Access 하기 위해 제공하는 방식
---
1. Activity.runOnUiThread(Runnable)
2. View.post(Runnable)
3. View.postDelayed(Runnable, long)
4. 위의 방법을 사용하기에 복잡한 작업은 Handler를 사용하여 UI Thread에서 전달받은 Message를 처리하는 방안을 고려해야 합니다.

## Looper
---
1. 하나의 Thread 당 단 하나의 Looper만 가질 수 있습니다.
2. Android에선 기본적으로 MainActivity가 실행됨과 동시에 자동으로 Main Thread의 Looper가 돌기 시작합니다.
3. 각 Thread의 Looper 내부에는 MessageQueue 라는 것이 존재하는데, 여기에는 해당 Thread가 처리해야 할 동작들이 **_'Message'_** 라는 형태로 하나씩 쌓이게 됩니다. (큐 라는 이름에서 알 수 있듯 당연히 FIFO 방식이며 **_선입선출_** 방식으로 처리됩니다.)
> **_Queue_**에 대해 모르시면 [자료구조 Stack, Queue 정리](https://narvis2.github.io/posts/Data-Structure-Stack-Queue/) 포스팅을 참고 해주세요.
4. MessageQueue가 비어있을 때는 아무런 동작도 하지 않다가 Message가 들어오면 Message를 꺼내 적절한 **_Handler_**로 전달합니다.
5. Thread에서 무한히 돌면서 자신이 속한 Thread의 MessageQueue에 Message나 Runnable이 들어오면 차례대로(선입선출) Handler에 전달하는 역할을 합니다.

## Message
---
1. **_"하나의 작은 작업 단위"_**, **MessageQueue**에는 이러한 작은 작업 단위를 하나씩 적재해두고, **_"Looper"_**가 이를 차례대로 처리한다.
2. **_Thread간 통신 할 내용을 담는 객체이자 MessageQueue의 일감 단위라고 생각하시면 됩니다._**
3. **Message**는 **_Runnable_**, **_"Message"_** 2가지가 있습니다. Looper 객체가 MessageQueue에서 Message를 확인했을 때 Runnable 객체가 담겨져 있으면 Handler에 Message를 전달하지 않고 run()함수를 실행하여 해당 Runnable 작업을 바로 시작하고, Runnable 객체가 없을 경우 Message 객체 내부에 있는 Handler의 handleMessage()를 수행하여 처리합니다.
4. **_Runnable_**은 Thread를 생성할때 생기는 run() 메서드를 따로 분리시킨 형태로 run() 추상 메서드를 반드시 구현해야합니다. run()함수 안에 실행될 동작이 들어갑니다.

## Handler
---
1. **_Looper_**로 부터 전달 받은 **_Message_** 나 **_Runnable_**을 **_handleMessage()_**를 통해 처리하거나 다른 **_Thread_**로 부터 받은 **_Message_**를 **_MessageQueue_**에 넣습니다.
2. **_Handler_**를 생성하게되면 호출한 **_Thread_**의 **_MessageQueue_**와 **_Looper_**에 자동으로 연결됩니다.
3. Message 객체를 생성하여 이를 전달하는 방식으로 구현합니다.
4. sendMessage() 메소드를 통해 MessageQueue에 Message 객체를 적재할 수 있습니다.
5. post로 시작하는 메소드들을 통해 Runnable 객체를 직접 적재할 수 있습니다.
6. UI를 갱신하기 위해서 사용합니다.
> **주의!!** Handler를 상속받아서 처리하는 모든 부분에서 UI가 Destory 즉, 파괴될때 handler.removeCallbacksAndMessages(null)을 호출해줘야 합니다.

## 동작 방식
---
1. Handler 의 sendMessage()를 통해 MessageQueue에 Message를 차례대로 넣습니다.
2. Looper는 무한히 돌면서 자신이 속한 Thread의 MessageQueue에 Message나 Runnable이 들어오면 차례대로 하나씩 MessageQueue에서 Message를 꺼내 적절한 Handler에게 전달합니다.
3. Looper로 부터 Message를 전달받은 Handler는 handleMessage() 메서드를 통해 Message를 처리합니다.

## 코드를 통해 살펴보기
---
간다한 코드를 통해 살펴 보겠습니다.  
1. Handler 생성 (Message를 받는 쪽) 코드 부터 살펴보겠습니다.
``` kotlin
class HomeHandler(fragment: HomeFragment) : Handler(Looper.getMainLooper()) {
    private val weekReference: WeakReference(fragment)

    override fun handleMessage(msg: Message) {
        val fragment = weakReference.get() as HomeFragment

        // HomeFragment 의 함수를 사용하기 위해 apply 사용 (연속적으로 접근 할때는 apply, with 등을 사용하는 것이 좋습니다.)
        fragment.apply { this: HomeFragment
            when (msg.what) {
                FINISH_MAIN_ACTIVITY -> {
                    // this 는 생략 가능 여기서는 알아보기 쉽게 명시적으로 선언
                    // 여기서는 간단하게 부모 Activity를 종료 시키도록 하였습니다.
                    this.finishActivity()
                }

                SHOW_SIMPLE_TOAST -> {
                    // this 는 생략 가능 여기서는 알아보기 쉽게 명시적으로 선언
                    this.showToast(msg.obj?.toString())
                }
                // 등등..
            }
        }
    }

    companion object {
        // 해당 변수로 Message를 구분합니다.
        const val FINISH_MAIN_ACTIVITY = 0
        const val SHOW_SIMPLE_TOAST = 1
        // 등등..
    }
}
```
> GC(Garbage Collector)는 "내가 다시 사용할 수 있냐, 없냐를 보고 없으면 쓰레기통에 버린다" 라고 생각하시면 편합니다.  
  WeakReference : GC(Garbage Collector)가 발생되기 전까지는 참조를 유지하고 GC가 발생되면 무조건 회수
                  (짧은 시간, 자주 쓰일 수 있는 객체를 사용할때 유용하게 사용될 수 있음.) 

2. Message를 보내는 쪽 코드 (sendMessage()를 통해 Message 넣기)
``` kotlin
private val homeHandler = HomeHandler(this)
// 함수 이름은 대충 지었습니다.
fun sendMainActivityFinish() {
    homeHandler.obtainMessage().apply { this: Message
        what = HomeHandler.FINISH_MAIN_ACTIVITY
        obj = "메인 엑티비티 종료" // 전달하고자 하는 객체 넣기 (여기서는 간단하게 String을 넣어줬습니다.)
        homeHandler.sendMessage(this)
    }
}
fun sendShowToast() {
    homeHandler.obtainMessage().apply { this: Message
        what = HomeHandler.SHOW_SIMPLE_TOAST
        obj = "안녕하세용" // 전달하고자 하는 객체 넣기 (여기서는 간단하게 String을 넣어줬습니다.)
        homeHandler.sendMessage(this)
    }
}
```
> sendMessage()를 통해 Message를 MessageQueue에 넣습니다. 이렇게 되면 Handler는 Looper 로 Message를 전달하고 Looper는 MessageQueue로 부터 해당 Message를 꺼냅니다. 이때 위의 코드에서는 Runnable이 없기 때문에 HomeHandler에게 Message를 전달합니다. Looper로 부터 Message를 전달받은 HomeHandler는 handleMessage()를 통해 Message를 처리합니다.  
**_참고_** <u>obtainMessage()</u> : obtainMessage()는 Message 인스턴스를 생성합니다. 다만 정적으로 생성된 재사용 객체로 관리되기 때문에 new Message()로 인스턴스를 생성하는 것 보다 효율적 입니다.

3. HomeFragment 의 handleMessage() 함수 내부에서 실행되는 HomeFragment의 함수
``` kotlin
    // HomeHandler 의 handleMessage() 에서 FINISH_MAIN_ACTIVITY 일때 호출하는 함수
    fun activityFinish() {
        activity.finish()
    }
    // HomeHandler 의 handleMessage() 에서 SHOW_SIMPLE_TOAST 일때 호출하는 함수
    fun showToast(msg: String) {
        Toast.makeText(requireContext(), msg, Toast.LENGTH_SHORT).show()
    }
```

4. UI 가 Destory 될 때 handler 제거하는 코드
``` kotlin
    override fun onDestoryView() {
        castHandler.removeCallbacksAndMessages(null)
        super.onDestoryView()
    }
```
> Activity의 경우 onDestory() 에서 하시면 됩니다.  
**주의!!** UI 가 Destory 될 때 remove 해주지 않으면 IllegalStateException이 발생할 수 있습니다.

5. Handler가 해당 Message를 가지고 있는지 확인하고 존재하면 제거하는 코드
``` kotlin
    private fun removeCheckShowToast() {
        if (castHandler.hasMessages(HomeHandler.SHOW_SIMPLE_TOAST)) {
            castHandler.removeMessages(HomeHandler.SHOW_SIMPLE_TOAST)
        }
    }
```

## Handler 에 Message를 보내는 Method 알아보기
---
- boolean sendEmptyMessage(int what) : What 멤버만 채워진 Message 객체 전달
- boolean sendEmptyMessageAtTime(int what, long upTimeMillies) : upTimeMillies에 지정된 시간에 what 멤머반 채워진 Message를 보냄 
- boolean sendEmptyMessageDelayed(int what, long delayMillies) : 현재 시간에서 delayMillies 만큼의 시간 후에 what 멤버만 채워진 Message 전달
- boolean sendMessage(Message msg) : Message객체 전달, MessageQueue의 가장 마지막에 Message 추가(Queue 형식)
- boolean sendMessageAtFrontOfQueue(Message msg) : Message 객체 전달, MessageQueue의 가장 처음 위치에 Message 추가
- boolean sendMessageAtTime(Message msg, long upTimeMillies) : upTimeMillies에 지정된 시간에 Message 객체 전달
- boolean sendMessageDelayed(Message msg, long delayMillis) : 현재 시작에서 delayMillies 만큼의 시간 후에 Message 객체 전달

## 마치며..
---
이번 포스팅에서는 Looper와 Handler에 대하여 알아보았습니다.  
간단하게 요약하자면 Thread간 통신을 위해 Looper와 Handler를 사용한다고 이해하시면 될 것 같습니다.  
다음에는 [Android Process 및 Thread](https://narvis2.github.io/posts/Android-Process-Thread/) 에 대하여 알보는 시간을 갖도록 하겠습니다.  