---
title: Android Network 관리
author: Narvis2
date: 2022-07-18 09:08:00 +0900
categories: [Android, Network]
tags: [android, Network]
---

안녕하세요. Narvis2 입니다.  
오늘은 Android Network 관리에 대하여 알아보도록 하겠습니다.  
실전 코드를 통하여 알아볼 것 이고 AndroidViewModel 를 사용하며 activityX 의 viewModels와 fragmentX activityViewModels를 통하여 Fragment의 상태를 공유하여 처리하는 코드입니다.  
Android에서는 ConnectivityManager를 통하여 연결 상태를 확인할 수 있습니다.  
Android에서는 셀룰러, 와이파이, 일반 네트워크(Default Network)가 존재하는데 기본으로 사용할 네트워크는 시스템에서 결정합니다.  
아래에서 코드를 통해 알아보겠습니다.

## 1. AndroidViewModel 을 사용한 Network 관리
Hilt + DataStore 를 기본으로 사용하겠습니다.  
DataStore은 셀룰러 on/off 유무를 저장하기 위해 사용하였으며, Hilt는 DataStore를 ViewModel의 생성자에 주입받기 위해 사용합니다.  
DataStore에 대하여 잘 모르시면 해당 링크를 참고하시길 권장 드립니다. -> [DataStore 정리](https://narvis2.github.io/posts/Android-DataStore/)
``` kotlin
@HiltViewModel // Hilt를 ViewModel에서 사용할 때는 해당 어노테이션을 달아주셔야 합니다.
class NetworkViewModel @Inject constructor(
    application: Application, 
    private val dataStore: DataStoreModule
) : AndroidViewModel(application) {
    // 해당 Enum Class를 바탕으로 DataBinding을 통해 UI 를 Controll 합니다.
    enum class NetworkStatus : EnumClassProguard {
        CONNECT_ERROR, CONNECT_NETWORK, DISCONNECT_NETWORK, CONNECT_NETWORK_BUT_NOT_USE_MOBILE_DATA
    }

    private var manager: ConnectivityManager? = null
    
    // Network가 연결 되었는지 유무
    private var isNetworkConn = false
    // Wifi 연결 유무
    private var isWifiConn = false
    // Cellular 연결 유무
    private var isCellularConn = false

    // 현재 네트워크 상태를 담고있는 LiveData, 해당 LiveData를 Activity/Fragment에서 Observing하여 Network 별 UI를 Controll 합니다.
    private val _currentNetworkStatus = MutableLiveData<NetworkStatus>()
    val currentNetworkStatus: LiveData<NetworkStatus>
        get() = _currentNetworkStatus
    
    // DataBinding에서 사용될 StateFlow 입니다.
    // StateFlow는 초기값이 항상 필요하며, StateFlow는 중복값을 무시하고 .value를 통해 현재 값을 가져올 수 있습니다.(양방향 Observing)
    val networkStatus = MutableStateFlow(NetworkStatus.CONNECT_ERROR)

    // Mobile Data 즉 Cellular 연결을 체크할지 유무입니다.
    private var checkMobileData = false

    // Network 가 연결되었을 경우, 연결되어있지 않을 경우를 Boolean으로 구분하여 처리합니다.
    // Coroutine Channel 은 정확히 한 번만 처리해야하는 Event를 처리하는데 사용됩니다. (단방향 Observing)
    private val _networkAction = Channel<Boolean>(Channel.CONFLATED)
    val networkAction = _networkAction.receiveAsFlow()

    // Wifi를 담당하는 NetworkCallback 입니다.
    private val wifiNetworkCallback = object : ConnectivityManager.NetworkCallback() {
        // 연결된 경우
        override fun onAvailable(network: Network) {
            super.onAvailable(network)
            isWifiConn = true
            changeNetworkStatus()
        }

        // 연결 끊긴 경우
        override fun onLost(network: Network) {
            super.onLost(network)
            isWifiConn = false
            changeNetworkStatus()
        }
    }

    // Cellular를 담당하는 NetworkCallback 입니다.
    private val cellularNetworkCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            super.onAvailable(network)
            isCellularConn = true
            changeNetworkStatus()
        }

        override fun onLost(network: Network) {
            super.onLost(network)
            isCellularConn = false
            changeNetworkStatus()
        }
    }

    // Default Network를 담당하는 NetworkCallback 입니다.
    private val defaultNetworkCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            super.onAvailable(network)
            isNetworkConn = true
            changeNetworkStatus()
        }

        override fun onLost(network: Network) {
            super.onLost(network)
            isNetworkConn = false
            changeNetworkStatus()
        }
    }

    // 네트워크 상태를 바꿔주는 함수 입니다.
    fun changeNetworkStatus() = viewModelScope.launch {
        // 모바일 데이터 사용 유무를 DataStore에서 가져옵니다.
        val isDataConn = dataStore.getUserMobileData.first()
        networkStatus.value = when {
            // 네트워크가 연결된 경우
            isNetworkConn && (isWifiConn || !checkMobileData || (isDataConn && isCellularConn)) -> {
                NetworkStatus.CONNECT_NETWORK
            }
            // 네트워크가 연결되어있지만 셀룰러를 사용하지 않는 경우
            isNetworkConn && !isDataConn && isCellularConn -> {
                NetworkStatus.CONNECT_NETWORK_BUT_NOT_USE_MOBILE_DATA
            }
            // 나머지의 경우
            else -> {
                NetworkStatus.DISCONNECT_NETWORK
            }
        }

        if (networkStatus.value != currentNetworkStatus.value) {
            _currentNetworkStatus.value = networkStatus.value
        }
    }

    // 네트워크 Callback을 등록할 때 사용하는 함수 입니다.
    fun register(checkMobileData: Boolean = true, checkNetworkStatus: Boolean = false) {
        this.checkMobileData = checkMobileData
        manager =
            application.applicationContext.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        // Android Version이 24보다 클 경우 DefaultNetwork 사용
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            registerDefaultNetwork()
        } else {
            isNetworkConn = true
        }
        // Wifi Network Callback 등록
        registerWifi()
        // Cellular Network Callback 등록
        registerCellular()

        if (checkNetworkStatus) {
            // 버전 관리 SDK_INT 23 이상부터는 activeNetworkInfo 는 Deprecated 됨
            if (Build.VERSION.SDK_INT >= 23) {
                checkActiveNetwork()
            } else {
                checkActiveNetworkInfo()
            }
            
            changeNetworkStatus()
        }
    }

    // 네트워크 Callback을 해제할 때 사용하는 함수 입니다.
    fun unRegister() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            unRegisterDefaultNetwork()
        }
        unRegisterWifi()
        unRegisterCellular()
        manager = null
    }

    // 활성화된 Network 정보를 가져오는 함수 입니다. SDK_INT 23 이상부터는 activeNetwork 를 통하여 가져옵니다.
    private fun checkActiveNetwork() {
        manager?.activeNetwork?.let {
            isNetworkConn = true
            isWifiConn = hasTransport(it, NetworkCapabilities.TRANSPORT_WIFI)
            isCellularConn = hasTransport(it, NetworkCapabilities.TRANSPORT_CELLULAR)
        }
    }

    private fun hasTransport(network: Network, transport: Int): Boolean =
        manager?.getNetworkCapabilities(network)?.hasTransport(transport) ?: false

    // 활성화된 Network 정보를 가져오는 함수 입니다. SDK_INT 23 이하에서는 activeNetworkInfo 를 통하여 가져옵니다.
    @Suppress("DEPRECATION")
    private fun checkActiveNetworkInfo() {
        manager?.activeNetworkInfo?.let {
            if (it.isConnectedOrConnecting) {
                isNetworkConn = true
                isWifiConn = it.type == ConnectivityManager.TYPE_WIFI
                isCellularConn = it.type == ConnectivityManager.TYPE_MOBILE
            }
        }
    }

    // WIFI Callback을 등록하는 함수 입니다.
    private fun registerWifi() {
        manager?.registerNetworkCallback(
            createNetworkRequest(NetworkCapabilities.TRANSPORT_WIFI),
            wifiNetworkCallback
        )
    }

    // WIFI Callback을 해제하는 함수 입니다.
    private fun unRegisterWifi() {
        manager?.unregisterNetworkCallback(wifiNetworkCallback)
    }

    // Cellular Callback을 등록하는 함수 입니다.
    private fun registerCellular() {
        manager?.registerNetworkCallback(
            createNetworkRequest(NetworkCapabilities.TRANSPORT_CELLULAR),
            cellularNetworkCallback
        )
    }

    // Cellular Callback을 해제하는 함수 입니다.
    private fun unRegisterCellular() {
        manager?.unregisterNetworkCallback(cellularNetworkCallback)
    }
    
    // Default Network Callback을 등록하는 함수 입니다.
    private fun registerDefaultNetwork() {
        manager?.registerDefaultNetworkCallback(defaultNetworkCallback)
    }

    // Default Network Callback을 해제하는 함수 입니다.
    private fun unRegisterDefaultNetwork() {
        manager?.unregisterNetworkCallback(defaultNetworkCallback)
    }

    // Default Network를 제외한 사용 가능한 다른 Network를 등록하는 함수 (여기서는 WIFI, CELLULAR)
    private fun createNetworkRequest(capability: Int): NetworkRequest {
        return NetworkRequest.Builder()
            .addTransportType(capability)
            .build()
    }

    // 모바일 데이터 차단 시 차단 해제로 바꿔주는 함수 입니다.
    private fun changeUseMobileDataState() = viewModelScope.launch {
        val useMobileData = dataStore.getUserMobileData.first()
        dataStore.putUseMobileData(!useMobileData)
        changeNetworkStatus()
    }

    // 새로고침 버튼 클릭 함수 입니다.
    fun onReTryClick() = viewModelScope.launch {
        when (networkStatus.value) {
            // 네트워크가 끊겼거나 Error인 경우
            NetworkStatus.DISCONNECT_NETWORK,
            NetworkStatus.CONNECT_ERROR -> {
                _networkAction.send(false)
            }
            // 모바일 데이터에 연결은 되었으나 모바일 데이터를 차단한 경우
            NetworkStatus.CONNECT_NETWORK_BUT_NOT_USE_MOBILE_DATA -> {
                changeUseMobileDataState()
            }
            // 네트워크가 연결되었을 경우
            NetworkStatus.CONNECT_NETWORK -> {
                _networkAction.send(true)
            }
        }
    }
}
```
## 2. NetworkFragment
저는 네트워크 연결이 끊겼으면 NetworkFragment를 띄워주는 식으로 하였습니다.
``` kotlin
@AndroidEntryPoint
class NetworkFragment : DialogFragment() {

    private lateinit var binding: FragmentNetworkBinding

    // navigation safe args 사용
    private val args: NetworkFragmentArgs by navArgs()

    private val viewModel: NetworkViewModel by activityViewModels()

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = DataBindingUtil.inflate(inflater, layoutResId, container, false)
        binding.lifecycleOwner = viewLifecycleOwner
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        binding.viewModel = viewModel
        viewModel.networkStatus.value = args.networkState

        networkViewModelCallback()
    }

    private fun networkViewModelCallback() = with(viewModel) {
        currentNetworkStatus.observe(viewLifecycleOwner) {
            when(it) {
                // 네트워크가 연결되면 NetworFragment 지워줌 
                NetworkViewModel.NetworkStatus.CONNECT_NETWORK -> {
                    findNavController().popBackStack()
                }
                else -> {}
            }
        }

        // "새로고침" 눌렀을 경우
        viewLifecycleOwner.lifecycleScope.launchWhenStarted {
            networkAction.collect {
                if (it) {
                // 네트워크가 연결되었을 경우
                findNavController().popBackStack()
                } else {
                // 네트워크가 연결되지 않았을 경우
                showSnackBar(binding.root, getString(R.string.disabled_network))
                }
            }
        }
    }

    override fun onStart() {
        super.onStart()
        val d = dialog
        when (activity) {
            is IntroActivity -> {
                if (d != null) {
                    // intro 화면에서 Dialog 를 Cancel 했을 경우
                    d.setOnCancelListener {
                        requireActivity().finish()
                    }
                } else {
                    isCancelable = false
                }
            }
            is MainActivity -> {
                if (d != null) {

                } else {
                    isCancelable = false
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        dialog?.window?.let { window ->
            window.setBackgroundDrawableResource(android.R.color.transparent)
            window.clearFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND) // 기본 배경 제거
            val params = window.attributes
            params.width = WindowManager.LayoutParams.MATCH_PARENT
            params.height = WindowManager.LayoutParams.MATCH_PARENT
            window.attributes = params
            setTransparentStatusBar(window)
        }
    }
}
```

## 3. NetworfFragment Navigation Graph
``` xml
<dialog
        android:id="@+id/networkFragment"
        android:name="com.example.networtest.view.fragment.network.NetworkFragment"
        android:label="NetworkFragment"
        tools:layout="@layout/fragment_network">

        <!-- Safe Args 사용 -->
        <argument
            android:name="networkState"
            android:defaultValue="CONNECT_ERROR"
            app:argType="com.example.networtest.view.fragment.network.NetworkViewModel$NetworkStatus" />
    </dialog>

<!-- 네트워크는 언제 어디서든 끊길 수 있기 때문에 전역에서 사용 -->
<action
    android:id="@+id/action_global_networkFragment"
    app:destination="@id/networkFragment" />
```

## 4. NetworkFragment xml 정의 (R.layout.fragment_network)
``` xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <import type="com.example.networtest.view.fragment.network.NetworkViewModel.NetworkStatus"/>

        <variable
            name="viewModel"
            type="com.example.networtest.view.fragment.network.NetworkViewModel" />
    </data>

    <FrameLayout
        android:id="@+id/network_fl"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#55000000">

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@color/colorTranslucentDim">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:layout_marginTop="200dp"
                android:orientation="vertical"
                android:paddingStart="20dp"
                android:paddingEnd="20dp">

                <ImageView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:src="@drawable/search_none_w" />
                
                <!-- 네트워크가 끊겼을 경우 "연결이 원활하지 않습니다.\n네트워크 연결 상태를 확인하거나\n다시 시도해 주세요." 를 보여줌 -->
                <!-- 나머지 경우 즉, Cellular 차단의 경우 "모바일데이터 차단 상태입니다.\n차단을 해제하거나\nWi-Fi를 설정해 주세요." 를 보여줌 -->
                <TextView
                    android:id="@+id/network_tv"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="15.8dp"
                    android:layout_marginBottom="17.2dp"
                    android:gravity="center"
                    android:letterSpacing="-0.02"
                    android:lineHeight="20sp"
                    android:lineSpacingExtra="6sp"
                    android:lineSpacingMultiplier="1.2"
                    android:text="@{viewModel.networkStatus == NetworkStatus.DISCONNECT_NETWORK ? @string/retry_disabled_network : @string/use_mobile_data}"
                    android:textAlignment="center"
                    android:textColor="@android:color/white"
                    android:textSize="14sp" />

                <LinearLayout
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_horizontal"
                    android:orientation="horizontal">

                    <!-- 네트워크가 끊겼을 경우 "새로고침" 보여줌 -->
                    <!-- 모바일 데이터가 차단되었을 경우 "차단해제" 보여줌 -->
                    <Button
                        android:id="@+id/retry_btn"
                        style="?android:attr/borderlessButtonStyle"
                        android:layout_width="89dp"
                        android:layout_height="40dp"
                        android:layout_marginStart="4dp"
                        android:layout_marginEnd="4dp"
                        android:background="@drawable/positive_bg3"
                        android:letterSpacing="-0.02"
                        android:lineSpacingExtra="-4sp"
                        android:textColor="#ffffff"
                        android:textSize="14sp"
                        android:text="@{viewModel.networkStatus == NetworkStatus.DISCONNECT_NETWORK ? @string/retry : @string/block_off }"
                        android:onClick="@{() -> viewModel.onReTryClick()}"
                        tools:text="@string/retry"/>
                </LinearLayout>
            </LinearLayout>
        </FrameLayout>
    </FrameLayout>
</layout>
```

## 5. NetworkViewModel 연결
``` kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding 
    private val mainViewModel: MainViewModel by viewModels()
    private val networkViewModel: NetworkViewModel by viewModels()

    private lateinit var navController: NavController

    val navHostFragment by lazy {
        supportFragmentManager.findFragmentById(R.id.navHostFragmentContainer) as NavHostFragment
    }

    override fun onCreate(saveInstanceState: Bundle?) {
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.lifecycleOwner = this
        binding.viewModel = mainViewModel

        navController = navHostFragment.navController

        // Network Callback 등록
        networkViewModel.register(checkNetworkStatus = true)

        initMainViewModelCallback()
        initNetworkViewModelCallback()
    }

    private fun initMainViewModelCallback() = with(mainViewModel) {
        isMobileDataOk.observe(viewLifecycleOwner) {
            // isModileDataOk 의 값이 변결될 때 마다 networkViewModel 의 changeNetworkStatus() 함수 호출
            networkViewModel.changeNetworkStatus()
        }
    }

    private fun initNetworkViewModelCallback() = with(networkViewModel) {
        currentNetworkStatus.observe(this@MainActivity) { it: NetworkViewModel.NetworkState!
            when (it) {
                NetworkViewModel.NetworkStatus.CONNECT_NETWORK -> {
                    // TODO: 네트워크가 연결되었을 때 동작 넣기
                }
                // 네트워크 차단 및 Error, Disconnect 의 경우 NetworkFragment 보여줌
                else -> {
                    navController.navigate(
                        NavigationDirections.actionGlobalNetworkFragment(it)
                    )
                }
            }
        }
    }
    
    // Activity가 onDestory() 될 때 Network Callback 해제
    override fun onDestroy() {
        networkViewModel.unRegister()
        super.onDestroy()
    }
}
```
## 5. MainViewModel 정의 (모바일데이터 차단/해제 기능)
``` kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val dataStore: DataStoreModule
) : ViewModel() {
    // DataStore 의 flow 값을 LiveData로 변경
    val isMobileDataOk: LiveData<Boolean> = liveData {
        dataStore.getUserMobileData.collect {
            emit(it)
        }
    }

    // 모바일 데이터 차단/해제 클릭 Listener
    fun onMobileClick() = viewModelScope.launch {
        // 누를때마다 isMobileDataOk 의 반대값을 넣어줌
        dataStore.putUseMobileData(!isMobileDataOk.value!!)
    }
}
```
## 6. MainActivity xml 정의 (R.layout.activity_main)
``` xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".activity.main.MainActivity">

    <data>
        <variable
            name="viewModel"
            type="com.example.networtest.view.activity.main.MainViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.fragment.app.FragmentContainerView
            android:id="@+id/navHostFragmentContainer"
            android:name="androidx.navigation.fragment.NavHostFragment"
            android:layout_width="0dp"
            android:layout_height="0dp"
            app:defaultNavHost="true"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:navGraph="@navigation/navigation" />
            
            <!-- isMobileDataOk 가 true 이면 "차단", false 이면 "차단해제" 보여줌 -->
            <Button
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:text="@{viewModel.isMobileDataOk ? @string/block : @string/block_off}"
                android:onClick="@{() -> viewModel.onMobileClick()}"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintBottom_toBottomOf="parent" /> 
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```
## 7. 모바일 데이터 차단 유무를 저장을 위한 DataStore 정의
DataStore에 대하여 자세히 알고싶으시면 해당 링크를 참고하세요. -> [DataStore 정리](https://narvis2.github.io/posts/Android-DataStore/)
``` kotlin
class DataStoreModule(
    private val context: Context
) {
    // DataStore 인스턴스 생성
    private val Context.dataStore by preferencesDataStore(name = "test.db")

    private val mobileDataKey = booleanPreferencesKey("use_mobile_data")

    val getUserMobileData: Flow<Boolean> = context.dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { preferences ->
            preferences[mobileDataKey] ?: false
        }

    suspend fun putUseMobileData(use: Boolean) {
        val getData = getUserMobileData.first()
        if (getData != use) {
            context.dataStore.edit { preferences ->
                preferences[mobileDataKey] = use
            }
        }
    }
}
```

## 8. DataStore Hilt Module 정의
``` kotlin
@Module
@InstallIn(SingletonComponent::class)
object LocalDataModule {

    @Singleton
    @Provides
    fun provideDataStoreModule(@ApplicationContext context: Context): DataStoreModule {
        return DataStoreModule(context)
    }
}
```

## 마무리
이번 포스팅에서는 Android ConnectivityManager 를 통한 Network 관리에 대하여 코드를 위주로 알아보았습니다.  
해당 코드에서는 Default Network, Wifi, Cellular 상태를 관리하며, Cellular 차단 기능 또한 들어있습니다.  
Network 의 연결이 끊겼을 시 NetworkFragment를 띄워주며 네트워크 끊김, Cellular는 연결되어 있지만 Cellular를 차단한 경우에 따라 UI를 분리하고 있습니다.  
또한 버튼을 클릭 시 모바일 데이터를 차단/해제 하는 코드 또한 존재합니다.  
다음 시간에는 위에서 사용중인 Flow 및 StateFlow, Coroutine Channel 에 대하여 자세히 알아보도록 하겠습니다.