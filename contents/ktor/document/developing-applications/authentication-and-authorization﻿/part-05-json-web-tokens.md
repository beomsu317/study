# JSON Web Tokens

JSON Web Token은 JSON 객체를 이용해 클라이언트, 서버 간 안전한 전송을 정의하는 open standard이다. 이 정보는 shared secret(`HS256` 알고리즘) 또는
public/private 키 쌍(예: `RS256`)을 통해 검증되고 신뢰할 수 있다.

Ktor는 Bearer 스키마를 사용해 `Authorization` 헤더에 전달된 JWT를 처리하고 다음을 수행할 수 있다.

- JSON web token의 시그니처를 검증
- JWT 페이로드에 대한 추가 검증을 수행한다.

# **Add dependencies**

`JWT` 인증을 활성화하기 위해, `ktor-auth`와 `ktor-auth-jwt` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
implementation("io.ktor:ktor-auth-jwt:$ktor_version")
```

# **JWT authorization flow**

JWT 인증 flow는 다음과 같다.

1. 클라이언트는 크레덴셜이 포함된 `POST` 요청을 특정 인증 route(예제의 경우 `/login`)로 생성한다.

```
POST http://localhost:8080/login
Content-Type: application/json

{
  "username": "jetbrains",
  "password": "foobar"
}
```

2. 크레덴셜이 유효한 경우 서버는 JSON web token을 생성하고 지정된 알고리즘으로 사인한다. 예를 들어, 지정된 shared secret이 `HS256`이거나, public/private 키
   쌍이 `RS256` 일 수 있다.
3. 서버는 생성된 JWT를 클라이언트에 전달한다.
4. 클라이언트는 `Bearer` 스키마를 이용해 `Authorization` 헤더에 전달된 JSON web token을 사용해 보호된 리소스에 요청을 생성할 수 있다.
5. 서버는 요청을 받고 다음 검증을 수행한다.
    - 토큰의 시그니처를 검증한다. 검증 방식은 token에 사인한 알고리즘에 의존되어 있다.
    - JWT 페이로드에 대한 추가 검증을 수행한다.
6. 검증 후, 서버는 보호된 리소스의 콘텐츠를 응답한다.

# **Install JWT**

`jwt` 인증 제공자를 설치하기 위해 `install` 블록 안에서 `jwt` 함수를 호출한다.

```kotlin
install(Authentication) {
    jwt {
        // Configure jwt authentication
    }
}
```

선택적으로 제공자 이름을 지정할 수 있다.

# **Configure JWT**

이 섹션에서 2가지 방법을 통해 토큰을 검증하는 것을 알아본다.

- `HS256` shared secret.
- `RS256` public/private key pair.

## **Step 1: Configure JWT settings**

JWT 관련 설정을 위해, 커스텀 `jwt` 그룹을 `application.conf` 설정 파일에 생성해야 한다. 위는 `HS256`, 아래는 `RS256` 방식을 의미한다.

```kotlin
jwt {
    secret = "secret"
    issuer = "http://0.0.0.0:8080/"
    audience = "http://0.0.0.0:8080/hello"
    realm = "Access to 'hello'"
}
```

```kotlin
jwt {
    privateKey =
        "MIIBVQIBADANBgkqhkiG9w0BAQEFAASCAT8wggE7AgEAAkEAtfJaLrzXILUg1U3N1KV8yJr92GHn5OtYZR7qWk1Mc4cy4JGjklYup7weMjBD9f3bBVoIsiUVX6xNcYIr0Ie0AQIDAQABAkEAg+FBquToDeYcAWBe1EaLVyC45HG60zwfG1S4S3IB+y4INz1FHuZppDjBh09jptQNd+kSMlG1LkAc/3znKTPJ7QIhANpyB0OfTK44lpH4ScJmCxjZV52mIrQcmnS3QzkxWQCDAiEA1Tn7qyoh+0rOO/9vJHP8U/beo51SiQMw0880a1UaiisCIQDNwY46EbhGeiLJR1cidr+JHl86rRwPDsolmeEF5AdzRQIgK3KXL3d0WSoS//K6iOkBX3KMRzaFXNnDl0U/XyeGMuUCIHaXv+n+Brz5BDnRbWS+2vkgIe9bUNlkiArpjWvX+2we"
    issuer = "http://0.0.0.0:8080/"
    audience = "http://0.0.0.0:8080/hello"
    realm = "Access to 'hello'"
}
```

위에 설정에 다음과 같은 방식으로 접근할 수 있다.

```kotlin
val secret = environment.config.property("jwt.secret").getString()
val issuer = environment.config.property("jwt.issuer").getString()
val audience = environment.config.property("jwt.audience").getString()
val myRealm = environment.config.property("jwt.realm").getString()
```

```kotlin
val privateKeyString = environment.config.property("jwt.privateKey").getString()
val issuer = environment.config.property("jwt.issuer").getString()
val audience = environment.config.property("jwt.audience").getString()
val myRealm = environment.config.property("jwt.realm").getString()
```

## **Step 2: Generate a token**

JSON web token을 생성하기
위해 [JWTCreator.Builder](https://javadoc.io/doc/com.auth0/java-jwt/latest/com/auth0/jwt/JWTCreator.Builder.html)를 사용한다.

```kotlin
post("/login") {
    val user = call.receive<User>()
    // Check username and password
    // ...
    val token = JWT.create()
        .withAudience(audience)
        .withIssuer(issuer)
        .withClaim("username", user.username)
        .withExpiresAt(Date(System.currentTimeMillis() + 60000))
        .sign(Algorithm.HMAC256(secret))
    call.respond(hashMapOf("token" to token))
}
```

```kotlin
post("/login") {
    val user = call.receive<User>()
    // Check username and password
    // ...
    val publicKey = jwkProvider.get("6f8856ed-9189-488f-9011-0ff4b6c08edc").publicKey
    val keySpecPKCS8 = PKCS8EncodedKeySpec(Base64.getDecoder().decode(privateKeyString))
    val privateKey = KeyFactory.getInstance("RSA").generatePrivate(keySpecPKCS8)
    val token = JWT.create()
        .withAudience(audience)
        .withIssuer(issuer)
        .withClaim("username", user.username)
        .withExpiresAt(Date(System.currentTimeMillis() + 60000))
        .sign(Algorithm.RSA256(publicKey as RSAPublicKey, privateKey as RSAPrivateKey))
    call.respond(hashMapOf("token" to token))
}
```

1. `post("/login")`은 `POST` 요청을 수신하기 위한 인증 route를 정의한다.
2. `call.receive<User>()`는 JSON 객체로 전송된 유저 크레덴셜을 수신하고 `User` 클래스 객체로 변환한다.
3. `JWT.create()`는 지정된 JWT 설정으로 토큰을 생성하고, 수신된 username으로 custom claim을 추가하고, 지정된 알고리즘으로 토큰에 서명한다.
    - `HS256`은 shared secret이 토큰 서명에 사용된다.
    - `RS256`은 public/private 키 쌍이 사용된다.
4. `call.respond`는 클라이언트에 JSON 객체로 토큰을 전송한다.

## **Step 3: Configure realm**

`realm` 속성을 사용하면 보호된 route에 접근할 때 `WWW-Authenticate` 헤더에 전달될 realm을 설정할 수 있다.

```kotlin
val myRealm = environment.config.property("jwt.realm").getString()
install(Authentication) {
    jwt("auth-jwt") {
        realm = myRealm
    }
}
```

## **Step 4: Configure a token verifier**

[verifier](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-authentication-provider/-configuration/verifier.html)
함수는 토큰 포맷과 서명을 검증한다.

- `HS256`은 토큰을 검증하기
  위해 [JWTVerifier](https://www.javadoc.io/doc/com.auth0/java-jwt/latest/com/auth0/jwt/JWTVerifier.html) 인스턴스를 전달해야 한다.
- `RS256`은 토큰을 검증하기 위해 사용되는 public key에 접근하기 위한 JWKS 엔드포인트로
  지정된 [JwkProvider](https://www.javadoc.io/doc/com.auth0/jwks-rsa/latest/com/auth0/jwk/JwkProvider.html)를 전달해야 한다. 우리의
  경우 issuer는 `http://0.0.0.0:8080` 이므로, JWKS 엔드포인트 주소는 `http://0.0.0.0:8080/.well-known/jwks.json`이다.

```kotlin
val secret = environment.config.property("jwt.secret").getString()
val issuer = environment.config.property("jwt.issuer").getString()
val audience = environment.config.property("jwt.audience").getString()
val myRealm = environment.config.property("jwt.realm").getString()
install(Authentication) {
    jwt("auth-jwt") {
        realm = myRealm
        verifier(
            JWT
                .require(Algorithm.HMAC256(secret))
                .withAudience(audience)
                .withIssuer(issuer)
                .build()
        )
    }
}
```

```kotlin
val issuer = environment.config.property("jwt.issuer").getString()
val audience = environment.config.property("jwt.audience").getString()
val myRealm = environment.config.property("jwt.realm").getString()
val jwkProvider = JwkProviderBuilder(issuer)
    .cached(10, 24, TimeUnit.HOURS)
    .rateLimited(10, 1, TimeUnit.MINUTES)
    .build()
install(Authentication) {
    jwt("auth-jwt") {
        realm = myRealm
        verifier(jwkProvider, issuer) {
            acceptLeeway(3)
        }
    }
}
```

## **Step 5: Validate JWT payload**

[validate](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-authentication-provider/-configuration/validate.html)
함수는 JWT 페이로드에 대한 추가적인 검증을 수행한다.

1. [JWTCredential](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-credential/index.html)
   객체로 나타내고 JWT 페이로드를 포함하는 `credential` 파라미터를 체크한다. 예제에선 커스텀 `username` claim의 값이 체크된다.
2. 성공적으로 인증된
   경우 [JWTPrincipal](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-principal/index.html)
   이 반환된다. 인증 실패의 경우 `null`을 반환한다.

```kotlin
install(Authentication) {
    jwt("auth-jwt") {
        validate { credential ->
            if (credential.payload.getClaim("username").asString() != "") {
                JWTPrincipal(credential.payload)
            } else {
                null
            }
        }
    }
}
```

## **Step 6: Define authorization scope**

`jwt` 제공자 설정 후, `authenticate` 함수를 통해 다양한 리소스에 대한 권한을 부여할 수
있다. [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를 사용해
인증된 [JWTPrincipal](https://api.ktor.io/ktor-features/ktor-auth-jwt/ktor-auth-jwt/io.ktor.auth.jwt/-j-w-t-principal/index.html)
과, JWT 페이로드를 얻을 수 있다. 다음 예제의 경우 커스텀 `username` claim과 토큰 만료 시간을 얻는다.

```kotlin
routing {
    authenticate("auth-jwt") {
        get("/hello") {
            val principal = call.principal<JWTPrincipal>()
            val username = principal!!.payload.getClaim("username").asString()
            val expiresAt = principal.expiresAt?.time?.minus(System.currentTimeMillis())
            call.respondText("Hello, $username! Token is expired at $expiresAt ms.")
        }
    }
}
```

## References

* [JSON Web Tokens | Ktor](https://ktor.io/docs/jwt.html)