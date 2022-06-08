# Gradle for Beginners

Gradle의 기본에 대해 알아보자. 안드로이드 Gradle은 두 가지가 존재(Project 단 Gradle 파일과 App 단 Gradle 파일)한다.

### Project Level Gradle File

이 파일에선 프로젝트에 대한 구성을 포함한다. 예를 들어, 프로젝트에서 사용할 Compose 버전 또는 플러그인 등.

### Module Level Gradle File

`android` 블럭에는 안드로이드 구성에 대한 설정을 포함한다. 예를 들어, 컴파일 SDK 버전 또는 패키지 이름 등.

즉, Gradle은 빌드 자동화 도구이다. Gradle 파일에 설정한 모든 구성을 취하고, 실행 가능한 앱을 만들기 위해 정의된 순서대로 task를 처리한다. Gradle 자체가 앱을 컴파일하는 것으로 자주
오해하는데, Gradle은 컴파일 툴을 사용해 앱을 빌드하는 것이다.

### Gradle Wrapper

모든 안드로이드 프로젝트에는 `gradlew`라는 실행 파일이 있다. 이것은 스크립트이며, 지정된 특정 버전의 Gradle을 설치하고 지정된 task를 실행한다.

다음 명령을 통해 앱을 빌드할 수 있으며, 지정된 모든 task가 실행된다.

```shell
./gradlew build
```

다음 명령을 통해 모든 task를 확인할 수 있다.

```shell
./gradlew tasks

> Task :tasks

------------------------------------------------------------
Tasks runnable from root project 'GradleTest'
------------------------------------------------------------

Android tasks
-------------
androidDependencies - Displays the Android dependencies of the project.
signingReport - Displays the signing info for the base and test modules
sourceSets - Prints out all the source sets defined in this project.

Build tasks
...
```

### Build Types

`buildTypes` 블럭에서 alpha, beta, production, debug 빌드 타입에 대한 구성을 정의할 수 있다.

기본적으로 `debug`, `release` 빌드가 있으며 디버그에 필요한 구성과 릴리즈에 필요한 구성을 할 수 있다.

beta 빌드에 대한 구성을 하고 싶다면 다음과 같이 `beta` 블럭을 만들어주면 된다.

```groovy
android {
    // ...
    buildTypes {
        // ...
        beta {
            minifyEnabled false
            applicationIdSuffix ".beta"
        }
    }
    // ...
}
```

`Build` > `Select Build Variant...`를 클릭하고 `beta` 빌드를 선택하여 적용할 수 있다.

### Product Flavor

Product Flavor는 `buildTypes`와 유사하지만, 실제 유저와 관련된 타입이다. 예를 들어, 과금 버전과 무료 버전 또는 최소한의 SDK 버전이 필요한 경우 Product Flavor를 적용할 수 있다.

과금 버전과 무료 버전을 만들 경우 다음과 같이 구성하면 된다.

```groovy
android {
    // ...
    flavorDimensions "paidMode"
    productFlavors {
        free {
            dimension "paidMode"
            applicationIdSuffix ".free"
        }
        paid {
            dimension "paidMode"
            applicationIdSuffix ".paid"
        }
    }
    // ...
}
```

`Build Variant` 메뉴에서 free, paid 접두사가 붙은 build variant를 확인할 수 있다. 즉, build type과 product flavor가 합쳐진다는 의미이다.

만약 추가되는 기능이 API 30 이상에서만 동작한다면 `minSdk`를 30으로 변경해야 한다. 다음과 같이 `minSdk` flavor를 추가해 `minSdk` 별로 빌드가 가능하도록 만들 수 있다.

```
android {
    // ...
    flavorDimensions "paidMode", "minSdk"
    productFlavors {
        free {
            dimension "paidMode"
            applicationIdSuffix ".free"
        }
        paid {
            dimension "paidMode"
            applicationIdSuffix ".paid"
        }
        minSdk30 {
            dimension "minSdk"
            minSdk 30
        }
        minSdk21 {
            dimension "minSdk"
            minSdk 21
        }
    }
    // ...
}
```

특정 디펜던시를 원하는 flavor에 적용되도록 하는 것도 가능하다. 다음과 같이 `androidx.compose.ui:ui-tooling` 디펜던시를 특정 flavor에만 적용할 수 있다.

```groovy
dependencies {
    // ...
    freeMinSdk21DebugImplementation 'androidx.compose.ui:ui-tooling:1.1.1'
}
```

### Source Sets

`java` 디렉토리 하위에 3개의 소스(main, androidTest, test)가 존재한다. androidTest, test에서 main 소스에 접근 가능하지만, main에선 androidTest, test에 접근할 수 없다.

디버그 빌드에서만 접근 가능한 `src/debug/java`를 생성한다. Project View에서 생성된 디렉토리를 확인할 수 있다. 여기에서 디버그 빌드에서만 사용되는 클래스를 작성할 수 있다. 다른 build variant와는 공유되지 않는다. `com.test` 패키지 생성 후 빈 `DemoClass` 클래스를 하나 생성한다.

동일하게 `testDebug/java` 및 패키지를 생성하고 빈 `DemoClassTest` 클래스를 생성한다. 이제 Android View로 변경하면 디버그 빌드 관련 소스가 보여진다. 

모든 build variant는 위와 같이 특정 소스 셋을 추가할 수 있다. 즉, 해당하는 build variant에만 적용 가능한 소스를 작성할 수 있다는 의미이다. 

## How to Automate Tasks Using Gradle

Gradle에서 커스텀 task를 만들기 전 우선 `clean` task를 보자.

```groovy
task clean(type: Delete) {      // delete 타입이므로 무엇인가 삭제한다는 의미
    delete rootProject.buildDir // build 디렉토리를 삭제
}
```

Module Gradle 파일 하단에 문자를 출력하는 task를 작성한다.

```groovy
// ...
task helloWorld() {
    println "Hello World"
}
```

빌드 로그를 보면 "Hello World"가 먼저 출력되고 후에 task가 실행된 것을 확인할 수 있다. 이는 task가 구성될 때 실행되기 때문이다.

```
Executing tasks: [helloWorld] in project /Users/beomsulee/Desktop/Project/GradleTest


> Configure project :app
Hello World

> Task :app:helloWorld UP-TO-DATE

BUILD SUCCESSFUL in 850ms
```

실제로 task가 실행되기 전 또는 후에 실행되길 원한다면, `doFirst` 또는 `doLast`를 사용한다.

```groovy
task helloWorld() {
    doFirst {
        println "Hello World first"
    }
    println "Hello World"
    doLast {
        println "Hello World last"
    }
}
```

추가된 코드는 실제 task가 실행되기 전/후에 실행되는 것을 확인할 수 있다.

```
> Configure project :app
Hello World

> Task :app:helloWorld
Hello World first
Hello World last

BUILD SUCCESSFUL in 202ms
```

task를 등록하기 위해 `tasks.register()`를 사용한다. 10개의 task를 등록하는 task를 실행해보자.

```groovy
task helloWorld() {
    10.times {i ->
        tasks.register("task$i") {
            doLast {
                println "Hello from task $i"
            }
        }
    }
}
```

이후 `./gradlew task4` 명령을 실행하면 등록된 task가 실행된다.

```
> Task :app:task4
Hello from task 4

BUILD SUCCESSFUL in 1s
```

만약 `helloWorld` task에서 등록된 task가 실행되게 하려면 `dependsOn` 키워드를 사용한다. 

```groovy
task helloWorld() {
    10.times {i ->
        tasks.register("task$i") {
            doLast {
                println "Hello from task $i"
            }
        }
    }
    dependsOn "task1" // helloWorld가 task1에 의존한다는 의미이다.
}
```

위 task를 실행하면 `task1`이 실행된 후 `helloWorld`가 실행되는 것을 확인할 수 있다. 즉, `helloWorld`가 `task1`에 의존하므로 `task1`이 먼저 실행된 후 실행된다.

```
> Task :app:task1
Hello from task 1

> Task :app:helloWorld

BUILD SUCCESSFUL in 119ms
```

`tasks.named`를 통해 원하는 task에 대해 의존할 task를 설정할 수 있다.

```groovy
task helloWorld() {
    10.times {i ->
        tasks.register("task$i") {
            doLast {
                println "Hello from task $i"
            }
        }
    }
    tasks.named("task1") {
        dependsOn "task4", "task6", "task8"
    }
    dependsOn "task1"
}
```

`task4` > `task6` > `task8` > `task1` > `helloWorld` 순으로 task가 진행되는 것을 확인할 수 있다. `task4`, `task6`, `task8`은 `task1`이 실행되기 전에 실행되어야 하기 때문에 먼저 실행되며, `task1`이 실행된 후에야 `helloWorld`가 실행된다.

```
> Task :app:task4
Hello from task 4

> Task :app:task6
Hello from task 6

> Task :app:task8
Hello from task 8

> Task :app:task1
Hello from task 1

> Task :app:helloWorld
```


### Create Backup Task

이제 실제로 필요할만한 task를 만들어보자. `app-debug.apk` 디버그 앱을 루트 디렉토리에 복사하는 task를 작성한다.

```groovy
task copyApk(type: Copy) { 
    def sourceDir = layout.buildDirectory.dir("intermediates/apk/debug/app-debug.apk")
    def destDir = "$rootDir/apk"
    from sourceDir
    into destDir
    rename "app-debug.apk", "gradle-sample.apk"
    doLast {
        def file = new File(destDir, "gradle-sample.apk")
        ant.checksum file: file.path // 파일에 대한 MD5 체크섬이 생성된다.
    }
}
```

각각의 task가 등록될 때 실행되는 `whenTaskAdded`를 통해 `assembleDebug`가 실행된 후 `copyApk`가 실행되도록 작성한다.

```groovy
tasks.whenTaskAdded { task ->
    if (task.name == "assembleDebug") { // apk가 assemble 되기 전 실행되는 task
        task.finalizedBy "copyApk" // assembleDebug가 실행된 후 copyApk 실행한다는 의미
    }
}
```

만약 Unit Test가 실행된 이후 task가 수행되길 원하는 경우 다음과 같이 `dependsOn`을 추가한다. Unit Test에 의존한다는 의미이며, Unit Test가 실패할 경우 해당 task는 수행되지 않는다.

```groovy
task copyApk(type: Copy, dependsOn: "test") {
    // ...
}
```

## References

* [Gradle for Beginners (Build Types, Product Flavors, Build Variants, Source Sets)](https://www.youtube.com/watch?v=o0M4f5djJTQ)
* [How to Automate Tasks Using Gradle - Android Studio Tutorial](https://www.youtube.com/watch?v=2SWgl-OdxDY)