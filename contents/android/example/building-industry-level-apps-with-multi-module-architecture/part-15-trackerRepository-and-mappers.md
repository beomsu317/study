# TrackerRepository & Mappers

이번엔 Repository와 tracker 기능을 위한 Mapper를 만들어보자.

`repository` 패키지를 생성하고 하위에 `TrackerRepository`를 만들자.

```kotlin
interface TrackerRepository {

    suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<Product>>
}
```

domain 레이어는 제일 안쪽에 있는 레이어로 data 레이어 또는 presentation 레이어에 접근할 수 없다. 만약 use case에서 repository를 사용할 때 repository 인터페이스 없이
직접적으로 domain 레이어의 구현체에 접근하는 경우 클린 아키텍처 가이드라인에 위반된다.

따라서 위와 같이 구현하는 경우 data 레이어의 `Product`에 접근할 수 없다.

domain model을 생성하여 이 문제를 해결할 수 있다. 이는 어떤 위치에서도 접근할 수 있는 객체에 대한 래퍼 클래스이다. `model` 패키지를 생성한 후 `TrackableFood` data class를
생성하고 작성한다.

```kotlin
data class TrackableFood(
        val name: String,
        val imageUrl: String?,
        val caloriesPer100g: Int,
        val carbsPer100g: Int,
        val proteinPer100g: Int,
        val fatPer100g: Int
)
```

그 다음 `TrackerRepository`를 다음과 같이 변경한다.

```kotlin
interface TrackerRepository {

    suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<TrackableFood>>
}
```

`model` 패키지 하위에 `MealType` sealed class를 생성 후 작성한다.

```kotlin
sealed class MealType(val name: String) {
    object Breakfast : MealType("breakfast")
    object Lunch : MealType("lunch")
    object Dinner : MealType("dinner")
    object Snack : MealType("snack")

    companion object {
        fun fromString(name: String): MealType {
            return when (name) {
                "breakfast" -> Breakfast
                "lunch" -> Lunch
                "dinner" -> Dinner
                "snack" -> Snack
                else -> Breakfast
            }
        }
    }
}

```

`model` 패키지 하위에 `TrackedFood`를 생성 후 작성한다.

```kotlin
data class TrackedFood(
        val name: String,
        val carbs: Int,
        val protein: Int,
        val fat: Int,
        val imageUrl: String?,
        val mealType: MealType,
        val amount: Int,
        val date: LocalDate,
        val calories: Int,
        val id: Int? = null
)
```

이제 `TrackerRepository`에서 `TrackedFood` data class를 사용할 수 있다. `getFoodsForDate`에서 `Flow`를 반환해야 하기 때문에 coroutine 디펜던시를
추가해주어야 한다. `buildSrc`에 `Coroutines` object를 추가해준다.

```kotlin
object Coroutines {
    const val version = "1.6.0"
    const val coroutines = "org.jetbrains.kotlinx:kotlinx-coroutines-core:$version"
}
```

그 다음 `tracker_domain` 모듈의 gradle에 코루틴 디펜던시를 추가해준다.

```kotlin
apply {
    from("$rootDir/base-module.gradle")
}

dependencies {
    "implementation"(project(Modules.core))
    "implementation"(Coroutines.coroutines)
}
```

```kotlin
interface TrackerRepository {

    suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<TrackableFood>>

    suspend fun insertTrackedFood(food: TrackedFood)

    suspend fun deleteTrackedFood(food: TrackedFood)

    fun getFoodsForDate(localDate: LocalDate): Flow<List<TrackedFood>>
}
```

`TrackerRepository`를 인터페이스로 만들게 되면 `FakeRepository`를 제공할 수 있어 별다른 디비 구성 없이 테스팅을 효율적으로 수행할 수 있다.

이제 `tracker_data` 모듈에 `repository` 패키지를 생성한 후 다음과 같이 `searchFood`를 구현해준다.

```kotlin
class TrackerRepositoryImpl(
        private val dao: TrackerDao,
        private val api: OpenFoodApi
) : TrackerRepository {
    override suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<TrackableFood>> {
        return try {
            val searchDto = api.searchFood(
                    query = query,
                    page = page,
                    pageSize = pageSize
            )
            Result.success(searchDto.products)
        } catch (e: Exception) {
            e.printStackTrace()
            Result.failure(e)
        }
    }
    // ...
}
```

위 코드의 `Result.success(searchDto.products)`에서 에러가 발생하게 되는데, `searchDto.products` 타입이 `List<TrackableFood>` 타입이 아니기 때문에
발생한다. 이를 해결하기 위해 Mapper를 작성해준다.

`tracker_data` 모듈에 `mapper` 패키지를 생성한 후 하위에 `Product.toTrackableFood` mapper를 작성한다.

```kotlin
fun Product.toTrackableFood(): TrackableFood? {
    val carbsPer100g = nutriments.carbohydrates100g.roundToInt()
    val proteinPer100g = nutriments.proteins100g.roundToInt()
    val fatPer100g = nutriments.fat100g.roundToInt()
    val caloriesPer100g = nutriments.energyKcal100g.roundToInt()
    return TrackableFood(
            name = productName ?: return null,
            carbsPer100g = carbsPer100g,
            proteinPer100g = proteinPer100g,
            fatPer100g = fatPer100g,
            caloriesPer100g = caloriesPer100g,
            imageUrl = imageFrontThumbUrl
    )
}
```

이후 `toTrackableFood()`를 사용해 매핑함으로써 해결할 수 있다.

```kotlin
class TrackerRepositoryImpl(
        private val dao: TrackerDao,
        private val api: OpenFoodApi
) : TrackerRepository {
    override suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<TrackableFood>> {
        return try {
            val searchDto = api.searchFood(
                    query = query,
                    page = page,
                    pageSize = pageSize
            )
            Result.success(
                    searchDto.products.mapNotNull { it.toTrackableFood() }
            )
        } catch (e: Exception) {
            e.printStackTrace()
            Result.failure(e)
        }
    }
    // ...
}
```

`TrackedFood`도 동일하게 mapper가 필요하다. 따라서 `mapper` 패키지에 `TrackedFoodMapper`를 생성한 후 다음과 같이 작성한다.

```kotlin
fun TrackedFoodEntity.toTrackedFood(): TrackedFood {
    return TrackedFood(
            name = name,
            carbs = carbs,
            protein = protein,
            fat = fat,
            imageUrl = imageUrl,
            mealType = MealType.fromString(type),
            amount = amount,
            date = LocalDate.of(year, month, dayOfMonth),
            calories = calories,
            id = id
    )
}

// 디비에 저장하기 위한 매퍼
fun TrackedFood.toTrackedFoodEntity(): TrackedFoodEntity {
    return TrackedFoodEntity(
            name = name,
            carbs = carbs,
            protein = protein,
            fat = fat,
            imageUrl = imageUrl,
            type = mealType.name,
            amount = amount,
            dayOfMonth = date.dayOfMonth,
            month = date.monthValue,
            year = date.year,
            calories = calories,
            id = id
    )
}
```

이제 `TrackerRepositoryImpl`를 다음과 같이 구현해준다.

```kotlin
class TrackerRepositoryImpl(
        private val dao: TrackerDao,
        private val api: OpenFoodApi
) : TrackerRepository {
    override suspend fun searchFood(
            query: String,
            page: Int,
            pageSize: Int
    ): Result<List<TrackableFood>> {
        return try {
            val searchDto = api.searchFood(
                    query = query,
                    page = page,
                    pageSize = pageSize
            )
            Result.success(
                    searchDto.products.mapNotNull { it.toTrackableFood() }
            )
        } catch (e: Exception) {
            e.printStackTrace()
            Result.failure(e)
        }
    }

    override suspend fun insertTrackedFood(food: TrackedFood) {
        dao.insertTrackedFood(food.toTrackedFoodEntity())
    }

    override suspend fun deleteTrackedFood(food: TrackedFood) {
        dao.deleteTrackedFood(food.toTrackedFoodEntity())
    }

    override fun getFoodsForDate(localDate: LocalDate): Flow<List<TrackedFood>> {
        return dao.getFoodsForDate(
                day = localDate.dayOfMonth,
                month = localDate.monthValue,
                year = localDate.year
        ).map { entities ->
            entities.map { it.toTrackedFood() }
        }
    }
}
```

`TrackerDataModule`에서 `TrackerRepository`를 제공하는 함수를 작성해준다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object TrackerDataModule {

    // ...
    @Provides
    @Singleton
    fun provideTrackerRepository(
            api: OpenFoodApi,
            db: TrackerDatabase
    ): TrackerRepository {
        return TrackerRepositoryImpl(
                dao = db.dao,   // dao를 직접 전달하지 않는 이유는 테스트 디비를 전달해 테스팅을 쉽게 수행할 수 있기 떄문
                api = api
        )
    }
}
```