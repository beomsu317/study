# Testing with Dagger-Hilt

`build.gradle` 파일에 다음 디펜던시를 추가한다.

```groovy
dependencies {
    // ... 
    androidTestImplementation 'com.google.dagger:hilt-android-testing:2.28-alpha'
    kaptAndroidTest 'com.google.dagger:hilt-android-compiler:2.28-alpha'
}
```

`androidTest`에 `HiltTestRunner` 클래스를 생성한 후 `HiltTestApplication::class.java.name`을 파라미터로 넘겨준다.

```kotlin
class HiltTestRunner: AndroidJUnitRunner() {

    override fun newApplication(
        cl: ClassLoader?,
        className: String?,
        context: Context?
    ): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

`build.gardle` 파일의 `defaultConfig`에 새로 생성한 `HiltTestRunner`를 등록한다.

```groovy
android {
    defaultConfig {
        // ...
        testInstrumentationRunner "com.androiddevs.shoppinglisttestingyt.HiltTestRunner"
    }
}
```

`TestAppModule`을 생성해 인젝션할 RoomDB를 생성한다. `Named` 어노테이션을 사용해 `test_db` 이름을 통해 인젝션되도록 설정한다.

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object TestAppModule {

    @Provides
    @Named(value="test_db")
    fun provideInMemeoryDb(@ApplicationContext context: Context) =
        Room.inMemoryDatabaseBuilder(context, ShoppingItemDatabase::class.java)
            .allowMainThreadQueries()
            .build()
}
```

`androidTest`의 `ShoppingDaoTest`를 다음과 같이 변경한다. `HiltAndroidRule`을 생성하고 DB를 인젝션 한 후 `dao`를 설정해준다. `hiltRule.inject()`를 이용해 모든 인젝션을 수행한다. `@RunWith` 어노테이션을 `@HiltAndroidTest`로 변경해준다.

```kotlin
@ExperimentalCoroutinesApi
@SmallTest
@HiltAndroidTest
class ShoppingDaoTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    @Inject
    @Named(value="test_db")
    lateinit var database: ShoppingItemDatabase
    lateinit var dao: ShoppingDao

    @Before
    fun setup() {
        hiltRule.inject()
        dao = database.shoppingDao()
    }
		...
}
```

## References

[Testing with Dagger-Hilt - Testing on Android - Part 11](https://www.youtube.com/watch?v=nOp_CEP_EjM&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=11)