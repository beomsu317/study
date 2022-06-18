# Caching headers

[CachingHeaders](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-caching-headers/index.html)
플러그인은 HTTP 캐싱에 사용되는 `Cache-Control`, `Expires` 헤더를 구성하는 기능을 추가할 수 있다. 이미지, CSS, JavaScript 파일 등과 같은 특정 콘텐츠 타입에 대해 다양한 캐싱
전략을 도입할 수 있다.

## **Install CachingHeaders**

`CachingHeaders` 플러그인을 설치하기 위해 `install` 함수에 이 플러그인을 전달한다. 서버의 생성 방법에 따라 다르게 구현된다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(CachingHeaders)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(CachingHeaders)
    // ...
}
```

`CachingHeaders` 설치 후 다양한 콘텐츠 타입에 대한 캐싱 설정을 구성할 수 있다.

## **Configure caching**

`CachingHeaders` 플러그인을 구성하기 위해 주어진 콘텐츠 타입에 대해 지정된 캐싱 옵션을
제공하려면 [options](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-caching-headers/-configuration/options.html)
함수를 정의해야
한다. [caching-headers](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/caching-headers) 예제는
CSS에 대한 `max-age` 옵션을 사용하여 `Cache-Control` 헤더를 추가하는 방법을 보여준다.

```kotlin
install(CachingHeaders) {
    options { outgoingContent ->
        when (outgoingContent.contentType?.withoutParameters()) {
            ContentType.Text.CSS -> CachingOptions(CacheControl.MaxAge(maxAgeSeconds = 3600))
            else -> null
        }
    }
}
```

[CachingOptions](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http.content/-caching-options/index.html)
객체는 `Cache-Control`와 `Expires` 헤더 값을 파라미터로 받는다.

- `cacheControl` 파라미터는 [CacheControl](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http/-cache-control/index.html) 값을
  받는다. [CacheControl.MaxAge](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http/-cache-control/-max-age/index.html)를
  사용해 visibility, revalidation 옵션 등과 같은 `max-age` 파라미터 및 관련 설정을 지정할 수 있다. `CacheControl.NoCache`/`CacheControl.NoStore`를
  사용해 캐싱을 비활성화 할 수 있다.
- `expires` 파라미터를 사용해 `Expires` 헤더를 `GMTDate` 또는 `ZonedDateTime` 값으로 지정할 수 있다.

## **Customize headers for specific routes**

특정 route에 대해 캐싱 헤더를 지정하고 싶다면, 원하는 헤더를 응답에 추가하면 된다. `CachingHeaders`를 설치할 필요가 없다. 다음은 `/profile` route에 캐싱을 비활성화하는 방법을
보여준다.

```kotlin
get("/profile") {
    call.response.headers.append(HttpHeaders.CacheControl, "no-cache, no-store")
    // ...
}
```

## References

* [Caching headers | Ktor](https://ktor.io/docs/caching.html)