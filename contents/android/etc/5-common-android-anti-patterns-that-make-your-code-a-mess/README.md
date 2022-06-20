# 5 Common Android Anti-Patterns That Make Your Code a Mess

## 모든 클래스의 Base와 같은 기반 클래스를 만드는 것

만약 다음과 같은 `BaseActivity`가 있고, 해당 Activity를 상속해 사용하는 `MainActivity`가 있다고 가정하자.

```kotlin
abstract class BaseActivity : Activity() {

fun showToast(text: String) {
        Toast.makeText(
            this,
            text,
            Toast.LENGTH_LONG
        ).show()
    }
}
```

```kotlin
class MainActivity : BaseActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        showToast("Hello world")
    }
}
```

이렇게 `BaseActivity`를 상속해 구현하면 `MainActivity`에서 `showToast`를 쉽게 호출할 수 있게 된다. 이는 많은 중복 코드를 줄일 수 있게 된다. 하지만 이 `BaseActivity`를 상속하는 모든 Activity들은 `BaseActivity`에 강한 커플링이 생기게 된다.

만약 `BaseActivity`의 `showToast`를 변경하는 경우, `BaseActivity`를 상속한 모든 Activity들이 영향을 받게 된다. 이러한 커플링을 없애고 중복 코드를 제거하기 위한 방법으로는 코틀린의 확장 함수를 사용할 수 있다.

```kotlin
fun Activity.showToast(text: String) {
    Toast.makeText(
        this,
        text,
        Toast.LENGTH_LONG
    ).show()
}
```

## 모든 디펜던시들을 하나의 App Module에 넣는 것

대부분의 사람들이 `SingletonComponent`에 모든 클래스를 넣는 방식을 사용한다. 이는 해서는 안되는 짓이다. 액티비티나 프레그먼트, 뷰와 같은 라이프사이클을 가진 모듈별로 나눠야 한다. 이는 메모리 효율성 뿐만 아니라 어떤 모듈에 어떤 디펜던시가 있는지 확인하기 쉽다. 또한 하나의 거대한 앱 모듈에 넣는 방식은 테스팅이 어렵다는 것이다.

`AppModule` 클래스에 모든 디펜던시가 있고 테스트 케이스에 특정 디펜던시만 필요하다고 가정하자. `UninstallModules` 어노테이션을 사용해 `AppModule` 클래스를 삭제할 수 있지만 모든 `AppModule` 디펜던시를 삭제하므로 inject가 불가하여 테스팅이 불가능하다.

```kotlin
@RunWith(AndroidJUnit4::class)
@HiltAndroidTest
@UninstallModules(AppModule::class)
class ExampleInstrumentedTest {
		
}
```

다른 모듈에 대한 디펜던시를 분리하여 fine-grained control이 가능하고 어떤 모듈을 제거하고 유지해야 할지 명확하게 알 수 있다.

## Activity를 모든 화면에 사용하는 것

모든 화면에 Activity를 사용하는 것은 좋지 않다. 구글은 Fragment 사용을 권장한다. 액티비티는 인텐트와 같은 트랜잭션으로 화면 전환 시 무거운 작업들이 수행되지만, Fragment는 컨테이너의 내용물을 수정해주기만 하면 되기 때문에 더 가볍다.

## Dispatcher를 하드코딩 하는 것

다음과 같은 CPU intensive 작업을 수행하기 위해 `Disaptchers.Default` 스코프를 사용한다고 가정하자.

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(): ViewModel() {
		private val _sharedFlow = MutableSharedFlow<String>()
		val sharedFlow = _sharedFlow.asSharedFlow()
		
		fun longRunningTask() {
				viewModelScope.launch(Dispatchers.Default) {
						delay(10000L)
						_sharedFlow.emit("Finished")
				}
		}
}
```

위와 같이 `Dispatchers.Default`를 하드코딩 하는 것은 코루틴에게 이 디스패처를 사용하라고 강제하는 것이다. 만약 코루틴을 테스트하는 경우 특별한 테스트 코루틴 디스패처를 사용하는데, 이 테스트 디스패처가 테스트 케이스의 모든 컨트롤을 가지게 되며, 내가 원하는 컨트롤을 가질 수 없게 된다.

이를 해결하는 간단한 방법은 디스패처를 injecting 하는 것이다. `DispatcherProvier` 인터페이스와 `StandardDispatchers` 클래스를 생성한다. 그리고 `StandardDispatchers`에 모든 디스패처 구현을 추가한다.

```kotlin
class StandardDispatchers: DispatcherProvider {
    override val main: CoroutineDispatcher
        get() = Dispatchers.Main
    override val io: CoroutineDispatcher
        get() = Dispatchers.IO
    override val default: CoroutineDispatcher
        get() = Dispatchers.Default
    override val unconfined: CoroutineDispatcher
        get() = Dispatchers.Unconfined
}

interface DispatcherProvider {
    val main: CoroutineDispatcher
    val io: CoroutineDispatcher
    val default: CoroutineDispatcher
    val unconfined: CoroutineDispatcher
}
```

`AppModule` 클래스에 injection을 위해 `StandardDispatchers`를 반환하는 `provideDispatcherProvider`를 생성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Singleton
    @Provides
    fun provideDispatcherProvider(): DispatcherProvider {
        return StandardDispatchers()
    }
}
```

ViewModel에서는 `DispatcherProvier`를 injection 후 `dispatchers.default`를 사용하면 된다.

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val dispatchers: DispatcherProvider
): ViewModel() {
    private val _sharedFlow = MutableSharedFlow<String>()
    val sharedFlow = _sharedFlow.asSharedFlow()

    fun longRunningTask() {
        viewModelScope.launch(dispatchers.default) {
            delay(10000L)
            _sharedFlow.emit("Finished")
        }
    }
}
```

그러나 테스팅을 수행하기 위한 테스트 디스패처를 인젝션하려면 다른 구현이 필요하다. 다음과 같이 `TestDispatchers`를 생성한 후 `AppModule`에 해당 클래스를 반환한다.

```kotlin
class TestDispatchers: DispatcherProvider {
    override val main: CoroutineDispatcher
        get() = TestCoroutineDispatcher()
    override val io: CoroutineDispatcher
        get() = TestCoroutineDispatcher()
    override val default: CoroutineDispatcher
        get() = TestCoroutineDispatcher()
    override val unconfined: CoroutineDispatcher
        get() = TestCoroutineDispatcher()
}
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Singleton
    @Provides
    fun provideDispatcherProvider(): DispatcherProvider {
        return TestDispatchers()
    }
}
```

이렇게 되면 `DispatcherProvider`의 모든 디스패처가 테스트 디스패처를 사용하므로 원하는 테스팅 결과를 얻을 수 있다.

## 코루틴 사용 시 GlobalScope를 사용하는 것

`GlobalScope`를 사용하는 경우 워닝이 발생하는데, 안드로이드에선 `GlobalScope` 사용하지 않거나 조심히 사용해야 한다. 하지만 사람들은 쉽게 코루틴을 실행시킬 수 있기 때문에 `GlobalScope`를 사용한다.

`GlobalScope`는 자신의 코루틴의 lifetime을 제한하지만, 실제로 안드로이드는 lifecycle을 더 많이 처리하기 때문에 `MainActivity`가 destroy되는 경우 종료되거나 취소되지 않는 한 `GlobalScope`는 남아있게 된다.

이렇게 되면 코루틴이 실행되며 자원을 계속 사용하기 때문에 GC는 `GlobalScope`에 사용되는 자원을 회수하지 않는다. 결국 메모리 릭이 발생하게 된다.

```kotlin
class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        GlobalScope.launch {
            viewModel.sharedFlow.collectLatest {

            }
        }
    }
}
```

이를 해결하는 간단한 방법은 안드로이드에 있는 코루틴 스코프를 사용하는 것이다. 다음과 같이 `lifecycleScope`를 사용하면 `MainActivity`가 destroy 되면 코루틴은 같이 취소되게 된다.

```kotlin
class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        lifecycleScope.launch {
            viewModel.sharedFlow.collectLatest {

            }
        }
    }
}
```

## References

* [5 Common Android Anti-Patterns That Make Your Code a Mess](https://www.youtube.com/watch?v=skW4wSuXCe0)