---
title: Android DataStore(SharedPreference 대체)
author: Narvis2
date: 2022-07-18 09:08:00 +0900
categories: [Android, DataStore]
tags: [android, datastore]
---

안녕하세요. Narvis2 입니다.  
오늘은 Android DataStore 에 대하여 알아보고자 합니다.  
DataStore 이전에는 가벼운 데이터를 저장하기 위해 SharedPreferences를 사용하였는데 SharedPreferences 에는 다양한 한계점이 있었습니다.  
첫째로 비동기 작업을 제대로 해주지 않으면 ANR 즉,  Application Not Responding(애플리케이션 응답 없음)이 발생할 수 있습니다.  
두번째는 오류가 발생하면 확인이 불가능고, Runtime에 Exception이 발생(RuntimeException)하어 앱이 강제 종료될 수 있었습니다.  
또한 type safety 가 보장되지 않아 어떤 데이터가 저장되고 추출되는지 일일히 type converting 즉, 형 변환을 해주어야 했습니다.  
밑에서 자세히 알아보도록 하겠습니다.

## SharedPreference의 한계점
우선 SharedPreference 의 단점에 대하여 알아보겠습니다.
- SharedPrefereence 는 동기 API를 제공하는 것과 MainThread로 부터 안전하지 않습니다. 즉, UI Thread를 Blocking 하여 ANR을 발생시킬 수 있습니다.
- Strong Consistency가 보장되지 않아 Multiple Thread 환경에서 안전하지 않습니다.
- type safety가 보장되지 않습니다.  
이러한 문제점을 개선하기 위해 DataStore가 등장하였습니다.  

## DataStore 
- 프로토콜 버퍼를 사용하여 key-value 쌍 또는 유형이 저장된 객체를 저장할 수 있는 데이터 저장소 솔루션입니다.
- Coroutine 및 Flow 를 사용하여 비동기적이고 일관된 Transaction 방식으로 데이터를 저장하는 것이 특징입니다.
- DataStore 는 내부적으로 Dispatchers.IO 를 사용하기 때문에 UI Thread에서 사용하여도 ANR이 발생하지 않습니다.
- Runtime Error로 부터 안전합니다.
- Protocol Buffer를 사용하여 Type Safety한 코드를 작성할 수 있습니다.
- DataStore 2가지  
> - Preferences DataStore : key 와 value로 구성되며 Type Safety 하지 못합니다.
> - Proto DataStore : 사용자가 정의한 데이터를 저장하며 Protocol Buffer를 이용하여 Schema를 정의해야 합니다. 데이터의 타입을 보장해 줍니다.(Type Safety)  
  
> **[_참고_](https://developer.android.com/topic/libraries/architecture/datastore?hl=ko)** -> 복잡한 대규모 데이터 저장, 부분 업데이트, 참조 무결성을 지원해야 할 경우에는 DataStore 대신 Room을 사용하는 것이 좋습니다. DataStore는 소규모 단순 데이터 저장에 적합하며 부분 업데이트나 참조 무결성은 지원하지 않습니다.

## Preferences DataStore Example 
Preferences DataStore 에 대하여 코드를 통해 알아보겠습니다.  
해당 코드에서는 Android Dagger-Hilt를 사용하여 의존성 관리 하였으며, 간단한 String값 저장 및 Boolean값을 저장하는 코드입니다.    
DataStore 의존성 관련은 [구글 공식 홈페이지](https://developer.android.com/topic/libraries/architecture/datastore?hl=ko)를 참고하여 주세요.
1. DataStore 정의
``` kotlin
class DataStoreModule(
    private val context: Context
) {
    // DataStore 인스턴스 생성
    private val Context.dataStore by preferencesDataStore(name = "test.db")

    // String 저장 Key 값
    private val stringKey = stringPreferencesKey("string_key_name")

    // DataStore로 부터 값 가져오기
    val getText = context.datastore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[stringKey] ?: ""
        }

    // DataStore에 값 저장
    suspend fun setText(text: String) {
        // DataStore에 저장된 String 값과 매개변수로 넘어오는 String값이 다를 경우만 저장(중복값 저장 안함)
        val getData = getText.first()
        if (getData != text) {
            context.dataStore.edit { preferences ->
                preferences[stringKey] = userId
            }
        }
    }

    // Boolean 저장 Key 값
    private val booleanKey = booleanPreferencesKey("boolean_key_name")

    // DataStore로 부터 값 가져오기
    val getIsAutoLogin = context.datastore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[booleanKey] ?: false
        }

    // DataStore에 값 저장
    suspend fun setIsAutoLoing(isAuto: Boolean) {
        // DataStore에 저장된 String 값과 매개변수로 넘어오는 String값이 다를 경우만 저장(중복값 저장 안함)
        val getData = getIsAutoLogin.first()
        if (getData != text) {
            context.dataStore.edit { preferences ->
                preferences[booleanKey] = userId
            }
        }
    }
}
```
2. Hilt를 이용한 의존성 주입
``` kotlin
@Module
@InstallIn(SingletonComponent::class) // Application 의 onCreate()에서 생성, onDestroy()에서 파괴
object LocalDataModule {
    @Singleton
    @Provides
    fun provideDataStoreModule(@ApplicationContext context: Context): DataStoreModule {
        return DataStoreModule(context)
    }
}
```
3. 사용
``` kotlin
@AndroidEntryPoint
class HomeFragment : Fragment() {
    private lateinit var binding: FragmentHomeBinding

    // Hilt를 통해 주입받는 객체는 Private 할 수 없음
    @Inject
    lateinit var dataStore: DataStoreModule
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.lifecycleOwner = viewLifecycleOwner
    
        // DataStore 값 가져오기
        viewLifecycleOwner.lifecycleScope.launch {
            dataStore.getText.collect { it: String 
                Timber.e("data store getText Value -> $it")
            }
        }

        // DataStore 값 저장하기
        CoroutineScope.launch(Dispatchers.IO) {
            dataStore.setText("test")
        }
    }
}
```

## 마치며
이번 포스팅에서는 DataStore에 대하여 알아보았습니다.  
Preferences DataStore에 대하여 알아보았는데 저는 Proto DataStore는 잘 사용하지 않습니다.  
구글 공식 홈페이지에 나와있듯 복잡한 데이터는 Room에 저장하는 것이 좋고 편하므로 복잡한 데이터를 저장할때는 Room을 이용합니다.  
이번 포스팅은 여기서 마치도록 하겠습니다.  