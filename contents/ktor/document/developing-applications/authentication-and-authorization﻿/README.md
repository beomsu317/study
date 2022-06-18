# Authentication and authorization

Ktor는 인증 및 권한 부여를 처리하기 위한 `Authentication` 기능을 제공한다. 일반적으로 사용 시나리오는 사용자 로그인, 특정 리소스에 대한 접근 권한 부여, 안전한 전송 등이 포함된다.
또한 [Sessions](https://ktor.io/docs/sessions.html)을 통한  `Authentication`을 사용해 유저의 정보를 유지할 수도 있다.

# **Supported authentication types**

Ktor는 다음 인증과 권한 부여 스키마를 지원한다.

## **HTTP authentication**

HTTP는 접근 제어를 위한 일반적인 프레임워크를 제공한다. Ktor에서는 HTTP 인증 스키마를 사용할 수 있다.

- [Basic](https://ktor.io/docs/basic.html) - `Base64` 인코딩을 사용해 username과 password를 제공한다. 일반적으로 HTTPS와 함께 사용하는 것을 권고한다.
- [Digest](https://ktor.io/docs/digest.html) - username과 password에 해시 함수를 적용하여 암호화된 형태로 유저 크레덴셜을 전달하는 인증 방식.
- `Bearer` - bearer 토큰이라 불리는 보안 토큰을 포함한 인증 스키마. [JSON Web Tokens](https://ktor.io/docs/authentication.html#jwt) 토큰을
  bearer 토큰으로 사용할 수 있고, Ktor에서 `jwt` 인증을 사용해 유저를 검증할 수 있다.

## **Form-based authentication**

[web form](https://developer.mozilla.org/en-US/docs/Learn/Forms)을 사용해 크레덴셜 정보를 수집하고 유저를 인증한다.

## **LDAP**

LDAP는 디렉토리 서비스 인증에 사용되는 open and cross-platform 프로토콜이다. Ktor는 지정된 LDAP 서버에 대해 유저 크레덴셜을
검증하는 [ldapAuthenticate](https://api.ktor.io/ktor-features/ktor-auth-ldap/ktor-auth-ldap/io.ktor.auth.ldap/ldap-authenticate.html)
함수를 제공한다.

## **OAuth**

[OAuth](https://ktor.io/docs/oauth.html)는 API에 대한 접근을 보호하기 위한 open standard이다. Ktor의 `oauth` 제공자는 구글, 페이스북, 트위터 등 외부
제공자를 사용해 인증을 구현할 수 있다.

## Session

[Sessions](https://ktor.io/docs/sessions.html)는 서로 다른 HTTP 요청 간 데이터를 유지하는 메커니즘을 제공한다. 일반적으로 로그인한 유저의 ID, 쇼핑 장바구니의 내용,
클라이언트 측 유저의 환경 설정 등을 저장한다. Ktor에서 이미 연결된 세션이 있는 유저는  `session` 제공자를 사용해 인증될 수 있다.

# **Add dependencies**

인증을 활성화하기 위해 `ktor-auth` 아티팩트를 포함한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

JWT, LDAP와 같은 일부 인증 제공자는 추가적인 아티팩트가 필요할 수 있다.

# **Install Authentication**

`Authentication` 플러그인을 설치한다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Authentication)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(Authentication)
    // ...
}
```

# **Configure Authentication**

`Authentication` 설치 후, `Authentication` 사용을 위해 설정이 필요하다.

## **Step 1: Choose an authentication provider**

특정 인증 제공자(basic, digest, form 등)를 사용하기 위해 `install` 블록에서 일치하는 함수를 호출해야 한다. 다음 예에선 `basic` 인증을 사용하기에, `basic` 함수를 호출했다.

```kotlin
install(Authentication) {
    basic {
        // Configure basic authentication
    }
}
```

## **Step 2: Specify a provider name**

특정 제공자를 사용하는 함수를 사용하면 선택적으로 제공자 이름을 지정할 수 있다. 다음 예제에서 `auth-basic`과 `auth-form` 이름을 사용하는 `basic`과 `form` 제공자를 설치했다.

```kotlin
install(Authentication) {
    basic("auth-basic") {
        // Configure basic authentication
    }
    form("auth-form") {
        // Configure form authentication
    }
    // ...
}
```

이 지정된 이름들은 다른 제공자를 사용해 다른 route를 인증하는데 사용할 수 있다.

## **Step 3: Configure a provider**

각 제공자 타입은 자체의 설정이 있다. 예를
들어, [BasicAuthenticationProvider.Configuration](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-basic-authentication-provider/-configuration/index.html)
클래스는 basic 함수에 전달된 옵션이 포함되어 있다. 이 클래스에서 노출되는 가장 중요한 함수는 `validate`이며, 이는 username과 password를 검증한다.

```kotlin
install(Authentication) {
    basic("auth-basic") {
        realm = "Access to the '/' path"
        validate { credentials ->
            if (credentials.name == "jetbrains" && credentials.password == "foobar") {
                UserIdPrincipal(credentials.name)
            } else {
                null
            }
        }
    }
}
```

`validate` 함수의 동작을 이해하기 위해 2개의 용어를 소개한다.

- principal(유저, 컴퓨터, 서비스 등)은 인증할 수 있는 엔티티이다. Ktor에서 다양한 인증 제공자는 다른 principal을 사용할 수 있다. 예를 들어, `basic`과 `form`
  제공자는 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)
  을 인증하는 반면 `jwt`
  제공자는 [JWTPrincipal](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-principal/index.html)
  을 확인한다.

> 만약 세션 인증을 사용하면, principal은 세션 데이터로 저장되는 data class 일 수 있다.
>

- 크레덴셜(user/password, API key 등)은 principal을 인증하기 위한 서버 속성의 집합이다. 예를 들어, `basic`과 `form`
  제공자는 [UserPasswordCredential](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-password-credential/index.html)
  을 제공하여 username과 password를 검증하지만 `jwt`
  는 [JWTCredential](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-credential/index.html)
  을 사용한다.

`validate` 함수는
지정된 [Credential](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-credential/index.html)을 검증하고 인증된
경우 [Principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-principal/index.html)을 반환하고, 실패한
경우 `null`을 반환한다.

> 특정 기준에 따라 인증을 건너뛰려면 `skipWhen`을 사용한다. 예를 들어, 세션이 이미 있는 경우 `basic` 인증을 스킵할 수 있다.
>

```kotlin
basic {
    skipWhen { call -> call.sessions.get<UserSession>() != null }
}
```

## **Step 4: Define authorization scope**

마지막 단계는 다른 리소스에 대한 권한 부여를 정의하는
것이다. [authenticate](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/authenticate.html) 함수를 사용해 수행할 수
있다. 이 함수는 중첩 route를 인증하는데 사용되는 제공자의 이름을 받을 수 있다. 다음 코드는 `auth-basic` 이름의 제공자를 사용해 `/login`와 `/orders` route를 보호한다.

```kotlin
routing {
    authenticate("auth-basic") {
        get("/login") {
            // ...
        }
        get("/orders") {
            // ...
        }
    }
    get("/") {
        // ...
    }
}
```

이름 없는 제공자를 사용하려면 제공자 이름을 생략할 수 있다.

## **Step 5: Get a principal inside a route handler**

인증에 성공한 경우 route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를 사용해
인증된 [Principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-principal/index.html)을 반환할 수 있다. 이
함수는 설정된 인증 제공자가 반환한 특정 principle 타입을 허용한다. 다음 예제에서 `call.principal`은 `UserIdPrincipal`을 얻기 위해 사용되며, 인증된 사용자의 이름을 얻는다.

```kotlin
routing {
    authenticate("auth-basic") {
        get("/") {
            call.respondText("Hello, ${call.principal<UserIdPrincipal>()?.name}!")
        }
    }
}
```

세션 인증을 사용하면, principle은 세션 데이터를 저장하는 data class가 된다. 그래서 `call.principal`에 data class를 전달해야 한다.

```kotlin
authenticate("auth-session") {
    get("/hello") {
        val userSession = call.principal<UserSession>()
    }
}
```

## References

* [Authentication and authorization | Ktor](https://ktor.io/docs/authentication.html)
* [Basic authentication](https://www.notion.so/Basic-authentication-f91cfed5f85e427c9f3e4fb8e53521af)
* [Digest authentication](https://www.notion.so/Digest-authentication-04d45153f6954beb88a2675508e0cfe1)
* [Form-based authentication](https://www.notion.so/Form-based-authentication-4d6d736e5420498c979567dbe3daaba4)
* [LDAP](https://www.notion.so/LDAP-9c4dc9eb87b4471cab58139f8615dc59)
* [JSON Web Tokens](https://www.notion.so/JSON-Web-Tokens-7fbb9d392434404ea54d3a76c1c97ad8)
* [OAuth](https://www.notion.so/OAuth-98bc859c470e47b68777375b8f9f07f4)
* [Session authentication](https://www.notion.so/Session-authentication-1aa139e87935492b85b73203578219d0)