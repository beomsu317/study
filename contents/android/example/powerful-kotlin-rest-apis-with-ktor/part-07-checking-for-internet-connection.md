# Checking for internet connection

이번에는 유저가 인터넷 연결이 되어있는지 아닌지에 대한 코드를 구현할 것이다. 에어플레인 모드나 네트워크 연결이 어떤 문제 때문에 연결되지 않는 경우 이를 감지하여 유저에게 알리고, API 요청을 중단한다. 만약
API 요청이 수행되는 경우 크래시가 발생할 수 있다.

`other` 패키지에 `InternetChecker` 파일을 생성 및 작성한다.

```kotlin
fun checkForInternetConnection(context: Context): Boolean {
    val connectivityManager = context.getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        val activeNetwork = connectivityManager.activeNetwork ?: return false
        val capabilities = connectivityManager.getNetworkCapabilities(activeNetwork) ?: return false
        return when {
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) -> true
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_ETHERNET) -> true
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR) -> true
            capabilities.hasTransport(NetworkCapabilities.TRANSPORT_VPN) -> true
            else -> false
        }
    }
    return connectivityManager.activeNetworkInfo?.isAvailable ?: false
}
```