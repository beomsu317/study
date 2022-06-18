# WebSocket Deflate extension

Ktor는 클라이언트 및 서버에 대해 `Deflate` 웹소켓 extension RFC-7692를 구현한다. extension은 보내기 전 프레임을 압축하고 수신 후 압축을 해제할 수 있다. 대용량의 텍스트 데이터를
전송하는 경우 이 extension을 유용하게 사용할 수 있다.

API는 프로덕션 준비가 되었지만, 마이너 릴리즈에서 약간 수정될 수 있다. 때문에 `@ExperimentalWebSocketExtensionsApi` 어노테이션으로 마크되어 있다. 만약 이 API를
사용하려면 `OptIn`이 필요하다.

```kotlin
@OptIn(ExperimentalWebSocketExtensionsApi::class)
```

## **Installation**

extension을 사용하기 위해 우선 설치해야 한다. `install` 메서드를 `extensions` 블록에서 사용한다.

```kotlin
// For client and server
install(WebSockets) {
    extensions {
        install(WebSocketDeflateExtension) {
            /**
             * Compression level to use for [java.util.zip.Deflater].
             */
            compressionLevel = Deflater.DEFAULT_COMPRESSION

            /**
             * Prevent to compress small outgoing frames.
             */
            compressIfBiggerThan(bytes = 4 * 1024)
        }
    }
}
```

### **Advanced configuration parameters**

#### **Context takeover**

클라이언트(와 서버)가 compression window를 사용해야 하는지 여부를 지정한다. 이 파라미터를 활성화하려면 단일 세션 당 할당되는 공간이 줄어든다. `java.util.zip.Deflater` API의
제한으로 window size를 구성할 수 없다. 값은 `15`로 고정되어 있다.

```kotlin
clientNoContextTakeOver = false

serverNoContextTakeOver = false
```

이 파라미터들은 [RFC-7692 Section 7.1.1](https://tools.ietf.org/html/rfc7692#section-7.1.1)에 설명되어 있다.

#### **Specify compress condition**

압축 조건을 명시적으로 지정하기 위해 `compressIf` 메서드를 사용할 수 있다. 다음은 텍스트인 경우에만 압축하도록 하는 예제이다.

```kotlin
compressIf { frame ->
    frame is Frame.Text
}
```

`compressIf`에 대한 모든 호출은 압축을 수행하기 전 평가된다.

#### **Fine-tune list of protocols**

전송된 프로토콜 목록은 `configureProtocols` 메서드를 사용해 필요에 따라 편집할 수 있다.

```kotlin
configureProtocols { protocols ->
    protocols.clear()
    protocols.add(
        //...
    )
}
```

## References

* [WebSocket Deflate extension | Ktor](https://ktor.io/docs/websocket-deflate-extension.html)