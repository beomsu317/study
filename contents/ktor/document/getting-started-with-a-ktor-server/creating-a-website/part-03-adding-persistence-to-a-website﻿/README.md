# Adding persistence to a website

이 튜토리얼 시리즈는 Ktor를 통해 간단한 블로그를 생성하는 방법을 보여준다.

* 우선 HTML 페이지와 이미지 같은 static content를 호스팅하는 것을 보여준다.
* 다음 튜토리얼에서 FreeMarker 템플릿 엔진을 사용해 interactive 한 앱을 만들어본다.
* 마지막으로 Exposed 프레임워크를 사용해 웹사이트에 persistense를 추가한다.

## Add dependencies

우선 Exposed와 H2 라이브러리에 대한 디펜던시를 추가한다. `gradle.properties` 파일을 열고 라이브러리 버전을 지정한다.

```
exposed_version = 0.36.2
h2_version = 1.4.200
```

그리고 `build.gradle.kts`를 열고 다음 디펜던시를 추가해준다.

```kotlin
val exposed_version: String by project
val h2_version: String by project

dependencies {
    implementation("org.jetbrains.exposed:exposed-core:$exposed_version")
    implementation("org.jetbrains.exposed:exposed-dao:$exposed_version")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposed_version")
    implementation("com.h2database:h2:$h2_version")
}
```

**Load Gradle Changes** 아이콘을 클릭하여 추가된 디펜던시를 설치한다.

## Update a model

Exposed는 `org.jetbrains.exposed.sql.Table` 클래스를 디비 테이블로 사용한다. `Article` 모델을 수정하기 위해 `models/Article.kt` 파일을 열고 다음과 같이
수정한다.

```kotlin
package com.example.models

import org.jetbrains.exposed.sql.*

data class Article(val id: Int, val title: String, val body: String)

object Articles : Table() {
    val id = integer("id").autoIncrement()
    val title = varchar("title", 128)
    val body = varchar("body", 1024)

    override val primaryKey = PrimaryKey(id)
}
```

`id`, `title`, `body` 컬럼은 기사에 대한 정보를 저장한다. `id` 컬럼은 primary key로써 동작한다.

> `Articles` 객체의 속성 타입을 보면 필요한 타입 인자가 있는 컬럼이 있음을 알 수 있다. `id`는 `Column<Int>` 타입을 가지며 `title`과 `body`는 `Column<String>`을
> 가진다.

## Connect to a database

[Data access object](https://en.wikipedia.org/wiki/Data_access_object)(DAO)는 특정 디비의 세부 정보를 노출하지 않고 디비에 인터페이스를 제공하는 패턴이다.
디비에 대한 요청을 추상화하기 위해 나중에 `DAOFacade` 인터페이스를 정의할 것이다.

`Exposed`를 사용하는 모든 디비 접근은 디비에 대한 연결을 구현해야 한다. 이를 위해, JDBC URL과 driver 클래스 이름을 `Database.connect` 함수에 전달해야
한다. `com.example` 패키지에 `dao` 패키지를 생성하고 `DatabaseFactory.kt` 파일을 추가한다. 그리고 다음 코드를 작성한다.

```kotlin
package com.example.dao

import com.example.models.*
import kotlinx.coroutines.*
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.*
import org.jetbrains.exposed.sql.transactions.experimental.*

object DatabaseFactory {
    fun init() {
        val driverClassName = "org.h2.Driver"
        val jdbcURL = "jdbc:h2:file:./build/db"
        val database = Database.connect(jdbcURL, driverClassName)
    }
}
```

> 여기서 `driverClassName`과 `jdbcURL`은 하드코딩 되어있다. Ktor는 이러한
> 설정을 [custom configuration group](https://ktor.io/docs/configurations.html#hocon-file)으로 추출할 수 있다.

### Create a table

연결을 얻은 후 모든 SQL 구문은 트랜잭션 안에 위치해야만 한다.

```kotlin
fun init() {
    // ...
    val database = Database.connect(jdbcURL, driverClassName)
    transaction(database) {
        // Statements here
    }
}
```

이 샘플 코드에서 기본 디비는 `transaction` 함수에 명시적으로 전달된다. 만약 하나의 디비만 가지고 있다면 이를 생략할 수 있다. 이 경우 Exposed는 트랜잭션에 마지막으로 연결된 디비를 자동으로
사용한다.

> `Database.connect` 함수는 트랜잭션을 호출하기 전까지는 실제 디비에 연결을 수립하지 않는 것을 명심하자. 이는 향후 연결을 위한 descriptor만 생성한다.

`Articles` 테이블이 이미 선언된 경우, `init` 함수의 맨 아래에 있는 `transaction` 호출로 래핑된 `SchemaUtils.create(Articles)`를 호출하여 아직 존재하지 않는 경우
이 테이블을 생성하도록 디비에 지시할 수 있다.

```kotlin
fun init() {
    // ...
    val database = Database.connect(jdbcURL, driverClassName)
    transaction(database) {
        SchemaUtils.create(Articles)
    }
}
```

### Execute queries

편의를 위해 `DatabaseFactory` 객체 내 `dbQuery` 유틸리티 함수를 생성하자. 이는 추후에 디비 요청에 사용한다. 블로킹 방식으로 트랜잭션을 사용하는 대신, 코루틴을 사용해 자체 코루틴에서 각
쿼리를 실행한다.

```kotlin
suspend fun <T> dbQuery(block: suspend () -> T): T =
    newSuspendedTransaction(Dispatchers.IO) { block() }
```

결과 `DatabaseFactory.kt` 파일은 다음과 같다.

```kotlin
package com.example.dao

import com.example.models.*
import kotlinx.coroutines.*
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.*
import org.jetbrains.exposed.sql.transactions.experimental.*

object DatabaseFactory {
    fun init() {
        val driverClassName = "org.h2.Driver"
        val jdbcURL = "jdbc:h2:file:./build/db"
        val database = Database.connect(jdbcURL, driverClassName)
        transaction(database) {
            SchemaUtils.create(Articles)
        }
    }

    suspend fun <T> dbQuery(block: suspend () -> T): T =
        newSuspendedTransaction(Dispatchers.IO) { block() }
}
```

### Load database config at startup

마지막으로 앱 실행 시 생성된 구성을 로드해야 한다. `Application.kt`를 열고 `Application.module` 블록에 `DatabaseFactory.init`을 호출한다.

```kotlin
fun Application.module() {
    DatabaseFactory.init()
    configureRouting()
    configureTemplating()
}
```

## Implement persistence logic

기사를 업데이트 하기 위한 필수 명령에 대해 추상화하기 위한 인터페이스를 만들자. `dao` 패키지에 `DAOFacade.kt` 파일을 만들고 다음과 같이 작성한다.

```kotlin
package com.example.dao

import com.example.models.*

interface DAOFacade {
    suspend fun allArticles(): List<Article>
    suspend fun article(id: Int): Article?
    suspend fun addNewArticle(title: String, body: String): Article?
    suspend fun editArticle(id: Int, title: String, body: String): Boolean
    suspend fun deleteArticle(id: Int): Boolean
}
```

모든 기사를 나열하고, 해당 ID로 기사를 보고, 새 기사를 추가하고, 편집하거나 삭제해야 한다. 모든 이러한 함수는 내부에서 디비 쿼리를 수행하므로 suspending 함수로 정의된다.

`DAOFacade` 인터페이스를 구현하려면, 이름에 마우스 포인터를 위치시키고 노란 전구 아이콘을 클릭하여 **Implement interface**을 선택한다. 호출된 대화 상자에서 기본 설정을 그대로 두고 **
OK**을 클릭한다.

`Implement Members` 다이얼로그에서 모든 함수를 선택하고 **OK**를 클릭한다.

<div align="center">
<img src="img/impl_dialog.png" width="80%">
</div>

IntelliJ IDEA는 `dao` 패키지에 `DAOFacadeImpl.kt` 파일을 생성한다.

### Get all articles

모든 엔트리를 반환하는 함수를 만들어보자. 요청은 `dbQuery` 호출로 래핑된다. `Table.selectAll` 확장 함수를 호출해 디비의 모든 데이터를 얻어올 수 있다. `Articles`
객체는 `Table`의 서브클래이므로 Exposed DSL 메서드를 사용해 작업한다.

```kotlin
package com.example.dao

import com.example.dao.DatabaseFactory.dbQuery
import com.example.models.*
import kotlinx.coroutines.runBlocking
import org.jetbrains.exposed.sql.*

class DAOFacadeImpl : DAOFacade {
    private fun resultRowToArticle(row: ResultRow) = Article(
        id = row[Articles.id],
        title = row[Articles.title],
        body = row[Articles.body],
    )

    override suspend fun allArticles(): List<Article> = dbQuery {
        Articles.selectAll().map(::resultRowToArticle)
    }
}
```

`Table.selectAll`은 `Query` 인스턴스를 반환하므로 `Article` 인스턴스 리스트를 얻으려면 각 row에 대한 데이터를 수동으로 추출하고 data class로 변환해야 한다. 이를
위해 `ResultRow`에서 `Article`로 빌드하는 헬퍼 함수인 `resultRowToArticle`를 사용한다.

`ResultRow`는 간결한 `get` 연산자를 사용해 지정된 column에 저장된 데이터를 가져오는 방법을 제공하므로, 배열이나 맵에 유사한 대괄호 구문을 사용할 수 있다.

> `Articles.id`의 타입은 `Expression` 인터페이스를 상속한 `Column<Int>`이다. 따라서 모든 column을 표현식으로 전달할 수 있다.

### Get an article

하나의 기사를 반환하는 함수를 만들어보자.

```kotlin
override suspend fun article(id: Int): Article? = dbQuery {
    Articles
        .select { Articles.id eq id }
        .map(::resultRowToArticle)
        .singleOrNull()
}
```

`select` 함수는 람다를 인자로 취한다. 람다 내 암시적인 리시버의 타입은 `SqlExpressionBuilder`이다. 이 타입을 명시적으로 사용하지는 않지만 쿼리를 빌드하는 column에 대한 여러가지
유용한 작업을 정의한다. 비교(`eq`, `less`, `greater`), 산술 연산(`plus`, `times`)을 사용하고 값이 제공된 값 리스트(`inList`, `noInList`)에 속하는지 여부를
확인하고 값이 null 또는 non-null 인지 여부 등을 확인할 수 있다.

`select`는 `Query` 값들의 목록을 반환한다. 이전과 같이 이들을 기사로 변환해야 한다. 이 경우 하나의 기사여야 하므로 결과를 반환한다.

### Add a new article

테이블에 새로운 기사를 삽입하기 위해 람다 인자를 받는 `Table.insert` 함수를 사용한다.

```kotlin
override suspend fun addNewArticle(title: String, body: String): Article? = dbQuery {
    val insertStatement = Articles.insert {
        it[Articles.title] = title
        it[Articles.body] = body
    }
    insertStatement.resultedValues?.singleOrNull()?.let(::resultRowToArticle)
}
```

람다 안에 어떤 값이 어떤 column에 설정되어야 하는지 지정한다. `it` 인자는 column과 값을 인자로 사용하는 `set` 연산자를 호출할 수 있는 `InsertStatement` 타입이 있다.

### Edit an article

기사를 업데이트 하기 위해 `Table.update`가 사용된다.

```kotlin
override suspend fun editArticle(id: Int, title: String, body: String): Boolean = dbQuery {
    Articles.update({ Articles.id eq id }) {
        it[Articles.title] = title
        it[Articles.body] = body
    } > 0
}
```

### Delete an article

기사를 삭제하기 위해 `Table.deleteWhere`을 사용한다.

```kotlin
override suspend fun deleteArticle(id: Int): Boolean = dbQuery {
    Articles.deleteWhere { Articles.id eq id } > 0
}
```

### Initialize DAOFacade

`DAOFacade` 인스턴스를 생성하고 앱 실행 전 샘플 기사를 디비에 추가하자. 다음 코드를 `DAOFacadeImpl.kt` 하단에 작성한다.

```kotlin
val dao: DAOFacade = DAOFacadeImpl().apply {
    runBlocking {
        if (allArticles().isEmpty()) {
            addNewArticle("The drive to develop!", "...it's what keeps me going.")
        }
    }
}
```

## Update routes

이제 route 핸들러에서 디비 연산을 사용하기 위한 준비가 완료되었다. `plugins/Routing.kt` 파일을 열고, 모든 기사를 보여주기 위해 `get` 핸들러에 `dao.allArticles`을 호출한다.

```kotlin
get {
    call.respond(FreeMarkerContent("index.ftl", mapOf("articles" to dao.allArticles())))
}
```

새 기사를 작성하기 위해 `dao.addNewArticle` 함수를 `post`에서 호출한다.

```kotlin
post {
    val formParameters = call.receiveParameters()
    val title = formParameters.getOrFail("title")
    val body = formParameters.getOrFail("body")
    val article = dao.addNewArticle(title, body)
    call.respondRedirect("/articles/${article?.id}")
}
```

기사를 보여주고 수정하기 위해 `dao.article`을 `get("{id}")`와 `get("{id}/edit")`에서 사용한다.

```kotlin
get("{id}") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    call.respond(FreeMarkerContent("show.ftl", mapOf("article" to dao.article(id))))
}
get("{id}/edit") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    call.respond(FreeMarkerContent("edit.ftl", mapOf("article" to dao.article(id))))
}
```

마지막으로 `post("{id}")` 핸들러에서 기사를 업데이트 하기 위해 `dao.editArticle`을 사용하고, 삭제하기 위해 `dao.deleteArticle`를 사용한다.

```kotlin
post("{id}") {
    val id = call.parameters.getOrFail<Int>("id").toInt()
    val formParameters = call.receiveParameters()
    when (formParameters.getOrFail("_action")) {
        "update" -> {
            val title = formParameters.getOrFail("title")
            val body = formParameters.getOrFail("body")
            dao.editArticle(id, title, body)
            call.respondRedirect("/articles/$id")
        }
        "delete" -> {
            dao.deleteArticle(id)
            call.respondRedirect("/articles")
        }
    }
}
```

## Run the application

이제 원하는 대로 동작하는 것을 확인해보자. `http://localhost:8080/`로 접근하고 기사를 생성, 수정, 삭제해보자. 기사는 `build/db.mv.db` 파일에 저장된다. IntelliJ
IDEA에서 [Database tool window](https://www.jetbrains.com/help/idea/database-tool-window.html?_ga=2.129308748.1396641199.1655526702-658241611.1655526702&_gl=1*zuaicj*_ga*NjU4MjQxNjExLjE2NTU1MjY3MDI.*_ga_9J976DJZ68*MTY1NTYxOTc1MS45LjEuMTY1NTYyMDg1Ny4w)
를 통해 이 파일의 내용을 볼 수 있다.

<div align="center">
<img src="img/db.png" width="80%"> 
</div>

## References

* [Adding persistence to a website](https://ktor.io/docs/interactive-website-add-persistence.html)