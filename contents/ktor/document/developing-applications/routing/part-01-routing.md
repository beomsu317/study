# Routing

Routing은 들어오는 요청을 처리하기 위한 Ktor의 코어 플러그인이다. 클라이언트가 특정 URL에 요청을 생성했을 때, 라우팅 메커니즘을 통해 이 요청을 제공하는 방법을 정의할 수 있다.

# **Install Routing**

Routing 플러그인은 다음 방식으로 설치된다.

```kotlin
install(Routing) {
    // ...
}
```

`Routing` 플러그인은 모든 애플리케이션에서 흔하게 사용되기 때문에, routing을 간단하게 설치하기 위한 `routing` 함수가 있다. 다음 코드에서 `install(Routing)`은 `routing`
함수로 대체된다.

```kotlin
routing {
    // ...
}
```

# **Define a route handler**

Routing 플러그인 설치 후 `routing` 안에서 route 함수를 호출하여 route를 정의할 수 있다.

```kotlin
routing {
    route("/hello", HttpMethod.Get) {
        handle {
            call.respondText("Hello")
        }
    }
}
```

Ktor는 간결하고 쉬운 route 핸들러를 정의할 수 있게 해주는 일련의 함수를 제공한다. 예를 들어, 이전 코드를 URL과 요청을 처리할 코드만 취급하는 `get` 함수로 대체할 수 있다.

```kotlin
routing {
    get("/hello") {
        call.respondText("Hello")
    }
}
```

유사하게 Ktor는 `put`, `post`, `head` 등의 메서드를 제공한다.

요약하자면, route를 정의하기 위해 다음 설정이 필요하다.

- HTTP verb
    - `GET`, `POST`, `PUT` 등과 같은HTTP 메서드를 선택한다. 가장 간편한 방법은 `get`, `post`, `put` 등과 같은 메서드 함수를 사용하는 것이다.
- Path pattern
    - URL path를 일치시키는데 사용되는 path pattern을 지정해라(`/hello`, `/customer/{id}`). `get` / `post` / etc 함수에 path pattern을 바로
      전달할 수 있다. 또는 `route` 함수를 사용해 route 핸들러를 그룹화하고, 중첩 route를 정의할 수 있다.
- Handler
    - 요청과 응답을 어떻게 처리할지 지정한다. 핸들러 내에서 `ApplicationCall`에 접근할 수 있으며, 클라이언트 요청을 처리하고 응답을 보낼 수 있다.

# **Specify a path pattern**

routing 함수(`route`, `get`, `post` 등)에 전달되는 Path pattern은 URL의 경로 컴포넌트에 매칭하기 위해 사용된다. path에는 `/`로 구분된 일려의 path segment가
포함될 수 있다.

> Ktor는 path 끝에 슬래시가 있는 path와 없는 path를 구별한다. `IgnoreTrailingSlash` 플러그인을 설치하여 동작을 변경할 수 있다.
>

다음은 몇 개의 path 예제이다.

- `/hello`
    - 단일 path segment를 포함하는 path.
- `/order/shipment`
    - 몇 개의 path segment를 포함하는 path. `route` / `get` / etc 함수에 이와 같은 path를 전달하거나 여러 route 함수를 중첩하여 sub-route를 구성하여.
- `/user/{login}`
    - path에 route 핸들러 내부에서 접근 가능한 `login` route 파라미터 전달.
- `/user/*`
    - 모든 path segment와 매칭되는 와일드카드 path.
- `/user/{...}`
    - 나머지 모든 URL path와 일치하는 tailcard가 있는 path.
- `/user/{param...}`
    - tailcard와 함께 route 파라미터를 포함하는 path.

## **Wildcard**

와일드카드(`*`)는 모든 path segment를 매칭한다. 예를 들어, `/user/*`는 `/user/john`과 매칭되지만, `/user`와는 매칭되지 않는다.

## Tailcard

tailcard(`{...}`)는 모든 나머지 URL path와 매칭되고, 여러 path segment를 포함할 수 있으며 비어있을 수 있다. 예를 들어, `/user/{...}`
는 `/user/john/settings` 및 `/user`와 매칭된다.

## **Route parameter**

route 파라미터(`{param}`) path segment와 매칭되고 `param`이라는 이름의 파라미터로 참조된다. 이 path segment는 필수적이지만, `?`를 추가해 선택적으로 만들 수
있다(`{param?}`).

- `/user/{login}`은 `/user/john`과 매칭되지만, `/user`와는 매칭되지 않는다.
- `/user/{login?}`는 `/user/john`, `/user`와 매칭된다.

route 핸들러 안에서 파라미터 값에 접근하기 위해 `call.parameters` 속성을 사용한다. 예를 들어, 다음 예제를 보면 `call.parameters[”login”]`을
사용하여 `/user/admin` path를 반환한다.

```kotlin
get("/user/{login}") {
    if (call.parameters["login"] == "admin") {
        // ...
    }
}
```

## **Route parameter with tailcard**

tailcard(`{param...}`)가 있는 route 파라미터는 모든 나머지 URL path와 매칭되고 파라미터를 키로 사용하여 각 path segment에 대한 여러 값을 파라미터에 넣는다. 예를
들어, `/user/{param...}`은 `/user/john/settings`과 매칭된다. route 핸들러 내부에서 path segment 값에 접근하기
위해 `call.parameters.getAll("param")`을 사용한다. 위 예제에서, `getAll` 함수는 `john`과 `settings` 값을 포함한 배열을 반환한다.

# **Define multiple route handlers**

여러 route 핸들러를 등록하고 싶으면 `routing` 함수에 추가할 수 있다.

```kotlin
routing {
    get("/customer/{id}") {

    }
    post("/customer") {

    }
    get("/order/{id}") {

    }
}
```

이 경우 각 route는 자체적인 함수를 가지며 지정된 엔드포인트에 응답한다.

다른 방법으로는 이 path들을 그룹화항 path를 정의한 다음 `route` 함수를 사용하여 해당 path에 대한 메서드를 중첩 함수로 배치하는 것이다.

```kotlin
routing {
    route("/customer") {
        get {

        }

        post {

        }
    }
}
```

그룹핑하는 것과 관계없이, Ktor는 또한 `route` 하기 위한 sub-route를 파라미터로 가질 수 있다. 다음 예제는 `/order/shipment` 요청에 대해 어떻게 처리할지 보여준다.

```kotlin
routing {
    route("/order") {
        route("/shipment") {
            get {

            }
            post {

            }
        }
    }
}
```

# **Route extension functions**

일반적인 패턴은 `Route` 타입에서 확장 함수를 사용해 실제 경로를 정의하는 것이, 쉽게 메서드에 접근할 수 있게 하고 단일 routing 함수에 모든 path를 포함하는 혼동을 제거한다. 이 패턴은 그룹화와
관계 없이 사용이 가능하다. 예를 들어, 첫 번째 예제는 다음과 같이 작성할 수 있다.

```kotlin
routing {
    customerByIdRoute()
    createCustomerRoute()
    orderByIdRoute()
    createOrder()
}

fun Route.customerByIdRoute() {
    get("/customer/{id}") {

    }
}

fun Route.createCustomerRoute() {
    post("/customer") {

    }
}

fun Route.orderByIdRoute() {
    get("/order/{id}") {

    }
}

fun Route.createOrder() {
    post("/order") {

    }
}
```

애플리케이션이 커짐에 따라 유지보수가 가능하게 하려면, [Structuring patterns](https://ktor.io/docs/structuring-applications.html)를 참고하자.

## References

* [Routing | Ktor](https://ktor.io/docs/routing-in-ktor.html)