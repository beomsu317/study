# HSTS

`HSTS`는 RFC6797에 따라 필수 HTTP Strict Transport Security 헤더를 요청에 추가하는 플러그인이다. 브라우저가 HSTS 정책 헤더를 수신하면, 더 이상 지정된 기간 동안 안전하지
않은 연결로 서버에 연결을 시도하지 않는다.

> HSTS 정책 헤더는 HTTP 연결에서 무시된다. HSTS 효과를 보기 위해 HTTPS 연결을 제공해야 한다.

브라우저가 HSTS 정책 헤더를 수신하면, 지정된 시간 동안 더 이상 안전하지 않은 연결로 서버에 연결을 시도하지 않는다.

## Add dependencies

`HSTS`를 사용하기 위해 `ktor-server-hsts` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-server-hsts:$ktor_version")
```

## Install HSTS

`HSTS` 플러그인을 설치하기 위해, 이를 `install` 함수에 전달해야 한다. 서버를 생성하는 방법에 따라 두 가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.hsts.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(HSTS)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.hsts.*
// ...
fun Application.module() {
    install(HSTS)
    // ...
}
```

위 코드는 HSTS를 기본 구성으로 설정하는 것을 보여준다.

## Configure HSTS

`HSTS`
는 [HSTSConfig](https://api.ktor.io/ktor-server/ktor-server-plugins/ktor-server-hsts/io.ktor.server.plugins.hsts/-h-s-t-s-config/index.html?_ga=2.196378604.1396641199.1655526702-658241611.1655526702&_gl=1*1lpjn1q*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUzNjA4NS4zLjEuMTY1NTUzNjU3MC4w)
를 통해 설정을 노출한다. 다음 예는 `maxAgeInSeconds` 속성을 사용해 클라이언트가 알려진 HSTS 호스트 목록에서 호스트를 유지해야 하는 기간을 지정하는 방법을 보여준다.

```kotlin
install(HSTS) {
    maxAgeInSeconds = 10
}
```

## References

* [HSTS | Ktor](https://ktor.io/docs/hsts.html)