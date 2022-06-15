# Architectural Skeleton

3개의 프레그먼트를 만들 것이다. 첫 번째는 쇼핑 아이템을 보여주는 프레그먼트, 두 번째는 쇼핑 아이템을 추가하는 프레그먼트, 세 번째는 이미지를 선택하는 프레그먼트이다.

3개의 작은 프레그먼트를 사용하기 때문에 3개에 대한 하나의 ViewModel을 생성하여 사용한다. 대규모의 프로젝트에서는 프레그먼트 당 하나의 ViewModel을 생성해주어야 한다.

`ui` 디렉토리 하위에 3개의 프레그먼트(`ShoppingFragment`, `AddShoppingItemFragment`, `ImagePickFragment`)를 생성하고, 하나의 ViewModel을 생성한다. ViewModel은 `ShoppingRepository`를 injection 한다.

```kotlin
class ShoppingViewModel @ViewModelInject constructor(
    private val repository: ShoppingRepository
) : ViewModel() {
}
```

`other` 디렉토리 하위에 `Event`를 생성한다. 이는 LiveData의 값이 처리됐는지 아닌지 여부를 판별할 수 있는 클래스이다. LiveData가 1번 emit 되면 그 다음 emit 할 때는 `null`을 반환한다.

만약 네트워크에서 값을 받아와 LiveData에 error를 반환하였다고 가정하자. 그리고 화면 전환을 하게되면 LiveData는 다시 값을 emit 하여 error가 2번 처리될 수 있다. 이러한 경우들을 방지하기 위해 `Event` 클래스를 사용한다.

```kotlin
open class Event<out T>(private val content: T) {

    var hasBeenHandled = false
        private set // Allow external read but not write

    /**
     * Returns the content and prevents its use again.
     */
    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) {
            null
        } else {
            hasBeenHandled = true
            content
        }
    }

    /**
     * Returns the content, even if it's already been handled.
     */
    fun peekContent(): T = content
}
```

ViewModel을 다음과 같이 작성한다.

```kotlin
class ShoppingViewModel @ViewModelInject constructor(
    private val repository: ShoppingRepository
) : ViewModel() {

    val shoppingItems = repository.observeAllShoppingItems()

    val totalPrice = repository.observeTotalPrice()

    private val _images = MutableLiveData<Event<Resource<ImageResponse>>>()
    val images: LiveData<Event<Resource<ImageResponse>>> = _images

    private val _curImageUrl = MutableLiveData<String>()
    val curImageUrl: LiveData<String> = _curImageUrl

    private val _insertShoppingItemStatus = MutableLiveData<Event<Resource<ShoppingItem>>>()
    val insertShoppingItemStatus: LiveData<Event<Resource<ShoppingItem>>> = _insertShoppingItemStatus

    fun setCurrentImageUrl(url: String) {
        _curImageUrl.postValue(url)
    }

    fun deleteShoppingItem(shoppingItem: ShoppingItem) = viewModelScope.launch {
        repository.deleteShoppingItem(shoppingItem)
    }

    fun insertShoppingIntoDb(shoppingItem: ShoppingItem) = viewModelScope.launch {
        repository.insertShoppingItem(shoppingItem)
    }

    fun insertShoppingItem(name: String, amountString: String, priceString: String) {

    }

    fun searchForIamge(imageQuery: String) {

    }
}
```

`Constants`에 2개의 MAX_LENGTH를 선언한다.

```kotlin
object Constants {
    // ...
    const val MAX_NAME_LENGTH = 20
    const val MAX_PRICE_LENGTH = 10
}
```

그리고 `AppModule`에 `ShoppingRepository`를 싱글톤으로 제공해주는 함수를 생성한다.

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object AppModule {
		...
    @Singleton
    @Provides
    fun provideDeafultShoppingRepository(
        dao: ShoppingDao,
        api: PixabayAPI
    ) = DefaultShoppingRepository(dao, api) as ShoppingRepository
}
```

다음과 같이 `ShoppingFragment`, `AddShoppingItemFragment`, `ImagePickFragment`에 동일한 ViewModel 설정을 해준다.

```kotlin
class ShoppingFragment: Fragment(R.layout.fragment_shopping) {

    lateinit var viewModel: ShoppingViewModel

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel = ViewModelProvider(requireActivity()).get(ShoppingViewModel::class.java)
    }
}
```

## References

* [Architectural Skeleton - Testing on Android - Part 9](https://www.youtube.com/watch?v=x2WahC3N_Yw&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=9)