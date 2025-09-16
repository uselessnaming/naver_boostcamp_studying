# Network 라이브러리

네트워크 라이브러리는 모두 인터넷이 필요하므로 `INTERNET`과 `ACCESS_NETWORK_STATE` 권한을 모두 허용해 준 후 사용해야 한다

## HttpUrlConnection
### 사용
HttpUrlConnection은 HTTP 통신을 위해 사용하는 가장 기본적인 URL Connection 클래스를 상속하며, abstract class이다        

처음에 `HttpUrlConnection()`으로 생성자를 만드려고 하면 빨간 줄과 함께 오류임을 알려준다        
이는 생성자가 protected로 선언되어 기본적으로 생성할 수 없음을 말해준다     

그래서 사용할 때 URL 객체의 `openConnection()`로 인스턴스를 받고 `setRequestMethod()`로 REST API 메소드를 정할 수 있다

다만 `url.openConnection()`만 할 경우 HttpUrlConnection()이 아닌 그 부모인 URL Connection 클래스로 선언되므로 as를 통한 타입 캐스팅이 필요하다

> 만약 as 캐스팅을 하지 않게 되면 HTTP Url Connection 클래스가 아니기 때문에 REST API 메소드를 정하는 함수를 사용할 수 없다

연결하게 된 후에는 connection에 대해 timeout 시간, request method, header에 사용할 프로퍼티 정보들을 추가한다       

### 종료
`url.openConnection()`에서 반았던 객체에 대해서 `responseCode`를 확인하여 성공했는지 분기가 가능하다        

BufferedReader와 InputStreamReader를 활용해 응답을 response로 입력받고, 이를 json으로 decoding 해주면 결과를 확인할 수 있다

그리고 try - catch 문으로 연결 시 발생하는 Exception에 대응하고 있다면 finally 를 추가하여 어떤 쪽이 되었던 끝나게 되면 `disconnect()`를 통해 메모리 누수를 방지해야 한다

### AsyncTask Deprecated
기존에는 AsyncTask를 활용하여 비동기적으로 실행되는 쓰레드 관리했다     
다만 Android 11(API 30)부터는 deprecated 되었다     

이유로는 순차 실행으로 인한 성능 저하와 메모리 누수가 있다      
Activity의 종료 시점과 AsyncTask의 끝나는 시점의 차이 있어서 Configuration Change가 발생하면 기존 Activity의 AsyncTask가 메모리에서 제거되지 않는 문제가 있었다

### 사용 경험
하나의 API를 사용할 때마다 Url에 따른 HttpUrlConnection 객체를 생성해야 하며, 그 때마다 Header를 각각의 번거로운 설정을 해야 한다

<details>
<summary>예시 코드</summary>

```kotlin
class GithubHttpUrlConnection {
    fun getAllIssues(): Result<List<Issue>> {
        val url = URL("$baseUrl/repos/")
        val connection = url.openConnection() as HttpURLConnection

        try{
            with(connection){
                connectTimeout = 5000
                requestMethod = "GET"

                setRequestProperty("X-Github-Api-Version", "2022-11-28")
                setRequestProperty("Accept", "application/vnd.github+json")
                setRequestProperty("Authorization", BuildConfig.GIT_TOKEN)

                if (responseCode == HttpURLConnection.HTTP_OK){
                    BufferedReader(InputStreamReader(inputStream)).use{ reader ->
                        val response = StringBuffer()
                        var inputLine: String?
                        while(reader.readLine().also{inputLine = it} != null){
                            response.append(inputLine)
                        }
                        Log.d("VIEWMODEL LOGGING", "response: ${response.toString()}")
                        val data = json.decodeFromString<List<Issue>>(response.toString())
                        return Result.success(value = data)
                    }
                } else {
                    return Result.failure(exception = Exception("response code : $responseCode"))
                }
            }
        } catch(e: Exception){
            return Result.failure(exception = e)
        } finally{
            connection.disconnect()
        }
    }
}
```
</details>

위와 같은 코드를 이용했는데, 결국 보일러 코드도 많이 생성되고 무엇보다 URL 별로 계속 생성해야 하는 부분이 불편하다고 생각이 들었다      

별도의 라이브러리 사용을 하지 않기 때문에 가볍게 사용할 수 있다는 장점이 있다.      
하지만 그만큼 사용 방법이 복잡하고 `NetworkOnMainThreadException`을 방지해야 하는 단점 또한 있다

그리고 HttpClient 클래스는 안드로이드 5.1 (API 22) 부터 Deprecated되었고, 안드로이드 6.0 부터는 아예 삭제되었다고 한다

이 부분은 업데이트가 장기적으로 이뤄지지 않음에 따라 지속적인 버그가 생겼고, 안드로이드에서 더 이상 표준적으로 지원하지 않게 되었다고 한다

## Retrofit2

Square의 JVM용 유형 안전 HTTP 클라이언트로 OkHttp를 기반으로 빌드된다       
선언적인 client interface를 생성 가능하며 여러 직렬화 라이브러리를 지원한다

Object로 Retrofit 객체를 Singleton으로 구현하여 사용이 가능하므로 HttpUrlConnection 방법에 비해 훨씬 간단했다       

다만 Header를 설정할 때 매번 API에서 설정해주지 않으려면 Interceptor를 사용해야 했다        
Interceptor는 내가 보넀던 request()를 가로채서 추가적인 작업을 거친 후에 다시 proceed()할 수 있도록 해주는 클래스이다       
그래서 retrofit builder 패턴으로 build할 때 header를 추가해주는 interceptor와 데이터를 역직렬화해주는 converter를 추가해줘야 한다       

위의 과정을 거치면 더 이상 추가적으로 retrofit을 설정할 필요는 없으며, interface만 잘 구현하면 잘 연결하여 사용할 수 있다

### 사용 방법
`execute()`함수를 사용하면 동기적으로 처리가 가능하다       
`enqueue()`함수를 사용하면 콜백을 통해 비동기적으로 처리가 가능하다     

<deatils>
<summary> retrofit 사용 예시 </summary>

```kotlin
@OptIn(ExperimentalSerializationApi::class)
object RetrofitClient {
    private const val BASE_URL = BuildConfig.BASE_URL
    private var retrofit: Retrofit? = null

    val json = Json {
        ignoreUnknownKeys = true
        namingStrategy = JsonNamingStrategy.SnakeCase
    }

    val client: Retrofit?
        get() {
            if (retrofit == null) {
                retrofit = Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .client(provideOkHttpClient(AppInterceptor()))
                    .addConverterFactory(json.asConverterFactory(MediaType.parse("application/json")!!))
                    .build()
            }
            return retrofit
        }

    private fun provideOkHttpClient(interceptor: AppInterceptor): OkHttpClient =
        OkHttpClient.Builder().run {
            addInterceptor(interceptor)
            build()
        }

    class AppInterceptor : Interceptor {
        override fun intercept(chain: Interceptor.Chain): Response = with(chain) {
            val newRequest = request().newBuilder()
                .addHeader("Authorization", BuildConfig.GIT_TOKEN)
                .addHeader("X-Github-Api-Version", "2022-11-28")
                .addHeader("Accept", "application/vnd.github+json")
                .build()
            Log.d("VIEWMODEL LOGGING", "new request: ${newRequest.headers()}")
            proceed(newRequest)
        }
    }
}

interface GithubService {
    @GET("/repos/{owner}/{repo}/issues")
    fun getIssues(
        @Path("owner") owner: String,
        @Path("repo") repo: String
    ): Call<List<Issue>>
}

override fun getAllIssuesWithRetrofit(
    owner: String,
    repo: String
): Call<List<Issue>> {
    val githubApi = RetrofitClient.client?.create(GithubService::class.java)
    val result = githubApi!!.getIssues(
        owner = "boostcampwm2025",
        repo = "_issues_repository"
    )
    Log.d("VIEWMODEL LOGGING", "result : ${result.request()}")
    return result
}
```
위의 코드처럼 사용이 가능하다       

github service 인터페이스는 URI와 REST API 메소드를 표기한다        
retrofit object에서는 여러 객체를 생성하지 않도록 retrofit 객체를 싱글톤으로 유지하며, 호출될 때 할당되도록 기존에는 null로 유지한다        
retrofit에 interceptor를 활용해 header를 붙이고, 역직렬화를 위한 converter를 설정한다       

retrofit으로 연결할 때에는 Retrofit 객체를 가져오고 create 함수와 interface를 연결하여 사용할 수 있다

</deatils>

[공식 문서](https://square.github.io/retrofit/)

## Ktor

JetBrains의 HTTP 클라이언트로, Kotlin 전용으로 빌드되었으며 Coroutine 기반이다      
다양한 엔진, 직렬 변환, 플랫폼을 지원한다

[공식 문서](https://ktor.io/)

## Ktor과 Retrofit2의 비교

결국 Retrofit과 Ktor의 성능적인 부분은 큰 차이가 없는 것으로 보인다     

다만 Retrofit은 오래된만큼 관련 자료가 많고, 비교적 안전하다는 장점이 있다      
그리고 Retrofit은 OkHttp 위에 있으므로 OkHttp의 속성 값 변경을 통해 더 세세한 조절이 가능하다       
OkHttp에서는 converter, retrofit, ok http를 사용하면 강력한 Network 기능을 제공할 수 있다고 한다

반면 Ktor의 경우에는 KMM, 즉 멀티 플랫폼 환경을 고려한다면 종속성이 없기 때문에 선택하는 것이 좋다

[Ktor 혹은 Retrofit에 대한 의견들](https://www.reddit.com/r/androiddev/comments/zna320/ktor_or_retrofit/?tl=ko)