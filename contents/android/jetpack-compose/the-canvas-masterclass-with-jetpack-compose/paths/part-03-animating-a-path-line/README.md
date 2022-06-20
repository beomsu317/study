# Animating a Path Line

Path를 애니메이션하는 방법을 알아보자.

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val pathPortion = remember {
                androidx.compose.animation.core.Animatable(initialValue = 0f)
            }
            LaunchedEffect(key1 = true) {
                pathPortion.animateTo(
                    targetValue = 1f,
                    animationSpec = tween(
                        durationMillis = 5000
                    )
                )
            }
            val path = Path().apply {
                moveTo(100f, 100f)
                quadraticBezierTo(400f, 400f, 100f, 400f)
            }
            val outPath = Path()
            // create a new path out of existing path that consist only specific protion of that path
            PathMeasure().apply {
                setPath(path, false)
                // get a specific segment of out actual path of 'path'
                // result of segment calculation will be saved in outPath
                getSegment(0f, pathPortion.value * length, outPath)
            }
            Canvas(modifier = Modifier.fillMaxSize()) {
                drawPath(
                    path = outPath,
                    color = Red,
                    style = Stroke(width = 5.dp.toPx())
                )
            }
        }
    }
}
```

<div align="center">
<img src="img/result.gif" width="40%">
</div>