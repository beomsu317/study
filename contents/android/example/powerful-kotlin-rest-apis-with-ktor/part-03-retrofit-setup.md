# Retrofit setup

이번엔 Retrofit에 대한 설정을 수행할 것이다.서버와 동일한 requests와 responses를 가져야 한다.

`data/remote/requests` 패키지를 생성해주고 여기에 request에 대한 data class들을 작성해준다. `requests` 패키지 하위에 `AccountRequest`
, `AddOwnerRequest`, `DeleteNoteRequest` data class를 작성해준다.

```kotlin
// register, login에 사용
data class AccountRequest(
        val email: String,
        val password: String
)
```

```kotlin
data class AddOwnerRequest(
        val owner: String,
        val noteID: String
)
```

```kotlin
data class DeleteNoteRequest(
        val id: String
)
```

그리고 `data/remote/response` 패키지를 생성하고 응답에 대한 data class들을 여기에 작성한다. 서버에서 `SimpleResponse`로 응답하므로 이에 대한 data class를 작성해준다.

```kotlin
data class SimpleResponse(
        val successful: Boolean,
        val message: String
)
```

`/data/remote` 패키지에 Retrofit `NoteApi` 인터페이스를 생성한다.

```kotlin
interface NoteApi {

    @POST("/register")
    suspend fun register(
            @Body registerRequest: AccountRequest
    ): Response<SimpleResponse>

    @POST("/login")
    suspend fun login(
            @Body loginRequest: AccountRequest
    ): Response<SimpleResponse>

    @POST("/addNote")
    suspend fun addNote(
            @Body note: Note
    ): Response<ResponseBody> // 아무런 응답이 없는 경우 ResponseBody 사용

    @POST("/deleteNote")
    suspend fun deleteNote(
            @Body deleteNoteRequest: DeleteNoteRequest
    ): Response<ResponseBody>

    @POST("/addOwnerToNote")
    suspend fun addOwnerToNote(
            @Body addOwnerRequest: AddOwnerRequest
    ): Response<SimpleResponse>

    @GET("/getNotes")
    suspend fun getNotes(): Response<List<Note>>
}
```

서버의 인증 로직인 Basic Auth를 사용하기 위해 OkHttp3 Interceptor를 사용해 헤더에 Auth에 대한 정보를 추가해주어야 한다.

API `/login`, `/register`에 대해 Auth Intercept를 수행하지 않기 위해 `other` 패키지 생성 후 `Constants` object 파일 생성하고 `IGNORE_AUTH_URLS`을
작성한다.

```kotlin
object Constans {

    val IGNORE_AUTH_URLS = listOf("/login", "/register")
}
```

`data/remote` 패키지에 `Interceptor`를 상속하는 `BasicAuthInterceptor` 클래스를 생성 및 작성한다.

```kotlin
class BasicAuthInterceptor : Interceptor {

    var email: String? = null
    var password: String? = null

    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        if (request.url.encodedPassword in IGNORE_AUTH_URLS) {
            return chain.proceed(request)
        }
        // add auth to request header
        val authenticatedRequest = request.newBuilder()
                .header("Authorization", Credentials.basic(email ?: "", password ?: ""))
                .build()
        return chain.proceed(authenticatedRequest)
    }
}
```