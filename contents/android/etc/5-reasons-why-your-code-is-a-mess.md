# 5 Reasons Why Your Code Is a Mess

코드를 망치는 5가지 이유를 알아보자.

## Reason 1

다음 코드를 보면 어떤 것을 의미하는지 쉽게 알 수 없다.

```kotlin
binding.btnCamera.setOnClickListener {
    if (viewModel.isCameraPreviewEnabled.value ||
            viewModel.hasCameraPermission.value) {
        // Why do we return in this case?
        return@setOnClickListener
    }
    viewModel.setCameraEnabled(!viewModel.isCameraPreviewEnabled.value)
}
```

이러한 간단한 boolean 표현을 위해 함수를 사용하여 가독성을 높일 수 있다.

```kotlin
binding.btnCamera.setOnClickListener {
    if (!canToggleCameraBadge()) {
        // Ahh, makes sense
        return@setOnClickListener
    }
    viewModel.setCameraEnabled(!viewModel.isCameraPreviewEnabled.value)
}

private fun canToggleCameraBadge(): Boolean {
    return viewModel.isCameraPreviewEnabled.value ||
            viewModel.hasCameraPermission.value
}
```

## Reason 2

긴 함수 이름을 사용하기 두려워하지 말아라.

```kotlin
if (shouldShowWriteToStorageRationaleAfterClickOnSave()) {
    showWriteToStorageRationale()
}
```

## Reason 3

머리를 써서 한줄로 코딩을 하려고 하는데, 이는 실제로 좋지 않은 습관이다. 나중에 코드를 다시 봐야할 때 이해하기 쉬운 코드가 좋은 코드이다. 다음 코드를 보자.

```kotlin
fun main(a: Array<String>) {
    (1..100).map { i ->
        println(
                mapOf(
                        0 to i,
                        i % 3 to "Fizz",
                        i % 5 to "Buzz",
                        i % 15 to "FizzBuzz",
                )
        )
    }
}
```

위 코드는 매우 복잡한데, 아래의 코드는 쉽게 읽을 수 있다.

```kotlin
fun main(a: Array<String>) {
    for (num in 1..100) {
        when {
            num % 15 == 0 -> println("FizzBuzz")
            num % 3 == 0 -> println("Fizz")
            num % 5 == 0 -> println("Buzz")
            else -> println(num)
        }
    }
}
```

## Reason 4

4번째 이유는 코드를 포매팅하지 않는 것이다. 안드로이드 스튜디오에 Ctrl + Option + L 단축키가 있으니 애용하자.

## Reason 5

5번째 이유는 여러 중첩을 사용하는 것이다. 중첩을 사용하면 할수록 가독성이 떨어진다.

```kotlin
fun myFunction() {
    if (conditionA) {
        doSomethingA()
        if (conditionB) {
            lifecycleScope.launch {
                try {
                    println(10f / 0f)
                } catch (e: Exception) {
                    println("oh no")
                }
            }
        }
    }
}
```

쉽게 해결하는 방법은 return 문을 사용하는 것이다. 다음 코드를 보면 쉽게 이해할 수 있다.

```kotlin
fun myFunction() {
    if (!conditionA) {
        return
    }
    doSomething()

    if (!conditionB) {
        return
    }
    divideByZero()
}

private fun divideByZero() {
    lifecycleScope.launch {
        try {
            println(10f / 0f)
        } catch (e: Exception) {
            println("oh no")
        }
    }
}
```

## References

* [5 Reasons Why Your Code Is a Mess (and How to Make it More Readable!)](https://www.youtube.com/watch?v=HpFXI6c7mmc)