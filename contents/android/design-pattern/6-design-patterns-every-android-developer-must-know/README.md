# 6 Design Patterns Every Android Developer Must Know

안드로이드 개발자가 알아야 할 6가지 디자인 패턴에 대해 알아본다.

## Singleton Pattern

앱 실행 시 오직 하나의 인스턴스만 존재하게 하는 디자인 패턴이다. 코틀린에선 다음과 같이 `object`를 사용해 쉽게 싱글톤을 만들 수 있다.

```kotlin
object Singleton {

    fun doSomething() {

    }
}
```

Tools -> Kotlin -> Show Kotlin Bytecode를 클릭하여 위 코드를 디컴파일하면 다음과 같다.

```java
public final class Singleton {
    @NotNull
    public static final Singleton INSTANCE;

    public final void doSomething() {
    }

    private Singleton() {
    }

    static {
        Singleton var0 = new Singleton();
        INSTANCE = var0;
    }
}
```

## Factory Pattern

ViewModel에 대해 알고 있다면 ViewModel Factory에 대해서도 들어봤을 수 있다. Factory 패턴은 유사한 타입의 객체에 대해 구별하고 원하는 것을 생성하기 위해 사용한다.

다음과 같은 코드가 있다고 가정하자.

```kotlin
enum class DialogType {
    DIALOG_CREATE_CHAT,
    DIALOG_DELETE_MESSAGE,
    DIALOG_EDIT_MESSAGE,
}

sealed class Dialog {
    object CreateChatDialog : Dialog()
    object DeleteMessageDialog : Dialog()
    object EditMessageDialog : Dialog()
}

object DialogFactory {
    fun createDialog(dialogType: DialogType): Dialog {
        return when (dialogType) { // 타입에 따라 다른 객체 반환
            DialogType.DIALOG_CREATE_CHAT -> Dialog.CreateChatDialog
            DialogType.DIALOG_DELETE_MESSAGE -> Dialog.DeleteMessageDialog
            DialogType.DIALOG_EDIT_MESSAGE -> Dialog.EditMessageDialog
        }
    }
}
```

## Builder Pattern

다음 코드를 보자. 빌더 패턴은 `Hamburger` 클래스의 이너 클래스이며, 이 빌더 클래스는 각 재료에 대해 설정할 수 있는 함수를 가지고 있다. 즉, 클래스 내의 속성 값에 대해 설정할 수 있는 쉬운 방법을
제공한다.

```kotlin
class Hamburger private constructor(
        val cheese: Boolean,
        val beef: Boolean,
        val onions: Boolean
) {
    class Builder {
        private var cheese: Boolean = true
        private var beef: Boolean = true
        private var onions: Boolean = false

        fun cheese(value: Boolean) = apply { cheese = value }
        fun beef(value: Boolean) = apply { beef = value }
        fun onions(value: Boolean) = apply { onions = value }
        fun build(value: Boolean) = Hamburger(cheese, beef, onions)
    }
}

fun main() {
    val hamburger = Hamburger.Builder()
            .beef(true)
            .cheese(false)
            .onions(true)
            .build()
}
```

하지만 코틀린에는 named parameter가 존재하므로 다음과 같이 간단하게 구현할 수 있다.

```kotlin
data class Hamburger(
        val cheese: Boolean = true,
        val beef: Boolean = true,
        val onions: Boolean = false
)

fun main() {
    val hamburger = Hamburger(
            cheese = false,
            onions = true
    )
}
```

## Facade Pattern

Facade Pattern은 불필요하게 보여지는 코드를 숨기고 필요한 코드만 보여주는 패턴이다. 다음 Retrofit API를 보자. `getHamburgers()` API를 정의했는데, 실제 구현되는 코드(네트워크
호출)는 외부에서 알 필요가 없다.

```kotlin
interface MyApi {

    @GET("hamburgers")
    suspend fun getHamburgers(): List<Hamburger>
}
```

## Dependency Injection Pattern

Dependency Injection 패턴은 원하는 부분에 인스턴스 객체를 전달하는 것이다. 생성자 파라미터에 constructor injection을 통해 inject 할 수도 있다.

Dagger는 이 패턴을 쉽게 사용하게 해주는 라이브러리이다. 다음과 같이 어떻게 인스턴스를 생성해야 할지에 대한 모듈을 만들어주어야 한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Singleton
    @Provides
    fun provideHamburger(): Hamburger {
        return Hamburger.Builder()
                .beef(true)
                .cheese(false)
                .onions(true)
                .build()
    }
}
```

이 패턴을 사용하면 쉽게 의존되는 객체를 변경할 수 있을 뿐더러 테스팅도 간단하게 구현할 수 있다.

## Adapter Pattern

Recycler View에 사용되는 어댑터 패턴이다. 스피너에 사용되는 array adapter도 있다. 어댑터 패턴은 아이템 리스트를 받아 UI에 부착시키는 역할 등을 수행한다.

```kotlin
class HamburgerAdapter(
        private val hamburgers: List<Hamburger>
) : RecyclerView.Adapter<HamburgerAdapter.HamburgerViewHolder>() {

    inner class HamburgerViewHolder(view: View) : RecyclerView.ViewHolder(view)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): HamburgerViewHolder {
        return HamburgerViewHolder(
                LayoutInflator.from(parent.context).inflate(
                        0,
                        parent,
                        false
                )
        )
    }
    
    override fun onBindViewHolder(holder: HamburgerViewHolder, position: Int) {
        
    }
    
    override fun getItemCount(): Int {
        return hamburgers.size
    }
}
```

## References

* [6 Design Patterns Every Android Developer Must Know](https://www.youtube.com/watch?v=4cCwuBsqfTI&t=2s)