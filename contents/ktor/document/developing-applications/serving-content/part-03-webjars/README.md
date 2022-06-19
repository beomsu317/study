# Webjars

[Webjars](https://www.webjars.org/) 플러그인을 사용하면 `Webjars`에서 제공하는 클라이언트 측 라이브러리를 제공할 수 있다. 이를 통해 JavaScript 및 CSS 라이브러리와
같은 자산을 `fat JAR`의 일부로 패키징할 수 있다.

## **Add dependencies**

`Webjars` 지원을 활성화하기 위해 다음 아티팩트를 포함한다.

* `ktor-server-webjars` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-server-webjars:$ktor_version")
```

* 필요한 클라이언트 측 라이브러리를 추가한다. 아래 예제는 bootstrap 아티팩트를 추가하는 방법을 보여준다.

```kotlin
implementation("org.webjars:bootstrap:$bootstrap_version")
```

## **Install Webjars**

`Webjars` 플러그인을 설치하기 위해 `install` 함수의 인자로 전달한다. 서버를 생성하는 방법에 따라 다르게 구현된다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.webjars.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Webjars)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.webjars.*
// ...
fun Application.module() {
    install(Webjars)
    // ...
}
```

## **Configure Webjars**

기본적으로 `WebJars`는 WebJars 에셋을 `/webjars` 경로에 제공한다. 다음 예에서 이를 변경하고 `/assets` 경로에서 `WebJars` 에셋을 제공하는 방법을 보여준다.

```kotlin
install(Webjars) {
    path = "assets"
}
```

예를 들어, `org.webjars:bootstrap` 디펜던시를 설치했다면 다음과 같이 `bootstrap.css`를 추가할 수 있다.

```js
<head>
    <link rel="stylesheet" href="/assets/bootstrap/bootstrap.css">
</head>
```

`Webjars`를 사용하면 디펜던시를 로드하는데 사용되는 경로를 변경하지 않고도 디펜던시 버전을 변경할 수 있다.

## References

* [Webjars | Ktor](https://ktor.io/docs/webjars.html)