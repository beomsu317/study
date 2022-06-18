# WAR

Ktor 애플리케이션은 Tomcat 및 Jetty를 포함하는 서블릿 컨테이너 내에서 실행 및 배포될 수 있다. 서블릿 컨테이너에서 배포하려면, WAR 아카이브를 생성하고 WAR를 지원하는 서버 또는 클라우드 서비스에
배포해야 한다.

- 서블릿 애플리케이션에서 사용하도록 Ktor 구성한다.
- 패키징된 WAR 애플리케이션을 실행하기 위해 Gretty와 War 플러그인을 적용한다.
- Ktor 서블릿 애플리케이션을 실행한다.
- WAR 아카이브를 생성하고 배포한다.

## **Configure Ktor in a servlet application**

Ktor를 사용하면 애플리케이션에서 바로 원하는 엔진(예: Netty, Jetty 또는 Tomcat)으로 서버를 만들고 시작할 수 있다. 이 경우 애플리케이션은 엔진 설정, 연결 및 SSL 옵션을 제어한다.

위 접근과 반대로, 서블릿 컨테이너는 애플리케이션의 생명주기와 연결 설정을 제어할 수 있어야 한다. Ktor는 애플리케이션에서 서블릿 컨테이너로 제어를
넘겨주는 [ServletApplicationEngine](https://api.ktor.io/ktor-server/ktor-server-servlet/ktor-server-servlet/io.ktor.server.servlet/-servlet-application-engine/index.html)
엔진을 제공한다.

> 연결 및 SSL 설정은 Ktor가 서블릿 컨테이너에 배포된 경우 적용되지 않는다.
>

### **Add dependencies**

서블릿 애플리케이션에서 Ktor를 사용하기 위해 `ktor-server-servlet` 아티팩트를 포함해야 한다.

```groovy
implementation("io.ktor:ktor-server-servlet:$ktor_version")
```

애플리케이션이 서블릿 컨테이너에 배포될 때 별도의 Jetty 및 Tomcat 아티팩트가 필요하지 않다.

### **Configure a servlet**

Ktor 서블릿을 등록하기 위해, `WEB-INF/web.xml` 파일을
열고 [ServletApplicationEngine](https://api.ktor.io/ktor-server/ktor-server-servlet/ktor-server-servlet/io.ktor.server.servlet/-servlet-application-engine/index.html)
을 `servlet-class`에 할당한다.

```xml
<servlet>
    <display-name>KtorServlet</display-name>
    <servlet-name>KtorServlet</servlet-name>
    <servlet-class>io.ktor.server.servlet.ServletApplicationEngine</servlet-class>
    <init-param>
        <param-name>io.ktor.ktor.config</param-name>
        <param-value>application.conf</param-value>
    </init-param>
    <async-supported>true</async-supported>
</servlet>
```

그 다음 서블릿에 대한 URL 패턴을 구성한다.

```xml
<servlet-mapping>
    <servlet-name>KtorServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

## **Configure Gretty**

Gretty 플러그인은 Jetty 및 Tomcat에서 서블릿 애플리케이션을 실행할 수 있도록 해준다. 이 플러그인을 설치하기 위해 `build.gradle` 파일을 열고 다음 코드를 `plugins` 블록에
추가한다.

```groovy
plugins {
    id("org.gretty") version "3.0.6"
}
```

`gretty` 블록을 다음과 같이 구성한다.

### Jetty

```groovy
gretty {
    contextPath = "/"
    logbackConfigFile = "src/main/resources/logback.xml"
}
```

### Tomcat

```groovy
gretty {
    servletContainer = "tomcat9"
    contextPath = "/"
    logbackConfigFile = "src/main/resources/logback.xml"
}
```

Tomcat를 사용하려면 `servletContainer`를 명시적으로 지정해주어야 한다.

마지막으로 `run` task를 구성한다.

```groovy
task run

afterEvaluate {
    run.dependsOn(tasks.findByName("appRun"))
}
```

## **Configure War**

War 플러그인은 WAR 아카이브를 생성할 수 있도록 해준다. `build.gradle` 파일의 `plugins` 블록에 다음 코드를 추가하여 설치할 수 있다.

```groovy
plugins {
    id("war")
}
```

## **Run an application**

`run` task를 사용해 구성된 Getty 플러그인과 함께 서블릿 애플리케이션을 실행할 수 있다. 예를 들어, 다음 커맨드는 jetty-war 예제를 실행한다.

```bash
./gradlew :jetty-war:run
```

## **Generate and deploy a WAR archive**

WAR 파일을 War 플러그인을 사용해 생성하려면 `war` task를 실행해야 한다. jetty-war 예제는 다음과 같다.

```bash
./gradlew :jetty-war:war
```

`jetty-war.war`는 `build/libs` 디렉토리에 생성된다. 서블릿 컨테이너에서 생성된 아카이브를 `jetty/webapps` 디렉토리로 복사하여 배포할 수 있다. 예를 들어,
아래의 `Dockerfile`은 Jetty 또는 Tomcat 서블릿 컨테이너 내에서 생성된 WAR를 실행하는 방법을 보여준다.

### Jetty

```docker
FROM jetty:9.4.43
EXPOSE 8080:8080
COPY ./build/libs/jetty-war.war/ /var/lib/jetty/webapps
WORKDIR /var/lib/jetty
CMD ["java","-jar","/usr/local/jetty/start.jar"]
```

### Tomcat

```docker
FROM tomcat:9.0.50
EXPOSE 8080:8080
COPY ./build/libs/tomcat-war.war/ /usr/local/tomcat/webapps
WORKDIR /usr/local/tomcat
CMD ["catalina.sh", "run"]
```

## References

* [WAR | Ktor](https://ktor.io/docs/war.html#run)