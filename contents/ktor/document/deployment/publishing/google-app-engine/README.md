# Google App Engine

이 튜토리얼에선 Ktor 프로젝트를 준비하고 Google App Engine에 배포하는 방법을 보여준다.

# **Prerequisites**

튜토리얼을 시작하기 전, 다음 단계를 수행해야 한다.

- Google Cloud Platform 등록
- Google Cloud SDK 설치 및 초기화
- 자바를 위한 App Engine extension 설치

```bash
gcloud components install app-engine-java
```

## **Clone a sample application**

샘플 애플리케이션을 열기 위해, 다음 단계를 수행한다.

1. [codeSnippets](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets) 프로젝트를 클론한다.
2. `engine-main` 모듈을 연다.

## **Prepare an application**

### **Step 1: Apply the Shadow plugin**

이 튜토리얼은 fat JAR를 사용해 Google App Engine에 애플리케이션을 배포하는 방법을 보여준다. fat JARs를 생성하기 위해 Shadow 플러그인 적용이
필요하다. `build.gradle.kts`
파일을 열고 `plugins` 블록에 플러그인을 추가한다.

```groovy
plugins {
    id("com.github.johnrengelman.shadow") version "7.1.2"
}
```

### **Step 2: Configure the App Engine plugin**

[Google App Engine Gradle plugin](https://github.com/GoogleCloudPlatform/app-gradle-plugin)은 Google App Engine 애플리케이션을
빌드하고 배포하기 위한 task를 제공한다. 이 플러그인을 사용하기 위해 다음 단계를 따른다.

1. `settings.gradle.kts` 파일을 열고 다음 코드를 추가하여 Central Maven 레포지토리의 플러그인을 참조한다.

```kotlin
pluginManagement {
    repositories {
        gradlePluginPortal()
        mavenCentral()
    }
    resolutionStrategy {
        eachPlugin {
            if (requested.id.id.startsWith("com.google.cloud.tools.appengine")) {
                useModule("com.google.cloud.tools:appengine-gradle-plugin:${requested.version}")
            }
        }
    }
}
```

2. `build.gradle`을 열고 `plugins` 블록에 플러그인을 적용한다.

```kotlin
plugins {
    id("com.google.cloud.tools.appengine") version "2.4.2"
}
```

3. `build.gradle.kts` 파일에 다음과 같이 `appengine` 블록을 추가한다.

```kotlin
import com.google.cloud.tools.gradle.appengine.appyaml.AppEngineAppYamlExtension

configure<AppEngineAppYamlExtension> {
    stage {
        setArtifact("build/libs/${project.name}-all.jar")
    }
    deploy {
        version = "GCLOUD_CONFIG"
        projectId = "GCLOUD_CONFIG"
    }
}
```

### **Step 3: Configure App Engine settings**

`app.yaml` 파일에서 애플리케이션에 대한 App Engine 설정을 구성한다.

1. `src/main/appengine` 내 `appengine` 디렉토리를 생성한다.
2. 디렉토리 내에서 `app.yaml` 파일을 생성하고 다음과 같이 작성한다.

```yaml
runtime: java11
entrypoint: 'java -jar google-appengine-standard-1.0-SNAPSHOT-all.jar'
```

`entrypoint` 옵션은 fat JAR를 실행하는데 사용되는 커맨드를 포함한다.

## **Deploy an application**

애플리케이션을 배포하기 위해 터미널을 열고 다음 단계를 따른다.

1. 먼저 애플리케이션 리소스를 보관하는 top-level 컨테이너인 Google Cloud 프로젝트를 생성한다. 예를 들어, 다음 커맨드는 `ktor-sample-app-engine` 이름의 프로젝트를 생성한다.

```bash
gcloud projects create ktor-sample-app-engine --set-as-default
```

2. Cloud 프로젝트를 위한 App Engine 애플리케이션을 생성한다.

```bash
gcloud app create
```

3. 애플리케이션을 배포하기 위해 `appengineDeploy` Gradle task를 실행한다.

```bash
./gradlew appengineDeploy
```

그리고 Google Cloud에서 애플리케이션이 빌드 및 퍼블리시되기 전까지 기다린다.

```
...done.
Deployed service [default] to [https://ktor-sample-app-engine.ew.r.appspot.com]
```

## References

* [Google App Engine | Ktor](https://ktor.io/docs/google-app-engine.html)