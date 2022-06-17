# How to Implement Pagination With Jetpack Compose

필립은 페이징 라이브러리를 좋아하지 않는데, 왜냐하면 페이징 라이브러리를 이용해 리스트를 얻었을 때 단일 아이템을 변경하기 어렵기 때문이다. 따라서 페이징을 직접 구현해본다.

## Prerequisites

이 [레포지토리](https://github.com/philipplackner/ComposePagingYT)에서 프로젝트를 클론한다.

## Implementation

아이템을 가져오는 레포지토리가 이미 구현되어 있다.

```kotlin
class Repository {

    private val remoteDataSource = (1..100).map {
        ListItem(
            title = "Item $it",
            description = "Description $it"
        )
    }

    suspend fun getItems(page: Int, pageSize: Int): Result<List<ListItem>> {
        delay(2000L)
        val startingIndex = page * pageSize
        return if(startingIndex + pageSize <= remoteDataSource.size) {
            Result.success(
                remoteDataSource.slice(startingIndex until startingIndex + pageSize)
            )
        } else Result.success(emptyList())
    }
}
```

루트 패키지에 `Paginator` 인터페이스를 하나 생성해준다.

```kotlin
interface Paginator<Key, Item> {

    suspend fun loadNextItems()
    fun reset()
}
```

그리고 이 구현인 `DefaultPaginator` 클래스를 만들어준다.

```kotlin
class DefaultPaginator<Key, Item>(
    private val initialKey: Key,
    // inline : 컴파일 타임 최적화, 람다 함수를 직접 호출하는 곳에 넣어준다. -> 좀 더 메모리 효율이 좋다.
    private inline val onLoadUpdated: (Boolean) -> Unit,
    private inline val onRequest: suspend (nextKey: Key) -> Result<List<Item>>,
    private inline val getNextKey: suspend (List<Item>) -> Key,
    private inline val onError: suspend (Throwable?) -> Unit,
    private inline val onSuccess: suspend (items: List<Item>, nextKey: Key) -> Unit
) : Paginator<Key, Item> {

    private var currentKey: Key = initialKey
    private var isMakingRequest = false

    override suspend fun loadNextItems() {
        if (isMakingRequest) {
            return
        }
        isMakingRequest = true
        onLoadUpdated(true)
        val result = onRequest(currentKey)
        isMakingRequest = false
        val items = result.getOrElse {
            onError(it)
            onLoadUpdated(false)
            return
        }
        currentKey = getNextKey(items)
        onSuccess(items, currentKey)
        onLoadUpdated(false)
    }

    override fun reset() {
        currentKey = initialKey
    }
}
```

이제 `MainViewModel`을 생성 및 작성해준다.

```kotlin
class MainViewModel: ViewModel() {

    private val repository = Repository()

    var state by mutableStateOf(ScreenState())

    private val paginator = DefaultPaginator(
        initialKey = state.page,
        onLoadUpdated = {
            state = state.copy(isLoading = it)
        },
        onRequest = { nextPage ->
            repository.getItems(nextPage, 20)
        },
        getNextKey = {
            state.page + 1
        },
        onError = {
            state = state.copy(error = it?.localizedMessage)
        },
        onSuccess = { items, newKey ->
            state = state.copy(
                items = state.items + items,
                page = newKey,
                endReached = items.isEmpty()
            )
        }
    )

    init {
        loadNextItems()
    }

    fun loadNextItems() {
        viewModelScope.launch {
            paginator.loadNextItems()
        }
    }
}

data class ScreenState(
    val isLoading: Boolean = false,
    val items: List<ListItem> = emptyList(),
    val error: String? = null,
    val endReached: Boolean = false,
    val page: Int = 0
)
```

`MainActivity`를 다음과 같이 작성한다.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposePagingYTTheme {
                val viewModel = viewModel<MainViewModel>()
                val state = viewModel.state
                LazyColumn(
                    modifier = Modifier.fillMaxSize()
                ) {
                    items(state.items.size) { i ->
                        val item = state.items[i]
                        // 마지막 아이템까지 로드되었는지 확인하고, 이를 통해 추가 데이터를 로드하도록 구현한다. 
                        if (i >= state.items.size - 1 && !state.endReached && !state.isLoading) {
                            viewModel.loadNextItems()
                        }
                        Column(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(16.dp)
                        ) {
                            Text(
                                text = item.title,
                                fontSize = 20.sp,
                                color = Color.Black
                            )
                            Spacer(modifier = Modifier.height(8.dp))
                            Text(item.description)
                        }
                    }
                    item {
                        if (state.isLoading) {
                            Row(
                                modifier = Modifier
                                    .fillMaxWidth()
                                    .padding(8.dp),
                                horizontalArrangement = Arrangement.Center
                            ) {
                                CircularProgressIndicator()
                            }
                        }
                    }
                }
            }
        }
    }
}
```

<div align="center">
<img src="img/result.gif" width="40%">
</div>

## References

* [How to Implement Pagination With Jetpack Compose - Android Studio Tutorial](https://www.youtube.com/watch?v=D6Eus3f6U9I)