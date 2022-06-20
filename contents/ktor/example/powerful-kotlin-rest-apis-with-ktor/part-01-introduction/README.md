# Introduction

## What will you get with this course?

이 코스에서는 자체의 백엔드 서버를 통해 Ktor REST API를 사용한 안드로이드 노트 앱을 만들 것이다. 안드로이드 앱은 [Building the note app](https://github.com/beomsu317/study/tree/main/contents/android/example/powerful-kotlin-rest-apis-with-ktor)를 참고하자.

노트 앱은 로그인과 함께 회원가입하는 화면이 있으며, 로그인 후 markdown 형태의 노트를 작성할 수 있게 만들고, 이 노트에 대한 오너(등록된 사용자)를 추가할 수도 있게 할 것이다. 로그아웃도 구현한다.

여기서 중요한 기능 중 하나는 로컬 디비에 캐시하는 기능이다. 네트워크 연결이 없는 경우 로컬 디비에 캐시해놓고, 네트워크 연결된 후 Refresh 될 때 서버와 Sync 되는 기능이다.

이 코스를 통해 배우는 것은 다음과 같다.

- Ktor REST API를 이용한 유저 회원가입, 로그인 기능
- Ktor 백엔드를 서버에 배포하는 법
- 몽고디비를 이용한 효율적으로 원격에 데이터를 저장하는 방법
- 유저 비밀번호를 안전하게 저장하기 위해 API 통신을 암호화
- 안드로이드에서 로컬 디비 캐싱 구현
- 안드로이드에서 markdown을 파싱

## Why you should use Ktor as backend

Ktor 백엔드는 Node.js, Django와 같은 다른 백엔드 프레임워크에 비해 많이 알려지지 못했다. 이는 Ktor가 별로인 것이 아니라 코틀린이 Javascript 또는 파이썬과 같은 언어에 비해 상대적으로 인기가 많지 않아서이다.

하지만 안드로이드 개발자이고 코틀린을 사용한다면 백엔드 프레임워크로 Ktor는 좋은 선택이 될 것이다.

## Why you should choose Ktor

- 코틀린에 친숙하기 때문에 쉽게 사용할 수 있다.
- Ktor는 완전히 코루틴에 기반한다.
- setup 및 사용하기가 간단하다.
- 코틀린의 강력한 기능을 사용할 수 있다.
- 실제 Ktor 사용하는 사람들이 Ktor를 좋아한다.

## What you need to start with this course

이 코스를 위한 환경 설정을 수행한다. 이 코스에서 사용될 소프트웨어는 다음과 같다.

- Android Studio
    - [https://developer.android.com/studio](https://developer.android.com/studio)
- IntelliJ IDEA
    - [https://www.jetbrains.com/idea/download/#section=windows](https://www.jetbrains.com/idea/download/#section=windows)
- Mongo DB
    - [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
- Postman
    - [https://www.postman.com/downloads/](https://www.postman.com/downloads/)

## Which siklls should you have to get the maximum out of this course?

1. Kotlin
2. Android Fundamentals
3. Basic understanding of coroutines
4. Understanding the concept behind MVVM

## How a REST API works

- API란 두 개의 프로그램이 어떤 규칙으로 통신할지에 대한 명세이다.
- REST API는 이 규칙 중 하나이다.
- REST API는 통신하기 위해 URL을 이용한다.

## Post Request

- parameters are not visible in the URL
- Get requests to get data, POST requests to post data