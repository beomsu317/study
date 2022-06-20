# Onboarding Components

`onboarding` 모듈에서 필요한 2가지 컴포넌트를 만들어보자.

`onboarding_presentation` 모듈의 `components` 패키지에 `SelectableButton` composable을 작성한다.

```kotlin
@Composable
fun SelectableButton(
    text: String,
    isSelected: Boolean,
    color: Color,
    selectedTextColor: Color,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    textStyle: TextStyle = MaterialTheme.typography.button
) {
    Box(
        contentAlignment = Alignment.Center,
        modifier = modifier
            .clip(RoundedCornerShape(100.dp))
            .border(
                width = 2.dp,
                color = color,
                shape = RoundedCornerShape(100.dp)
            )
            .background(
                color = if(isSelected) color else Color.Transparent,
                shape = RoundedCornerShape(100.dp)
            )
            .clickable {
                onClick()
            }
            .padding(LocalSpacing.current.spaceMedium)
    ) {
        Text(
            text = text,
            style = textStyle,
            color = if(isSelected) selectedTextColor else color
        )
    }
}
```

`onboarding_presentation` 모듈의 `components` 패키지에 `UnitTextField` composable을 작성한다.

```kotlin
@Composable
fun UnitTextField(
    value: String,
    onValueChange: (String) -> Unit,
    unit: String,     // years, cm, ...
    modifier: Modifier = Modifier,
    textStyle: TextStyle = TextStyle(
        color = MaterialTheme.colors.primaryVariant,
        fontSize = 70.sp
    )
) {
    val spacing = LocalSpacing.current
    Row(
        modifier = modifier,
        horizontalArrangement = Arrangement.Center
    ) {
        BasicTextField(
            value = value,
            onValueChange = onValueChange,
            textStyle = textStyle,
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Number
            ),
            singleLine = true,
            modifier = Modifier
                .width(IntrinsicSize.Min) // 필요한 만큼 차지하는 옵션 -> fixed 된 경우 텍스트가 중앙에 위치하지 않게 된다. xml의 wrap_content와 같다.
                .alignBy(LastBaseline) // 기준선이 글자 bottom
        )
        Spacer(modifier = Modifier.width(spacing.spaceSmall))
        Text(
            text = unit,
            modifier = Modifier.alignBy(LastBaseline)
        )
    }
}
```