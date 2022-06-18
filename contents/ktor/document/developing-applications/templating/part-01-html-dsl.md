# HTML DSL

HTML DSL은 [kotlinx.html](https://github.com/Kotlin/kotlinx.html) 라이브러리에 포함되어 있고, HTML 블록으로 응답할 수 있도록 도와준다. HTML DSL로 완전한
HTML을 작성할 수 있고, 변수에 view를 삽입하고, 템플릿을 이용해 복잡한 HTML 레이아웃을 만들 수 있다.

```kotlin
implementation("io.ktor:ktor-html-builder:$ktor_version")
```

# **Add dependencies**

HTML DSL은 설치가 필요없지만 `ktor-html-builder` 아티팩트가 필요하다. 다음을 통해 디펜던시를 추가한다.

```kotlin
implementation("io.ktor:ktor-html-builder:$ktor_version")
```

# **Send HTML in response**

HTML 응답을 하기
위해 [respondHtml](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/respond-html.html)
메서드를 사용한다.

```kotlin
routing {
    get("/") {
        val name = "Ktor"
        call.respondHtml(HttpStatusCode.OK) {
            head {
                title {
                    +name
                }
            }
            body {
                h1 {
                    +"Hello from $name!"
                }
            }
        }
    }
}
```

위 코드의 경우 다음 HTML이 클라이언트에 보여진다.

```html

<head>
    <title>Ktor</title>
</head>
<body>
<h1>Hello from Ktor!</h1>
</body>
```

# **Templates**

일반적인 HTML을 생성하는 것 외에도, Ktor는 복잡한 레이아웃을 빌드할 수 있는 템플릿 엔진을 제공한다. HTML 페이지의 다른 부분에 대한 템플릿 계층을 만들 수 있다. 예를 들어, 전체 페이지를 위한 루트
템플릿, 페이지 헤더, 푸터를 위한 자식 템플릿 등등. Ktor는 템플릿 작업을 위해 다음 API를 노출한다.

1. 특정 템플릿을 기반으로 빌드한 HTML을 응답하기
   위해 [respondHtmlTemplate](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/respond-html-template.html)
   메서드를 사용한다.
2. 탬플릿을 만들기
   위해 [Template](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/-template/index.html)
   인터페이스를
   구현하고, [Template.apply](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/-template/apply.html)
   메서드를 오버라이딩 해야 한다.
3. 생성된 템플릿 클래스 안에서 다양한 콘텐츠 타입을 위한 placeholder를 정의할 수 있다.
    - [Placeholder](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/-placeholder/index.html)
      는 콘텐츠를 삽입하기 위해
      사용된다. [PlaceholderList](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/-placeholder-list/index.html)
      는 여러 아이템을 보여주는 콘텐츠를 삽입하기 위해 사용될 수 있다(예: 리스트 아이템 ).
    - [TemplatePlaceholder](https://api.ktor.io/ktor-features/ktor-html-builder/ktor-html-builder/io.ktor.html/-template-placeholder/index.html)
      는 자식 템플릿을 삽입하고 중첩된 레이아웃을 생성하는데 사용된다.

## **Example**

다음 코드가 있다고 가정하자.

```html

<body>
<h1>Ktor</h1>
<article>
    <h2>Hello from Ktor!</h2>
    <p>Kotlin Framework for creating connected systems.</p>
</article>
</body>
```

이 페이지를 2개의 부분의 나눌 수 있다.

- 페이지 헤더를 위한 루트 레이아웃 템플릿과 기사를 위한 자식 템플릿.
- 기사 콘텐츠를 위한 자식 템플릿.

2. `respondHtmlTemplate` 메서드를 호출하고 템플릿 클래스를 파라미터로 전달한다. 우리의 경우 `Template` 인터페이스를 구현해야하는 `LayoutTemplate` 클래스이다.

```kotlin
get("/") {
    call.respondHtmlTemplate(LayoutTemplate()) {
        // ...
    }
}
```

블럭 안에서, 템플릿에 접근할 수 있고 속성 값을 지정할 수 있다. 이 값은 템플릿 클래스에 지정된 placeholder 값을 대체한다. 다음 단계에 `LayoutTemplate`을 생성하고, 속성을 정의할 것이다.

3. 루트 레이아웃 템플릿은 다음과 같다.

```kotlin
class LayoutTemplate : Template<HTML> {
    val header = Placeholder<FlowContent>()
    val content = TemplatePlaceholder<ContentTemplate>()
    override fun HTML.apply() {
        body {
            h1 {
                insert(header)
            }
            insert(ContentTemplate(), content)
        }
    }
}
```

이 클래스는 2개의 속성을 노출한다.

- `header` 속성은 `h1` 태그 내 삽입된 콘텐츠를 지정한다.
- `content` 속성은 기사 콘텐츠를 위한 자식 템플릿을 지정한다.

4. 자식 템플릿은 다음과 같다.

```kotlin
class ContentTemplate : Template<FlowContent> {
    val articleTitle = Placeholder<FlowContent>()
    val articleText = Placeholder<FlowContent>()
    override fun FlowContent.apply() {
        article {
            h2 {
                insert(articleTitle)
            }
            p {
                insert(articleText)
            }
        }
    }
}
```

이 템플릿은 `article` 내 삽입될 값인, `articleTitle`과 `articleText` 속성을 노출한다.

4. 이제 지정된 속성 값을 사용해 HTML 응답을 빌드할 수 있다.

```kotlin
get("/") {
    call.respondHtmlTemplate(LayoutTemplate()) {
        header {
            +"Ktor"
        }
        content {
            articleTitle {
                +"Hello from Ktor!"
            }
            articleText {
                +"Kotlin Framework for creating connected systems."
            }
        }
    }
}
```

## References

* [HTML DSL | Ktor](https://ktor.io/docs/html-dsl.html)