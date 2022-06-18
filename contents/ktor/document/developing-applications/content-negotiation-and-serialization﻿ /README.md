# Content negotiation and serialization

[ContentNegotiation](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-content-negotiation/io.ktor.server.plugins.contentnegotiation/-content-negotiation.html?_ga=2.201126894.1396641199.1655526702-658241611.1655526702&_gl=1*1f7mw7u*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMDU4NS4w) 플러그인은 두 가지의 주 목적을 제공한다.

* 클라이언트와 서버 간 미디어 타입의 협상
* 지정된 포맷에 대한 Serializing/deserializing. Ktor는 바로 사용할 수 있는 JSON, XML, CBOR 포맷을 지원한다.

## Add dependencies

### ContentNegotiation

`ContentNegotiation`을 사용하기 위해 `ktor-server-content-negotiation` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-content-negotiation:$ktor_version")
```

특정한 포맷의 Serializers는 추가적인 아티팩트가 필요할 수 있다. 예를 들어, `kotlinx.serialization`는 `ktor-serialization-kotlinx-json` 디펜던시가 필요하다.

### JSON

JSON 데이터를 serialize/deserialize 하기 위해 `kotlinx.serialization`, `Gson`, `Jackson` 라이브러리 중 하나를 선택해야 한다.

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

## Install ContentNegotiation

`ContentNegotiation` 플러그인을 설치하기 위해, 이를 `install` 함수에 파라미터로 전달해야 한다. 서버를 생성하는 방법에 따라 다음과 같이 나눠진다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.contentnegotiation.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(ContentNegotiation)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.contentnegotiation.*
// ...
fun Application.module() {
    install(ContentNegotiation)
    // ...
}
```

## Configure a serializer

Ktor는 바로 사용할 수 있는 JSON, XML, CBOR 포맷을 제공한다. 또한 자체적인 커스텀 serializer를 구현할 수 있다.

### JSON serializer

애플리케이션에 JSON serializer를 등록하기 위해 `json` 함수를 호출한다.

```kotlin
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.json.*

install(ContentNegotiation) {
    json()
}
```

`json` 메서드는 serialization 설정이 가능한 [JsonBuilder](https://kotlin.github.io/kotlinx.serialization/kotlinx-serialization-json/kotlinx.serialization.json/-json-builder/index.html)를 제공한다.

```kotlin
install(ContentNegotiation) {
    json(Json {
        prettyPrint = true
        isLenient = true
    })
}
```

### XML serializer

XML serializer를 애플리케이션에 등록하기 위해 `xml` 메서드를 호출한다.

```kotlin
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.xml.*

install(ContentNegotiation) {
    xml()
}
```

`xml` 메서드에서 XML serialization 설정이 가능하다.

```kotlin
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.xml.*
import nl.adaptivity.xmlutil.*
import nl.adaptivity.xmlutil.serialization.*

install(ContentNegotiation) {
    xml(format = XML {
        xmlDeclMode = XmlDeclMode.Charset
    })
}
```

### CBOR serializer

CBOR serializer를 애플리케이션에 등록하기 위해 `cbor` 메서드를 호출한다.

```kotlin
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.cbor.*

install(ContentNegotiation) {
    cbor()
}
```

`cbor` 메서드에서 CBOR serialization 설정이 가능한 [CborBuilder](https://kotlin.github.io/kotlinx.serialization/kotlinx-serialization-cbor/kotlinx.serialization.cbor/-cbor-builder/index.html)를 제공한다.

```kotlin
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.cbor.*
import kotlinx.serialization.cbor.*

install(ContentNegotiation) {
    cbor(Cbor {
        ignoreUnknownKeys = true
    })
}
```

### Custom serializer

특정 `Content-Type`에 대한 커스텀 serializer를 등록하기 위해 `register` 메서드를 호출한다. 다음 예제에서, `application/json` 및 `application/xml` 데이터를 deserialize 하기 위해 두 개의 커스텀 serializer가 등록되었다. 

```kotlin
install(ContentNegotiation) {
    register(ContentType.Application.Json, CustomJsonConverter())
    register(ContentType.Application.Xml, CustomXmlConverter())
}
```

## Receive and send data

### Create a data class

수신된 데이터를 객체로 deserialize 하기 위해 data class를 생성해야 한다.

```kotlin
data class Customer(val id: Int, val firstName: String, val lastName: String)
```

`kotlinx.serialization`를 사용하면, `@Serializable` 어노테이션을 추가해야 한다.

```kotlin
import kotlinx.serialization.*

@Serializable
data class Customer(val id: Int, val firstName: String, val lastName: String)
```

### Receive data

요청에 대한 내용을 수신하고 변환하려면 data class를 파라미터로 받아들이는 `receive` 메서드를 호출해야 한다.

```kotlin
post("/customer") {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
}
```
트

요청의 `Content-Type`은 요청 처리를 위한 serializer를 선택하기 위해 사용된다. 다음 예제에서는 서버 측에서 `Customer` 객체로 변환되는 JSON 또는 XML 데이터가 포함된 HTTP 클라이언트 요청 샘플이다.

```
POST http://0.0.0.0:8080/customer
Content-Type: application/json

{
  "id": 3,
  "firstName" : "Jet",
  "lastName": "Brains"
}
```

```
POST http://0.0.0.0:8080/customer
Content-Type: application/xml

<Customer id="3" firstName="Jet" lastName="Brains"/>
```

### Send data

data 객체를 응답으로 전달하기 위해 `respond` 메서드를 사용한다.

```kotlin
get("/customer/{id}") {
    val id = call.parameters["id"]
    val customer: Customer = customerStorage.find { it.id == id!!.toInt() }!!
    call.respond(customer)
}
```

이 경우, Ktor는 `Accept` 헤더를 사용해 필요한 serializer를 선택할 수 있도록 한다. 

## Implement a custom serializer

Ktor에서 자체적인 serializer를 작성할 수 있다. 이를 위해 [ContentConverter](https://api.ktor.io/ktor-shared/ktor-serialization/io.ktor.serialization/-content-converter/index.html?_ga=2.197531631.1396641199.1655526702-658241611.1655526702&_gl=1*1lefqkc*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMDk3NC4w) 인터페이스를 구현해야 한다.

```kotlin
interface ContentConverter {
    suspend fun serialize(contentType: ContentType, charset: Charset, typeInfo: TypeInfo, value: Any): OutgoingContent?
    suspend fun deserialize(charset: Charset, typeInfo: TypeInfo, content: ByteReadChannel): Any?
}
```

[GsonConverter](https://github.com/ktorio/ktor/blob/main/ktor-shared/ktor-serialization/ktor-serialization-gson/jvm/src/GsonConverter.kt)를 참고하자.

## References

* [Content negotiation and serialization](https://ktor.io/docs/serialization.html)