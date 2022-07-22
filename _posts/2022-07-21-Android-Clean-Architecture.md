---
title: Android Naver Open API 영화 검색 어플
author: Narvis2
date: 2022-07-21 11:11:00 +0900
categories: [Android, Naver Open API]
tags: [android, cleanArchitecture, MVVM]
---

안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 **_"Naver Open API를 이용한 영화 검색"_** 어플에 사용된 기술과 프로젝트에 대하여 알아보겠습니다.  
  
재직중인 회사에서 사용하는 기술을 과제의 주제에 벗어나지 않는 선에서 모두 보여드리고 노력하였습니다.  
  
해당 프로젝트 전체 코드 👉🏿 [Naver Open API를 이용한 영화 검색 어플](https://github.com/narvis2/MovieSearchApp)

## 앱 기능
---
1. Naver Open API를 사용하여 원하는 영화를 검색합니다. 
2. 검색한 내용을 RecyclerView를 통해 보여주며 Paging3를 사용하여 Page에 따라 list를 추가로 불러와 보여줍니다. 
> **_참고_** : 해당 포스팅에 Paging3에 관하여 자세히 정리 했습니다. 👉🏿 [Paging3에 대하여](https://narvis2.github.io/posts/Android-Paging3/)
3. RecyclerView에 썸네일 이미지, 제목, 평점, 연도, 감독, 출연 배우를 표기하고 Item Click 시 해당 API Response에 있는 link를 바탕으로 WebView를 띄워줍니다.
4. 해당 어플은 Network가 끊겼을 때 할 수 있는 동작이 없기 때문에 ConnectivityManager를 통해 Network Callback을 받아 네트워크가 끊겼을 때 네트워크가 끊겼다는 화면을 보여줍니다. 네트워크가 연결되면 자동으로 해당 화면이 사라지고 어플을 계속 컨트롤 할 수 있습니다.
> **_참고_** : 해당 포스팅에 ConnectivityManager를 통한 Netwrok 관리에 관하여 자세히 정리 했습니다.  
> 👉🏿 [Netwrok 관리](https://narvis2.github.io/posts/Android-Network/) 
5. 프로젝트의 패턴은 Claen Architecture MVVM 을 채택하였습니다.
6. AAC ViewModel 을 사용하여 앱 구성요소 변경에도 앱의 데이터와 내용이 사라지지 않게 설계하였습니다.
7. Single Activity를 적용하였습니다.
> Jetpack Navigation을 통해 Single Activity를 적용하였으며, 이렇게한 이유 역시 재직중인 회사에서 사용하는 기술을 과제의 주제에 벗어나지 않는 선에서 모두 보여드리고자 선택하였습니다.  
> 🚩**_참고_**🚩 Single Activity의 장단점
> - 1️⃣ 장점 : Activity Stack에 Activity를 쌓아두기 보다. Fragment BackStack에서 Fragment를 관리하는 것이 메모리 관리에서의 효율도 챙기고 화면 전환시 Activity 보다 순조롭습니다. 
> - 2️⃣ 장점 : 데이터 공유에 있어 이점이 있습니다. Activity간 Data를 전달하려면 Intent를 통해 직렬화/역직렬화를 과정을 거쳐야 하며, 이는 메모리 공유에 비해 결코 가벼운 작업이 아닙니다.
> - 3️⃣ 장점 : 재사용성이 증가합니다. View 나 Business Logic을 Fragment 단위로 분리 가능합니다. 이는 아키텍쳐 원칙에서 가장 중요한 원칙인 관심사 분리를 통해 의존성을 분리하고 독립성을 가지게 합니다.
> - 4️⃣ 장점 : 유연한 UI 구현이 가능합니다. (Navigation Component, BottomSheetDialog 등..)
> - 1️⃣ 단점 : 비동기로 인해 예기치 않은 동작이 발생할 수 있습니다.
> - 2️⃣ 단점 : **_Transaction 내에서 문제가 발생한다면 디버그가 매우 어려운 IllegalStateException을 발생시킵니다._**
8. StateFlow, Coroutine Channel을 사용하였습니다.
> Clean Architecture에서 LiveData 사용시 View의 의존성이 없는 Domain, Data Layer에서는 비동기 방식으로 데이터를 처리하기에 자연스러운 방법이 없기에 StateFlow, Coroutine Channel 을 사용하였습니다.
> **_참고_** : 해당 포스팅에 StateFlow, Channel 에 관하여 자세히 정리 했습니다. 👉🏿 [StateFlow, Channel](https://narvis2.github.io/posts/Android-StateFlow/)
9. Dagger-Hilt 를 통한 DI(Dependency Injection) 관리를 적용하였습니다.
> Domain Layer, Data Layer 와 최종적으로 Presentation Layer의 의존성 관리를 위해 Hilt 를 사용하였습니다.

## Clean Architecture 란
---
- Clean Code 로 소프트웨어 공학의 대가 **"로버트 C.마틴"**이 제시한 소프트웨어 디자인 철학입니다.
- 소프트웨어의 관심사를 계층별로(domain, data, presentation)로 분리하여 코드 종속성이 외부로부터 내부로 의존하도록 하는 것이 주요 원칙입니다.

## Clean Architecture 장점
---
- 코드의 재사용성이 용이해집니다.
> **_참고_** : 멀티 모듈로 분리하여 작성시 API는 같고(회원가입, 로그인) 앱의 구성만 다른 경우 Domain Layer, Data Layer는 그대로 복사해서 가져와 사용할 수 있음
- Unit Test에 있어 용이해집니다.
> **_참고_** : 비즈니스 규칙은 UI, DB, 백엔드 서버등 외부와 무관하게 테스팅이 가능합니다.
- UI 독립성
> **_참고_** : UI 변경이 시스템의 나머지 부분에 영향을 미치지 않습니다.
- 유지보수 용이
> **_참고_** : 각 모듈의 의존성을 분리하므로 만약 코드 한 곳을 바꾼다고 하여도 나머지 부분을 변경할 필요가 없습니다.

## Android Clean Architecture 구조
---
![Desktop View](/assets/img/architecture/clean-architecture-structure.png){: width="100%" }
- 위의 사진을 참고하시면 이해가 쉽습니다.
- 위의 사진의 Entity는 Android에서는 Model로 정의합니다.
> **_참고_** : Domain Layer의 Model은 사용자에게 직접 보여줄 Model 로 API에서 내려오는 Response에 직접 접근하지 않고 DTO를 통해 접근한다고 생각하시면 됩니다.
- 아래에서 자세히 알아보겠습니다.
### 1. Domain Layer
![Desktop View](/assets/img/architecture/domain-layer-structure.png){: width="50%", height="50%" }
- 어떤 모듈에도 의존적이지 않는 독립적인 Module 입니다.(최상위 모듈)
- 비즈니스 로직을 처리하는 곳 입니다.
- Data Layer에 접근하기 위한 interface를 갖고 있습니다. (Repository interface정의)
- 안드로이드에 의존성을 가지지 않은 순수 Java 및 Kotlin 코드로만 구성됩니다.
> **_참고_** : Paging3를 사용 시 Android 의존성을 가지고있지 않을 방법이 없어 위의 그림에서는 어쩔 수 없이 android 의존성을 가졌습니다. 
  
1. UseCase
- 행동들의 최소 단위, 즉 비즈니스 로직을 구현합니다. 
> **_참고_** : 사용자를 가져오는 UseCase, Login을 하는 UseCase, Token을 가져오는 UseCase 등이 포함 됩니다.
- 보통 UseCase 하나당 하나의 기능을 담당합니다.
>**_참고_** : UseCase 이름만 보고 이것이 무슨 기능을 하는지 짐작하고 구분할 수 있어야 합니다.
- Pesentation Layer 에서 어떠한 이벤트나 동작에 의하여 호출되는 방향으로 설계합니다.
>**_참고_** : 보통 ViewModel 의 생성자에 DI(Dependency Inject)을 통해 주입받아 사용합니다. 
- UseCase 생성시 어떤 DataBase or Remote(API)를 사용했는지에 대한 고민을 하지 않고 Domain Layer에서 정의한 Repository 함수를 호출하는 방식으로 정의합니다.  
- Data Layer 에서 실제로 어떻게 데이터를 가져올지에 대한 정의는 하지 않고 해당 Repository의 메서드를 호출하는 방식으로 구현합니다.
>**_참고_** : 다음은 Naver 검색 API 를 통해 검색 결과를 PagingSource로 가져오는 UseCase 예제입니다. 👇
``` kotlin
class GetMovieListPagingDataUseCase @Inject constructor(
    private val naverRepository: NaverRepository
) {
    operator fun invoke(
        query: StateFlow<String>
    ): Flow<PagingData<MovieInfoModel>> {
        return naverRepository.getMovieList(query)
    }
}
```
2. Repository
- 데이터의 출처(Local DB 인지 API 응답인지 등..)와 관계없이 동일 Interface로 데이터에 접속할 수 있도록 만듭니다.
- UseCase가 필요로 하는 데이터 저장/수정 등의 기능을 제공하는 Interface입니다.
- Interface에 함수만 정의하고 구현은 Data Layer에서 합니다.
>**_참고_** : 다음은 Naver 검색 API 를 통해 검색 결과를 PagingSource로 가져오는 Repository 예제입니다. 👇
``` kotlin
interface NaverRepository {
    fun getMovieList(query: StateFlow<String>): Flow<PagingData<MovieInfoModel>>
}
```
3. Model
- 앱의 실질적인 데이터가 여기에 구현됩니다. 즉, UI에 보여질 실제 데이터입니다.
> - **_참고_** : 만약 API 호출을 통해 Response를 받아 왔다면 해당 Response에 직접 접근하지 않고 Domain Layer의 Model을 통해 접근한다고 생각하시면 됩니다. DTO(Data Transfer Object)와 유사하다고 보시면 됩니다.
> - **_참고_** : 다음은 Naver 검색 API 를 통해 검색 결과의 사용자에게 직접 보여줄 Model 예제입니다. 👇
``` kotlin
data class MovieInfoModel(
    val title: String,
    val link: String,
    val image: String,
    val subtitle: String,
    val pubDate: String,
    val director: String,
    val actor: String,
    val userRating: Float
) {
    val rating = userRating / 2
}
```

### 2. Data Layer
![Desktop View](/assets/img/architecture/data-layer-structure.png){: width="50%", height="50%" }
- Domain Layer에서 정의한 Repository 구현제, dataSource, Retrofit API 정의, Room DB 정의, Mapper, API Response Model 등으로 구성됩니다.
- Domain Layer에 대한 의존성을 가지고 있습니다.
- 데이터 베이스(Local DB)와 서버(Remote)의 통신을 담당합니다.
- ✔️ **Data Model** : API 통신을 통해서 받게되는 Response 나 Local DB를 통해 얻게되는 Entity를 정의합니다.
> **_참고_** : 다음은 Naver 검색 API 를 통해 검색 결과 Response 입니다. 👇
``` kotlin
data class MovieResponse(
    @SerializedName("lastBuildDate")
    val lastBuildDate: String,
    @SerializedName("total")
    val total: Int,
    @SerializedName("start")
    val start: Int,
    @SerializedName("display")
    val display: Int,
    @SerializedName("items")
    val items: List<MovieInfo>
)
data class MovieInfo(
    @SerializedName("title")
    val title: String,
    @SerializedName("link")
    val link: String,
    @SerializedName("image")
    val image: String,
    @SerializedName("subtitle")
    val subtitle: String,
    @SerializedName("pubDate")
    val pubDate: String,
    @SerializedName("director")
    val director: String,
    @SerializedName("actor")
    val actor: String,
    @SerializedName("userRating")
    val userRating: String
)
```
---
- ✔️ **DataSource**
- 🚩 RemoteDataSource : 네트워크 통신을 담당하는 interface 입니다. 
- 🚩 LocalDataSource : Local Database와의 통신을 담당하는 interface 입니다.
- **_참고_** : 다음은 Naver 검색 API 에 접근하는 RemoteDataSource 입니다. 👇
``` kotlin
interface RemoteDataSource {
    suspend fun requestSearchMovie(
        query: String,
        start: Int,
        display: Int
    ): Response<MovieResponse>
}
```
- **_참고_** : 다음은 Naver 검색 API 에 접근하는 RemoteDataSource의 구현체입니다. Hilt로 부터 생성자에 NaverApiService를 주입받아 사용합니다.  👇
``` kotlin
class RemoteDataSourceImpl @Inject constructor(
    private val naverApiService: NaverApiService
) : RemoteDataSource {

    override suspend fun requestSearchMovie(
        query: String,
        start: Int,
        display: Int
    ): Response<MovieResponse> {
        return naverApiService.searchMovie(query, start, display)
    }
}
```
---
  
- ✔️ **Api**
- 🚩 Retrofit 을 이용한 Netwrok 통신을 하기 위한 interface 입니다.
- 🚩 Url에 접근하는 함수만 만들어 놓고, Retrofit의 인스턴스 생성 및 세팅은 Hilt를 통해합니다.
> **_참고_** : GET 방식을 사용하며 Query로 query, start, display를 넣어주고 있습니다. 여기서 start는 아이템 시작 index이며, display는 limit 입니다.  
> 다음은 Naver 검색 API URL에 접근하는 코드입니다. 👇
``` kotlin
interface NaverApiService {
        @GET("/v1/search/movie")
        suspend fun searchMovie(
            @Query("query") query: String,
            @Query("start") start: Int,
            @Query("display") display: Int
        ): Response<MovieResponse>
}
```
---
- ✔️ **Repository (Domain Layer의 Repository 구현체)**
- 🚩 DataSource 를 interface 형태로 참조하여 Domain Layer의 Repository를 구현합니다.
> **_참고_** : DI(Dependency Inject)을 사용하여 생성자에 DataSource를 주입받습니다.
- 🚩 실제로 어떻게 데이터를 가져올지에 대한 정의를 하여 UseCase에서 해당 Repository 구현체의 함수를 사용합니다.
> **_참고_** : 해당 코드에서는 Paging3를 사용하여 Naver API Response 결과를 Flow<PagingSource<T>> 의 형태로 가져옵니다. 👇
``` kotlin
class NaverRepositoryImpl @Inject constructor(
        private val remoteDataSource: RemoteDataSource
) : NaverRepository {
        override fun getMovieList(query: StateFlow<String>): Flow<PagingData<MovieInfoModel>> {
            return Pager(
                config = PagingConfig(pageSize = 10),
                pagingSourceFactory = {
                    MovieInfoListPagingDataSource(
                        remoteDataSource = remoteDataSource,
                        searchQuery = query,
                        limit = 10
                    )
                }
            ).flow.distinctUntilChanged().flowOn(Dispatchers.IO)
        }
}
```
---
- ✔️ **Mapper**
- 🚩 Data Model 에 의해 가져온 Data를 Domain Model 로 맞게 바꿔주는 매핑 작업을 하여 UseCase 에서 Domain Model 을 반환합니다.
> **_참고_** : API 를 통해 받아온 Response를 UI에 맞는 Domain Model로 맵핑 👇 (Kotlin 확장 함수 기능을 사용하여 처리합니다.)
``` kotlin
object ObjectMapper {
        // List<MovieInfo> -> List<MovieInfoModel> 로 변환
        fun List<MovieInfo>.toMovieInfoListModel(): List<MovieInfoModel> = map {
            MovieInfoModel(
                title = it.title,
                link= it.link,
                image= it.image,
                subtitle= it.subtitle,
                pubDate= it.pubDate,
                director= it.director,
                actor= it.actor,
                userRating= it.userRating.toFloat()
            )
        }
        // MovieInfo -> MovieInfoModel 로 변환
        fun MovieInfo.toMovieInfoModel(): MovieInfoModel = MovieInfoModel(
            title = this.title,
            link= this.link,
            image= this.image,
            subtitle= this.subtitle,
            pubDate= this.pubDate,
            director= this.director,
            actor= this.actor,
            userRating= this.userRating.toFloat()
        )
}
```
--- 
- ✔️ **MovieInfoListPagingDataSource**
- 🚩 Naver로 부터 받아오는 Response 의 값을 PagingSource에 담아 반환합니다.
> **_참고_**  
> - 해당 프로젝트는 Paging3를 사용하고 있습니다.  
> - Hilt를 사용하여 remoteDataSource 를 생성자에 주입받고 있습니다.
> - **_참고_** : 해당 포스팅에 Paging3에 관하여 자세히 정리 했습니다. 👉🏿 [Paging3에 대하여](https://narvis2.github.io/posts/Android-Paging3/)
  
``` kotlin
class MovieInfoListPagingDataSource @Inject constructor(
    private val remoteDataSource: RemoteDataSource,
    private val searchQuery: StateFlow<String>,
    private val limit: Int
) : PagingSource<Int, MovieInfoModel>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, MovieInfoModel> {
        val pageNumber = params.key ?: 1
        try {
            val response = withContext(Dispatchers.IO) {
                remoteDataSource.requestSearchMovie(
                    query = searchQuery.value,
                    start = pageNumber,                        
                    display = limit
                )
            }

            if (response.isSuccessful) {
                response.body()?.let {
                    val movieList = it.items.toMovieInfoListModel()
                    /**
                     * nextKey -> naver Api의 start 는 index 이고, 현재 display(limit) 가 10 이므로
                     * 10 이후에 start 값은 11 이 되어야함
                     * 스크롤되어 페이지가 추가될때 마다 10개씩 가져온다.
                     */
                    return LoadResult.Page(
                        data = movieList,
                        prevKey = null,
                        nextKey = if (it.items.size >= limit) {
                            pageNumber + limit
                        } else {
                            null
                        }
                    )
                }
            }

            return LoadResult.Error(
                LoadException("영화 정보를 가져오는데 실패하였습니다.")
            )
        } catch (e: Exception) {
            Timber.e("paging catch Error -> ${e.message}")
            return LoadResult.Error(e)
        }
    }
    override fun getRefreshKey(state: PagingState<Int, MovieInfoModel>): Int? {
        // 새로고침 될때 항상 전부 새로고침 되도록 null 을 return
        return null
    }
}
```
### 3. Presentation Layer (App Module)
![Desktop View](/assets/img/architecture/presentation-layer-structure.png){: width="50%", height="50%" }
- 화면의 입력에 대한 처리 등 UI와 관련된 부분을 담당합니다.
> **_참고_** : Activity, Fragment, View, ViewModel, Di 등을 포함합니다.
- Android에 높은 의존성을 가지고 있습니다.
- Domain Layer, Data Layer 에 대한 의존성을 가지고 있습니다.
- ViewModel에서 domain layer의 UseCase를 주입 받아 사용하여 각 ViewModel당 기능 이 무엇인지 쉽게 알 수 있습니다.
- MVVM 패턴을 사용하여 ViewModel에서 데이터를 받아 View(Activity/Fragment)에 넘겨주는 방식으로 구현되어 있습니다.
- 해당 프로젝트는 Single Activity를 적용했습니다.
> **_참고_** Single Activity의 장단점
> - 1️⃣ 장점 : Activity Stack에 Activity를 쌓아두기 보다. Fragment BackStack에서 Fragment를 관리하는 것이 메모리 관리에서의 효율도 챙기고 화면 전환시 Activity 보다 순조롭습니다. 
> - 2️⃣ 장점 : 데이터 공유에 있어 이점이 있습니다. Activity간 Data를 전달하려면 Intent를 통해 직렬화/역직렬화를 과정을 거쳐야 하며, 이는 메모리 공유에 비해 결코 가벼운 작업이 아닙니다.
> - 3️⃣ 장점 : 재사용성이 증가합니다. View 나 Business Logic을 Fragment 단위로 분리 가능합니다. 이는 아키텍쳐 원칙에서 가장 중요한 원칙인 관심사 분리를 통해 의존성을 분리하고 독립성을 가지게 합니다.
> - 4️⃣ 장점 : 유연한 UI 구현이 가능합니다. (Navigation Component, BottomSheetDialog 등..)
> - 1️⃣ 단점 : 비동기로 인해 예기치 않은 동작이 발생할 수 있습니다.
> - 2️⃣ 단점 : **_Transaction 내에서 문제가 발생한다면 디버그가 매우 어려운 IllegalStateException을 발생시킵니다._**
- ✔️ **Base**
- 🚩 Presentaion Layer의 Base에 사용될 Class 모음입니다.
> - 해당 프로젝트에는 BaseActivity, BaseFragment, BaseViewModel, BaseDialogFragment, MyLoadStateAdapter 가 정의되어 있습니다.
> - **_참고_** -> [깃허브](https://github.com/narvis2/MovieSearchApp/tree/main/app/src/main/java/com/example/moviesearchapp/base)
- ✔️ **Di**
- 🚩 DI(Dependency Injection) 즉 의존성을 관리하는 페지키 입니다.
- 1️⃣ ApiModule : Hilt Module로 관리하며 @InstallIn에 SingletonComponent를 넣어 관리되고 있습니다.  
> - SingletonComponent는 @Singleton 어노테이션과 같이 사용되며, Application이 onCreate()가 되면 생성되고, Application이 onDestory()가 되면 자동으로 파괴됩니다.  
> - ApiModule 에서는 Retrofit 인스턴스를 만들어 Hilt 를 통해 Provide합니다. 해당 부분에서 OkHttp3 Interceptor 를 사용하여 Header 부분에 **X-Naver-Client-Id** 와 **X-Naver-Client-Secret**를 넣어줍니다.  
> - Hilt 를 통해 NaverApiService를 제공하고 있으며, 해당 NaverApiService는 RemoteDataSource의 생성자에 주입됩니다. 또한 RemoteDataSource 역시 Hilt를 통해 제공되고 있습니다. (이렇게 제공된 RemoteDataSource는 Hilt를 통해 RepositoryModule의 생성자에서 주입 받습니다.)
> - 다음은 ApiModule 코드 입니다. 👇 [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/di/ApiModule.kt)
    
``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object ApiModule {

    @Singleton
    @Provides
    fun provideRemoteDataSource(
        naverApiService: NaverApiService
    ): RemoteDataSource {
        return RemoteDataSourceImpl(
            naverApiService
        )
    }

    @Singleton
    @Provides
    fun provideNaverApiService(
        retrofit: Retrofit
    ): NaverApiService {
        return retrofit.create(NaverApiService::class.java)
    }

    @Singleton
    @Provides
    fun provideRetrofit(
        client: OkHttpClient,
        converter: GsonConverterFactory
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.API_HOST)
            .client(client)
            .addConverterFactory(converter)
            .build()
    }

    @Singleton
    @Provides
    fun provideGsonConverterFactory(): GsonConverterFactory {
        return GsonConverterFactory.create()
    }

    @Singleton
    @Provides
    fun provideOkHttpClient(
        logging: HttpLoggingInterceptor,
        interceptor: Interceptor,
    ): OkHttpClient = OkHttpClient.Builder().apply {
        if (BuildConfig.DEBUG) {
            addInterceptor(logging)
        }
        addInterceptor(interceptor)
    }.build()

    @Singleton
    @Provides
    fun provideLoggingInterceptor(): HttpLoggingInterceptor = HttpLoggingInterceptor().apply {
        level = if (BuildConfig.DEBUG) {
            HttpLoggingInterceptor.Level.BODY
        } else {
            HttpLoggingInterceptor.Level.NONE
        }
    }

    @Singleton
    @Provides
    fun provideInterceptor(): Interceptor = Interceptor { chain ->
        val response = chain.proceed(
            chain.request().newBuilder().apply {
                addHeader("X-Naver-Client-Id", BuildConfig.NAVE_CLIENT_ID)
                addHeader("X-Naver-Client-Secret", BuildConfig.NAVE_CLIENT_SECRET)
            }.build()
        )

        response
    }
}
```
- 2️⃣ RepositoryModule : Hilt Module로 관리하며 @InstallIn에 SingletonComponent를 넣어 관리되고 있습니다.  
> - SingletonComponent는 @Singleton 어노테이션과 같이 사용되며, Application이 onCreate()가 되면 생성되고, Application이 onDestory()가 되면 자동으로 파괴됩니다.  
> - Hilt를 통해 생성자로 부터 RemoteDataSource를 주입받아 사용하고 있으며 NaverRepository를 Hilt를 통해 제공합니다.
> - 다음은 Repository Module Code 입니다. 👇 [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/di/RepositoryModule.kt)
``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
        @Provides
        @Singleton
        fun provideNaverRepository(
            remoteDataSource: RemoteDataSource
        ): NaverRepository {
            return NaverRepositoryImpl(remoteDataSource)
        }
}
```
- 3️⃣ UseCaseModule : Hilt Module로 관리하여 @InstallIn 에 ViewModelComponent 를 넣어 관리되고 있습니다.
> - 해당 UseCase는 ViewModel 의 생성자에 주입받아 사용됩니다. 따라서 @InstallIn 에 ViewModelComponent를 사용합니다.
> - ViewModelComponent 는 @ViewModelScope 와 같이 사용되며, AAC ViewModel의 생명주기에 따라 관리됩니다.
> - 즉, ViewModel 이 생성되면 생성되고, ViewModel의 onCleared()가 호출되면 자동으로 파괴됩니다.
> - 다음은 UseCase Module Code 입니다. 👇 [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/di/UseCaseModule.kt)
``` kotlin
@Module
@InstallIn(ViewModelComponent::class)
object UseCaseModule {
        @Provides
        @ViewModelScoped
        fun provideGetMovieListPagingDataUseCase(
            naverRepository: NaverRepository
        ): GetMovieListPagingDataUseCase {
            return GetMovieListPagingDataUseCase(naverRepository)
        }
}
```
- ✔️ **MovieApplication**
- 🚩 Hilt를 사용하도록 설정하고 Debug에 필요한 Timber를 등록합니다.
> **_참고_** : @HiltAndroidApp -> 컴파일 타임 시 표준 컴포넌트 빌딩에 필요한 클래스들을 초기화 해줍니다. Hilt를 사용하는 모든 앱은 @HiltAndroidApp 이 달린 Application 클래스를 반드시 포함해야 합니다.
``` kotlin
@HiltAndroidApp
class MovieApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        if (BuildConfig.DEBUG) {
            Timber.plant(TimberDebugTree())
        }
    }
}
```
- ✔️ **Activity**
- 🚩 해당 프로젝트에서는 MainActivity 와 WebViewActivity로 구분됩니다.
- 1️⃣ MainActivity : 해당 프로젝트는 SingleActivity를 채택하고 있어 MainActivity 에서는 Navigation의 NavHostFragment를 연결해주고 있으며, Network 관리를 통해 Network Callback을 등록하고 Network 상태 값을 Observing 하는 코드가 포함되어 있습니다. 또한 뒤로 가기 클릭 시 한번 눌렀을 경우 "한번 더 누르면 종료됩니다." 라는 SnackBar를 띄워주기 위해 BackStack을 관리하는 코드가 포함되어 있습니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/view/activity/main/MainActivity.kt)
- 2️⃣ WebViewActivity : 영화 상세 정보를 WebView로 띄워주기 위해 생성했습니다. onPause()와 onResume(), onDestory()에 따라 WebView를 관리해주고 있으며, Intent를 통해 Main -> WebView 로 이동 시 내부/외부 WebView를 띄워줄 수 있는 코드가 포함되어 있습니다. 해당 WebViewActivity는 내부 웹뷰 일 경우 동작합니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/view/activity/web/WebViewActivity.kt)  
  
- ✔️ **Fragment**
- 🚩 해당 프로젝트에서는 HomeFragment 와 NetworkFragment로 구분됩니다.
- 1️⃣ HomeFragment : 네이버 Open API 를 활용하여 영화 정보를 받아와 사용자에게 보여주는 역할을 합니다. 또한 NavHostFragment로 모든 Navgigation의 시작점이 됩니다. 네이버 Open API 를 활용하여 영화 정보를 받아와 사용자에게 보여 주기 위해 RecyclerView 를 포함하고 있으며, 또한 PagingData의 Load 상태에 따라 DataBinding으로 View를 Controll 하고 있습니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/view/fragment/home/HomeFragment.kt)
- 2️⃣ HomeViewModel : 앞서 Domain Layer의 UseCase를 Hilt를 통해 생성자에 주입받아 네이버 Open API 데이터를 가져오는 역할을 합니다. 이렇게 가져온 데이터는 HomeFragment 에서 Observing 하여 RecyclerView 에 넣어줍니다. 또한 MutableStateFlow를 사용하여 Data Binding을 통해 HomeFragment 의 View를 Controll 해주고 있으며, Coroutine Channel 을 통해 해당 Click Event의 Listener를 관리하고 있습니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/view/fragment/home/HomeViewModel.kt)
- 3️⃣ NetworkFragment : 네트워크가 끊겼을 때 보여주는 DialogFragment 입니다. 네트워크가 연결되면 자동으로 dialog를 dissmiss 해주는 코드를 포함하고 해당 다이얼로그가 보여질 때 다른 View에서는 해당 다이얼로그를 Cancel 하지 못하게하는 코드가 포함되어 있습니다.
- 4️⃣ NetworkViewModel : ConnectivityManager를 통해 Network Callback을 받아 Network를 관리하는 코드가 포함되어 있습니다. 해당 ViewModel은 MainActivity에서 생성되며, NetworkFragment 에서는 FragmentX를 사용하여 activityViewModels() 코드로 Fragment 상태를 공유합니다.
> Network 관리에 대해서는 옆의 포스팅을 참고하시길 바랍니다. 👉 [Android ConnectivityManager를 통한 Network 관리](https://narvis2.github.io/posts/Android-Network/)

- ✔️**MovieInfoAdapter(PagingAdapter)**
- RecyclerView 에 데이터를 표시하는 기본 UI 구성요소 입니다.
- PagingData를 입력 유형으로 사용하고 내부 load 이벤트를 수신합니다.
- BackGround Thread 에서 DiffUtil 을 사용하여 미세 조정한 후 데이터를 load하므로 UI Thread에 새 항복을 추가하는 동안 문제가 발생하지 않습니다.
- Root Click Listener를 포함하고 있고 Domain Layer에 정의한 MovieInfoModel을 데이터로 사용합니다. 
- 해당 Adapter 또한 Data Binding을 통해 View에 데이터를 연결해주고 있습니다.
> - **_참고_** : [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/view/fragment/home/adapter/MovieInfoAdapter.kt)
> - **_참고_** : 해당 포스팅에 Paging3에 관하여 자세히 정리 했습니다. 👉🏿 [Paging3에 대하여](https://narvis2.github.io/posts/Android-Paging3/)

- ✔️**Utils**
- 🚩 해당 프로젝트에서는 데이터 바인딩을 위한 BindingAdapters, Flow를 Lifecycler에 맞게 Observing하기 위해 만들어진 FlowObserver, keyBoardUtils, Listener, TimberDebugTree 등이 포함되어 있습니다.
- 1️⃣ BindingAdpaters [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/utils/BindingAdapters.kt)
> - 서버에서 받아오는 image Url을 ImageView에 넣어주기 위해 Glide를 사용하는 함수, Html 을 String으로 바꿔주는 함수, 중복 클릭 방지 함수 등이 포함되어 있습니다.
- image Load 함수 👇
``` kotlin
@BindingAdapter(value = ["loadImage", "placeHolder", "error"], requireAll = false)
fun ImageView.loadImage(
    imageUrl: String,
    @DrawableRes
    placeholder: Int = 0,
    @DrawableRes
    error: Int = 0,
) {
    // imageUrl 이 비어있을 때 로직 추가
    if (imageUrl.isEmpty()) {
        if (placeholder != 0) {
            this.setImageResource(placeholder)
        }

        return
    }

    val options = RequestOptions()

    @SuppressLint("CheckResult")
    if (error != 0) {
        // 로드가 실패할 경우 표시할 리소스를 설정합니다.
        options.error(error)
    }

    Glide.with(this.context)
        .setDefaultRequestOptions(options)
        .load(imageUrl).apply {
            @SuppressLint("CheckResult")
            if (placeholder != 0) {
                // 리소스가 로드되는 동안 표시할 드로어블 리소스의 Android 리소스 ID를 설정합니다.
                apply(options.placeholder(placeholder))
            }
        }
        .into(object : CustomTarget<Drawable>() {
            override fun onResourceReady(resource: Drawable, transition: Transition<in Drawable>?) {
                setImageDrawable(resource)
            }

            override fun onLoadCleared(placeholder: Drawable?) {
                setImageDrawable(placeholder)
            }
        })
}
```
- 중복 클릭 방지 함수
``` kotlin
@BindingAdapter(value = ["onSingleClick", "interval"], requireAll = false)
fun View.onSingleClick(listener: View.OnClickListener? = null, interval: Long? = null) {
    if (listener != null) {
        setOnClickListener(object : Listener.OnSingleClickListener(interval ?: 1000L) {
            override fun onSingleClick(v: View?) {
                listener.onClick(v)
            }
        })
    } else {
        setOnClickListener(null)
    }
}
```
- Html을 String으로 바꿔주는 함수
``` kotlin
@BindingAdapter("htmlToString")
fun TextView.htmlToString(text: String) {
    this.text = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        Html.fromHtml(text, Html.FROM_HTML_MODE_LEGACY).toString()
    } else {
        @Suppress("DEPRECATION")
        Html.fromHtml(text).toString()
    }
}
```
- 2️⃣ FlowObserver : Flow를 Activity/Fragment 의 Lifecycler에 맞게 Observing 하기위해 만든 클래스 입니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/utils/FlowObserver.kt)
- 해당 포스팅에 FlowObserver에 관하여 자세히 설명했습니다. 👉🏿 [StateFlow, Channel](https://narvis2.github.io/posts/Android-StateFlow/)
  
``` kotlin
// FlowObserver -> lifecycle 이 onStart가 되면 구독 시작, onStop이 되면 구독 취소
class FlowObserverInStop<T>(
    lifecycleOwner: LifecycleOwner,
    private val flow: Flow<T>,
    private val collector: suspend (T) -> Unit
) {

    private var job: Job? = null

    init {
        lifecycleOwner.lifecycle.addObserver(LifecycleEventObserver { source, event ->
            when (event) {
                Lifecycle.Event.ON_START -> {
                    job = source.lifecycleScope.launch {
                        flow.collect {
                            collector(it)
                        }
                    }
                }
                Lifecycle.Event.ON_STOP -> {
                    job?.cancel()
                    job = null
                }
                else -> {}
            }
        })
    }
}

inline fun <reified T> Flow<T>.observeOnLifecycleStop(
    lifecycleOwner: LifecycleOwner,
    noinline collector: suspend (T) -> Unit
) = FlowObserverInStop(lifecycleOwner, this, collector)

// .onEach{ } 사용할때 사용
inline fun <reified T> Flow<T>.observeInLifecycleStop(
    lifecycleOwner: LifecycleOwner
) = FlowObserverInStop(lifecycleOwner, this) {}
```

- 3️⃣ Listener : 중복 클릭 방지를 위해 사용됩니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/utils/Listener.kt)
``` kotlin
class Listener {
    abstract class OnSingleClickListener(private val minIntervalMS: Long = 1000L) : View.OnClickListener {
        // 마지막으로 클릭한 시간
        private var mLastClickTime: Long = 0

        abstract fun onSingleClick(v: View?)

        override fun onClick(v: View?) { //현재 클릭한 시간
            val currentClickTime = SystemClock.uptimeMillis()
            // 이전에 클릭한 시간과 현재시간의 차이
            val elapsedTime = currentClickTime - mLastClickTime
            // 마지막클릭시간 업데이트
            mLastClickTime = currentClickTime
            // 내가 정한 중복클릭시간 차이를 안넘었으면 클릭이벤트 발생못하게 return
            if (elapsedTime <= minIntervalMS) return
            // 중복클릭시간 아니면 이벤트 발생
            onSingleClick(v)
        }
    }
}
```
- 4️⃣ KeyboardUtils : 해당 프로젝트에서는 Keyboard가 보이지 않을때 Focus를 해제하기 위해 사용됩니다. [깃허브](https://github.com/narvis2/MovieSearchApp/blob/main/app/src/main/java/com/example/moviesearchapp/utils/KeyboardUtils.kt)
> - OnGlobalLayoutListener 을 사용하여 View 의 전체 영역이 바뀔때 Event를 받아 사용합니다.
> - **_주의_** : windowSoftInputMode 에 adjustNothing 을 사용하면 View 의 변경을 알 수 없어 Listener 가 동작하지 않습니다.

## 마치며
해당 포스팅는 Naver Open API를 사용하여 영화를 검색하고 검색한 영화를 사용자에게 보여주는 앱에 대하여 사용 기술에 대한 설명에 초점에 두고 작성하였습니다.  
최대한 주어진 주제에 벗어나지 않는 선에서 제가 사용할 수 있는 기술을들 적용해 보았습니다.  
긴 글 읽어주셔서 감사합니다.  
[프로젝트 전체 코드](https://github.com/narvis2/MovieSearchApp)