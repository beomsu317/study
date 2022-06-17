# Resource & Event wrapper class

API 응답을 위해 사용되는 `Resource` 클래스와 스낵바 등의 one time event에 대한 `Event` 래퍼 클래스를 만들어보자.

우선 `other` 패키지에 `Resource` data class를 생성하고 작성하자.

```kotlin
// out 키워드는 Number로 Type이 전달되는 경우 하위 타입인 Int도 적용 가능하다고 알려주는 것
data class Resource<out T>(val status: Status, val data: T?, val message: String?) {
    companion object {
        fun <T> success(data: T?): Resource<T> {
            return Resource(Status.SUCCESS, data, null)
        }
        fun <T> error(msg: String, data: T?): Resource<T> {
            return Resource(Status.ERROR, data, msg)
        }
        fun <T> loading(data: T?): Resource<T> {
            return Resource(Status.LOADING, data, null)
        }
    }
}

enum class Status {
    SUCCESS,
    ERROR,
    LOADING
}
```

이번엔 one time event를 위한 `Event` 클래스를 `other` 패키지에 만들자.

```kotlin
open class Event<out T>(private val content: T) {

    var hasBeenHandled = false
        private set

    fun getContentIfNotHandled() = if (hasBeenHandled) {
        null
    } else {
        hasBeenHandled = true
        content
    }

    // Even if content already handled peekContent can provide content
    fun peekContent() = content
}
```