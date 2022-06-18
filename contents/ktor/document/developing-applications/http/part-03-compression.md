# Compression

Ktor는 [Compression](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-compression/index.html) 플러그인을 통해 나가는 콘텐츠들에 대한 압축 기능을 제공한다. `gzip` 및 `deflate`를 포함한 다양한 압축 알고리즘을 사용할 수 있다. 데이터 압축에 필요한 조건(예: 콘텐츠 타입 또는 응답 크기)을 지정하거나 특정 요청 파라미터를 기반으로 압축할 수도 있다.

# **Install Compression**

`Compression` 플러그인을 설치하기 위해 `install` 함수에 이 플러그인을 전달한다.

```kotlin
import io.ktor.features.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Compression)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*
// ...
fun Application.module() {
    install(Compression)
    // ...
}
```

# **Configure compression settings**

압축 설정을 여러 방법으로 할 수 있다. 특정 인코더만 활성화하고, 우선 순위를 지정하고, 특정 콘텐츠 타입만 압축하는 등의 작업을 수행한다.

## **Add specific encoders**

특정 인코더만 활성화하려면, 해당하는 확장 함수를 호출하면 된다.

```kotlin
install(Compression) {
    gzip()
    deflate()
}
```

`priority` 속성을 통해 압축 알고리즘 우선순위를 지정할 수 있다.

```kotlin
install(Compression) {
    gzip {
        priority = 0.9
    }
    deflate {
        priority = 1.0
    }
}
```

위 예제에서 `deflate`가 더 높은 우선순위를 가지고 있다. 서버는 먼저 [Accept-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding) 헤더 내 [quality](https://developer.mozilla.org/en-US/docs/Glossary/Quality_Values)를 확인한 다음 지정된 우선 순위를 고려한다.

## **Configure content type**

기본적으로 Ktor는 `audio`, `video`, `image`, `text/event-stream`과 같은 특정 콘텐츠 타입을 압축하지 않는다. [matchContentType](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/match-content-type.html)을 호출해 압축할 콘텐츠 타입을 선택하거나 [excludeContentType](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/exclude-content-type.html)을 사용해 압축에서 원하는 미디어 타입을 제외할 수 있다. 다음 코드에선 모든 텍스트 하위 타입 및 Javascript 코드를 압축하는 방법을 보여준다.

```kotlin
install(Compression) {
    gzip {
        matchContentType(
            ContentType.Text.Any,
            ContentType.Application.JavaScript
        )
    }
}
```

## **Configure response size**

`Compression` 플러그인을 사용하면 크기가 지정된 값을 초과하지 않는 응답에 대해 압축을 비활성화 할 수 있다. 이를 위해 원하는 값을 [minimumSize](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/minimum-size.html) 함수에 전달한다.

```kotlin
install(Compression) {
    deflate {
        minimumSize(1024)
    }
}
```

## **Specify custom conditions**

필요에 따라 [condition](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/condition.html) 함수를 사용해 커스텀 조건을 제공하고, 특정 요청 파라미터에 따라 데이터를 압축할 수 있다. 다음 코드는 특정 URI에 대한 요청을 압축하는 방법을 보여준다.

```kotlin
install(Compression) {
    gzip {
        condition {
            request.uri == "/orders"
        }
    }
}
```

## **HTTPS security**

압축이 활성화된 HTTPS는 [BREACH](https://en.wikipedia.org/wiki/BREACH) 공격에 취약하다. 다양한 방법을 사용해 이 공격을 방지할 수 있다. 예를 들어, referrer 헤더가 cross-site 요청인 경우 압축을 비활성화 할 수 있다. Ktor에서  referrer 헤더를 체크하여 방지할 수 있다.

```kotlin
install(Compression) {
    gzip {
        condition {
            request.headers[HttpHeaders.Referrer]?.startsWith("https://my.domain/") == true
        }
    }
}
```

## **Implement custom encoder**

필요에 따라 [CompressionEncoder](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-compression-encoder/index.html) 인터페이스를 구현함으로써 자체 인코더를 제공할 수 있다.

## References

* [Compression | Ktor](https://ktor.io/docs/compression.html)