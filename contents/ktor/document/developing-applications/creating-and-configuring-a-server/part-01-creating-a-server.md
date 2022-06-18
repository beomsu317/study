# Creating a server

Ktor 서버 애플리케이션을 실행하기 위해 우선 서버를 만들어야 한다. 서버 구성은 다르게 설정될 수 있다. 서버 엔진(Netty, Jetty, etc.), host, port 기타 등등. Ktor 생성 및
실행하는데 2가지 주요한 부분이 있다.

- `embeddedServer` 함수는 간단한 방법으로 서버 파라미터를 구성하고, 애플리케이션을 빠르게 실행한다.
- `EngineMain`은 서버의 구성을 더 유연하게 제공한다. **application.conf** 파일에서 서버 파라미터를 지정할 수 있고, 애플리케이션 재컴파일 없이 구성을 변경할 수 있다. 또한 커맨드 라인
  인자로 서버 파라미터를 전달할 수 있다.

# **embeddedServer**

`embeddedServer` 함수는 간단한 방법으로 서버 파라미터를 구성하고, 애플리케이션을 빠르게 실행할 수 있다. 다음 코드를 보면, 엔진을 설정하고, 포트를 파라미터로 전달한다. `Netty` 엔진을
사용하고 `8080` 포트로 리스닝한다.

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

# **EngineMain**

`EngineMain`은 선택된 엔진으로 실행되고, 외부의 `application.conf` 파일에 지정된 설정들을 로드한다. 이 파일은 다양한 서버 파라미터를 포

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module(testing: Boolean = false) {
    routing {
        get("/") {
            call.respondText("Hello, world!")
        }
    }
}
```

```
ktor {
    deployment {
        port = 8080
    }
    application {
        modules = [ com.example.ApplicationKt.module ]
    }
}
```

## References

* [Creating a server | Ktor](https://ktor.io/docs/create-server.html)