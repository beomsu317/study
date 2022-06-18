# Basic authentication

Basic 인증 스키마는 접근 제어 및 인증을 위한 HTTP 프레임워크의 일부분이다. 이 스키마에서 유저 크레덴셜은 Base64를 사용하여 인코딩된 username/pasword 쌍으로 전송된다.

Ktor는 로그인 유저와 특정 route를 보호하는데 basic 인증을 사용할 수 있다.

> basic 인증은 username과 password를 평문으로 전송하기 때문에, HTTPS/TLS를 사용해 보호해야 한다.

# **Add dependencies**

`basic` 인증을 활성화하기 위해 `ktor-auth` 아티팩트를 추가해야 한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

# **Basic authentication flow**

basic 인증 flow는 다음과 같다.

1. 클라이언트는 `Authorization` 헤더 없이 특정 route에 요청을 생성한다.
2. 서버는 `401`(Unauthorized) 상태와 `WWW-Authenticate` 응답 헤더를 사용해 basic 인증 스키마가 이 route에 제공된다는 것을 응답한다.
   일반적인 `WWW-Authenticate` 헤더는 다음과 같다.

```
WWW-Authenticate: Basic realm="Access to the '/' path", charset="UTF-8"
```

Ktor에서 basic 인증 제공자를 구성할 때 해당 속성을 사용하여 realm 및 charset을 지정할 수 있다.

3. 보통 클라이언트에선 로그인 다이얼로그가 보여진다. 클라이언트가 Base64를 사용한 username과 password 쌍을 포함한 `Authorization` 헤더를 사용해 요청한다.

```
Authorization: Basic amV0YnJhaW5zOmZvb2Jhcg
```

4. 서버는 클라이언트로부터 전달된 크레덴셜을 검증하고 요청된 콘텐츠로 응답한다.

# **Install basic authentication**

`basic` 인증을 설치하기 위해 `install` 블록에서 `basic` 함수를 호출한다.

```kotlin
install(Authentication) {
    basic {
        // Configure basic authentication
    }
}
```

선택적으로 특정 rotue에 대한 인증에 사용하기 위해 제공자 이름을 지정할 수 있다.

# **Configure basic authentication**

Ktor에서 다른 인증 제공자를 구성하는 방법에 대한 일반적인 아이디어를
얻으려면 [Configure Authentication](https://ktor.io/docs/authentication.html#configure)
을 참조해라. 이 섹션에서 `basic` 인증 제공자의 구성에 대해 알아본다.

## **Step 1: Configure a basic provider**

`basic` 인증
제공자는 [BasicAuthenticationProvider.Configuration](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-basic-authentication-provider/-configuration/index.html)
를 통해 설정을 노출한다. 다음 예제에서 설정이 지정된다.

- `realm` 속성은 `WWW-Authenticate` 헤더에 전달할 영역을 설정한다.
- `validate` 함수는 username과 password를 검증한다.

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

`validate` 함수는 `UserPasswordCredential`을 체크하고 인증이 성공한 경우 `UserIdPrincipal`, 실패한 경우 `null`을 반환한다.

> 메모리 테이블에 저장된 username과 password의 해시를 통해
> 검증하려면 [UserHashedTableAuth](https://ktor.io/docs/basic.html#validate-user-hash)를 사용하면 된다.
>

## **Step 2: Define authorization scope**

`basic` 제공자 구성 후, `authenticate` 함수를 사용해 다양한 리소스에 대한 권한을 정의할 수 있다. 인증이 성공적으로 이루어진 경우, route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)을
얻을 수 있고, 인증된 사용자 이름을 얻을 수 있다.

```kotlin
routing {
    authenticate("auth-basic") {
        get("/") {
            call.respondText("Hello, ${call.principal<UserIdPrincipal>()?.name}!")
        }
    }
}
```

# **Validate with UserHashedTableAuth**

Ktor는 메모리에 저장된 유저 정보를 사용하는 [UserHashedTableAuth](https://ktor.io/docs/basic.html#validate-user-hash)를 통해 유저를 검증할 수 있다.
이를 통해 데이터 소스가 유출된 경우 유저 password를 유출되지 않도록 할 수 있다.

유저 검증을 위해 `UserHashedTableAuth`를 사용하는 경우 다음 단계를 따라라.

1. [getDigestFunction](https://api.ktor.io/ktor-utils/ktor-utils/io.ktor.util/get-digest-function.html) 함수를 사용해 지정된 알고리즘
   및 salt 제공자로 digest 함수를 생성한다.

```kotlin
val digestFunction = getDigestFunction("SHA-256") { "ktor${it.length}" }
```

2. `UserHashedTableAuth` 인스턴스를 초기화하고 다음 속성을 지정한다.

-
- username과 해시된 password를 `table` 속성을 통해 제공한다.
- `digester` 속성에 digest 함수를 할당한다.

```kotlin
val hashedUserTable = UserHashedTableAuth(
    table = mapOf(
        "jetbrains" to digestFunction("foobar"),
        "admin" to digestFunction("password")
    ),
    digester = digestFunction
)
```

3. `validate`
   함수에서 [UserHashedTableAuth.authenticate](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-hashed-table-auth/authenticate.html)
   함수를 호출해 유저를 인증하고 검증된 경우 `UserIdPrincipal` 인스턴스를 반환한다.

```kotlin
install(Authentication) {
    basic("auth-basic-hashed") {
        realm = "Access to the '/' path"
        validate { credentials ->
            hashedUserTable.authenticate(credentials)
        }
    }
}
```

## References

* [Basic authentication | Ktor](https://ktor.io/docs/basic.html)