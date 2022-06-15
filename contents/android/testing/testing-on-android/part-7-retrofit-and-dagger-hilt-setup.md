# Retrofit & Dagger-Hilt Setup

## Get API and set

다음 API를 사용하기 위해 가입 후 key를 발급받는다.

- [https://pixabay.com/api/docs/](https://pixabay.com/api/docs/)

`gradle.properties`에 `API_KEY`를 추가한다.

```kotlin
API_KEY="API_KEY"
```

`build.gradle` 파일에 `buildConfigField`로 API_KEY를 가져온다.

```kotlin
defaultConfig {
        // ...
        buildConfigField("String", "API_KEY", API_KEY)
        // ...
    }
```

이를 `.gitignore`에 추가하여 안전하게 보호할 수 있다.

## Setup image result & response

`data.remote.responses` 디렉토리에 `ImageResul`, `ImageResponse` data class를 생성한다.

```kotlin
data class ImageResult(
    val comments: Int,
    val downloads: Int,
    val favorites: Int,
    val id: Int,
    val imageHeight: Int,
    val imageSize: Int,
    val imageWidth: Int,
    val largeImageURL: String,
    val likes: Int,
    val pageURL: String,
    val previewHeight: Int,
    val previewURL: String,
    val previewWidth: Int,
    val tags: String,
    val type: String,
    val user: String,
    @SerializedName("user_id")
    val userId: Int,
    val userImageURL: String,
    val views: Int,
    val webformatHeight: Int,
    val webformatURL: String,
    val webformatWidth: Int
)
```

```kotlin
data class ImageResponse(
    val hits: List<ImageResult>,
    val total: Int,
    val totalHits: Int
)
```

## Setup pixabay interface

`data.remote`에 Retrofit2에서 사용할 `PixabayAPI` 인터페이스를 생성한다.

```kotlin
interface PixabayAPI {

    @GET("/api/")
    suspend fun searchForIamge(
        @Query("q") searchQuery: String,
        @Query("key") apiKey: String = BuildConfig.API_KEY
    ): Response<ImageResponse>
}
```

## Setup Dagger-Hilt

`Application`을 상속하는 클래스 생성 후 Dagger-Hilt 사용을 위해  `@HiltAndroidApp` 어노테이션을 추가한다.

```kotlin
@HiltAndroidApp
class ShoppingApplication: Application() {
}
```

`AndroidManifest.xml`의 `application` 태그에 `android:name=".ShoppingApplication"`을 추가한다.

```xml
<application
        android:name=".ShoppingApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        ...
        >
</application>
```

`di` 디렉토리를 만들고 `ApplicationComponent::class`를 install 하여 앱이 실행되는 동안 사용될 Room, Retrofit2 싱글톤을 정의한다.

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object AppModule {

    @Singleton
    @Provides
    fun provideShoppingItemDatabase(
        @ApplicationContext context: Context
        ) = Room.databaseBuilder(context, ShoppingItemDatabase::class.java, DATABASE_NAME).build()

    @Singleton
    @Provides
    fun provideShoppingDao(
        database: ShoppingItemDatabase
    ) = database.shoppingDao()

    @Singleton
    @Provides
    fun providePixabayApi(): PixabayAPI {
        return Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .baseUrl(BASE_URL)
            .build()
            .create(PixabayAPI::class.java)
    }
}
```

## References

[Setting up Project & Room DB - Testing on Android - Part 5](https://www.youtube.com/watch?v=2p6cfaIK3_g&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=5)