# Configuration

Ktor는 host, port와 같이 다양한 서버 파라미터로 구성하는 것을 허용한다. 구성은 서버 생성에 사용한 방식 (`embeddedServer` 또는 `EngineMain`)에 의존한다.

- `embeddedServer`의 경우, 필요한 파라미터를 `embeddedServer` 생성자에 전달하여 코드에서 파라미터를 구성할 수 있다.
- `EngineMain`의 경우, Ktor는 HOCON 포맷으로 사용되는 외부 파일에서 구성을 로드한다. 이 방법을 사용하면 매우 유연하게 서버의 구성을 관리할 수 있다. 또한 커맨드라인 인자를 통해 서버
  파라미터를 전달할 수 있다.

# **embeddedServer**

`embeddedServer` 함수는 서버 엔진, host, port 등을 포함하여 서버 구성을 위한 다양한 파라미터를 허용한다. 이 섹션에서 다른 설정을 통해 실행되는 `embeddedServer` 예제를 보자.

## **Basic configuration**

다음 코드는 Netty 엔진과 8080 포트로 설정된 것을 보여준다.

```kotlin
fun main() {
    embeddedServer(Netty, port = 8080) {
        // ...
    }.start(wait = true)
}
```

## **Engine configuration**

다음 예제는, 선택된 엔진에 특정한 설정을 구성하기 위한 `configure` 파라미터를 추가했다.

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

## **Custom environment**

다음
예제에서 [ApplicationEngineEnvironment](https://api.ktor.io/ktor-server/ktor-server-host-common/ktor-server-host-common/io.ktor.server.engine/-application-engine-environment/index.html)
인터페이스를 통해 커스텀 환경의 서버를 실행하는지 보여준다.

```kotlin
fun main() {
    embeddedServer(Netty, environment = applicationEngineEnvironment {
        log = LoggerFactory.getLogger("ktor.application")
        config = HoconApplicationConfig(ConfigFactory.load())

        module {
            main()
        }

        connector {
            port = 8080
            host = "127.0.0.1"
        }
    }).start(true)
}
```

커스텀 환경을 HTTPS를 제공하도록 사용할 수
있다. [ssl-embedded-server](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/ssl-embedded-server)
에서 어떻게 하는지 보여준다.

# **HOCON file**

## Overview

서버 실행을 위해 `EngineMain`을 사용하면, Ktor는 `application.conf` HOCON 파일에서 설정을 로드한다. 이 파일은 `ktor.application.modules` 속성을 사용해 지정된
로드할 modules이 포함되어 있어야 한다.

```
ktor {
    application {
        modules = [ com.example.ApplicationKt.module ]
    }
}
```

이 경우 Ktor는 `Application.module` 함수를 호출한다.

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

port, host, SSL 설정 등 다양한 서버 설정을 할 수 있다. 몇개의 예제를 보자.

- `ktor.deployment.port` 속성을 사용해 리스닝 포트를 `8080`으로 설정했다.

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

- `8443` SSL 포트로 리스닝하며, 별도의 `security` 블럭에 SSL 설정을 지정한다.

```
ktor {
    deployment {
        port = 8080
        sslPort = 8443
    }
    application {
        modules = [ com.example.ApplicationKt.module ]
    }

    security {
        ssl {
            keyStore = keystore.jks
            keyAlias = sampleAlias
            keyStorePassword = foobar
            privateKeyPassword = foobar
        }
    }
}
```

- `application.conf` 파일에서 커스텀 JWT 저장을 위한 `jwt` 그룹을 포함한다.

```
jwt {
    secret = "secret"
    issuer = "http://0.0.0.0:8080/"
    audience = "http://0.0.0.0:8080/hello"
    realm = "Access to 'hello'"
}
```

## **Predefined properties**

다음은 설정 파일에서 내에서 사용 가능한 미리 지정되 설정이다.

- `ktor.deployment.host` : 호스트 주소.
    - Example: `0.0.0.0`
- `ktor.deployment.port` / `ktor.deployment.sslPort` : 리스닝 포트 / SSL 포트. SSL은 추가적인 옵션이 필요하다.
    - Example: `80` / `443`
- `ktor.deployment.watch` : 자동 새로고침에 사용되는 경로 지정.
- `ktor.deployment.rootPath` : 서블릿 컨텍스트 경로.
    - Example: `/`
- `ktor.deployment.shutdown.url` : shutdown URL. 이 옵션은 Shutdown URL 플러그인에 사용된다.
- 엔진을 위한 속성.

`ktor.deployment.sslPort` 설정하는 경우, 다음의 SSL 속성을 설정해주어야 한다.

- `ktor.security.ssl.keyStore` : SSL key store.
- `ktor.security.ssl.keyAlias` : SSL key store의 alias.
- `ktor.security.ssl.keyStorePassword` : SSL key store의 비밀번호.
- `ktor.security.ssl.privateKeyPassword` : SSL private key의 비밀번호.

## **Command line**

`EngineMain`을 사용해 서버를 생성하는 경우, 커맨드라인을 사용해 옵션을 오버라이드 후 서버를 실행할 수 있다. 예를 들어, `application.conf` 파일에 지정된 포트를 다음과 같이 오버라이드 할
수 있다.

```bash
java -jar sample-app.jar -port=8080
```

다음은 사용 가능한 커맨드라인 옵션들이다.

- `jar` : JAR 파일 경로.
- `config`: `application.conf` 파일 대신 사용할 커스텀 설정 파일 경로.

    ```bash
    java -jar sample-app.jar -config=anotherfile.conf
    ```

- `host` : 호스트 주소.
- `port` : 리스닝 포트.
- `watch` : 자동 새로고침에 사용되는 경로 지정.

SSL 옵션

- `sslPort` : 리스닝 SSL 포트.
- `sslKeyStore` : SSL key store.

커맨드 라인 옵션에 없는 정의된 속성을 오버라이드하려면 `-P` 플래그를 사용한다.

```bash
java -jar sample-app.jar -P:ktor.deployment.callGroupSize=7
```

## **Environment variables**

HOCON에서 `${ENV}` 문법을 사용해 환경 변수를 사용할 수 있다. 예를 들어, `PORT` 환경 변수를 `ktor.deployment.port` 속성에 다음과 같이 설정할 수 있다.

```
ktor {
    deployment {
        port = ${PORT}
    }
}
```

이 경우 환경 변수는 지정된 리스닝 포트로 사용된다. `PORT` 환경 변수가 없을 경우, 기본 값을 `${?PORT}` 이전에 사용하면 된다.

```
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
    }
}
```

# **Read configuration in code**

Ktor는 `application.conf`에 지정된 속성 값을 코드에서 접근할 수 있게 해준다. 예를 들어, `ktor.deployment.port` 속성에 값을 지정했다고 가정하자.

```
ktor {
    deployment {
        port = 8080
    }
}
```

[ApplicationEnvironment.config](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.application/-application-environment/config.html)
를 사용해 설정에 접근할 수 있고, 다음과 같이 필요한 속성 값을 얻을 수 있다.

```kotlin
fun Application.module(testing: Boolean = false) {
    val port = environment.config.propertyOrNull("ktor.deployment.port")?.getString() ?: "8080"
    routing {
        get {
            call.respondText("Listening on port $port")
        }
    }
}
```

이는 설정 파일의 커스텀 설정을 유지할 때와 이 값에 접근할 때 유용하게 사용된다.

# **Example: How to specify an environment using a custom property**

서버가 로컬에서 실행 중인지 또는 프로덕션 머신에서 실행 중인지에 따라 다른 작업을 수행할 수 있다. 이를 위해 `application.conf` ****파일에 ****커스텀 속성을 추가하고, 서버가 로컬에서 실행
중인지 프로덕션에서 실행 중인지에 따라 값이 달라지는 환경 변수로 초기화 할 수 있다. 아래 예제에서 `KTOR_ENV` 환경 변수가 커스텀 `ktor.environment` 속성에 할당되어 있다.

```
ktor {
    environment = ${?KTOR_ENV}
}
```

`ktor.environment` 값을 가져와 필요한 작업을 수행할 수 있다.

```kotlin
fun Application.module(testing: Boolean = false) {
    val env = environment.config.propertyOrNull("ktor.environment")?.getString()
    routing {
        get {
            call.respondText(
                when (env) {
                    "dev" -> "Development"
                    "prod" -> "Production"
                    else -> "..."
                }
            )
        }
    }
}
```

## References

* [Configuration | Ktor](https://ktor.io/docs/configurations.html)