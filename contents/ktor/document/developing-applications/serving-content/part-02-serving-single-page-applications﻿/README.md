# Serving single-page applications

Ktor는 React, Angular 또는 Vue를 포함한 single-page 애플리케이션을 제공하는 기능을 제공한다.

## Add dependencies

single-page 애플리케이션을 제공하기 위해 `ktor-server-core` 디펜던시만 필요하다. 다른 디펜던시를 필요하지 않다.

## Serve an application

Single-page 애플리케이션을 제공하기 위해 로컬 파일시스템 또는 classpath에서 콘텐츠를 제공할 위치를 정의해야 한다. 최소한 single-page 애플리케이션이 포함될 폴더/리소스 패키지를 지정해야
한다.

### Serve framework-specific applications

React, Angular, Vue 등과 같은 특정 프레임워크를 사용해 생성된 single-page 애플리케이션의 빌드를 제공할 수 있다. React 애플맄에ㅣ션이 포함된 프로젝트 루트에 `react-app` 폴더가
있다고 가정하자. 이 애플리케이션은 다음과 같은 구조이며, `index.html`이 메인 페이지이다.

```
react-app
├── index.html
├── ...
└── static
    └── ...
```

애플리케이션을 제공하기 위해 routing 블록
내 [singlePageApplication](https://api.ktor.io/ktor-server/ktor-server-core/io.ktor.server.http.content/single-page-application.html?_gl=1*1vnldx8*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTY0NjgwMC4xMi4xLjE2NTU2NDcyMDcuMA..&_ga=2.168705249.1396641199.1655526702-658241611.1655526702)
를 호출하고 `react` 함수에 폴더 이름을 전달한다.

Ktor는 자동으로 `index.html`를 찾는다.

### Customize serving settings

리소스에서 single-page 애플리케이션을 제공하는 방법을 보여주기 위해 애플리케이션이 `sample-web-app` 리소스 패키지에 위치한다고 가정한다.

```
sample-web-app
├── main.html
├── ktor_logo.png
├── css
│   └──styles.css
└── js
    └── script.js
```

애플리케이션을 제공하기 위해 다음 설정이 사용된다.

```kotlin
routing {
    singlePageApplication {
        useResources = true
        filesPath = "sample-web-app"
        defaultPage = "main.html"
        ignoreFiles { it.endsWith(".txt") }
//            react("react-app")
    }
}
```

* `useResources` : 리소스 패키지의 애플리케이션 제공 활성화
* `filesPath` : 애플리케이션이 있는 경로를 지정
* `defaultPage` : `main.html`을 제공한 default 리소스로 지정
* `ignoreFiles` : 마지막에 `.txt`를 포함한 경로 무시 

## References

* [Serving single-page applications](https://ktor.io/docs/serving-spa.html)