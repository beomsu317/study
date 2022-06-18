# AWS Elastic Beanstalk

이 튜토리얼에선 Ktor 애플리케이션을 AWS Elastic Beanstalk에 준비하고 배포하는 방법을 알아본다.

## **Prerequisites**

튜토리얼을 시작하기 전 AWS 계정이 필요하다.

## **Clone a sample application**

샘플 애플리케이션을 열기 위해 다음 단계를 따른다.

1. [codeSnippets](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets) 프로젝트를 클론한다.
2. `embedded-server` 또는 `engine-main` 샘플을 연다.

## **Prepare an application**

### **Step 1: Configure a port**

우선 들어오는 요청의 리슨 포트를 지정해야 한다. Beanstalk는 포트 5000에서 애플리케이션으로 요청을 전달한다. 선택적으로 기본 포트를 `PORT` 환경 변수를 설정해 변경할 수 있다. Ktor 서버 생성
방법에 따라 포트는 다음과 같은 방법으로 구성될 수 있다.

- `embedded-server` 브랜치의 경우, `System.getenv`를 사용해 환경변수를 얻거나 환경 변수에 정의되어 있지 않으면 5000 기본 값을 사용한다. `Application.kt` 파일을
  열어 `port` 파라미터를 다음과 같이 변경한다.

```kotlin
fun main() {
    embeddedServer(Netty, port = (System.getenv("PORT") ?: "5000").toInt()) {
        // ...
    }.start(wait = true)
}
```

- `engine-main` 브랜치의 경우, `${ENV}` 문법을 사용해 `port` 파라미터에 환경변수를 할당할 수 있다. `application.conf` 파일을 열고 다음과 같이 변경한다.

```
ktor {
    deployment {
        port = 5000
        port = ${?PORT}
    }
}
```

### **Step 2: Apply the Shadow plugin**

이 튜토리얼에서는 Elastic Beanstalk에서 fat JAR를 사용해 배포한다. fat JAR를 생성하기 위해 Shadow 플러그인을 추가한다. `build.gradle.kts` 파일을 열고 `plugins`
블록에
플러그인을 추가한다.

```groovy
plugins {
    id("com.github.johnrengelman.shadow") version "7.1.2"
}
```

그런 다음 [main application class](https://ktor.io/docs/server-dependencies.html#create-entry-point) 구성을 확인한다.

```kotlin
application {
   mainClass.set("io.ktor.server.netty.EngineMain")
}
```

## **Build a Fat JAR**

Fat JAR를 빌드하기 위해 터미널을 열고 `shadowJar` task를 실행한다.

```bash
./gradlew :aws-elastic-beanstalk:shadowJar
```

빌드가 완료되면 `build/libs` 디렉토리에 `aws-elastic-beanstalk-all.jar` 파일이 생성된 것을 확인할 수 있다.

## **Deploy an application**

애플리케이션을 배포하기 위해, [AWS Management Console](https://aws.amazon.com/console/)에 로그인하고 다음 단계를 따른다.

1. AWS Services 그룹의 Elastic Beanstalk 서비스를 연다.
2. Create Application을 클릭한다.
3. 다음 애플리케이션 설정을 지정한다.
    - Application name : 애플리케이션 이름 지정
    - Platform : Java
    - Platform branch : Corretto 11 running on64bit Amazon Linux 2 선택
    - Application code : Upload your code 선택
    - Source code origin : Local file 선택 후 Choose file 버튼 클릭하고 생성된 Fat JAR를 선택
4. Create application 클릭 및 Beanstalk가 환경을 설정하고 애플리케이션을 퍼블리시하기 전까지 대기

```
INFO    Instance deployment completed successfully.
INFO    Application available at Samplektorapp-env.eba-bnye2kpu.us-east-2.elasticbeanstalk.com.
INFO    Successfully launched environment: Samplektorapp-env
```

## References

* [AWS Elastic Beanstalk | Ktor](https://ktor.io/docs/elastic-beanstalk.html#deploy-app)