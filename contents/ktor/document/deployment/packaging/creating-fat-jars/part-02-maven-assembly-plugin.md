# Maven Assembly plugin

Maven Assembly [Assembly plugin](http://maven.apache.org/plugins/maven-assembly-plugin/) 플러그인은 프로젝트의 output을 디펜던시, 모듈, 사이트 문서와 다른 파일들을
포함하는 단일 배포 가능한 아카이브로 결합하는 기능을 제공한다. Ktor 애플리케이션에 대해 어떻게 어셈블리를 빌드하고 실행하는 방법을 보여준다.

# **Configure the Assembly plugin**

어셈블리를 빌드하려면 우선 Assembly 플러그인을 구성해야 한다.

1. `pom.xml` 파일에서 [main.application.class](https://ktor.io/docs/server-dependencies.html#create-entry-point)가 지정되었는지 확인한다.

```xml
<properties>
    <main.class>com.example.ApplicationKt</main.class>
</properties>
```

명시적인 `main` 함수 없이 `EngineMain`을 사용한다면 앱의 메인 클래스는 사용된 엔진에 따라 달라지며 `io.ktor.server.netty.EngineMain`과 같이 보일 수 있다.

2. `plugins` 블록에 `maven-assembly-plugin`을 추가한다.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>${main.class}</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>assemble-all</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## **Build an assembly**

애플리케이션의 어셈블리를 빌드하려면, 터미널을 열고 다음 커맨드를 입력한다.

```bash
mvn package
```

이 빌드가 완료되면 `mainModule-1.0-SNAPSHOT-jar-with-dependencies.jar` 파일이 `target` 디렉토리에 있는 걸 확인할 수 있다.

## **Run the application**

빌드된 어플리케이션을 실행하기 위해:

1. 터미널을 열고 다음 명령을 통해 앱을 실행한다.

```shell
java -jar target/ktor-maven-sample-0.0.1-jar-with-dependencies.jar
```

2. 다음 메시지가 보여질 때까지 대기한다.

```bash
[main] INFO  Application - Responding at http://0.0.0.0:8080
```

위 링크를 클릭하여 기본 브라우저로 애플리케이션을 열 수 있다.

<div align="center">
<img src="img/part-02/result.png" width="80%">
</div>

## References

* [Maven Assembly plugin | Ktor](https://ktor.io/docs/maven-assembly-plugin.html#run)
