# Partial content

이 플러그인은 HTTP 메시지의 일부만 클라이언트로 다시 보내는데
사용되는 [HTTP range requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests) 처리에 대한 지원을 추가한다. Partial
Content는 콘텐츠를 스트리밍하거나 다운로드 관리자를 사용해 부분 다운로드를 재개하거나 신뢰할 수 없는 네트워크에 적합하다.

`PartialContent`는 다음 제한을 가진다.

* `HEAD`와 `GET` 요청에 대해서만 동작하며 클라이언트가 다른 방법과 함께 `Range` 헤더를 사용하려 하면 `405 Method Not Allowed`를 반환한다.
* `Content-Length` 헤더가 정의된 응답에만 동작한다.
* ranges를 제공할 때 [Compression](https://ktor.io/docs/compression.html)을 비활성화한다.

## Add dependencies

`PartialContent`를 사용하기 위해 `ktor-server-partial-content` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-server-partial-content:$ktor_version")
```

## Install PartialContent

`PartialContent`를 사용하기 위해, 이를 `install` 함수에 전달해야 한다. 서버를 생성하는 방법에 따라 두 가지 방식으로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.partialcontent.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(PartialContent)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.partialcontent.*

// ...
fun Application.module() {
    install(PartialContent)
    // ...
}
```

## References

* [Partial content | Ktor](https://ktor.io/docs/partial-content.html)