# Side Effects

함수가 결과 값을 반환하는 것 이외에 다른 일을 할 때 그 함수가 Side Effect를 가진다고 한다.

기본적으로 Composable에서 Side Effect가 없어야하며, 필요한 경우 Composable의 Lifecycle을 인식하는 관리된 환경에서 Side Effect를 호출해야 한다.

이런 방식으로 `i`에 대한 증가를 구현하려면 side effect에 대해서 고려해야 한다. 다음과 같이 구현되면 예상되지 않은 결과를 얻을 수 있다.

```kotlin
var i = 0

@Composable
fun MyComposable() {
    i++
    Button(onClick = {}) {
        Text(text = "Click me")
    }
}
```

`SideEffect`를 사용하면 매번 recompose가 완료된 경우 `SideEffect`의 블록이 호출된다. 하지만 어떠한 이유로 composition이 실패하게 되면 `SideEffect` 블록의 코드는 실행되지 않는다.

```kotlin
var i = 0

@Composable
fun MyComposable() {
		// MyComposable이 성공적으로 recompose 되었을 때 SideEffect가 실행됨
		// 만약 composition이 실패하면 이 SideEffect 블록은 실행되지 않는다.
    SideEffect {
        i++
    }
    Button(onClick = {}) {
        Text(text = "Click me")
    }
}
```

composable이 composing을 끝냈을 때 cleanup이 필요한 경우 `DisposableEffect`를 사용할 수 있다.

```kotlin
var i = 0

@Composable
fun MyComposable(backPressedDispatcher: OnBackPressedDispatcher) {
    // remember 블록을 사용해 재초기화되지 않도록 함
    val callback = remember {
        object: OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                // Do something
            }
        }
    }
    // 매 recomposition 시 이 콜백을 추가하지만 제거할 수 있는 방법이 없어 메모리 릭이 발생하게 된다.
    backPressedDispatcher.addCallback(callback)
    DisposableEffect(key1 = , effect = ) {
        i++
    }
    Button(onClick = {}) {
        Text(text = "Click me")
    }
}
```

다음과 같이 `DisposableEffect`를 사용하여 위 문제를 해결할 수 있다.

```kotlin
@Composable
fun MyComposable(backPressedDispatcher: OnBackPressedDispatcher) {
    // remember 블록을 사용해 재초기화되지 않도록 함
    val callback = remember {
        object: OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                // Do something
            }
        }
    }
    // 매 recomposition 시 이 콜백을 추가하지만 제거할 수 있는 방법이 없어 메모리 릭이 발생하게 된다.
    // 이 때 DisposableEffect를 사용해 cleanup이 가능하다.
    DisposableEffect(key1 = backPressedDispatcher) {
        backPressedDispatcher.addCallback(callback)
        // dispose 해야하는 해당 콜백이 추가될 때 다음 블록이 트리거된다.
        onDispose {
            // 콜백 제거
            callback.remove()
        }
    }
    Button(onClick = {}) {
        Text(text = "Click me")
    }
}
```

다음과 같이 클릭하면 `counter`가 증가하고, 클릭 5번마다 “Hello” 스낵바를 보여주도록 구현해보자.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val scaffoldState = rememberScaffoldState()
            // composition lifecycle을 알고있는 scope -> 안전하게 suspend function 호출 가능
            val scope = rememberCoroutineScope()
            Scaffold(scaffoldState = scaffoldState) {
                var counter by remember {
                    mutableStateOf(0)
                }
                if (counter % 5 == 0 && counter > 0) {
                    scope.launch {
                        scaffoldState.snackbarHostState.showSnackbar("Hello")
                    }
                }
                Button(onClick = { counter++ }) {
                    Text("Click me : ${counter}")
                }
            }
        }
    }
}
```

<div align="center">
<img src="img/part-10/counter.png" width="40%">
</div>

버튼을 계속 누르게되면 스낵바가 큐잉되어 순서대로 호출되게 된다. 만약 이렇게 큐잉되지 않게 하고 싶은 경우 `scope.launch`를 `LaunchedEffect`로 변경하여 조건이 아닌 경우 코루틴을 취소하도록 할 수 있다.

```kotlin
if (counter % 5 == 0 && counter > 0) {
		// LaunchedEffect는 counter가 위 조건이 아닌 경우 블록 안의 코루틴을 취소시킨다.
	  LaunchedEffect(key1 = scaffoldState.snackbarHostState) {
	  scaffoldState.snackbarHostState.showSnackbar("Hello")
	  }
}
```

다음과 같이 비동기 작업이 필요할 때 `produceState`를 사용할 수 있다. 4초 후 `counter`를 4로 설정하여 recompose 하게 된다.

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val scaffoldState = rememberScaffoldState()
            Scaffold(scaffoldState = scaffoldState) {
                // 0으로 초기화
                var counter = produceState(initialValue = 0) {
                    // 비동기 작업을 실행할 수 있다.
                    // 3초 후 value를 4로 설정
                    delay(3000L)
                    // 비동기 결과로 value에 값을 대입하면 Scaffold 블록을 recompose한다.
                    value = 4
                }
                if (counter.value % 5 == 0 && counter.value > 0) {
                    // LaunchedEffect는 counter가 위 조건이 아닌 경우 블록 안의 코루틴을 취소시킨다.
                    LaunchedEffect(key1 = scaffoldState.snackbarHostState) {
                        scaffoldState.snackbarHostState.showSnackbar("Hello")
                    }
                }
                Button(onClick = {  }) {
                    Text("Click me : ${counter.value}")
                }
            }
        }
    }
}
```

<div align="center">
<img src="img/part-10/produce_state.png" width="40%">
</div>

## References

* [Side Effects & Effect Handlers - Android Jetpack Compose - Part 10](https://www.youtube.com/watch?v=f_iIMscTNjQ&list=PLQkwcJG4YTCSpJ2NLhDTHhi6XBNfk9WiC&index=10)
* [Side-effects in Compose](https://developer.android.com/jetpack/compose/side-effects)
* [[Compose] 6. Side-effects - LaunchedEffect, rememberCoroutineScope, rememberUpdatedState, DisposableEffect, SideEffect, produceState, derivedStateOf, snapshotFlow](https://tourspace.tistory.com/412)