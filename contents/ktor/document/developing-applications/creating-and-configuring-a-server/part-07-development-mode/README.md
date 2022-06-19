# Development mode

Ktor는 개발을 위한 특별한 모드를 제공한다. 이는 다음 기능을 제공한다.

- 서버 재시작 없이 애플리케이션 클래스 자동 리로드 기능
- 디버깅 파이프라인을 위한 추가 정보
- 5** 서버 에러의 경우 응답 페이지에 대한 추가 디버깅 정보

## **Enable development mode**

개발 모드를 다음과 같은 다양한 방법으로 활성화할 수 있다. 애플리케이션 설정 파일, 환경 변수 또는 시스템 속성을 사용하여.

### **application.conf**

`application.conf` 파일에 `development` 옵션을 `true`로 설정하여 활성화한다.

```
ktor {
    development = true
}
```

### **The 'io.ktor.development' system property**

`io.ktor.development` 시스템 속성을 통해 개발 모드 활성화한다.

- IntelliJ IDEA를 통해 개발 모드로 실행하고 싶은 경우 `-D` 플래그와 같이 `io.ktor.development`를 VM 옵션에 전달한다.

```
-Dio.ktor.development=true
```

- 애플리케이션이 Gradle task를 사용하면, `applicationDefaultJvmArgs` 속성에 `io.ktor.development`를 전달하여 활성화한다.

```kotlin
application {
    applicationDefaultJvmArgs = listOf("-Dio.ktor.development=true")
}
```

### **The 'io.ktor.development' environment variable**

네이티브 클라이언트에 개발 모드를 활성화하려면, `io.ktor.development` 환경 변수를 사용한다.

## References

* [Development mode | Ktor](https://ktor.io/docs/development-mode.html)