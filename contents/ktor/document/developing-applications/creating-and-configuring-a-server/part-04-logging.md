# Logging

Ktor는 [SLF4J](http://www.slf4j.org/) 라이브러리를 사용해 애플리케이션의 이벤트를 로깅할 수 있도록
제공한다. [CallLogging](https://ktor.io/docs/call-logging.html) 플러그인을 설정해 클라이언트의 요청을 로깅할 수 있다.

# **Add Logback dependencies**

로깅을 활성화하기 위해, Logback 아티팩트를 포함시켜야 한다.

```kotlin
implementation("ch.qos.logback:logback-classic:$logback_version")
```

# **Configure Logback**

Logback을 설정하기 위해 `logback.xml` 파일을 리소스 위치(`src/main/resources`)에 저장해야 한다. 다음 예제는 콘솔에 로그를 출력하는 `STDOUT` appender 샘플
Logback 설정을 보여준다.

```xml

<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="trace">
        <appender-ref ref="STDOUT"/>
    </root>
    <logger name="io.netty" level="INFO"/>
</configuration>
```

파일로 출력을 원하는 경우 `FILE` appender를 사용하면 된다.

```xml

<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>testFile.log</file>
        <append>true</append>
        <encoder>
            <pattern>%d{YYYY-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="trace">
        <appender-ref ref="FILE"/>
    </root>
    <logger name="io.netty" level="INFO"/>
</configuration>
```

# **Access the logger**

로거 인스턴스는 [Logger](http://www.slf4j.org/api/org/slf4j/Logger.html) 인터페이스를 구현하는 클래스로 표현된다. `Application`
내에서 [Application.log](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.application/log.html)
속성을 사용해 로거 인스턴스에 접근할 수 있다. 다음은 모듈 안에서 로깅하는 방법을 보여준다.

```kotlin
fun Application.module(testing: Boolean = false) {
    log.info("Hello from module!")
}
```

`call.application.environment.log` 속성을
사용해 [ApplicationCall](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.application/-application-call/index.html)
의 로거에 접근할 수 있다.

```kotlin
routing {
    get("/api/v1") {
        call.application.environment.log.info("Hello from /api/v1!")
    }
}
```

클라이언트 요청 로깅을 활성화하려면, [CallLogging](https://ktor.io/docs/call-logging.html) 플러그인을 사용하라.

## References

* [Logging | Ktor](https://ktor.io/docs/logging.html)