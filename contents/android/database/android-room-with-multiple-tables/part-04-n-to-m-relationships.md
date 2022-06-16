# N-to-M Relationships

이번에는 N-M 관계를 구현해볼 것이다. Students - Subjects

N-M 관계를 표현하기 위해 각 테이블의 기본키를 가진 테이블을 만들어주어야 한다. `entities.relations` 디렉토리 하위에 `StudentSubjectCrossRef()`를 생성한다.

```kotlin
// Student 테이블과 Subject 테이블이 관계되어 있음을 알려주기 위해 primaryKeys를 전달한다.
@Entity(primaryKeys = ["studentName", "subjectName"])
data class StudentSubjectCrossRef(
    val studentName: String, // Student 테이블의 기본키
    val subjectName: String  // Subject 테이블의 기본키
)
```

Student가 어떤 Subject 들을 수강했는지, Subject가 어떤 Student에 의해 수강되는지 알기 위해 2개의 클래스가 추가로 필요하다. 우선 `StudentWithSubject()` 데이터 클래스를 생성한다.

```kotlin
// 하나의 student가 어떤 subject 들을 수강했는지
data class StudentWithSubjects(
    @Embedded val student: Student,
    @Relation(
        parentColumn = "studentName",
        entityColumn = "subjectName",
        associateBy = Junction(StudentSubjectCrossRef::class)
    )
    val subjects: List<Subject>
)
```

`StudentWithSubject()`와 동일하게 `SubjectWithStudent()`를 생성한다.

```kotlin
data class SubjectWithStudents(
    @Embedded val subject: Subject,
    @Relation(
        parentColumn = "subjectName",
        entityColumn = "studentName",
        associateBy = Junction(StudentSubjectCrossRef::class)
    )
    val students: List<Student>
)
```

## References

* [N-to-M Relationships - Android Room Database With Multiple Tables](https://www.youtube.com/watch?v=AHn5JQVlJJM&list=PLQkwcJG4YTCS3AD2C-yWtJUGTYMh5h3Zz&index=4)