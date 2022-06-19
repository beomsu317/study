# Type-safe routing﻿

Ktor는 type-safe routing 구현할 수
있는 [Resources](https://api.ktor.io/ktor-shared/ktor-resources/io.ktor.resources/-resources/index.html?_gl=1*18y3k2k*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTYzNjk3OC4xMS4xLjE2NTU2NDE1MjEuMA..&_ga=2.122060747.1396641199.1655526702-658241611.1655526702)
플러그인을 제공한다. 이를 위해 타입이 지정된 route로 동작해야하는 클래스를 만들고 `@Resource` 키워드를 사용해 이 클래스에 어노테이션을 달아야 한다. 이러한 클래스에서는
kotlinx.serialization 라이브러리에서 제공하는 `@Serializable` 어노테이션도 있어야 한다.

## Add dependencies

### Add kotlinx.serialization

[resource classes](https://ktor.io/docs/type-safe-routing.html#resource_classes)에는 `@Serializable` 어노테이션이 있어야
하므로 [Setup](https://github.com/Kotlin/kotlinx.serialization#setup) 섹션에 설명된 대로 코틀린 serialization 플러그인을 추가해야 한다.

### Add Resources dependencies

`Resources`를 사용하기 위해 `ktor-server-resources` 아티팩트가 필요하다.

```kotlin
implementation("io.ktor:ktor-server-resources:$ktor_version")
```

## Install Resources

`Resources` 플러그인을 설치하기 위해, 이를 `install` 함수로 전달한다. 서버 생성 방법에 따라 2가지로 나뉜다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.resources.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Resources)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.server.application.*
import io.ktor.server.resources.*

// ...
fun Application.module() {
    install(Resources)
    // ...
}
```

## Create resource classes

각 리소스 클래스는 다음 어노테이션이 있어야 한다.

* `@Serializable` 어노테이션
* `@Resource` 어노테이션

아래에서 단일 경로 세그먼트 정의, 쿼리 및 경로 파라미터 등 몇 가지 예를 살펴본다.

### Resource URL

다음 예는 `/articles` 경로에 응답하는 리소스를 지정하는 `Articles` 클래스를 정의하는 방법을 보여준다.

```kotlin
import io.ktor.resources.*

@Serializable
@Resource("/articles")
class Articles()
```

### Resources with a query parameter

다음 `Articles` 클래스는 쿼리 파라미터로 동작하는 `sort` 문자열 속성을 가지고 있으며 이를 통해 `sort` 쿼리 파라미터 경로(`/articles?sort=new`)에 응답하는 리소스를 정의할 수
있다.

```kotlin
@Serializable
@Resource("/articles")
class Articles(val sort: String? = "new")
```

속성은 primitive 또는 `@Serializable` 어노테이션이 있는 타입이여야 한다.

### Resources with nested classes

여러 경로 세그먼트를 포함하는 리소스를 생성하기 위해 클래스를 중첩할 수 있다. 이 경우 중첩 클래스는 외부 클래스 타입의 속성이 있어야 한다. 다음 예제는 `/articles/new` 경로에 응답하는 리소스를
보여준다.

```kotlin
@Serializable
@Resource("/articles")
class Articles() {
    @Serializable
    @Resource("new")
    class New(val parent: Articles = Articles())
}
```

### Resources with a path parameter

다음 예제는 경로 세그먼트와 일치하는 중첩된 `{id}` 정수 경로 파라미터를 추가하고 이를 `id`라는 파라미터로 캡쳐하는 방법을 보여준다.

```kotlin
@Serializable
@Resource("/articles")
class Articles() {
    @Serializable
    @Resource("{id}")
    class Id(val parent: Articles = Articles(), val id: Long)
}
```

예를 들어, 이 리소스는 `/articles/12`에 응답하는데 사용할 수 있다.

### Example: A resource for CRUD operations

위 예제들을 요약하고 CRUD 작업을 위한 `Articles` 리소스를 생성해보자.

```kotlin
@Serializable
@Resource("/articles")
class Articles(val sort: String? = "new") {
    @Serializable
    @Resource("new")
    class New(val parent: Articles = Articles())

    @Serializable
    @Resource("{id}")
    class Id(val parent: Articles = Articles(), val id: Long) {
        @Serializable
        @Resource("edit")
        class Edit(val parent: Id)
    }
}
```

이 리소스는 모든 기사를 리스팅하거나, 새로운 기사를 작성, 수정 등에 사용할 수 있다.

## Define route handlers

타입이 지정된 리소스에 대한 route 핸들러를 정의하려면 리소스 클래스를 verb(`get`, `post`, `put` 등) 함수에 전달해야 한다. 예를 들어, 다음 route 핸들러는 `/articles` 경로에
대해 응답한다.

```kotlin
@Serializable
@Resource("/articles")
class Articles()

fun Application.module() {
    install(Resources)
    routing {
        get<Articles> {
            // Get all articles ...
            call.respondText("List of articles")
        }
    }
}
```

다음 예제는 [Example: A resource for CRUD operations](https://ktor.io/docs/type-safe-routing.html#example_crud)에서
생성한 `Articles` 리소스에 대한 route 핸들러를 정의하는 방법을 보여준다. Route 핸들러 내 `Article`을 파라미터로써 접근할 수 있고, 속성 값을 얻을 수 있다.

```kotlin
fun Application.module() {
    install(Resources)
    routing {
        get<Articles> { article ->
            // Get all articles ...
            call.respondText("List of articles sorted starting from ${article.sort}")
        }
        get<Articles.New> {
            // Show a page with fields for creating a new article ...
            call.respondText("Create a new article")
        }
        post<Articles> {
            // Save an article ...
            call.respondText("An article is saved", status = HttpStatusCode.Created)
        }
        get<Articles.Id> { article ->
            // Show an article with id ${article.id} ...
            call.respondText("An article with id ${article.id}", status = HttpStatusCode.OK)
        }
        get<Articles.Id.Edit> { article ->
            // Show a page with fields for editing an article ...
            call.respondText("Edit an article with id ${article.parent.id}", status = HttpStatusCode.OK)
        }
        put<Articles.Id> { article ->
            // Update an article ...
            call.respondText("An article with id ${article.id} updated", status = HttpStatusCode.OK)
        }
        delete<Articles.Id> { article ->
            // Delete an article ...
            call.respondText("An article with id ${article.id} deleted", status = HttpStatusCode.OK)
        }
    }
}
```

## References

* [Type-safe routing﻿](https://ktor.io/docs/type-safe-routing.html)