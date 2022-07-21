---
title: Android Paging3 에 대하여
author: Narvis2
date: 2022-07-21 17:06:00 +0900
categories: [Android, Paging3]
tags: [android, paging3]
---
안녕하세요. Narvis2 입니다.  
이번 포스팅에서는 Paging3 에 대하여 알아보도록 하겠습니다.  
페이징이란 대량의 데이터를 한 번에 불러오는 것이 아니라 필요한 만큼 ""_일부분을 나눠서 가져오는 것_""을 말합니다.
API에 따라 limit(한 번에 보여줄 데이터의 개수 제한), offset(데이터의 인덱스)등으로 페이징이 처리 되어 있습니다.  
해당 포스팅에서는 Naver Open API 를 기준으로 설명 드리겠습니다.  

## Paging 3 사용 이유
- 페이징 된 데이터의 메모리 캐싱으로 시스템 리소스를 효율적으로 사용할 수 있습니다.
- 요청 중복 제거 기능을 지원합니다.
- RecyclerView 를 스크롤 할 때 밑으로 스크롤 시 nextKey에 따라 자동으로 data를 load 합니다.
- Refresh(새로고침), Retry(재시도), 오류 처리를 지원합니다.

## Paging3 의 구조
- 페이징 라이브러리는 **[안드로이드 권장 아키텍쳐](https://developer.android.com/topic/architecture#recommended-app-arch)**에 맞게 설계 되어있으며, 총 3개의 Layer로 구성 됩니다.
![Desktop View](/assets/img/architecture/paging-structure.png){: width="100%" }
- 위의 그림을 참고하여 설명하겠습니다.
### 1. Repository Layer
- PagingSource : 각 페이지에서 데이터를 얻는 방법을 정의합니다. (Local DB에서 데이터를 저장하거나 서버에서 받아오도록 설정이 가능합니다.) 즉, 데이터를 가져오는 방법을 정의합니다.
- RemoteMediaor :  로컬 데이터베이스를 이용해 네트워크 데이터를 캐싱하기 위해 사용됩니다.(7월 29일 기준 실험용으로 나와있으며 향후 변경될 수 있습니다.)
  
### 2. ViewModel Layer
- PagingConfig를 바탕으로 PagingSource에서 데이터를 얻어 PagingData를 만들고 Flow형으로 UI에 넘겨줍니다.
- Pager : PagingSource를 리턴타입으로 하는 람다와 PagingConfig를 생성자로 받으며 PagingData 를 반응형 스트림으로 생성할 수 있습니다.
> **_참고_** PagingConfig 
> - 페이징 구성 클래스로 PagingSource를 구성하는 방법을 정의합니다. 
> - pageSize 파라미터로 각 페이지에 얼마나 많은 데이터가 있어야 하는지 정의합니다. 즉 각 페이지에 보여줄 데이터의 갯수를 정의합니다.
- PagingData : 페이징된 데이터를 담아두는 컨테이너 입니다. Paging Source에서 load한 결과를 저장하며 UI Layer의 PagingDataAdapter로 넘겨 줍니다.  (ViewModel Layer를 UI Layer에 연결하는 구성요소 입니다.)  
페이지로 나눈 데이터의 SnapShot을 보유하는 컨테이너 입니다.
  
### 3. UI Layer
- PagingDataAdapter
> - PagingData 를 입력 유형으로 사용하고 내부 로드 이벤트를 수신합니다.
> - RecyclerViwe 의 Adapter 와 연결하여 사용합니다.
> - BackGround Thread에서 DiffUtil 을 사용하여 미세 조정한 후 데이터를 load 하므로 UI Thread에 새 항목을 추가하는 동안 문제가 발생하지 않습니다.

## 코드를 통해 알아보기 [깃허브](https://github.com/narvis2/MovieSearchApp)
- 해당 프로젝트는 Naver Open API 를 이용하였으며 Clean Archtecture MVVM 구조 입니다.
### 1. PagingSource 정의
- suspend fun load(params: LoadParams<Int>) : 각 페이지에서 데이터를 load 하는 방법을 정의합니다.
- nextKey : 스크롤을 밑으로 내렸을 때 더 많은 데이터를 로드할 수 있는 경우 다음 페이지에 대한 Key 입니다. 이 key를 기점으로 다음 Page의 데이터를 Load합니다. 다음 Page가 없을 경우 Null 입니다.
- prevKey : 스크롤을 위로 올렸을 때 더 많은 데이터를 로드할 수 있는 경우 이전 페이지에 대한 Key입니다. 이 key를 기점으로 이전 Page의 데이터를 Load 합니다. 이전 Page에 load할 데이터가 없을 경우 Null 입니다.
- params.key 는 현재 페이지의 Key로 Naver API 의 경우 처음 Index 가 1이기 때문에 null 일 경우 1을 반환합니다.(params.key는 처음 load할 경우 null입니다.)
- getRefreshKey(state: PagingState<Int, MovieInfoModel>) 함수는 Adapter Refresh를 할 경우 해당 메서드가 호춛되는데 여기서는 Refresh 할 경우 처음 부터 전부 데이터를 가져오기 위해 Null을 반환합니다.
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
### 2. RepositoryImpl (레파지토리 구현)
- Flow<PagingData<T>> 객체를 Return 합니다.
- pageSize 는 한 페이지당 몇 개의 Data를 보여줄지 설정합니다.
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
### 3. UseCase에서 받기
- 해당 UseCase에서 NaverRepository를 받아 처리합니다.
- 이렇게 처리된 UseCase는 ViewModel 에서 Hilt를 통해 생성자에 주입받아 사용합니다.
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
### 4. ViewModel 에서 사용
- ViewModel 에서 GetMovieListPagingDataUseCase를 Hilt를 통해 생성자에서 받아 처리합니다.
- 해당 LoadingType 은 PagingData 의 loadState에 따라 DataBinding을 통해 UI를 업데이트 해주기 위해 사용합니다.
- cachedIn(viewModelScope)를 사용하여 캐싱에 사용될 Coroutine Scope를 지정합니다.
``` kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getMovieListPagingDataUseCase: GetMovieListPagingDataUseCase
) : BaseViewModel() {

    val loadingType = MutableStateFlow(MovieLoadingType.LOADING)

    val getMovieList: Flow<PagingData<MovieInfoModel>> =
        getMovieListPagingDataUseCase(searchQuery).cachedIn(viewModelScope)
    
    enum class MovieLoadingType {
        LOADING,
        EMPTY,
        ERROR,
        VIEW
    }
}
```
### PagingAdapter 구현
- RecyclerView 의 Adapter 에 사용될 PagingAdapter를 구현합니다.
- DiffUtil을 사용하고 있으며, 클릭 리스너를 가지고 있습니다. 또한 DataBinding으로 처리합니다.
- DiffUtil 은 areItemsTheSame()이 ture를 반환하면, areContentsTheSame()를 호출하고 areContentsTheSame()또한 true를 반환하면 같은 객체라고 판단합니다.
``` kotlin
class MovieInfoAdapter : PagingDataAdapter<MovieInfoModel, MovieInfoAdapter.MovieInfoViewHolder>(
    diffUtil
) {

    interface OnItemClickListener {
        fun onClick(item: MovieInfoModel, position: Int)
    }

    interface MovieRankingPresenter {
        fun onRootClick(view: View?)
    }

    private var listener: OnItemClickListener? = null

    fun setOnItemClickListener(listener: OnItemClickListener) {
        this.listener = listener
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MovieInfoViewHolder {
        val binding: MovieInfoItemBinding = DataBindingUtil.inflate(
            LayoutInflater.from(parent.context),
            R.layout.movie_info_item,
            parent,
            false
        )

        return MovieInfoViewHolder(binding)
    }

    override fun onBindViewHolder(holder: MovieInfoViewHolder, position: Int) {
        getItem(position)?.let {
            holder.bind(it, listener)
        }
    }

    inner class MovieInfoViewHolder(
        private val binding: MovieInfoItemBinding
    ) : RecyclerView.ViewHolder(binding.root), MovieRankingPresenter {

        private var listener: OnItemClickListener? = null

        fun bind(info: MovieInfoModel, listener: OnItemClickListener?) {
            binding.info = info
            binding.presenter = this
            this.listener = listener
            binding.executePendingBindings()
        }

        override fun onRootClick(view: View?) {
            binding.info?.let {
                listener?.onClick(it, bindingAdapterPosition)
            }
        }
    }

    companion object {
        private val diffUtil = object : DiffUtil.ItemCallback<MovieInfoModel>() {
            override fun areItemsTheSame(
                oldItem: MovieInfoModel,
                newItem: MovieInfoModel
            ): Boolean {
                return oldItem.title == newItem.title
            }

            override fun areContentsTheSame(
                oldItem: MovieInfoModel,
                newItem: MovieInfoModel
            ): Boolean {
                return oldItem == newItem
            }
        }
    }
}
```

### Fragment 구현 
- ViewModel로 부터 Flow<PagingData<MovieInfoModel>>를 받아 PagingDataAdapter에 넘겨줍니다.
- PagingAdapter의 loadState를 받아와 상황별로 UI를 컨트롤 합니다.
``` kotlin
@AndroidEntryPoint
class HomeFragment : BaseFragment<FragmentHomeBinding, HomeViewModel>(
    R.layout.fragment_home
), View.OnFocusChangeListener, TextWatcher {
    override val viewModel: HomeViewModel by viewModels()

    private val movieInfoAdapter by lazy {
        MovieInfoAdapter()
    }

    private lateinit var scrollLayoutManager: LinearLayoutManager
    private var scrollReset = false

    private val scrollListener = object : RecyclerView.OnScrollListener() {
        override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
            super.onScrolled(recyclerView, dx, dy)
            val lastVisibleItemPosition = scrollLayoutManager.findFirstVisibleItemPosition()

            // 스크롤 시 Scroll Top Button 을 보여줍니다.
            if (lastVisibleItemPosition >= 1) {
                binding.topButton.show()
            } else {
                binding.topButton.hide()
            }
        }
    }

    private val keyboardObserver = object : KeyboardUtils.SoftKeyboardToggleListener {
        override fun onToggleSoftKeyboard(isVisible: Boolean, height: Int) {
            // keyboard 가 보이지 않을 때 searchEt 포커스를 해제합니다.
            if (!isVisible) {
                binding.searchEt.let {
                    if (it.isFocusable) {
                        it.clearFocus()
                    }
                }
            }
        }
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        KeyboardUtils.addKeyboardToggleListener(requireActivity(), keyboardObserver)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.viewModel = this.viewModel
        binding.focusListener = this
        binding.textListener = this
        binding.noResultLayout.noResultText = resources.getString(R.string.str_no_search)

        binding.listView.apply {
            scrollLayoutManager = layoutManager as LinearLayoutManager

            adapter = movieInfoAdapter.withLoadStateHeaderAndFooter(
                header = MyLoadStateAdapter { movieInfoAdapter.retry() },
                footer = MyLoadStateAdapter { movieInfoAdapter.retry() }
            )

            addOnScrollListener(scrollListener)
        }

        // ViewModel로 부터 받아온 PagingData 를 PagingDataAdapter 에 넣어줍니다.
        viewModel.getMovieList.observeOnLifecycleStop(viewLifecycleOwner) {
            movieInfoAdapter.submitData(it)
        }

        // PagingDataAdapter 의 loadState 에 따라 UI를 컨트롤 합니다.
        movieInfoAdapter.loadStateFlow.observeOnLifecycleStop(viewLifecycleOwner) {
            viewModel.loadingType.value = when {
                // 초기 load 또는 새로고침이 실패하면 -> ERROR
                it.source.refresh is LoadState.Error && movieInfoAdapter.itemCount == 0 -> {
                    if (binding.refreshView.isRefreshing) {
                        binding.refreshView.isRefreshing = false
                    }

                    MovieLoadingType.ERROR
                }

                // List 가 비어있는 경우 -> EMPTY
                it.source.refresh is LoadState.NotLoading && movieInfoAdapter.itemCount == 0 -> {
                    if (binding.refreshView.isRefreshing) {
                        binding.refreshView.isRefreshing = false
                    }

                    MovieLoadingType.EMPTY
                }

                // Local Db 또는 Remote 에서 새로 고침이 성공한 경우 -> VIEW
                it.source.refresh is LoadState.NotLoading -> {
                    if (binding.listRefreshView.isRefreshing) {
                        binding.listRefreshView.isRefreshing = false
                    }

                    if (scrollReset) {
                        scrollReset = false
                        binding.listView.stopScroll()
                        scrollLayoutManager.scrollToPositionWithOffset(0, 0)
                    }

                    MovieLoadingType.VIEW
                }

                else -> MovieLoadingType.LOADING
            }
        }

        // PagingDataAdapter 의 Click Listener 입니다.
        movieInfoAdapter.setOnItemClickListener(object : MovieInfoAdapter.OnItemClickListener {
            override fun onClick(item: MovieInfoModel, position: Int) {
                startActivity(
                    WebViewActivity.getInAppBrowserIntent(
                        activity = requireActivity(),
                        url = item.link,
                        pageTitle = "영화 상세 정보",
                        showTitle = true
                    )
                )
            }
        })

        // IME_ACTION_SEARCH : 키보드 search 모양 클릭 시 Listener 등록
        binding.searchEt.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_SEARCH) {
                viewModel.onSearchClick()
                true
            } else {
                false
            }
        }

        initViewModelCallback()
    }

    private fun initViewModelCallback() = with (viewModel) {
        // 스크롤 Top 버튼 클릭 시 해당 Callback이 실행됩니다.
        scrollTop.onEach {
            binding.listView.stopScroll()
            scrollLayoutManager.scrollToPositionWithOffset(0, 0)
        }.observeInLifecycleStop(viewLifecycleOwner)

        // Refresh Layout 으로 Refresh 할 경우 해당 Callback이 실행됩니다.
        refresh.onEach {
            if (binding.refreshView.isRefreshing) {
                binding.refreshView.isRefreshing = false
            }

            scrollReset = true
            movieInfoAdapter.refresh()
            onScrollTop()
        }.observeInLifecycleStop(viewLifecycleOwner)
        
        // 검색 버튼 클릭 시 해당 Callback이 실행됩니다.
        search.onEach {
            onRefreshList()
        }.observeInLifecycleStop(viewLifecycleOwner)
    }

    // Edit Text의 OnFocusChangeListener를 정의합니다.
    override fun onFocusChange(view: View?, hasFocus: Boolean) {
        if (view != null) {
            when (view.id) {
                R.id.searchEt -> {
                    viewModel.searchFocusView.value = hasFocus
                }
            }
        }
    }

    // Edit Text의 TextWatcher를 정의합니다. 
    override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {}

    override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {}

    override fun afterTextChanged(s: Editable?) {
        viewModel.notBlankContents.value = if (!s.isNullOrEmpty()) {
            viewModel.searchFocusView.value = true
            true
        } else {
            false
        }
    }

    override fun onDetach() {
        KeyboardUtils.removeKeyboardToggleListener(keyboardObserver)
        super.onDetach()
    }
}
```

### 마치며
이번 포스팅에서는 Paging3를 사용하는 이유와 Clean Architecture에서 Paging3를 사용하는 방법에 대하여 알아보았습니다.  
페이징이란 대량의 데이터를 한 번에 불러오는 것이 아니라 필요한 만큼 ""_일부분을 나눠서 가져오는 것_"" 이라는 점을 기억하시면 되겠습니다.  
해당 프로젝트의 전체 코드는 [Naver Open API 영화 검색](https://github.com/narvis2/MovieSearchApp)에서 확인하실수 있습니다.
