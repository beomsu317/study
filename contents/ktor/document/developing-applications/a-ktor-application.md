# A Ktor application

Ktor는 다양한 서버와 클라이언트 사이드 애플리케이션으로 사용될 수 있다. 정적 또는 동적 페이지, HTTP 엔드포인트, RESTful 시스템, 마이크로서비스까지 모두 만들 수 있다.

이 섹션에서는 Ktor 서버 애플리케이션의 기본적인 부분들을 다룰 것이다.

# **The simplest Ktor application**

Ktor의 목표 중 하나는 단순함을 유지하는 것이다. *Hello World* 애플리케이션은 다음과 같다.

```kotlin
fun main() {
    embeddedServer(Netty, port = 8080) {
        routing {
            get("/") {
                call.respondText("Hello, world!")
            }
        }
    }.start(wait = true)
}
```

Netty 엔진을 사용하고, 포트를 8080으로 설정했다. `/`로 GET 요청하면 *Hello, world!* 텍스트를 출력한다.

## References

* [A Ktor application | Ktor](https://ktor.io/docs/a-ktor-application.html)