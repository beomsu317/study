# Handle One-Time Events with Kotlin Channel

에러 메시지를 보여주고 싶은 경우 Livedata나 State flow를 사용하여 전달할 수 있다. 하지만 단말기를 회전하게 되면 이벤트가 한 번 더 실행하게 된다. 따라서 스낵바나, 토스트를 보여주는 경우 단말을
회전했을 때 한 번 더 보여지게 된다. 이 이벤트들은 모두 한 번만 실행되어야 하는 이벤트들이다. 이는 코틀린 `Channel`을 통해 해결할 수 있다.

버튼을 클릭하면 클릭 이벤트를 `ViewModel`로 보내고 `ViewModel`에서 UI에 스낵바를 출력하도록 이벤트를 보내는 One time event 예제를 만들어보자.
우선 [이 레포지토리](https://github.com/philipplackner/KotlinChannels)를 클론한다.

`activity_main.xml`에 버튼 하나를 만들어준다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
                                                   xmlns:app="http://schemas.android.com/apk/res-auto"
                                                   xmlns:tools="http://schemas.android.com/tools"
                                                   android:layout_width="match_parent"
                                                   android:layout_height="match_parent"
                                                   tools:context=".MainActivity">

    <Button
            android:id="@+id/btnShowSnackbar"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Show Snackbar"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintLeft_toLeftOf="parent"
            app:layout_constraintRight_toRightOf="parent"
            app:layout_constraintTop_toTopOf="parent"/>

</androidx.constraint>
```

`MainViewModel`에서 에러 이벤트를 보여주기 위해 이벤트 채널을 생성한다.

```kotlin
class MainViewModel : ViewModel() {

    sealed class MyEvent {
        data class ErrorEvent(val message: String) : MyEvent()
    }

    private val eventChannel = Channel<MyEvent>()
    val eventFlow = eventChannel.receiveAsFlow()

    fun triggerEvent() = viewModelScope.launch {
        eventChannel.send(MyEvent.ErrorEvent("This is an error"))
    }
}
```

`MainActivity`에서 이벤트를 observe하여 이벤트 발생 시 스낵바를 보여주도록 작성한다.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        viewModel = ViewModelProvider(this).get(MainViewModel::class.java)

        binding.btnShowSnackbar.setOnClickListener {
            viewModel.triggerEvent()
        }

        lifecycleScope.launchWhenStarted {
            viewModel.eventFlow.collect { event ->
                when (event) {
                    is MainViewModel.MyEvent.ErrorEvent -> {
                        Snackbar.make(binding.root, event.message, Snackbar.LENGTH_LONG).show()
                    }
                }
            }
        }
    }
}
```

<div align="center">
<img src="img/result.gif">
</div>

## References

* [Handle One-Time Events with Kotlin's Channels - Android Studio Tutorial](https://www.youtube.com/watch?v=6v8iJDJdtMc)