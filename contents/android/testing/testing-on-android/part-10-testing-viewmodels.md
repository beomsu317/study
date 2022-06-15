# Testing ViewModels

`androidTest`에 있던 `LiveDataUtilAndroidTest`를 복사해 `LiveDataUtilTest`로 변경해준다.

```kotlin
@VisibleForTesting(otherwise = VisibleForTesting.NONE)
fun <T> LiveData<T>.getOrAwaitValueTest(
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
            this@getOrAwaitValueTest.removeObserver(this)
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

다음과 같이 `ShoppingViewModelTest` 클래스를 생성한 후 각 테스트 케이스를 작성한다.

```kotlin
class ShoppingViewModelTest {

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    private lateinit var viewModel: ShoppingViewModel

    @Before
    fun setup() {
        viewModel = ShoppingViewModel(FakeShoppingRepository())
    }

    @Test
    fun `insert shopping item with empty field, returns error`() {
        viewModel.insertShoppingItem("name", "", "3.0")

        val value = viewModel.insertShoppingItemStatus.getOrAwaitValueTest()

        assertThat(value.getContentIfNotHandled()?.status).isEqualTo(Status.ERROR)
    }

    @Test
    fun `insert shopping item with too long name, returns error`() {
        val string = buildString {
            for (i in 1..Constants.MAX_NAME_LENGTH + 1) {
                append(1)
            }
        }
        viewModel.insertShoppingItem(string, "5", "3.0")

        val value = viewModel.insertShoppingItemStatus.getOrAwaitValueTest()

        assertThat(value.getContentIfNotHandled()?.status).isEqualTo(Status.ERROR)
    }

    @Test
    fun `insert shopping item with too long price, returns error`() {
        val string = buildString {
            for (i in 1..Constants.MAX_PRICE_LENGTH + 1) {
                append(1)
            }
        }
        viewModel.insertShoppingItem("name", "5", string)

        val value = viewModel.insertShoppingItemStatus.getOrAwaitValueTest()

        assertThat(value.getContentIfNotHandled()?.status).isEqualTo(Status.ERROR)
    }

    @Test
    fun `insert shopping item with too high amount, returns error`() {
        viewModel.insertShoppingItem("name", "999999999999999999", "3.0")

        val value = viewModel.insertShoppingItemStatus.getOrAwaitValueTest()

        assertThat(value.getContentIfNotHandled()?.status).isEqualTo(Status.ERROR)
    }

    @Test
    fun `insert shopping item with valid input, returns success`() {
        viewModel.insertShoppingItem("name", "5", "3.0")

        val value = viewModel.insertShoppingItemStatus.getOrAwaitValueTest()

        assertThat(value.getContentIfNotHandled()?.status).isEqualTo(Status.SUCCESS)
    }
}
```

`ShoppingViewModel`에 위 테스트 케이스에 맞게 함수를 작성해준다.

```kotlin
class ShoppingViewModel @ViewModelInject constructor(
    private val repository: ShoppingRepository
) : ViewModel() {
    // ... 

    fun insertShoppingItem(name: String, amountString: String, priceString: String) {
        if (name.isEmpty() || amountString.isEmpty() || priceString.isEmpty()) {
            _insertShoppingItemStatus.postValue(Event(Resource.error("The fileds must not be empty", null)))
            return
        }
        if (name.length > Constants.MAX_NAME_LENGTH) {
            _insertShoppingItemStatus.postValue(Event(Resource.error("The name of the item must not exceed ${Constants.MAX_NAME_LENGTH} characters", null)))
            return
        }
        if (priceString.length > Constants.MAX_PRICE_LENGTH) {
            _insertShoppingItemStatus.postValue(Event(Resource.error("The price of the item must not exceed ${Constants.MAX_PRICE_LENGTH} characters", null)))
            return
        }
        val amount = try {
            amountString.toInt()
        } catch (e: Exception) {
           _insertShoppingItemStatus.postValue(Event(Resource.error("Please enter a valid amount", null)))
           return
        }
        val shoppingItem = ShoppingItem(name, amount, priceString.toFloat(), _curImageUrl.value ?: "")
        insertShoppingIntoDb(shoppingItem)
        setCurrentImageUrl("")
        _insertShoppingItemStatus.postValue(Event(Resource.success(shoppingItem)))
    }

    fun searchForIamge(imageQuery: String) {
        if (imageQuery.isEmpty()) {
            return
        }
        // postValue 대신 value를 사용하면 모든 observer에게 값이 변경됨을 알린다. 
				// 만약 짧은 시간에 여러번 postValue를 사용하면 마지막 값만 알려주게 된다. 
				// 따라서 적절한 상태에 대해 알려주기 위해 사용한다.
        _images.value = Event(Resource.loading(null))
        viewModelScope.launch {
            val response = repository.searchForImage(imageQuery)
            _images.value = Event(response)
        }
    }
}
```

위 테스트 케이스를 실행하면 모두 정상적으로 동작하지만 `Module with the Main dispatcher had failed to initialize` 에러가 발생하게 된다. 이는 테스트 시 코루틴의 Main Dispatcher가 없어 발생하는 에러이다.

위 문제를 해결하기 위해 `TestRule`을 생성하여 처리할 수 있다. `MainCoroutineRule`을 생성고 `TestWatcher()`, `TestCoroutineScope`을 상속한다. `TestWatcher()`는 `TestRule`을 상속받는 클래스이다. 이 클래스를 통해 테스트 시작, 종료 시 Rule을 정할 수 있다.

```kotlin
@ExperimentalCoroutinesApi
class MainCoroutineRule(
    private val dispatcher: CoroutineDispatcher = TestCoroutineDispatcher()
): TestWatcher(), TestCoroutineScope by TestCoroutineScope(dispatcher) {

    override fun starting(description: Description?) {
        super.starting(description)
        // 테스트 시작 시 TestCoroutineDispatcher 설정
        Dispatchers.setMain(dispatcher)
    }

    override fun finished(description: Description?) {
        super.finished(description)
        cleanupTestCoroutines()
        // 테스트 종료 시 dispatcher reset
        Dispatchers.resetMain()
    }
}
```

`MainCoroutineRule()` 추가후 다시 테스트를 실행하면 아무 에러 없이 잘 실행되는 것을 확인할 수 있다.

```kotlin
@ExperimentalCoroutinesApi
class ShoppingViewModelTest {

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    @get:Rule
    var mainCoroutineRule = MainCoroutineRule()
    // ...
}
```

## References

* [Testing ViewModels - Testing in Android - Part 10](https://www.youtube.com/watch?v=B-dJTFeOAqw&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=10)