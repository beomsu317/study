# Adding Ktor dependencies

여기에선 Ktor에서 필요한 디펜던시에 대해 알아본다.

## Configure the repositories

Ktor 디펜던시를 추가하기 전, 프로젝트에 대한 레포지토리 설정을 해야한다.

* Production
    * Ktor의 Production 릴리즈는 Maven 저장소에서 사용할 수 있다.

```kotlin
repositories {
    mavenCentral()
}
```

* Early Access Program (EAP)
    * Ktor의 [EAP](https://ktor.io/eap/) 버전을 얻기
      위해 [Space repository](https://maven.pkg.jetbrains.space/public/p/ktor/eap/io/ktor/?_ga=2.96474975.1396641199.1655526702-658241611.1655526702&_gl=1*1kxhg8*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMjI3Ny4w)
      를 참조해야 한다.

```kotlin
repositories {
    maven {
        url = uri("https://maven.pkg.jetbrains.space/public/p/ktor/eap")
    }
}
```

Ktor
EAP는 [Kotlin dev repository](https://maven.pkg.jetbrains.space/kotlin/p/kotlin/dev?_ga=2.198057839.1396641199.1655526702-658241611.1655526702&_gl=1*184hlv6*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyOTI1OS4yLjEuMTY1NTUzMjI3Ny4w)
가 필요하다.

```kotlin
repositories {
    maven {
        url = uri("https://maven.pkg.jetbrains.space/kotlin/p/kotlin/dev")
    }
}
```

## Add dependencies

### Core dependencies

모든 Ktor 애플리케이션은 최소한 다음 디펜던시가 필요하다.

* `ktor-server-core` : Ktor의 core 기능 포함
* 엔진에 대한 디펜던시(예를 들어, `ktor-server-netty`)

다른 플랫폼의 경우 Ktor는 `-jvm`과 같은 접미사가 있는 플랫폼 별 아티팩트를 제공한다. 예를 들어, `ktor-server-core-jvm`, `ktor-server-netty-jvm` 등. Gradle은
주어진 플랫폼에 적합한 아티팩트를 resolve 하지만, Maven은 이 기능을 지원하지 않는다. 즉, Maven의 경우 플랫폼 별 접미사를 수동으로 추가해야 한다는 의미이다. 기본적으로 Ktor 애플리케이션의
디펜던시 블록은 다음과 같다.

```kotlin
dependencies {
    implementation("io.ktor:ktor-server-core:2.0.2")
    implementation("io.ktor:ktor-server-netty:2.0.2")
}
```

### Logging dependency

Ktor는 다양한 로깅 프레임워크(Logback 또는 Log4j)에 대한 facade로 SLF4J API를 사용하고 애플리케이션 이벤트를 로깅할 수 있다.

### Plugin dependencies

Ktor의 기능 확장을 위한 Plugin은 추가적인 디펜던시가 필요하다. 

## Create an entry point for running an application

Gradle/Maven을 통해 Ktor 서버를 실행하는 것은 서버를 생성하는 방법에 따라 다르다. 다음 방법 중 하나로 애플리케이션의 main 클래스를 지정할 수 있다.

* `embeddedServer`를 사용하는 경우 main 클래스를 지정하는 방법

```kotlin
application {
    mainClass.set("com.example.ApplicationKt")
}
```

* `EngineMain`을 사용하는 경우 main 클래스로 설정해야 한다. Netty의 경우 다음과 같다.

```kotlin
application {
    mainClass.set("io.ktor.server.netty.EngineMain")
}
```

## References

* [Adding Ktor dependencies](https://ktor.io/docs/server-dependencies.html)