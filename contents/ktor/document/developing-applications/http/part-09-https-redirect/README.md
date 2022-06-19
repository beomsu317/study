# HttpsRedirect

`HttpsRedirect`은 영향을 받는 모든 HTTP 호출이 호출을 처리하기 전 해당 HTTPS에 대한 리다이렉션 수행하도록 하는 플러그인이다.

기본적으로 `301 Moved Permanently`으로 리다이렉션이지만, `302 Found` 리다이렉션으로 구성할 수 있다.

## Add dependencies

`HttpsRedirect`를 사용하기 위해 `ktor-server-http-redirect` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-server-http-redirect:$ktor_version")
```

## Install HttpsRedirect

`HttpsRedirect` 플러그인을 설치하려면, 이를 `install` 함수로 전달해야 한다. 서버를 생성하는 방법에 따라 2가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.httpsredirect.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(HttpsRedirect)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.httpsredirect.*
// ...
fun Application.module() {
    install(HttpsRedirect)
    // ...
}

```

위 코드는 HttpsRedirect 플러그인을 기본 구성으로 설치하는 것을 보여준다.

> reverse proxy 뒤에 있는 경우 `ForwardedHeaderSupport` 또는 `XForwardedHeaderSupport` 플러그인을 설치해야 `HttpsRedirect` 플러그인이 적절하게 HTTPS 요청을 처리할 수 있다.

## Configure HttpsRedirect

다음 코드는 HTTPS 포트를 구성하고 요청된 리소스에 대해 `301 Moved Permanently`를 반환하는 방법을 보여준다.

```kotlin
install(HttpsRedirect) {
    sslPort = 8443
    permanentRedirect = true
}
```

## References

* [HttpsRedirect | Ktor](https://ktor.io/docs/https-redirect.html)