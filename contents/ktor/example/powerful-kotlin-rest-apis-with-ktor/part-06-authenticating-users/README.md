# Authenticating users

지금까지 구현한 인증 로직을 사용하게 되면 유저를 검증하지 않아 다른 사람이 내 계정을 생성하는 등의 문제가 발생할 수 있다. 리얼 앱을 만들 때는 3rd-party 구글, 페이스북 등의 oauth를 사용하는 것을
추천한다.

이 코스에서는 Ktor feature로 존재하는 Authentication feature를 사용하여 인증을 구현할 것이다.

`Authentication.Configuration` 확장 함수를 만들어 `install(Authentication)`에 추가할 것이다. `Application.kt` 파일을 다음과 같이 작성해준다.

```kotlin
@Suppress("unused") // Referenced in application.conf
@kotlin.jvm.JvmOverloads
fun Application.module(testing: Boolean = false) {
    // ...
    install(Authentication) {
        configureAuth()
    }
}

private fun Authentication.Configuration.configureAuth() {
    basic {
        realm = "Note Server"
        validate { credentials -> // 이 블럭에서 사용자 검증
            val email = credentials.name
            val password = credentials.password
            if (checkPasswordForEmail(email, password)) {
                // ktor tracked whoes authenticated or not
                UserIdPrincipal(email)
            } else null
        }
    }
}
```