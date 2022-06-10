# The Ultimate Guide to Koin

Koin을 통해 dependency injection을 하는 방법을 알아보자. 

## Concept of Dependency Injection

Dependency injection에 대해 간단히 알아본다. 만약 다음과 같이 `MainRepositoryImpl` 클래스가 있다고 가정하자.

```kotlin
class MainRepositoryImpl {

    fun doNetworkCall() {

    }
}
```

그리고 `MainViewModel`에서 해당 레포지토리의 `doNetworkCall()` 메서드를 호출한다고 하자. `MainViewModel`에서 직접 `MainRepositoryImpl` 인스턴스를 만들고 네트워크 호출을 한다. 이렇게 구현된다면 모든 테스트 코드에 해당 레포지토리 인스턴스가 생성되는 문제가 발생한다. 만약 이 `MainRepositoryImpl`이 실제 API 서버와 호출된다면 테스트 케이스에서도 동일하게 실제 API 서버와 통신하게 된다. 하지만 테스트 케이스에선 실제 서버와 통신하는 것을 원하지 않는다.

```kotlin
class MainViewModel: ViewModel() {

    private val repository = MainRepositoryImpl()

    fun doNetworkCall() {
        repository.doNetworkCall()
    }
}
```

이를 해결하기 위해 `MainRepository` 인터페이스를 생성하고 기존의 `MainRepositoryImpl`에서 상속하도록 구현한다.

```kotlin
interface MainRepository {

    fun doNetworkCall()
}
```

```kotlin
class MainRepositoryImpl: MainRepository {

    override fun doNetworkCall() {

    }
}
```

그리고 `MainViewModel`의 생성자로 `MainRepository` 인터페이스를 전달한다. 이제 `MainViewModel`의 생성자로 원하는 레포지토리를 전달할 수 있기 때문에 좀 더 유연해졌다. 테스트 코드 작성 시에도 원하는 레포지토리(실제 API 서버가 아닌 다른 로직)를 전달하여 테스트할 수 있다. 

```kotlin
class MainViewModel(
    private val repository: MainRepository
): ViewModel() {

    fun doNetworkCall() {
        repository.doNetworkCall()
    }
}
```

Koin은 이러한 dependency injection을 도와주는 라이브러리이다.

## Difference from Dagger-Hilt

Dagger-Hilt는 컴파일 타임에 inject하는 라이브러리이다. 그러므로 앱 실행 시점에는 어디에 어떤 의존성이 필요한지 알고 있다. Dagger-Hilt는 많은 어노테이션, 어노테이션 프로세싱을 가지고 있으며, 우리가 보지 못하는 클래스들을 생성하고 이 클래스들이 dependency injection을 제공한다.

이와 다르게 Koin는 컴파일 타임에 inject하지 않는다. 그러므로 코드 실행 시 디펜던시를 요구할 때 제공하여 조금 느리게 동작한다. 하지만 속도 체감은 크게 느껴지지 않으므로 원하는 라이브러리를 사용하면 된다.

## How to use Koin

### Dependencies

다음과 같이 디펜던시를 추가해준다. 

```groovy
dependencies {
    // ...
    implementation "io.insert-koin:koin-android:3.2.0-beta-1"
    implementation "io.insert-koin:koin-androidx-navigation:3.2.0-beta-1"
    implementation "io.insert-koin:koin-androidx-compose:3.2.0-beta-1"
    testImplementation "io.insert-koin:koin-test-junit4:3.2.0-beta-1"

    // Retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-moshi:2.9.0'
    implementation "com.squareup.okhttp3:okhttp:5.0.0-alpha.3"

    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:2.4.1"
}
```

`MyApi` 인터페이스 생성 후 임의의 API를 호출하는 `callApi()` 메서드를 작성한다.

```kotlin
interface MyApi {

    @GET("my/endpoint")
    fun callApi()
}
```

`MainRepositoryImpl`의 생성자에 `MyApi`를 전달한 후 `doNetworkCall()`에서 해당 `callApi()` 메서드를 호출하도록 작성한다.

```kotlin
class MainRepositoryImpl(
    private val api: MyApi
): MainRepository {

    override fun doNetworkCall() {
        api.callApi()
    }
}
```

이제 Koin이 어떤 디펜던시를 제공해야하는지 알려주기 위해 모듈을 작성한다. `AppModule` 파일을 생성하고 다음과 같이 작성한다. 

```kotlin
val appModule = module {
    // singleton
    single {
        // 이 타입이 필요한 부분에 injection
        Retrofit.Builder()
            .baseUrl("https://google.com")
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
            .create(MyApi::class.java)
    }
    single<MainRepository> { // 인터페이스 타입에 제공
        // get()을 통해 이미 제공되는 타입의 인스턴스를 전달할 수 있다.
        MainRepositoryImpl(get())
    }
    // factory는 항상 새 인스턴스를 제공한다.
    factory {
    }
    viewModel { // ViewModel을 제공
        MainViewModel(get())
    }
}
```

`MainActivity`에 `viewModel`을 inject 한다. 추가적으로 `MyApi`를 inejct하고 싶은 경우 주석과 같이 작성하면 된다.

```kotlin
class MainActivity : ComponentActivity() {

//    private val viewModel by viewModel<MainViewModel>()   // only xml

//    private val api = get<MyApi>()        // how to inject MyApi
//    private val api by inject<MyApi>()    // lazy inject

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            KoinGuideTheme {
                val viewModel = getViewModel<MainViewModel>()   // compose
                viewModel.doNetworkCall()
            }
        }
    }
}
```

현재 `MainViewModel`의 `doNetworkCall()`을 수행하게 되면 에러가 발생하므로, 단순히 로그만 출력되도록 변경한다.

```kotlin
class MainViewModel(
    private val repository: MainRepository
): ViewModel() {

    fun doNetworkCall() {
        println("Something")
//        repository.doNetworkCall()
    }
}
```

Koin을 사용하기 위해 `MainApplication` 클래스를 다음과 같이 작성하고 `AndroidManifest.xml`에 추가한다.

```kotlin
class MainApplication: Application() {

    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger()     // koin logger
            androidContext(this@MainApplication) // context injection
            modules(appModule)
        }
    }
}
```

Injection 한 ViewModel이 정상적으로 동작하는 것을 확인할 수 있다.

```
...
2022-06-10 23:08:14.681 15972-15972/com.plcoding.koinguide I/System.out: Something
...
```

만약 애플리케이션 수명주기가 아닌 액티비티 수명주기 스코프에서만 동작하게 하려면 다른 모듈이 필요하다.

```kotlin
val activityModule = module {
    // MainActivity 수명주기에서만 동작
    scope<MainActivity> {
        scoped { "Hello" }
    }
}
```

`MainApplication`에 `activityModule`을 추가한다.

```kotlin
class MainApplication: Application() {

    override fun onCreate() {
        super.onCreate()
        startKoin {
            // ...
            modules(appModule, activityModule)
        }
    }
}
```

이제 `MainActivity`에 `AndroidScopeComponent`를 상속하고, 

```kotlin
class MainActivity : ComponentActivity(), AndroidScopeComponent {

    override val scope: Scope by activityScope() // 작성해주어야 activity scope의 인스턴스 inject 가능
    private val hello by inject<String>()
    
    // ...

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        println(hello)
        // ...
    }

}
```

```
2022-06-10 23:14:43.804 16118-16118/com.plcoding.koinguide I/System.out: Hello
```

만약 두 가지 `String` 타입을 injection 하고 싶다면, qualifier를 사용한다. 다음과 같이 qualifier를 통해 구분할 수 있도록 `ActivityModule`을 작성한다.

```kotlin
val activityModule = module {
    // MainActivity 수명주기에서만 동작
    scope<MainActivity> {
        scoped(qualifier = named("hello")) { "Hello" }
        scoped(qualifier = named("bye")) { "Bye" }
    }
}
```

```kotlin
class MainActivity : ComponentActivity(), AndroidScopeComponent {

    override val scope: Scope by activityScope()
    private val hello by inject<String>(named("hello")) // 위 선언한 qualifier를 전달하여 injection 할 수 있다.
    // ...
}
```

```
2022-06-10 23:21:13.157 16236-16236/com.plcoding.koinguide I/System.out: Hello
```

## References

* [The Ultimate Guide to Koin (Dependency Injection with Kotlin) - Android Studio Tutorial](https://www.youtube.com/watch?v=EathumJlWh8&t=2s)