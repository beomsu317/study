# Form-based authentication

Form 기반 인증은 [web form](https://developer.mozilla.org/en-US/docs/Learn/Forms)을 사용해 크레덴셜 정보를 수집하고 유저를 인증한다.

> 전달되는 username, password는 평문이므로 HTTPS를 사용해야 한다.

## **Add dependencies**

`form` 인증을 활성화하기 위해 `ktor-auth` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
```

## **Form-based authentication flow**

from 기반 인증 flow는 다음과 같다.

1. 인증되지 않은 클라이언트가 특정 route로 요청을 생성한다.
2. 서버는 HTML 기반 web form으로 구성된 HTML 페이지를 반환하며 username과 password를 묻는 프롬프트를 띄운다.
3. 유저가 username과 password를 제출하면, 클라이언트는 web form 데이터(username과 password를 포함한)를 포함하는 요청을 생성한다.
    * Ktor에서 username과 password를 가져오는데 사용하는 파라미터 이름을 지정해야 한다.

```
POST http://localhost:8080/
Content-Type: application/x-www-form-urlencoded

username=jetbrains&password=foobar
```

4. 서버는 클라이언트로부터 전달된 크레덴셜을 검증하고 요청 콘텐츠에 응답한다.

## **Install form authentication**

`form` 인증 제공자를 설치하기 위해 [form](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/form.html)
함수를 `install` 블록에서 호출한다.

```kotlin
install(Authentication) {
    form {
        // Configure form authentication
    }
}
```

선택적으로 제공자 이름을 지정할 수 있다.

## **Configure form authentication**

### **Step 1: Configure a form provider**

`form` 인증
제공자는 [FormAuthenticationProvider/Configuration](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-form-authentication-provider/-configuration/index.html)
클래스에 설정을 노출한다. 예제에서 다음 설정이 지정된다.

- `userParamName`, `passwordParamName` 속성은 username과 password를 가져오는데 사용하는 파라미터를 지정한다.
- `validate` 함수는 username과 password를 검증한다.

```kotlin
install(Authentication) {
    form("auth-form") {
        userParamName = "username"
        passwordParamName = "password"
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

`validate` 함수는 `UserPasswordCredential`을 검증하고, 인증에 성공한 경우 `UserIdPrincipal`을 반환하고, 실패한 경우 `null`을 반환한다.

> [UserHashedTableAuth](https://ktor.io/docs/basic.html#validate-user-hash)을 사용해 메모리에 존재하는 테이블을 이용해 검증할 수 있다.

### **Step 2: Define authorization scope**

`form` 제공자를 구성한 후 `authenticate` 함수를 통해 리소스에 대한 권한을 설정할 수 있다. 인증이 성공한 경우 route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)을
얻을 수 있다.

```kotlin
routing {
    authenticate("auth-form") {
        post("/") {
            call.respondText("Hello, ${call.principal<UserIdPrincipal>()?.name}!")
        }
    }
}
```

## References

* [Form-based authentication | Ktor](https://ktor.io/docs/form.html)