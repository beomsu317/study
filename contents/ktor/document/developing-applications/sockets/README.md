# Sockets

Ktor는 서버와 클라이언트에 대한 HTTP/WebSocket 처리 외에도 TCP, UDP와 같은 raw socket을
지원한다. [java.nio](https://docs.oracle.com/javase/8/docs/api/java/nio/package-summary.html)에서 `suspending` API를 제공한다.

## Add dependencies

`Sockets`를 사용하기 위해, `ktor-network` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-network:$ktor_version")
```

클라이언트에서 [secure sockets](https://ktor.io/docs/servers-raw-sockets.html#secure)을 사용하기 위해 `io.ktor:ktor-network-tls`를 추가해야
한다.

## Server

### Create a server socket

서버 소켓을 빌드하기 위해, `SelectorManager` 인스턴스를 생성하고 `SocketBuilder.tcp()` 함수를 호출한다. 그리고 `bind`를 사용해 서버 소켓을 지정된 포트에 바인딩한다.

```kotlin
val selectorManager = SelectorManager(Dispatchers.IO)
val serverSocket = aSocket(selectorManager).tcp().bind("127.0.0.1", 9002)
```

위 예제에선 TCP 소켓을 생성한다. UDP 소켓의 경우 `SocketBuilder.udp()`를 사용한다.

### Accept incoming connections

서버 소켓을 생성한 후 소켓 연결을 수락하고, 연결된 소켓 인스턴스를 반환하는 `ServerSocket.accept` 함수를 호출해야 한다.

```kotlin
val socket = serverSocket.accept()
```

소켓이 연결된 후 소켓에서 데이터를 읽거나 쓰는 방식으로 데이터를 수신/전송 할 수 있다.

### Receive data

클라이언트로부터 데이터를 받기
위해 [ByteReadChannel](https://api.ktor.io/ktor-io/io.ktor.utils.io/-byte-read-channel/index.html?_ga=2.159800573.1396641199.1655526702-658241611.1655526702&_gl=1*151mytj*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjAuMTY1NTU0MTcyMi4w)
을 반환하는 `Socket.openReadChannel` 함수가 필요하다.

```kotlin
val receiveChannel = socket.openReadChannel()
```

`ByteReadChannel`은 데이터를 비동기적으로 읽는 API를 제공한다. 예를 들어, `ByteReadChannel.readUTF8Line`를 사용해 UTF-8 문자들의 라인을 읽을 수 있다.

```kotlin
val name = receiveChannel.readUTF8Line()
```

### Send data

클라이언트에 데이터를 보내기
위해 [ByteWriteChannel](https://api.ktor.io/ktor-io/io.ktor.utils.io/-byte-write-channel/index.html?_ga=2.193468269.1396641199.1655526702-658241611.1655526702&_gl=1*yy8pci*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjAuMTY1NTU0MTcyMi4w)
을 반환하는 `Socket.openWriteChannel` 함수를 호출한다.

```kotlin
val sendChannel = socket.openWriteChannel(autoFlush = true)
```

`ByteWriteChannel`는 비동기적으로 일련의 바이트를 쓰는 API를 제공한다. 예를 들어, UTF-8 문자들의 라인을 쓰는 경우 `ByteWriteChannel.writeStringUtf8`을 사용한다.

```kotlin
val name = receiveChannel.readUTF8Line()
sendChannel.writeStringUtf8("Hello, $name!\n")
```

### Close a socket

연결된 소켓에 대한 리소스를 해제하기 위해 `Socket.close`를 호출한다.

```kotlin
socket.close()
```

### Example

다음은 서버 측에서 소켓을 사용하는 방법을 보여준다.

```kotlin
package com.example

import io.ktor.network.selector.*
import io.ktor.network.sockets.*
import io.ktor.utils.io.*
import kotlinx.coroutines.*

fun main(args: Array<String>) {
    runBlocking {
        val selectorManager = SelectorManager(Dispatchers.IO)
        val serverSocket = aSocket(selectorManager).tcp().bind("127.0.0.1", 9002)
        println("Server is listening at ${serverSocket.localAddress}")
        while (true) {
            val socket = serverSocket.accept()
            println("Accepted $socket")
            launch {
                val receiveChannel = socket.openReadChannel()
                val sendChannel = socket.openWriteChannel(autoFlush = true)
                sendChannel.writeStringUtf8("Please enter your name\n")
                try {
                    while (true) {
                        val name = receiveChannel.readUTF8Line()
                        sendChannel.writeStringUtf8("Hello, $name!\n")
                    }
                } catch (e: Throwable) {
                    socket.close()
                }
            }
        }
    }
}
```

## Client

### Create a socket

클라이언트 소켓을 빌드하기 위해 `SelectorManager` 인스턴스를 생성하 `SocketBuilder.tcp()` 함수를 호출한다. 그 다음 `connect`를 사용해 연결을 수립하고, 연결된 소켓을 얻는다.

```kotlin
val selectorManager = SelectorManager(Dispatchers.IO)
val socket = aSocket(selectorManager).tcp().connect("127.0.0.1", 9002)
```

연결된 소켓을 얻어오면, 해당 소켓을 읽거나 써서 데이터를 수신/전송할 수 있다.

### Create a secure socket (SSL/TLS)

Secure 소켓은 TLS 연결을 수립하도록 허용한다. Secure 소켓을 사용하기 위해 `ktor-network-tls` 디펜던시 추가가 필요하다. 그리고 `Socket.tls` 함수를 연결된 소켓에서 호출한다.

```kotlin
val selectorManager = SelectorManager(Dispatchers.IO)
val socket = aSocket(selectorManager).tcp().connect("127.0.0.1", 8443).tls()
```

`tls` 함수를
사용하면 [TLSConfigBuilder](https://api.ktor.io/ktor-network/ktor-network-tls/io.ktor.network.tls/-t-l-s-config-builder/index.html?_ga=2.96817631.1396641199.1655526702-658241611.1655526702&_gl=1*1q5hzyr*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjAuMTY1NTU0MTcyMi4w)
에서 제공하는 TLS 파라미터를 설정할 수 있다.

```kotlin
val selectorManager = SelectorManager(Dispatchers.IO)
val socket = aSocket(selectorManager).tcp().connect("youtrack.jetbrains.com", port = 443)
    .tls(coroutineContext = coroutineContext) {
        trustManager = object : X509TrustManager {
            override fun getAcceptedIssuers(): Array<X509Certificate?> = arrayOf()
            override fun checkClientTrusted(certs: Array<X509Certificate?>?, authType: String?) {}
            override fun checkServerTrusted(certs: Array<X509Certificate?>?, authType: String?) {}
        }
    }
```

### Receive data

서버로부터 데이터를 받기
위해 [ByteReadChannel](https://api.ktor.io/ktor-io/io.ktor.utils.io/-byte-read-channel/index.html?_ga=2.162542844.1396641199.1655526702-658241611.1655526702&_gl=1*26kcjf*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjAuMTY1NTU0MTcyMi4w)
을 반환하는 `Socket.openReadChannel` 함수를 호출해야 한다.

```kotlin
val receiveChannel = socket.openReadChannel()
```

`ByteReadChannel`은 데이터를 비동기적으로 읽어올 수 있는 API를 제공한다. 예를 들어, UTF-8 문자들에 대한 라인을 `ByteReadChannel.readUTF8Line`로 읽어올 수 있다.

```kotlin
val greeting = receiveChannel.readUTF8Line()
```

### Send data

서버에 데이터를 전송하기
위해 [ByteWriteChannel](https://api.ktor.io/ktor-io/io.ktor.utils.io/-byte-write-channel/index.html?_ga=2.159185914.1396641199.1655526702-658241611.1655526702&_gl=1*14hxn9g*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjAuMTY1NTU0MTcyMi4w)
를 반환하는 `Socket.openWriteChannel` 함수를 호출한다.

```kotlin
val sendChannel = socket.openWriteChannel(autoFlush = true)
```

`ByteWriteChannel`은 일련의 바이트를 비동기적으로 쓸 수 있는 API를 제공한다. 예를 들어, UTF-8 문자들의 라인을 쓰는 경우 `ByteWriteChannel.writeStringUtf8`를
사용한다.

```kotlin
val myMessage = readln()
sendChannel.writeStringUtf8("$myMessage\n")
```

### Close connection

연결된 소켓에 대한 리소스를 해제하고 싶은 경우 `Socket.close` 및 `SelectorManager.close`를 호출한다.

```kotlin
socket.close()
selectorManager.close()
```

### Example

다음은 클라이언트 측에서 소켓이 어떻게 사용되는지 보여준다.

```kotlin
package com.example

import io.ktor.network.selector.*
import io.ktor.network.sockets.*
import io.ktor.utils.io.*
import kotlinx.coroutines.*
import kotlin.system.*

fun main(args: Array<String>) {
    runBlocking {
        val selectorManager = SelectorManager(Dispatchers.IO)
        val socket = aSocket(selectorManager).tcp().connect("127.0.0.1", 9002)

        val receiveChannel = socket.openReadChannel()
        val sendChannel = socket.openWriteChannel(autoFlush = true)

        launch(Dispatchers.IO) {
            while (true) {
                val greeting = receiveChannel.readUTF8Line()
                if (greeting != null) {
                    println(greeting)
                } else {
                    println("Server closed a connection")
                    socket.close()
                    selectorManager.close()
                    exitProcess(0)
                }
            }
        }

        while (true) {
            val myMessage = readln()
            sendChannel.writeStringUtf8("$myMessage\n")
        }
    }
}
```

## References

* [Sockets](https://ktor.io/docs/servers-raw-sockets.html)