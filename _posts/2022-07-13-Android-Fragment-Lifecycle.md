---
title: Android Fragment Lifecycle 생명주기 정리
author: Narvis2
date: 2022-07-13 09:28:00 +0900
categories: [Android, Lifecycle]
tags: [android, fragment, lifecycle]
---
이전 [Activity Lifecycle 정리](https://narvis2.github.io/posts/Android-Activity-Lifecycle/) 포스팅에 이어 이번에는 Fragment 의 Lifecycle에 대해 알아보겠습니다.
## Android Fragment Lifecycle 동작 순서
---
이번에도 [구글 공식 홈페이지](https://developer.android.com/guide/fragments/lifecycle)에 나와있는 이미지를 바탕으로 살펴보겠습니다.  
해당 이미지를 참고하여 Fragment Lifecycle 의 순서를 보면 Fragment 생성시 onAttach() -> onCreate() -> onCreateView() -> onViewCreate() -> onViewStateRestored() -> onStart() -> onResume() 순으로 Callbak이 호출되며, Fragment의 구성요소 변경 혹은 일부가 사라지면 onPause() -> onStop() ->
onSaveInstanceState() -> onDestoryView() -> onDestory() -> onDetach() 순으로 Callback이 호출되어 onDetach() 이후로 Fragment가 완전이 소멸되고 Activity와의 연결이 끊어집니다.

![Desktop View](/assets/img/lifecycle/fragment-view-lifecycle.png){: width="100%" }

## Fragment Lifecycle Callback Method 역할
---
**각 Callback Method 의 역할에 관하여 살펴보겠습니다.**
### 1. onAttach()
- Fragment가 Activity에 Attach 될때 호출된다.
- onAttach() 함수 인자로 context가 주어지기 때문에 부모 Activity에 접근이 가능하다.

### 2. onCreate()
- Fragment 에서 필요한 Resource를 초기화 하는 곳 (Ui 관련 초기화는 할 수 없다.)
- Fragment가 생성되는 시점이다.
- View를 제외하고 Activity에서 전달된 값들을 받는 등 변수들을 초기화할때 사용한다.  
즉, Fragment를 생성하면서 넘겨준 값들이 있다면 여기서 변수에 넣어주면된다.
- Bundle 타입으로 saveInstanceState 파라미터가 함께 제공 된다.  
해당 파라미터는 Fragment가 처음 생성됐을때만 null로 넘어오며, onSaveInstanceState() 함수를 재정의 하지 않았더라도 그 이후 재생성부터는 non-null 값으로 넘어온다.

### 3. onCreateView()
- Fragment와 xml을 연결하는 부분이다.
- onCreate() 이후에는 onCreateView() 와 onViewCreated() Callbak 함수가 이어서 호출된다.
- onCreateView() 의 반환값으로 정상적인 Fragment View 객체를 제공했을 때만 Fragment View 의 Lifecycle 이 생성된다.
- Fragment 와 xml 을 연결하는 작업만하고 View 의 Controll은 onViewCreated()에서 하는 것이 좋다.  
그 이유는 onCreateView()의 반환된 View 객체는 onViewCreated() 의 파라미터로 전달되고 이 시점 부터는 Fragment View 의 Lifecycle 이 **_INITIALIZED_** 상태로 업그데이트 됐기 때문이다.
``` kotlin
class HomeFragment : Fragment() {
    private lateinit var binding: FragmentHomeBinding
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)
        return binding.root
    }
}
```

### 4. onViewCreated()
- View 가 생성되면 호출된다.
- Fragment View 의 Lifecycle 이 INITIALIZED 상태로 업데이트 된다.
- Ui 관련 코드를 작성하는 곳으로 View 의 초기값을 설정해주거나 LiveData 옵저빙, StateFlow 옵저빙, Databinding, RecyclerView 또는 ViewPager2에 사용될 Adapter 세팅 등을 할 수 있다.
``` kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
    binding.lifecycleOwner = viewLifecycleOwner
    
    // todo: Ui Update or LiveData Observing...
}
```
### 5. onViewStateRestored()
- 저장해둔 모든 state 값이 Fragment 의 View 계층구조에 복원 됐을 때 호출된다.
- 이 시점의 Fragment View의 Lifecycle은 **_INITIALIZED_** 에서 **_CREATED_"" 상태로 변경되었음을 알린다.

### 6. onStart()
- Fragment가 사용자에게 보여지기 직전에 호출된다. (사용자에게 Fragment가 보이도록 해줌)
- 이 시점부터는 Fragment의 child FragmentManager를 통해 FragmentTransaction을 안전하게 수행할 수 있다.

### 7. onResume()
- Fragment가 비로서 보여지는 단계이다.
- Fragment가 보이는 상태에서 모든 Animator와 Transition 효과가 종료되고, Fragment가 사용자와 상호작용 할 수 있을 때 onResume()이 호출된다.
- Fragment가 다시 보여질 때 실행해야 할 코드를 넣으면 된다. ex) RecyclerView notifyDataSetChanged 등등
> **주의!!** FragmentTransaction 을 통해 add 하는 경우에는 아래에 깔린 FirstFragment 의 Lifecycle 은 여전히 **_RESUME_** 상태이다.

### 8. onPause()
- 사용자와 상호작용을 중지한 상태이다.
- Fragment의 일부가 사라지면 호출된다.
- 부모 Activity가 아닌 다른 Activity가 위로 올라오거나, 다른 Fragment가 Add되는 경우 onPause() 즉, 일시정지 상태로 들어간다.
- UI 관련 코드를 정지하고, 중요한 데이터를 저장
> **주의!!** Fragment 와 View의 Lifecycle 이 **_PAUSED_** 가 아닌 **_STARTED_** 가 된다.

### 9. onStop()
- Fragment가 완전히 사라지면 호출된다. (완전한 정지)
- Fragment가 더 이상 화면에 보여지지 않게되면 Fragment와 View의 Lifecycle은 **_CREATED_** 상태가 되고, onStop() Callback 함수가 호출된다.

### 10. onDestoryView()
- Fragment와 관련된 View가 제거될때 호출된다.
- 모든 exit animation 과 transition이 완료되고, Fragment가 화면으로부터 벗어났을 경우 Fragment View 의 Lifecycle 은 **_DESTROYED_**가 된다.
- 이 시점부터는 getViewLifecycleOwnerLiveData() 의 리턴값으로 null 이 반환된다.
- Activity에서 Fragment를 생성했을 당시 addToBackStack()을 요청했을 경우 onDestory()를 호출하지 않고, 인스턴스가 저장되어 있다가 Fragment를 다시 부를 때 onCreateView()를 실행하여 다시 화면에 보여지게 한다.
> **주의!!** 해당 시점에서는 Garbage collection에 의해 수거될 수 있도록 Fragment View에 대한 모든 참조가 제거되어야 한다.

### 11. onDestory()
- Fragment 가 제거되거나 FragmentManager가 destory 됐을 경우, Fragment의 Lifecycle은 **_DESTROYED_** 상태가 되고, onDestroy() 콜백 함수가 호출된다.
- View가 제거된 후 Fragment 가 완전히 소멸되기 전에 호출된다.

### 12. onDetach()
- Fragment가 완전히 소멸되고 Activity와의 연결도 끊어질 때 호출된다.

## 마지며..
Fragment는 Add 와 replace 방식이 있는데 동작하는 방식이 다릅니다. 저는 요즘 개발할때 androidx jetpack navigation component를 사용하여 Single Activity로 개발합니다. 현재 재직중인 회사 또한 Single Activity를 사용중이며, navigation component를 사용하면 fragment 간 전환이 매우 편리해집니다. 물론 navigation의 단점 또한 존재합니다. navigation component에 관해서는 추후 포스팅을 따로 올려 다뤄보도록 하겠습니다.  

---
[Activity Lifecycle 정리](https://narvis2.github.io/posts/Android-Activity-Lifecycle/)