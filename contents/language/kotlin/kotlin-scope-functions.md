# Kotlin Scope Functions - Let, Also, Apply, Run, With

코틀린을 처음 시작한 경우 Let, Also, Apply, Run, With 키워드에 대해 헷갈릴 수 있다. 이를 언제 사용하는지 알아보자.

## Let

Let 키워드는 보통 null을 체크할 때 사용한다. 다음 예제를 보자.

```kotlin
class MainActivity : ComponentActivity() {

    private var number: Int? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            if (number != null) {
                val number2 = number + 1
            }
        }
    }
}
```

`val number2 = number + 1` 코드에서 에러가 발생하는데, 이는 `number` global 변수가 다른 스레드에서 사용될 수 있다는 것을 의미한다. 즉, 다른 스레드에서 `number`에 null을 할당할 수 있다는 것이다. 코틀린은 null-safe 언어이기 때문에 오류가 발생하는 것이다. 다음과 같은 방식으로 null이 아님을 알려줄 수 있지만, 지금도 다른 스레드에서 `number`에 충분히 null을 할당할 수 있기 때문에 문제가 될 수 있다.

```kotlin
val number2 = number!! + 1
```

다음과 같이 `?`를 사용해 `number`가 null이 아닌 경우에만 let 블록을 실행하도록 할 수 있다. 만약 블록을 실행하는 도중 `number`가 변경되는 경우 let 블록에 들어오기 전 `number`의 값으로 수행되기 때문에 문제가 되지 않는다.

```kotlin
number?.let {
		val number2 = it + 1
}
```

다음과 같이 변수에도 할당할 수 있다.

```kotlin
val x = number?.let {
		val number2 = it + 1
		number2
} ?: 3
```

## Also

let과 비슷하지만 마지막 라인을 반환하는 let과는 다르게 also는 마지막 라인을 반환하지 않는다. 또한 블록의 코드를 통해 전달된 요소가 변경되지 않는다. 다음과 같은 코드가 있을 때 `i * i` 코드는 적용되어 반환되지만 `i++` 코드의 경우 적용되지 않는다. 로깅할 때 주로 사용한다.

```kotlin
private var i = 0
fun getSquareI() = (i * i).also {
		i++  
}
```

## Apply

apply 키워드는 객체를 변경할 때 사용된다. 만약 객체에 대해 많은 변경이 있을 경우 apply를 사용해야 한다.

블록 내에서 `it`와 같은 키워드를 사용하지 않고 직접 객체 내 함수를 사용할 수 있다. 블록 안의 변경들이 적용된 객체를 반환한다.

```kotlin
val intent = Intent().apply {
		putExtra("", "")
    putExtra("", 0)
    action = ""
}
```

## Run

run 키워드의 경우 apply 키워드와 유사하지만 마지막 라인의 값을 반환한다.

```kotlin
val intent = Intent().run {
		putExtra("", "")
    putExtra("", 0)
    action = ""
    this
}
```

## With

with 키워드의 경우 표현만 살짝 다르고 Run과 완전 동일하다.

```kotlin
with(Intent()) {
		this            
}
```

## References

* [Let, Also, Apply, Run, With - Kotlin Scope Functions](https://www.youtube.com/watch?v=Vy-dS2SVoHk&list=WL&index=1)