# ROOM Database, Dagger-Hilt Setup

이번에는 Room 및 Dagger-Hilt를 적용한다.

## Setup Dagger-Hilt

루트 디렉토리에 `MyApplication` 클래스를 생성하고 다음과 같이 주석을 추가해준다. AndroidManifest.xml 파일도 수정해준다.

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

`MainActivity`에 `@AndroidEntryPoint` 주석을 추가한다.

Constants에 `BASE_URL`을 선언한다.

```kotlin
object Constants {

    const val BASE_URL = "https://api.unsplash.com"
    // ...
}
```

di 패키지를 생성한 후 NetworkModule 파일을 생성한다.

```kotlin
@ExperimentalSerializationApi
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .readTimeout(15, TimeUnit.SECONDS)
            .connectTimeout(15, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        val contentType = MediaType.get("application/json")
        val json = Json {
            ignoreUnknownKeys = true // 필요 필드 제외하고 모두 무시
        }

        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(json.asConverterFactory(contentType))
            .build()
    }

    @Singleton
    @Provides
    fun provideUnsplashApi(retrofit: Retrofit): UnsplashApi {
        return retrofit.create(UnsplashApi::class.java)
    }
}
```

## Setup Room

Constants에 `BASE_URL`을 선언한다.

```kotlin
object Constants {

    // ...
    const val UNSPLASH_REMOTE_KEYS_TABLE = "unsplash_remote_keys_table"
}
```

data 패키지에 RmoteMediator가 다음 어떤 페이지를 로드할지 알려주는 `UnsplashRemoteKeys` 엔티티를 만들어준다.

```kotlin
// RemoteKeys를 로컬 디비에 저장하여 RmoteMediator가 다음에 어떤 페이지를 로드할지 알려주기 위함
@Entity(tableName = UNSPLASH_REMOTE_KEYS_TABLE)
data class UnsplashRemoteKeys(
    @PrimaryKey(autoGenerate = false)
    val id: String,
    val prevPage: Int?,
    val nextPage: Int?
)
```

data/local/dao 패키지 생성 후 `UnsplashImageDao`, `UnsplashRemoteKeysDao`를 생성한다.

```kotlin
@Dao
interface UnsplashImageDao {

    @Query("SELECT * FROM unsplash_image_table")
    fun getAllImages(): PagingSource<Int, UnsplashImage>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun addImages(images: List<UnsplashImage>)

    @Query("DELETE FROM unsplash_image_table")
    suspend fun deleteAllImages()
}
```

```kotlin
@Dao
interface UnsplashRemoteKeysDao {

    // UnsplashImage의 id를 전달하면 해당 id의 Remote Key를 반환
    @Query("SELECT * FROM unsplash_remote_keys_table WHERE id=:id")
    suspend fun getRemoteKeys(id: String): UnsplashRemoteKeys

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun addAllRemoteKeys(remoteKeys: List<UnsplashRemoteKeys>)

    @Query("DELETE FROM unsplash_remote_keys_table")
    suspend fun deleteAllRemoteKeys()
}
```

이제 디비를 제공하기 위해 local 패키지에 `UnsplashImageDatabase`를 생성한다.

```kotlin
@Database(
    entities = [UnsplashImage::class, UnsplashRemoteKeys::class],
    version = 1
)
abstract class UnsplashDatabase : RoomDatabase() {

    abstract fun unsplashImageDao(): UnsplashImageDao
    abstract fun unsplashRemoteKeysDao(): UnsplashRemoteKeysDao
}
```

Constant에 디비 이름을 지정한다.

```kotlin
object Constants {
    // ...
    const val UNSPLASH_DATABASE = "unsplash_database "
    // ...
}
```

그리고 di 패키지에 `DatabaseModule` object를 생성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Provides
    @Singleton
    fun provideDatabase(
        @ApplicationContext context: Context
    ): UnsplashDatabase {
        return Room.databaseBuilder(
            context,
            UnsplashDatabase::class.java,
            UNSPLASH_DATABASE
        ).build()
    }
}
```

## References

* [Paging 3 & Jetpack Compose - Android Development | Part 3 - ROOM Database, Dagger-Hilt Setup](https://www.youtube.com/watch?v=JfixpS_F35k&list=PLSrm9z4zp4mEWwyiuYgVMWcDFdsebhM-r&index=36)