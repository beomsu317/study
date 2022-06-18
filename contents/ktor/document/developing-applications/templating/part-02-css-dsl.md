# CSS DSL

CSS DSL은 HTML DSL의 확장이며,
코틀린에서 [kotlin-css](https://github.com/JetBrains/kotlin-wrappers/blob/master/kotlin-css/README.md) 래퍼를 통해 작가의 스타일시트를
허용한다.

# **Add dependencies**

CSS DSL은 설치가 필요 없으며, 다음 아티팩트 포함이 필요하다.

1. `ktor-html-builder` 아티팩트를 추가한다.

```kotlin
implementation("io.ktor:ktor-html-builder:$ktor_version")
```

2. `kotlin-css-jvm` 아티팩트를 추가한다.

```kotlin
implementation("org.jetbrains.kotlin-wrappers:kotlin-css:$kotlin_css_version")
```

# **Use CSS DSL**

CSS 응답을 보내기 위해, 스타일시트를 문자열로 serialize 하기 위한  `respondCss` 메서드를 추가하여 `ApplicationCall`을 확장해야 하고, `CSS` 콘텐츠 타입으로 클라이언트에
전송해야 한다.

```kotlin
suspend inline fun ApplicationCall.respondCss(builder: CssBuilder.() -> Unit) {
    this.respondText(CssBuilder().apply(builder).toString(), ContentType.Text.CSS)
}
```

그런 다음 필요한 경로 내 CSS를 제공할 수 있다.

```kotlin
get("/styles.css") {
    call.respondCss {
        body {
            backgroundColor = Color.darkBlue
            margin(0.px)
        }
        rule("h1.page-title") {
            color = Color.white
        }
    }
}
```

최종적으로, HTML DSL로 만든 HTML 문서에 지정된 CSS를 사용할 수 있다.

```kotlin
get("/html-dsl") {
    call.respondHtml {
        head {
            link(rel = "stylesheet", href = "/styles.css", type = "text/css")
        }
        body {
            h1(classes = "page-title") {
                +"Hello from Ktor!"
            }
        }
    }
}
```

## References

* [CSS DSL | Ktor](https://ktor.io/docs/css-dsl.html)