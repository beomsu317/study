# Status pages

`StatusPages` 플러그인은 Ktor 애플리케이션이 적절한 실패 상태를 응답하도록 허용한다. 플러그인은 다음과 같이 설치할 수 있다.

```kotlin
fun Application.main() {
    install(StatusPages)
}
```

3개의 주 설정 옵션이 있다.

1. `exceptions` - 매핑된 익셉션 클래스를 기반으로 응답 구성
2. `status` - status code 값에 대한 응답 구성
3. `statusFile` - classpath의 표준 파일 응답 구성

# Exceptions

예외 구성은 예외가 발생하는 호출에 대한 간단한 인터셉션 패턴을 제공할 수 있다. 기본적으로 500 HTTP status code를 구성할 수 있다.

```kotlin
install(StatusPages) {
    exception<Throwable> { cause ->
        call.respond(HttpStatusCode.InternalServerError)
    }
}
```

더 구체적인 응답은 더 복잡한 사용자 상호 작용을 허용한다.

```kotlin
install(StatusPages) {
    exception<AuthenticationException> { cause ->
        call.respond(HttpStatusCode.Unauthorized)
    }
    exception<AuthorizationException> { cause ->
        call.respond(HttpStatusCode.Forbidden)
    }
}
```

이러한 커스터마이즈는 custom status code와 쌍을 이룰 때 작동할 수 있다. 예를 들어, 권한 없는 유저에게 로그인 페이지를 제공될 때.

각 호출은 하나의 익셉션 핸들러에 잡히며, 이는 발생한 예외의 객체 그래프에서 가장 가까운 예외이다. 동일한 객체 계층에서 다수의 익셉션이 처리될 때 하나만 실행된다.

```kotlin
install(StatusPages) {
    exception<IllegalStateException> { cause ->
        fail("will not reach here")
    }
    exception<ClosedFileSystemException> {
        throw IllegalStateException()
    }
}
intercept(ApplicationCallPipeline.Fallback) {
    throw ClosedFileSystemException()
}
```

단일 처리는 재귀적인 콜 스택을 피하는 것을 의미한다. 예를 들어, 이 구성으로 인해 생성된 `IllegalStateException`이 클라이언트에 전파된다.

```kotlin
install(StatusPages) {
    exception<IllegalStateException> { cause ->
        throw IllegalStateException("")
    }
}
```

# **Logging exceptions**

위의 처리를 추가하면 route에 의해 생성된 예외가 무시될 것이다. 실제 생성된 에러를 로깅하기 위해 수동으로 `cause`를 로깅하거나, 아래와 같이 re-throw 할 수 있다.

```kotlin
install(StatusPages) {
    exception<Throwable> { cause ->
        call.respond(HttpStatusCode.InternalServerError, "Internal Server Error")
        throw cause
    }
}
```

# **Status**

`status` 구성은 애플리케이션 내에서 status 응답을 위한 커스텀 액션을 제공한다. 아래는 응답 텍스트에 HTTP status code에 대한 정보를 제공하는 기본적인 구성이다.

```kotlin
install(StatusPages) {
    status(HttpStatusCode.NotFound) {
        call.respond(
            TextContent(
                "${it.value} ${it.description}",
                ContentType.Text.Plain.withCharset(Charsets.UTF_8),
                it
            )
        )
    }
}
```

# **StatusFile**

`status` 구성이 커스텀 액션을 응답 객체에 제공하는 반면, 더 일반적인 솔루션은 방문자가 에러를 보거나 권한 부여 실패 시 표시되는 에러 HTML 제공하는 것이다. `statusFile` 구성은 이러한 유형의
기능을 제공한다.

```kotlin
install(StatusPages) {
    statusFile(HttpStatusCode.NotFound, HttpStatusCode.Unauthorized, filePattern = "error#.html")
}
```

이렇게 하면 classpath에서 2개의 리소스가 해결된다.

1. 404는 error404.html을 반환
2. 401은 error401.html을 반환

`statusFile` 구성은 `#` 문자를 구성된 status들에 목록 내 status code 값으로 대체한다.

# **Redirections using StatusPages**

`call.respondRedirect("/moved/here", permanent = true)`를 실행하여 리다이렉션을 수행할 때, 나머지 callee 함수가 실행된다. 따라서 리다이렉션을 수행할 때 함수를
return해야 한다.

```kotlin
routing {
    get("/") {
        if (condition) {
            return@get call.respondRedirect("/invalid", permanent = false)
        }
        call.respondText("Normal response")
    }
}
```

리다이렉션 시 예외를 사용하므로 정상적인 흐름이 중단되므로 하위 기능이 반환되는 것을 걱정할 필요 없이 리다이렉션을 실행할 수 있다.`StatusPage` 플러그인을 사용해 이를 구현할 수 있다.

```kotlin
fun Application.module() {
    install(StatusPages) {
        exception<HttpRedirectException> { e ->
            call.respondRedirect(e.location, permanent = e.permanent)
        }
    }
    routing {
        get("/") {
            if (condition) {
                redirect("/invalid", permanent = false)
            }
            call.respondText("Normal response")
        }
    }
}

class HttpRedirectException(val location: String, val permanent: Boolean = false) : RuntimeException()

fun redirect(location: String, permanent: Boolean = false): Nothing = throw HttpRedirectException(location, permanent)
```

## References

* [Status pages | Ktor](https://ktor.io/docs/status-pages.html)