# Tracker Use Cases

이번엔 Tracker의 use case들을 만들어보자.

`tracker_domain` 모듈에 `use_case` 패키지를 생성한다.

### Search Food

`use_case` 패키지 하위에 `SearchFood` use case를 작성한다.

```kotlin
class SearchFood(
        private val repository: TrackerRepository
) {
    suspend operator fun invoke(
            query: String,
            page: Int = 1,
            pageSize: Int = 40
    ): Result<List<TrackableFood>> {
        if (query.isBlank()) {
            return Result.success(emptyList())
        }
        return repository.searchFood(query.trim(), page, pageSize)
    }
}
```

### Track Food

`SearchFood`를 복사해 `TrackFood`로 붙여넣는다. 그리고 `TrackFood` use case를 작성한다.

```kotlin
class TrackFood(
        private val repository: TrackerRepository
) {
    suspend operator fun invoke(
            food: TrackableFood,
            amount: Int,
            mealType: MealType,
            date: LocalDate
    ) {
        repository.insertTrackedFood(
                TrackedFood(
                        name = food.name,
                        carbs = ((food.carbsPer100g / 100f) * amount).roundToInt(),
                        protein = ((food.proteinPer100g / 100f) * amount).roundToInt(),
                        fat = ((food.fatPer100g / 100f) * amount).roundToInt(),
                        calories = ((food.caloriesPer100g / 100f) * amount).roundToInt(),
                        imageUrl = food.imageUrl,
                        mealType = mealType,
                        amount = amount,
                        date = date
                )
        )
    }
}
```

### Get Foods for Date

`GetFoodsForDate` use case를 작성한다.

```kotlin
class GetFoodsForDate(
        private val repository: TrackerRepository
) {
    operator fun invoke(date: LocalDate): Flow<List<TrackedFood>> {
        return repository.getFoodsForDate(date)
    }
}
```

### Delete Tracked Food

`DeleteTrackedFood` use case를 작성한다.

```kotlin
class DeleteTrackedFood(
        private val repository: TrackerRepository
) {
    suspend operator fun invoke(trackedFood: TrackedFood) {
        return repository.deleteTrackedFood(trackedFood)
    }
}
```

### Calculate Meal Nutrientes

`CalculateMealNutrients` use case를 작성한다.

```kotlin
class CalculateMealNutrients(
        private val preferences: Preferences
) {

    operator fun invoke(trackedFoods: List<TrackedFood>): Result {
        val allNutrients = trackedFoods
                .groupBy { it.mealType }
                .mapValues { entry ->
                    val type = entry.key
                    val foods = entry.value
                    MealNutrients(
                            carbs = foods.sumOf { it.carbs },
                            protein = foods.sumOf { it.protein },
                            fat = foods.sumOf { it.fat },
                            calories = foods.sumOf { it.calories },
                            mealType = type
                    )
                }
        val totalCarbs = allNutrients.values.sumOf { it.carbs }
        val totalProtein = allNutrients.values.sumOf { it.protein }
        val totalFat = allNutrients.values.sumOf { it.fat }
        val totalCalories = allNutrients.values.sumOf { it.calories }

        val userInfo = preferences.loadUserInfo()
        val caloryGoal = dailyCaloryRequirement(userInfo)
        val carbsGoal = (caloryGoal * userInfo.carbRatio / 4f).roundToInt()
        val proteinGoal = (caloryGoal * userInfo.proteinRatio / 4f).roundToInt()
        val fatGoal = (caloryGoal * userInfo.fatRatio / 9f).roundToInt()

        return Result(
                carbsGoal = carbsGoal,
                proteinGoal = proteinGoal,
                fatGoal = fatGoal,
                caloriesGoal = caloryGoal,
                totalCarbs = totalCarbs,
                totalProtein = totalProtein,
                totalFat = totalFat,
                totalCalories = totalCalories,
                mealNutrients = allNutrients
        )
    }

    private fun bmr(userInfo: UserInfo): Int {
        return when (userInfo.gender) {
            is Gender.Male -> {
                (66.47f + 13.75f * userInfo.weight +
                        5f * userInfo.height - 6.75f * userInfo.age).roundToInt()
            }
            is Gender.Female -> {
                (665.09f + 9.56f * userInfo.weight +
                        1.84f * userInfo.height - 4.67 * userInfo.age).roundToInt()
            }
        }
    }

    private fun dailyCaloryRequirement(userInfo: UserInfo): Int {
        val activityFactor = when (userInfo.activityLevel) {
            is ActivityLevel.Low -> 1.2f
            is ActivityLevel.Medium -> 1.3f
            is ActivityLevel.High -> 1.4f
        }
        val caloryExtra = when (userInfo.goalType) {
            is GoalType.LoseWeight -> -500
            is GoalType.KeepWeight -> 0
            is GoalType.GainWeight -> 500
        }
        return (bmr(userInfo) * activityFactor + caloryExtra).roundToInt()
    }

    data class MealNutrients(
            val carbs: Int,
            val protein: Int,
            val fat: Int,
            val calories: Int,
            val mealType: MealType
    )

    data class Result(
            val carbsGoal: Int,
            val proteinGoal: Int,
            val fatGoal: Int,
            val caloriesGoal: Int,
            val totalCarbs: Int,
            val totalProtein: Int,
            val totalFat: Int,
            val totalCalories: Int,
            // 각 MealType에 대한 영양 정보
            val mealNutrients: Map<MealType, MealNutrients>
    )
}
```

### Dependency Injection

지금까지 구현한 use case는 5가지이다. 이를 하나의 클래스를 통해 제공하기 위해 래퍼 클래스를 생성한다. `tracker_domain` 모듈의 `use_case` 패키지에 `TrackerUseCases`를
작성한다.

```kotlin
data class TrackerUseCases(
        val trackFood: TrackedFood,
        val searchFood: SearchFood,
        val getFoodsForDate: GetFoodsForDate,
        val deleteTrackedFood: DeleteTrackedFood,
        val calculateMealNutrients: CalculateMealNutrients
)
```

`tracker_domain` 모듈에 `di` 패키지를 생성하고 하위에 `TrackerDomainModule` 모듈을 작성한다.

```kotlin
@Module
@InstallIn(ViewModelComponent::class)
object TrackerDomainModule {

    @Provides
    @ViewModelScoped
    fun provideTrackerUseCases(
            repository: TrackerRepository,
            preferences: Preferences
    ): TrackerUseCases {
        return TrackerUseCases(
                trackFood = TrackFood(repository),
                searchFood = SearchFood(repository),
                getFoodsForDate = GetFoodsForDate(repository),
                deleteTrackedFood = DeleteTrackedFood(repository),
                calculateMealNutrients = CalculateMealNutrients(preferences)
        )
    }
}
```