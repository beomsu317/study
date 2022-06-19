# ForwardedHeaderSupport

`ForwardedHeaderSupport`와 `XForwardedHeaders`는 reverse proxy 뒤에 있을 때 원본 요청에 대한 정보를 얻기 위해 reverse proxy 헤더를 처리할 수 있도록 도와주는 플러그인이다.

- `ForwardedHeaderSupport`는 `Forwarded` 헤더를 처리한다.
- `XForwardedHeaderSupport`는 다음의 `X-Forwarded-` 헤더를 처리한다.
  - `X-Forwarded-Host`/`X-Forwarded-Server`
  - `X-Forwarded-For`
  - `X-Forwarded-By`
  - `X-Forwarded-Proto`/`X-Forwarded-Protocol`
  - `X-Forwarded-SSL`/`Front-End-Https`

> 요청을 처리하는 이러한 헤더를 지원하는 reverse proxy가 있는 경우에만 이 플러그인을 설치해라. 다른 경우 클라이언트가 이런 헤더를 조작할 수 있다.


## Add dependencies

`ForwardedHeaders`/`XForwardedHeaders` 플러그인을 사용하기 위해 `ktor-server-forwarded-header` 아티팩트를 추가해야 한다.

```kotlin
implementation("io.ktor:ktor-server-forwarded-header:$ktor_version")
```

## Install plugins

`ForwardedHeaders`/`XForwardedHeaders` 플러그인을 설치하기 위해, 이를 `install` 함수에 전달한다. 서버를 생성하는 방법에 따라 두 가지로 나뉜다. 

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.forwardedheaders.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(ForwardedHeaders)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.forwardedheaders.*
// ...
fun Application.module() {
    install(ForwardedHeaders)
    // ...
}
```

`ForwardedHeaders`와 `XForwardedHeaders`는 다른 설정이 필요하지 않다.

## Get request information

### The proxy request information

proxy 요청에 해당하는 정보를 얻으려면 route handler 내 `call.request.local` 속성을 사용한다. 다음 코드는 host와 port에 대한 정보를 어떻게 얻는지 보여준다.

```kotlin
get("/hello") {
  val localHost = call.request.local.remoteHost
  val localPort = call.request.local.port
}
```

### The original request information

`call.request.origin` 속성을 사용해 original 요청에 대한 정보를 얻을 수 있다.

```kotlin
get("/hello") {
  val remoteHost = call.request.origin.remoteHost
  val remotePort = call.request.origin.port
}
```

## References

* [ForwardedHeaderSupport | Ktor](https://ktor.io/docs/forward-headers.html)