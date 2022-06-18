# WebSockets serialization

`ContentNegotiation` 플러그인과 유사하게 웹소켓에서 text frame을 serialize/deserialize 할 수 있다. Ktor는 JSON, XML, CBOR 포맷에 대해 지원하고 있다.

## Add dependencies

### JSON

JSON 데이터를 serialize/deserialize 하기 위해 `kotlinx.serialization`, `Gson`, `Jackson` 중 하나의 라이브러리를 선택한다.

1. Kotlin serialization 플러그인을 추가하기 위해 [Setup](https://github.com/Kotlin/kotlinx.serialization#setup) 섹션을 참고하자.
2. `ktor-serialization-kotlinx-json` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-serialization-kotlinx-json:$ktor_version")
```

### XML

XML을 serialize/deserialize 하기 위해 `ktor-serialization-kotlinx-xml` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-serialization-kotlinx-xml:$ktor_version")
```

### CBOR

CBOR을 serialize/deserialize 하기 위해 `ktor-serialization-kotlinx-cbor` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-serialization-kotlinx-cbor:$ktor_version")
```

## Configure a serializer

Ktor는 JSON, XML, CBOR 포맷에 대해 지원한다.

### JSON serializer

웹소켓 설정에서, JSON serializer를 등록하기 위해, `Json` 파라미터와 함께 `KotlinxWebsocketSerializationConverter` 인스턴스를 만들고 `contentConverter` 속성에 생성된 인스턴스를 할당한다.

```kotlin
import io.ktor.serialization.kotlinx.*
import kotlinx.serialization.json.*

install(WebSockets) {
    contentConverter = KotlinxWebsocketSerializationConverter(Json)
}
```

### XML serializer

웹소켓 설정에서, XML serializer를 등록하기 위해, `XML` 파라미터와 함께 `KotlinxWebsocketSerializationConverter` 인스턴스를 만들고 `contentConverter` 속성에 생성된 인스턴스를 할당한다.

```kotlin
import nl.adaptivity.xmlutil.serialization.*

install(WebSockets) {
    contentConverter = KotlinxWebsocketSerializationConverter(XML)
}
```

### CBOR serializer

웹소켓 설정에서, CBOR serializer를 등록하기 위해, `Cbor` 파라미터와 함께 `KotlinxWebsocketSerializationConverter` 인스턴스를 만들고 `contentConverter` 속성에 생성된 인스턴스를 할당한다.

```kotlin
import io.ktor.serialization.kotlinx.cbor.*

install(WebSockets) {
    contentConverter = KotlinxWebsocketSerializationConverter(Cbor)
}
```

## Receive and send data

### Create a data class

Frame을 객체로 serialize/deserialize 하기 위해 data class를 생성한다.

```kotlin
data class Customer(val id: Int, val firstName: String, val lastName: String)
```

`kotlinx.serialization`를 사용할 경우 `@Serializable` 어노테이션을 추가한다.

```kotlin
@Serializable
data class Customer(val id: Int, val firstName: String, val lastName: String)
```

### Receive data

Text frame을 받고 변환하기 위해 해당하는 data class 파라미터로 받는 `receiveDeserialized` 함수를 호출한다.

```kotlin
webSocket("/customer") {
    val customer = receiveDeserialized<Customer>()
    println("A customer with id ${customer.id} is received by the server.")
}
```

### Send data

Data 객체를 text frame에 지정된 포맷으로 전달하기 위해 `sendSerialized` 함수를 사용한다.

```kotlin
webSocket("/customer/1") {
    sendSerialized(Customer(1, "Jane", "Smith"))
}
```

## References

* [WebSockets serialization](https://ktor.io/docs/websocket-serialization.html)