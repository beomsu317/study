# Session authentication

[Sessions](https://ktor.io/docs/sessions.html)은 다른 HTTP 요청 간 데이터를 유지하는 메커니즘을 제공한다. 일반적으로 로그인 유저를 저장하거나, 장바구니의 콘텐츠, 유저
환경설정 등.

Ktor에서 이미 연결된 세션이 있는 유저는 `session` 제공자를 통해 인증될 수 있다. 예를 들어, 유저가 [web form](https://ktor.io/docs/form.html)을 이용해 로그인한 경우,
username을 쿠키 세션으로 저장하고, 추후 요청들을 `session` 제공자를 사용해 인증할 수 있다.

## **Add dependencies**

`session` 인증을 활성화하기 위해 다음 아티팩트를 추가한다.

- `ktor-server-sessions` 디펜던시

```kotlin
implementation("io.ktor:ktor-server-sessions:$ktor_version")
```

- `ktor-auth` 디펜던시

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

## **Session authentication flow**

세션 인증 flow는 다양하고 유저가 어떻게 인증했는지에 의존한다. `form-based` 인증을 사용한 경우의 flow를 알아본다.

1. 클라이언트는 web form 데이터(username, password 포함)를 포함한 서버로의 요청을 생성한다.
2. 서버는 클라이언트로부터 받은 크레덴셜을 검증하고 username을 쿠키 세션으로 저장한 후 요청 콘텐츠와 username이 포함된 쿠키로 응답한다.
3. 클라이언트는 보호된 리소스에 대해 쿠키를 사용해 후속 요청을 만든다.
4. 수신된 쿠키 데이터 기반하여, Ktor는 이 유저에 대한 쿠키 세션이 있는지 확인하고, 선택적으로 수신된 세션 데이터에 대한 추가 검증을 수행한다. 검증이 성공하면 요청 콘텐츠 로응답한다.

## **Install session authentication**

`session` 인증 제공자를 설치하기 위해 `install` 블록에서 `session` 함수를 호출한다.

```kotlin
install(Authentication) {
    session<UserSession> {
        // Configure session authentication
    }
}
```

## **Configure session authentication**

이 섹션은 [form-based authentication](https://ktor.io/docs/form.html)으로 유저를 인증, 유저의 정보를 쿠키 세션으로 저장, `session` 제공자를 사용한 후속
요청에 대한 인증하는 방법을 설명한다.

### **Step 1: Create a data class**

우선 세션 데이터를 저장할 data class를 생성한다. 이 클래스는 [validate](https://ktor.io/docs/session-auth.html#configure-session-auth) 함수에서
인증에 성공한 경우 `Principal`을 반환하기
위해 [Principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-principal/index.html)을 상속해야 한다.

```kotlin
data class UserSession(val name: String, val count: Int) : Principal
```

### **Step 2: Install and configure a session**

data class 생성 후 `Sessions` 플러그인을 설치하고 구성해야 한다. 다음 코드에서는 지정된 쿠키 경로 및 만료 시간을 사용해 쿠키 세션을 설치 및 구성하는 방법을 보여준다.

```kotlin
install(Sessions) {
    cookie<UserSession>("user_session") {
        cookie.path = "/"
        cookie.maxAgeInSeconds = 60
    }
}
```

### **Step 3: Configure session authentication**

`session` 인증 제공자는
설정을 [SessionAuthenticationProvider/Configuration](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-session-authentication-provider/-configuration/index.html)
클래스를 노출한다. 예제에서 다음 설정이 지정된다.

- `validate` 함수는 session 인스턴스를 체크하고, 인증에 성공하면 `Principal`을 반환한다.
- `challenge` 함수는 실패했을 경우 지정된 작업을 수행한다. 예를 들어, 로그인 페이지로의
  리다이렉션이나 [UnauthorizedResponse](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-unauthorized-response/index.html)
  를 전송하는 것.

```kotlin
install(Authentication) {
    session<UserSession>("auth-session") {
        validate { session ->
            if (session.name.startsWith("jet")) {
                session
            } else {
                null
            }
        }
        challenge {
            call.respondRedirect("/login")
        }
    }
}
```

### **Step 4: Save user data in a session**

[call.sessions.set](https://ktor.io/docs/sessions.html#set-content) 함수를 사용해 로그인 유저에 대한 정보를 세션으로 저장한다.

```kotlin
authenticate("auth-form") {
    post("/login") {
        val userName = call.principal<UserIdPrincipal>()?.name.toString()
        call.sessions.set(UserSession(name = userName, count = 1))
        call.respondRedirect("/hello")
    }
}
```

### **Step 5: Define authorization scope**

`session` 제공자 구성 후 `authenticate` 함수를 사용해 다양한 리소스에 대한 권한을 정의할 수 있다. 인증이 성공한 경우 route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 `Principal`([UserSession](https://ktor.io/docs/session-auth.html#data-class) 인스턴스)을 얻을 수 있다.

```kotlin
authenticate("auth-session") {
    get("/hello") {
        val userSession = call.principal<UserSession>()
        call.sessions.set(userSession?.copy(count = userSession.count + 1))
        call.respondText("Hello, ${userSession?.name}! Visit count is ${userSession?.count}.")
    }
}
```

## References

* [Session authentication | Ktor](https://ktor.io/docs/session-auth.html)