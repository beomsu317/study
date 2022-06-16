# One-to-N Relationships

이번에는 1:N 관계를 구현해보자. School - Students

`Student` 데이터 클래스를 다음과 같이 생성한다.

```kotlin
@Entity
data class Student(
        @PrimaryKey(autoGenerate = false)
        val studentName: String,
        val semester: Int,
        val schoolName: String
)
```

`entities.relations`에 `SchoolWithStudents` 관계를 생성해준다.

```kotlin
data class SchoolWithStudents(
        @Embedded val school: School,
        @Relation(
                parentColumn = "schoolName", // School의 schoolName
                entityColumn = "schoolName"  // Student의 schoolName
        )
        val students: List<Student>
)
```

그리고 DAO에 `insertStudent()`와 `getSchoolWithStudents()`를 만들어준다.

```kotlin
@Dao
interface SchoolDao {
    // ...
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertStudent(student: Student)
    // ...
    @Transaction
    @Query("SELECT * FROM school WHERE schoolName = :schoolName")
    suspend fun getSchoolWithStudents(schoolName: String): List<SchoolWithStudents>
}
```

## References

[One-to-N Relationships - Android Room Database With Multiple Tables](https://www.youtube.com/watch?v=K8yKH5Ak84E&list=PLQkwcJG4YTCS3AD2C-yWtJUGTYMh5h3Zz&index=3)