# Setting up our Room database

이번엔 로컬 Room DB를 설정해 볼 것이다.

`data/local/entities` 패키지를 생성한 후 `Note` data class를 생성 및 작성해준다.

```kotlin
@Entity(tableName = "notes")
data class Note(
        val title: String,
        val content: String,
        val date: Long,
        val owners: List<String>,
        val color: String,
        @Expose(deserialize = false, serialize = false)
        var isSynced: Boolean = false,
        @PrimaryKey(autoGenerate = false)
        val id: String = UUID.randomUUID().toString()
)
```

`owners List<String>`은 primitive data type이 아니므로 Room에 저장하기 위해 TypeConverter를 만들어주어야 한다. `data/local` 패키지에 `Converters`
클래스를 생성 후 TypeConverter 를 작성해준다.

```kotlin
class Converters {

    @TypeConverter
    fun fromList(list: List<String>): String {
        return Gson().toJson(list)
    }

    @TypeConverter
    fun toList(string: String): List<String> {
        return Gson().fromJson(string, object : TypeToken<List<String>>() {}.type)
    }
}
```

Room DB에 접근하기 위한 `NoteDao` 인터페이스를 생성 및 작성한다.

```kotlin
@Dao
interface NoteDao {

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertNote(note: Note)

    @Query("DELETE FROM notes WHERE id = :noteID")
    suspend fun deleteNoteById(noteID: String)

    @Query("DELETE FROM notes WHERE isSynced = 1")
    suspend fun deleteAllSyncedNotes()

    @Query("SELECT * FROM notes WHERE id = :noteID")
    fun observeNoteById(noteID: String): LiveData<Note>

    @Query("SELECT * FROM notes WHERE id = :noteID")
    suspend fun getNoteById(noteID: String): Note?

    @Query("SELECT * FROM notes ORDER BY date DESC")
    fun getAllNotes(): Flow<List<Note>>

    @Query("SELECT * FROM notes WHERE isSynced = 0")
    suspend fun getAllUnsyncedNotes(): List<Note>
}
```

이제 `data/local` 패키지에 `NoteDatabase` 클래스를 만들어준다.

```kotlin
@Database(
        entities = [Note::class],
        version = 1
)
@TypeConverters(Converters::class)
abstract class NotesDatabase : RoomDatabase() {
    abstract fun noteDao(): NoteDao
}
```