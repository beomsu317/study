# Webjars

이 플러그인은 [webjars](https://www.webjars.org/)로 static 콘텐츠를 제공하도록 활성화한다. 이를 통해 자바스크립트 라이브러리 및 CSS 같은 asset을 uber-jar의 일부로
패키징할 수 있다.

# **Add dependencies**

`Webjars` 지원을 활성화하기 위해 `ktor-webjars` 아티팩트를 포함한다.

```kotlin
implementation("io.ktor:ktor-webjars:$ktor_version")
```

# **Install Webjars**

`Webjars` 플러그인을 설치하기 위해 `install` 함수의 인자로 전달한다. 서버를 생성하는 방법에 따라 다르게 구현된다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Webjars)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(Webjars)
    // ...
}
```

# **Configure Webjars**

```kotlin
install(Webjars) {
    path = "assets" //defaults to /webjars
    zone = ZoneId.of("EST") //defaults to ZoneId.systemDefault()
}
```

`/assets/` 경로에서 모든 webjars asset을 제공하도록 플러그인을 구성한다. `zone` 인자는 캐싱을 지원하기 위한 `Last-Modified` 헤더와 함께 사용할 올바른 타임 존을 구성한다.

# **Versioning support**

webjars를 사용하면 개발자가 템플릿을 로드하는데 사용되는 path의 변경 없이 디펜던시의 버전을 변경할 수 있다.

`org.webjars:jquery:3.2.1`를 import 했다고 가정하면, 다음 html 코드를 사용해 가져올 수 있다.

```xml

<head>
    <script src="/webjars/jquery/jquery.js"></script>
</head>
```

버전을 지정할 필요가 없으며 디펜던시를 업데이트하기로 한 경우 템플릿을 수정할 필요가 없다.

## References

* [Webjars | Ktor](https://ktor.io/docs/webjars.html)