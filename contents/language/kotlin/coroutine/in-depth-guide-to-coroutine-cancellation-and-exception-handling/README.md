# In-Depth Guide to Coroutine Cancellation & Exception Handling

코루틴에 대한 취소와 예외 처리에 대해 알아보자.

## Coroutine Exception Handling

다음과 같이 `onCreate()`에서 `try/catch`를 이용해 코루틴의 예외를 잡아보자.

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            try {
                launch {
                    throw Exception()
                }
            } catch (e: Exception) {
                println("Caught exception: $e")
            }
        }
    }
}
```

이를 실행해보면 실제로는 해당 예외가 잡히지 않아 로그캣에 보여지지 않는다. 즉, `try/catch`를 사용하여 직접 처리할 수 없다.

`launch`로 코루틴을 실행시키는 방법과 `async`로 코루틴을 실행시키는 방법이 있다. `launch`는 아무런 결과도 받지 않지만, `async`는 결과를 전달 받을 수 있다.

```kotlin
lifecycleScope.launch {
    val string = async {
        delay(500L)
        "Result"
    }
    println(string.await()) // string의 값을 얻을 때까지 대기한 후 수행
}
```

다음과 같이 예외를 발생시키면, 앱 실행시 즉시 크래시가 발생한다.

```kotlin
lifecycleScope.launch { // 예외를 전파 받으며 처리되지 않으면 크래시 발생
    val string = async {
        delay(500L)
        throw Exception("error") // 부모 코루틴에게 예외를 전달
        "Result"
    }
}
```

다음과 같이 `launch`를 `async`로 변경하면 부모 코루틴으로 전파되긴 하지만, 크래시가 발생하지 않는다. 사용자가 두 `async`에 대해 `await()`을 사용하여야 크래시가 발생한다.

```kotlin
lifecycleScope.async {
    val string = async {
        delay(500L)
        throw Exception("error")
        "Result"
    }
}
```

다음과 같이 새로운 `launch` 블록에 `try/catch`로 `deffered` 값에 대해 `await()`을 하는 경우 예외를 잡을 수 있다. 하지만 이렇게 예외를 처리하고 있는 것은 잘못된 부분이다.

```kotlin
val deferred = lifecycleScope.async {
    val string = async {
        delay(500L)
        throw Exception("error")
        "Result"
    }
}

lifecycleScope.launch {
    try {
        deferred.await()
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

코루틴 예외를 처리하기 위한 `handler`를 설정한 후 루트 코루틴에 설치한다.

```kotlin
val handler = CoroutineExceptionHandler { _, throwable ->
    println("Caught exception: $throwable")
}

lifecycleScope.launch(handler) {
    throw Exception("error")
}
```

앱을 실행하면 다음과 같이 로그가 발생하며, 크래시는 발생되지 않는다.

```
2022-07-11 22:52:05.170 12531-12531/com.example.androidtest I/System.out: Caught exception: java.lang.Exception: error
```

이러한 방식으로 코루틴에서 발생한 예외에 대한 처리를 수행한다.

다음과 같이 자식 코루틴에서 발생한 예외도 루트 코루틴에서 처리될 수 있다. 핸들러에선 `CancellationException`을 호출하지 않는다. 즉, 코루틴에서 `CancellationException` 예외가
발생해도 핸들러에선 취급하지 않는다. `CancellationException`은 코루틴이 기본적으로 처리하는 예외이기 때문이다. 처리되지 않는 예외들만 핸들러에서 처리된다.

```kotlin
val handler = CoroutineExceptionHandler { _, throwable ->
    println("Caught exception: $throwable")
}

lifecycleScope.launch(handler) {
    launch {
        throw Exception("error")
    }
}
```

앱에서 사용할 수 있는 코루틴 2가지가 존재한다. 다음 코드를 작성하고 실행해보자.

```kotlin
CoroutineScope(Dispatchers.Main).launch {
    launch {
        delay(300L)
        throw Exception("Coroutine 1 failed")
    }
    launch {
        delay(400L)
        println("Coroutine 2 finished")
    }
}
```

다음과 같이 첫 번째 코루틴에서 예외가 발생하고 두 번째 코루틴은 실행되지 않는다.

```
java.lang.Exception: Coroutine 1 failed
```

다음과 같이 핸들러를 추가한다. 크래시는 발생하지 않지만 동일하게 두 번째 코루틴이 실행되지 않는다.

```kotlin
val handler = CoroutineExceptionHandler { _, throwable ->
    println("Caught exception: $throwable")
}

CoroutineScope(Dispatchers.Main + handler).launch {
    launch {
        delay(300L)
        throw Exception("Coroutine 1 failed")
    }
    launch {
        delay(400L)
        println("Coroutine 2 finished")
    }
}
```

`CoroutineScope`를 사용하게 되면, 어떠한 이유로 예외가 발생되는 경우 해당 코루틴 및 자식 코루틴 모두 취소된다.

`supervisorScope`를 사용하면 두 코루틴 각각 독립적으로 실행된다.

```kotlin
CoroutineScope(Dispatchers.Main + handler).launch {
    supervisorScope {
        launch {
            delay(300L)
            throw Exception("Coroutine 1 failed")
        }
        launch {
            delay(400L)
            println("Coroutine 2 finished")
        }
    }
}
```

```
2022-07-11 23:07:13.805 12885-12885/com.example.androidtest I/System.out: Caught exception: java.lang.Exception: Coroutine 1 failed
2022-07-11 23:07:13.904 12885-12885/com.example.androidtest I/System.out: Coroutine 2 finished
```

따라서 `CoroutineScope`와 `supervisorScope`를 적절히 잘 사용해야 한다.

`ViewModel.viewModelScope`를 확인해보면 실제로 `SupervisorJob()`으로 설정되어 있다.

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
    get() {
        val scope: CoroutineScope? = this.getTag(JOB_KEY)
        if (scope != null) {
            return scope
        }
        return setTagIfAbsent(
            JOB_KEY,
            CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        )
    }
```

## Coroutine Cancellation

다음과 같이 구현한 후 로그를 확인해보자.

```kotlin
lifecycleScope.launch {
    val job = launch {
        try {
            delay(500L)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        println("Coroutine 1 finished")
    }
    delay(300L)
    job.cancel()
}
```

`job`이 취소되어 `Coroutine 1 finished`가 출력되지 않아야 할 것으로 보이지만 출력하고 있다.

```
2022-07-11 23:18:28.060 13157-13157/com.example.androidtest W/System.err: kotlinx.coroutines.JobCancellationException: StandaloneCoroutine was cancelled; job=StandaloneCoroutine{Cancelling}@298ccec
2022-07-11 23:15:48.678 13069-13069/com.example.androidtest I/System.out: Coroutine 1 finished
```

`delay()`를 호출하면 `CancellationException`이 발생하는데 이 예외가 `try/catch` 블록에서 먹혔기 때문이다. 따라서 위와 같이 `JobCancellationException`이
로그에 출력되는 것을 확인할 수 있다. 따라서 부모 코루틴으로 예외가 전파되지 않아, 부모 코루틴은 자식 코루틴이 취소되었는지 확인할 방법이 없다.

이는 다음 두 가지 방법을 통해 해결할 수 있다.

1. `HttpException` 등 특정 예외에 대해서만 처리하면 해결할 수 있다.
2. `CancellationException`인 경우 다시 `throw`하여 부모 코루틴으로 전파한다.

## References

* [In-Depth Guide to Coroutine Cancellation & Exception Handling - Android Studio Tutorial](https://www.youtube.com/watch?v=VWlwkqmTLHc&t=6)


