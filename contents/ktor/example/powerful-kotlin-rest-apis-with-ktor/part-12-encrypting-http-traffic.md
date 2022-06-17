# Encrypting HTTP traffic

이번엔 HTTP 트래픽을 암호화한 HTTPS를 사용해보자. 서버가 발급한 인증서를 통해서 HTTP 통신을 암/복호화 한다. 검증된 인증서만으로 통신할 수 있기 때문이다.

여기서는 자체적으로 인증서를 만들어 사용할 것이다. 이는 API 요청/응답에 대해 암호화한다.

## Create keystore

다음 명령을 이용해 자신의 keystore를 생성한다.

```bash
keytool -genkey -v -keystore mykey.jks -alias my_keystore -keyalg RSA -keysize 4096
```

생성한 keystore를 `build` 디렉토리에 넣어준다. 그리고 `application.conf` 파일에 `sslPort`를 지정해준다.

```bash
ktor {
    deployment {
        port = 8080
        sslPort = 8081
        port = ${?PORT}
    }
    application {
        modules = [ com.androiddevs.ApplicationKt.module ]
    }
    security {
        ssl {
            keyStore = build/mykey.jks
            keyAlias = my_keystore
            keyStorePassword = hackerman
            privateKeyPassword = hackerman
        }
    }
}
```