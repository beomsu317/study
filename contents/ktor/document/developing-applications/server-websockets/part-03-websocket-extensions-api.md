# WebSocket extensions API

Ktor 웹소켓 API는 자체 extension 또는 커스텀 extension 작성을 지원한다.

API는 프로덕션 준비가 되었지만, 마이너 릴리즈에서 약간 수정될 수 있다. 때문에 `@ExperimentalWebSocketExtensionsApi` 어노테이션으로 마크되어 있다. 만약 이 API를
사용하려면 `OptIn`이 필요하다.

```kotlin
@OptIn(ExperimentalWebSocketExtensionsApi::class)
```

## **Installing extension**

extension을 설치하고 구성하기 위해 2가지 메서드를 제공한다. `extensions`와 `install`.

```kotlin
install(WebSockets) {
    extensions { /* WebSocketExtensionConfig.() -> Unit */
        install(MyWebSocketExtension) { /* MyWebSocketExtensionConfig.() -> Unit */
            /* Optional extension configuration. */
        }
    }
}
```

extension들은 설치 순서대로 적용된다.

## **Checking if the extension is negotiated**

모든 설치된 extension들은 협상 프로세스를 거치며 성공적으로 협상된 extension은 요청 중
사용된다. `WebSocketSession.extensions: : List<WebSocketExtension<*>>` 속성을 현재 세션에 사용되는 모든 extension 리스트와 함께 사용할 수 있다.

extension이 사용 중인지 확인하는 2가지 방법이 있다. `WebSocketSession.extension`
과 `WebSocketSession.extensionOrNull`.

```kotlin
webSocket("/echo") {
    val myExtension = extension(MyWebSocketException) // will throw if `MyWebSocketException` is not negotiated
    // or
    val myExtension = extensionOrNull(MyWebSocketException)
        ?: close() // will close the session if `MyWebSocketException` is not negotiated
}
```

## **Writing a new extension**

새로운 extension을 구현하기 위한 2개의 인터페이스가 있다. `WebSocketExtension<ConfigType: Any>`
와 `WebSocketExtensionFactory<ConfigType : Any, ExtensionType : WebSocketExtension<ConfigType>>`. 단일 구현은 클라이언트와 서버에서 모두
작동할 수 있다.

다음 예제는 단일 프레임 로깅 extension을 구현하는 방법이다.

```kotlin
class FrameLoggerExtension(val logger: Logger) : WebSocketExtension<FrameLogger.Config> {
}
```

이 기능에는 두 그룹의 필드와 메서드가 있다. 첫 번째 그룹은 extension 협상을 위한 것이다.

```kotlin
/** List of protocols will be sent in client request for negotiation **/
override val protocols: List<WebSocketExtensionHeader> = emptyList()

/**
 * This method will be called for server and will process `requestedProtocols` from client.
 * In the result it will return list of extensions that server agrees to use.
 */
override fun serverNegotiation(requestedProtocols: List<WebSocketExtensionHeader>): List<WebSocketExtensionHeader> {
    logger.log("Server negotiation")
    return emptyList()
}

/**
 * This method will be called on the client with list of protocols, produced by `serverNegotiation`. It will decide if these extensions should be used.
 */
override fun clientNegotiation(negotiatedProtocols: List<WebSocketExtensionHeader>): Boolean {
    logger.log("Client negotiation")
    return true
}
```

두 번째 그룹은 실제 프레임 처리를 위한 장소이다. 메서드는 프레임 가져와 필요한 경우 새로 처리된 프레임을 생성한다.

```kotlin
override fun processOutgoingFrame(frame: Frame): Frame {
    logger.log("Process outgoing frame: $frame")
    return frame
}

override fun processIncomingFrame(frame: Frame): Frame {
    logger.log("Process incoming frame: $frame")
    return frame
}
```

또한 몇 개의 세부 구현이 있다. 기능은 `Config`와 원래의 `factory`에 대한 참조를 가지고 있다.

```kotlin
class Config {
    lateinit var logger: Logger
}

/**
 * Factory which can create current extension instance.
 */
override val factory: WebSocketExtensionFactory<Config, FrameLogger> = FrameLoggerExtension
```

factory는 보통 companion object로 구현되어 있다.

```kotlin
companion object : WebSocketExtensionFactory<Config, FrameLogger> {
    /* Key to discover installed extension instance */
    override val key: AttributeKey<FrameLogger> = AttributeKey("frame-logger")

    /** List of occupied rsv bits.
     * If the extension occupy a bit, it can't be used in other installed extensions. We use that bits to prevent feature conflicts(prevent to install multiple compression features). If you're implementing feature using some RFC, rsv occupied bits should be referenced there.
     */
    override val rsv1: Boolean = false
    override val rsv2: Boolean = false
    override val rsv3: Boolean = false

    /** Create feature instance. Will be called for each WebSocket session **/
    override fun install(config: Config.() -> Unit): FrameLogger {
        return FrameLogger(Config().apply(config).logger)
    }
}
```

## References

* [WebSocket extensions API | Ktor](https://ktor.io/docs/websocket-extensions-api.html)