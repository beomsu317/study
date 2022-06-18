# Serving static content

HTTP 엔드포인트를 만들든 웹사이트를 만들든 많은 애플리케이션은 파일(stylesheet, scripts, image 등)을 제공한다.

Ktor를 사용하여 파일의 콘텐트를 로드하고 요청에 대한 응답으로 보내는 것은 가능하지만, 이것이 일반적인 기능인 경우 Ktor는 static 플러그인을 사용해 전체 프로세스를 간소화한다.

static route를 정의하는 첫 번째 단계는 콘텐츠가 제공되어야 할 path를 정의하는 것이다. 예를 들어, `assets` route 하위에서 모든 static 콘텐츠가 다뤄진다면 다음과 같이 설정한다.

```kotlin
routing {
    static("assets") {

    }
}
```

다음 단계는 콘텐츠를 제공할 위치를 정의하는 것이다.

# Folders

폴더에서 콘텐츠를 제공하기 위해, `files` 함수를 사용해 폴더 이름을 지정해야 한다. path는 항상 애플리케이션 path에 상대 경로이다.

```kotlin
routing {
    static("assets") {
        files("css")
    }
}
```

`files("css")`는 `css` 폴더에 있는 모든 파일이 주어진 URL 패턴(`assets`)에서 static 콘텐츠로 제공되도록 허용한다. 이는 `/assets/stylesheet.css`
가 `/css/stylesheet.css`로 제공된다는 의미이다.

다양한 폴더를 하나의 path에 가질 수 있다. 다음 예제를 보자.

```kotlin
routing {
    static("assets") {
        files("css")
        files("js")
    }
}
```

## **Serving individual files**

폴더에서 파일을 제공하는 것 외에도, 파일 함수를 사용하여 개별 파일을 지정할 수 있다. 선택적으로 2번째 인자를 취급하며 물리적 파일 이름을 가상의 이름으로 매핑한다.

```kotlin
routing {
    static("static") {
        file("image.png")
        file("random.txt", "image.png")
    }
}
```

## **Serving all files including in subfolders**

`files` 함수는 `“.”` 문자를 취할 수 있는데, 이는 요청 path와 실제 파일 이름이 일치하는 한 모든 파일을 제공한다.

```kotlin
routing {
    static("static") {
        files(".")
    }
}
```

## **Defining a default file**

특정 path에서 기본 파일을 로딩하도록 정의할 수 있다.

```kotlin
routing {
    static("assets") {
        files("css")
        default("index.html")
    }
}
```

`/assets`에 요청하면 `index.html`을 제공한다.

## **Serving pre-compressed files**

Ktor는 미리 압축된 파일을 제공하고, 동적 압축을 사용하지 않는 기능을 제공한다. 예를 들어, `assets/html`에서 미리 압축된 파일을 제공하려면, `preCompressed` 함수 안에
있는 `files`를 호출해야 한다.

```kotlin
static("static") {
    preCompressed {
        files("assets/html")
    }
}
```

압축 타입의 우선 순위를 다른 압축 타입보다 높일 수도 있다.

```kotlin
static("static") {
    preCompressed(CompressedFileType.BROTLI, CompressedFileType.GZIP) {
        files("assets/html")
    }
}
```

`/static/index.html`로 요청이 들어온 경우, `assets/html/index.html.br`을 먼저 제공한다.

## **Changing the default root folder**

Ktor는 콘텐츠가 제공되는 다른 루트 폴더를 지정할 수 있는 기능을 제공한다. 이는 동적으로 정의하는 경우 유용하다. `staticRootFolder` 속성을 설정해 정의할 수 있다.

```kotlin
static("docs") {
    staticRootFolder = File("/system/folder/docs")
    files("public")
}
```

`/docs`로 요청이 오면 물리적인 폴더인 `/system/folder/docs/public`로 매핑된다.

# **Embedded Application Resources**

콘텐츠를 애플리케이션에 리소스로 포함하고 `resource` 및 `resources` 함수를 사용해 제공할 수 있다.

```kotlin
static("assets") {
    resources("css")
}
```

`resources("css")`는 리소스 `css` 하위에 있는 모든 파일이 주어진 URL 패턴(`assets`)에서 static 콘텐츠로 제공되도록 허용한다. `/assets/stylesheet.cs`
는 `/css/stylesheet.cs` 제공을 의미한다.

하나의 path에서 여러 릿스를 제공할 수 있다.

```kotlin
routing {
    static("assets") {
        resources("css")
        resources("js")
    }
}
```

## **Serving individual resources**

리소스 파일일 제공하는 것 외에도, `resource` 함수를 사용해 개별 파일을 지정할 수도 있다. 선택적으로 2번째 파라미터를 취급하여 물리적 파일 이름을 가상 이름으로 매핑할 수 있다.

```kotlin
routing {
    static("static") {
        resource("image.png")
        resource("random.txt", "image.png")
    }
}
```

## **Defining a default resource**

특정 path에 대해 로드할 기본 파일을 정의할 수도 있다.

```kotlin
routing {
    static("assets") {
        resources("css")
        defaultResource("index.html")
    }
}
```

## **Changing the default resource package**

Ktor는 콘텐츠가 제공되는 다른 기본 리소스 패키지를 지정할 수 있는 기능을 제공한다.

`staticBasePackage` 속성을 설정하여 이를 수행할 수 있다.

```kotlin
static("docs") {
    staticBasePackage = File("/system/folder/docs")
    files("public")
}
```

# Sub-routes

`static` 함수를 중첩해 sub-route를 만들 수 있다.

```kotlin
static("assets") {
    files("css")
    static("themes") {
        files("data")
    }
}
```

`/assets/themes`가 `/data`에서 파일을 로드하도록 허용한다.

# **Handling errors**

요청 콘텐츠가 없다면, Ktor는 자동적으로 `404 Not Found` HTTP status code를 응답한다.

# **Customising Content Type header**

Ktor는 확장자 기반으로 파일의 콘텐츠 타입을 자동으로 조회하고 적절한 `Content-Type` 헤더를 설정한다. 지원되는 MIME 타입 목록은 `ktor-server-core` 아티팩트에
있는 `mimelist.csv` 리소스 파일에 정의되어 있다.

# **Example**

다음은 폴더와 리소스를 모두 사용해 static 파일을 제공하는 예제이다.

```kotlin
fun Application.main() {
    install(DefaultHeaders)
    install(CallLogging)
    routing {
        routeFilesystem()
        routeResources()
    }
}

fun Route.routeFilesystem() {
    get("/") {
        call.respondHtml {
            head {
                title { +"Ktor: static-content" }
                styleLink("/static/styles.css")
                script(src = "/static/script.js") {}
            }
            body {
                p {
                    +"Hello from Ktor static content sample application"
                }
                p {
                    +"Current directory is ${System.getProperty("user.dir")}"
                }
                img(src = "static/image.png") {
                    onClick = "message('clicked the image')"
                }
            }
        }
    }
    static("static") {
        // When running under IDEA make sure that working directory is set to this sample's project folder
        staticRootFolder = File("files")
        files("css")
        files("js")
        file("image.png")
        file("random.txt", "image.png")
        default("index.html")
    }
    static("custom") {
        staticRootFolder = File("/tmp") // Establishes a root folder
        files("public") // For this to work, make sure you have /tmp/public on your system
        static("themes") {
            // services /custom/themes
            files("data")
        }
    }
}

fun Route.routeResources() {
    get("/resources") {
        call.respondHtml {
            head {
                title { +"Ktor: static-content" }
                styleLink("/static-resources/styles.css")
            }
            body {
                p {
                    +"Hello from Ktor static content served from resources, if the background is cornflowerblue"
                }
            }
        }
    }

    static("static-resources") {
        resources("css")
    }
}
```

## References

* [Serving static content | Ktor](https://ktor.io/docs/serving-static-content.html)