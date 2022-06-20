# Dagger-Hilt setup

이번엔 DI를 위한 Dagger-Hilt 설정을 수행한다.

Dagger-Hilt를 사용하기 위해 `Application` 클래스를 상속한 `NoteApplication` 클래스를 생성하고 다음과 같이 작성한다.

```kotlin
@HiltAndroidApp
class NoteApplication : Application() {

    override fun onCreate() {
        super.onCreate()
        // 디버그 로깅을 위해 DebugTree plant
        Timber.plant(Timber.DebugTree())
    }
}
```

`AndroidManifest.xml` 파일의 `application` 태그의 `name`에 `NoteApplication`을 추가해준다.

```xml

<manifest>
    <application
            android:name=".NoteApplication">
    </application>
</manifest>
```

이제 인젝션을 위한 모듈을 작성해보자.

`other/Constants` 파일에 `DATABASE_NAME`과 `BASE_URL`을 작성한다.

```kotlin
object Constants {

    const val DATABASE_NAME = "notes_db"

    const val ENCRYPTED_SHARED_PREF_NAME = "enc_shared_pref"

    const val BASE_URL = "http://10.0.2.2:8080" // 에뮬레이터에서 실행하는 경우 10.0.2.2로 설정해 로컬 PC에 접근할 수 있도록 한다.

    val IGNORE_AUTH_URLS = listOf("/login", "/register")
}
```

`di` 패키지를 생성한 후 하위에 `AppModule.kt` 파일을 생성 및 작성한다.

```kotlin
@Module
@InstallIn(ApplicationComponent::class) // Dagger-Hilt 낮은 버전은 Singleton을 객체를 위해 ApplicationComponent::class 사용
object AppModule {

    @Singleton
    @Provides
    fun provideNotesDatabase(
            @ApplicationContext context: Context
    ) = Room.databaseBuilder(context, NotesDatabase::class.java, DATABASE_NAME).build()

    @Singleton
    @Provides
    fun provideNoteDao(db: NotesDatabase) = db.noteDao()

    @Singleton
    @Provides
    fun provideBasicAuthInterceptor() = BasicAuthInterceptor()

    @Singleton
    @Provides
    fun provideNoteApi(
            basicAuthInterceptor: BasicAuthInterceptor
    ): NoteApi {
        val client = OkHttpClient.Builder()
                .addInterceptor(basicAuthInterceptor)
                .build()

        return Retrofit.Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .client(client)
                .build()
                .create(NoteApi::class.java)
    }

    // 네트워크 연결이 안되었을 때 로그인 된 상태를 유지하기 위해 암호화된 shared preferences를 가질 것이다.
    // 루팅된 단말기에서 shared preferences를 읽어 다른 유저로 변경할 가능성도 있다.
    @Singleton
    @Provides
    fun provideEncryptedSharedPreferences(
            @ApplicationContext context: Context
    ): SharedPreferences {
        // 암호화하기 위해 키가 필요함
        val masterKey = MasterKey.Builder(context)
                .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
                .build()
        return EncryptedSharedPreferences.create(
                context,
                ENCRYPTED_SHARED_PREF_NAME,
                masterKey,
                EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
                EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }
}
```