# Testing

ktor는 웹서버를 생성하지 않고, 소켓에 바인딩하지 않으며, 실제 HTTP 요청을 하지 않는 특별한 테스팅 엔진을 제공한다. 대신 내부 메커니즘에 직접 연결되어 애플리케이션 호출을 직접 처리한다. 따라서 테스트를
위해 전체 웹서버를 실행하는 것과 비교하면 테스트가 빠르다.

## **Add dependencies**

Ktor 애플리케이션 서버를 테스트하려면 다음 아티팩트를 추가해야 한다.

- `ktor-server-test-host` 디펜던시

```kotlin
testImplementation("io.ktor:ktor-server-test-host:$ktor_version")
```

- 테스트에서 assertion를 수행하기 위한 유틸리티 기능 셋을 제공하는 `kotlin-test` 디펜던시

```kotlin
testImplementation("org.jetbrains.kotlin:kotlin-test:$kotlin_version")
```

## **Testing overview**

테스팅 엔진을 사용하기 위해 다음 단계를 수행한다.

1. JUnit 테스트 클래스와 테스트 함수를 생성한다.
2. `testApplication` 함수를 사용해 로컬에서 실행되는 테스트 앱의 구성된 인스턴스를 설정한다.
3. Test 앱 내 [Ktor HTTP client](https://ktor.io/docs/create-client.html) 인스턴스를 사용해 서버에 대한 요청을 생성, 응답을 받아 assertion을 만든다.

다음 예제는 `/` 경로에 대한 `GET` 요청을 받고 일반 텍스트로 응답하는 간단한 Ktor 애플리케이션을 테스트하는 방법을 보여준다.

```kotlin
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.server.application.*
import io.ktor.server.testing.*
import kotlin.test.*

class ApplicationTest {
    @Test
    fun testRoot() = testApplication {
        val response = client.get("/")
        assertEquals(HttpStatusCode.OK, response.status)
        assertEquals("Hello, world!", response.bodyAsText())
    }
}
```

## Configure a test application

### Step 1: Add application modules

앱을 테스트하기 위해 모듈이 `testApplication`에 로드되어 있어야 한다. `testApplication`에 모듈을 로드하는 것은 서버를 생성하는 방법(`application.conf` 설정 파일을
이용/코드 상의 `embeddedServer` 함수를 이용)에 따라 다르다.

#### Add modules automatically

`resources` 폴더에 `application.conf` 파일을 가지고 있다면, `testApplication`은 파일에 설정된 속성들로 모듈이 로드된다. 특정 모듈의 자동 로드를 비활성화하려면 다음을
수행한다.

1. 테스트를 위한 [custom configuration file](https://ktor.io/docs/testing.html#environment)을 제공
2. [ktor.application.modules](https://ktor.io/docs/configurations.html#hocon-overview) 구성 속성을 사용해 로드할 모듈을 지정

#### Add modules manually

`embeddedServer`를 사용할 경우 테스트 앱에 `application` 함수를 사용해 수동으로 모듈을 추가할 수 있다.

```kotlin
fun testModule1() = testApplication {
    application {
        module1()
        module2()
    }
}
```

### Step 2: (Optional) Add routing

`routing` 함수를 이용해 테스트 앱에 route를 추가할 수 있다.

* 테스트 앱에 모듈을 추가하는 것 대신 테스트를 원하는 지정된 route를 추가할 수 있다.
* 테스트 앱에서만 필요한 route를 추가할 수 있다. 다음 예제는 사용자 세션을 초기화하는데 사용되는 `/login-test` 엔드포인트를 추가하는 방법을 보여준다.

```kotlin
fun testHello() = testApplication {
    routing {
        get("/login-test") {
            call.sessions.set(UserSession("abc123"))
        }
    }
}
```

### Step 3: (Optional) Customize environment

테스트 앱을 위한 커스텀 환경을 빌드하기 위해 `environment` 함수를 사용한다. 예를 들어, 테스트를 위한 커스텀 구성을 사용하려면, `test/resources` 폴더에 사용자 지정 구성 파일을
만들고 `config` 속성을 사용해 로드할 수 있다.

```kotlin
@Test
fun testHello() = testApplication {
        environment {
            config = ApplicationConfig("application-custom.conf")
        }
    }
```

구성 속성을 지정하는 다른
방법은 [MapApplicationConfig](https://api.ktor.io/ktor-server/ktor-server-core/io.ktor.server.config/-map-application-config/index.html?_ga=2.87955544.1396641199.1655526702-658241611.1655526702&_gl=1*brpmn*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjEuMTY1NTU0ODE1OS4w)
를 사용하는 것이다. 이는 앱 시작 전 앱 구성에 접근할 때 유용하게 사용될 수 있다. 다음 예제는 `config`를 사용해 `MapApplicationConfig`를 `testApplication` 함수에 전달하는
방법을 보여준다.

```kotlin
@Test
fun testRequest() = testApplication {
        environment {
            config = MapApplicationConfig("ktor.environment" to "test")
        }
        // Request and assertions
    }
```

#### Step 4: (Optional) Mock external services

Ktor는 `externalServices` 함수를 사용해 외부 서비스에 대해 mocking 하는 것을 허용한다. 함수 내 2개의 파라미터를 받는 `hosts` 함수를 호출해야 한다.

* `hosts` 파라미터는 외부 서비스의 URL을 허용한다.
* `block` 파라미터를 사용하면 외부 서비스에 대한 mock 역할을 수행하는 `Application`를 구성할 수 있다.

다음 샘플은 Google API에서 반환받은 JSON 응답을 시뮬레이트하기 위한 `externalServices`를 사용하는 방법을 보여준다.

```kotlin
fun testHello() = testApplication {
    externalServices {
        hosts("https://www.googleapis.com") {
            install(io.ktor.server.plugins.contentnegotiation.ContentNegotiation) {
                json()
            }
            routing {
                get("oauth2/v2/userinfo") {
                    call.respond(UserInfo("1", "JetBrains", "", "", "", ""))
                }
            }
        }
    }
}
```

### Step 5: (Optional) Configure a client

`testApplication`은 `client` 속성을 이용해 기본 구성으로 HTTP 클라이언트에 대한 접근을 제공한다. 클라이언트를 커스터마이즈하고 추가적인 플러그인이 필요한 경우, `createClient`
함수를 사용할 수 있다. 예를 들어, JSON 데이터를 test POST/PUT 요청으로 전송하기 위해, `ContentNegotiation` 플러그인을 설치할 수 있다.

```kotlin
fun testPostCustomer() = testApplication {
    val client = createClient {
        install(ContentNegotiation) {
            json()
        }
    }
}
```

### Step 6: Make a request

앱을 테스트하기 위해 [configured client](https://ktor.io/docs/testing.html#configure-client)를 사용해 요청하고 응답을 받을 수 있다. 다음
예제에서는 `POST` 요청을 처리하는 `/customer` 엔드포인트를 테스트하는 방법을 보여준다.

### Step 7: Assert results

응답을 받은
후 [kotlin.test](https://kotlinlang.org/api/latest/kotlin.test/?_ga=2.197482607.1396641199.1655526702-658241611.1655526702&_gl=1*1by766j*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTU0MTcyMi40LjEuMTY1NTU0ODE1OS4w)
라이브러리에서 제공되는 assertions를 만들어 결과에 대해 검증할 수 있다.

```kotlin
fun testPostCustomer() = testApplication {
    val client = createClient {
        install(ContentNegotiation) {
            json()
        }
    }
    val response = client.post("/customer") {
        contentType(ContentType.Application.Json)
        setBody(Customer(3, "Jet", "Brains"))
    }
    assertEquals("Customer stored correctly", response.bodyAsText())
    assertEquals(HttpStatusCode.Created, response.status)
}
```

## Test POST/PUT requests

### Send form data

`POST`/`PUT` 테스트 요청에 데이터를 전송하려면, `Content-Type` 헤더와 요청 바디를 지정해야 한다. 이를 위해
각각 [addHeader](https://api.ktor.io/ktor-server/ktor-server-test-host/ktor-server-test-host/io.ktor.server.testing/-test-application-request/add-header.html)
, [setBody](https://api.ktor.io/ktor-server/ktor-server-test-host/ktor-server-test-host/io.ktor.server.testing/set-body.html)
함수를 사용할 수 있다. 다음 예제는 `x-www-form-urlencoded`, `multipart/form-data` 타입을 사용하여 form data를 전송하는 방법을 보여준다.

### x-www-form-urlencoded

`x-www-form-urlencoded` 콘텐츠 타입을 사용하여 전송된 form 파라미터로 테스트 요청을 만드는 방법을
보여준다. [formUrlEncode](https://api.ktor.io/ktor-http/ktor-http/io.ktor.http/form-url-encode.html) 함수는 키/값 쌍 목록에서 form
파라미터를 인코딩하는데 사용된다.

```kotlin
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.server.application.*
import io.ktor.server.testing.*
import kotlin.test.*

class ApplicationTest {
    @Test
    fun testPost() = testApplication {
        val response = client.post("/signup") {
            header(HttpHeaders.ContentType, ContentType.Application.FormUrlEncoded.toString())
            setBody(
                listOf(
                    "username" to "JetBrains",
                    "email" to "example@jetbrains.com",
                    "password" to "foobar",
                    "confirmation" to "foobar"
                ).formUrlEncode()
            )
        }
        assertEquals("The 'JetBrains' account is created", response.bodyAsText())
    }
}
```

```kotlin
fun Application.main() {
    routing {
        post("/signup") {
            val formParameters = call.receiveParameters()
            val username = formParameters["username"].toString()
            call.respondText("The '$username' account is created")
        }
    }
}
```

### **multipart/form-data**

다음 예제는 `multipart/form-data`를 빌드하고 파일 업로드를 테스트하는 방법을 보여준다.

```kotlin
import io.ktor.client.request.*
import io.ktor.client.request.forms.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.http.content.*
import io.ktor.server.application.*
import io.ktor.server.testing.*
import io.ktor.utils.io.streams.*
import java.io.*
import kotlin.test.*

class ApplicationTest {
    @Test
    fun testUpload() = testApplication {
        val boundary = "WebAppBoundary"
        val response = client.post("/upload") {
            setBody(
                MultiPartFormDataContent(
                    formData {
                        append("description", "Ktor logo")
                        append("image", File("ktor_logo.png").readBytes(), Headers.build {
                            append(HttpHeaders.ContentType, "image/png")
                            append(HttpHeaders.ContentDisposition, "filename=\"ktor_logo.png\"")
                        })
                    },
                    boundary,
                    ContentType.MultiPart.FormData.withParameter("boundary", boundary)
                )
            )
        }
        assertEquals("Ktor logo is uploaded to 'uploads/ktor_logo.png'", response.bodyAsText())
    }
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.http.content.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import java.io.File

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

### Send JSON data

테스트 POST/PUT 요청에서 JSON 데이터를 전송하기 위해 새 클라이언트를 만들고 지정된 포맷을 serializing/deserializing 할 수 있는 `ContentNegotiation` 플러그인을
설치해야 한다. 요청 내 `contentType` 함수를 사용하여 `Content-Type` 헤더를 지정하고 `setBody`를 사용해 요청 바디를 지정할 수 있다. 다음 예제는 `POST` 요청을
처리하는 `/customer` 엔드포인트를 테스트하는 방법을 보여준다.

```kotlin
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.serialization.kotlinx.json.*
import io.ktor.server.application.*
import io.ktor.server.testing.*
import kotlinx.serialization.*
import kotlinx.serialization.json.*
import kotlin.test.*

class CustomerTests {
    @Test
    fun testPostCustomer() = testApplication {
        val client = createClient {
            install(ContentNegotiation) {
                json()
            }
        }
        val response = client.post("/customer") {
            contentType(ContentType.Application.Json)
            setBody(Customer(3, "Jet", "Brains"))
        }
        assertEquals("Customer stored correctly", response.bodyAsText())
        assertEquals(HttpStatusCode.Created, response.status)
    }
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.http.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.*
import kotlinx.serialization.json.*

@Serializable
data class Customer(val id: Int, val firstName: String, val lastName: String)

fun Application.main() {
    install(ContentNegotiation) {
        json(Json {
            prettyPrint = true
            isLenient = true
        })
    }
    routing {
        post("/customer") {
            val customer = call.receive<Customer>()
            customerStorage.add(customer)
            call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
        }
    }
}
```

### Preserve cookies during testing

테스팅할 때 요청 간 쿠키를 보존해야 하는
경우 [cookiesSession](https://api.ktor.io/ktor-server/ktor-server-test-host/ktor-server-test-host/io.ktor.server.testing/cookies-session.html)
함수 내에서 `handleRequest`를 호출할 수 있다. 다음 테스트 예제에서 쿠키가 보존되기 때문에 각 요청 후 다시 로드 횟수가 증가한다.

```kotlin
import io.ktor.client.plugins.cookies.*
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import io.ktor.server.application.*
import io.ktor.server.testing.*
import kotlin.test.*

class ApplicationTest {
    @Test
    fun testRequests() = testApplication {
        val client = createClient {
            install(HttpCookies)
        }

        val loginResponse = client.get("/login")
        val response1 = client.get("/user")
        assertEquals("Session ID is 123abc. Reload count is 1.", response1.bodyAsText())
        val response2 = client.get("/user")
        assertEquals("Session ID is 123abc. Reload count is 2.", response2.bodyAsText())
        val response3 = client.get("/user")
        assertEquals("Session ID is 123abc. Reload count is 3.", response3.bodyAsText())
        val logoutResponse = client.get("/logout")
        assertEquals("Session doesn't exist or is expired.", logoutResponse.bodyAsText())
    }
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.sessions.*
import io.ktor.util.*

data class UserSession(val id: String, val count: Int)

fun Application.main() {
    install(Sessions) {
        val secretEncryptKey = hex("00112233445566778899aabbccddeeff")
        val secretSignKey = hex("6819b57a326945c1968f45236589")
        cookie<UserSession>("user_session") {
            cookie.path = "/"
            cookie.maxAgeInSeconds = 10
            transform(SessionTransportTransformerEncrypt(secretEncryptKey, secretSignKey))
        }
    }
    routing {
        get("/login") {
            call.sessions.set(UserSession(id = "123abc", count = 0))
            call.respondRedirect("/user")
        }

        get("/user") {
            val userSession = call.sessions.get<UserSession>()
            if (userSession != null) {
                call.sessions.set(userSession.copy(count = userSession.count + 1))
                call.respondText("Session ID is ${userSession.id}. Reload count is ${userSession.count}.")
            } else {
                call.respondText("Session doesn't exist or is expired.")
            }
        }

        get("/logout") {
            call.sessions.clear<UserSession>()
        }
    }
}
```

## Test HTTPS

HTTPS 엔드포인트에 대한 테스트가 필요하면, [URLBuilder.protocol](https://ktor.io/docs/request.html#url) 속석을 사용해 요청을 만드는데 사용되는 프로토콜을 변경한다.

```kotlin
@Test
fun testRoot() = testApplication {
    val response = client.get("/") {
        url {
            protocol = URLProtocol.HTTPS
        }
    }
    assertEquals("Hello, world!", response.bodyAsText())
}
```

## Test WebSockets

클라이언트에서 제공하는 웹소켓을 [WebSocket](https://ktor.io/docs/websocket-client.html) 플러그인을 사용해 웹소켓을 테스트할 수 있다.

```kotlin
import io.ktor.client.plugins.websocket.*
import io.ktor.server.application.*
import io.ktor.websocket.*
import io.ktor.server.testing.*
import kotlin.test.*

class ModuleTest {
    @Test
    fun testConversation() {
        testApplication {
            val client = createClient {
                install(WebSockets)
            }

            client.webSocket("/echo") {
                val greetingText = (incoming.receive() as? Frame.Text)?.readText() ?: ""
                assertEquals("Please enter your name", greetingText)

                send(Frame.Text("JetBrains"))
                val responseText = (incoming.receive() as Frame.Text).readText()
                assertEquals("Hi, JetBrains!", responseText)
            }
        }
    }
}
```

## **End-to-end testing with HttpClient**

테스팅 엔진 외에도 [Ktor HTTP Client](https://ktor.io/docs/client.html)을 사용해 end-to-end 테스팅을 수행할 수 있다.

```kotlin
import e2e.TestServer
import io.ktor.client.*
import io.ktor.client.call.*
import io.ktor.client.request.*
import io.ktor.client.statement.*
import kotlinx.coroutines.runBlocking
import org.junit.Assert.assertEquals
import org.junit.Test

class EmbeddedServerTest: TestServer() {
  @Test
  fun rootRouteRespondsWithHelloWorldString(): Unit = runBlocking {
    val response: String = HttpClient().get("http://localhost:8080/").body()
    assertEquals("Hello, world!", response)
  }
}
```

## References

* [Testing | Ktor](https://ktor.io/docs/testing.html)