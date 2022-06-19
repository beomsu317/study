# Configuring HTTP/2

HTTP/2는 최신의 binary duplex multiplexing protocol이다.

Jetty, Netty, Tomcat 엔진은 Ktor가 사용할 수 있는 HTTP/2 구현을 제공한다. 그러나 큰 차이가 있으며, 엔진 별 추가 설정이 필요하다. host가 Ktor에 맞게 적절히 구성되면,
HTTP/2 지원이 자동적으로 활성화된다.

주요 필요사항:

- SSL certificate (can be self-signed)
- 특정 엔진에 대한 적절한 ALPN 구현

## **SSL certificate**

사양에 따라 HTTP/2는 암호화가 필요하지 않지만, 모든 브라우저는 HTTP/2 사용하기 위한 암호화 연결이 필요하다. 이러한 이유로 TLS 환경이 요구된다. 그러므로 암호화를 위한 인증서가 필요하다. 테스트
목적으로 JDK에서 `keytool`을 통해 생성할 수 있다.

```bash
keytool -keystore test.jks -genkeypair -alias testkey -keyalg RSA -keysize 4096 -validity 5000 -dname 'CN=localhost, OU=ktor, O=ktor, L=Unspecified, ST=Unspecified, C=US'
```

또는 [generateCertificate](https://ktor.io/docs/ssl.html) 함수를 사용해서도 생성할 수 있다.

다음 단계는 키스토어를 사용하기 위한 Ktor 구성이다. 다음 `application.conf`를 보자.

```
ktor {
    deployment {
        port = 8080
        sslPort = 8443
    }

    application {
        modules = [ com.example.ApplicationKt.main ]
    }

    security {
        ssl {
            keyStore = test.jks
            keyAlias = testkey
            keyStorePassword = foobar
            privateKeyPassword = foobar
        }
    }
}
```

## **ALPN implementation**

HTTP/2 활성화하기 위해
ALPN([Application-Layer Protocol Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation))이
필요하다. 첫 번째 옵션은 외부의 boot classpath에 추가해야 하는 ALPN 구현을 사용하는 것이다. 다른 옵션은 OpenSSL 네이티브 바인딩과 미리 컴파일된 네이티브 바이너리를 사용하는 것이다. 또한 각
특정 엔진은 이러한 방법 중 하나만 지원할 수 있다.

### **Jetty**

ALPN API는 Java 8 부터 지원하기 때문에, Jetty 엔진은 HTTP/2를 사용하기 위한 설정이 필요하지 않다. 다음 것만이 필요하다.

1. Jetty 엔진 서버 생성
2. SSL 구성 추가
3. `sslPort` 설정

[http2-jetty](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/http2-jetty) 예제는 Jetty에 대한
HTTP/2 지원을 보여준다.

### Netty

Netty에서 HTTP/2를 활성화하는 쉬운 방법은 OpenSSL 바인딩을 사용하는 것이다. API jar를 디펜던시에 추가한다.

```bash
implementation "io.netty:netty-tcnative:$tcnative_version"
```

그리고 네이티브 구현(BoringSSL 라이브러리에 정적으로 링킹되어 있음, Open SSL의 fork)

```bash
implementation "io.netty:netty-tcnative-boringssl-static:$tcnative_version"
implementation "io.netty:netty-tcnative-boringssl-static:$tcnative_version:$tcnative_classifier"
```

`tc.native.classifier`는 다음 중 하나여야 한다. `linux-x86_64`, `osx-x86_64`, `windows-x86_64`
. [http2-netty](https://github.com/ktorio/ktor-documentation/tree/main/codeSnippets/snippets/http2-netty) 예제는 Netty에 대한
HTTP/2 지원을 보여준다.

## References

* [Configuring HTTP/2 | Ktor](https://ktor.io/docs/advanced-http2.html)
