# AutoHeadResponse

`AutoHeadResponse` 플러그인은 자동으로 `GET`으로 정의된 모든 route에 `HEAD` 응답을 하는 기능을 제공한다.

# **Usage**

이 기능을 사용하기 위해 `AutoHeadResponse`을 설치해야 한다.

```kotlin
fun Application.main() {
    install(AutoHeadResponse)
    routing {
        get("/home") {
            call.respondText("This is a response to a GET, but HEAD also works")
        }
    }
}
```

위 예제에서 `/home` route는 `HEAD` 요청에 대한 응답을 수행한다.

이 플러그인을 사용하면 동일한 `GET` path에 대해 `HEAD` 정의가 무시된다는 점에 유의하는 것이 중요하다.

# Options

`AutoHeadResponse`는 추가적인 설정 옵션을 제공하지 않는다.

## References

* [AutoHeadResponse | Ktor](https://ktor.io/docs/autoheadresponse.html)