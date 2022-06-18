# Docker Compose

Ktor 앱을 Docker Compose에서 실행하는 방법에 대해 알아본다. Exposed를 사용해 H2 파일 디비에
연결한 [Adding persistence to a website](https://ktor.io/docs/interactive-website-add-persistence.html) 프로젝트를 사용할 것이다. 여기에서
H2를 별도의 디비 서비스로 실행되는 PostgreSQL 디비로 교체하고 Ktor 앱은 웹 서비스로 실행한다.

## Get the application ready

### Add PostgreSQL dependency

첫째로 PostgreSQL 라이브러리 디펜던시를 추가한다. `gradle.properties` 파일을 열고 라이브러리 버전을 지정한다.

```
postgresql_version = 42.3.0
```

그리고 `build.gradle.kts`를 열고 다음 디펜던시를 추가한다.

```kotlin
val postgresql_version: String by project

dependencies {
    implementation("org.postgresql:postgresql:$postgresql_version")
}
```

### Connect to a database

[tutorial-website-interactive-persistence](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/tutorial-website-interactive-persistence)
샘플은 디비 연결을 위해 `com/example/dao/DatabaseFactory.kt` 파일에 하드코딩된 `driverClassName`와 `jdbcURL`를 사용한다. PostgreSQL 디비에 대한 연결
설정을 커스텀 설정 그룹으로 추출해보자. `src/main/resources/application.conf` 파일을 열고 다음과 같이 `ktor` 그룹 외부에 `storage` 그룹을 추가한다.

```kotlin
storage {
    driverClassName = "org.postgresql.Driver"
    jdbcURL = "jdbc:postgresql://db:5432/ktorjournal?user=postgres"
}
```

`jdbcURL`는 다음 컴포넌트를 포함하고 있다.

* `db:5432`는 PostgreSQL 디비가 실행되는 호스트와 포트이다.
* `ktorjournal`는 서비스가 실행될 때 생성되는 디비 이름이다.

이 설정들은 나중에 [docker-compose.yml](https://ktor.io/docs/docker-compose.html#configure-docker) 파일에 구성된다.

`com/example/dao/DatabaseFactory.kt`을 열고 `init` 함수를 업데이트하여 구성 파일에서 스토리지 설정을 로드한다.

```kotlin
fun init(config: ApplicationConfig) {
    val driverClassName = config.property("storage.driverClassName").getString()
    val jdbcURL = config.property("storage.jdbcURL").getString()
    val database = Database.connect(jdbcURL, driverClassName)
    transaction(database) {
        SchemaUtils.create(Articles)
    }
}
```

`init` 함수는 이제 `ApplicationConfig`를 수락하며 `config.property`를 사용해 커스텀 설정을 로드한다.

마지막으로 `com/example/Application.kt`를 열고 `environment.config`를 `DatabaseFactory.init`에 전달하여 앱 실행 시 연결 설정을 로드한다.

```kotlin
fun Application.module() {
    DatabaseFactory.init(environment.config)
    configureRouting()
    configureTemplating()
}
```

### Configure the Shadow plugin

Docker에서 실행하기 위해 앱에 필요한 모든 파일이 컨테이너에 배포되어 있어야 한다. 빌드 시스템에 따라 이를 수행하는 다양한 플러그인이 있다.

* [Gradle Shadow plugin](https://ktor.io/docs/fatjar.html)
* [Maven Assembly plugin](https://ktor.io/docs/maven-assembly-plugin.html)

예를 들어, Shadow 플러그인을 적용할 경우 `build.gradle.kts` 파일을 열고 플러그인 블록에 `shadow` 플러그인을 추가한다.

```kotlin
plugins {
    application
    kotlin("jvm")
    id("com.github.johnrengelman.shadow") version "7.1.2"
}
```

## Configure Docker

### Prepare Docker image

앱을 dockerize 하기 위해 `Dockerfile`을 프로젝트의 루트에 생성하고 다음과 같이 작성한다.

```
FROM openjdk:11
EXPOSE 8080:8080
RUN mkdir /app
COPY ./build/libs/*.jar /app/ktor-docker-sample.jar
ENTRYPOINT ["java","-jar","/app/ktor-docker-sample.jar"]
```

`Dockerfile`은 `docker compose up`을 실행하기 전 fat JAR를 생성해야 한다.

### Configure Docker Compose

프로젝트의 루트에 `docker-compose.yml`을 생성하고 다음 컨텐츠를 추가한다.

```yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ktorjournal
      POSTGRES_HOST_AUTH_METHOD: trust
    ports:
      - "54333:5432"
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U postgres" ]
      interval: 1s
```

* `web` 서비스는 이미지 내부에 패키징된 Ktor 앱을 실행하는데 사용된다.
* `db` 서비스는 `postgres` 이미지를 사용하여 journal의 기사를 저장하기 위한 `ktorjournal` 디비를 생성한다.

## Build and run services

1. `docker compose up`을 실행하기 전 Ktor 앱을 포함한 fat JAR를 생성한다.

```shell
./gradlew :tutorial-website-interactive-docker-compose:shadowJar
```

2. 그 다음 `docker compose up`을 실행한다.

```shell
docker compose --project-directory snippets/tutorial-website-interactive-docker-compose up
```

그리고 Docker Compose가 이미지를 pulls/builds 하고 컨테이너를 시작할 때까지 기다린다. `http://localhost:8080/`에 접근해 기사를 생성, 수정, 삭제할 수 있다.

## References

* [Docker Compose](https://ktor.io/docs/docker-compose.html)