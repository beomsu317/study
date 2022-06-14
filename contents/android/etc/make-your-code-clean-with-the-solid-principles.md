# Make Your Code Clean With the SOLID Principles

SOLID는 코드를 깔끔하고 유지보수가 쉽게 만들기 위한 5가지 원칙이다. 각 문자는 각자 하나의 의미를 나타낸다.

## **Single-Responsibility Principle**

다음 레포지토리에서 잘못된 부분을 찾아보자.

```kotlin
class MainRepository(
        private val auth: FirebaseAuth
) {
    suspend fun loginUser(email: String, password: String) {
        try {
            auth.signInWithEmailAndPassword(email, password)
        } catch (e: Exception) {
            val file = File("errors.txt")
            file.appendText(
                    text = e.message.toString()
            )
        }
    }
}
```

위 코드는 SOLID 원칙의 첫 번째(S) 원칙을 위배했다. S는 Single responsibility를 의미하며, 하나의 클래스 또는 함수는 하나의 책임만 가져야 한다는 의미이다. 위 함수를 적절히 수정하려면
다음과 같이 에러 로그를 출력하는 클래스를 만들고 이 클래스를 이용해야 한다.

```kotlin
class FileLogger {
    fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}
```

```kotlin
class MainRepository(
        private val auth: FirebaseAuth,
        private val fileLogger: FileLogger
) {
    suspend fun loginUser(email: String, password: String) {
        try {
            auth.signInWithEmailAndPassword(email, password)
        } catch (e: Exception) {
            fileLogger.logError(e.message.toString())
        }
    }
}
```

## **Open-Closed Principle**

Open-Closed 원칙은 확장에 열려있어야 하고, 변경에 닫혀있어야 한다는 의미이다. 다음과 같은 클래스가 있다고 하자. 로그로 저장되는 파일명을 변경하기 위해 클래스 내의 `errors.txt`
→ `errors2.txt`로 변경하는 경우 예상치 못한 동작이 일어날 수 있다.

```kotlin
class FileLogger {
    fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}
```

따라서 저장되는 파일명을 변경하고 싶은 경우 다음과 같이 해당 클래스를 확장(상속)하여 사용해야 한다. 내부적인 변경에 닫혀있고, 확장에 열려있는 것이 2번째 원칙의 의미이다.

```kotlin
open class FileLogger {
    open fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}

class CustomErrorFileLogger : FileLogger() {

    override fun logError(error: String) {
        val file = File("my_custom_error_file.txt")
        file.appendText(
                text = error
        )
    }
}
```

## **Liskov Substitution Principle**

부모 클래스를 자식 클래스로 치환해도 부모 클래스를 사용하는데 문제가 없어야 하는 원칙이다. 다음 코드를 보면 `logError` 함수를 오버라이딩하지 않고 있는데, 이 때 부모 클래스를 자식 클래스로 치환하는
경우 `logError`를 사용할 수 없다.

```kotlin
open class FileLogger {
    open fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}

class CustomErrorFileLogger : FileLogger() {

    fun customErrorLog(error: String) {
        val file = File("my_custom_error_file.txt")
        file.appendText(
                text = error
        )
    }
}
```

이는 3번째 원칙에 위배되며, `logError`를 사용해 치환이 가능하도록 다음과 같이 구현해야 한다.

```kotlin
open class FileLogger {
    open fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}

class CustomErrorFileLogger : FileLogger() {

    override fun logError(error: String) {
        val file = File("my_custom_error_file.txt")
        file.appendText(
                text = error
        )
    }
}
```

## **Interface Segregation Principle**

클라이언트(구현체) 자신이 사용하지 않는 메서드에 의존하지 않아야 하는 원칙이다. 즉, 인터페이스를 클라이언트(구현체)에 특화되도록 분리시키는 법칙이다. 만약 다음 인터페이스가 있다고 하면 두 개의 함수를 모두
구현해주어야 한다. `printLogs()` 함수가 상속하는 모든 구현체에 구현해야 되는 것이면 문제가 되지 않는다. 하지만 특정 클래스만 필요한 경우 이 함수를 강제할 수 없다. 해당 함수를 선택적이도록 하려면
다음과 같이 빈 함수를 만들어주면 된다.

```kotlin
interface FileLogger {
    
    fun printLogs() {

    }
    
    fun logError(error: String) {
        val file = File("errors.txt")
        file.appendText(
                text = error
        )
    }
}

class CustomErrorFileLogger : FileLogger {

    override fun logError(error: String) {
        val file = File("my_custom_error_file.txt")
        file.appendText(
                text = error
        )
    }
}
```

## **Dependency Inversion Principle**

자신보다 변하기 쉬운 것에 의존하던 것을 인터페이스나 상위 클래스를 두어 쉬운 것의 변화에 영향받지 않게 하는 것이 이 원칙의 의미이다. 다음 코드에서 `FirebaseAuth`를 직접 사용하여 auth를 수행하는데
이는 이 원칙에 위배된다.

```kotlin
class MainRepository(
        private val auth: FirebaseAuth,
        private val fileLogger: FileLogger
) {
    suspend fun loginUser(email: String, password: String) {
        try {
            auth.signInWithEmailAndPassword(email, password)
        } catch (e: Exception) {
            fileLogger.logError(e.message.toString())
        }
    }
}

class MainRepository @Inject constructor(
        private val auth: Authenticator,
        private val fileLogger: FileLogger
) {
    suspend fun loginUser(email: String, password: String) {
        try {
            auth.signInWithEmailAndPassword(email, password)
        } catch (e: Exception) {
            fileLogger.logError(e.message.toString())
        }
    }
}
```

다음과 같이 `Authenticator` 인터페이스를 만들고 이 인터페이스에 대해 하위 클래스를 구현해준다.

```kotlin
interface Authenticator {

    suspend fun signInWithEmailAndPassword(email: String, password: String)

}

class FirebaseAuthenticator : Authenticator {

    override suspend fun signInWithEmailAndPassword(email: String, password: String) {
        FirebaseAuth.getInstance().signInWithEmailAndPassword(email, password)
    }

}

class CustomApiAuthenticator : Authenticator {

    override suspend fun signInWithEmailAndPassword(email: String, password: String) {
        // auth with rerofit process
    }

}
```

`auth`에 `Authenticator`를 사용해 인증하는 경우 `Authenticator`를 싱글톤으로 만들고, Retrofit 또는 Firebase, Fake 어떤 인증이라도 이 클래스는 어떻게 인증하는지에
대해 몰라도 된다. 단지 `Authenticator`만 사용하면 된다.

```kotlin
class MainRepository @Inject constructor(
        private val auth: Authenticator,
        private val fileLogger: FileLogger
) {
    suspend fun loginUser(email: String, password: String) {
        try {
            auth.signInWithEmailAndPassword(email, password)
        } catch (e: Exception) {
            fileLogger.logError(e.message.toString())
        }
    }
}
```

## References

* [Make Your Code Clean With the SOLID Principles](https://www.youtube.com/watch?v=t8VTLxMsufU)