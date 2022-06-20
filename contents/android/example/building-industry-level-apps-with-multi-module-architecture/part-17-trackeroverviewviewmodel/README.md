# TrackerOverviewViewModel

이번엔 `TrackerOverviewViewModel`을 작성해보자.

`tracker_presentation` 모듈에 `tracker_overview` 패키지를 생성한 후 이벤트 처리를 위한 `TrackerOverviewEvent`를 작성한다.

```kotlin
sealed class TrackerOverviewEvent {
    object OnNextDayClick : TrackerOverviewEvent()
    object OnPreviousDayClick : TrackerOverviewEvent()
    data class OnToggleMealClick(val meal: Meal) : TrackerOverviewEvent()
    data class OnDeleteTrackedFoodClick(val trackedFood: TrackedFood) : TrackerOverviewEvent()
    data class OnAddFoodClick(val meal: Meal) : TrackerOverviewEvent()
}
```

동일한 패키지에 위에 정의되지 않은 `Meal` 클래스를 작성한다.

```kotlin
data class Meal(
        val name: UiText,
        @DrawableRes val drawableRes: Int,
        val mealType: MealType,
        val carbs: Int = 0,
        val protein: Int = 0,
        val fat: Int = 0,
        val calories: Int = 0,
        val isExpanded: Boolean = false
)

val defaultMeals = listOf(
        Meal(
                name = UiText.StringResource(R.string.breakfast),
                drawableRes = R.drawable.ic_breakfast,
                mealType = MealType.Breakfast
        ),
        Meal(
                name = UiText.StringResource(R.string.lunch),
                drawableRes = R.drawable.ic_lunch,
                mealType = MealType.Lunch
        ),
        Meal(
                name = UiText.StringResource(R.string.dinner),
                drawableRes = R.drawable.ic_dinner,
                mealType = MealType.Dinner
        ),
        Meal(
                name = UiText.StringResource(R.string.snacks),
                drawableRes = R.drawable.ic_snack,
                mealType = MealType.Snack
        )
)
```

왜 `TrackerFood`는 domain 모듈에 있고 `Meal` 클래스는 presentation 모듈에 있을까? 간단하게 UI state를 처리하기 위함이다. 여기서 사용되는 `Meal` UI에 직접 반영하여
화면에 보여주기 위해 존재한다. 이 클래스는 data나 domain 모듈에 필요하지 않다.

`MealType`에 대한 이미지를 보여주기 위해 `core` 모듈의 `res/drawable`
디렉토리에 [여기](https://github.com/philipplackner/CalorieTracker/tree/Part16/TrackerOverviewViewModel/core/src/main/res/drawable)
5개의 drawable을 추가한다. `tracker_presentation`에서만 사용되지만 리소스의 경우 `core`에 모아놓는 것을 선호하므로 혼란스러워 하지 말아라.

동일한 패키지에 State를 나타내기 위한 `TrackerOverviewState` data class를 생성하고 작성한다.

```kotlin
data class TrackerOverviewState(
        val totalCarbs: Int = 0,
        val totalProtein: Int = 0,
        val totalFat: Int = 0,
        val totalCalories: Int = 0,
        val carbsGoal: Int = 0,
        val proteinGoal: Int = 0,
        val fatGoal: Int = 0,
        val caloriesGoal: Int = 0,
        val date: LocalDate = LocalDate.now(),
        val trackedFoods: List<TrackedFood> = emptyList(),
        val meals: List<Meal> = defaultMeals
)
```

`core` 모듈의 `domain/preferences/Preferences` 인터페이스에 다음을 추가한다.

```kotlin
interface Preferences {

    // ...
    fun saveShouldShowOnBoarding(shouldShow: Boolean)
    fun loadShouldShowOnBoarding(): Boolean

    companion object {
        // ...
        const val KEY_SHOULD_SHOW_ONBOARDING = "should_show_onboarding"
    }

}
```

그리고 `core` 모듈에서 `Preferences`의 구현체인 `data/preferences/DefaultPreferences`에 추가된 함수를 구현해준다.

```kotlin
class DefaultPreferences(
        private val sharedPref: SharedPreferences
) : Preferences {

    // ...
    override fun saveShouldShowOnBoarding(shouldShow: Boolean) {
        sharedPref.edit()
                .putBoolean(Preferences.KEY_SHOULD_SHOW_ONBOARDING, shouldShow)
                .apply()
    }

    override fun loadShouldShowOnBoarding(): Boolean {
        return sharedPref.getBoolean(
                Preferences.KEY_SHOULD_SHOW_ONBOARDING,
                true
        )
    }
}
```

이제 동일 패키지에 `TrackerOverviewViewModel`을 생성 후 작성한다.

```kotlin
@HiltViewModel
class TrackerOverviewViewModel @Inject constructor(
        preferences: Preferences,
        private val trackerUseCases: TrackerUseCases
) : ViewModel() {

    var state by mutableStateOf(TrackerOverviewState())
        private set

    private val _uiEvent = Channel<UiEvent>()
    val uiEvent = _uiEvent.receiveAsFlow()

    private var getFoodsForDateJob: Job? = null

    init {
        preferences.saveShouldShowOnBoarding(false)
    }

    fun onEvent(event: TrackerOverviewEvent) {
        when (event) {
            is TrackerOverviewEvent.OnAddFoodClick -> {
                viewModelScope.launch {
                    _uiEvent.send(
                            UiEvent.Navigate(
                                    route = Route.SEARCH
                                            + "/${event.meal.mealType.name}"
                                            + "/${state.date.dayOfMonth}"
                                            + "/${state.date.monthValue}"
                                            + "/${state.date.year}"
                            )
                    )
                }
            }
            is TrackerOverviewEvent.OnDeleteTrackedFoodClick -> {
                viewModelScope.launch {
                    trackerUseCases.deleteTrackedFood(event.trackedFood)
                    refreshFoods()
                }
            }
            is TrackerOverviewEvent.OnNextDayClick -> {
                state = state.copy(
                        date = state.date.plusDays(1)
                )
                refreshFoods()
            }
            is TrackerOverviewEvent.OnPreviousDayClick -> {
                state = state.copy(
                        date = state.date.minusDays(1)
                )
                refreshFoods()
            }
            is TrackerOverviewEvent.OnToggleMealClick -> {
                state = state.copy(
                        meals = state.meals.map {
                            if (it.name == event.meal.name) {
                                it.copy(isExpanded = !it.isExpanded)
                            } else it
                        }
                )
            }
        }
    }

    private fun refreshFoods() {
        getFoodsForDateJob?.cancel()
        getFoodsForDateJob = trackerUseCases
                .getFoodsForDate(state.date)
                .onEach { foods ->
                    val nutrientsResult = trackerUseCases.calculateMealNutrients(foods)
                    state = state.copy(
                            totalCarbs = nutrientsResult.totalCarbs,
                            totalProtein = nutrientsResult.totalProtein,
                            totalFat = nutrientsResult.totalFat,
                            totalCalories = nutrientsResult.totalCalories,
                            carbsGoal = nutrientsResult.carbsGoal,
                            proteinGoal = nutrientsResult.proteinGoal,
                            fatGoal = nutrientsResult.fatGoal,
                            caloriesGoal = nutrientsResult.caloriesGoal,
                            trackedFoods = foods,
                            meals = state.meals.map {
                                val nutrientsForMeal =
                                        nutrientsResult.mealNutrients[it.mealType]
                                                ?: return@map it.copy(
                                                        carbs = 0,
                                                        protein = 0,
                                                        fat = 0,
                                                        calories = 0
                                                )
                                it.copy(
                                        carbs = nutrientsForMeal.carbs,
                                        protein = nutrientsForMeal.protein,
                                        fat = nutrientsForMeal.fat,
                                        calories = nutrientsForMeal.calories
                                )
                            }
                    )
                }
                .launchIn(viewModelScope)
    }
}
```