# Sessions

Session는 다른 HTTP 요청 간 데이터를 유지하는 메커니즘을 제공한다. 일반적으로 로그인 유저를 저장하거나, 장바구니의 콘텐츠, 유저 환경설정 등. Ktor에 쿠키 또는 커스텀 헤더를 사용해 세션을 구현하고,
세션 데이터를 서버에 저장할지 또는 클라이언트로 전달할지 선택하고, 세션 데이터를 서명 및 암호화하는 등의 작업을 수행할 수 있다.

`Sessions` 플러그인 설치, 세션의 콘텐츠를 설정하여 어떻게 세션을 구성하는지 알아본다.

## Add dependencies

세션 지원을 활성화하기 위해 `ktor-server-sessions` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-server-sessions:$ktor_version")
```

## Install Sessions

`Sessions` 플러그인을 설치하기 위해, 이를 `install` 함수에 전달한다. 서버를 생성하는 방법에 따라 두 가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.sessions.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Sessions)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.sessions.*

// ...
fun Application.module() {
    install(Sessions)
    // ...
}
```

## Session configuration overview

`Sessions` 플러그인을 설정하기 위해, 다음 단계를 수행해야 한다.

1. [Create a data class](https://ktor.io/docs/sessions.html#data_class) : 세션을 설정하기 전, 세션 데이터를 저장하기 위한 data class 생성이
   필요하다.
2. [Choose how to pass data between the server and client](https://ktor.io/docs/sessions.html#cookie_header) : 쿠키 또는 커스텀
   헤더를 사용해야 한다. 쿠키는 plain HTML 애플리케이션에 적합하며, 커스텀 헤더는 API 용으로 사용된다.
3. [Choose where to store the session payload](https://ktor.io/docs/sessions.html#client_server) : 클라이언트 또는 서버에서 쿠키/헤더
   값을 사용해 serialize 된 세션 데이터를 클라이언트에 전달하거나 페이로드를 서버에 저장하고 세션 식별자만 전달할 수 있다.
    * 만약 세션 페이로드를 서버에 저장하는 경우, [어떻게 저장할지 선택](https://ktor.io/docs/sessions.html#storages)할 수 있다. (서버 메모리 또는 폴더)
    * 또한 세션을 유지하기 위한 커스텀 저장소를 구현할 수 있다.
4. [Protect session data](https://ktor.io/docs/sessions.html#protect_session) : 클라이언트에 전달되는 민감한 세션 데이터를 보호하기 위해, 세션
   페이로드에 서명하고 암호화해야 한다.

`Sessions` 설정 후 route handler를 통해 세션 데이터를 받고, 설정할 수 있다.

## Create a data class

세션을 설정하기 전 세션 데이터를 저장할 data class를 생성해야 한다. 예를 들어, 다음의 `UserSession` 클래스는 세션 ID와 페이지 뷰의 수를 저장하는데 사용된다.

```kotlin
data class UserSession(val id: String, val count: Int)
```

여러 세션을 사용하려면 여러 data class를 생성해야 한다.

>
선택적으로 [Principal](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-auth/io.ktor.server.auth/-principal/index.html?_ga=2.70054992.1396641199.1655526702-658241611.1655526702&_gl=1*z5ttc7*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMzc4OC4w)
을 상속해 data class를 생성하여 인증할 수 있다.

## Pass session data: Cookie vs Header

### Cookie

세션 데이터를 쿠키를 사용해 전달하려면, `install(Sessions)` 블록 내에서 지정된 이름과 data class로  `cookie` 메서드를 호출한다.

```kotlin
install(Sessions) {
    cookie<UserSession>("user_session")
}
```

위 예제에서 세션 데이터는 `Set-Cookie` 헤더에 추가된 `user_session` 속성을 사용해 클라이언트에 전달된다. `cookie` 블록에서 다른 쿠키 속성 전달해 구성할 수 있다. 다음 예제에선 쿠키
경로와 만료 시간을 지정하는 방법을 보여준다.

```kotlin
install(Sessions) {
    cookie<UserSession>("user_session") {
        cookie.path = "/"
        cookie.maxAgeInSeconds = 10
    }
}
```

필수 속성이 명시적으로 노출되지 않은 경우, `extensions` 속성을 사용하면 된다. 예를 들어, `SameSite` 속성을 다음과 같이 전달할 수 있다.

```kotlin
install(Sessions) {
    cookie<UserSession>("user_session") {
        cookie.extensions["SameSite"] = "lax"
    }
}
```

### **Header**

커스텀 헤더를 통해 세션 데이터를 전달하려면, `install(Sessions)` 블럭 안에서 지정된 이름과 data class와 함께 `header` 함수를 호출해야 한다.

```kotlin
install(Sessions) {
    header<CartSession>("cart_session")
}
```

위 예제에서 세션 데이터는 `cart_session` 커스텀 헤더를 통해 클라이언트에 전달된다. 클라이언트 측에서는 각 요청마다 추가하여 세션 데이터를 얻어야 한다.

> CORS 플러그인을 사용해 cross-origin 요청에 대해 처리하는 경우, 커스텀 헤더를 CORS 설정에 추가해야 한다.
> ```kotlin
> install(CORS) {
>    allowHeader("cart_session")
>    exposeHeader("cart_session")
> }
> ```

## Store session payload: Client vs Server

Ktor에서 세션 데이터를 2가지 방법으로 관리할 수 있다.

- 서버와 클라이언트 간 세션 데이터를 전달한다. 이 경우 페이로드를 serialize 및 transform하여 클라이언트에 전송된 데이터를 서명하거나 암호화하는 방법을 지정할 수 있다.
- 세션 데이터를 서버에 저장하고 세션 ID만 클라이언트에 전달한다. 페이로드를 서버의 어디에 저장할 지 선택해야 한다. 예를 들어, 세션을 메모리에 저장하거나, 특정 폴더 또는 Redis에 저장하는 등. 커스텀
  저장소를 구현할 수도 있다.

## Store session payload on server

Ktor는 서버에 세션 데이터를 저장하고, 세션 ID만 서버와 클라이언트 측에 전달할 수 있다. 이 경우 서버 내 어디에 페이로드를 유지할지 선택해야 한다.

### In-memory storage

[SessionStorageMemory](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-sessions/io.ktor.server.sessions/-session-storage-memory/index.html?_ga=2.155539579.1396641199.1655526702-658241611.1655526702&_gl=1*1al8e30*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMzc4OC4w)
는 세션 내용을 메모리에 저장하도록 활성화한다. 이 저장소는 서버가 실행중인 경우에만 유지되며, 서버가 멈추면 세션 데이터는 버려진다. 다음과 같이 서버 메모리에 저장할 수 있다.

```kotlin
cookie<CartSession>("cart_session", SessionStorageMemory()) {
}
```

> `SessionStorageMemory`는 오직 개발 목적으로 의도되었다.

### Directory storage

[directorySessionStorage](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-sessions/io.ktor.server.sessions/directory-session-storage.html?_ga=2.122101707.1396641199.1655526702-658241611.1655526702&_gl=1*g4j2h9*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMzc4OC4w)
는 세션 데이터를 특정 디렉토리 하위에 저장하기 위해 사용한다. 예를 들어, `build/.sessions` 디렉토리에 저장하려는 경우, `directorySessionStorage`를 다음과 같이 구현한다.

```kotlin
header<CartSession>("cart_session", directorySessionStorage(File("build/.sessions"))) {
}
```

### Custom storage

Ktor는 커스텀 저장소를 구현하기
위한 [SessionStorage](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-sessions/io.ktor.server.sessions/-session-storage/index.html?_ga=2.104340288.1396641199.1655526702-658241611.1655526702&_gl=1*18kudrb*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMzc4OC4w)
인터페이스를 제공한다.

```kotlin
interface SessionStorage {
    suspend fun invalidate(id: String)
    suspend fun write(id: String, provider: suspend (ByteWriteChannel) -> Unit)
    suspend fun <R> read(id: String, consumer: suspend (ByteReadChannel) -> R): R
}
```

이 세 가지 함수는 `suspending`이며, `ByteReadChannel`과 `ByteWriteChannel`을 사용하여 비동기 채널에서 데이터를 읽고
쓴다. [SessionStorageMemory](https://github.com/ktorio/ktor/blob/main/ktor-server/ktor-server-plugins/ktor-server-sessions/jvm/src/io/ktor/server/sessions/SessionStorageMemory.kt)
를 참고하자.

## Protect session data

### Sign session data

세션 데이터에 서명하면 세션 내용에 대한 변조를 방지할 수 있지만, 사용자가 이 내용을 볼 수 있다. 세션에 서명하기 위해 서명
키를 `SessionTransportTransformerMessageAuthentication` 생성자에 전달하고, 이 인스턴스를 `transform` 함수에 전달한다.

```kotlin
install(Sessions) {
    val secretSignKey = hex("6819b57a326945c1968f45236589")
    cookie<CartSession>("cart_session", SessionStorageMemory()) {
        cookie.path = "/"
        transform(SessionTransportTransformerMessageAuthentication(secretSignKey))
    }
}
```

`SessionTransportTransformerMessageAuthentication`는 기본 알고리즘으로 `HmacSHA256`를 사용하며, 변경 가능하다.

### Sign and encrypt session data

세션 데이터에 대해 서명과 암호화를 하면 세션 내용을 읽거나 변조할 수 없도록 막을 수 있다. 세션에 서명하고 암호화하기 위해 sing/encrypt
키를 `SessionTransportTransformerEncrypt` 생성자에 전달하고, 이 인스턴스를 `transform` 함수에 전달한다.

```kotlin
install(Sessions) {
    val secretEncryptKey = hex("00112233445566778899aabbccddeeff")
    val secretSignKey = hex("6819b57a326945c1968f45236589")
    cookie<UserSession>("user_session") {
        cookie.path = "/"
        cookie.maxAgeInSeconds = 10
        transform(SessionTransportTransformerEncrypt(secretEncryptKey, secretSignKey))
    }
}
```

기본적으로 `SessionTransportTransformerEncrypt`는 `AES`와 `HmacSHA256` 알고리즘을 사용하며, 이는 변경할 수 있다.

## Get and set session content

특정 route에서 세션 컨텐츠를 설정하기 위해선 `call.sessions` 속성을 사용한다. `set` 메서드를 통해 새로운 세션 인스턴스를 생성할 수 있다.

```kotlin
get("/login") {
    call.sessions.set(UserSession(id = "123abc", count = 0))
    call.respondRedirect("/user")
}
```

세션 컨텐츠를 얻기 위해, 등록된 세션 타입 중 하나를 타입 파라미터로 수신하는 `get` 메서드를 호출할 수 있다.

```kotlin
get("/user") {
    val userSession = call.sessions.get<UserSession>()
}
```

세션을 변경하기 위해(예를 들어, counter를 증가하는 것) `copy` 메서드를 사용한다.

```kotlin
get("/user") {
    val userSession = call.sessions.get<UserSession>()
    if (userSession != null) {
        call.sessions.set(userSession.copy(count = userSession.count + 1))
        call.respondText("Session ID is ${userSession.id}. Reload count is ${userSession.count}.")
    } else {
        call.respondText("Session doesn't exist or is expired.")
    }
}
```

세션을 clear 하려면 `clear` 함수를 호출하면 된다.

```kotlin
get("/logout") {
    call.sessions.clear<UserSession>()
    call.respondRedirect("/user")
}
```

## References

* [Sessions](https://ktor.io/docs/sessions.html)