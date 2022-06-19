# Auto-reload

개발 중 서버를 재시작하는 것은 시간을 소비할 수 있다. Ktor는 Auto-reload 기능을 통해 이러한 부분을 극복한다. 코드 변경 시 애플리케이션 클래스를 다시 로드하고 빠른 피드백 루프를 제공한다.
Auto-reload를 사용하기 위해 다음 단계를 따라야 한다.

1. 개발자 모드 활성화
2. (선택적으로)Watch path 설정
3. 변경사항에 대한 재컴파일 활성화

## Enable the development mode

Auto-reload 기능을 사용하기 위해 우선 [development mode](https://ktor.io/docs/development-mode.html#enable)를 활성화해야 한다.

- `EngineMain`을 통해 서버를 실행하는 경우, `application.conf` 파일에 development mode를 활성화한다.
- `embeddedServer`를 통해 서버를 실행하는 경우 [io.ktor.development](https://ktor.io/docs/development-mode.html#system-property) 시스템
  속성을 사용한다.

development mode가 활성화되면, Ktor는 작업 디렉토리의 출력 파일을 자동으로 감시한다. 필요에 따라 감시 경로를 지정하여 감시 폴더의 집합의 범위를 줄일 수 있다.

## Configure watch paths

development mode를 활성화한 경우 Ktor는 작업 디렉토리로부터 출력되는 파일을 감시하기 시작한다. 예를 들어, Gradle을 사용한 `ktor-sample` 프로젝트 빌드의 경우 다음 폴더가 감시된다.

```
ktor-sample/build/classes/kotlin/main/META-INF
ktor-sample/build/classes/kotlin/main/com/example
ktor-sample/build/classes/kotlin/main/com
ktor-sample/build/classes/kotlin/main
ktor-sample/build/resources/main
```

감시 경로를 사용하면 감시 폴더 집합의 범위를 줄일 수 있다. 이를 위해 감시 경로를 지정해야 한다. 예를 들어, `ktor-sample/build/classes`의 하위 폴더를 모니터링하려면, `classes`에
감시 경로로 전달하면 된다. 서버를 실행하는 방법에 따라 다음과 같은 방법으로 감시 경로를 지정할 수 있다.

- `application.conf` 내, `watch` 옵션을 지정

```kotlin
ktor {
    development = true
    deployment {
        watch = [classes]
    }
}
```

일부 감시 경로를 컴마로 분리해 전달할 수 있다.

```kotlin
watch = [classes, resources]
```

- `embeddedServer`를 사용하는 경우 `watchPaths` 파라미터로 감시 경로 전달

```kotlin
embeddedServer(Netty, port = 8080, watchPaths = listOf("classes")) {
    routing {
        get("/") {
            call.respondText("Hello, world!")
        }
    }
}.start(wait = true)
```

## Recompile on changes

Auto-reload 기능은 출력된 파일의 변화를 감지하기 때문에, 프로젝트를 다시 빌드해야 한다. IntelliJ IDEA에서 수동으로 이 작업을 수행하거나 `-t` 커맨드라인 옵션을 사용해 Gradle에서
continuous build execution을 활성화할 수 있다.

- IntelliJ IDEA에서 프로젝트를 수동으로 재빌드하기 위해, 메인 메뉴에서 **Build | Rebuild Project**를 선택한다.
- Gradle을 사용해 프로젝트를 자동으로 재빌드하기 위해 터미널에서 `build` task를 `-t` 옵션을 사용하여 빌드 작업을 실행할 수 있다.

```bash
./gradlew -t build
```

> 테스트를 스킵하기 위해 `-x` 옵션을 `build` task에 전달할 수 있다.
> ```bash
> ./gradlew -t build -x test -i
> ```

## References

* [Auto-reload | Ktor](https://ktor.io/docs/auto-reload.html)