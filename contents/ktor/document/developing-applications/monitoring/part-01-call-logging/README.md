# Call logging

Ktor는 애플리케이션 이벤트를 [SLF4J](http://www.slf4j.org/) 라이브러리를 이용해 로깅하는 기능을 제공한다.

`CallLogging` 플러그인은 incoming 클라이언트 요청에 대한 로깅을 수행한다.

## Add dependencies

`CallLogging`을 사용하기 위해 `ktor-server-call-logging` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-call-logging:$ktor_version")
```

## **Install CallLogging**

`CallLogging` 플러그인 설치를 위해 `install` 함수에 이 플러그인을 파라미터로 전달한다.

```kotlin
import io.ktor.features.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(CallLogging)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*
// ...
fun Application.module() {
    install(CallLogging)
    // ...
}
```

## **Configure logging settings**

`CallLogging` 구성을 여러 방법으로 수행할 수 있다. 로깅 레벨 지정, 특정 조건에 기반한 요청 필터, 커스터마이즈 로그 메시지 등. [CallLogging.Configuration](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-call-logging/-configuration/index.html)에서 설정 구성에 대해 확인할 수 있다.

### **Set the logging level**

기본적으로 Ktor는 `Level.TRACE` 로깅 레벨을 사용한다. 변경하기 위해 `level` 속성을 사용한다.

```kotlin
install(CallLogging) {
    level = Level.INFO
}
```

### **Filter log requests**

`filter` 속성을 사용해 요청 필터링 조건을 추가할 수 있다. 다음은 `/api/v1` 로그에 대해서만 로깅한다.

```kotlin
install(CallLogging) {
    filter { call ->
        call.request.path().startsWith("/api/v1")
    }
}
```

### **Customize a log message format**

`format` 함수를 통해 요청/응답과 관련된 데이터를 로깅할 수 있다. 다음 예제는 각 요청에 대한 응답 상태, HTTP 요청 메서드, `User-Agent` 헤더 값을 로깅한다.

```kotlin
install(CallLogging) {
    format { call ->
        val status = call.response.status()
        val httpMethod = call.request.httpMethod.value
        val userAgent = call.request.headers["User-Agent"]
        "Status: $status, HTTP method: $httpMethod, User agent: $userAgent"
    }
}
```

### **Put call parameters in MDC**

`CallLogging` 플러그인은 MDC(Mapped Diagnostic Context)를 지원한다. `mdc` 함수를 사용해 MDC에 지정된 이름의 원하는 컨텍스트 값을 넣을 수 있다. 다음 예제에서 `name` 쿼리 파라미터는 MDC에 추가된다.

```kotlin
install(CallLogging) {
    mdc("name-parameter") { call ->
        call.request.queryParameters["name"]
    }
}
```

`ApplicationCall` 수명 동안 추가된 값에 접근할 수 있다.

```kotlin
import org.slf4j.MDC
// ...
MDC.get("name-parameter")
```

## References

* [Call logging | Ktor](https://ktor.io/docs/call-logging.html)