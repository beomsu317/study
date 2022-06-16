# One-to-One Relationships with Room

이전 강의에서 얘기했던 E-R-Model을 실제 안드로이드 앱에 구현해볼 것이다. 다음 디펜던시를 추가하자.

```kotlin
// Room
implementation "androidx.room:room-runtime:2.2.5"
kapt "androidx.room:room-compiler:2.2.5"

// Kotlin Extensions and Coroutines support for Room
implementation "androidx.room:room-ktx:2.2.5"

// Coroutines
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.7'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'

// Coroutine Lifecycle Scopes
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
```

우선 1:1 관계(School - Director) 데이터베이스를 생성해볼 것이다.

`entities` 패키지를 만들고 하위에 `Director` 데이터 클래스를 생성한다.

```kotlin
@Entity
data class Director(
    @PrimaryKey(autoGenerate = false)
    val directorName: String,
		val schoolName: String  // 비교하기 위함
)
```

`School` 데이터 클래스도 동일하게 생성해준다.

```kotlin
@Entity
data class School(
    @PrimaryKey(autoGenerate = false)
    val schoolName: String
)
```

그리고 `entities.relations` 패키지를 만들고 `SchoolAndDirector` 데이터 클래스를 생성한다.

```kotlin
data class SchoolAndDirector(
		// 모든 School 클래스의 필드들이 SchoolAndDirector에 삽입되도록 @Embedded 어노테이션 추가
    @Embedded val school: School,
		// @Relation
    @Relation(
        parentColumn = "schoolName", // School의 schoolName 참조
        entityColumn = "schoolName" // 기본키가 아닌 parentColumn과 비교할 수 있는 엔틴티 컬럼
    )
    val director: Director
)
```

다음과 같이 DAO 인터페이스를 만든다.

```kotlin
@Dao
interface SchoolDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertSchool(school: School)

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertDirector(director: Director)

    // multi-threading 이슈 발생하기 때문에 @Transaction 주석 추가
    @Transaction
    @Query("SELECT * FROM school WHERE schoolName = :schoolName")
    suspend fun getSchoolAndDirectorWithSchoolName(schoolName: String): List<SchoolAndDirector>
}
```

현재 데이터베이스를 생성하지 않아 테스트가 어렵다.

## References

* [One-to-One Relationships with Room - Android Room Database With Multiple Tables](https://www.youtube.com/watch?v=2K_eFam0qBg&list=PLQkwcJG4YTCS3AD2C-yWtJUGTYMh5h3Zz&index=2)