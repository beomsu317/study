# Implementing the caching mechanism

이번엔 캐싱 매커니즘을 구현해보자. 인터넷 연결 없이도 로컬 디비에서 노트를 가져올 수 있도록 만들자.

`NetworkBoundResource`를 작성하여 구현할 것이다. 자바에서는 구현이 복잡했었지만, 코틀린의 경우 flow를 반환하여 구현이 상대적으로 간단한다. `NetworkBoundResource`는
single source of truth에 기반하는데, 이는 하나의 데이터 소스에서만 데이터를 반환해야 한다는 의미이다. 우리는 로컬 디비를 소스로 사용할 것이며, 네트워크에서 전달된 데이터를 로컬 디비에 저장한
다음, 로컬 디비에 저장된 데이터를 보여주는 방식으로 구현한다.

`other` 패키지에 `NetworkBoundResource` 파일을 생성 및 작성한다. `NetworkBoundResource`는 다른 프로젝트에서도 사용 가능하다.

```kotlin
// inline is keyword for compiler
// take the code that function and put it to the place where we call that function
// so don't need all that address stuff
// ResultType -> Local Database return type
// RequestType -> whatever your api returns basically
inline fun <ResultType, RequestType> networkBoundResource(
    crossinline query: () -> Flow<ResultType>,      // declare logic how to get data from database
    crossinline fetch: suspend () -> RequestType,   // define the logic to get data from actual api, RequestType is actual network response that we get from api
    crossinline saveFetchResult: suspend (RequestType) -> Unit, // got that response from fetch() function, insert data to local database
    crossinline onFetchFailed: (Throwable) -> Unit = { Unit },  // define logic what we want to do when something wrong
    crossinline shouldFetch: (ResultType) -> Boolean = { true } // determine if we actually want to fetch data
) = flow {
        emit(Resource.loading(null))
        // get data from database
        val data = query().first()

        val flow = if (shouldFetch(data)) { // fetch data from api
            // we have data already
            emit(Resource.loading(data))

            try {
                val fetchedResult = fetch()     // get data from api
                saveFetchResult(fetchedResult)  // store in local database
                query().map { Resource.success(it) }    // load data from local database
            } catch (t: Throwable) {
                onFetchFailed(t)
                query().map { Resource.error("Couldn't reach server. It might be down", it) }
            }
        } else {
            query().map { Resource.success(it) }
        }
        emitAll(flow)
    }
```

이제 이를 기반으로 `NoteRepository`에 `getAllNotes()` 함수를 작성해준다.

```kotlin
class NoteRepository @Inject constructor(
    private val noteDao: NoteDao,
    private val noteApi: NoteApi,
    private val context: Application // for check internet connection
) {

    fun getAllNotes(): Flow<Resource<List<Note>>> {
        return networkBoundResource(
            query = {
                noteDao.getAllNotes()
            },
            fetch = {
                noteApi.getNotes()
            },
            saveFetchResult = { response ->
                response.body()?.let {
                    // TODO: insert notes in database
                }
            },
            shouldFetch = {
                // 1. timestamp
                // 2. always fetch it
                checkForInternetConnection(context)
            }
        )
    }
}
```