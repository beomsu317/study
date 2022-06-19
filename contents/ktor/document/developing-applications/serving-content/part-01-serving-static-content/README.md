# Serving static content

HTTP 엔드포인트를 만들든 웹사이트를 만들든 많은 애플리케이션은 파일(stylesheet, scripts, image 등)을 제공한다. Ktor를 사용하여 파일의 콘텐트를 로드하고 요청에 대한 응답으로 보내는 것은
가능하지만, 이것이 일반적인 기능인 경우 Ktor는 static 플러그인을 사용해 전체 프로세스를 간소화한다. static route를 정의하는 첫 번째 단계는 콘텐츠가 제공되어야 할 path를 정의하는 것이다.
예를 들어, `assets` route 하위에서 모든 static 콘텐츠가 다뤄진다면 다음과 같이 설정한다.

```kotlin
routing {
    static("assets") {

    }
}
```

다음 단계는 콘텐츠를 제공할 위치를 정의하는 것이다.

* [Folders](https://ktor.io/docs/serving-static-content.html#folders) - 로컬 파일 시스템에서 static 파일을 어떻게 제공하는 방법을 설명한다. 이 경우
  상대 경로는 현재 작업 디렉토리를 사용하여 해결한다.
* [Embedded application resources](https://ktor.io/docs/serving-static-content.html#resources) - 클래스 경로에서 static 파일을
  제공하는 방법을 설명한다.

## Folders

폴더에서 콘텐츠를 제공하기 위해 샘플 프로젝트의 루트에 `files` 디렉토리가 있다고 가정한다. 이 디렉토리에는 다음 파일이 포함되어 있다.

```
files
├── index.html
├── ktor_logo.png
├── css
│   └──styles.css
└── js
    ├── script.js
    └── script.js.gz
```

이 섹션에서 이러한 파일을 제공하는 두 가지 경우를 고려한다.

* 물리적인 경로와 일치하는 URL 경로를 사용해 모든 파일을 재귀적으로 제공
* 커스터마이징 된 URL 경로를 사용해 파일/폴더 제공

|Physical path|URL path matches physical path|URL path is customized|
|:---:|:--:|:--------------------:|
|`files/index.html` | `/index.html` |  `/index.html or /`  |
|`files/ktor_logo.png` | `/ktor_logo.png` |  `/images/ktor_logo.png`  |
|`files/css/styles.css` | `/css/styles.css` |  `/assets/styles.css`  |
|`files/js/script.js(.gz)` | `/js/script.js` |  `/assets/script.js`  |

> Ktor는 확장자 기반으로 파일의 콘텐츠 타입을 자동으로 조회하고 적절한 `Content-Type` 헤더를 설정한다.

### Change the default root folder

기본적으로 Ktor는 현재 작업 디렉토리에서 static 파일을 제공하기 위한 경로를 계산한다. 만약 앱의 static 파일이 특정 폴더에 있는 경우 `staticRootFolder` 속성을 통해 콘텐츠가 제공되는
루트 폴더를 설정할 수 있다. 프로젝트 루트에 있는 `files` 폴더에 대한 구성은 다음과 같다.

```kotlin
static("/") {
    staticRootFolder = File("files")
}
```

이는 `/`에 대한 모든 요청을 물리적인 `files` 폴더로 매핑한다. 다음 단계는 `file` 또는 `files` 함수를 이용해 static 파일을 제공하는 방법을 지정해야 한다.

### Serve all files including in subfolders

`files` 폴더에 있는 모든 파일을 재귀적으로 제공하려면 `"."` 문자를 `files` 함수에 전달해야 한다.

```kotlin
static("/") {
    staticRootFolder = File("files")
    files(".")
}
```

이 경우 Ktor는 URL 경로와 실제 파일 이름이 일치하는 한 `files` 내 모든 파일을 제공한다. 다음은 URL 경로를 커스터마이징하는 방법을 알아본다.

### Serve individual files

개별 파일을 제공하려면 `file` 함수를 사용한다. 예를 들어, `files/index.html` 파일을 제공하려면, 다음과 같이 설정해야 한다.

```kotlin
static("/") {
    staticRootFolder = File("files")
    file("index.html")
}
```

`Routing` 플러그인의 경우 `static` 함수를 중첩하여 sub-route를 정의할 수 있다. 다음 예제에서는 `/images` URL 경로 아래의 `ktor_logo.png` 파일을 제공하는 방법을
보여준다.

```kotlin
static("/") {
    staticRootFolder = File("files")
    static("images") {
        file("ktor_logo.png")
        file("image.png", "ktor_logo.png")
    }
}
```

`file` 함수는 선택적으로 물리적인 파일 이름을 가상 파일에 매핑할 수 있는 두 번째 인자를 취한다. 위 예에 대해, `ktor_logo.png` 이미지는 다음 경로에 대한 요청에 대해 제공된다.

* `/images/ktor_logo.png`
* `/images/image.png`

### Define a default file

특정 경로에 대해 `default` 함수를 사용해 로드할 기본 파일을 정의할 수도 있다. 다음 코드는 deafult 파일로 `index.html`을 정의한 것이다.

```kotlin
static("/") {
    staticRootFolder = File("files")
    default("index.html")
}
```

이 경우 `/`에 대한 요청에 대해 `files/index.html`를 제공한다.

### Serve content from a folder

개별 파일을 제공하는 것 뿐만 아니라, 폴더에서 콘텐츠를 제공할 수 있다. 이를 위해 `files` 함수를 사용해 폴더 이름을 지정해야 한다. 다음 예제는 stylesheet와 script를 제공하는 방법을
보여준다.

```kotlin
static("/") {
    staticRootFolder = File("files")
    static("assets") {
        files("css")
        files("js")
    }
}
```

`files("css")`는 `css` 폴더에 있는 모든 파일이 주어진 URL 패턴(이 경우 `assets`)에서 static 콘텐츠로 제공되도록 허용한다. 이는 `/assets/styles.css`에 대한
요청이 `files/css/styles.css` 파일을 제공한다는 의미이다.

### **Serving pre-compressed files**

Ktor는 미리 압축된 파일을 제공하고, [dynamic compression](https://ktor.io/docs/compression.html)을 사용하지 않는 기능을 제공한다. 예를 들어, `css`
, `js` 폴더에서 미리 압축된 파일을 제공하기 위해 `preCompressed` 함수 내에서 `files`를 호출한다.

```kotlin
static("assets") {
    preCompressed {
        files("css")
        files("js")
    }
}
```

압축 타입의 우선 순위를 다른 압축 타입보다 높일 수도 있다. 다음 예제에선 `*.br` 파일을 `*.gz`보다 먼저 제공하려 한다.

```kotlin
static("assets") {
    preCompressed(CompressedFileType.BROTLI, CompressedFileType.GZIP) {
        files("css")
        files("js")
    }
}
```

예를 들어, `/assets/script.js`로 요청이 있을 경우 Ktor는 `js/script.js.br`를 먼저 제공하려 한다.

## **Embedded Application Resources**

애플리케이션 리소스에서 static 파일을 제공하는 방법을 보여주기 위해 샘플 프로젝트의 `resources` 디렉토리에 `static` 패키지가 있다고 가정하자. 이 패키지엔 다음 파일이 포함되어 있다.

```
static
├── index.html
├── ktor_logo.png
├── css
│   └──styles.css
└── js
    └── script.js
```

이 섹션에서 이러한 파일을 제공하는 두 가지 경우를 고려한다.

* 물리적인 경로와 일치하는 URL 경로를 사용해 모든 파일을 재귀적으로 제공
* 커스터마이징 된 URL 경로를 사용해 파일/폴더 제공

|Physical path|URL path matches physical path|URL path is customized|
|:---:|:--:|:--------------------:|
|`files/index.html` | `/index.html` |  `/index.html or /`  |
|`files/ktor_logo.png` | `/ktor_logo.png` |  `/images/ktor_logo.png`  |
|`files/css/styles.css` | `/css/styles.css` |  `/assets/styles.css`  |
|`files/js/script.js(.gz)` | `/js/script.js` |  `/assets/script.js`  |

> Ktor는 확장자 기반으로 파일의 콘텐츠 타입을 자동으로 조회하고 적절한 `Content-Type` 헤더를 설정한다.

### **Changing the default resource package**

기본적으로 Ktor는 리소스 루트 디렉토리에서 static 리소스를 제공하기 위한 경로를 계산한다. 앱의 static 파일이 지정된 리소스 패키지 내 저장되어 있는 경우, `staticBasePackage` 속성을
사용해 콘텐츠가 제공되는 기본 패키지로 설정할 수 있다. 예를 들어, `resources` 폴더 내부의 `static` 패키지의 경우 구성은 다음과 같다.

```kotlin
static("/") {
    staticBasePackage = "static"
}
```

이는 `/`에 대한 모든 요청이 `static` 패키지로 매핑된다는 의미이다. 다음 단계에서 `resource` 또는 `resources` 함수를 통해 static 리소스를 제공하는 방법을 지정해야 한다.

### Serve all resources including in subfolders

`static` 폴더 내 모든 파일을 재귀적으로 제공하기 위해 `"."` 문자를 `resources` 함수에 전달한다.

```kotlin
static("/") {
    staticBasePackage = "static"
    resources(".")
}
```

이 경우 Ktor는 URL 경로와 실제 파일 이름이 일치하는 한 `static`에서 모든 파일을 제공한다. 다음 장에서는 URL 경로를 커스터마이징하는 방법을 알아본다.

### Serve individual resources

개별 리소스를 제공하기 위해 `resource` 함수를 사용한다. 예를 들어, `static/index.html` 파일을 제공하려면 다음과 같이 구성해야 한다.

```kotlin
static("/") {
    staticBasePackage = "static"
    resource("index.html")
}
```

`Routing` 플러그인의 경우 `static` 함수를 중첩하여 sub-route를 정의할 수 있다. 다음 예는 `/images` URL 경로 아래의 `ktor_logo.png` 파일이 제공하는 방법을 보여준다.

```kotlin
static("/") {
    staticBasePackage = "static"
    static("images") {
        resource("ktor_logo.png")
        resource("image.png", "ktor_logo.png")
    }
}
```

`resource` 함수는 선택적으로 물리적인 파일 이름을 가상 파일에 매핑할 수 있는 두 번째 인자를 취한다. 위 예에 대해, `ktor_logo.png` 이미지는 다음 경로에 대한 요청에 대해 제공된다.

* `/images/ktor_logo.png`
* `/images/image.png`

Ktor는 확장자를 기반으로 파일의 콘텐츠 타입을 자동으로 조회하고 적절한 `Content-Type` 헤더를 설정한다.

### Define a default resource

특정 경로에 대해 `defaultResource` 함수를 사용해 default 리소스가 로딩되도록 정의할 수 있다. 다음 예는 `index.html`을 default 리소스로 정의하는 방법을 보여준다.

```kotlin
static("/") {
    staticBasePackage = "static"
    defaultResource("index.html")
}
```

이 경우 `/` 요청에 대해 Ktor는 `static/index.html`를 제공한다.

### Serve content from a resource folder

개별 파일을 제공하는 것 외에도, 리소스 폴더의 컨텐츠를 제공할 수 있다. 이를 위해 `resources` 함수를 사용해 리소스 폴더 이름을 지정해야 한다. 다음 예는 샘플 프로젝트의 stylesheet과
script를 제공하는 방법을 보여준다.

```kotlin
static("/") {
    staticBasePackage = "static"
    static("assets") {
        resources("css")
        resources("js")
    }
}
```

`resources("css")`는 `css` 리소스 폴더에 있는 모든 파일이 주어진 URL 패턴(이 경우 `assets`)에서 static 콘텐츠로 제공되도록 허용한다. 이는 `/assets/styles.css`에
대한 요청이 `files/css/styles.css`를 제공한다는 의미이다.

## Handle errors

요청 콘텐츠가 없다면, Ktor는 자동으로 `404 Not Found` HTTP 상태 코드를 반환한다.

## Examples

다음 예제 앱은 폴더와 리소스를 모두 사용해 static 파일을 제공한다.

```kotlin
package com.example

import io.ktor.server.application.*
import io.ktor.server.http.content.*
import io.ktor.server.routing.*
import java.io.*

fun Application.module() {
    routing {
        static("/") {
            staticRootFolder = File("files")
            file("index.html")
            default("index.html")
            static("images") {
                file("ktor_logo.png")
                file("image.png", "ktor_logo.png")
            }
            static("assets") {
                files("css")
                files("js")
            }
        }
    }
}
```

## References

* [Serving static content | Ktor](https://ktor.io/docs/serving-static-content.html)