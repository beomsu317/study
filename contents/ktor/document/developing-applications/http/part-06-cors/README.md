# CORS

서버가 [cross-origin requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)를 처리해야
한다면, [CORS](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-c-o-r-s/index.html) Ktor
플러그인을 설치하고 구성해야 한다. 이 플러그인을 사용하면 허용된 호스트, HTTP 메서드, 클라이언트가 설정한 헤더 등을 구성할 수 있다.

## Add dependencies

`CORS`를 사용하기 위해 `ktor-server-cors` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-cors:$ktor_version")
```

## **Install CORS**

`CORS` 플러그인을 설치하기 위해 `install` 함수에 플러그인을 전달한다. 서버를 생성하는 방법에 따라 2가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.cors.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(CORS)
        // ...
    }.start(wait = true)
}

```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.plugins.cors.*

// ...
fun Application.module() {
    install(CORS)
    // ...
}

```

## **Configure CORS**

CORS 관련 구성
설정은 [CORS.Configuration](https://api.ktor.io/ktor-server/ktor-server-core/ktor-server-core/io.ktor.features/-c-o-r-s/-configuration/index.html)
클래스에 의해 노출된다.

### Overview

`8080` 포트로 동작하고, `/customer` route로 JSON 데이터를 응답하는 서버가 있다고 가정하자. 다음 코드는 이 요청을 cross-origin 요청으로 만들기 위해 다른 포트에서 작업하는
클라이언트에서 Fetch API를 사용해 만든 샘플 요청을 보여준다.

```jsx
function saveCustomer() {
    fetch('http://0.0.0.0:8080/customer',
        {
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            method: "POST",
            body: JSON.stringify({id: 3, firstName: "Jet", lastName: "Brains"})
        })
        .then(response => response.text())
        .then(data => {
            console.log('Success:', data);
            alert(data);
        })
        .catch((error) => {
            console.error('Error:', error);
        });
}
```

이 요청을 백엔드에서 허용하기 위해, `CORS` 플러그인을 구성해야 한다.

```kotlin
install(CORS) {
    allowHost("0.0.0.0:8081")
    allowHeader(HttpHeaders.ContentType)
}
```

### **Hosts**

Cross-origin 요청을 할 수 있는 허용된 호스트를 지정하려면 `allowHost` 함수를 사용해야 한다. hostname 외에도 포트, 서브도메인 목록, 지원되는 HTTP 스키마를 지정할 수 있다.

```kotlin
install(CORS) {
    allowHost("client-host")
    allowHost("client-host:8081")
    allowHost("client-host", subDomains = listOf("en", "de", "es"))
    allowHost("client-host", schemes = listOf("http", "https"))
}
```

모든 호스트의 cross-origin 요청을 허용하려면 `anyHost` 함수를 사용한다.

```kotlin
install(CORS) {
    anyHost()
}
```

### **HTTP methods**

기본적으로 `CORS` 플러그인은 `GET`, `POST`, `HEAD` 메서드만 허용한다. 메서드를 추가하고 싶은 경우 `allowMethod` 함수를 사용한다.

```kotlin
install(CORS) {
    allowMethod(HttpMethod.Options)
    allowMethod(HttpMethod.Put)
    allowMethod(HttpMethod.Patch)
    allowMethod(HttpMethod.Delete)
}
```

### Allow headers

`CORS` 플러그인은 `Access-Control-Allow-Headers`로 처리되는 클라이언트 헤더를 기본적으로 허용한다.

- `Accept`
- `Accept-Language`
- `Content-Language`

헤더를 추가하고 싶다면 `allowHeader` 함수를 사용한다.

```kotlin
install(CORS) {
    allowHeader(HttpHeaders.ContentType)
    allowHeader(HttpHeaders.Authorization)
}
```

커스텀 헤더를 허용하기 위해 `allowHeaders` 또는 `allowHeadersPrefixed` 함수를 사용한다. 예를 들어, 다음 코드는 `custom-`으로 prefix 된 헤더를 허용하는 것을 보여준다.

```kotlin
install(CORS) {
    allowHeadersPrefixed("custom-")
}
```

### Expose headers

`Access-Control-Expose-Headers` 헤더는 브라우저의 JavaScript가 접근할 수 있는 허용 목록에 지정된 헤더를 추가한다. 이러한 헤더를 구성하려면 `exposeHeader` 함수를 사용한다.

```kotlin
install(CORS) {
    // ...
    exposeHeader("X-My-Custom-Header")
    exposeHeader("X-Another-Custom-Header")
}
```

### **Credentials**

기본적으로 브라우저는 크레덴셜 정보를 cross-origin 요청으로 전송하지 않는다. 이를 전달하는 것을 허용하려면, `allowCredentials` 속성을
사용해 `Access-Control-Allow-Credentials` 응답 헤더를 `true`로 설정해야 한다.

```kotlin
install(CORS) {
    allowCredentials = true
}
```

### **Miscellaneous**

`CORS` 플러그인은 CORS 관련 설정을 지정할 수 있게 해준다. 예를 들어, `maxAgeInSeconds`를 사용해 다른 실행 전 요청을 보내지 않고 실행 전 요청에 대한 응답을 캐싱할 수 있는 기간을 지정할
수 있다.

```kotlin
install(CORS) {
    maxAgeInSeconds = 3600
}
```

## References

* [CORS | Ktor](https://ktor.io/docs/cors.html#configure)