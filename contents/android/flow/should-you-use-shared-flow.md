# Should You Use SharedFlow?

`SharedFlow`의 위험성에 대해 알아본다. `SharedFlow`에 대해 정확히 알고 사용해야 한다. 또한 이와 유사한 개념인 `Channel`에 대해서도 알아본자.

다음과 같이 `SharedFlow`와 `Channel`을 사용하는 코드가 있다고 가정한다. 두 코드 모두 one-time event를 보내는데 사용된다. one-time event는 스낵바 또는 토스트를 보여주거나 할 때 사용되는 이벤트이다.

```kotlin
class MainViewModel: ViewModel() {

    private val _sharedFlow = MutableSharedFlow<Int>()
    val sharedFlow = _sharedFlow.asSharedFlow()

    private val channel = Channel<Int>()
    val flow = channel.receiveAsFlow()

    init {
        viewModelScope.launch {
            repeat(1000000) {
                _sharedFlow.emit(it)
                delay(1000L)
            }
        }
        viewModelScope.launch {
            repeat(1000000) {
                channel.send(it)
                delay(1000L)
            }
        }
    }
}
```

## Difference between SharedFlow and Channel 

`MainActivity`에서 다음과 같이 `SharedFlow`와 `Channel`을 collect 한다고 하자.

```kotlin
class MainActivity : ComponentActivity() {

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.sharedFlow.collect { number ->
                    println("Collected $number from shared flow")
                }
            }
        }
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.flow.collect { number ->
                    println("Collected $number from channel")
                }
            }
        }
    }
}
```

`SharedFlow`는 hot flow이다. 이는 `emit` 했을 때 이 `emit`에 대한 observer가 없어도 emit 한다는 의미이다. 이것은 이벤트를 놓칠 수도 있다는 의미이다. 위 코드를 보면 `SharedFlow`를 `STARTED` 시점에 collect를 수행하므로 이벤트를 놓치지 않을 것이라고 생각할 수 있지만 이는 옳지 않다. 예를 들어, 화면이 회전되는 경우 액티비티는 다시 생성되므로 Activity Lifecycle은 일시적으로 destory 상태가 된다. `repeatOnLifecycle`은 이러한 과정에서 코루틴을 취소(STOP)시키고 다시 실행(START)시키는 역할을 수행한다. 문제는 이 부분에서 발생한다. Activity가 STOP 상태이고, Destroy 되지 않았을 때 `emit` 하게되면 `SharedFlow`를 collect 하는 observer가 취소된 상태가 된다. 따라서 one-time event가 발생하더라도 잃어버리는 상태가 된다.

`Channel`은 `SharedFlow`와 다르게 cold flow이므로, observer가 있는 경우에만 `emit`한다.

다음과 같은 `MainService`가 있다고 가정하자.

```kotlin
class MainService: Service() {
    override fun onBind(intent: Intent?): IBinder? {
        return  null
    }

    override fun onCreate() {
        super.onCreate()
        thread {
            while (true) {
                println("Service is running...")
                Thread.sleep(2000L)
            }
        }
    }
}
```

`MainActivity`에서 서비스를 실행시키고 결과를 확인해보자.

```kotlin
class MainActivity : ComponentActivity() {

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        Intent(this, MainService::class.java).also {
            startService(it)
        }
    }
}
```

앱 실행 시 `SharedFlow`와 `Channel`이 모두 collect 되는 것을 확인할 수 있다.

```
2022-06-11 23:18:44.706 7075-7075/com.example.androidtest I/System.out: Collected 0 from shared flow
2022-06-11 23:18:44.708 7075-7075/com.example.androidtest I/System.out: Collected 0 from channel
2022-06-11 23:18:45.709 7075-7075/com.example.androidtest I/System.out: Collected 1 from shared flow
2022-06-11 23:18:45.711 7075-7075/com.example.androidtest I/System.out: Collected 1 from channel
2022-06-11 23:18:46.713 7075-7075/com.example.androidtest I/System.out: Collected 2 from shared flow
2022-06-11 23:18:46.714 7075-7075/com.example.androidtest I/System.out: Collected 2 from channel
```

백그라운드로 이동했을 때 둘 모두 `emit` 되지 않게 되며, 액티비티를 다시 실행시켰을 경우 `Channel`은 `emit`이 멈췄던 부분에서 다시 시작하지만, `SharedFlow`의 경우 계속 `emit`하여 액티비티가 시작했을 때 현재의 값을 `emit` 한다. 즉, 59 ~ 110 이전의 이벤트들은 잃어버리게 된다.  

```
2022-06-11 23:19:43.119 7075-7075/com.example.androidtest I/System.out: Collected 58 from shared flow
2022-06-11 23:19:43.120 7075-7075/com.example.androidtest I/System.out: Collected 58 from channel
2022-06-11 23:20:37.805 7075-7075/com.example.androidtest I/System.out: Collected 59 from channel
2022-06-11 23:20:37.997 7075-7075/com.example.androidtest I/System.out: Collected 111 from shared flow
2022-06-11 23:20:38.807 7075-7075/com.example.androidtest I/System.out: Collected 60 from channel
2022-06-11 23:20:39.000 7075-7075/com.example.androidtest I/System.out: Collected 112 from shared flow
2022-06-11 23:20:39.810 7075-7075/com.example.androidtest I/System.out: Collected 61 from channel
```

이런 방식(one-time event)의 사용은 `SharedFlow`에게 맞지 않다. `SharedFlow`는 여러 observer가 있는 경우에 유용하다. 만약 Session Management 클래스가 있고, connection, disconnection event 등을 수행할 때 이 이벤트들에 대한 observer를 앱의 여러 위치에서 사용할 수 있다.

하지만 하나의 observer 만을 필요로 한다면, `Channel`을 사용하는 것이 좋다. 

## References

* [Should You Use SharedFlow?](https://www.youtube.com/watch?v=QNrNKPKe5oc)