# Digest authentication

Digest 인증 스키마는 HTTP 프레임워크 중 일부이며, 접근 제어 및 인증에 사용된다. 해시 함수는 네트워크를 통해 전송하기 전 username과 password에 적용된다.

Ktor는 digest 인증을 특정 route의 유저 로그인에 사용할 수 있다.

# **Add dependencies**

`digest` 인증을 활성화하기 위해 `ktor-auth` 아티팩트를 포함해야 한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

# **Digest authentication flow**

digest 인증 flow는 다음과 같다.

1. 클라이언트가 `Authorization` 헤더 없이 서버의 route에 요청한다.
2. 서버는 `401`(Unauthorized) 상태와 digest 인증 정보 제공을 위해 `WWW-Authenticate` 응답 헤더를 반환한다. 일반적으로 `WWW-Authenticate` 헤더는 다음과 같다.

    ```
    WWW-Authenticate: Digest
            realm="Access to the '/' path",
            nonce="e4549c0548886bc2",
            algorithm="MD5"
    ```

   Ktor에서 digest 인증 제공자를 구성할 때 realm과 nonce 값을 생성하는 방법을 지정할 수 있다.

3. 보통 클라이언트는 유저 크레덴셜을 입력할 수 있는 다이얼로그가 보여진다. 그런 다음 클라이언트는 `Authorization` 헤더와 함께 요청을 생성한다.

    ```
    Authorization: Digest username="jetbrains",
            realm="Access to the '/' path",
            nonce="e4549c0548886bc2",
            uri="/",
            algorithm=MD5,
            response="6299988bb4f05c0d8ad44295873858cf"
    ```

   `response` 값은 다음과 같다.

    1. `HA1 = MD5(username:realm:password)`
    2. `HA2 = MD5(method:digestURI)`
    3. `response = MD5(HA1:nonce:HA2)`
4. 서버는 클라이언트에서 전송된 크레덴셜을 검증하고 요청된 콘텐츠로 응답한다.

# **Install digest authentication**

`digest` 인증 제공자를 설치하기 위해 `digest` 함수를 `install` 블록에서 호출한다.

```kotlin
install(Authentication) {
    digest {
        // Configure digest authentication
    }
}
```

선택적으로 제공자 이름을 지정할 수 있다.

# **Configure digest authentication**

`digest` 인증 제공자의 설정을 알아보자.

## **Step 1: Provide a user table with digests**

`digest` 인증 제공자는 유저 크레덴셜을 digest message의 일부인 `HA1`을 사용해 검증한다. 그래서 username과 일치하는 `HA1` 해시를 포함하는 유저 테이블을 제공할 수 있다. 다음
예제에서 `getMd5Digest`
함수가 `HA1` 해시를 생성하기 위해 사용된다.

```kotlin
fun getMd5Digest(str: String): ByteArray = MessageDigest.getInstance("MD5").digest(str.toByteArray(UTF_8))

val myRealm = "Access to the '/' path"
val userTable: Map<String, ByteArray> = mapOf(
    "jetbrains" to getMd5Digest("jetbrains:$myRealm:foobar"),
    "admin" to getMd5Digest("admin:$myRealm:password")
)
```

## **Step 2: Configure a digest provider**

`digest` 인증
제공자는 [DigestAuthenticationProvider.Configuration](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-digest-authentication-provider/-configuration/index.html)
클래스를 통해 설정을 노출한다. 예제에서 다음 설정이 지정된다.

- `WWW-Authenticate` 헤더로 전달되는 realm이 `realm` 속성으로 설정된다.
- `digestProvider` 함수는 지정된 사용자 이름에 대한 digest의 `HA1` 부분을 가져온다.

```kotlin
fun Application.main() {
    install(Authentication) {
        digest("auth-digest") {
            realm = myRealm
            digestProvider { userName, realm ->
                userTable[userName]
            }
        }
    }
}
```

[nonceManager](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-digest-authentication-provider/-configuration/nonce-manager.html)
속성을 사용해 nonce 값을 어떻게 생성할지 지정할 수 있다.

## **Step 3: Define authorization scope**

`digest` 제공자를 구성한 후 `authenticate` 함수를 통해 리소스에 대한 권한을 설정할 수 있다. 인증이 성공한 경우 route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)을
얻을 수 있다.

```kotlin
routing {
    authenticate("auth-digest") {
        get("/") {
            call.respondText("Hello, ${call.principal<UserIdPrincipal>()?.name}!")
        }
    }
}
```

## References

* [Digest authentication | Ktor](https://ktor.io/docs/digest.html)