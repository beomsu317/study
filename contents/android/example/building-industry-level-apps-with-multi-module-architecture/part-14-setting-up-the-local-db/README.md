# Setting Up the Local DB

이번엔 아이템을 저장하기 위한 로컬 디비를 설정한다. 이는 `tracker_data` 모듈에서 작성된다.

`local/entity` 패키지를 생성하고 디비에 저장하기 위한 `TrackedFoodEntity` data class를 작성해준다.

```kotlin
@Entity
data class TrackedFoodEntity(
        val name: String,
        val carbs: Int,
        val protein: Int,
        val fat: Int,
        val imageUrl: String?,
        val type: String,
        val amount: Int,
        val dayOfMonth: Int,
        val month: Int,
        val year: Int,
        val calories: Int,
        @PrimaryKey val id: Int? = null
)
```

그리고 `local` 패키지에 `TrackerDao`를 다음과 같이 작성한다.

```kotlin
@Dao
interface TrackerDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertTrackedFood(trackedFoodEntity: TrackedFoodEntity)

    @Delete
    suspend fun deleteTrackedFood(trackedFoodEntity: TrackedFoodEntity)

    @Query("""
        SELECT *
        FROM trackedfoodentity
        WHERE dayOfMonth = :day AND month = :month AND year = :year 
    """)
    fun getFoodsForDate(day: Int, month: Int, year: Int): Flow<List<TrackedFoodEntity>>
}
```

그 후 `local` 패키지에 `TrackerDatabase`를 다음과 같이 작성한다.

```kotlin
@Database(
        entities = [TrackedFoodEntity::class],
        version = 1
)
abstract class TrackerDatabase : RoomDatabase() {
    abstract val dao: TrackerDao
}
```

지금까지 구현한 디비를 사용하기 위해 `di` 패키지의 `TrackerDataModule`에 디비를 제공해주는 함수를 작성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object TrackerDataModule {

    // ...
    @Provides
    @Singleton
    fun provideTrackerDatabase(app: Application): TrackerDatabase {
        return Room.databaseBuilder(
                app,
                TrackerDatabase::class.java,
                "tracker_db"
        ).build()
    }
}
```