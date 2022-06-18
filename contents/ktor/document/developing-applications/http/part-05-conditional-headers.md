# Conditional headers

[ConditionalHeaders](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-conditional-headers/index.html)
은 마지막 요청 이후 변경되지 않은 경우 콘텐츠 바디의 전송을 방지하는 플러그인이다. 이는 다음 헤더를 통해 적용된다.

- `Last-Modified` 응답 헤더는 리소스 변경 시간을 포함하고 있다. 예를 들어, 클라이언트 요청에 `If-Modified-Since` 값을 포함하고 있으면, Ktor는 지정된 날짜 이후에 리소스가 수정된
  경우에만 전체 응답을 보낸다. static 파일의 경우 Ktor는 `ConditionalHeaders`를 설치한 후 `Last-Modified` 헤더를 자동으로 추가한다.
- `Etag` 응답 헤더는 특정 리소스 버전의 식별자이다. 예를 들어, 클라이언트 요청에 `If-None-Match` 값이 포함되어 있다면, 이 값이 `Etag`와 일치하는 경우 Ktor는 전체 응답을 보내지
  않는다. `ConditionalHeaders`를 설정할 때 `Etag` 값을 지정할 수 있다.

## **Install ConditionalHeaders**

`ConditionalHeaders` 플러그인 설치를 위해 `install` 함수에 이 플러그인을 전달한다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(ConditionalHeaders)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(ConditionalHeaders)
    // ...
}
```

## **Configure headers**

`ConditionalHeaders`를 구성하기 위해, `install`
블록에서 [version](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-conditional-headers/-configuration/version.html)
함수를 호출해야 한다. 이 함수는
주어진 [OutgoingContent](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http.content/-outgoing-content/index.html)에 대한 리소스
버전의 리스트에 대한 접근을
제공한다. [EntityTagVersion](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http.content/-entity-tag-version/index.html)
과 [LastModifiedVersion](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http.content/-last-modified-version/index.html)
클래스 객체를 통해 필요한 버전을 지정할 수 있다.

다음 코드는 CSS에 대한 `Etag`, `Last-Modified` 헤더를 추가하는 방법을 보여준다.

```kotlin
install(ConditionalHeaders) {
    val file = File("src/main/kotlin/com/example/Application.kt")
    version { outgoingContent ->
        when (outgoingContent.contentType?.withoutParameters()) {
            ContentType.Text.CSS -> listOf(
                EntityTagVersion(file.lastModified().hashCode().toString()),
                LastModifiedVersion(Date(file.lastModified()))
            )
            else -> emptyList()
        }
    }
}
```

## References

* [Conditional headers | Ktor](https://ktor.io/docs/conditional-headers.html)