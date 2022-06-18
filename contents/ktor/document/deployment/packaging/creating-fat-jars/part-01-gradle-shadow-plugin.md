# Gradle Shadow plugin

Gradle[Shadow](https://plugins.gradle.org/plugin/com.github.johnrengelman.shadow) 플러그인은 모든 코드 디펜던시를 포함한 실행가능한 JAR 파일을
생성해준다(fat JAR). 여기에서 fat JAR를 생성하고 실행하는 방벙을 알아본다.

## **Configure the Shadow plugin**

Fat JAR를 빌드하기 위해 우선 Shadow 플러그인 구성이 필요하다.

1. `build.gradle(.kts)` 파일을 열고 `plugins` 블록에 플러그인을 추가한다.

```kotlin
plugins {
    id("com.github.johnrengelman.shadow") version "7.0.0"
}
```

2. `shadowJar` task를 추가한다.

```kotlin
tasks {
    shadowJar {
        manifest {
            attributes(Pair("Main-Class", "com.example.ApplicationKt"))
        }
    }
}
```

명시적 `main` 함수 없이 `EngineMain`을 사용하면, `Main-Class`는 사용된 엔진에 따라 달라지며 다음과 같을 수 있다: `io.ktor.server.netty.EngineMain`.

## Build a Fat JAR

Fat JAR를 빌드하기 위해 터미널을 열고 `shadowJar` task를 실행한다.

```bash
./gradlew shadowJar
```

빌드가 완료되면, `ktor-gradle-sample-1.0-SNAPSHOT-all.jar` 파일이 `build/libs` 디렉토리에 생성된다.

## **Run the application**

빌드된 애플리케이션을 실행하려면

1. 터미널에서 `build/libs` 폴더로 이동한다.
2. 다음 명령을 실행하여 애플리케이션을 실행한다.

```bash
java -jar ktor-gradle-sample-1.0-SNAPSHOT-all.jar
```

3. 다음 메시지가 보여질 때까지 기다린다.

`[main] INFO Application - Responding at http://0.0.0.0:8080`

위 링크를 클릭하여 기본 브라우저로 애플리케이션을 열 수 있다.

<div align="center">
<img src="img/part-01/result.png" width="80%">
</div>

## References

* [Gradle Shadow plugin | Ktor](https://ktor.io/docs/fatjar.html)