# OAuth

[OAuth](https://oauth.net/)는 접근 위임을 위한 open standard이다. OAuth는 구글, 페이스북, 트위터 등 외부 제공자를 사용해 유저를 인증하는데 사용할 수 있다.

`oauth` 제공자는 권한 부여 코드 flow를 지원한다. OAuth 파라미터를 한 곳에서 구성할 수 있으며, Ktor는 필요한 파라미터를 사용하여 지정된 인증 서버에 자동으로 요청한다.

## **Add dependencies**

`OAuth` 지원을 활성화하기 위해 `ktor-auth` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

## **OAuth authorization flow**

OAuth 인증 flow는 다음과 같다.

1. 유저가 로그인 페이지에 접근한다.
2. Ktor는 지정된 제공자에 대한 인증 페이지로 자동 리다이렉션하고 필요한 파라미터를 전달한다.
    - 선택된 API에 접근하기 위한 클라이언트 ID.
    - 인증 완료 후 보여질 콜백 또는 리다이렉션 URL.
    - Ktor에 필요한 3-party 리소스 스코프.
    - 액세스 토큰을 얻는 데 사용되는 부여 타입.
3. 인증 페이지는 Ktor 애플리케이션에 필요한 권한 수준과 함께 동의 화면을 보여준다. 이 퍼미션은 2번에 전달된 스코프에 의존한다.
4. 유저가 요청 권한을 승인한 경우, 권한 서버는 리다이렉션 URL로 리다이렉션하고 권한 코드를 전송한다.
5. Ktor는 지정된 액세스 토큰 URL에 대해 한 번 더 요청을 수행하고 다음 파라미터를 전달한다.
    - authorization code.
    - client ID and client secret.

   인증 서버는 액세스 토큰을 반환한다.

6. 클라이언트는 토큰을 사용해 선택한 제공자의 필수 서비스에 요청할 수 있다. 대부분의 경우 토큰은 `Authorization` 헤더의 `Bearer` 스키마로 전송된다.
7. 서비스는 토큰을 검증하고, 권한 부여를 위해 해당 스코프를 사용하며, 요청된 데이터를 반환한다.

## **Install OAuth**

`oauth` 인증 제공자를 설치하기 위해 `install` 블록
내 [oauth](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/oauth.html) 함수를 호출한다.

```kotlin
install(Authentication) {
    oauth {
        // Configure oauth authentication
    }
}
```

선택적으로 제공자 이름을 지정할 수 있다.

## **Configure OAuth**

이 섹션에선 구글을 사용해 유저를 인증하기 위한 `oauth` 제공자를 어떻게 설정하는지 알아본다.

> Google API에 접근하기 위해 구글 클라우드 콘솔의 크레덴셜이 필요하다. 크레덴셜은 `oauth` 제공자를 구성하는데 사용되는 클라이언트 ID와 클라이언트 secret가 포함된다.
>

### **Step 1: Create the HTTP client**

`oauth` 제공자를 구성하기 전 OAuth 서버로 요청하기 위해 `HttpClient`를 초기화해야 한다. `HttpClient` 디펜던시를 추가하고 구성하는
것을 [Client overview](https://ktor.io/docs/client.html)에서 확인할 수 있다.

```kotlin
val httpClient = HttpClient(CIO) {
    install(JsonFeature) {
        serializer = KotlinxSerializer()
    }
}
```

클라이언트는 `Json` 플러그인이 설치되어 있어야 한다. 이는 JSON 데이터를 deserialize 하기 위함이다.

### **Step 2: Configure the OAuth provider**

다음 코드는 `auth-oauth-google` 이름으로 `oauth` 제공자를 생성하고 구성하는 방법을 보여준다.

```kotlin
install(Authentication) {
    oauth("auth-oauth-google") {
        urlProvider = { "http://localhost:8080/callback" }
        providerLookup = {
            OAuthServerSettings.OAuth2ServerSettings(
                name = "google",
                authorizeUrl = "https://accounts.google.com/o/oauth2/auth",
                accessTokenUrl = "https://accounts.google.com/o/oauth2/token",
                requestMethod = HttpMethod.Post,
                clientId = System.getenv("GOOGLE_CLIENT_ID"),
                clientSecret = System.getenv("GOOGLE_CLIENT_SECRET"),
                defaultScopes = listOf("https://www.googleapis.com/auth/userinfo.profile")
            )
        }
        client = httpClient
    }
}
```

- `urlProvider`는 인증이 완료되면 열려질 리다이렉션 route를 지정한다.
- `providerLookup`은 필요한 제공자를 위해 OAuth 설정을 지정할 수 있다. 이 설정은 Ktor가 자동적으로 OAuth 서버에 요청을 생성할 수 있다.
- `client` 속성은 Ktor가 OAuth 서버에 요청을 생성하기 위한 `HttpClient`를 지정한다.

### **Step 3: Add a login route**

`oauth` 제공자 구성 후 `oauth` 제공자의 이름을 받는 `authenticate` 함수 내부에 보호된 login route를 생성해야 한다.

```kotlin
routing {
    authenticate("auth-oauth-google") {
        get("/login") {
            // Redirects to 'authorizeUrl' automatically
        }
    }
}
```

유저는 Ktor 애플리케이션에 필요한 권한 수준이 포함된 인증 페이지를 볼 수 있다. 이
권한은 [providerLookup](https://ktor.io/docs/oauth.html#configure-oauth-provider)에 지정된 `defaultScopes`에 의존한다.

### **Step 4: Add a redirect route**

login route 외에, 인증 완료 후 호출될 리다이렉션 route를 설정해야 한다. 이 route의
주소는 [urlProvider](https://ktor.io/docs/oauth.html#configure-oauth-provider) 속성을 사용해 지정한다.

route 안에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 [OAuthAccessTokenResponse](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-o-auth-access-token-response/index.html)
객체를 얻을 수 있다. `OAuthAccessTokenResponse`를 사용하면 OAuth 서버에서 반환된 토큰 및 기타 파라미터에 접근할 수 있다.

```kotlin
routing {
    authenticate("auth-oauth-google") {
        get("/login") {
            // Redirects to 'authorizeUrl' automatically
        }

        get("/callback") {
            val principal: OAuthAccessTokenResponse.OAuth2? = call.principal()
            call.sessions.set(UserSession(principal?.accessToken.toString()))
            call.respondRedirect("/hello")
        }
    }
}
```

샘플에서 토큰을 받은 후 다음 작업을 수행한다.

- 토큰은 다른 route 내에서 콘텐츠에 접근할 수 있는 cookie session에 저장된다.
- 유저는 Google API에 대한 요청이 이루어진 다음 route로 리다이렉션 된다.

### **Step 5: Make a request to API**

리다이렉션 route에서 토큰을 수신 및 세션으로 저장한 후 토큰을 사용해 외부 API를 요청할 수 있다. 다음
코드는 [HttpClient](https://ktor.io/docs/oauth.html#create-http-client)를 사용해 이러한 요청을 수행하고 `Authorization` 헤더에서 이 토큰을 전송하여
사용자 정보를 얻는 방법을 보여준다.

```kotlin
get("/hello") {
    val userSession: UserSession? = call.sessions.get<UserSession>()
    if (userSession != null) {
        val userInfo: UserInfo = httpClient.get("https://www.googleapis.com/oauth2/v2/userinfo") {
            headers {
                append(HttpHeaders.Authorization, "Bearer ${userSession.token}")
            }
        }
        call.respondText("Hello, ${userInfo.name}!")
    } else {
        call.respondRedirect("/")
    }
}
```

## References

* [OAuth | Ktor](https://ktor.io/docs/oauth.html)
* [ktor-documentation/codeSnippets/snippets/auth-oauth-google at main · ktorio/ktor-documentation](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/auth-oauth-google)