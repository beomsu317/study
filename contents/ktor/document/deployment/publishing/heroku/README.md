# Heroku

이 튜토리얼에선 Ktor 애플리케이션을 Heroku에 준비하고 배포하는 방법을 알아본다.

## **Prerequisites**

튜토리얼을 시작하기 전 다음 조건이 충족되는지 확인하자.

- Heroku 계정
- Heroku CLI

## **Clone a sample application**

샘플 애플리케이션을 열기 위해 다음 단계를 따른다.

1. [ktor-get-started-sample](https://github.com/ktorio/ktor-get-started-sample) 프로젝트를 클론한다.

## **Prepare an application**

### **Step 1: Configure a port**

우선 들어오는 요청을 리스닝할 포트를 지정해야 한다. Heroku는 `PORT` 환경변수를 사용하므로 이 변수의 값을 사용하도록 애플리케이션을 구성해야 한다. Ktor 서버를 구성하는 방법에 따라 방법이 나뉜다.

- `embedded-server` 브랜치를 선택한 경우, `System.getenv`를 사용해 환경변수를 가져와야 한다. `src/main/kotlin/com/example` 폴더에
  위치한 `Application.kt` 파일을 열고 `port` 파라미터 값을 변경한다.

```kotlin
fun main() {
    embeddedServer(Netty, port = System.getenv("PORT").toInt()) {
        // ...
    }.start(wait = true)
}
```

- `engine-main` 브랜치를 선택한 경우, `${ENV}` 문법을 사용해 `port`에 환경변수를 할당한다. `application.conf` 파일을 열고 다음과 같이 수정한다.

```
ktor {
    deployment {
        port = 8080
        port = ${?PORT}
    }
}
```

### **Step 2: Add a stage task**

`build.gradle.kts` 파일을 열고 Heroku 플랫폼에서 실행되는 실행 파일을 만들기 위해 Heroku에서 사용하는 커스텀 `stage` task를 추가한다.

```kotlin
tasks {
  create("stage").dependsOn("installDist")
}
```

`installDist` task는 이미 샘플 프로젝트에 추가된 Gradle application 플러그인과 함께 제공된다.

### **Step 3: Create a Procfile**

프로젝트 루트에 `Procfile`을 생성하고 다음 내용을 추가한다.

```
web: ./build/install/ktor-get-started-sample/bin/ktor-get-started-sample
```

이 파일은 `stage` task로 생성된 애플리케이션 실행 파일의 경로를 지정하여 Heroku가 애플리케이션을 시작할 수 있도록 한다.

## **Deploy an application**

Git을 사용해 애플리케이션을 Heroku에 배포하기 위해, 터미널을 열고 다음 단계를 따른다.

1. 변경사항을 커밋한다.

```bash
git add .
git commit -m "Prepare app for deploying"
```

2. Heroku CLI 로그인

```bash
heroku login
```

3. `heroku create` 명령을 사용해 Heroku 애플리케이션 생성

```bash
heroku create ktor-sample-heroku
```

이 커맨드는 2가지를 수행한다.

- web dashboard에서 이용 가능한 새로운 Heroku 애플리케이션 생성
- `heroku`라 불리는 새 Git remote를 로컬 레포지토리에 추가한다.

4. 애플리케이션을 배포하기 위해 `heroku main`에 변경사항을 push

```shell
git push heroku embedded-server:main
```

그리고 Heroku가 빌드 및 퍼블리시 하기 전까지 기다린다.

```
...
remote: https://ktor-sample-heroku.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
```

## References

* [Heroku | Ktor](https://ktor.io/docs/heroku.html#deploy-app)