# Status pages

`StatusPages` 플러그인은 Ktor 애플리케이션이 적절한 실패 상태를 응답하도록 허용한다. 플러그인은 다음과 같이 설치할 수 있다.

## Add dependencies

`StatusPage`를 사용하기 위해 `ktor-server-status-pages` 아티팩트를 추가해야 한다.

```kotlin
implementation("io.ktor:ktor-server-status-pages:$ktor_version")
```

## Install StatusPages

`StatusPages` 플러그인을 설치하기 위해, 이를 `install` 함수에 전달한다. 서버를 생성하는 방법에 따라 두 가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.statuspages.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(StatusPages)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.statuspages.*

// ...
fun Application.module() {
    install(StatusPages)
    // ...
}
```

## Configure StatusPages

3개의 주 설정 옵션이 있다.

1. `exceptions` : 매핑된 익셉션 클래스를 기반으로 응답 구성
2. `status` : status code 값에 대한 응답 구성
3. `statusFile` : classpath의 표준 파일 응답 구성

### Exceptions

`exception` 핸들러는 `Throwable` 예외를 발생시키는 호출을 처리할 수 있다. 대부분의 경우, 모든 예외에 대해 `500` HTTP 상태 코드를 구성할 수 있다.

```kotlin
install(StatusPages) {
    exception<Throwable> { call, cause ->
        call.respondText(text = "500: $cause", status = HttpStatusCode.InternalServerError)
    }
}
```

더 구체적인 응답은 더 복잡한 사용자 상호 작용을 허용한다.

```kotlin
install(StatusPages) {
    exception<Throwable> { call, cause ->
        if (cause is AuthorizationException) {
            call.respondText(text = "403: $cause", status = HttpStatusCode.Forbidden)
        } else {
            call.respondText(text = "500: $cause", status = HttpStatusCode.InternalServerError)
        }
    }
}
```

### Status

`status` 핸들러는 특정 상태에 기반한 지정된 콘텐츠를 응답하는 기능을 제공한다. 다음 예제에선 리소스가 없는 경우 응답 방법을 보여준다. (`404` 상태 코드)

```kotlin
install(StatusPages) {
    status(HttpStatusCode.NotFound) { call, status ->
        call.respondText(text = "404: Page Not Found", status = status)
    }
}
```

### **StatusFile**

`statusFile` 핸들러는 상태 코드에 기반한 HTML 페이지를 제공할 수 있도록 한다. `error401.html`과 `error402.html` 페이지가 `resources` 폴더에 있다고 가정하자. 이
경우 `statusFile`을 사용해 `401`과 `402` 상태 코드에 대해 처리할 수 있다.

```kotlin
install(StatusPages) {
    statusFile(HttpStatusCode.Unauthorized, HttpStatusCode.PaymentRequired, filePattern = "error#.html")
}
```

`statusFile` 핸들러는 모든 `#` 문자를 구성된 상태 목록 내 상태 코드 값으로 변경한다.

## References

* [Status pages | Ktor](https://ktor.io/docs/status-pages.html)