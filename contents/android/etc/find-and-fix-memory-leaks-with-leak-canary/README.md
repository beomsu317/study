# Find and Fix MEMORY LEAKS with Leak Canary in Android 👀

안드로이드 개발 시 일어나는 메모리 릭에 대해 알아보자.

자바에는 백그라운드로 동작하는 가비지 컬렉터가 존재한다. 가비지 컬렉터는 사용하지 않는 오브젝트나 변수들을 모아 메모리에서 free 해주는 작업을 수행한다. c 또는 c++에서 메모리 사용 후 수동으로 free를
해줘야 하지만 자바는 가비지 컬렉터가 자동으로 그 역할을 수행한다.

하지만 앱에서 destroy 되었는데도 정리되지 않는 특정 오브젝트가 있다. 이렇게 되면 메모리르 점유하고 있기 때문에 좋지 않다.

다음과 같이 `companion object`에 `Context`를 변수로 저장하는 경우 워닝이 발생한다. 이렇게 구현하면 액티비티가 clear 되도 `context` 변수가 clear 되지 않는다.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }

    companion object {
        lateinit var context: Context
    }
}
```

`SecondActivity`에서 `MainActivity.context`에 `this`로 할당된 후 `SecondActivity`가 destroy 되는 경우 이 액티비티의 `context`
는 `MainActivity.context`에 여전히 남아있게 된다. 따라서 가비지 컬렉터는 `context`가 여전히 사용되는 줄 알고 clear 하지 않는다.

```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MainActivity.context = this
    }
}
```

이제 `MainActivity`에서 `SecondActivity`를 시작하도록 작성해보자.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Intent(this, SecondActivity::class.java).also {
            startActivity(it)
        }
    }

    companion object {
        lateinit var context: Context
    }
}
```

실행 후 `SecondActivity`에서 뒤로가기를 `SecondActivity`는 destory 되었지만 `context`가 `MainActivity.context`에 남아있어 누르면 메모리 릭이 발생한다.

하지만 지금 상태에선 이 메모리 릭을 탐지하는 방법이 없다. 하지만 **leakcanary**라는 라이브러리가 존재하며 프로젝트에 포함하여 자동으로 메모리 릭을 탐지할 수 있다.

```groovy
debugImplementation "com.squareup.leakcanary:leakcanary-android:2.8.1"
```

<div align="center">
<img src="img/result.gif" width="40%">
</div>

## References

* [Find and Fix MEMORY LEAKS with Leak Canary in Android 👀](https://www.youtube.com/watch?v=VvkRe9vP5Oc)