# LDAP

LDAP는 유저에 대한 정보를 저장할 수 있는 다양한 디렉토리 서비스와 함께 동작하는 프로토콜이다. Ktor를 사용해 `basic`, `digest`, `form-based`로 LDAP 유저를 인증할 수 있다.

## **Add dependencies**

`LDAP` 인증을 활성화하기 위해, `ktor-auth`와 `ktor-auth-ldap` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-auth:$ktor_version")
implementation("io.ktor:ktor-auth-ldap:$ktor_version")
```

## **Configure LDAP**

### **Step 1: Choose an authentication provider**

LDAP 유저를 인증하기 위해 username과 password를 검증할 인증 제공자를 선택해야 한다. Ktor는 `basic`, `digest`, `form-based` 제공자를 사용할 수 있다. 예를
들어, `basic` 인증 제공자는 `install` 블록 안에서 `basic`을 호출한다.

```kotlin
install(Authentication) {
    basic {
        validate { credentials ->
            // Authenticate an LDAP user
        }
    }
}
```

`validate` 함수는 유저 크레덴셜을 검증한다.

### **Step 2: Authenticate an LDAP user**

LDAP 유저를 인증하기
위해, [ldapAuthenticate](https://api.ktor.io/ktor-features/ktor-auth-ldap/ktor-auth-ldap/io.ktor.auth.ldap/ldap-authenticate.html)
함수를 호출해야 한다. 이
함수는 [UserPasswordCredential](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-password-credential/index.html)
을 받고 지정된 LDAP 서버에 대해 검증한다.

```kotlin
install(Authentication) {
    basic("auth-ldap") {
        validate { credentials ->
            ldapAuthenticate(credentials, "ldap://0.0.0.0:389", "cn=%s,dc=ktor,dc=io")
        }
    }
}
```

`validate` 함수는 인증이 성공한
경우 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)을,
실패한 경우 `null`을 반환한다.

선택적으로 추가 검증을 추가할 수 있다.

```kotlin
install(Authentication) {
    basic("auth-ldap") {
        validate { credentials ->
            ldapAuthenticate(credentials, "ldap://localhost:389", "cn=%s,dc=ktor,dc=io") {
                if (it.name == it.password) {
                    UserIdPrincipal(it.name)
                } else {
                    null
                }
            }
        }
    }
}
```

### **Step 3: Define authorization scope**

LDAP 설정 후 `authenticate` 함수를 통해 리소스에 대한 권한을 설정할 수 있다. 인증이 성공한 경우 route
핸들러에서 [call.principal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/principal.html) 함수를
통해 [UserIdPrincipal](https://api.ktor.io/ktor-features/ktor-auth/ktor-auth/io.ktor.auth/-user-id-principal/index.html)을
얻을 수 있다.

```kotlin
routing {
    authenticate("auth-ldap") {
        get("/") {
            call.respondText("Hello, ${call.principal<UserIdPrincipal>()?.name}!")
        }
    }
}
```

## References

* [LDAP | Ktor](https://ktor.io/docs/ldap.html#authenticate-route)
