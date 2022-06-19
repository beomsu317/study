# SSL and certificates

대부분의 경우 Ktor 서비스들은 Nginx 또는 Apache 같은 reverse proxy 뒤에 존재한다. 이는 reverse proxy 서버가 SSL을 포함한 보안 문제를 처리하기 위함이다.

Ktor 서버가 SSL을 직접 제공하려면, Ktor는 [Java KeyStore (JKS)](https://docs.oracle.com/javase/8/docs/api/java/security/KeyStore.html)를 인증서를 위한 저장 시설로 사용한다. [keytool](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)을 사용해 키스토어에 저장된 인증서를 변환하고 관리할 수 있다. 이는 인증 기관에서 발급한 PEM 인증서를 Ktor에서 지원하는 JKS 포맷으로 변환해야 하는 경우 유용할 수 있다.

> *Let's Encrypt*를 사용해 무료 인증서를 사용할 수 있다. 

## Generate a self-signed certificate

### Generate a certificate in code

Ktor는 [KeyStore](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/security/KeyStore.html) 인스턴스를 반환하는 [generateCertificate](https://api.ktor.io/ktor-network/ktor-network-tls/ktor-network-tls-certificates/io.ktor.network.tls.certificates/generate-certificate.html?_gl=1*zqo5xg*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyNjcwMS4xLjEuMTY1NTUyNzA5OS4w&_ga=2.191689962.1396641199.1655526702-658241611.1655526702) 함수를 호출해 테스팅 목적의 자체적으로 서명된 인증서를 제공할 수 있다.

```kotlin
implementation("io.ktor:ktor-network-tls-certificates:$ktor_version")
```

다음 코드는 어떻게 인증서를 생성하고 keystore 파일에 저장하는지를 보여준다.

```kotlin
fun main() {
    val keyStoreFile = File("build/keystore.jks")
    val keystore = generateCertificate(
        file = keyStoreFile,
        keyAlias = "sampleAlias",
        keyPassword = "foobar",
        jksPassword = "foobar"
    )
}
```

Ktor를 시작할 때 인증서가 필요하기 때문에, 서버 시작 전 인증서를 생성해야 한다. 

### Generate a certificate using keytool

[keytool](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)을 사용해 자체 서명된 인증서를 생성할 수 있다.

```shell
keytool -keystore keystore.jks -alias sampleAlias -genkeypair -keyalg RSA -keysize 4096 -validity 3 -dname 'CN=localhost, OU=ktor, O=ktor, L=Unspecified, ST=Unspecified, C=US'
```

위 커맨드 수행 후 `keytool`은 keystore 비밀번호를 지정한 다음 JKS 파일을 생성하도록 제안한다.

## Convert PEM certificates to JKS

인증 기고나이 PEM 포맷의 인증서를 발행하는 경우, Ktor에서 SSL 설정하기 전 이를 JKS 포맷으로 변환해야 한다. `openssl`과 `keytool` 유틸을 사용해 변환할 수 있다. 예를 들어, `key.pem` 파일에 개인키를, 공개키 인증서가 `cert.pem`에 있는 경우 변환 과정은 다음과 같다.

1. `openssl`을 사용해 PEM을 PKCS12 포맷으로 변환한다. 

```shell
openssl pkcs12 -export -in cert.pem -inkey key.pem -out keystore.p12 -name "sampleAlias"
```

`key.pem`에 대한 암호와 `keystore.p12`에 대한 새 암호를 입력하라는 프롬프트가 표시된다.

2. `keytool`을 사용해 PKCS12를 JKS로 변환한다.

```shell
keytool -importkeystore -srckeystore keystore.p12 -srcstoretype pkcs12 -destkeystore keystore.jks
```

`keystore.p12` 파일에 대한 암호와 `keystore.jks`를 위한 새 암호를 입력하라는 프롬프트가 표시된다. 이후 `keystore.jks`가 생성된다.

## Configure SSL in Ktor

Ktor에서 SSL 설정을 지정하는 것은 Ktor 서버를 설정하는 방법에 달려있다. 설정 파일의 `application.conf`를 사용하거나, 코드에서 `embeddedServer` 함수를 사용해서 설정하는 방법이 있다.

### HOCON file

서버 설정을 위해 `application.conf`를 사용한다면, 다음 속성을 사용해 SSL을 활성화한다.

1. SSL 포트를 `ktor.deployment.sslPort` 속성을 사용해 지정한다.

```
ktor {
    deployment {
        sslPort = 8443
    }
}
```

2. 별도의 `security` 그룹에 keystore 설정을 제공한다.

```
ktor {
    security {
        ssl {
            keyStore = keystore.jks
            keyAlias = sampleAlias
            keyStorePassword = foobar
            privateKeyPassword = foobar
        }
    }
}
```

### embeddedServer

서버 설정을 위해 `embeddedServer` 함수를 사용한다면, [custrom environment](https://ktor.io/docs/configurations.html#embedded-custom)를 전달하고, [sslConnector](https://api.ktor.io/ktor-server/ktor-server-host-common/io.ktor.server.engine/ssl-connector.html?_gl=1*1rg84d9*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTUyNjcwMS4xLjEuMTY1NTUyNzA5OS4w&_ga=2.192361450.1396641199.1655526702-658241611.1655526702)를 사용해 SSL 설정을 제공해야 한다. 

## References

* [SSL and certificates](https://ktor.io/docs/ssl.html)