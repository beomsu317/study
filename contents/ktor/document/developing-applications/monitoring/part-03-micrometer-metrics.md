# Micrometer metrics

[MicrometerMetrics](https://api.ktor.io/ktor-features/ktor-metrics-micrometer/ktor-metrics-micrometer/io.ktor.metrics.micrometer/-micrometer-metrics/index.html)
플러그인은 Ktor 서버 애플리케이션의 [Micrometer](https://micrometer.io/docs) metrics를 활성화하고 Prometheus, JMX, Elastic 등의 모니터링 시스템을 선택할
수 있다. 기본적으로 Ktor는 HTTP 요청을 모니터링을 위한 metric과 JVM 모니터링을 위한 low-level metric 셋을 노출한다. 이 metric을 커스터마이징하거나 새롭게 만들 수 있다.

## **Add dependencies**

`MicrometerMetrics`를 활성화하기 위해 다음 아티팩트들을 추가한다.

- `ktor-metrics-micrometer` 디펜던시.

```kotlin
implementation("io.ktor:ktor-metrics-micrometer:$ktor_version")
```

- 모니터링 시스템에 필요한 디펜던시. 다음은 Prometheus 아티팩트.

```kotlin
implementation("io.micrometer:micrometer-registry-prometheus:$prometeus_version")
```

## **Install MicrometerMetrics**

`MicrometerMetrics` 플러그인 설치를 위해 `install` 함수에 이 플러그인을 전달한다.

```kotlin
import io.ktor.features.*

// ...
fun main() {
    embeddedServer(Netty, port = 8080) {
        install(MicrometerMetrics)
        // ...
    }.start(wait = true)
}
```

```kotlin
import io.ktor.features.*

// ...
fun Application.module() {
    install(MicrometerMetrics)
    // ...
}
```

### **Exposed metrics**

Ktor는 HTTP 요청 모니터링을 위해 다음 metric을 노출한다.

- `ktor.http.server.requests.active` : 동시 HTTP 요청의 양을 계산하는 [gauge](https://micrometer.io/docs/concepts#_gauges). 이 측정항목은
  태그를 제공하지 않는다.
- `ktor.http.server.requests` : 각 요청의 시간을 측정하기 위한 [timer](https://micrometer.io/docs/concepts#_timers). 이 metric은 요청된
  URL `address`, HTTP `method`, route 핸들링 요청에 대한 `route` 등 요청 데이터를 모니터링하기 위한 태그 셋을 제공한다.

HTTP metric 외에도 Ktor는 JVM을 모니터링하기 위한 metric 셋을 노출한다.

## **Create a registry**

`MicrometerMetrics` 설치 후 모니터링 시스템에 대한 registry를 생성하고 registry 속성에 할당해야 한다. 다음 예제에서는 다른 route 핸들러에서 이 registry를 재사용할 수 있는
기능을 갖도록 `PrometheusMeterRegistry`가 `install` 블록 외부에서 생성되었다.

```kotlin
fun Application.module(testing: Boolean = false) {
    val appMicrometerRegistry = PrometheusMeterRegistry(PrometheusConfig.DEFAULT)
    install(MicrometerMetrics) {
        registry = appMicrometerRegistry
    }
}
```

## **Configure metrics**

`MicrometerMetrics`
플러그인은 [MicrometerMetrics.Configuration](https://api.ktor.io/ktor-features/ktor-metrics-micrometer/ktor-metrics-micrometer/io.ktor.metrics.micrometer/-micrometer-metrics/-configuration/index.html)
을 사용해 접근할 수 있는 다양한 설정 옵션을 제공한다.

### Timers

각 타이머마다 태그를 커스터마이징하기 위해 각 요청에 대해 호출되는 `timers` 함수를 사용해야 한다.

```kotlin
install(MicrometerMetrics) {
    // ...
    timers { call, exception ->
        tag("region", call.request.headers["regionId"])
    }
}
```

### **Distribution statistics**

`distributionStatisticConfig` 속성을
사용해 [distribution statistics](https://micrometer.io/docs/concepts#_configuring_distribution_statistics)을 구성한다.

```kotlin
install(MicrometerMetrics) {
    distributionStatisticConfig = DistributionStatisticConfig.Builder()
        .percentilesHistogram(true)
        .maximumExpectedValue(Duration.ofSeconds(20).toNanos().toDouble())
        .serviceLevelObjectives(
            Duration.ofMillis(100).toNanos().toDouble(),
            Duration.ofMillis(500).toNanos().toDouble()
        )
        .build()
}
```

### **JVM and system metrics**

HTTP metric 외에도 Ktor는 JVM 모니터링을 위한 metric 셋을 노출한다. `meterBinders` 속성을 사용해 이 metric 목록을 커스터마이징 할 수 있다.

```kotlin
install(MicrometerMetrics) {
    meterBinders = listOf(
        JvmMemoryMetrics(),
        JvmGcMetrics(),
        ProcessorMetrics()
    )
}
```

또한 빈 리스트를 할당해 비활성화 할 수 있다.

## **Prometheus: expose a scrape endpoint**

Prometheus를 모니터링 시스템으로 사용하면 HTTP 엔드포인트를 Prometheus scraper에 노출해야 한다. Ktor는 다음 방법으로 수행할 수 있다.

1. 필요한 주소(`/metric`)로 `GET` 요청을 허용하는 route를 생성한다.
2. `call.respond`를 사용해 스크래핑 데이터를 Prometheus에 보낸다.

    ```kotlin
    fun Application.module(testing: Boolean = false) {
        val appMicrometerRegistry = PrometheusMeterRegistry(PrometheusConfig.DEFAULT)
        install(MicrometerMetrics) {
            registry = appMicrometerRegistry
        }
        routing {
            get("/metrics") {
                call.respond(appMicrometerRegistry.scrape())
            }
        }
    }
    ```

## References

* [Micrometer metrics | Ktor](https://ktor.io/docs/micrometer-metrics.html)