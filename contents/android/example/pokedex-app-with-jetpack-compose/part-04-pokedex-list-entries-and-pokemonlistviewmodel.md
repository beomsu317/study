# Pokédex List Entries & PokémonListViewModel

이번에는 포켓몬 리스트 엔트리와 `PokémonListViewModel`을 설정한다. `models/` 패키지를 만들고 하위에 `PokedexListEntry` data class를 생성한다.

```kotlin
data class PokedexListEntry(
    val pokemonName: String,
    val imageUrl: String,
    val number: Int
)
```

`PokemonListViewModel`을 생성하고 Dominant Color를 생성해주는 함수를 구현한다.

```kotlin
@HiltViewModel
class PokemonListViewModel @Inject constructor(
    private val repository: PokemonRepository
): ViewModel() {

    fun clacDominantColor(drawable: Drawable, onFinish: (Color) -> Unit) {
        // paleatte API 포맷에 맞지 않으므로 bitmap을 mutable로 변경한 다음 Bitmap.Config로 변경해 API를 사용가능하도록 한다.
        val bmp = (drawable as BitmapDrawable).bitmap.copy(Bitmap.Config.ARGB_8888, true)

        Palette.from(bmp).generate { palette ->
            palette?.dominantSwatch?.rgb?.let { colorValue ->
                onFinish(Color(colorValue))
            }
        }
    }
}
```

리스트 구현을 위해 `PokedexEntry`와 `PokedexRow`를 구현해준다.

```kotlin
@Composable
fun PokedexEntry(
    entry: PokedexListEntry,
    navController: NavController,
    modifier: Modifier = Modifier,
    // 해당 네비게이션의 스코프를 가진 ViewModel을 제공한다.
    viewModel: PokemonListViewModel = hiltViewModel()
) {
    val defaultDominantColor = MaterialTheme.colors.surface
    var dominantColor by remember {
        mutableStateOf(defaultDominantColor)
    }
		// coil 사용
    val painter = rememberImagePainter(
        data = entry.imageUrl,
        builder = {
            crossfade(true)
        },
    )

    Box(
        contentAlignment = Center,
        modifier = modifier
            .shadow(5.dp, RoundedCornerShape(10.dp))
            .clip(RoundedCornerShape(10.dp))
            .aspectRatio(1f)
            .background(
                Brush.verticalGradient(
                    listOf(
                        dominantColor,
                        defaultDominantColor
                    )
                )
            )
            .clickable {
                navController.navigate(
                    "pokemon_detail_screen/${dominantColor.toArgb()}/${entry.pokemonName}"
                )
            }
    ) {
        Column {
            Image(
                painter = painter,
                contentDescription = entry.pokemonName,
                modifier = Modifier
                    .size(120.dp)
                    .align(CenterHorizontally)
                    .weight(0.8f),
            )
            when (painter.state) {
                is ImagePainter.State.Loading -> {
                    CircularProgressIndicator(
                        color = MaterialTheme.colors.primary,
                        modifier = Modifier
                            .fillMaxSize()
                            .scale(0.5f)
                            .align(CenterHorizontally)
                    )
                }
                is ImagePainter.State.Error -> {

                }
                is ImagePainter.State.Success -> {
										// painter
                    LaunchedEffect(key1 = painter) {
                        launch {
                            val image = painter.imageLoader.execute(painter.request).drawable
                            viewModel.calcDominantColor(image!!) { color ->
                                dominantColor = color
                            }
                        }
                    }
                }
            }
            Text(
                text = entry.pokemonName,
                fontFamily = RobotoCondensed,
                fontSize = 20.sp,
                textAlign = TextAlign.Center,
                modifier = Modifier
                    .fillMaxWidth()
                    .weight(0.2f)
            )
        }
    }
}

@Composable
fun PokedexRow(
    rowIndex: Int,
    entries: List<PokedexListEntry>,
    navController: NavController
) {
    Column {
        Row {
            PokedexEntry(
                entry = entries[rowIndex * 2],
                navController = navController,
                modifier = Modifier.weight(1f)
            )
            Spacer(modifier = Modifier.width(16.dp))
            if(entries.size >= rowIndex * 2 + 2) {
                PokedexEntry(
                    entry = entries[rowIndex * 2 + 1],
                    navController = navController,
                    modifier = Modifier.weight(1f)
                )
            } else {
                Spacer(modifier = Modifier.weight(1f))
            }
        }
        Spacer(modifier = Modifier.height(16.dp))
    }
```

## References

* [Pokédex List Entries & PokémonListViewModel - Pokédex App With Jetpack Compose - Part 4](https://www.youtube.com/watch?v=D06EV3PngJY&list=PLQkwcJG4YTCTimTCpEL5FZgaWdIZQuB7m&index=4)