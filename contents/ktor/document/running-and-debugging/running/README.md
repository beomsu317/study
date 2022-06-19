# Running

Ktor 서버 앱을 실행할 때, 다음 사항을 고려해라.

* 서버를 만드는 데 사용되는 방법은 패키징된 Ktor 앱을 실행할 때 command-line 인자를 전달하여 서버 파라미터 변수를 재정의할 수 있는지 여부에 영향을 준다.
* Gradle/Maven 빌드 스크립트는 `EngineMain`을 사용해 서버를 시작할 때 main 클래스 이름을 지정해야 한다.
* 서블릿 컨테이너에서 앱을 실행하는 경우, 지정된 서블릿 구성이 필요하다.

여기서는 이러한 구성 세부사항을 살펴보고 IntelliJ IDEA에서 Ktor 앱을 실행시키는 방법과 패키지 앱으로 실행하는 방법을 보여준다.

## Configuration specifics

### Configuration in code vs HOCON file

Ktor 앱을 실행하는 것은 서버를 생성하는 방식에 따라 다르다.(`embeddedServer` 또는 `EngineMain`)

* `embeddedServer`는 서버 파라미터(host, port 등)가 코드 상에서 설정된다. 따라서 앱이 실행될 때 이 파라미터를 변경할 수 없다.
* `EngineMain`은 Ktor가 `HOCON` 포맷을 사용하는 외부 파일을 통해 구성된다. 이 접근방법을 사용하면 command-line에서 패키징된 앱을 실행하고 인자를 전달해 필요한 서버 파라미터를
  재정의할 수 있다.

### Starting EngineMain - Gradle and Maven specifics

서버 생성 시 `EngineMain`을 사용하면 원하는 엔진으로 서버를 시작하기 위한 `main` 함수를 지정해야 한다. 다음 예제는 Netty 엔진으로 서버를 실행하는데 사용되는 `main` 함수를 보여준다.

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)
```

`main` 함수 내 엔진 구성 없이 Gradle/Maven을 사용해 Ktor 서버를 실행하기 위해, 빌드 스크립트에 `main` 클래스 이름을 지정해야 한다.

```kotlin
application {
    mainClass.set("io.ktor.server.netty.EngineMain")
}
```

### WAR specifics

Ktor는 원하는 엔진(Netty, Jetty, Tomcat 같이)으로 서버를 만들고 시작할 수 있다. 이 경우 앱은 엔진 설정, 연결, SSL 옵션을 제어한다.

이와 반대로, 서블릿 컨테이너는 앱 수명 주기 및 연결 설정을 제어해야 한다. Ktor는 앱의 모든 제어를 서블릿 컨테이너로 위임하는 `ServletApplicationEngine`라는 특별한 엔진을 제공한다. 

## Run an application

### Run an application in IDEA

Ktor 앱을 IntelliJ IDEA를 통해 2가지 방법으로 실행할 수 있다.

* `main` 함수 옆의 화살표 아이콘 클릭 후 `Run 'ApplicationKt'`을 선택하는 방법.
* `Ktor` run configuration을 생성하여 실행.

### Run an application using Gradle/Maven

Gradle 또는 Maven을 사용해 앱을 실행하기 위해 다음 플러그인이 필요하다.

* Gradle을 위한 [Application](https://docs.gradle.org/current/userguide/application_plugin.html) 플러그인
* Maven을 위한 [Exec](https://www.mojohaus.org/exec-maven-plugin/) 플러그인

### Run a packaged application

앱을 배포하기 전 패키징 작업을 수행해야 한다. 결과 패키지에서 Ktor 앱을 실행하는 것을 패키지 타입에 따라 다르며 다음과 같을 수 있다.

* Ktor 서버가 fat JAR로 패키지되어 있는 경우, 설정된 포트를 오버라이드 하여 실행할 수 있다.

```shell
java -jar sample-app.jar -port=8080
```

* Gradle [Application](https://docs.gradle.org/current/userguide/application_plugin.html) 플러그인을 사용해 패키징된 경우 해당하는 실행파일을 실행한다.

```kotlin
java -jar sample-app.jar -port=8080
```

* 서블릿 Ktor 앱을 실행하기 위해, [Gretty](https://ktor.io/docs/war.html#run) 플러그인의 `run` task를 이용한다. 

## References

* [Running | Ktor](https://ktor.io/docs/running.html)