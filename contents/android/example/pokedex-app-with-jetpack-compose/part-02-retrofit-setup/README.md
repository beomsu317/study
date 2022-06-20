# Retrofit Setup

이번에는 Pokemon API에 사용해야 할  Retrofit을 설정한다.

[https://pokeapi.co/](https://pokeapi.co/)에 접속하고 “Direct link to results”를 클릭해  API 스펙을 확인한다. `data/remote/responses` 패키지를 만들고 그 하위에 위 API 스펙의 포켓몬들의 data class를 만들어준다. 또한 [https://pokeapi.co/api/v2/pokemon/](https://pokeapi.co/api/v2/pokemon/)에 접속해 페이징할 수 있도록 `PokemonList`  data class를 만든다. 다음은 `Pokemon`과  `PokemonList` 코드이다.

```kotlin
data class Pokemon(
    val abilities: List<Ability>,
    val base_experience: Int,
    val forms: List<Form>,
    val game_indices: List<GameIndice>,
    val height: Int,
    val held_items: List<HeldItem>,
    val id: Int,
    val is_default: Boolean,
    val location_area_encounters: String,
    val moves: List<Move>,
    val name: String,
    val order: Int,
    val past_types: List<Any>,
    val species: Species,
    val sprites: Sprites,
    val stats: List<Stat>,
    val types: List<Type>,
    val weight: Int
)
```

```kotlin
data class PokemonList(
    val count: Int,
    val next: String,
    val previous: Any,
    val results: List<Result>
)
```

API를 요청하기 위해 `data.remote` 패키지 하위에 `PokeApi`를 인터페이스를 생성한다.

```kotlin
interface PokeApi {

    @GET("pokemon")
    suspend fun getPokemonList(
        @Query("limit") limit: Int,
        @Query("offset") offset: Int
    ): PokemonList

    @GET("pokemon/{name}")
    suspend fun getPokemonInfo(
        @Path("name") name: String
    ): Pokemon
}
```

`util` 패키지를 만들고 `Resource` sealed class를 만들어준다. 이를 통해 데이터에 대한 상태를 알려줄 수 있다.

```kotlin
sealed class Resource<T>(val data: T? = null, val message: String? = null) {
    class Success<T>(data: T) : Resource<T>(data)
    class Error<T>(message: String, data: T? = null) : Resource<T>(data, message)
    class Loading<T>(data: T? = null) : Resource<T>(data)
}
```

`repository` 패키지를 만들고 `PokemonRepository()`를 생성한다.

```kotlin
// 나중에 ViewModel에 인젝션될 것이기 때문에 @ActivityScoped 어노테이션 추가
@ActivityScoped
class PokemonRepository @Inject constructor(
    private val api: PokeApi
) {

    suspend fun getPokemonList(limit: Int, offset: Int): Resource<PokemonList> {
        val response = try {
            api.getPokemonList(limit, offset)
        } catch (e:Exception) {
            return Resource.Error("An unknown error occured.")
        }
        return Resource.Success(response)
    }

    suspend fun getPokemonInfo(pokemonName: String): Resource<Pokemon> {
        val response = try {
            api.getPokemonInfo(pokemonName)
        } catch (e:Exception) {
            return Resource.Error("An unknown error occured.")
        }
        return Resource.Success(response)
    }
}
```

`util` 패키지 하위에 `BASE_URL`을 선언한다.

```kotlin
object Constants {

    const val BASE_URL = "https://pokeapi.co/api/v2/"
}
```

`di` 패키지 하위에 다음 `PokemonRepository`와 `PokeApi`를 싱글턴으로 생성한다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Singleton
    @Provides
    fun providePokemonRepository(
        api: PokeApi
    ) = PokemonRepository(api)

    @Singleton
    @Provides
    fun providePokemonApi(): PokeApi {
        return Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .baseUrl(Constants.BASE_URL)
            .build()
            .create(PokeApi::class.java)
    }
}
```

## References

* [Retrofit Setup - MVVM Pokédex App with Jetpack Compose - Part 2](https://www.youtube.com/watch?v=aaChg9aJDW4&list=PLQkwcJG4YTCTimTCpEL5FZgaWdIZQuB7m&index=2)