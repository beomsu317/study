# Sending responses

Ktor는 들어오는 요청과 이에 대한 응답을 route 핸들러에서 처리할 수 있다. plain text, HTML 문서, 템플릿, serialize data object 등 타입의 응답을 전송할 수 있다. 각 응답은
콘텐츠 타입, 헤더, 쿠키 등 다양한 응답 파라미터로 구성할 수 있다.

route 핸들러에서 다음 API를 사용할 수 있다.

- [call.respondText](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond-text.html)
  , [call.respondHtml](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/respond-html.html)
  등 특정 콘텐츠 타입을 보내기 위한 함수의 집합.
- [call.respond](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond.html) 함수는
  응답에 어떤 데이터든 허용한다. 예를 들어, [ContentNegotiation](https://ktor.io/docs/serialization.html) 플러그인이 활성화되어 있는 경우 data 객체를
  serialized 후 전송할 수 있다.
- 응답 파라미터에 접근을 제공하고 status code를 설정하고, 헤더를 추가하고, 쿠키를 구성할 수 있도록
  하는 [ApplicationResponse](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/-application-response/index.html)
  객체를
  반환하는 [call.response](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.application/-application-call/response.html)
  속성이다.
- [call.respondRedirect](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond-redirect.html)
  는 리다이렉트 기능을 제공한다.

# **Set response payload**

## Plain text

응답으로 평문을 전송한다.

```kotlin
get("/") {
    call.respondText("Hello, world!")
}
```

## **HTML**

Ktor는 클라이언트에게 HTML 응답을 위한 2개의 주요 방법을 제공한다.

- Kotlin HTML DSL을 통한 HTML 빌드
- FreeMarker, Velocity 등 JVM 템플릿 엔진을 사용

Kotlin DSL을 사용해 HTML 빌드를
전송하려면 [call.respondHtml](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/respond-html.html)
함수를 사용한다.

```kotlin
routing {
    get("/") {
        val name = "Ktor"
        call.respondHtml(HttpStatusCode.OK) {
            head {
                title {
                    +name
                }
            }
            body {
                h1 {
                    +"Hello from $name!"
                }
            }
        }
    }
}
```

응답에 템플릿을 전송하려면 지정된 콘텐츠와
함께 [call.respond](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond.html) 함수를
사용한다.

```kotlin
get("/index") {
    val sampleUser = User(1, "John")
    call.respond(FreeMarkerContent("index.ftl", mapOf("user" to sampleUser)))
}
```

또는 [call.respondTemplate](https://api.ktor.io/ktor-features/ktor-freemarker/ktor-freemarker/io.ktor.freemarker/respond-template.html)
함수를 사용하거나.

```kotlin
get("/index") {
    val sampleUser = User(1, "John")
    call.respondTemplate("index.ftl", mapOf("user" to sampleUser))
}
```

## **Object**

Ktor에서 data object를 serialization를 활성화하려면, [ContentNegotiation](https://ktor.io/docs/serialization.html) 플러그인을 설치하고 컨버터(
JSON 같은)를 등록해야 한다. 그런 다음 `call.respond` 함수를 사용해 data object를 응답할 수 있다.

```kotlin
get("/customer/{id}") {
    val id = call.parameters["id"]
    val customer: Customer = customerStorage.find { it.id == id!!.toInt() }!!
    call.respond(customer)
}
```

## File

파일 콘텐츠를 클라이언트에게
응답하려면, [call.respondFile](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond-file.html)
함수를 사용해야 한다. 다음 코드는 지정된 파일을 응답으로 보내고 `Content-Disposition` 헤더를 통해 다운로드 가능하도록 만드는 방법을 보여준다.

```kotlin
fun Application.main() {
    routing {
        get("/download") {
            val file = File("files/ktor_logo.png")
            call.response.header(
                HttpHeaders.ContentDisposition,
                ContentDisposition.Attachment.withParameter(ContentDisposition.Parameters.FileName, "ktor_logo.png")
                    .toString()
            )
            call.respondFile(file)
        }
    }
}
```

## **Raw payload**

raw body payload를
응답하려면 [call.respondBytes](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond-bytes.html)
함수를 사용한다.

# **Set response parameters**

## Status code

응답에 status code를
설정하려면 [ApplicationResponse.status](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/-application-response/status.html)
를 호출해야 한다. 미리정의된 status code 값을 전달할 수 있다.

```kotlin
get("/") {
    call.response.status(HttpStatusCode.OK)
}
```

또는 커스텀 status code를 지정할 수 있다.

```kotlin
get("/") {
    response.status(HttpStatusCode(418, "I'm a tea pot"))
}
```

payload를 보내기 위한 함수에는 status code를 지정하기 위한 오버로드가 있다.

## **Content type**

[ContentNegotiation](https://ktor.io/docs/serialization.html) 플러그인 설치 시 Ktor는 응답에 대한 콘텐츠 타입을 자동적으로 선택한다. 필요한 경우 콘텐츠 타입을
수동적으로 지정할 수 있다. 예를 들어, 다음 `call.respondText` 함수는 `ContentType.Text.Plain`를 파라미터로 허용한다.

```kotlin
get("/") {
    call.respondText("Hello, world!", ContentType.Text.Plain, HttpStatusCode.OK)
}
```

## **Headers**

응답에 헤더를 설정하는 몇 가지 방법이 있다.

- [ApplicationResponse.headers](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/-application-response/headers.html)
  컬렉션에 헤더 추가

    ```kotlin
    get("/") {
        call.response.headers.append(HttpHeaders.ETag, "7c876b7e")
    }
    ```

- [ApplicationResponse.header](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/header.html)
  함수 호출

    ```kotlin
    get("/") {
        call.response.header(HttpHeaders.ETag, "7c876b7e")
    }
    ```

- `ApplicationResponse.etag`, `[ApplicationResponse.link](http://ApplicationResponse.link)` 등 특정 함수 사용

    ```kotlin
    get("/") {
        call.response.etag("7c876b7e")
    }
    ```

- 커스텀 헤더를 추가하려면 다음과 같이 구현하면 된다.

    ```kotlin
    get("/") {
        call.response.header("Custom-Header", "Some value")
    }
    ```

## Cookies

응답 내 쿠키에
접근하려면, [ApplicationResponse.cookies](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/-application-response/cookies.html)
속성을 사용한다.

```kotlin
get("/") {
    call.response.cookies.append("yummy_cookie", "choco")
}
```

Ktor는 쿠키를 통한 세션을 처리하는 기능을 제공한다.

# **Redirects**

리다이렉션을 응답에 생성하기
위해 [respondRedirect](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.response/respond-redirect.html)
함수를 사용한다.

```kotlin
get("/") {
    call.respondRedirect("/moved", permanent = true)
}

get("/moved") {
    call.respondText("Moved content")
}
```

## References

* [Sending responses | Ktor](https://ktor.io/docs/responses.html)