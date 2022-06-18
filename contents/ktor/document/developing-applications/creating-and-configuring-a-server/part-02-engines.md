# Engines

Ktor 서버 애플리케이션을 실행하기 위해, 서버를 구성해야 한다. 서버 설정은 다른 설정이 포함될 수 있다. 서버 엔진, 다양한 엔진 지정 옵션, host, port 등. 다음 엔진들이 지원된다.

- Netty
- Jetty
- Tomcat
- CIO (Coroutine-based I/O)

추가적으로 위 언급된 엔진들은, Ktor 테스팅을 위한 특별한 엔진 타입인 `TestEngine`을 제공한다.

# **Add dependencies**

원하는 엔진 설정 전, 해당하는 디펜던시를 `build.gradle` 또는 `pom.xml` 파일에 추가해야 한다.

- `ktor-server-netty`
- `ktor-server-jetty`
- `ktor-server-tomcat`
- `ktor-server-cio`

다음 Netty 디펜던시를 추가하는 예제이다.

```kotlin
implementation("io.ktor:ktor-server-netty:$ktor_version")
```

# **Choose how to create a server**

Ktor 서버 애플리케이션은`embeddedServer`를 사용해 코드 상에 서버 파라미터를 전달하거나 `EngineMain`을 사용해 `application.conf` 파일에서 설정을 불러오는 방법, 이 2가지
방법으로 생성하고 실행할 수 있다.

### **embeddedServer**

`embeddedServer` 함수는 지정된 타입의 엔진을 생성하는 데 사용하기 위한 engine factory를 허용한다. 아래 예제에서 Netty factory를 전달해 Netty 엔진으로 실행되게
했으며, `8080` 포트로 리스닝하도록 한다.

```kotlin
fun main() {
    embeddedServer(Netty, port = 80) {
        routing {
            get("/") {
                call.respondText("Hello, world!")
            }
        }
    }.start(wait = true)
}
```

### **EngineMain**

`EngineMain`은 서버를 실행할 엔진을 표현한다. 다음 엔진들을 사용한다.

- `io.ktor.server.netty.EngineMain`
- `io.ktor.server.jetty.EngineMain`
- `io.ktor.server.tomcat.EngineMain`
- `io.ktor.server.cio.EngineMain`

`EngineMain.main` 함수는 `application.conf` 파일의 설정을 불러오고, 선택된 엔진을 시작한다.

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

빌드 시스템 작업을 사용해 서버를 실행하려면 `EngineMain`을 메인 클래스로 구성해야 한다.

```kotlin
application {
    mainClassName = "io.ktor.server.netty.EngineMain"
}
```

# **Configure an engine**

이 섹션에선 다양한 엔진 옵션을 설저하는 것을 알아본다.

## **embeddedServer**

`embeddedServer` 함수는 `configure` 옵션 파라미터를 통해 엔진 옵션을 지정할 수 있다. 이 파라미터는 모든 엔진의 옵션을
포함하며, [ApplicationEngine.Configuration](https://api.ktor.io/ktor-server/ktor-server-host-common/ktor-server-host-common/io.ktor.server.engine/-application-engine/-configuration/index.html)
클래스로 노출된다.

```kotlin
fun main() {
    embeddedServer(Netty, port = 8080, configure = {
        connectionGroupSize = 2
        workerGroupSize = 5
        callGroupSize = 10
    }) {
        // ...
    }.start(wait = true)
}
```

이러한 옵션 외에도 추가 엔진 속성을 구성할 수 있다.

### **Netty**

Netty
옵션은 [NettyApplicationEngine.Configuration](https://api.ktor.io/ktor-server/ktor-server-netty/ktor-server-netty/io.ktor.server.netty/-netty-application-engine/-configuration/index.html)
클래스로 노출된다.

```kotlin
embeddedServer(Netty, configure = {
    requestQueueLimit = 16
    shareWorkGroup = false
    configureBootstrap = {
        // ...
    }
    responseWriteTimeoutSeconds = 10
}) {
    // ...
}.start(true)
```

### Jetty

Jetty를 사용하는 경우 `Server` 인스턴스를 제공하는 `configureServer` 블록에서 구성할 수 있다.

```kotlin
embeddedServer(Jetty, configure = {
    configureServer = { // this: Server ->
        // ...
    }
}) {
    // ...
}.start(true)
```

### **CIO**

CIO
옵션은 [CIOApplicationEngine.Configuration](https://api.ktor.io/ktor-server/ktor-server-cio/ktor-server-cio/io.ktor.server.cio/-c-i-o-application-engine/-configuration/index.html)
클래스로 노출된다.

```kotlin
embeddedServer(CIO, configure = {
    connectionIdleTimeoutSeconds = 45
}) {
    // ...
}.start(true)
```

### Tomcat

Tomcat을 사용하면 `Tomcat` 인스턴스를 제공한 `configureTomcat` 속성을 사용해 구성한다.

```kotlin
embeddedServer(Tomcat, configure = {
    configureTomcat = { // this: Tomcat ->
        // ...
    }
}) {
    // ...
}.start(true)
```

## **EngineMain**

`EngineMain`을 사용하면 `ktor.deployment` 그룹 내 `application.conf` 파일에서 모든 엔진에 대한 공통적인 옵션을 지정할 수 있다.

```
ktor {
    deployment {
        connectionGroupSize = 2
        workerGroupSize = 5
        callGroupSize = 10
    }
}
```

## References

* [Engines | Ktor](https://ktor.io/docs/engines.html#EngineMain)