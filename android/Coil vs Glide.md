# 이미지 로드 라이브러리
대표적으로 이미지 로드 라이브러리에는 Coil과 Glide 두 라이브러리를 비교한다

## Coil
`AsyncImage`를 호출하여 Coil에 이미지를 로드하는 Composable을 활용한다.     
`AsyncImage` 내부 코드에서는 `rememberAsyncImagePainter`를 호출하고, 이는 `AsyncImagePainter`를 반환한다.       
`AsyncImagePainter`는 파라미터로 `ImageRequest`와 `ImageLoader`를 갖는다. 내부에는 state, painter 등의 값들을 보관하고 관리한다. State는 Empty, Loading, Success, Error로 구분되며 이는 Image 가져온 결과에 대한 상태를 나타내는 것이다. 또한 `AsyncImagePainter`는 `Painter` class와 `RememberObserver` interface를 상속받아 구현하고 있다. RememberObserver를 상속하고 있으므로 `onRemembered()` 함수를 `override`하여 구현하고 있다. 여기서 만약 rememberScope 값이 있다면 구현하지 않고 `return`하게 된다. 없을 경우 새로 생성하며 `snapshowFlow`를 활용하여 request의 변경에 따라 `imageLoader`의 `execute()`함수를 호출한다. 

`ImageLoader`는 실제로 데이터를 불러오기 위한 interface이다. 이를 구현하는 클래스는 `RealImageLoader` 하나만이 존재한다. 여기서 `execute()` 함수를 확인할 수 있다.

```kotlin
override suspend fun execute(request: ImageRequest): ImageResult {
    if (request.target is ViewTarget<*>) {
        // We don't use the async call that returns the job for Compose to micro-optimize the performance.
        // The job is only needed in case of the Views implementation.
        return coroutineScope {
            // Start executing the request on the main thread.
            val job = async(Dispatchers.Main.immediate) {
                executeMain(request, REQUEST_TYPE_EXECUTE)
            }

            // Update the current request attached to the view and await the result.
            request.target.view.requestManager.getDisposable(job)
            job.await()
        }
    } else {
        // Start executing the request on the main thread.
        return withContext(Dispatchers.Main.immediate) {
            executeMain(request, REQUEST_TYPE_EXECUTE)
        }
    }
}
```
?? 왜 구분?

코드를 확인해보면 request의 target이 `ViewTarget` 타입이라면 job을 생성하여 `executeMain()` 값을 return 하도록 한다. 그 외에 context를 교체해도 `executeMain()` 값을 return 하도록 한다. `requestDelegate.start()`를 통해서 lifecycle에 이벤트를 등록한다. 만약 `ENQUEUE` 요청이라면 lifecycle이 `ACTIVE`상태가 될 때까지 대기한다. 그 이후 `placeHolder`, 이미지의 목표 크기를 정한다. 그리고 난 후에 `Interceptor.Chain` interface를 상속받은 `RealInterceptorChain`클래스의 `proceed()` 함수를 호출해서 이미지를 불러온다.

**Memory Cache**        
Coil은 내부적으로 이미지를 캐싱하고 기억하고 있어 데이터를 다시 불러올 때 빠르게 로드할 수 있다. `ImageLoader` interface를 보면 `MemoryCache`와 `DiskCache`를 지니도록 되어 있다. Cache 단계는 `Network` -> `DiskCache` -> `WeakMemoryCache` -> `StrongMemoryCache` 순서로 이뤄진다. 이미지 데이터를 `StrongMemoryCache`에 hit 했을 때, 존재한다면 이를 바로 가져가 사용한다. 이 경우에는 네트워크나 디코딩 작업 등 작업들을 넘어가기 때문에 빠르게 진행된다. 만약 `StrongMemoryCache`에 존재하지 않아서 실패했다면, 다음으로는 `WeakMemoryCache`에 hit한다. 여기서는 `WeakMemoryCache`에서 저장된 데이터를 그대로 불러와 `StrongMemoryCache`로 가져오고 사용한다. 만약 `WeakMemoryCache`에도 존재하지 않는다면 `DiskCache`에 접근한다. `DiskCache`에 hit하면 파일 시스템으로 저장된 `ByteArray` 데이터를 불러와 디코딩하여 파일을 `StrongMemoryCache`에 저장하게 된다. 만약 `DiskCache`에도 miss한다면 Network 요청을 통해서 이미지를 받아오게 된다.

`MemoryCache` 인터페이스를 구현하는 `RealMemoryCache` 클래스가 파라미터로 `StrongMemoryCache`와 `WeakMemoryCache`를 보유하고 있다. 여기서 `StrongMemoryCache`를 보면 `trimMemory()`에서 메모리가 일정 수준보다 부족하게 된다면 `clearMemory()`를 호출해서 참조로 호출되고 있는 부분을 끊는다. 그리고 이보다는 조금 덜 부족하다면 `tirmToSize()`함수를 통해서 반으로 줄이게 된다. 이렇게 데이터를 사용하는 부분이 종료되어 `StrongMemoryCache`에서 참조가 끊기게 되면 이는 `WeakMemoryCache`로 넘어가게 된다. 그리고 여기서 메모리가 부족해지면 Garbage Collector가 `WeakMemoryCache`에 있는 데이터를 정리한다. `StrongMemoryCache`는 GC에 의해 사라지지 않지만, `WeakMemoryCache`는 GC에 의해 사라질 수 있다.       
`StrongMemoryCache`는 `LRUCache`를 cache로 구현하고 있다. 즉 최근에 사용한 값들을 유지하고, 꽉 찬 상태로 새로운 값이 들어온다면 가장 오래 사용하지 않은 값을 삭제시키는 것이다. 그에 비해서 `WeakMemoryCache`는 `LinkedHashMap`을 사용하여 cache를 구현하고 있기 떄문에 순차적으로 사라지게 된다.

`DiskCache`는 이보다는 조금 느리지만 더 큰 용량을 저장할 수 있다. 이 이유는 내부적으로 `Filesystem`을 사용하기 때문이다. 데이터를 `Filesystem`을 통해 저장하고 있다. 그래서 GC는 이 정보 또한 삭제할 수 없다. 파일 시스템으로 실제 저장되어 있기 때문에 메모리를 정리하는 것과는 연관이 없다. 그리고 요청이 들어왔을 때 해당 파일에서 `ByteArray`로 저장된 값을 읽어오고, 이를 기반으로 `Bitmap`으로 디코딩하여 사용한다. `MemoryCache`보다는 느리지만 `Network` 요청보다는 훨씬 속도가 빠르다.

**첫 이미지 로드**      
첫 이미지를 로드할 때 속도에 조금 향상시킬만한 요소가 있다. 이는 `onRemembered()` 함수에서 scope 변수 선언된 부분을 확인해보면 `CoroutineScope`의 Dispatcher가 `Dispatchers.Main.immediate`로 호출되어 있다. 이는 `Dispatchers.Main`과 달리 queue에 대기하지 않고, 바로 등록되어 즉각적으로 처리될 수 있도록 한다. 따라서 로드될 때 시간을 더 빠르게 처리할 수 있다.

