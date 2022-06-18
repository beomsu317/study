# Server WebSockets

Ktor는 웹소켓 프로토콜을 지원하며 서버에서 실시간 데이터 전송이 필요한 애플리케이션을 만들 수 있다. 예를 들어, 채팅 애플리케이션을 만들 수 있다.

- frame size, ping period 등 기본 웹소켓 설정 구성.
- 서버와 클라이언트 간 메시지 교환을 위한 웹소켓 세션 처리.
- 웹소켓 extension 추가. 예를 들어, [Deflate](https://ktor.io/docs/websocket-deflate-extension.html) extension 또는 커스텀 extension을
  사용할 수 있다.

## **Add dependencies**

`WebSockets` 지원을 활성화하기 위해 `ktor-websockets` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-websockets:$ktor_version")
```

## **Install WebSockets**

`WebSockets` 플러그인을 설치하기 위해 `install` 함수에 이 플러그인을 전달한다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(WebSockets)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(WebSockets)
    // ...
}
```

## **Configure WebSockets settings**

선택적으로 `install`
함수에 [WebSocketOptions](https://api.ktor.io/ktor-features/ktor-websockets/ktor-websockets/io.ktor.websocket/-web-sockets/-web-socket-options/index.html)
을 전달해 플러그인을 구성할 수 있다.

```kotlin
install(WebSockets) {
    pingPeriod = Duration.ofSeconds(15)
    timeout = Duration.ofSeconds(15)
    maxFrameSize = Long.MAX_VALUE
    masking = false
}
```

## **Handle WebSockets sessions**

### **API overview**

`WebSockets` 플러그인 설치 및 구성 후 웹소켓 세션을 처리할 준비가 된다.
우선 [webSocket](https://api.ktor.io/ktor-features/ktor-websockets/ktor-websockets/io.ktor.websocket/web-socket.html) 함수를
통해 웹소켓 엔드포인트를 정의한다.

```kotlin
routing {
    webSocket("/echo") {
        // Handle a WebSocket session
    }
}
```

이러한 엔드포인트의 경우, 서버는 기본 구성이 사용될 때 `ws://localhost:8080/echo`에 대한 웹소켓 요청을 받아들인다.

`webSocket`
블록에서 [DefaultWebSocketServerSession](https://api.ktor.io/ktor-features/ktor-websockets/ktor-websockets/io.ktor.websocket/-default-web-socket-server-session/index.html)
클래스로 표시되는 웹소켓 세션 처리가 필요하다. 세션 구성은 다음과 같다.

1. `send` 함수를 사용해 텍스트 컨텐츠를 클라이언트에 전송한다.
2. `incoming`과 `outgoing` 속성을 사용해 웹소켓 프레임을 수신 및 전송하기 위한 채널에 접근한다.
   프레임은 [Frame](https://api.ktor.io/ktor-http/ktor-http-cio/ktor-http-cio/io.ktor.http.cio.websocket/-frame/index.html)
   클래스로 표현된다.
3. 세션을 처리할 때 프레임 타입을 체크한다.
    - `Frame.Text`는 텍스트 프레임이다. 이 타입은 `Frame.Text.readText()`를 사용해 컨텐츠를 읽을 수 있다.
    - `Frame.Binary`는 바이너리 프레임이다. 이 타입은 `Frame.Binary.readBytes()`를 사용해 컨텐츠를 읽을 수 있다.
    - `Frame.Close`는 클로징 프레임이다. `Frame.Close.readReason()`을 호출해 현재 세션에 대한 종료 이유를 얻을 수 있다.
4. `close` 함수를 사용해 특정 이유와 함께 close를 전송한다.

### **Example: Handle a single session**

다음 예제에서 하나의 클라이언트에 대한 세션을 처리하는 `echo` 웹소켓 엔드포인트를 생성하는 방법을 보여준다.

```kotlin
routing {
    webSocket("/echo") {
        send("Please enter your name")
        for (frame in incoming) {
            when (frame) {
                is Frame.Text -> {
                    val receivedText = frame.readText()
                    if (receivedText.equals("bye", ignoreCase = true)) {
                        close(CloseReason(CloseReason.Codes.NORMAL, "Client said BYE"))
                    } else {
                        send(Frame.Text("Hi, $receivedText!"))
                    }
                }
            }
        }
    }
}
```

### **Example: Handle multiple sessions**

여러 웹소켓 세션을 처리하기 위해 각 세션을 서버에 저장해야 한다. 예를 들어, 고유한 이름과 함께 연결을 정의하고 지정된 세션과 연결할 수 있다. 다음 `Connection` 클래스는 어떻게 이를 정의하는지
보여준다.

```kotlin
class Connection(val session: DefaultWebSocketSession) {
    companion object {
        var lastId = AtomicInteger(0)
    }

    val name = "user${lastId.getAndIncrement()}"
}
```

그런 다음, 새로운 클라이언트가 웹소켓 엔드포인트에 연결할 때 `webSocket` 핸들러에서 새로운 연결을 생성한다.

```kotlin
routing {
    val connections = Collections.synchronizedSet<Connection?>(LinkedHashSet())
    webSocket("/chat") {
        val thisConnection = Connection(this)
        connections += thisConnection
        send("You've logged in as [${thisConnection.name}]")

        for (frame in incoming) {
            when (frame) {
                is Frame.Text -> {
                    val receivedText = frame.readText()
                    val textWithUsername = "[${thisConnection.name}]: $receivedText"
                    connections.forEach {
                        it.session.send(textWithUsername)
                    }
                }
            }
        }
    }
}
```

### Testing

`withTestApplication` 블록 내 `handleWebSocketConversation` 함수를 통해 웹소켓 대화를 테스트할 수 있다.

```kotlin
class ModuleTest {
    @Test
    fun testConversation() {
        withTestApplication(Application::module) {
            handleWebSocketConversation("/echo") { incoming, outgoing ->
                val greetingText = (incoming.receive() as Frame.Text).readText()
                assertEquals("Please enter your name", greetingText)

                outgoing.send(Frame.Text("JetBrains"))
                val responseText = (incoming.receive() as Frame.Text).readText()
                assertEquals("Hi, JetBrains!", responseText)

                outgoing.send(Frame.Text("bye"))
                val closeReason = (incoming.receive() as Frame.Close).readReason()?.message
                assertEquals("Client said BYE", closeReason)
            }
        }
    }
}
```

## **The WebSocket API and Ktor**

웹소켓 API의 표준 이벤트는 다음과 같은 방식으로 Ktor에 매핑된다.

- `onConnect`는 블록 시작에서 일어난다.
- `onMessage`는 성공적으로 메시지를 읽었거나(`incoming.receive()`) suspend iteration(`for(frame in incoming)`)을 사용했을 때 일어난다.
- `onClose`는 `incoming` 채널이 닫혔을 때 일어난다. suspend iteration이 완료되거나 메시지 수신을 시도할 때 `ClosedReceiveChannelException`을 throw
  한다.
- `onError`는 다른 예외와 동일하다.

`onClose`, `onError`
모두 [closeReason](https://api.ktor.io/ktor-http/ktor-http-cio/ktor-http-cio/io.ktor.http.cio.websocket/-default-web-socket-session/close-reason.html)
속성이 설정된다.

```kotlin
webSocket("/echo") {
    println("onConnect")
    try {
        for (frame in incoming) {
            val text = (frame as Frame.Text).readText()
            println("onMessage")
            received += text
            outgoing.send(Frame.Text(text))
        }
    } catch (e: ClosedReceiveChannelException) {
        println("onClose ${closeReason.await()}")
    } catch (e: Throwable) {
        println("onError ${closeReason.await()}")
        e.printStackTrace()
    }
}
```

이 샘플의 경우 무한 루프는 `ClosedReceiveChannelException` 또는 다른 예외가 발생했을 때 탈출한다.

## References

* [Server WebSockets | Ktor](https://ktor.io/docs/websocket.html)