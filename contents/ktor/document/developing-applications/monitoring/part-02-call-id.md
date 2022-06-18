# CallId

[CallId](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-call-id/io.ktor.server.plugins.callid/-call-id.html?_ga=2.92034522.1396641199.1655526702-658241611.1655526702&_gl=1*yzs6lj*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjEuMTY1NTU0MzI0My4w)
플러그인은 유니크한 request ID 또는 call ID를 사용해 클라이언트 요청을 추적할 수 있다. 일반적으로 Ktor에서 call ID로 작업하는 것은 다음과 같다.

1. 다음 방법을 통해 특정 요청에 대한 call ID를 얻어온다.
    * Reverse proxy 또는 cloud provider는 `X-Request-Id`와 같은 특정 헤더에 call ID를 추가할 수 있다. 이 경우 Ktor는 call ID를 검색할 수 있다.
    * 반면에 요청이 call ID 없이 오면 Ktor 서버에서 생성할 수 있다.
2. Ktor는 사전에 정의된 딕셔너리를 사용해 받아오거나 생성된 call ID에 대해 검증한다. 또한 자체적인 조건을 제공하여 call ID를 검증할 수 있다.
3. 마지막으로 `X-Request-Id`와 같은 특정 헤더에서 클라이언트에게 call ID를 전송할 수 있다.

`CallLogging`과 함께 `CallId`를 사용하면 MDC 컨텍스트에 call ID를 입력하고 각 요청에 대한 call ID를 표시하도록 로거를 구성하여 문제를 해결할 수 있다.

## Add dependencies

`CallId`를 사용하기 위해 `ktor-server-call-id` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-call-id:$ktor_version")
```

## Install CallId

`CallId` 플러그인 설치를 위해, 이를 `install` 함수에 전달한다. 서버를 생성하는 방법에 따라 2가지 방법으로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.callid.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(CallId)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.callid.*

// ...
fun Application.module() {
    install(CallId)
    // ...
}
```

## Configure CallId

### Retrieve a call ID

`CallId`는 call ID를 얻기 위한 여러 방법을 제공한다.

* 특정 헤더에서 call ID를 얻어오기 위해, `retrieveFromHeader` 함수를 사용한다.

```kotlin
install(CallId) {
    retrieveFromHeader(HttpHeaders.XRequestId)
}
```

`header` 함수를 이용해서도 받아올 수 있고, 동일한 헤더에 call ID를 전송할 수 있다.

* 필요한 경우  `ApplicationCall`에서 call ID를 받아올 수 있다.

```kotlin
install(CallId) {
    retrieve { call ->
        call.request.header(HttpHeaders.XRequestId)
    }
}
```

모든 call ID는 기본적인 딕셔너리를 사용해 검증한다.

### Generate a call ID

들어오는 요청에 call ID가 없다면, `generate` 함수를 사용해 생성할 수 있다.

* 다음과 같이 사전 정의된 딕셔너리에서 특정 길이의 call ID를 생성하는 방법을 보여준다.

```kotlin
install(CallId) {
    generate(10, "abcde12345")
}
```

* 다음 예제에서 `generate` 함수는 call ID를 생성하기 위한 블록을 허용한다.

```kotlin
install(CallId) {
    val counter = atomic(0)
    generate {
        "generated-call-id-${counter.getAndIncrement()}"
    }
}
```

### Verify a call ID

얻어오거나 생성된 모든 call ID는 다음과 같이 기본 딕셔너리를 사용해 검증된다.

```
CALL_ID_DEFAULT_DICTIONARY: String = "abcdefghijklmnopqrstuvwxyz0123456789+/=-"
```

이는 대문자가 포함된 call ID는 검증이 실패한다는 의미이다. 필요한 경우 `verify` 함수를 사용해 덜 엄격한 규칙을 적용할 수 있다.

```kotlin
install(CallId) {
    verify { callId: String ->
        callId.isNotEmpty()
    }
}
```

### Send a call ID to the client

Call ID를 얻어오거나 생성한 후 클라이언트에 전송할 수 있다.

* `header` 함수를 통해 call ID를 얻어와 동일한 헤더에 전송할 수 있다.

```kotlin
install(CallId) {
    header(HttpHeaders.XRequestId)
}
```

* `replyToHeader` 함수는 지정된 헤더에서 call ID를 전송한다.

```kotlin
install(CallId) {
    replyToHeader(HttpHeaders.XRequestId)
}
```

* 필요하다면 `ApplicationCall`을 사용해 응답으로 call ID를 전송할 수 있다.

```kotlin
reply { call, callId ->
    call.response.header(HttpHeaders.XRequestId, callId)
}
```

## Put a call ID into MDC

`CallId`와 `CallLogging`을 함께 사용하면 MDC 컨텍스트에 call ID를 입력하고 각 요청에 대한 call ID를 표시하도록 로거를 구성해 이슈를 해결할 수 있다. 이를
위해 `CallLogging` 설정 블록에서 `callIdMdc` 함수를 호출하고, MDC 컨텍스트에 넣을 원하는 키를 지정한다.

```kotlin
install(CallLogging) {
    callIdMdc("call-id")
}
```

이 키는 call ID를 로그에 보여주기 위해 [logger configuration](https://ktor.io/docs/logging.html#configure-logger)으로 전달된다. 예를
들어 `logback.xml` 파일은 다음과 같을 수 있다.

```xml

<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %X{call-id} %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

## References

* [CallId | Ktor](https://ktor.io/docs/call-id.html)