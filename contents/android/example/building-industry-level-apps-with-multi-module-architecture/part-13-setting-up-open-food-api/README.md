# Setting up Open Food API

이번엔 Open Food API를 이용해 음식을 검색하기 위한 Retrofit API 인터페이스를 설정할 것이다.

`tracker_data` 모듈에 `remote` 패키지를 생성한다. 그리고 `remote` 패키지에 `openFoodApi` 인터페이스를 생성한다.

```kotlin
interface OpenFoodApi {

    @GET("cgi/search.pl?search_simple=1&json=1&action=process&fields=product_name,nutriments,image_front_thumb_url")
    suspend fun searchFood(
            @Query("search_terms") query: String,
            @Query("page") page: Int,
            @Query("page_size") pageSize: Int
    ): SearchDto

    companion object {
        const val BASE_URL = "https://us.openfoodfacts.org/"
    }
}
```

반환 값인 `SearchDto`를 만들어주기 위해 `remote/dto` 패키지를 생성한 후 다음 3가지 data class를 작성해준다.

```kotlin
data class Nutriments(
        @field:Json(name = "carbohydrates_100g")
        val carbohydrates100g: Double,
        @field:Json(name = "energy-kcal_100g")
        val energyKcal100g: Double,
        @field:Json(name = "fat_100g")
        val fat100g: Double,
        @field:Json(name = "proteins_100g")
        val proteins100g: Double
)
```

```kotlin
data class Product(
        @field:Json(name = "image_front_thumb_url")
        val imageFrontThumbUrl: String?,
        val nutriments: Nutriments,
        @field:Json(name = "product_name")
        val productName: String?
)
```

```kotlin
data class SearchDto(
        val products: List<Product>,
)
```

`tracker_data` 모듈에서 사용되는 의존성들을 제공해주기 위한 모듈을 만들어준다. `tracker_data` 모듈에 `di` 패키지를 생성하고 `TrackerDataModule`을 작성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object TrackerDataModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        // Http 로깅하기 위함
        return OkHttpClient.Builder()
                .addInterceptor(HttpLoggingInterceptor().apply {
                    level = HttpLoggingInterceptor.Level.BODY
                })
                .build()
    }

    @Provides
    @Singleton
    fun provideOpenFoodApi(client: OkHttpClient): OpenFoodApi {
        return Retrofit.Builder()
                .baseUrl(OpenFoodApi.BASE_URL)
                .addConverterFactory(MoshiConverterFactory.create())
                .client(client)
                .build()
                .create()
    }
}
```