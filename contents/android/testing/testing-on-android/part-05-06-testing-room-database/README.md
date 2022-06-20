# Testing Room Database

## Setup Project

다음 레포지토리를 클론한다.

* [GitHub - philipplackner/ShoppingListTestingYT at TestingRoomDB](https://github.com/philipplackner/ShoppingListTestingYT/tree/TestingRoomDB)

## Setup Room DB

### Setup Item

Room DB에 사용될 `ShoppingItem` 데이터 클래스를 생성한다.

```kotlin
@Entity(tableName = "shopping_items")
data class ShoppingItem(
    var name: String,
    var amount: Int,
    var price: Float,
    var imageUrl: String,
    @PrimaryKey(autoGenerate = true)
    val id: Int? = null
)
```

### Setup DAO

Room DB를 위한 `ShoppingDao`를 생성한다.

```kotlin
@Dao
interface ShoppingDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertShoppingItem(shoppingItem: ShoppingItem)

    @Delete
    suspend fun deleteShoppingItem(shoppingItem: ShoppingItem)

    @Query("SELECT * FROM shopping_items")
    fun observeAllShoppingItems(): LiveData<List<ShoppingItem>>

    @Query("SELECT SUM(price * amount) FROM shopping_items")
    fun observeTotalPrice(): LiveData<Float>
}
```

### Setup DB

Room DB를 생성한다.

```kotlin
@Database(
    entities = [ShoppingItem::class],
    version = 1
)
abstract class ShoppingItemDatabase : RoomDatabase() {

    abstract fun shoppingDao(): ShoppingDao
}
```

## Testing Room Database

테스트가 안드로이드 에뮬레이터 또는 기기에서 돌아가고, instrumenet test인 것을 알려주기 위해 `@RunWith(AndroidJUnit4::class)` 어노테이션을 사용한다. 이 클래스에 작성될 테스트가 unit 테스트라는 것을 알려주기 위해 `@SamllTest` 어노테이션 사용하며, integrated test인 경우 `@MediumTest` 어노테이션, UI 테스트의 경우 `@LargeTest` 어노테이션을 사용한다.

`@Before`에 Room DB를 생성한다. `inMemoryDatabaseBuilder` 빌더를 사용해 저장되는 데이터베이스가 아니고 메모리에만 존재하는 데이터베이스를 생성한다. 또한 메인 스레드에서 접근할 수 있도록 `allowMainThreadQueries()`를 추가한다.

```kotlin
// runBlockingTest가 실험적인 테스트 스코프이기 떄문에 워닝을 없애기 위해 다음 주석 사용
@ExperimentalCoroutinesApi
@RunWith(AndroidJUnit4::class)
@SmallTest
class ShoppingDaoTest {

    private lateinit var database: ShoppingItemDatabase
    private lateinit var dao: ShoppingDao

    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            ShoppingItemDatabase::class.java
        ).allowMainThreadQueries().build()
        dao = database.shoppingDao()
    }

    @After
    fun teardown() {
        database.close()
    }
}
```

### Insert item test

`dao`의 `insertShoppingItem()` 함수가 suspend 함수이기 때문에 scope를 지정해준다. `runBlockingTest`는 테스트에 최적화된 scope이며 최적화를 위해 `delay` 함수를 스킵한다.

삽입한 아이템이 정상적으로 DB에 있는지 확인하기 위해 `dao.observeAllShoppingItems()`를 호출하여 `shoppingItem`을 포함하고 있는지 확인한다.

```kotlin
@Test
fun insertShoppingItem() = runBlockingTest {
		val shoppingItem = ShoppingItem("name", 1, 1f, "url", id = 1)
		dao.insertShoppingItem(shoppingItem)

    val allShoppingItems = dao.observeAllShoppingItems().getOrAwaitValue()
    assertThat(allShoppingItems).contains(shoppingItem)
}
```

`.getOrAwaitValue()`는 LiveData가 값을 반환할 때까지 기다려주는 함수이다.

```kotlin
/**
 * Gets the value of a [LiveData] or waits for it to have one, with a timeout.
 *
 * Use this extension from host-side (JVM) tests. It's recommended to use it alongside
 * `InstantTaskExecutorRule` or a similar mechanism to execute tasks synchronously.
 */
@VisibleForTesting(otherwise = VisibleForTesting.NONE)
fun <T> LiveData<T>.getOrAwaitValue(
    time: Long = 2,
    timeUnit: TimeUnit = TimeUnit.SECONDS,
    afterObserve: () -> Unit = {}
): T {
    var data: T? = null
    val latch = CountDownLatch(1)
    val observer = object : Observer<T> {
        override fun onChanged(o: T?) {
            data = o
            latch.countDown()
            this@getOrAwaitValue.removeObserver(this)
        }
    }
    this.observeForever(observer)

    try {
        afterObserve.invoke()

        // Don't wait indefinitely if the LiveData is not set.
        if (!latch.await(time, timeUnit)) {
            throw TimeoutException("LiveData value was never set.")
        }

    } finally {
        this.removeObserver(observer)
    }

    @Suppress("UNCHECKED_CAST")
    return data as T
}
```

위 테스트를 실행하게 되면, 다음 에러가 발생하게 된다. 이는 테스트 케이스에서 LiveData의 값을 받기 전 테스트가 종료되는 문제이다.

```
java.lang.IllegalStateException: This job has not completed yet
```

이 때 `InstantTaskExecutorRule()`을 사용해 테스트가 백그라운드에서가 아닌 동일한 스레드에서 동작하게 하여 동기적인 처리가 가능하도록 해준다.

```kotlin
class ShoppingDaoTest {

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    private lateinit var database: ShoppingItemDatabase
    private lateinit var dao: ShoppingDao
    // ...
}
```

### Delete item test

테스트가 독립적으로 동작하기 때문에 동일한 `ShoppingItem`을 넣어줘도 괜찮다. 아이템 삽입 후 삭제하고 해당 아이템이 포함되지 않는지 검증하여 확인한다.

```kotlin
@Test
    fun deleteShoppingItem() = runBlockingTest {
        val shoppingItem = ShoppingItem("name", 1, 1f, "url", id = 1)
        dao.insertShoppingItem(shoppingItem)
        dao.deleteShoppingItem(shoppingItem)

        val allShoppingItems = dao.observeAllShoppingItems().getOrAwaitValue()
        assertThat(allShoppingItems).doesNotContain(shoppingItem)
    }
```

### Price sum test

모든 가격을 더하는 테스트를 수행하기 위해 3개의 아이템을 넣고 예측 결과 값과 동일한지 검증한다.

```kotlin
@Test
    fun observeTotalPriceSum() = runBlockingTest {
        val shoppingItem1 = ShoppingItem("name", 2, 10f, "url", id = 1)
        val shoppingItem2 = ShoppingItem("name", 4, 5.5f, "url", id = 2)
        val shoppingItem3 = ShoppingItem("name", 0, 100f, "url", id = 3)

        dao.insertShoppingItem(shoppingItem1)
        dao.insertShoppingItem(shoppingItem2)
        dao.insertShoppingItem(shoppingItem3)

        val totalPriceSum = dao.observeTotalPrice().getOrAwaitValue()

        assertThat(totalPriceSum).isEqualTo(2 * 10f + 4 * 5.5f)
    }
```

## References

* [Setting up Project & Room DB - Testing on Android - Part 5](https://www.youtube.com/watch?v=2p6cfaIK3_g&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=5)