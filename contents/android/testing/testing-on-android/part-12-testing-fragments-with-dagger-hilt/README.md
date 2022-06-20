# Testing Fragments with Dagger-Hilt

프레그먼트 기반 테스트를 수행하기 위해 `build.gradle`에 다음 디펜던시를 추가한다.

```kotlin
debugImplementation "androidx.fragment:fragment-testing:1.3.0.-alpha08"
```

`debug` 디렉토리 생성 후 `java.com.androiddevs.shoppinglisttestingyt` 패키지를 만든다. 그리고 Hilt 테스트에 사용할 액티비티를 생성한다. 테스트에 사용되는 액티비티이기 때문에 debug 자체 manifest에만 등록하면 된다.

```kotlin
@AndroidEntryPoint
class HiltTestActivity: AppCompatActivity() {
}
```

생성한 액티비티를 manifest에 등록한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.androiddevs.shoppinglisttestingyt">

    <application>
        <activity android:name=".HiltTestActivity"
            android:exported="false"/>
    </application>

</manifest>
```

`androidTest` 디렉토리 하위에 `HiltExt` 파일을 생성한 후 다음 코드를 작성한다.

```kotlin
@ExperimentalCoroutinesApi
// inline 키워드는 구현 자체를 코드에 넣어 오버헤드를 줄일 수 있다. 이는 람다식을 파라미터로 전달할 경우 많이 사용된다.
// reified 키워드는 제네릭 파라미터의 클래스 정보에 접근할 수 있다는 의미이다. 따라서 컴파일 타임에 어떤 타입인지 알 수 있다.
inline fun <reified T : Fragment> launchFragmentInHiltContainer(
    fragmentArgs: Bundle? = null,
    themeResId: Int = R.style.FragmentScenarioEmptyFragmentActivityTheme,
    fragmentFactory: FragmentFactory? = null,
		// hilt container에서 실행된 프레그먼트에 대한 참조를 얻기 위해 사용
    crossinline action: T.() -> Unit = {}
) {
		// 여기에서 실행한 액티비티를 메인 액티비티로 만들기 위해 mainActivityIntent를 정의한다.
		// 실제 애플레케이션에는 메인 액티비티가 필요하지만 테스트에는 필요가 없기 때문이다.
    val mainActivityIntent = Intent.makeMainActivity(
        ComponentName(
            ApplicationProvider.getApplicationContext(),
            HiltTestActivity::class.java
        )
    ).putExtra(FragmentScenario.EmptyFragmentActivity.THEME_EXTRAS_BUNDLE_KEY, themeResId)

		// 위 인텐트를 사용해 activity에 대한 참조를 얻고
    ActivityScenario.launch<HiltTestActivity>(mainActivityIntent).onActivity { activity ->
        fragmentFactory?.let {
            activity.supportFragmentManager.fragmentFactory = it
        }
        val fragment = activity.supportFragmentManager.fragmentFactory.instantiate(
            Preconditions.checkNotNull(T::class.java.classLoader),
            T::class.java.name
        )
				// 인자를 설정하고
        fragment.arguments = fragmentArgs

				// 해당 프레그먼트를 실행
        activity.supportFragmentManager.beginTransaction()
            .add(android.R.id.content, fragment, "")
            .commitNow()

				// 그 후 해당 프레그먼트에 대한 참조를 얻는다
        (fragment as T).action()
    }
}
```

다음과 같이 Fragment 테스트를 수행할 수 있다.

```kotlin
@ExperimentalCoroutinesApi
@SmallTest
@HiltAndroidTest
class ShoppingDaoTest {
    // ...
    @Test
    fun testLaunchFragmentInHiltContainer() {
        launchFragmentInHiltContainer<ShoppingFragment> {

        }
    }
    // ...
}
```

## References

* [Testing Fragments with Dagger-Hilt - Testing on Android - Part 12](https://www.youtube.com/watch?v=k4zG93ogWFY&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=12)