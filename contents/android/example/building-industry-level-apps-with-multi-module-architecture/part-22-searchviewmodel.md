# SearchViewModel

이번엔 음식을 검색하는 Search UI에 필요한 `SearchViewModel`을 작성해보자. Search UI는 다음과 같다. 음식을 토글하면 다음과 같이 음식을 얼마나 먹었는지 track 할 수 있는 UI가
보여진다.

<div align="center">
<img src="img/part-22/track_ui.gif" width="40%">
</div>

`tracker_presentation` 모듈에 `search` 패키지를 생성한 후 `SearchEvent` sealed class를 작성해준다.

```kotlin
sealed class SearchEvent {
    data class OnQueryChange(val query: String) : SearchEvent()
    object OnSearch : SearchEvent()
    data class OnToggleTrackableFood(val food: TrackableFood) : SearchEvent()
    data class OnAmountForFoodChange(
            val food: TrackableFood,
            val amount: String
    ) : SearchEvent()
    data class OnTrackFoodClick(
            val food: TrackableFood,
            val mealType: MealType,
            val date: LocalDate
    ) : SearchEvent()
    data class OnSearchFocusChange(val isFocused: Boolean) : SearchEvent()
}
```

`search` 패키지에서 domain의 `TrackableFood`를 직접 사용하는 것이 아닌 Search UI에 사용될 `TrackableFoodUiState` data class 별도로 작성한다.

```kotlin
data class TrackableFoodUiState(
        val food: TrackableFood,
        val isExpanded: Boolean = false,
        val amount: String = ""
)
```

`search` 패키지에 `SearchState` data class를 작성한다.

```kotlin
data class SearchState(
        val query: String = "",
        val isHintVisible: Boolean = false,
        val isSearching: Boolean = false,
        val trackableFood: List<TrackableFoodUiState> = emptyList()
)
```

이제 위 구현을 기반으로 `search` 패키지에 `SearchViewModel`을 작성한다.

```kotlin
@HiltViewModel
class SearchViewModel @Inject constructor(
        private val trackerUseCases: TrackerUseCases,
        private val filterOutDigits: FilterOutDigits
) : ViewModel() {

    var state by mutableStateOf(SearchState())
        private set

    private val _uiEvent = Channel<UiEvent> { }
    val uiEvent = _uiEvent.receiveAsFlow()

    fun onEvent(event: SearchEvent) {
        when (event) {
            is SearchEvent.OnQueryChange -> {
                state = state.copy(query = event.query)
            }
            is SearchEvent.OnAmountForFoodChange -> {
                state = state.copy(
                        trackableFood = state.trackableFood.map {
                            if (it.food == event.food) {
                                it.copy(amount = filterOutDigits(event.amount))
                            } else it
                        }
                )
            }
            is SearchEvent.OnSearch -> {
                executeSearch()
            }
            is SearchEvent.OnToggleTrackableFood -> {
                state = state.copy(
                        trackableFood = state.trackableFood.map {
                            if (it.food == event.food) {
                                it.copy(isExpanded = !it.isExpanded)
                            } else it
                        }
                )
            }
            is SearchEvent.OnSearchFocusChange -> {
                state = state.copy(
                        isHintVisible = !event.isFocused && state.query.isBlank()
                )
            }
            is SearchEvent.OnTrackFoodClick -> {
                trackFood(event)
            }
        }
    }

    private fun executeSearch() {
        viewModelScope.launch {
            state = state.copy(
                    isSearching = true,
                    trackableFood = emptyList()
            )
            trackerUseCases
                    .searchFood(state.query)
                    .onSuccess { foods ->
                        state = state.copy(
                                trackableFood = foods.map {
                                    TrackableFoodUiState(it)
                                },
                                isSearching = false,
                                query = ""
                        )
                    }
                    .onFailure {
                        state = state.copy(isSearching = false)
                        _uiEvent.send(
                                UiEvent.ShowSnackbar(
                                        UiText.StringResource(R.string.error_something_went_wrong)
                                )
                        )
                    }
        }
    }

    private fun trackFood(event: SearchEvent.OnTrackFoodClick) {
        viewModelScope.launch {
            val uiState = state.trackableFood.find { it.food == event.food }
            trackerUseCases.trackFood(
                    food = uiState?.food ?: return@launch,
                    amount = uiState.amount.toIntOrNull() ?: return@launch,
                    mealType = event.mealType,
                    date = event.date
            )
            _uiEvent.send(UiEvent.NavigateUp)
        }
    }
}
```