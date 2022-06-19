# Creating an interactive website

이 튜토리얼 시리즈는 Ktor를 통해 간단한 블로그를 생성하는 방법을 보여준다.

* 우선 HTML 페이지와 이미지 같은 static content를 호스팅하는 것을 보여준다.
* 다음 튜토리얼에서 FreeMarker 템플릿 엔진을 사용해 interactive 한 앱을 만들어본다.
* 마지막으로 Exposed 프레임워크를 사용해 웹사이트에 persistense를 추가한다.

## Adjust FreeMarker configuration

IntelliJ IDEA를 위한 Ktor 플러그인은 이미 `plugins/Templating.kt` 파일에 FreeMarker 플러그인용 코드를 생성했다.

```kotlin
import freemarker.cache.*
import io.ktor.server.application.*
import io.ktor.server.freemarker.*

fun Application.configureTemplating() {
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(this::class.java.classLoader, "templates")
    }
}
```

`templateLoader` 설정은 앱에게 FreeMarker 템플릿이 `templates` 디렉토리에 위치한다는 것을 알려준다. `outputFormat`도 추가한다.

```kotlin
import freemarker.cache.*
import freemarker.core.*
import io.ktor.server.application.*
import io.ktor.server.freemarker.*

fun Application.configureTemplating() {
    install(FreeMarker) {
        templateLoader = ClassTemplateLoader(this::class.java.classLoader, "templates")
        outputFormat = HTMLOutputFormat.INSTANCE
    }
}
```

`outputFormat` 설정은 유저가 제공한 control characters를 HTML entities로 변환하는데 도움이 된다. 이는 저널 항목 중 `<b>Hello</b>` 같은 문자열이 포함되어 있을 때
실제로 **Hello**가 아닌 `<b>Hello</b>`로 출력된다. 이러한 이스케이핑은 XSS 공격을 방지하는데 필수적인 단계이다.

## Create a model

우선 기사를 표현하는 모델을 생성해야 한다. `com.example` 패키지에 `models` 패키지를 생성하고 `Article.kt` 파일을 해당 패키지에 생성한다. 그리고 다음과 같이 작성한다.

```kotlin
package com.example.models

import java.util.concurrent.atomic.AtomicInteger

class Article
private constructor(val id: Int, var title: String, var body: String) {
    companion object {
        private val idCounter = AtomicInteger()

        fun newEntry(title: String, body: String) = Article(idCounter.getAndIncrement(), title, body)
    }
}
```

기사는 3개의 속성(`id`, `title`, `body`)을 갖는다. `title`과 `body` 속성은 직접 지정할 수 있지만, `id`는 `AtomicInteger`(2개의 기사는 동일한 `id`를 가질 수
없도록 하는 thread-safe한 데이터 구조)를 사용해 자동으로 생성된다.

`Article.kt`에서 기사를 저장하기 위한 mutable list를 생성하고, 몇 개의 엔트리를 추가한다.

```kotlin
val articles = mutableListOf(
    Article.newEntry(
        "The drive to develop!",
        "...it's what keeps me going."
    )
)
```

## Define routes

이제 저널을 위한 route를 정의할 준비가 되었다. `com/example/plugins/Routing.kt` 파일을 열고 `configureRouting`을 다음과 같이 구현한다.

```kotlin
fun Application.configureRouting() {
    routing {
        // ...
        get("/") {
            call.respondRedirect("articles")
        }
        route("articles") {
            get {
                // Show a list of articles
            }
            get("new") {
                // Show a page with fields for creating a new article
            }
            post {
                // Save an article
            }
            get("{id}") {
                // Show an article with a specific id
            }
            get("{id}/edit") {
                // Show a page with fields for editing an article
            }
            post("{id}") {
                // Update or delete an article
            }
        }
    }
}
```

이 코드는 다음과 같이 동작한다.

* `get("/")` 핸들러는 `/` 경로에 대한 모든 `GET` 요청을 `/articles`로 리다이렉션한다.
* `route("articles")` 핸들러는 [group routes](https://ktor.io/docs/routing-in-ktor.html#multiple_routes)로 사용되며 다양한 액션을 수행한다.
  기사의 목록을 보여주거나, 새로운 기사 추가 등. 예를 들어, 파라미터가 없는 중첩된 `get` 함수는 `/articles` 경로에 대한 `GET` 요청에 응답하지만 `get("new")`
  는 `/articles/new`에 대한 `GET` 요청에 응답한다.

## Show a list of articles

우선 `/articles` URL 경로에 접근했을 때 모든 기사 목록을 보여주도록 구현해보자.

### Serve the templated content

`com/example/plugins/Routing.kt`을 열고 `get` 핸들러에 다음 코드를 추가한다.

```kotlin
import com.example.models.*
import io.ktor.server.application.*
import io.ktor.server.freemarker.*
import io.ktor.server.http.content.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.util.*

fun Application.configureRouting() {
    routing {
        route("articles") {
            get {
                call.respond(FreeMarkerContent("index.ftl", mapOf("articles" to articles)))
            }
        }
    }
}
```

`call.respond` 함수는 클라이언트에 전송되는 콘텐츠를 나타내는 `FreeMarkerContent` 객체를 받는다. 이 경우 `FreeMarkerContent` 생성자는 2개의 파라미터를 받는다.

* `template`은 `FreeMarker` 플러그인으로부터 로드할 템플릿 이름이다. `index.ftl` 파일은 아직 존재하지 않지만 곧 생성할 것이다.
* `model`은 템플릿을 렌더링 중에 전달할 데이터 모델이다. 이 경우 `articles` 템플릿 변수 내
  이미 [생성된 기사 목록](https://ktor.io/docs/creating-interactive-website.html#model)를 전달한다.

### Create a template

FreeMarker 플러그인은 `templates` 디렉토리에 위치하는 템플릿을 로드하도록 구성한다. 우선 `resources` 내 `templates` 디렉토리를 생성한다.
그리고 `resources/templates`에 `index.ftl` 파일을 생성한 후 다음과 같이 작성한다.

```html
<#-- @ftlvariable name="articles" type="kotlin.collections.List
<com.example.models.Article>" -->
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <title>Kotlin Journal</title>
    </head>
    <body style="text-align: center; font-family: sans-serif">
    <img src="/static/ktor_logo.png">
    <h1>Kotlin Ktor Journal </h1>
    <p><i>Powered by Ktor & Freemarker!</i></p>
    <hr>
    <#list articles?reverse as article>
    <div>
        <h3>
            <a href="/articles/${article.id}">${article.title}</a>
        </h3>
        <p>
            ${article.body}
        </p>
    </div>
    </
    #list>
    <hr>
    <p>
        <a href="/articles/new">Create article</a>
    </p>
    </body>
    </html>
```

메인 코드 중 일부를 알아보자.

* `@ftlvariable`이 있는 주석은 `List<Article>` 타입의 기사라는 변수를 선언한다. 이 주석은 IntelliJ IDEA가 `articles` 템플릿 변수에 의해 노출된 속성을 아는데 도움이
  된다.
* 다음 파트는 저널의 헤더 요소가 포함되어 있다. (로고와 헤드)
* `list` 태그 내에서 모든 기사들에 대해 반복하고 이에 대한 콘텐츠를 보여준다. 기사의 타이틀은 특정 기사(`/articles/${article.id}` 경로)에 대한 링크로 렌더링된다. 특정 기사를 보여주는
  페이지는 추후에 [Show a created article](https://ktor.io/docs/creating-interactive-website.html#show_article)에서 구현될 것이다.
* 하단의 링크는 새로운 기사를 만들기 위해 `/articles/new`로 이동한다.

이제 앱을 실행하고 저널의 메인 페이지를 확인할 수 있다.

### Refactor a template

앱의 모든 페이지에 로고와 헤더를 보여주기 위해 `index.ftl`을 리팩토링하고 공통의 코드를 별도의 템플릿으로 나눈다. 이는 FreeMarker의 매크로를 통해 수행할 수
있다. `resources/templates/_layout.ftl` 파일을 만들고 다음과 같이 작성한다.

```html
<#macro header>
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Kotlin Journal</title>
</head>
<body style="text-align: center; font-family: sans-serif">
<img src="/static/ktor_logo.png">
<h1>Kotlin Ktor Journal </h1>
<p><i>Powered by Ktor & Freemarker!</i></p>
<hr>
<#nested>
<a href="/">Back to the main page</a>
</body>
</html>
</#macro>
```

그리고 `index.ftl` 파일을 `_layout.ftl`를 재사용하도록 업데이트한다.

```html
<#-- @ftlvariable name="articles" type="kotlin.collections.List
<com.example.models.Article>" -->
    <#import "_layout.ftl" as layout />
    <@layout.header>
    <#list articles?reverse as article>
    <div>
        <h3>
            <a href="/articles/${article.id}">${article.title}</a>
        </h3>
        <p>
            ${article.body}
        </p>
    </div>
</#list>
<hr>
<p>
    <a href="/articles/new">Create article</a>
</p>
</@layout.header>
```

## Create a new article

이제 `/articles/new` 경로에 대한 요청을 처리할 수 있다. `Routing.kt`를 열고 다음 코드를 `get("new")`에 추가한다.

```kotlin
get("new") {
    call.respond(FreeMarkerContent("new.ftl", model = null))
}
```

여기서는 새로운 기사가 아직 존재하지 않으므로 데이터 모델 없이 `new.ftl` 템플릿으로 응답한다.

```html
<#import "_layout.ftl" as layout />
<@layout.header>
<div>
    <h3>Create article</h3>
    <form action="/articles" method="post">
        <p>
            <input type="text" name="title">
        </p>
        <p>
            <textarea name="body"></textarea>
        </p>
        <p>
            <input type="submit">
        </p>
    </form>
</div>
</@layout.header>
```

`new.ftl` 템플릿은 기사 콘텐츠를 제출할 수 있는 폼을 제공한다. 이 폼이 `POST` 요청의 데이터를 `/articles` 경로로 보내는 경우 폼 파라미터를 읽고 스토리지에 새 기사를 추가하는 핸들러를
구현해야 한다. `Routing.kt` 파일로 돌아가 `post` 핸들러에 다음 코드를 추가한다.

```kotlin
post {
    val formParameters = call.receiveParameters()
    val title = formParameters.getOrFail("title")
    val body = formParameters.getOrFail("body")
    val newEntry = Article.newEntry(title, body)
    articles.add(newEntry)
    call.respondRedirect("/articles/${newEntry.id}")
}
```

`call.receiveParameters` 함수는 폼 파라미터와 값을 받기 위해 사용된다. 새 기사를 저장한 후 `call.respondRedirect`를 호출해 이 기사를 보여주도록 한다. 특정 기사의 URL
경로는 런타임에 값을 가져와야 하는 ID 파라미터가 포함되어 있다. 다음 챕터에서 경로 파라미터를 어떻게 취급하는지 알아본다.

## Show a created article

특정 기사의 내용을 보여주기 위해 기사 ID를 경로 파라미터로 사용한다. `Routing.kt`의 `get("{id}")`를 다음과 같이 작성한다.

```kotlin
get("{id}") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    call.respond(FreeMarkerContent("show.ftl", mapOf("article" to articles.find { it.id == id })))
}
```

`call.parameters`는 URL로 전달된 기사 ID를 얻기 위해 사용된다. 이 ID의 기사를 보여주기 위해 저장소에서 기사를 찾아 `article` 템플릿 변수에 전달해야 한다.

`resources/templates/show.ftl` 템플릿을 생성한 후 다음과 같이 작성한다.

```html
<#-- @ftlvariable name="article" type="com.example.models.Article" -->
<#import "_layout.ftl" as layout />
<@layout.header>
<div>
    <h3>
        ${article.title}
    </h3>
    <p>
        ${article.body}
    </p>
    <hr>
    <p>
        <a href="/articles/${article.id}/edit">Edit article</a>
    </p>
</div>
</@layout.header>
```

이 페이지 하단의 `/articles/${article.id}/edit` 링크는 이 기사를 수정하거나 삭제할 수 있는 폼을 제공한다.

## Edit or delete an article

기사를 수정하는 route는 다음과 같다.

```kotlin
get("{id}/edit") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    call.respond(FreeMarkerContent("edit.ftl", mapOf("article" to articles.find { it.id == id })))
}
```

기사를 보여주는 route와 유사하게 `call.parameters`를 사용하여 기사의 ID를 얻어오고 저장소에서 해당 기사를 찾는다.

`resources/templates/edit.ftl`을 생성하고 다음과 같이 작성한다.

```html
<#-- @ftlvariable name="article" type="com.example.models.Article" -->
<#import "_layout.ftl" as layout />
<@layout.header>
<div>
    <h3>Edit article</h3>
    <form action="/articles/${article.id}" method="post">
        <p>
            <input type="text" name="title" value="${article.title}">
        </p>
        <p>
            <textarea name="body">${article.body}</textarea>
        </p>
        <p>
            <input type="submit" name="_action" value="update">
        </p>
    </form>
</div>
<div>
    <form action="/articles/${article.id}" method="post">
        <p>
            <input type="submit" name="_action" value="delete">
        </p>
    </form>
</div>
</@layout.header>
```

HTML 폼은 `PATCH`와 `DELETE` verbs를 지원하지 않기 때문에, 위 페이지는 2개(수정/삭제)의 분리된 폼이 필요하다. 서버 측에선 `name`과 `value` 속성을 확인해 이러한 폼에서
보낸 `POST` 요청을 구별할 수 있다.

`Routing.kt` 파일을 열고 `post("{id}")`에 다음과 같이 작성한다.

```kotlin
post("{id}") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    val formParameters = call.receiveParameters()
    when (formParameters.getOrFail("_action")) {
        "update" -> {
            val index = articles.indexOf(articles.find { it.id == id })
            val title = formParameters.getOrFail("title")
            val body = formParameters.getOrFail("body")
            articles[index].title = title
            articles[index].body = body
            call.respondRedirect("/articles/$id")
        }
        "delete" -> {
            articles.removeIf { it.id == id }
            call.respondRedirect("/articles")
        }
    }
}
```

이 코드는 다음과 같이 동작한다.

* `call.parameters`는 수정할 기사의 ID를 얻어온다.
* `call.receiveParameters`는 유저가 수행할 작업(`update` 또는 `delete`)을 가져오는데 사용한다.
* 작업에 따라 기사는 저장소에서 업데이트되거나 삭제된다.

## Run the application

앱이 예상대로 동작하는지 보자. 앱 실행 후 `http://localhost:8080/`로 접근하고 기사 생성, 수정, 삭제를 시도해보자. 

<div align="center">
<img src="img/result.png" width="80%">
</div>

만약 서버를 종료시키게 되면 모든 저장된 기사들은 없어지게 된다. 다음 튜토리얼에서 기사를 지속하는 방법에 대해 알아본다.

## References

* [Creating an interactive website](https://ktor.io/docs/creating-interactive-website.html)