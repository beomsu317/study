# Bug Fixes

이번엔 지금까지 구현하며 존재했던 자잘한 이슈들을 수정해볼 것이다. 다음은 수정해야할 이슈들이다.

1. Search UI에서 hint가 보여지지 않는 이슈

- `SearchScreen`에서 `SearchTextField` 호출 시 `shouldShowHint` 파라미터를 추가한다.

```kotlin
@ExperimentalComposeUiApi
@Composable
fun SearchScreen(
        scaffoldState: ScaffoldState,
        mealName: String,
        dayOfMonth: Int,
        month: Int,
        year: Int,
        onNavigateUp: () -> Unit,
        viewModel: SearchViewModel = hiltViewModel()
) {
    // ...
    Column(
            modifier = Modifier
                    .fillMaxSize()
                    .padding(spacing.spaceMedium)
    ) {
        // ...
        SearchTextField(
                text = state.query,
                onValueChange = {
                    viewModel.onEvent(SearchEvent.OnQueryChange(it))
                },
                shouldShowHint = state.isHintVisible,  // 추가
                onSearch = {
                    keyboardController?.hide()
                    viewModel.onEvent(SearchEvent.OnSearch)
                },
                onFocusChanged = {
                    viewModel.onEvent(SearchEvent.OnSearchFocusChange(it.isFocused))
                }
        )
        // ...
    }
}
```

2. 키보드가 자동으로 없어지지 않는 이슈

- `tracker_presentation` 모듈의 `SearchScreen`에 다음과 같이 `keyboardController?.hide()`를 추가해준다.

 ```kotlin
 @ExperimentalComposeUiApi
@Composable
fun SearchScreen(
        scaffoldState: ScaffoldState,
        mealName: String,
        dayOfMonth: Int,
        month: Int,
        year: Int,
        onNavigateUp: () -> Unit,
        viewModel: SearchViewModel = hiltViewModel()
) {
    // ... 
    Column(
            modifier = Modifier
                    .fillMaxSize()
                    .padding(spacing.spaceMedium)
    ) {
        // ...
        SearchTextField(
                text = state.query,
                onValueChange = {
                    viewModel.onEvent(SearchEvent.OnQueryChange(it))
                },
                onSearch = {
                    keyboardController?.hide() // 추가
                    viewModel.onEvent(SearchEvent.OnSearch)
                },
                onFocusChanged = {
                    viewModel.onEvent(SearchEvent.OnSearchFocusChange(it.isFocused))
                }
        )
        // ...
        LazyColumn(modifier = Modifier.fillMaxSize()) {
            items(state.trackableFood) { food ->
                TrackableFoodItem(
                        trackableFoodUiState = food,
                        onClick = {
                            viewModel.onEvent(SearchEvent.OnToggleTrackableFood(food.food))
                        },
                        onAmountChange = {
                            viewModel.onEvent(
                                    SearchEvent.OnAmountForFoodChange(
                                            food.food, it
                                    )
                            )
                        },
                        onTrack = {
                            keyboardController?.hide() // 추가
                            viewModel.onEvent(
                                    SearchEvent.OnTrackFoodClick(
                                            food = food.food,
                                            mealType = MealType.fromString(mealName),
                                            date = LocalDate.of(year, month, dayOfMonth)
                                    )
                            )
                        },
                        modifier = Modifier.fillMaxWidth()
                )
            }
        }
    }
    // ...
}
 ```

3. 음식을 track 하기위해 amount를 입력할 때 숫자 키보드가 아닌 이슈

- `TrackableFoodItem`에서 amount를 보여주는 `BasicTextField`의 `keyboardOptions`에 키보드 타입을 설정해준다.

```kotlin
@Composable
fun TrackableFoodItem(
        trackableFoodUiState: TrackableFoodUiState,
        onClick: () -> Unit,
        onAmountChange: (String) -> Unit,
        onTrack: () -> Unit,
        modifier: Modifier = Modifier
) {
    // ...
    Column(
            // ...
    ) {
        // ...
        AnimatedVisibility(visible = trackableFoodUiState.isExpanded) {
            Row(
                    // ...
            ) {
                Row {
                    BasicTextField(
                            value = trackableFoodUiState.amount,
                            onValueChange = onAmountChange,
                            keyboardOptions = KeyboardOptions(
                                    imeAction = if (trackableFoodUiState.amount.isNotBlank()) {
                                        ImeAction.Done
                                    } else ImeAction.Default,
                                    keyboardType = KeyboardType.Number // 추가
                            ),
                            // ...
                    )
                    // ...
                }
                // ...
            }
        }
    }
}
```

4. 음식 track 후 Refresh 해야 track 된 음식이 보여지는 이슈

- `TrackerOverviewViewModel`의 `init` 블럭에 `refreshFoods()`를 추가한다.

```kotlin
@HiltViewModel
class TrackerOverviewViewModel @Inject constructor(preferences: Preferences, private val trackerUseCases: TrackerUseCases
) : ViewModel() {

    // ...
    init {
        refreshFoods() // 추가
        preferences.saveShouldhowOnBoarding(false)
    }
// ...
}
```

1. 매번 onboarding 보여지는 이슈

- `app` 모듈의 `MainActivity`에서

```kotlin
@ExperimentalComposeUiApi
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    @Inject
    lateinit var preferences: Preferences // 추가

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val shouldShowOnBoarding = preferences.loadShouldShowOnBoarding() // 추가
        setContent {
            CaloryTrackerTheme {
                // ...
                Scaffold(
                        modifier = Modifier.fillMaxSize(),
                        scaffoldState = scaffoldState
                ) {
                    NavHost(
                            navController = navController,
                            // 추가
                            startDestination = if (shouldShowOnBoarding) {
                                Route.WELCOME
                            } else Route.TRACKER_OVERVIEW
                    ) {
                        // ...
                    }
                }
            }
        }
    }
}
```

1. 아침에 먹은 음식이 점심, 저녁, 스낵에도 보여지는 이슈

- `TrackerOverviewScreen`에 `mealType`을 확인 후 동일한 `mealType`이면 보여주도록 수정한다.

```kotlin
@Composable
fun TrackerOverviewScreen(
        onNavigate: (UiEvent.Navigate) -> Unit,
        viewModel: TrackerOverviewViewModel = hiltViewModel()
) {
    // ...

    LazyColumn(
            // ...
    ) {
        // ...
        items(state.meals) { meal ->
            ExpendableMeal(
                    meal = meal,
                    onToggleClick = {
                        viewModel.onEvent(TrackerOverviewEvent.OnToggleMealClick(meal))
                    },
                    content = {
                        Column(
                                // ...
                        ) {
                            val foods = state.trackedFoods.filter {
                                it.mealType == meal.mealType
                            }
                            foods.forEach { food ->
                                TrackedFoodItem(
                                        trackedFood = food,
                                        onDeleteClick = {
                                            viewModel.onEvent(
                                                    TrackerOverviewEvent.OnDeleteTrackedFoodClick(food)
                                            )
                                        }
                                )
                                Spacer(modifier = Modifier.height(spacing.spaceMedium))
                            }
                        }
                    },
                    // ...
            )
        }
    }
}
```

<div align="center" class="row">
<img src="img/part-25/result1.gif" width="40%">
<img src="img/part-25/result2.gif" width="40%">
</div>



