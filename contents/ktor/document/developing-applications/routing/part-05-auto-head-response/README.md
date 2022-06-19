# AutoHeadResponse

`AutoHeadResponse` 플러그인은자동으로 `GET`으로 정의된 모든 route에 `HEAD` 응답을 하는 기능을 제공한다. 실제 콘텐츠를 가져오기 전 클라이언트에서 어떻게든 응답을 처리해야 하는
경우 `AutoHeadResponse`를 사용해 별도의 head 핸들러를 생성하는 것을 방지할 수 있다. 예를
들어, [respondFile](https://ktor.io/docs/responses.html#file) 함수를 호출하면 `Content-Length`와 `Content-Type` 헤더가 응답에 자동으로 추가되며,
파일을 다운로드 하기 전 클라이언트에서 이 정보를 얻을 수 있다.

## Add dependencies

`AutoHeadResponse`를 사용하기 위해 `ktor-server-auto-head-response` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-auto-head-response:$ktor_version")
```

## **Usage**

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

## Options

`AutoHeadResponse`는 추가적인 설정 옵션을 제공하지 않는다.

## References

* [AutoHeadResponse | Ktor](https://ktor.io/docs/autoheadresponse.html)