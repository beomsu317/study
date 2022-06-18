# Data conversion

`DataConversion`은 값의 리스트를 serialize 및 deserialize 할 수 있게 해주는 플러그인이다.

기본적으로 원시 타입과 enum을 처리하지만, 추가적인 타입을 처리하도록 구성할 수 있다.

[Locations plugin](https://ktor.io/docs/locations.html)을 사용하고 파라미터의 일부로 커스텀 타입을 지원하려는 경우, 이 서비스를 사용해 새로운 커스텀 컨버터를 추가할 수
있다.

## **Basic installation**

DataConversion을 설치하는 것은 쉬우며 원시 타입을 포함해야 한다.

```kotlin
install(DataConversion)
```

## **Adding converters**

DataConversion 구성, 타입 변환을 정의하는 `convert<T>` 메서드를 제공한다. 내부에서 디코더와 인코더에 콜백을 허용하는 `decode`와 `encode` 메서드를 제공해야 한다.

- decode callback : `converter: (values: List<String>, type: Type) -> Any?`
    - URL에서 반복되는 값을 나타내는 문자열 목록인 `values`(예: `a=1&a=2`)를 받고 변환할 `type`을 허용한다. decoded value를 반환해야 한다.
- encode callback : `converter: (value: Any?) -> List<String>`
    - 임의의 value를 받으며, 값을 나타내는 문자열 리스트를 반환해야 한다. 단일 요소의 리스트를 반환할 때 `key=item1`으로 serialize 된다. 여러 값은 쿼리
      스트링에서 `samekey=item1&samekey=item2`로 serialize 된다.

```kotlin
install(DataConversion) {
    convert<Date> { // this: DelegatingConversionService
        val format = SimpleDateFormat.getInstance()

        decode { values, _ -> // converter: (values: List<String>, type: Type) -> Any?
            values.singleOrNull()?.let { format.parse(it) }
        }

        encode { value -> // converter: (value: Any?) -> List<String>
            when (value) {
                null -> listOf()
                is Date -> listOf(SimpleDateFormat.getInstance().format(value))
                else -> throw DataConversionException("Cannot convert $value as Date")
            }
        }
    }
}
```

다른 잠재적인 용도는 특정 enum이 serialize 되는 방식을 커스터마이즈하는 것이다. 기본적으로 enum은 대소문자를 구분하는 `.name`을 사용하여 serialize 및 deserialize 된다. 하지만
예를 들어 소문자로 serialize하고 대소문자를 구분하지 않는 방식으로 deserialize 할 수 있다.

```kotlin
enum class LocationEnum {
    A, B, C
}

@Location("/")
class LocationWithEnum(val e: LocationEnum)

@Test
fun `location class with custom enum value`() = withLocationsApplication {
    application.install(DataConversion) {
        convert(LocationEnum::class) {
            encode { if (it == null) emptyList() else listOf((it as LocationEnum).name.toLowerCase()) }
            decode { values, type -> LocationEnum.values().first { it.name.toLowerCase() in values } }
        }
    }
    application.routing {
        get<LocationWithEnum> {
            call.respondText(call.locations.resolve<LocationWithEnum>(LocationWithEnum::class, call).e.name)
        }
    }

    urlShouldBeHandled("/?e=a", "A")
    urlShouldBeHandled("/?e=b", "B")
}
```

# **Accessing the service**

DataConversion 서비스에 쉽게 접근할 수 있다.

```kotlin
val dataConversion = call.conversionService
```

## **The ConversionService interface**

```kotlin
interface ConversionService {

    fun fromValues(values: List<String>, type: TypeInfo): Any?
    fun toValues(value: Any?): List<String>
}
```

```kotlin
class DelegatingConversionService(
    private val klass: KClass<*>,
    private val decoder: ((values: List<String>) -> Any?)?,
    private val encoder: ((value: Any?) -> List<String>)?
) : ConversionService {

    fun fromValues(values: List<String>, type: TypeInfo): Any?
    fun toValues(value: Any?): List<String>
}
```

## References

* [Data conversion | Ktor](https://ktor.io/docs/data-conversion.html)