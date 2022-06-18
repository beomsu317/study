# DoubleReceive

`DoubleReceive` 플러그인은 `RequestAlreadyConsumedException`
예외없이 [receive a request body](https://ktor.io/docs/requests.html#body_contents)를 여러번 호출하는 기능을 제공한다. 이는 일반적으로 
플러그인이 요청 바디를 사용하여 route 핸들러 내에서 수신할 수 없을 때 유용하다. 예를 들어, `DoubleReceive`를 사용해 [CallLogging](https://ktor.io/docs/call-logging.html) 플러그인을 이용해 요청 바디를 로깅하고 후에 `post` route handler에서 한 번 더 바디를 수신할 수 있다.

## Add dependencies

`DoubleReceive`를 사용하기 위해 `ktor-server-double-receive` 아티팩트를 포함해야 한다.

```kotlin
implementation("io.ktor:ktor-server-double-receive:$ktor_version")
```

## Install DoubleReceive

`DoubleReceive` 플러그인 설치를 위해 `install` 함수에 이를 전달해야 한다. 서버를 설정하는 방식에 따라 두 가지 방법으로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.doublereceive.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(DoubleReceive)
        // ...
    }.start(wait = true)
}
```

또는 

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.doublereceive.*
// ...
fun Application.module() {
    install(DoubleReceive)
    // ...
}
```

이후 [receive a request body](https://ktor.io/docs/requests.html#body_contents) 여러 번 수행할 수 있으며, 모든 호출은 동일한 인스턴스를 반환한다. 예를 들어, [CallLogging](https://ktor.io/docs/call-logging.html)을 사용해 요청 바디에 로깅을 활성화할 수 있다.

```kotlin
install(CallLogging) {
    level = Level.TRACE
    format { call ->
        runBlocking {
            "Body: ${call.receiveText()}"
        }
    }
}
```

그리고 route handler에서 한 번 더 요청 바디를 받을 수 있다.

```kotlin
post("/") {
  val receivedText = call.receiveText()
  call.respondText("Text '$receivedText' is received")
}
```

## Configure DoubleReceive

기본 설정은 `DoubleReceive`가 다음 타입의 요청 바디를 수신할 수 있도록 제공한다.

* ByteArray
* String
* Parameters
* data classes used by the ContentNegotiation plugin

기본적으로 `DoubleReceive`는 다음을 제공하지 않는다.

* 동일한 요청에서 다른 타입 수신
* stream 또는 channel 수신

이러한 제한을 극복하기 위해 `cacheRawRequest` 속성을 `false`로 설정한다.

```kotlin
install(DoubleReceive) {
    cacheRawRequest = false
}
```

## References

* [DoubleReceive | Ktor](https://ktor.io/docs/double-receive.html)