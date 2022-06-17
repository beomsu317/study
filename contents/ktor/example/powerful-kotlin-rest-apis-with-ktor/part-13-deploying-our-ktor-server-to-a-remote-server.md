# Deploying our Ktor server to a remote server

이번엔 Ktor 서버를 배포해보는 작업을 해보자. 우분투(20.04) 서버를 기반으로 배포한다.

## Ktor

Ktor 서버를 배포하기 위해 모든 디펜던시, 코들 등을 가진 .jar 파일을 만들어야 한다. 우리의 서버가 인증서를 가지고 있지만 앞에 선언한 build/ 경로를 가지고 있지 않으므로, 이 경로를 변경해주어야
한다. `application.conf` 파일의 `keyStore`을 다음과 같이 변경해준다.

```
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
            keyStore = /home/mykey.jks
            keyAlias = my_keystore
            keyStorePassword = hackerman
            privateKeyPassword = hackerman
        }
    }
}
```

`build.gradle`을 열어 다음 `jar` 블럭을 추가해주면 옆에 화살표가 생긴다.

```groovy
// ...
jar {
    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
    manifest {
        attributes(
                'Implementation-Title': 'Ktor Server',
                "Main-Class": mainClassName
        )
    }
    destinationDirectory = file("$rootDir/release")
    archivesBaseName = 'app'
}
// ...
```

이 화살표를 누르면 `release` 디렉토리 하위에 Ktor 서버에 대한 패키지 .jar 파일이 생성된다. 이제 이 .jar 파일와 인증서 파일을 서버로 옮기기만 하면 된다.

## Deploy server

.jar 파일과 인증서를 옮긴 후 `/home` 디렉토리에 위치시킨다. .jar를 실행시키기 위한 java를 설치해준다.

```bash
apt-get install openjdk-8-jdk
```

다음 서버에 MongoDB를 설치해준다. Ubuntu에 MongoDB를 설치하는 방법은 다음과
같다. [여기에](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/) 가이드되어 있다.

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | apt-key add -
```

```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

```bash
apt-get update
```

```bash
apt-get install -y mongodb-org
```

MongoDB 설치가 완료됐으면 다음 명령을 통해 실행시킨다.

```bash
systemctl start mongod
```

### Register service

`/etc/systemd/system` 디렉토리에 서비스 `notes.service`를 생성 및 작성한다.

```
[Unit]
Description=Notes Service
After=mongod.service
StartLimitIntervakSec=0

[Service]
Type=simple
RestartSec=1
Restart=always
User=root
ExecStart=/usr/bin/java -jar /home/app-0.0.1.jar

[Install]
WantedBy=multi-user.target
```

다음 명령을 통해 서비스를 실행한다.

```bash
systemctl start notes
```

서버가 정상적으로 실행되었는지 확인한다.

```bash
systemctl status notes
```

실행된 서비스의 모든 로그를 보여준다.

```bash
journalctl -u notes.service
```

만약 방화벽 때문에 포트가 막혀있다면 ufw를 설치해 해결할 수 있다. 다음 명령을 통해 incoming 패킷을 허용해준다.

```bash
ufw status
```

```bash
ufw enable
```

```bash
ufw allow 8002
```