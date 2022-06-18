# Shutdown URL

이 플러그인은 접근할 때 서버를 종료하는 URL을 활성화한다.

사용하는 2가지 방법이 있다. 

* [Automatically using HOCON](https://ktor.io/docs/shutdown-url.html#hocon)
* [Installing the plugin](https://ktor.io/docs/shutdown-url.html#install)

## Configure shutdown URL in HOCON

HOCON의 [ktor.deployment.shutdown.url](https://ktor.io/docs/configurations.html#predefined-properties) 속성을 사용해 종료 URL을
구성할 수 있다.

```kotlin
ktor {
    deployment {
        shutdown.url = "/shutdown"
    }
}
```

## Configure shutdown URL by installing the plugin

Shutdown URL을 코드 상에서 설정하고 설치하기 위해 `ShutDownUrl.ApplicationCallPlugin`을 `install` 함수에 전달하고 `shutDownUrl` 속성을 사용한다.

```kotlin
install(ShutDownUrl.ApplicationCallPlugin) {
    shutDownUrl = "/shutdown"
    exitCodeSupplier = { 0 }
}
```

## References

* [Shutdown URL | Ktor](https://ktor.io/docs/shutdown-url.html)