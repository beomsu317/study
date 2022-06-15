# How to Make a 3D Animated Drop Down

이번엔 3D Animated Drop Down을 만들어보자. `DropDown` Composable 함수를 다음과 같이 작성한다.

```kotlin
@Composable
fun DropDown(
    text: String,
    modifier: Modifier = Modifier,
    initiallyOpened: Boolean = false,
    content: @Composable () -> Unit
) {
    var isOpen by remember {
        mutableStateOf(initiallyOpened)
    }
    val alpha = animateFloatAsState(
        targetValue = if (isOpen) 1f else 0f,
        animationSpec = tween(
            durationMillis = 300
        )
    )
    val rotateX = animateFloatAsState(
        // open 되었을 때 -90 에서 0으로 애니메이션 한다.
        targetValue = if (isOpen) 0f else -90f,
        animationSpec = tween(
            durationMillis = 300
        )
    )
    Column(
        modifier = modifier
            .fillMaxWidth()
    ) {
        Row(
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically,
            modifier = Modifier
                .fillMaxWidth()
        ) {
            Text(
                text = text,
                color = Color.White,
                fontSize = 16.sp
            )
            Icon(
                imageVector = Icons.Default.ArrowDropDown,
                contentDescription = "Open or close the drop down",
                tint = Color.White,
                modifier = Modifier
                    .clickable {
                        isOpen = !isOpen
                    }
                    // x scale은 동일하며, y scale은 열렸을 때 -1f, 닫혔을 때 1f
                    // 이렇게 하면 다른 이미지를 리로딩하지 않아 효율적
										// ArrowDown 버튼 하나만으로 구현 가능
                    .scale(1f, if (isOpen) -1f else 1f)
            )
        }
        Spacer(modifier = Modifier.height(10.dp))
        Box(
            contentAlignment = Alignment.Center,
            modifier = Modifier
                .fillMaxWidth()
                // 어떠한 3d 변형이라도 가능하도록 만들어줌
                .graphicsLayer {
                    // pivot을 지정해주지 않으면 중간이 pivot이 되어 그 기준으로 애니메이션 하므로 다음과 같이 pivot을 정해준다.
                    // x는 50%로 주어 중간에 위치하게 하고, y는 0을 주어 0 위치에 둔다.
                    transformOrigin = TransformOrigin(0.5f, 0.0f)
                    rotationX = rotateX.value
                }
                // 알파 값을 줘서 열렸을 때 보여지고, 닫혔을 때 안보여지도록 한다.
                .alpha(alpha = alpha.value)
        ) {
            content()
        }
    }
}
```

`MainActivity`에서 다음과 같이 `DropDown`을 호출하고, `Text`를 추가하여 해당 텍스트가 내용으로 보여지도록 한다.

```kotlin
class MainActivity : ComponentActivity() {
    @ExperimentalComposeUiApi
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Surface(
                color = Color(0xFF101010),
                modifier = Modifier.fillMaxSize()
            ) {
                DropDown(
                    text = "Hello World!",
                    modifier = Modifier
                        .padding(15.dp)
                ) {
                    Text(
                        text = "This is now revealed!",
                        modifier = Modifier
                            .fillMaxWidth()
                            .height(100.dp)
                            .background(Color.Green)
                    )
                }
            }
        }
    }
}
```

<div align="center">
<img src="img/part-16/result.gif" width="40%">
</div>

## References

* [How to Make a 3D Animated Drop Down in Jetpack Compose - Android Studio Tutorial](https://www.youtube.com/watch?v=WdQUDHOwlgE&list=PLQkwcJG4YTCSpJ2NLhDTHhi6XBNfk9WiC&index=16)