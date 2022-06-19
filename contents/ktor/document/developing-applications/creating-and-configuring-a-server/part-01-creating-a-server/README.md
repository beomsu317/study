# Creating a server

Ktor 앱을 생성하기 전 애플리케이션 배포 방법을 고려해야 한다.

* 독립형 패키지로
    * 이 경우 네트워크 요청을 처리하는데 사용되는 앱 엔진은 앱의 일부여야 한다. 앱이 엔진 설정, 연결 및 SSL 옵션을 제어한다.
* 서블릿으로
    * 이 경우 앱의 생명주기와 연결 설정을 제어하는 서블릿 컨테이너 내부에서 Ktor 앱을 배포할 수 있다(Tomcat, Jetty 같은).

## Embedded server

Ktor 서버 앱을 독립형 패키지로 전달하기 위해 먼저 서버를 생성해야 한다. 서버 구성은 다른 설정이 포함될 수 있다(서버 엔진, 다양한 엔진별 옵션, 호스트, 포트 등). 서버를 생성하고 실행하기 위한 2가지
접근방식이 있다.

* `embeddedServer` 함수는 간단한 방법으로 서버 파라미터를 구성하고, 애플리케이션을 빠르게 실행할 수 있다.
* `EngineMain`은 더 유연하게 서버 설정을 제공할 수 있다. 서버 파라미터를 파일로 지정하여 재컴파일 없이 구성을 변경할 수 있다. 또한 커맨드 라인에 서버 파라미터를 전달하여 설정을 오버라이드 할 수
  있다.

### Configuration in code

`embeddedServer` 함수는 간단한 방법으로 서버 파라미터를 구성하고, 애플리케이션을 빠르게 실행할 수 있다. 다음 코드에서 엔진과 포트를 파라미터로 받아 서버를 시작한다. `Netty` 엔진을
사용하고, `8080` 포트에서 listen 한다.

```kotlin
package com.example

import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

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

### Configuration in file

`EngineMain`은 선택된 엔진으로 서버를 시작하고 외부 `application.conf` 파일에 지정된 앱 모듈을 로드한다. 로드할 모듈 외 이 파일에는 다양한 서버 파라미터가 포함될 수 있다.

```kotlin
package com.example

import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {
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

## Servlet container

Ktor 앱은 Tomcat과 Jetty를 포함하는 서블릿 컨테이너에서 실행 및 배포될 수 있다. 서블릿 컨테이너에서 배포하기 위해 WAR archive를 생성해야 하며, WAR를 지원하는 서버 또는 클라우드 서비스에
배포해야 한다.

## References

* [Creating a server | Ktor](https://ktor.io/docs/create-server.html)