# 5 Fatal Coroutine Mistakes Nobody Tells You About

코루틴을 사용하며 가장 흔히 발생하는 실수 5가지를 알아보자.

## Suspend function running sequentially

다음과 같은 코드가 있다고 가정하자. `getFirstName()` 함수의 경우 서버에 요청하여 가져오는 동작을 수행한다고 가정한다.

```kotlin
suspend fun getUserFirstNames(userIds: List<String>): List<String> {
    val firstNames = mutableListOf<String>()
    for (id in userIds) {
        firstNames.add(getFirstName(id))
    }
    return firstNames
}

suspend fun getFirstName(userId: String): String {
    delay(1000L)
    return "First name"
}
```

위와 같이 구현된 경우 suspend 함수를 순차적으로 실행하게 된다. 즉, 각 `getFirstName(id)`를 호출할 때 1000L 만큼 delay 된다는 의미이다. 우리가 원하는 것은 모든 호출이 suspend 되는 것이 아닌 동시적으로 발생하는 것을 원하기 때문에 `async`를 사용하여 해결할 수 있다.

```kotlin
suspend fun getUserFirstNames(userIds: List<String>): List<String> {
    val firstNames = mutableListOf<Deferred<String>>()
    coroutineScope {
        for (id in userIds) {
            // async는 이 함수의 suspend 함수와는 독립적으로 동작한다.
            val firstName = async {
                getFirstName(id)
            }
            firstNames.add(firstName)
        }
    }
		// 모든 결과를 기다리기 위해 awaitAll() 사용
    return firstNames.awaitAll()
}

suspend fun getFirstName(userId: String): String {
    delay(1000L)
    return "First name"
}
```

## Check coroutine cancellation

다음과 같은 코드가 있다고 가정하자.

```kotlin
suspend fun doSomething() {
    val job = CoroutineScope(Dispatchers.Default).launch {
        var random = Random.nextInt(100_000)
        while (random != 50000) {
            random = Random.nextInt(100_000)
        }
    }
		// 500L 동안 못찾은 경우 
    delay(500L)
		// cancel
    job.cancel()
}
```

위 코드의 문제는 취소를 확인하지 않는다는 것이다. `while` 루프를 실행하는 도중 코루틴이 취소되면 코루틴이 취소되었는지에 대한 정보가 없기 때문에 계속해서 `while` 루프를 수행할 것이다. `isActive`를 사용해 코루틴이 취소되었는지 확인하거나, `ensureActive()` 함수를 통해 취소를 확인할 수 있다.

```kotlin
suspend fun doSomething() {
    val job = CoroutineScope(Dispatchers.Default).launch {
        var random = Random.nextInt(100_000)
        while (random != 50000 && isActive) {
            random = Random.nextInt(100_000)
						ensureActive()
        }
    }
    delay(500L)
    job.cancel()
}
```

## Main-safe coroutine

다음과 같은 코드가 있다고 가정하자.

```kotlin
suspend fun doNetworkCall(): Result<String> {
    val result = networkCall()
    return if (result == "Success") {
        Result.success(result)
    } else Result.failure(Exception())
}

suspend fun networkCall(): String {
    delay(3000L)
    return if (Random.nextBoolean()) "Success" else "Error"
}
```

위 코드의 문제는 `networkCall()` 함수가 main-safe 하지 않다는 것이다. 이 함수가 디스패처를 지정하지 않아 메인 스레드에서 실행되는 경우가 발생할 수 있다. 이는 `withContext` 블록을 사용해 해결할 수 있다. Room, Retrofit과 같은 3rd-party 라이브러리들은 이미 이런 방식으로 구현되어 있어 따로 지정해 줄 필요가 없다.

```kotlin
suspend fun doNetworkCall(): Result<String> {
    val result = networkCall()
    return if (result == "Success") {
        Result.success(result)
    } else Result.failure(Exception())
}

suspend fun networkCall(): String {
    return withContext(Dispatchers.IO) {  // Dispatchers.IO 디스패처로 변경
        delay(3000L)
        if (Random.nextBoolean()) "Success" else "Error"
    }
}
```

## Coroutine CancellationException

다음과 같은 코드가 있다고 가정하자.

```kotlin
suspend fun riskyTask() {
    try {
        delay(3000L)
        println("The answer is ${10/0}")
    } catch (e: Exception) {
        println("Oops, that didn't work")
    }
}
```

suspend 함수 내 `try`, `catch`를 구현한 경우 이 cancellation exception을 부모로 전달하지 않는 문제가 있다. 부모에게 Cancellation Exception을 전달하기 위해 직접 `throw`를 해주거나, 처리하고 싶은 예외를 지정해주면 된다.

```kotlin
suspend fun riskyTask() {
    try {
        delay(3000L)
        println("The answer is ${10/0}")
    } catch (e: Exception) {
        if (e is CancellationException) {
            throw e
        }
        println("Oops, that didn't work")
    }
}
```

```kotlin
suspend fun riskyTask() {
    try {
        delay(3000L)
        println("The answer is ${10/0}")
    } catch (e: HttpException) { // HttpException 예외 지정
        println("Oops, that didn't work")
    }
}
```

## Typically should not expose suspend function to ui

다음과 같은 코드가 있다고 가정하자.

```kotlin
class MainActivity : ComponentActivity() {
    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val button = findViewById<Button>(R.id.dialog_button)
        button.setOnClickListener {
            lifecycleScope.launch {
                viewModel.postValueToApi()
            }
        }
    }
}
```

```kotlin
class MainViewModel: ViewModel() {

    suspend fun postValueToApi() {
        delay(10000L)
    }
}
```

위 코드의 문제는 `ViewModel`이 UI에 suspend 함수를 노출하고 있는 것이다. `MainActivity`에서 `lifecycleScope`를 통해 `ViewModel`의 `postValueToApi()` 함수를 호출하고 있는데, 화면 회전 등 액티비티가 destroy 되는 경우 해당 함수가 취소되게 된다. 그래서 대부분의 경우 `ViewModel` 내에서 coroutine scope를 지정해준다.

```kotlin
class MainActivity : ComponentActivity() {
    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val button = findViewById<Button>(R.id.dialog_button)
        button.setOnClickListener {
            viewModel.postValueToApi()
        }
    }
}
```

```kotlin
class MainViewModel: ViewModel() {

    fun postValueToApi() {
        viewModelScope.launch {
            delay(10000L)
        }
    }
}
```

## References

* [5 Fatal Coroutine Mistakes Nobody Tells You About](https://www.youtube.com/watch?v=cr5xLjPC4-0&t=17s)