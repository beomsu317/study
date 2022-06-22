# Top 7 Clean Code Tips

클린 코드를 작성하기 위한 7가지 팁을 알아보자.

## The Rule of Three

반복되는 코드 블록이나 라인이 있는 경우 굉장히 나쁜 코드이다. 만약 해당 코드가 변경되면 나머지 동일한 코드들도 변경해주어야 한다. The Rule of Three는 3번째 똑같은 코드가 나타나는 경우 리팩토링을 통해 중복되는 코드를 제거해야 한다.

## Don’t comment what happens

다음 코드가 있다고 가정하자.

```kotlin
val persons = listOf(
    Person("Philipp", 23),
    Person("Chris", 27),
    Person("John", 56)
)

fun findPerson(name: String): Person? {
    // Loop through persons list to find the right person
    for (person in persons) {
        if (person.name == name) {
            return person
        }
    }
    return null
}
```

what 보다는 why에 집중하여 주석을 적어주는 것이 좋다.

## Don’t be afraid of long names

위 코드의 경우 `findPersonByName`으로 함수명을 변경하면 좀 더 알아보기 쉽다.

```kotlin
fun findPersonByName(name: String): Person? {
    // Loop through persons list to find the right person
    for (person in persons) {
        if (person.name == name) {
            return person
        }
    }
    return null
}
```

필요하다면 긴 함수명을 작성하는 것도 좋다.

## Avoid genius code

다음 코드가 있다고 가정하자.

```kotlin
fun fizzBuzz(a: Array<String>) {
    (1..100).map { i ->
        println(
            mapOf(
                0 to i,
                i % 3 to "Fizz",
                i % 5 to "Buzz",
                i % 15 to "FizzBuzz"
            )[0]
        )
    }
}

fun fizzBuzz2() {
    for (i in 1..100) {
        when {
            i % 15 == 0 -> println("FizzBuzz")
            i % 5 == 0 -> println("Buzz")
            i % 3 == 0 -> println("Fizz")
            else -> println(i)
        }
    }
}
```

첫 번째 함수는 알아보기 힘들지만, 두 번째 함수의 경우 한 눈에 알아보기 쉽다.

## The law of Demeter

다음과 같은 코드가 있다고 가정하자.

```kotlin
class MyTimer(
    val startingTime: String
) {
    val timer = Timer()

    fun start() {
        timer.schedule(
            object: TimerTask() {
                override fun run() {
                    println("Timer is running")
                }
            },
            100L
        )
    }
}

class Clock(
    timer: MyTimer
) {
    init {
        timer.timer.cancel()
    }
}
```

`Clock` 클래스는 `MyTimer` 클래스를 `timer` 변수에 가져오고 `init` 블럭에선 `timer.timer.cancel()`을 호출하고 있다. 이렇게 `timer.timer` 방식으로 참조되는 부분을 피해야 한다. 따라서 다음과 같은 방식으로 작성하는게 깔끔하다.

```kotlin
class MyTimer(
    val startingTime: String
) {
    private val timer = Timer()

    fun cancel() {
        timer.cancel()
    }

    fun start() {
        timer.schedule(
            object: TimerTask() {
                override fun run() {
                    println("Timer is running")
                }
            },
            100L
        )
    }
}

class Clock(
    timer: MyTimer
) {
    init {
        timer.cancel()
    }
}
```

## Avoid side-effects

side-effects는 함수의 범위를 벗어나는 것을 의미한다. 다음 코드를 보자.

```kotlin
class MyTimer2 {

    suspend fun startTicking() {
        while (true) {
            delay(1000L)
            time++
        }
    }
}

var time = 0
```

`MyTimer2` 클래스 외적인 것에 변화가 있기 때문에 이를 side-effect라 한다. 이것은 애플리케이션에서 예상하지 못한 문제를 발생시킨다. 이러한 전역 변수 또는 `static` 변수를 사용할 때는 유의하도록 하자.

## Avoid if-else

다음 코드를 보자.

```kotlin
class MyTimer3 {

    var isActive = false

    suspend fun startTicking() {
        if (!isActive) {
            isActive = true
            while (true) {
                delay(1000L)
                time++
            }
        }
    }
}
```

여기서 문제는 `if (!isActive)`를 사용함으로써 하나의 인덴테이션이 추가된다는 것이다. 이렇게 되면 더욱 복잡해져 가독성이 떨어지게 된다. 다음과 같이 `isActive`인 경우 반환함으로써 복잡성을 낮출 수 있다.

```kotlin
class MyTimer3 {

    var isActive = false

    suspend fun startTicking() {
        if (isActive) {
            return
        }
        isActive = true
        while (true) {
            delay(1000L)
            time++
        }
    }
}
```

## References

* [My Top 7 Clean Code Tips for Android Developers (You Can Immediately Apply These!)](https://www.youtube.com/watch?v=d3YfoIDS46E)