# How to Make Your Code Clean With Kotlin Sealed Classes

Sealed class를 통해 코드를 깔끔하게 작성하는 법을 알아보자.

우선 sealed class는 다른 언어에는 없는 특이한 클래스이다. enum 클래스와 비슷하다. 이를 우리가 설정한 값으로 표현할 수 있다. 다음과 같은 2개의 클래스를 보자.

```kotlin
// set of fixed values
enum class EnumCar {
    BMW,
    AUDI,
    MERCEDES
}

sealed class SealedCar {
    object BMW : SealedCar()
    object Audi : SealedCar()
    object Mercedes : SealedCar()
}
```

enum class는 각각 고정된 값을 할당한다. 상수가 BMW(0), AUDI(1), MERCEDES(2)로 표현되어 코드를 쉽게 읽을 수 있다.

sealed class는 내부의 class 또는 object는 부모의 sealed class(`SealedCar`)를 상속받아 구현된다.

enum class의 경우 각각 상수로 저장되기 때문에 쉽게 직렬화 또는 역직렬화가 가능하다. 이는 서버와 클라이언트 간 동일한 enum 클래스를 가지고 있는 경우 쉽게 데이터를 전달할 수 있다는 것을 의미한다.

sealed class는 generic type의 파라미터를 가질 수 있으며, instance specific data를 가질 수 있다. 만약 Car에 모델명을 가지고 있게 하고 싶은 경우 다음과 같이 구현할 수
있다.

```kotlin
sealed class SealedCar(val model: String) {
    object BMW : SealedCar("M5")
    // ...
}
```

이 방식은 enum class도 동일하게 적용할 수 있다.

```kotlin
enum class EnumCar(val model: String) {
    BMW("M5"),
    // ...
}
```

하지만 sealed class는 이를 data class로 만들 수 있다. 이는 instance specific data를 의미한다. 또한 클래스 내 구현이 가능하므로 enum class보다 더 유연하게 구현할 수
있다. enum class도 블럭을 추가하여 구현할 수 있지만 일반 클래스처럼 동작하지 않는다.

```kotlin
sealed class SealedCar(val model: String) {
    class BMW(model: String) : SealedCar(model) {
    }
    // ...
}
```

이제 실질적으로 sealed class가 어떻게 쓰이는지 확인해보자. 다음과 같이 `MainActivity`와 `MainRepository`가 있다고 가정하자.

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel: MainViewModel = viewModel()
            val uiState = viewModel.uiState.value
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                LazyColumn(modifier = Modifier.fillMaxSize()) {
                    items(uiState.items.size) {
                        Text(
                            text = uiState.items[it].name,
                            modifier = Modifier
                                .padding(15.dp)
                        )
                    }
                }

                if (uiState.isLoading) {
                    CircularProgressIndicator()
                }

                when (uiState.error) {
                    MainViewModel.UiState.Error.NetworkError -> {
                        Text(
                            text = stringResource(id = R.string.error_network)
                        )
                    }
                    MainViewModel.UiState.Error.InputEmpty -> {

                    }
                    MainViewModel.UiState.Error.InputTooShort -> {

                    }
                }
            }
        }
    }
}
```

```kotlin
class MainRepository {

    suspend fun getData(): Resource<List<Person>> {
        return try {
            delay(3000L)
            if (Random.nextInt(2) == 0) {
                throw Exception()
            }
            Resource.Success(
                listOf(
                    Person(
                        name = "Chris P. Bacon",
                        isMale = true
                    ),
                    Person(
                        name = "Anita Hanjaab",
                        isMale = false
                    ),
                    Person(
                        name = "Mia khalifa",
                        isMale = false
                    ),
                )
            )
        } catch (e: Exception) {
            Resource.Error("Network error")
        }
    }
}
```

`Resource`는 다음과 같이 `Success`, `Error` 클래스를 가지고 있다.

```kotlin
sealed class Resource<T>(val data: T? = null, val message: String? = null) {

    // success case contains data
    class Success<T>(data: T) : Resource<T>(data)

    // error case attach error message, data can be null
    class Error<T>(message: String, data: T? = null) : Resource<T>(data, message)
}
```

`MainViewModel`에서 다음과 같이 `Error`를 UI에 전달하는 데 사용할 수 있다.

```kotlin
class MainViewModel(
    val repository: MainRepository
) : ViewModel() {

    private val _uiState = mutableStateOf(UiState())
    val uiState: State<UiState> = _uiState

    init {
        fetchData()
    }

    private fun fetchData() {
        viewModelScope.launch {
            _uiState.value = UiState(isLoading = true)
            when (val data = repository.getData()) {
                is Resource.Success -> {
                    _uiState.value = UiState(items = data.data ?: listOf())
                }
                is Resource.Error -> {
                    _uiState.value = UiState(error = UiState.Error.NetworkError)
                }
            }
        }
    }

//    sealed class SealedUiState {
//        object Loading: SealedUiState()
//        data class Error(val error: UiState.Error): SealedUiState()
//        data class Success(val items: List<Person>): SealedUiState()
//    }

    // 위의 sealed class로 구현하게 되면 하나의 상태에 국한되지만, 아래의 data class로 구현하면 여러 상태를 표현할 수 있다.
    // 예를 들어, 캐싱 레이어가 있다고 가정하고 네트워크 연결이 되어있지 않은 상태에서 DB에 캐싱된 데이터를 가져온다고 하자. 그렇게 되면 에러와 아이템을 같이 보여주어야 한다.
    data class UiState(
        val isLoading: Boolean = false,
        val error: Error? = null,
        val items: List<Person> = listOf()
    ) {
        sealed class Error {
            // stringResource에서 오류 메시지를 얻어야 하므로 sealed class로 전달
            object NetworkError : Error()
            object InputTooShort : Error()
            object InputEmpty : Error()
        }
    }
}
```

`Person` data class가 다음과 같이 있다고 가정하자. 이렇게 구현하게 되면 남자, 여자를 구분하는데 가독성이 좋지 않으며, 이외의 성별에 대해서는 표현할 수 없다.

```kotlin
data class Person(
    val name: String,
    val isMale: Boolean
)
```

따라서 다음과 같이 sealed class로 구현하는 경우 가독성이 좋아지게 된다.

```kotlin
data class Person(
    val name: String,
    val gender: Gender = Gender.Male
) {
    sealed class Gender() {
        object Male : Gender()
        object Female : Gender()
        object otherGender : Gender()
    }
}
```

## References

* [How to Make Your Code Clean With Kotlin Sealed Classes](https://www.youtube.com/watch?v=qzzkui-Z6CM&list=WL&index=1&t=20s)