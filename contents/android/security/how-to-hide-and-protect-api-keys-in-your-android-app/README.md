# How to Hide & Protect API Keys in Your Android App

안드로이드 앱에서 API 키를 숨기는 방법을 알아보자.

API가 Github 레포지토리에 올라가기 원치 않는 파일들을 `.gitignore`에 선언하여 업로드가 되지 않도록 할 수 있다. 다음 `.gitignore` 파일을 보자.

```
*.iml
.gradle
/local.properties
/.idea/caches
/.idea/libraries
/.idea/modules.xml
/.idea/workspace.xml
/.idea/navEditor.xml
/.idea/assetWizardSettings.xml
.DS_Store
/build
/captures
.externalNativeBuild
.cxx
local.properties
```

`local.properties`는 내 특정 기기에 대한 로컬 gradle 속성을 가지고 있다. 이 파일에서 API 키를 gradle 변수로 전달할 수 있다. 즉, API 키를 gralde 변수로 분리하여 관리할 수
있다.

`local.properties`에 `APK_KEY`를 선언한다.

```groovy
API_KEY = abc
```

그리고 앱 단 `build.gradle` 파일에서 `buildConfigField`를 통해 해당 API 키를 받아올 수 있다. 다음과 같이 `local.properties` 파일을
읽어 `buildConfigField`로 API 키를 전달한다.

```groovy
android {
    compileSdk 32

    defaultConfig {
        // ...

        Properties properties = new Properties()
        properties.load(project.rootProject.file("local.properties").newDataInputStream())

        buildConfigField "String", "API_KEY", "\"${properties.getProperty("API_KEY")}\""
    }
    // ...
}
```

프로젝트를 rebuild 한 후 `MainActivity`에서 다음과 같이 API를 얻어올 수 있다.

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val apiKey = BuildConfig.API_KEY
    }
}
```

애초에 안드로이드 앱에 포함되는 API는 민감한 정보이면 안 된다. 즉, 공격자가 API 키를 확보하더라도 이를 통해서 할 수 있는 것들이 제한적이여야 한다. 중요한 로직에 대한 API 키는 서버 측에 저장하고,
서버를 통해 수행되도록 해야 한다.

## References

* [How to Hide & Protect API Keys in Your Android App (Reverse Engineering)](https://www.youtube.com/watch?v=-2ckvIzs0nU&t=5s)