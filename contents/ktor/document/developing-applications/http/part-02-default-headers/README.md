# Default headers

`DefaultHeaders` 플러그인은 표준 `Server`와 `Date` 헤더를 응답에 추가한다. `Server` 헤더를 오버라이드하여 추가적인 기본 헤더를 제공할 수도 있다.

## Add dependencies

`DefaultHeaders`를 사용하기 위해 `ktor-server-default-headers` 아티팩트가 필요하다.

```kotlin
ktor-server-default-headers
```

## **Install DefaultHeaders**

`DefaultHeaders` 플러그인을 설치하려면, 초기화 코드의 `install` 함수에 전달해야 한다. 서버의 생성 방법에 따라 다르게 추가된다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.defaultheaders.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(DefaultHeaders)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.defaultheaders.*
// ...
fun Application.module() {
    install(DefaultHeaders)
    // ...
}

```

`DefaultHeaders` 플러그인은 `Server`와 `Date` 헤더를 응답에 추가한다. 필요한 경우 `Server`를 오버라이드할 수 있다.

## **Add additional headers**

default headers의 목록을 커스터마이징하기 위해 `header(name, value)` 함수를 사용해 `install` 할 헤더를 전달한다. `name` 파라미터는 `HttpHeaders` 값을 받는다.

```kotlin
install(DefaultHeaders) {
    header(HttpHeaders.ETag, "7c876b7e")
}
```

커스텀 헤더를 추가하려면 이름을 문자열로 전달한다.

```kotlin
install(DefaultHeaders) {
    header("Custom-Header", "Some value")
}
```

## **Override headers**

`Server`를 오버라이드하려면, 해당하는 `HttpHeaders` 값을 사용한다.

```kotlin
install(DefaultHeaders) {
    header(HttpHeaders.Server, "Custom")
}
```

`Date` 헤더는 성능상의 이유로 캐싱되고 `DefaultHeaders`를 사용하는 것으로 오버라이딩되지 않는다.

## References

* [Default headers | Ktor](https://ktor.io/docs/default-headers.html)
