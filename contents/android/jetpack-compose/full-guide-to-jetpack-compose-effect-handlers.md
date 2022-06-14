# Full Guide to Jetpack Compose Effect Handlers

Jetpack Compose Effect Handlers에 대해 알아보자.

Effect Handler에 대해 알기 전 Side Effect라는 개념을 알아야 한다. Side Effect는 Composable 함수 밖에서 발생하는 Effect를 의미한다. 다음 예제를 보며 확인해보자.

```kotlin
class MainActivity : ComponentActivity() {

    private var i = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            var text by remember {
                mutableStateOf("")
            }
            Button(onClick = { text += "#" }) {
                i++  // side effect -> escape the scope of a composable function
                Text(text = text)
            }
        }
    }
}
```

만약 `i`가 네트워크 호출을 한다고 가정하면 Button의 블록이 recompose 될 때마다 네트워크 호출을 수행하게 된다. 이러한 경우를 회피하기 위해 Effect Handler를 사용한다.

## LaunchedEffect

가장 흔하게 사용되는 Effect이다. 다음과 같은 `ViewModel`이 있다고 가정하자.

```kotlin
class LaunchedEffectViewModel(): ViewModel() {

    private val _sharedFlow = MutableSharedFlow<ScreenEvents>()
    val sharedFlow = _sharedFlow.asSharedFlow()

    init {
        viewModelScope.launch {
            _sharedFlow.emit(ScreenEvents.ShowSnackbar("Hello World!"))
        }
    }
    sealed class ScreenEvents {
        data class ShowSnackbar(val message: String): ScreenEvents()
        data class Navigate(val rotute: String): ScreenEvents()
    }
}
```

Composable 함수에서 `sharedFlow.collect`는 side effect이므로, 이 flow를 collect 하는 `LaunchedEffectFlowDemo`를 작성해보자.

```kotlin
@Composable
fun LaunchedEffectFlowDemo(
    viewModel: LaunchedEffectViewModel
) {
    LaunchedEffect(key1 = true) {
        viewModel.sharedFlow.collect { event ->
            when (event) {
                is LaunchedEffectViewModel.ScreenEvents.ShowSnackbar -> {

                }
                is LaunchedEffectViewModel.ScreenEvents.Navigate -> {

                }
            }
        }
    }
}
```

`LaunchedEffect`를 통해 애니메이션도 만들 수 있다.

```kotlin
@Composable
fun LaunchedEffectAnimation(
    counter: Int
) {
    val animatable = remember {
        androidx.compose.animation.core.Animatable(0f)
    }
    LaunchedEffect(key1 = counter) {
        animatable.animateTo(counter.toFloat())
    }
}
```

## RememberCoroutineScope

`RememberCoroutineScope`는 Composable 함수이며 Composition에서 Coroutine Scope의 참조를 얻을 수 있다. 보통의 경우 ViewModel의 ViewModel의 scope를 많이 사용하지만, 가끔씩 이 scope를 사용해야 될 때가 있다.

```kotlin
@Composable
fun RememberCoroutineScopeDemo() {
    val scope = rememberCoroutineScope()
    Button(onClick = {
        scope.launch {
            delay(1000L)
            println("Hello World!")
        }
    }) {
    }
}
```

## RememberUpdatedState

다음과 같이 일정 시간이 지나면 `onTimeout` 람다 함수를 호출해주는 코드가 있다고 하자. 이 코드의 문제점은 `RememberUpdatedStateDemo` Composable을 호출할 때마다 `onTimeout` 함수가 다르다는 것이다.

```kotlin
@Composable
fun RememberUpdatedStateDemo(
    onTimeout: () -> Unit
) {
    LaunchedEffect(key1 = true) {
        delay(3000L)
        onTimeout()
    }
}
```

이 함수를 호출했을 때 시간만 지연되기를 원한다면 `rememberUpdatedState`를 사용하면 된다. `onTimeout`이 변경될 때마다 `updatedOnTimeout`이 업데이트 된다.

```kotlin
@Composable
fun RememberUpdatedStateDemo(
    onTimeout: () -> Unit
) {
    val updatedOnTimeout by rememberUpdatedState(newValue = onTimeout)
    LaunchedEffect(key1 = true) {
        delay(3000L)
        updatedOnTimeout()
    }
}
```

## DisposableEffect

다음과 같이 `ON_PAUSE` 상태에 텍스트를 출력해주는 `observer`가 있다고 가정하자. 여기서 발생되는 문제는 `observer`는 정리되어야 하지만 정리되지 않는다는 것이다.

```kotlin
@Composable
fun DisposableEffectDemo() {
    val lifecycleOwner = LocalLifecycleOwner.current
    val observer = LifecycleEventObserver { _, event ->
        if (event == Lifecycle.Event.ON_PAUSE) {
            println("On pause called")
        }
    }
    lifecycleOwner.lifecycle.addObserver(observer)
}
```

Composable이 Composition을 떠날 때 `onDispose`를 수행하여 정리해줄 수 있다.

```kotlin
@Composable
fun DisposableEffectDemo() {
    val lifecycleOwner = LocalLifecycleOwner.current
    DisposableEffect(key1 = lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_PAUSE) {
                println("On pause called")
            }
        }
        lifecycleOwner.lifecycle.addObserver(observer)
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
        }
    }
}
```

## SideEffect

SideEffect는 별다른 파라미터를 취하지 않고, 람다 블럭만 존재한다. SideEffect는 composable이 성공적으로 recompose 되었을 때 호출된다.

```kotlin
@Composable
fun SideEffectDemo(
    nonComposeCounter: Int
) {
    SideEffect {
        println("Called after every successful recomposition")
    }
}
```

## produceState

`produceState`는 `LaunchedEffect`와 같이 Coroutine Scope를 준다. 따라서 블럭 안에서 suspend 함수를 호출할 수 있다. 이 함수는 flow와 비슷하다. 1초가 지난 후 `value`를 ++하여 `State`를 업데이트 한다.

```kotlin
@Composable
fun ProduceStateDemo(countUpTo: Int): State<Int> {
    return produceState(initialValue = 0) {
        while (value < countUpTo) {
            delay(1000L)
            value++
        }
    }
}
```

## DerivedStateOf

다음과 같은 코드가 있다고 가정하자. 다음 코드의 문제는 `counterText`가 계속 recompute되어 리소스가 낭비된다는 점이다.

```kotlin
@Composable
fun DerivedStateOfDemo() {
    var counter by remember {
        mutableStateOf(0)
    }
    val counterText = "The counter is ${counter}" // 이 코드가 지속적으로 재실행되어 리소스가 낭비된다.
    Button(onClick = { counter++ }) {
        Text(text = counterText)
    }
}
```

`derivedStateOf`를 사용해 위 리소스 낭비를 줄일 수 있다.

```kotlin
@Composable
fun DerivedStateOfDemo() {
    var counter by remember {
        mutableStateOf(0)
    }
    // counterText에 처음 접근했을 때 text를 compute하고 캐싱한다.
    // counter가 변경되어 counterText는 recompute 되고, 사용되는 모든 곳에 변경됐다고 알린다. 
    val counterText by derivedStateOf {
        "The counter is ${counter}"
    }
    Button(onClick = { counter++ }) {
        // 여기선 캐시된 counterText를 사용한다.
        Text(text = counterText)
    }
}
```

## SnapshotFlow

`snapshotFlow`는 state를 flow로 변환하기 위해 사용된다. 즉, 이전에는 flow를 compose에서 사용하기 위해 `.collectAsState()`를 호출하여 변경했다면, 이번엔 state → flow로 변경하는 역할을 한다. 다음은 스낵바에 노출되는 메시지를 flow로 변환하여 emit하는 과정이다.

```kotlin
@Composable
fun SnapshotFlowDemo() {
    val scaffoldState = rememberScaffoldState()
    LaunchedEffect(key1 = scaffoldState) {
        snapshotFlow { scaffoldState.snackbarHostState }
            .mapNotNull { it.currentSnackbarData?.message }
            .distinctUntilChanged()
            .collect { message ->
                println("A snackbar with message ${message} was shown")
            }
    }
} 
```

## References

* [Full Guide to Jetpack Compose Effect Handlers](https://www.youtube.com/watch?v=gxWcfz3V2QE)