# How to Do PROPER Permission Handling in Jetpack Compose

Jetpack Compose에서 여러개의 권한을 적절하게 처리하는 방법을 알아보자.

Compose에서 쉽게 권한을 처리할 수 있는 라이브러리 디펜던시를 추가하자.

```groovy
implementation "com.google.accompanist:accompanist-permissions:0.21.1-beta" 
```

보통 권한 체크를 `onCreate`, `onViewCreated`에 위치시키는데, 여기에 두게 되면 유저가 권한을 승인한 후 앱 세팅으로 이동해 거부할 수 있다는 것을 간과하게 된다. 따라서 앱으로 다시 이동하면
앱은 거부된 상태로 실행되게 된다. 그래서 `onStart` 에 위치시켜야 한다.

`MainActivity`를 다음과 같이 구현해준다.

```kotlin
@ExperimentalPermissionsApi
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val permissionsState = rememberMultiplePermissionsState(
                    permissions = listOf(
                            Manifest.permission.RECORD_AUDIO,
                            Manifest.permission.CAMERA,
                    )
            )

            val lifecycleOwner = LocalLifecycleOwner.current
            DisposableEffect(
                    key1 = lifecycleOwner,
                    effect = {
                        val observer = LifecycleEventObserver { _, event ->
                            if (event == Lifecycle.Event.ON_RESUME) {
                                permissionsState.launchMultiplePermissionRequest()
                            }
                        }
                        lifecycleOwner.lifecycle.addObserver(observer)
                        onDispose {
                            lifecycleOwner.lifecycle.removeObserver(observer)
                        }
                    }
            )

            Column(
                    modifier = Modifier.fillMaxSize(),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.Center
            ) {
                permissionsState.permissions.forEach { perm ->
                    when (perm.permission) {
                        Manifest.permission.CAMERA -> {
                            when {
                                perm.hasPermission -> {
                                    Text(text = "Camera permission accepted")
                                }
                                // deny 1 time
                                perm.shouldShowRationale -> {
                                    Text(text = "Camera permission is need to access the camera")
                                }
                                perm.isPermanentlyDenied() -> {
                                    Text(
                                            text = "Camera permission was permanatly denied. You can " +
                                                    "enable it in the app settings"
                                    )
                                }
                            }
                        }
                        Manifest.permission.RECORD_AUDIO -> {
                            when {
                                perm.hasPermission -> {
                                    Text(text = "Record audio permission accepted")
                                }
                                // deny 1 time
                                perm.shouldShowRationale -> {
                                    Text(text = "Record audio permission is need to access the camera")
                                }
                                perm.isPermanentlyDenied() -> {
                                    Text(
                                            text = "Record audio permission was permanatly denied. You can " +
                                                    "enable it in the app settings"
                                    )
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}

@ExperimentalPermissionsApi
fun PermissionState.isPermanentlyDenied(): Boolean {
    return !shouldShowRationale && !hasPermission
}
```

<div align="center">
<img src="img/result.gif" width="40%">
</div>

## References

* [How to Do PROPER Permission Handling in Jetpack Compose - Android Studio Tutorial](https://www.youtube.com/watch?v=ltHN50BdDc4)