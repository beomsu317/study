# The Ultimate Dagger-Hilt Guide (Dependency Injection)

DI 및 Dagger-Hilt의 사용법에 대해 알아본다.

## Dependency Injection

디펜던시 인젝션은 싱글톤, 팩토리 패턴과 같은 디자인 패턴이다. 디펜던시 인젝션은 결국 인스턴스를 주입하는 것으로 보면 된다.

## Dagger-Hilt

Dagger-Hilt는 구글에서 제공하는 라이브러리이다. 이 라이브러리는 디펜던시 인젝션을 수행할 때 많은 기능과 유연함을 제공한다. Dagger2는 너무 복잡하기 때문에 이를 간단하게 사용할 수 있는
Dagger-Hilt를 만들었다. Dagger-Hilt는 쉽게 디펜던시의 생명주기를 관리할 수 있다. Dagger-Hilt는 컴파일 타임에 인젝션을 수행한다. 따라서 런타임에 어디에 어떤 디펜던시가 들어가야 알고
있다.

## Example

예제를 통해 디펜던시를 인젝션하는 방법을 알아보자.

### Dependencies

다음과 같이 디펜던시를 추가한다.

```groovy
plugins {
    // ...
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
}
dependencies {
    // ...
    implementation "com.google.dagger:hilt-android:2.40.5"
    kapt "com.google.dagger:hilt-android-compiler:2.40.5"
    implementation "androidx.hilt:hilt-lifecycle-viewmodel:1.0.0-alpha03"
    kapt "androidx.hilt:hilt-compiler:1.0.0"
    implementation 'androidx.hilt:hilt-navigation-compose:1.0.0'

    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:5.0.0-alpha.3'
}
```

`data/remote` 패키지를 생성한 후 더미 API인 `MyApi`를 생성한다.

```kotlin
interface MyApi {

    @GET("test")
    suspend fun doNetworkCall()
}
```

`domain/repository` 패키지를 생성한 후 `MyRepository` 인터페이스를 생성한다.

```kotlin
interface MyRepository {

    suspend fun doNetworkCall()
}
```

`data/remote/repository` 패키지 생성 후 위 인터페이스를 구현한 `MyRepositoryImpl` 클래스를 생성한다.

```kotlin
class MyRepositoryImpl(
    private val api: MyApi
) : MyRepository {

    override suspend fun doNetworkCall() {
        api.doNetworkCall()
    }
}
```

여기서 `MyApi`를 가져와 사용해야 하는데 이 때 디펜던시 인젝션이 필요하다.

Dagger-Hilt로 인스턴스 생성을 위임하기 위해 모듈에 위 인젝션할 클래스를 정의한다. 모듈을 기능 기반, 스코프 기반으로 선언하는 것을 추천한다.

`di` 패키지 생성 후 `AppModule` object class를 생성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class) // 스코프 정의
object AppModule {

    @Provides
    @Singleton
    fun provideMyApi(): MyApi {
        return Retrofit.Builder()
            .baseUrl("https://test.com")
            .build()
            .create(MyApi::class.java)
    }
}
```

이제 앱에서 `MyApi` 인젝션이 필요한 곳에 인스턴스를 제공할 수 있다.

`@InstallIn(SingletonComponent::class)`는 디펜던시에 대한 생명주기를 나타내며 `@Singleton`은 스코프를 나타낸다. 즉, 앱의 생명주기에 하나의 인스턴스를 제공한다는 의미이다.

이제 `MainViewModel`을 생성하고 `MyRepository`를 인젝션해보자.

```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: MyRepository
) : ViewModel() {
}
```

`MyRepository`도 동일하게 `AppModule`에 정의해준다.

```kotlin
@Module
@InstallIn(SingletonComponent::class) // 스코프 정의
object AppModule {

    // ...

    @Provides
    @Singleton
    fun provideMyRepository(api: MyApi): MyRepository {
        return MyRepositoryImpl(api)
    }
}
```

Android 컴포넌트 클래스(여기선 액티비티)에 인젝션을 수행해야 하기 때문에 `@AndroidEntryPoint` 어노테이션을 `MainActivity`에 추가해준다.

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val viewModel = hiltViewModel<MyViewModel>()

        }
    }
}
```

애플리케이션 컨텍스트를 인젝션하기 위해 `Application` 클래스를 생성하고 `@HiltAndroidApp` 어노테이션을 추가해야 한다. 메니페스트에도 애플리케이션 클래스를 선언해준다.

```kotlin
@HiltAndroidApp
class MyApp : Application()
```

## How to provide dependencies of the same type

만약 두 개의 동일한 타입의 클래스를 다르게 제공해야 하는 경우는 어떻게 제공되어야 할까? 다음과 같이 `@Named` 어노테이션을 통해 동일한 타입의 디펜던시를 구별할 수 있다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    // ...

    @Provides
    @Singleton
    fun provideMyRepository(
        api: MyApi,
        @Named("hello1") hello1: String
    ): MyRepository {
        return MyRepositoryImpl(api)
    }

    @Provides
    @Singleton
    @Named("hello1")
    fun provideString1() = "Hello 1"

    @Provides
    @Singleton
    @Named("hello2")
    fun provideString2() = "Hello 2"
}
```

## Easier way to provide abstract/interface class

Abstract/Interface 클래스에 대해 더욱 간단하게 제공하는 방법이 있다. 다음은 이에 대한 디펜던시를 제공하는 다른 방법이다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {

    @Binds
    @Singleton
    abstract fun bindMyRepository(
        myRepositoryImpl: MyRepositoryImpl
    ): MyRepository
}
```

`MyRepositoryImpl` 클래스에 `@Inject` 어노테이션을 추가해준다.

```kotlin
class MyRepositoryImpl @Inject constructor(
    private val api: MyApi
) : MyRepository {
    // ...
}
```

이렇게 구현하면 이전에 구현했던 방식과 동일한 디펜던시를 제공하지만, 더 적은 코드로 Abstract/Interface 클래스를 제공할 수 있다.

## How to inject dependencies in service

서비스는 생성자를 구현할 수 없기 때문에, 필드 인젝션이 필요하다.

```kotlin
@AndroidEntryPoint
class MyService : Service() {

    @Inject // 필드 인젝션
    lateinit var repository: MyRepository

    override fun onCreate() {
        super.onCreate()
        // 인젝션된 인스턴스 사용
        repository.doNetworkCall()
    }

    // ...
}
```

## Lazy injection

객체를 생성하는 시점을 미루는 방법이다. 대부분의 디펜던시는 인젝션 될 때 생성된다. 하지만 lazy는 인젝션될 때가 아닌 실제로 사용될 때 인젝션된다.

```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: Lazy<MyRepository>
) : ViewModel() {

    init {
        repository.get() // MyRepository 반환
    }
}
```

예를 들어, Interceptor가 auth token이 필요로 할 때, 컴파일 타임에는 token에 대해 알 수 없으나 유저가 로그인한 후 token을 얻기 때문에 로그인할 때까지 delay 하는데 사용될 수
있다.

## References

* [The Ultimate Dagger-Hilt Guide (Dependency Injection) - Android Studio Tutorial](https://www.youtube.com/watch?v=bbMsuI2p1DQ&t=3s)