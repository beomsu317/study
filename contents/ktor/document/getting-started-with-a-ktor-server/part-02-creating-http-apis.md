# Creating HTTP APIs

이번 실습을 통해, 코틀린을 사용하여 HTTP API를 만들어보자. routes가 어떻게 구조되어 있는지, serialization 플러그인이 테스크를 어떻게 간소화하는지, 애플리케이션의 일부를 어떻게 수동 또는
자동으로 테스트하는지 알아보자.

# **What we will build**

가상 비즈니스의 고객에 대한 JSON API를 응답하도록 구현해볼 것이다.

시스템의 모든 고객 및 주문을 리스팅하며, 각각 고객 및 주문에 대한 정보를 얻고, 새로운 엔트리 추가 및 오래된 엔트리를 제거하는 기능을 제공하는 것을 편리한 방법으로 빌드한다.

routes를 정의하고, 이를 파일 기반으로 구성하는 2가지 방법을 사용한다. 애플리케이션에서 routes를 정의하는 유일한 방법은 아니지만, 유지 가능한 접근 방식을 보여준다.

[template project](https://github.com/ktorio/ktor-http-api-sample/)를 클론한다.

# **Project setup**

템플릿 프로젝트는 이 프로젝트에 필요한 기본적인 Gradle 디펜던시를 가지고 있다. 따라서 따로 디펜던시를 추가하지 않아도 된다.

## Dependencies

build.gradle 파일의 디펜던시를 보자.

```groovy
dependencies {
    implementation "io.ktor:ktor-server-core:$ktor_version"
    implementation "io.ktor:ktor-server-netty:$ktor_version"
    implementation "ch.qos.logback:logback-classic:$logback_version"
    implementation "io.ktor:ktor-serialization:$ktor_version"

    testImplementation "io.ktor:ktor-server-test-host:$ktor_version"
    testImplementation "org.jetbrains.kotlin:kotlin-test"
}
```

- ktor-server-core : Ktor의 코어 컴포넌트를 추가한다.
- ktor-server-netty : Netty 엔진 추가한다. 별다른 외부 애플리케이션 컨테이너에 의존 없이 서버 기능을 사용할 수 있게 해준다.
- logback-classic : 포맷된 로그를 콘솔에 보여준다.
- ktor-serialization : 코틀린 오브젝트를 JSON, vice versa와 같은 serialized form으로 편리한 변환을 제공한다.
- ktor-server-test-host : 전체 HTTP 스택을 사용하지 않고도 Ktor 애플리케이션의 일부를 테스트할 수 있게 해준다. 유닛 테스트를 수행할 것이다.

## **Configurations: application.conf and logback.xml**

resources 디렉토리에 HOCON 포맷의 application.conf를 가지고 있다. Ktor는 이 파일을 통해 어떤 포트로 실행할지, 엔트리 포인트가 어딘지 결정한다.

동일한 폴더에 logback.xml 파일이 있는데, 서버의 기본적인 로깅 구조를 설정한다.

## Entry point

application.conf에서 애플리케이션의 엔트리 포인트가 `com.jetbrains.handson.httpapi.ApplicationKt.module`로 설정되어 있다. 이는 Application.kt
파일의 `Application.module()` 함수와 일치한다. 지금은 아무것도 구현되어 있지 않다.

```kotlin
fun main(args: Array<String>): Unit = io.ktor.server.netty.EngineMain.main(args)

fun Application.module() {

}
```

이 엔트리 포인트는 Ktor 플러그인을 설치하고 API를 위한 routing을 정의하기 때문에 중요한 부분이다.

# **Customer routes**

애플리케이션의 Customer 사이드를 다뤄보자. 고객과 관련된 데이터 모델을 생성해야 한다. 또한 고객이 추가, 리스팅, 제거되는 엔드포인트가 필요하다.

## **The Customer model**

고객은 몇 가지 기본적인 정보를 텍스트 형식으로 저장해야 한다. 고객은 식별하기 위한 `id`와 이름, 성 그리고 이메일 주소를 가지고 있다. 코틀린의 data class를 사용해 쉽게 생성할 수 있다.

models 패키지를 생성하고 Customer.kt 파일을 생성한다.

```kotlin
@kotlinx.serialization.Serializable
data class Customer(
    val id: String,
    val firstName: String,
    val lastName: String,
    val email: String
)
```

[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)의 `@Serializable` 어노테이션를 사용한다. 이는 자동으로 JSON 표현을
API 응답으로 생성할 수 있게 해준다.

먼저 이러한 고객을 위치시킬 장소를 정의해야 한다.

## **Storing customers**

코드를 복잡하게 하지 않기 위해 in-memory storage를 이용한다. 실제 앱에선 DB를 사용하기 때문에 Ktor를 재시작해도 데이터를 잃지 않는다. Customer.kt 파일에 다음 코드를 추가한다.

```kotlin
val customerStorage = mutableListOf<Customer>()
```

`Customer` 클래스와 이 객체에 대한 저장소를 생성했다. 이제 엔드포인트를 만들고 API를 통해 노출시켜보자.

## **Defining the routing for customers**

`/customer` 엔드포인트에서 GET, POST, DELETE 요청에 응답해야 한다. HTTP 메서드에 상응하는 route를 만들자. CustomerRoutes.kt 파일을 새로운 routes 패키지에
생성한다.

```kotlin
fun Route.customerRouting() {
    route("/customer") {
        get {

        }
        get("{id}") {

        }
        post {

        }
        delete("{id}") {

        }
    }
}
```

`route` 함수를 사용해 `/customer` 엔드포인트에 대해 그룹핑했다. 그리고 각 메서드별로 블럭을 생성했다. 이런 방식으로 route를 정의한다. `Order` route를 다룰 때는 다른 방식을
사용해본다.

`get`에 대해 2가지 방법으로 응답한다. 하나는 파라미터 없이, 다른 하나는 `{id}`를 전달한다. 전자는 모든 고객 리스트를 보여주고, 후자는 특정 고객만 보여준다.

## **Listing all customers**

모든 고객을 보여주기 위해 `customerStorage` 리스트를 `call.respond` 함수를 통해 반환한다. 코틀린 객체를 지정된 포맷으로 serialized 하여 반환한다. `get` 핸들러는 다음과
같다.

```kotlin
fun Route.customerRouting() {
    route("/customer") {
        get {
            if (customerStorage.isNotEmpty()) {
                call.respond(customerStorage)
            } else {
                call.respondText("No customers found", status = HttpStatusCode.NotFound)
            }
        }
        // ...
    }
}
```

이 작업을 수행하기 위해 Ktor의 [content negotiation](https://ktor.io/docs/serialization.html) 활성화가 필요하다. content negotiation은 어떤
역할을 할까? 다음 요청을 생각해보자.

```
GET http://0.0.0.0:8080/customer
Accept: application/json
```

클라이언트가 요청을 생성할 때 content negotiation은 서버가 해당 컨텐트의 타입을 제공할 수 있는지 확인하기 위해 `Accept` 헤더를 검사하는 것을 허락한다.

`ContentNegotiation` 플러그인을 설치하고 JSON을 제공하도록 활성화한다. `Application.module()` 함수에 다음과 같이 추가한다.

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
}
```

JSON은 kotlinx.serialization[kotlinx.serialization](https://ktor.io/docs/kotlin-serialization.html)에 의해
제공된다. `@Serializable` 어노테이션을 `Customer` 클래스에 추가했는데, 이는 Ktor가 `Customer` 클래스를 어떻게 serialize 하는지를 알려준다.

## **Returning a specific customer**

다른 route는 ID를 통해 특정 고객을 반환하는 route이다.

```
GET http://0.0.0.0:8080/customer/200
Accept: application/json
```

Ktor에서는 path는 특정 path의 segment와 일치하는 파라미터를 포함할 수 있다. 이 파라미터를 indexed access operator(`call.parameters["myParamName"]`)로
접근할 수 있다. `get(”{id}”)` 블럭을 다음과 같이 작성한다.

```kotlin
get("{id}") {
    val id = call.parameters["id"] ?: return@get call.respondText(
        "Missing or malformed id",
        status = HttpStatusCode.BadRequest
    )
    val customer =
        customerStorage.find { it.id == id } ?: return@get call.respondText(
            "No customer with id $id",
            status = HttpStatusCode.NotFound
        )
    call.respond(customer)
}
```

우선 파라미터 `id`가 존재하는지를 검증한다. 존재하지 않는다면 400(Bad Request) 상태 코드와 에러 메시지를 반환한다. 파라미터가 존재하면 `customerStorage`에 있는 일치하는
레코드를 `find`로 확인한다. 찾은 경우 해당 객체를 반환하며 없을 경우 404(Not Found) 상태 코드와 에러 메시지를 반환한다.

400(Bad Request)는 `id`가 null 인 경우에 반환되지만 이 경우는 실제로 발생해서는 안된다. 왜냐? 이는 파라미터 `{id}`가 전달되지 않는 경우에만 발생되기 때문이다. 그러나 이 경우
route는 이전에 정의한 path로 요청을 처리한다.

## **Creating a customer**

이제 클라이언트 객체의 JSON을 `POST`로 전송하는 옵션을 구현하고 이를 `customerStorage`에 저장한다.

```kotlin
post {
    val customer = call.receive<Customer>()
    customerStorage.add(customer)
    call.respondText("Customer stored correctly", status = HttpStatusCode.Created)
}
```

`call.receive`는 Content Negotiation 플러그인과 통합된다. 파라미터와 함께 호출되면 `Customer`는 자동으로 JSON을 `Customer` 객체로 deserialize한다. 고객을
저장하고 201(Created) 응답을 반환한다.

## **Deleting a customer**

고객을 제거하는 코드는 특정 고객을 리스팅하는 코드와 유사하다. `id`를 가져와 `customerStorage`를 수정한다.

```kotlin
delete("{id}") {
    val id = call.parameters["id"] ?: return@delete call.respond(HttpStatusCode.BadRequest)
    if (customerStorage.removeIf { it.id == id }) {
        call.respondText("Customer removed correctly", status = HttpStatusCode.Accepted)
    } else {
        call.respondText("Not Found", status = HttpStatusCode.NotFound)
    }
}
```

`get`과 유사하게 `id`의 null 여부를 확인한다. `id`가 존재하지 않다면 400(Bad Request) 에러가 발생한다.

## **Registering the routes**

지금까지 `Rotue`의 확장 함수에 routes를 정의하였다. Ktor는 routes에 대해 알지 못하기 때문에, 이를 등록해주어야 한다. `Application.module`에 각 route를 직접 추가할 수
있지만 그룹화하여 파일로 만드는 것이 유지관리하기 더 쉽다.

다음 코드를 `CustomerRoutes.kt` 파일에 추가하자.

```kotlin
fun Application.registerCustomerRoutes() {
    routing {
        customerRouting()
    }
}
```

이제 이 함수를 `Application.kt`의 `Application.module()`에서 호출해주면 된다.

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
}
```

이제 고객 관련 API는 모두 구현되었다.

# **Order routes**

이제 `Customer` 엔드포인트 API를 가지고 있으니, `Orders`를 구현해보자. 일부 구현은 `Customer`와 유사하지만 개개인의 오더 아이템의 가격을 합하는 route를 포함하여 다른 방식으로
route를 구현할 것이다.

## **Defining the model**

시스템에 저장하고자 하는 order는 order number로 구분해야하고, order item의 목록을 포함하고 있어야 한다. 이 order item은 텍스트로 설명되어 있는데, 몇개의 아이템인지와 각각의 가격이
있다.

`Order.kt` 파일을 생성하고 두 개의 다음 data class를 생성한다.

```kotlin
@Serializable
data class Order(val number: String, val contents: List<OrderItem>)

@Serializable
data class OrderItem(val item: String, val amount: Int, val price: Double)
```

이 주문에 대해 저장할 공간이 필요하다. `orderStorage`를 샘플 주문으로 미리 채울 것이다. `Order.kt` 파일에 정의한다.

```kotlin
val orderStorage = listOf(
    Order(
        "2020-04-06-01",
        listOf(
            OrderItem("Ham Sandwich", 2, 5.50),
            OrderItem("Water", 1, 1.50),
            OrderItem("Beer", 3, 2.30),
            OrderItem("Cheesecake", 1, 3.75)
        )
    ),
    Order(
        "2020-04-03-01",
        listOf(
            OrderItem("Cheeseburger", 1, 8.50),
            OrderItem("Water", 2, 1.50),
            OrderItem("Coke", 2, 1.76),
            OrderItem("Ice Cream", 1, 2.35)
        )
    )
)
```

## **Defining order routes**

다음과 같이 3개의 다른 GET 요청에 대해 응답할 것이다.

```
GET http://0.0.0.0:8080/order/
Content-Type: application/json

GET http://0.0.0.0:8080/order/{id}
Content-Type: application/json

GET http://0.0.0.0:8080/order/{id}/total
Content-Type: application/json
```

첫 번째는 모든 주문 목록을 반환하고, 두 번째는 전달된 `id`의 주문을 반환하며, 세 번째는 주문의 총 가격을 반환한다.

다른 HTTP 메서드를 `route` 함수에 모든 route를 그룹핑하는 것 대신 개개의 함수를 사용한다.

## **Listing all and individual orders**

주문 목록을 리스팅하기 위해, customer와 동일한 패턴을 사용한다. 차이점은 하나의 함수에 정의한다는 것이다. `OrderRoutes.kt` 파일을 `routes` 패키지에
생성하고, `listOrderRoute()` 함수를 구현하자.

```kotlin
fun Route.listOrdersRoute() {
    get("/order") {
        if (orderStorage.isNotEmpty()) {
            call.respond(orderStorage)
        }
    }
}
```

개개의 주문을 요청하는 부분에도 동일하게 적용한다. customer와 유사하지만 자체의 함수로 캡슐화되어 있다.

```kotlin
fun Route.getOrderRoute() {
    get("/order/{id}") {
        val id = call.parameters["id"] ?: return@get call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
        val order = orderStorage.find { it.number == id } ?: return@get call.respondText(
            "Not Found",
            status = HttpStatusCode.NotFound
        )
        call.respond(order)
    }
}
```

## **Totalizing an order**

주문의 총 가격을 구하는 것은 주문 아이템을 반복하고 이를 합산하면 된다. `totalizeOrderRoute` 함수를 구현한다.

```kotlin
fun Route.totalizeOrderRoute() {
    get("/order/{id}/total") {
        val id = call.parameters["id"] ?: return@get call.respondText("Bad Request", status = HttpStatusCode.BadRequest)
        val order = orderStorage.find { it.number == id } ?: return@get call.respondText(
            "Not Found",
            status = HttpStatusCode.NotFound
        )
        val total = order.contents.map { it.price * it.amount }.sum()
        call.respond(total)
    }
}
```

한 가지 주목해야 할 점은 여기서 볼 수 있듯이 중앙 섹션이 파라미터가 될 수 있다는 점이다(`/order/{id}/total`).

## **Registering the routes**

이제 customer와 같이 routes를 등록해주어야 한다. route의 개수가 늘어남에 따라 그룹핑 하는 것이 더 깔끔해진다. `OrderRoutes.kt`에 `Application` 확장
함수인 `registerOrderRoutes`를 추가한다.

```kotlin
fun Application.registerOrderRoutes() {
    routing {
        listOrderRoute()
        getOrderRoute()
        totalizeOrderRoute()
    }
}
```

다음 `Application.module()`에서 위 함수를 호출한다.

```kotlin
fun Application.module() {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```

이제 모든 것이 구현되었고, 애플리케이션을 실행하혀 테스트하면 된다.

# **Manually testing HTTP endpoints**

모든 엔드포인트가 준비되었으니, 테스트해볼 시간이다. 어떤 브라우저든 GET 요청을 사용할 수 있으나, 다른 메서드를 테스트하려면 다른 도구가 필요하다. curl 또는
Postman. [IntelliJ IDEA Ultimate Edition](https://www.jetbrains.com/idea/)을 사용하는 경우 요청을 지정하고 실행할 수 있는 `.http` 파일을 지원하는
클라이언트가 이미 존재한다.

나는 [IntelliJ IDEA Ultimate Edition](https://www.jetbrains.com/idea/)이 없으므로 이 가이드는 스킵한다.

# **Automated testing**

수동 테스트는 필수적이지만, 자동 테스팅도 의미가 있다.

`ktor-server-test-host`를 통해 기본 엔진을 시작하지 않고도 Ktor의 엔드포인트 테스트를 수행할 수 있다. 이 프레임워크는 요청들을 테스트하기 위한 헬퍼 메소드를 제공한다. 가장 중요한
하나는 `withTestApplication`이다.

order route가 포맷된 JSON 콘텐트로 반환하는지를 확인하기 위한 unit teset를 작성해보자. `test/kotlin` 하위에 `OrderTests.kt` 파일을 생성하고 다음과 같이 작성한다.

```kotlin
class OrderRouteTests {
    @Test
    fun testGetOrder() {
        withTestApplication({ module(testing = true) }) {
            handleRequest(HttpMethod.Get, "/order/2020-04-06-01").apply {
                assertEquals(
                    """{"number":"2020-04-06-01","contents":[{"item":"Ham Sandwich","amount":2,"price":5.5},{"item":"Water","amount":1,"price":1.5},{"item":"Beer","amount":3,"price":2.3},{"item":"Cheesecake","amount":1,"price":3.75}]}""",
                    response.content
                )
                assertEquals(HttpStatusCode.OK, response.status())
            }
        }
    }
}
```

`withTestApplication`을 사용하여 테스트로써 실행하라고 애플리케이션에게 말해줄 수 있다. `handleRequest` 헬퍼 메서드를 사용해 특정 엔드포인트에 대한 요청을 할 수 있다. 지금의
경우 `/order/{id}`이다.

`number`와 같이 문자열에는 많은 더블 쿼터들이 포함되어 있기 때문에, `“””`를 사용해 raw string을 작성할 수 있다.

이 애플리케이션을 컴파일하려하면 에러가 발생하는데, 이는 `module`에 `testing` 파라미터가 없기 때문이다. 다음과 같이 해당 파라미터를 추가한다.

```kotlin
fun Application.module(testing: Boolean = false) {
    install(ContentNegotiation) {
        json()
    }
    registerCustomerRoutes()
    registerOrderRoutes()
}
```

이제 테스트를 실행하고 결과를 보자. 이와 같이 다른 엔드포인트 API에 대해서도 테스트 코드를 작성하고 자동화할 수 있다.

# **What's next**

HTTP API 애플리케이션을 이 단계에서 알아보았다. 여기에서 Authentication과 같은 추가적인 플러그인을 알아보자

## **Feature requests**

- **Authentication** : API가 인증된 사용자에게만 접근할 수 있도록 한다. JWT 또는 다른 인증 메서드를 통해 제한된 접근을 구현할 수 있다.
- **Learn more about route organization!** route을 다른 방법으로 구성하는 것을 [Routing](https://ktor.io/docs/routing-in-ktor.html)에서
  확인할 수 있다.
- **Persistence!** 현재 애플리케이션을 종료하면 저장된 모든 것을 잃게 된다. PostgreSQL 또는 MongoDB를 사용해 데이터의 지속성을 유지할 수 있다.
- **Integrate with a client!** 이제 데이터를 노출했으므로 이 데이터를 다시 사용할 수 있는 방법을 알아보는 것이 좋다. 예를 들어 Ktor HTTP Client를 사용해 API Client를
  만들거나 Javascript 또는 Kotlin/JS를 사용해 웹사이트에 접근해보자.

API가 **browser clients**와 원활하게 작동하도록 CORS에 대한 정책도 설정해야 한다. 간단하게 다음 코드를 `Application.module()`의 가장 상단에 추가하는 방법이 있다.

```kotlin
install(CORS) {
    anyHost()
}
```

## References

* [Creating HTTP APIs | Ktor](https://ktor.io/docs/creating-http-apis.html)