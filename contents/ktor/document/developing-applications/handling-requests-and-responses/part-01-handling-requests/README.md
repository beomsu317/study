# Handling requests

Ktor는 들어오는 요청과 이에 대한 응답 전송을 route 핸들러에서 처리할 수 있다. 요청을 처리할 때 다양한 액션을 수행할 수 있다.

- 헤더 쿠키 등 요청 정보를 얻을 수 있다.
- route 파라미터 값을 얻을 수 있다.
- 쿼리 스트링 파라미터를 얻을 수 있다.
- data 객체, form 파라미터, 파일 등 바디 콘텐츠를 받을 수 있다.

## **General request information**

route 핸들러 내부에서 `call.request` 속성을 사용해 요청 정보에 접근할 수 있다.
이는 [ApplicationRequest](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/index.html)
인스턴스를 반환하며 다양한 요청 파라미터에 접근을 제공한다. 예를 들어, 다음 코드는 요청 URL에 대한 정보를 가져온다.

```kotlin
routing {
    get("/") {
        val uri = call.request.uri
        call.respondText("Request uri: $uri")
    }
}
```

[ApplicationRequest](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/index.html)
객체는 다양한 요청 데이터에 접근할 수 있게 해준다.

- Headers
    - 모든 요청 헤더에 접근하기
      위해 [ApplicationRequest.headers](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/headers.html)
      속성을 사용한다. 확장 함수를 사용해 `acceptEncoding`, `contentType`, `cacheControl` 등 특정 헤더에 접근할 수 있다.
- Cookies
    - [ApplicationRequest.cookies](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/cookies.html)
      속성은 요청과 관련된 쿠키에 대한 접근을 제공한다.
- Connection details
    - [ApplicationRequest.local](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/local.html)
      속성을 사용해 호스트명, 포트, 스키마 등 연결 상세 정보에 접근할 수 있다.
- `X-Forwarded-` headers
    - HTTP 프록시 또는 로드 밸런서를 통해 전달된 요청에 대한 정보를 얻으려면, [ForwardedHeaderSupport](https://ktor.io/docs/forward-headers.html)
      플러그인을
      설치하고 [ApplicationRequest.origin](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/origin.html)
      속성을 사용한다.

## **Path parameters**

요청을 처리할 때, `call.parameters` 속성을 통해 route 파라미터에 접근할 수 있다. 예를 들어, 다음 `call.parameters["login"]`은 `/user/admin` path를
제공한다.

```kotlin
get("/user/{login}") {
    if (call.parameters["login"] == "admin") {
        // ...
    }
}
```

## **Query parameters**

쿼리 스트링의 파라미터에 접근하기
위해서 [ApplicationRequest.queryParameters](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/-application-request/query-parameters.html)
속성을 사용한다. 예를 들어, `/products?price=asc` 요청의 경우, `price` 쿼리 파라미터에 다음과 같이 접근할 수 있다.

```kotlin
get("/products") {
    if (call.request.queryParameters["price"] == "asc") {
        // Show products from the lowest price to the highest
    }
}
```

또한 [ApplicationRequest.queryString](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/query-string.html)
함수를 사용해 전체 쿼리 스트링을 얻을 수 있다.

## **Body contents**

이 섹션은 `POST`, `PUT`, `PATCH`와 함께 바디 콘텐츠를 받는 방법을 보여준다.

### Objects

Ktor는 요청의 미디어 타입을 협상하고, 콘텐츠를 객체로 deserialize하는 [ContentNegotiation](https://ktor.io/docs/serialization.html) 플러그인을 제공한다.
요청의 콘텐츠를 받고 변환하기 위해 data class를 파라미터로 받는 `receive` 메서드를 호출한다.

```kotlin
post("/customer") {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
}
```

### Form parameters

Ktor는 [receiveParameters](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/receive-parameters.html)
함수를 사용해 `x-www-form-urlencoded` 및 `multipart/form-data` 타입으로 전송된 파라미터를 수신할 수 있다. 다음은 바디의 form 파라미터를 전달하는 HTTP Client
예제이다.

```
POST http://localhost:8080/signup
Content-Type: application/x-www-form-urlencoded

username=JetBrains&email=example@jetbrains.com&password=foobar&confirmation=foobart
```

파라미터 값을 다음과 같이 얻을 수 있다.

```kotlin
post("/signup") {
    val formParameters = call.receiveParameters()
    val username = formParameters["username"].toString()
    call.respondText("The '$username' account is created")
}
```

### **Multipart form data**

multipart 요청의 일부로 전송된 파일을 수신해야 하는
경우, [receiveMultipart](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/receive-multipart.html)
함수를 호출하고, 필요에 따라 각 부분을 반복한다. 아래 예제에서 byte stream으로써 파일을 받기 위해 `PartData.FileItem`을 사용했다.

```kotlin
fun Application.main() {
    routing {
        var fileDescription = ""
        var fileName = ""

        post("/upload") {
            val multipartData = call.receiveMultipart()

            multipartData.forEachPart { part ->
                when (part) {
                    is PartData.FormItem -> {
                        fileDescription = part.value
                    }
                    is PartData.FileItem -> {
                        fileName = part.originalFileName as String
                        var fileBytes = part.streamProvider().readBytes()
                        File("uploads/$fileName").writeBytes(fileBytes)
                    }
                }
            }

            call.respondText("$fileDescription is uploaded to 'uploads/$fileName'")
        }
    }
}
```

> 업로드된 파일 사이즈를 정하기 위해 `Content-Length` 헤더 값을 `post` 핸들러에서 얻을 수 있다.
> ```kotlin
> post("/upload") {
>    val contentLength = call.request.header(HttpHeaders.ContentLength)
>    // ...
> }
> ```

### Raw payload

raw body payload에 접근하고 싶은 경우 다음 함수를 사용한다.

- [receiveChannel](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/receive-channel.html)
- [receiveStream](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/receive-stream.html)
- [receiveText](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.request/receive-text.html)

## References

* [Handling requests | Ktor](https://ktor.io/docs/requests.html)