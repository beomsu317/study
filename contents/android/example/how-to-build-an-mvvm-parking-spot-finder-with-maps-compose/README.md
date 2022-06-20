# **How to Build an MVVM Parking Spot Finder with Maps Compose - Android Studio Tutorial**

차를 어디에 주차했는지 알려주는 앱을 만들어보자. 구글에서 maps compose가 신규로 제공된다. 구글맵을 직접 Jetpack Compose에서 사용할 수 있다.

우선 구글맵을 사용하기 위해 Google Cloud Platform에서 프로젝트를 하나 생성한다. 그리고 Maps SDK for Android를 선택하여 활성화한다.

<div align="center">
<img src="img/gcp.png">
</div>

그리고 사용자 인증 정보에서 API 키를 생성한다.

<div align="center">
<img src="img/api.png">
</div>

[여기서](https://github.com/philipplackner/ParkingSpotFinder) Initial Project를 클론한 후 <application> 태그 안에 구글맵 API_KEY를 설정한다.

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.plcoding.mapscomposeguide">

    <application>
        <meta-data android:name="com.google.android.geo.API_KEY"
                   android:value="AIzaSyA_HBcRD3292Nb4jU3EEfVKXc3lzD6UqDU"/>
    </application>

</manifest>
```

[SNAZZY MAPS](https://snazzymaps.com/)에서 여러 맵 스타일을 만들거나 가져올 수
있다. [이 스타일](https://snazzymaps.com/style/87718/fallout-pip-boy)을 가져와 `MapStyle`을 생성 후 `json`에 붙여넣자.

```kotlin
object MapStyle {
    val json = """ ... """
}
```

`presentation` 패키지를 생성한 후 `MapState`를 생성 및 작성해준다.

```kotlin
data class MapState(
        val properties: MapProperties = MapProperties(), // 맵 설정
        val isFalloutMap: Boolean = false
)
```

UI에 대한 이벤트를 처리하기 위해 `MapEvent`를 생성하고 다음과 같이 작성한다.

```kotlin
sealed class MapEvent {
    object ToggleFalloutMap : MapEvent()
}
```

동일 패키지에 `MapsViewModel`을 생성한 후 다음과 같이 작성한다.

```kotlin
class MapsViewModel : ViewModel() {

    var state by mutableStateOf(MapState())

    fun onEvent(event: MapEvent) {
        when (event) {
            is MapEvent.ToggleFalloutMap -> {
                state = state.copy(
                        properties = state.properties.copy(
                                mapStyleOptions = if (state.isFalloutMap) {
                                    null
                                } else MapStyleOptions(MapStyle.json)
                        ),
                        isFalloutMap = !state.isFalloutMap
                )
            }
        }
    }
}
```

동일 패키지에 `MapScreen`을 생성하고 다음과 같이 작성한다.

```kotlin
@Composable
fun MapScreen(
        viewModel: MapsViewModel = androidx.lifecycle.viewmodel.compose.viewModel()
) {
    val scaffoldState = rememberScaffoldState()
    val uiSettings = remember { // remember를 사용해 recompose 시에도 MapUiSettings() 다시 생성하지 않도록 하기 위함
        MapUiSettings(zoomControlsEnabled = false)
    }
    Scaffold(
            scaffoldState = scaffoldState,
            floatingActionButton = {
                FloatingActionButton(onClick = {
                    viewModel.onEvent(MapEvent.ToggleFalloutMap)
                }) {
                    Icon(
                            imageVector = if (viewModel.state.isFalloutMap) {
                                Icons.Default.ToggleOff
                            } else Icons.Default.ToggleOn, contentDescription = "Toggle Fallout map"
                    )
                }
            }
    ) {
        GoogleMap(
                modifier = Modifier.fillMaxSize(),
                properties = viewModel.state.properties,
                uiSettings = uiSettings,
                onMapLongClick = {

                }
        )
    }
}
```

마지막으로 `MainActivity`에서 `MapScreen`을 호출해준다.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MapsComposeGuideTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                        modifier = Modifier.fillMaxSize(),
                        color = MaterialTheme.colors.background
                ) {
                    MapScreen()
                }
            }
        }
    }
}
```

이제 토글 버튼을 통해 맵을 변경할 수 있다.

<div align="center">
<img src="img/result1.gif" width="40%">
</div>

이제 Room DB를 이용해 파킹 장소를 저장해서 유저가 어디에 차를 주차했는지 알려주자.

`data` 패키지를 생성한 후 `remote`가 필요하지 않으므로 구분하지 않고 `data` 패키지에서 모든 작업을 수행한다.

Room에 저장할 `ParkingSpotEntity`를 작성하자.

```kotlin
@Entity
data class ParkingSpotEntity(
        val lat: Double,
        val lng: Double,
        @PrimaryKey val id: Int? = null
)
```

다음으로 `ParkingSpotDao`를 생성 및 작성해준다.

```kotlin
@Dao
interface ParkingSpotDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertParkingSpot(spot: ParkingSpotEntity)

    @Delete
    suspend fun delelteParkingSpot(spot: ParkingSpotEntity)

    @Query("SELECT * FROM parkingspotentity")
    fun getParkingSpots(): Flow<List<ParkingSpotEntity>>
}
```

그리고 `ParkingSpotDatabase`를 만들어준다.

```kotlin
@Database(
        entities = [ParkingSpotEntity::class],
        version = 1
)
abstract class ParkingSpotDatabase : RoomDatabase() {

    abstract val dao: ParkingSpotDao
}
```

실제 사용할 데이터를 만들기 위해 `domain` 패키지를 생성하고 `ParkingSpot` data class를 만들어준다.

```kotlin
data class ParkingSpot(
        val lat: Double,
        val lng: Double,
        val id: Int? = null
)
```

`ParkingSpotEntity`에서 `ParkingSpot` data class로 변환해주는 mapper 함수를 만들자. `data` 패키지에 `ParkingSpotMapper` 파일을 생성한 후 다음과 같이
확장 함수를 만들어준다.

```kotlin
fun ParkingSpotEntity.toParkingSpot(): ParkingSpot {
    return ParkingSpot(
            lat = lat,
            lng = lng,
            id = id
    )
}

fun ParkingSpot.toParkingSpotEntity(): ParkingSpotEntity {
    return ParkingSpotEntity(
            lat = lat,
            lng = lng,
            id = id
    )
}
```

`domain` 패키지에 `repository` 패키지를 생성한 후 `ParkingSpotRepository` 인터페이스를 만들어준다.

```kotlin
interface ParkingSpotRepository {

    suspend fun insertParkingSpot(spot: ParkingSpot)

    suspend fun deleteParkingSpot(spot: ParkingSpot)

    fun getParkingSpots(): Flow<List<ParkingSpot>>
}
```

`data` 패키지 하위에 `ParkingSpotRepositoryImpl`를 생성 및 구현해준다.

```kotlin
class ParkingSpotRepositoryImpl(
        private val dao: ParkingSpotDao
) : ParkingSpotRepository {
    override suspend fun insertParkingSpot(spot: ParkingSpot) {
        dao.insertParkingSpot(spot.toParkingSpotEntity())
    }

    override suspend fun deleteParkingSpot(spot: ParkingSpot) {
        dao.delelteParkingSpot(spot.toParkingSpotEntity())
    }

    override fun getParkingSpots(): Flow<List<ParkingSpot>> {
        return dao.getParkingSpots().map { spots ->
            spots.map { it.toParkingSpot() }
        }
    }
}
```

Dagger-Hilt를 사용하기 위해 `ParkingSpotApplication` 클래스를 만들고 `@HiltAndroidApp` 어노테이션을 추가한다. 매니페스트도 변경해준다.

```kotlin
@HiltAndroidApp
class ParkingSpotApplication : Application()
```

`di` 패키지를 생성한 후 Dependency Injection을 위한 Module을 생성 및 작성해준다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideParkingSpotDatabase(app: Application): ParkingSpotDatabase {
        return Room.databaseBuilder(
                app,
                ParkingSpotDatabase::class.java,
                "parking_spots.db"
        ).build()
    }

    @Provides
    @Singleton
    fun provideParkingSpotRepositry(db: ParkingSpotDatabase): ParkingSpotRepository {
        return ParkingSpotRepositoryImpl(db.dao)
    }
}
```

`MapEvent`에 2개의 이벤트를 추가로 작성해준다.

```kotlin
sealed class MapEvent {
    // ...
    data class OnMapLongClick(val latLng: LatLng) : MapEvent()
    data class OnInfoWindowLongClick(val spot: ParkingSpot) : MapEvent()
}
```

`MapState`에 `parkingSpots`을 추가해준다.

```kotlin
data class MapState(
        // ...
        val parkingSpots: List<ParkingSpot> = emptyList()
)
```

`MapViewModel`에 DI 및 이벤트 처리를 추가로 작성한다.

```kotlin
@HiltViewModel
class MapsViewModel @Inject constructor(
        private val repository: ParkingSpotRepository
) : ViewModel() {

    var state by mutableStateOf(MapState())

    init {
        viewModelScope.launch {
            repository.getParkingSpots().collectLatest { spots ->
                state = state.copy(
                        parkingSpots = spots
                )
            }
        }
    }

    fun onEvent(event: MapEvent) {
        when (event) {
            is MapEvent.ToggleFalloutMap -> {
                state = state.copy(
                        properties = state.properties.copy(
                                mapStyleOptions = if (state.isFalloutMap) {
                                    null
                                } else MapStyleOptions(MapStyle.json)
                        ),
                        isFalloutMap = !state.isFalloutMap
                )
            }
            is MapEvent.OnMapLongClick -> {
                viewModelScope.launch {
                    repository.insertParkingSpot(
                            ParkingSpot(
                                    event.latLng.latitude,
                                    event.latLng.longitude
                            )
                    )
                }
            }
            is MapEvent.OnInfoWindowLongClick -> {
                viewModelScope.launch {
                    repository.deleteParkingSpot(event.spot)
                }
            }
        }
    }
}
```

`MapScreen`에 클릭 관련 이벤트에 대해 처리해주는 로직을 구현한다.

```kotlin
@Composable
fun MapScreen(
        viewModel: MapsViewModel = androidx.lifecycle.viewmodel.compose.viewModel()
) {
    val scaffoldState = rememberScaffoldState()
    val uiSettings = remember { // remember를 사용해 recompose 시에도 MapUiSettings() 다시 생성하지 않도록 하기 위함
        MapUiSettings(zoomControlsEnabled = false)
    }
    Scaffold(
            scaffoldState = scaffoldState,
            floatingActionButton = {
                FloatingActionButton(onClick = {
                    viewModel.onEvent(MapEvent.ToggleFalloutMap)
                }) {
                    Icon(
                            imageVector = if (viewModel.state.isFalloutMap) {
                                Icons.Default.ToggleOff
                            } else Icons.Default.ToggleOn, contentDescription = "Toggle Fallout map"
                    )
                }
            }
    ) {
        GoogleMap(
                modifier = Modifier.fillMaxSize(),
                properties = viewModel.state.properties,
                uiSettings = uiSettings,
                onMapLongClick = {
                    viewModel.onEvent(MapEvent.OnMapLongClick(it))
                }
        ) {
            viewModel.state.parkingSpots.forEach { spot ->
                Marker(
                        position = LatLng(spot.lat, spot.lng),
                        title = "Parking spot (${spot.lat}, ${spot.lng})",
                        snippet = "Long click to delete",
                        onInfoWindowLongClick = {
                            viewModel.onEvent(
                                    MapEvent.OnInfoWindowLongClick(spot)
                            )
                        },
                        onClick = {
                            it.showInfoWindow()
                            true
                        },
                        icon = BitmapDescriptorFactory.defaultMarker(
                                BitmapDescriptorFactory.HUE_GREEN
                        )
                )
            }
        }
    }
}
```

`MainActivity`에 `@AndroidEntryPoint` 어노테이션을 추가하고 실행해보자.

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    // ...
}
```

<div align="center">
<img src="img/result2.gif" width="40%">
</div>

## References

* [How to Build an MVVM Parking Spot Finder with Maps Compose - Android Studio Tutorial](https://www.youtube.com/watch?v=0rc75uR0CNs&t=1s)