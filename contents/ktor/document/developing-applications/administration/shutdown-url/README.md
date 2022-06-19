# Shutdown URL

[ShutDownUrl](https://api.ktor.io/ktor-server/ktor-server-host-common/io.ktor.server.engine/-shut-down-url/index.html?_ga=2.129315020.1396641199.1655526702-658241611.1655526702&_gl=1*16n1wpz*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTYyNjcwMC4xMC4xLjE2NTU2MjY3MzIuMA..)
플러그인을 사용하면 서버를 종료하는데 사용되는 URL을 구성할 수 있다.

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