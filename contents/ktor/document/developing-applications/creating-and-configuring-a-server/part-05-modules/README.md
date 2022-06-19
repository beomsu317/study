# Modules

Ktor는 특정 module 내 route의 집합을 정의하여 module을 구성해 사용할 수 있다. `module`은 `Application` 클래스의 확장 함수이다. 다음 에제에서 `module1` 확장
함수는 `/moudle1` 경로로 GET 요청을 받도록 정의한다.

```kotlin
fun Application.module1() {
    routing {
        get("/module1") {
            call.respondText("Hello from 'module1'!")
        }
    }
}
```

애플리케이션에서 모듈을 로딩하는 것은 `embeddedServer` 함수를 사용하거나 `application.conf` 설정 파일을 사용하는 서버를 생성하는 방법에 의존한다.

## **embeddedServer**

일반적으로 `embeddedServer` 함수는 람다 인자로써 암시적 모듈을
허용한다. [Configuration in code](https://ktor.io/docs/create-server.html#embedded-server) 섹션에서 예제를 볼 수 있다. 또한 앱 로직을 분리된 모듈로
추출할 수 있고 이 모듈 함수들을 `embeddedServer` 블록에서 호출할 수 있다.

```kotlin
package com.example

import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*

fun main() {
    embeddedServer(Netty, port = 8080) {
        module1()
        module2()
    }.start(wait = true)
}

fun Application.module1() {
    routing {
        get("/module1") {
            call.respondText("Hello from 'module1'!")
        }
    }
}

fun Application.module2() {
    routing {
        get("/module2") {
            call.respondText("Hello from 'module2'!")
        }
    }
}
```

## **HOCON file**

`application.conf` 파일로 서버를 설정하는 경우, `ktor.application.modules` 속성을 사용해 로드할 모듈을 지정한다.

3개의 모듈이 `com.example` 패키지와 `org.sample` 패키지 2개의 패키지에 정의되어 있다고 가정하자.

```kotlin
package com.example

import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module1() {
    routing {
        get("/module1") {
            call.respondText("Hello from 'module1'!")
        }
    }
}

fun Application.module2() {
    routing {
        get("/module2") {
            call.respondText("Hello from 'module2'!")
        }
    }
}
```

```kotlin
package org.sample

import io.ktor.server.application.*
import io.ktor.server.response.*
import io.ktor.server.routing.*

fun Application.module3() {
    routing {
        get("/module3") {
            call.respondText("Hello from 'module3'!")
        }
    }
}
```

설정 파일에서 이 모듈들을 참조하려면, 완전한 이름을 `,`로 분리하여 제공해야 한다. 완전한 모듈 이름은 클래스의 완전한 이름과 확장 함수 이름이 포함된다.

```
ktor {
    application {
        modules = [ com.example.ApplicationKt.module1,
                    com.example.ApplicationKt.module2,
                    org.sample.SampleKt.module3 ]
    }
}
```

## References

* [Modules | Ktor](https://ktor.io/docs/modules.html)