# Locations

Ktor는 URL 구성 및 파라미터 읽기 모두에 대해 타입이 지정된 방식으로 route를 생성하는 메커니즘을 제공한다.

## **Add dependencies**

`Locations` 지원을 활성화하려면 `ktor-locations` 아티팩트를 포함해야 한다.

```kotlin
implementation("io.ktor:ktor-locations:$ktor_version")
```

## **Install Locations**

`Locations` 플러그인을 설치하기 위해 `install` 함수에 전달한다. 서버를 생성하는 방법에 따라 `embeddedServer` 함수 호출은 다음과 같다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.locations.*
// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(Locations)
        // ...
    }.start(wait = true)
}
```

또는 지정된 `module`에 설치하는 방법은 다음과 같다.

```kotlin
import io.ktor.server.application.*
import io.ktor.server.locations.*
// ...
fun Application.module() {
    install(Locations)
    // ...
}
```

## **Define route classes**

처리하려는 각 타입의 route에 대해, 처리를 원하는 파라미터를 포함하는 클래스(보통 data class)를 생성해야 한다.

파라미터는 [Data Conversion](https://ktor.io/docs/data-conversion.html) 플러그인에서 지원하는 타입이여야 한다. 기본적으로 `Int`, `Long`, `Float`
, `Double`, `Boolean`, `String`, enums and `Iterable`을 사용할 수 있다.

### **URL parameters**

해당 클래스는 중괄호 `{`와 `}` 사이 placeholder와 일치하는 경로를 지정하는 `@Location`으로 어노테이션 되어 있어야 한다. 예를 들어, `{propertyName}`과 같이. 중괄호 사이
이름은 클래스의 속성과 일치해야 한다.

```kotlin
@Location("/list/{name}/page/{page}")
data class Listing(val name: String, val page: Int)
```

- `/list/movies/page/10`과 매치됨
- `Listing(name = "movies", page = 10)`로 생성됨

### **GET parameters**

`@Location`의 path의 부분이 아닌 추가 클래스 속성을 제공하고 싶다면, 해당 파라미터는 GET의 쿼리 문자열 또는 POST 파라미터에서 가져온다.

```kotlin
@Location("/list/{name}")
data class Listing(val name: String, val page: Int, val count: Int)
```

- `/list/movies?page=10&count=20`와 매치됨
- `Listing(name = "movies", page = 10, count = 20)`로 생성됨

## **Define route handlers**

`@Location`으로 어노테이트된 클래스를 정의한 후 플러그인 아티팩트는 route 핸들러를 정의하기 위한 새로운 타입의 메서드를 노출한다. `get`, `options`, `header`, `post`
, `put`, `delete`, `patch`.

```kotlin
routing {
    get<Listing> { listing ->
        call.respondText("Listing ${listing.name}, page ${listing.page}")
    }
}
```

## **Build URLs**

`@Location` 어노테이션이 달린 클래스의 인스턴스로 `application.locations.href`를 호출하여 route에 대한 URL을 구성할 수 있다.

```kotlin
val path = application.locations.href(Listing(name = "movies", page = 10, count = 20))
```

`path`는 `/list/movies?page=10&count=20`이 된다.

```kotlin
@Location("/list/{name}")
data class Listing(val name: String, val page: Int, val count: Int)
```

만약 이러한 URL을 구성하고 URL 포맷을 변경을 결정했다면, `@Location` path만 업데이트하면 되므로 편리하다.

## **Subroutes with parameters**

다음과 같이 `@Location` 어노테이션이 달린 다른 클래스를 참조하는 클래스를 생성하고 등록해야 한다.

```kotlin
routing {
    get<Type.Edit> { typeEdit -> // /type/{name}/edit
        // ...
    }
    get<Type.List> { typeList -> // /type/{name}/list/{page}
        // ...
    }
}
```

상위 location에 정의된 파라미터를 얻기 위해, 내부 route에 대한 클래스에 해당 속성 이름을 포함하기만 하면 된다.

```kotlin
@Location("/type/{name}") data class Type(val name: String) {
    // In these classes we have to include the `name` property matching the parent.
    @Location("/edit") data class Edit(val parent: Type)
    @Location("/list/{page}") data class List(val parent: Type, val page: Int)
}
```

## References

* [Locations | Ktor](https://ktor.io/docs/locations.html)