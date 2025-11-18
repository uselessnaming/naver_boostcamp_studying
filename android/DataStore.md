# DataStore

> [참고 자료](https://developer.android.com/topic/libraries/architecture/datastore?utm_source=chatgpt.com&hl=ko)

## SharedPreference에서의 이전
`DataStore` 이전에는 `SharedPreferences`를 사용했다. `SharedPreferences`는 간단한 key-value로 값을 저장하는 구조를 가지고 있다.       
`SharedPreferences`는 동기 API로 UI 스레드에서 안전하게 호출될 것으로 생각되기 쉽지만, 실제로는 IO에서 동작하게 된다.       
그리고 `apply()`는 내부 구현이 `fsync()`이기 때문에 UI 스레드를 block하게 된다.     
`fsync()`는 앱 내 어떤 서비스가 시작되거나 종료될 때마다 혹은 액티비티가 되거나 종료될 때 실행 대기 중인 호출을 트리거한다. 결국 UI 스레드가 `apply()`로부터 예약된 `fsync()`를 기다리며 멈춰있게 되고 ANR의 가능성이 높아진다.     
따라서 이런 `SharedPreferences`보다는 이 단점들을 보완하여 나온 `DataStore`를 사용하는 것을 권장한다.

> [참고 자료](https://android-developers.googleblog.com/2020/09/prefer-storing-data-with-jetpack.html)

## DataStore 종류
**Preferences DataStore** : `SharedPreferences`를 사용하는 것처럼 단순 key-value를 저장할 때 사용 / Schema 필요 없음
**Proto DataStore** : `Protocol Buffer`를 활용하여 Schema를 지정하고 사용 / Typed Data를 저장할 때 효율적

## DataStore vs Room
부분적 update가 필요할 경우, 참조 무결성이 지켜질 필요가 있는 경우 혹은 복잡하고 큰 데이터 셋을 관리해야 할 때라면 `DataStore`를 사용하는 것보다 `Room`을 사용하는 것을 권장하고 있다.        
`DataStore`는 간단한 데이터를 관리하기 좋으며, 참조 무결성과 부분적 update는 지원하지 않고 있다.

## 동기 DataStore
기본적으로 DataStore는 비동기를 제공한다. 하지만 상황 상 동기적으로 처리해야 할 부분들이 있을 수 있다.

```kotlin
val data = runBlocking { context.dataStore.data.first() }
```
위와 같이 `runBlocking` 블록을 활용하여 동기식으로 값을 가져올 수 있다.     
단 UI 스레드에서 해당 호출 시 ANR 혹은 응답 없는 UI가 존재할 수 있기 때문에 미리 로드해서 해당 문제를 해결해야 한다.        

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    lifecycleScope.launch {
        context.dataStore.data.first()
    }
}
```
이렇게 데이터를 읽고 캐싱하는 것이 완전한 Disk I/O 작업을 막을 수 있기에 권장하는 방법이다.

## DataStore 사용 규칙
1. 같은 프로세스에서 특정 파일의 DataStore를 2개 이상 만들지 않기
2. `DataStore<T>`의 일반 유형은 변경 불가해야 한다.
3. 동일한 파일에서 `SingleProcessDataStore`와 `MultiProcessDataStore` 동시에 사용하지 않기