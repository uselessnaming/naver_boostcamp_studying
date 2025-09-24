# LiveData
관찰 가능한 데이터 홀더 클래스로 **수명 주기를 인식**한다       

Observer 클래스로 표현되는 관찰자의 수명 주기가 `STARTED` 혹은 `RESUMED` 상태면 LiveData는 관찰자를 활성 상태로 간주한다. LiveData에 등록된 Observer들 중 비활성 관찰자는 변경 사항에 관한 알림을 받지 않습니다.

LifecyclerOwner 인터페이스를 구현하는 객체와 페어링된 관찰자를 등록할 수 있다. Activity나 Fragment의 경우, 이를 활용하면 LiveData 객체를 안전하게 관찰할 수 있고, 수명 주기가 끝나면 삭제되어 메모리 누수 걱정을 하지 않아도 된다      

> 기본적으로 LiveData는 Java의 라이브러리이다.        

[공식 자료](https://developer.android.com/topic/libraries/architecture/livedata?hl=ko)

## 장점
LiveData의 장점

### UI와 데이터 상태의 일치 보장
LiveData는 Observer 패턴을 따르기 떄문에, LiveData에서 데이터 갱신이 일어나면 이를 Observer가 확인하고 처리해줍니다.        
따라서 Event 생성 시마다 UI 업데이트를 할 필요가 없어졌습니다.

LiveData의 `mObservers`를 확인하면 observer와 wrapper에 대한 정보를 가지고 있습니다. ObserverWrappter의 내부에는 activeStateChanged 함수가 존재합니다. Lifecycle Owner의 lifecycle에 따라 active 여부를 결정하고, active 상태라면 `distpachingValue()` 함수를 호출합니다. 이 함수를 통해서 활성 상태에 있는 observer들에게 데이터 값의 변화를 알립니다. 
따라서 처음에 observer와 lifecycle owner를 등록해주면 LiveData는 수동 제어 없이 자동적으로 lifecycle을 감지하고 그에 맞게 동작하게 됩니다.

### 수명 주기를 더 이상 수동으로 처리하지 않음
UI 구성 요소는 Observer를 제어하지 않는다.      
다만 Observer를 통해 변화된 데이터에 따라 처리해 줄 뿐이다.     
LiveData는 관찰하는 동안 수명 주기 상태의 변경을 인식하기 떄문에 이를 자동으로 관리해준다

### 메모리 누수 없음
관찰자는 Lifecycle 객체에 결합되어 수명 주기가 끝난다면 자동으로 삭제된다

LiveData의 `LifecycleBoundObserver` 클래스의 `onStateChanged()` 함수를 확인해보면 lifecycle이 DESTORYED 상태라면 observer를 제거하고 있습니다. 이를 통해 Observer가 Lifecycle 객체에 결합되어 lifecycle이 종료될 경우 같이 제거되는 것을 알 수 있습니다.

### Stopped 상태의 Activity에서도 비정상 종료 없음
Activity가 백 스택에 있는 것을 포함해 관찰자에 결합된 수명 주기가 비활성 상태에 있다면 관찰자는 어떤 LiveData도 받지 않는다

### 최신 데이터 유지
관찰자가 비활성 상태에서 활성 상태가 될 때 최신 데이터를 수집한다       

### 적절한 Configuration Change 대응
Configuration Change를 통해 Activity나 Fragment가 재생성되면 사용 가능한 최신 데이터를 즉시 받게 된다

### Resource 공유
싱글톤 패턴을 사용하는 LiveData 객체를 확장해 시스템 서비스를 wrapping할 수 있다. LiveData 객체가 시스템 서비스에 한 번 연결되면 resource가 필요한 모든 관찰자가 LiveData 객체를 볼 수 있다

> 이 부분은 직접 해보지 않아서 아직 잘 모르겠다

## 앱 아키텍처의 LiveData
Activity나 Fragment에서는 상태 보유가 아닌 데이터를 표시하는 역할을 하며, LiveData 인스턴스를 보유는 하면 안됩니다.     
> 데이터와 UI가 분리되어 단위 테스트의 설계가 쉬워진다

Data Layer에서 LiveData 객체 작업은 하면 안된다. LiveData는 비동기 데이터 스트림을 처리하도록 설계되어있지 않기 때문이다        
LiveData 변환과 MediatorLiveData를 사용해 LiveData 객체 작업을 할 수 있긴 하지만 데이터 스트림을 결합하는 기능이 매우 제한적이며, 이를 통해 만들어진 객체는 모두 Main Thread에서만 관찰된다.        
따라서 위의 동작을 해야 한다면 Kotlin Flow를 활용해 데이터 스트림을 관리하고 이후에 asLiveData() 함수를 통해서 ViewModel의 LiveData로 변환하는 것이 좋다        
만약 Java라면 RxJava와 Executor를 활용하는 것이 좋다고 한다

## `Transformations`클래스
관찰자에게 LiveData를 전달하기 전, 객체에 저장된 값을 변경하고 싶거나 다른 LiveData 인스턴스를 반환해야 하는 경우가 있다.       
이를 위해 존재하는 클래스이다.      

### `Transformations.map()`
LiveData 객체에 저장된 값에 함수를 적용하여 결과를 다운스트림으로 전파한다

### `Transformations.switchMap()`
map()과 같이 LiveData 객체로 저장된 값에 함수를 적용하고 결과를 래핑 해제한 후에 다운 스트림으로 전달한다

## 여러 LiveData 소스 병합
`MediatorLiveData`는 `LiveData`의 서브 클래스로, 해당 클래스를 사용하여 여러 LiveData 소스를 병합할 수 있다

`MediatorLiveData`는 원본 LiveData 소스 객체가 변경될 때마다 트리거된다     

## LiveData와 Coroutine
`liveData` 블록은 Coroutine과 LiveData 간 구조화된 동시 실행 프리미티브 역할을 한다
> 구조화된 동시 실행 프리미티브?        
> 각각의 Coroutine을 임의 생성하여 사용하면 메모리 누수의 확률이 올라가고 개별적 관리가 어려움      
> 이를 위해 Coroutine을 어떤 Lifecycle에 연결되어, 해당 생명 주기가 종료되면 자동으로 정리되도록 하는 방식임

코드 블록은 LiveData가 활성화되면 실행을 시작하고, LiveData가 비활성화되면 구성 가능한 제한 시간 후 자동 취소된다       
코드 블록이 완료 전에 취소된다면 LiveData가 다시 활성화되었을 경우 다시 시작된다        
이전 실행에서 성공적으로 완료되었다면 다시 시작되지 않는다      
자동으로 취소되었을 때만 재시작한다     
블록이 Exception과 같은 것으로 취소되었다면 재시작되지 않는다       

emit을 통해서 값을 내보낼 수 있다       
다만 emit() 호출은 LiveData 값이 기본 스레드에서 설정될 때까지 블록의 실행을 정지한다

또한 emit 혹은 emitSource 함수를 통해서 이전에 추가했던 소스를 삭제하고 덮어쓸 수 있다

## LiveData에서 값을 설정하는 방식
LiveData에서 데이터를 추가하기 위해서는 `setValue()`와 `postValue()`가 있다

### `postValue()`
**Background thread**에서 값을 변경한다.       
background thread에서 동작하다가 Main thread에 값을 Post하는 방식이다

내부적으로는 `Handler(Looper.mainLooper().post(() -> setValue()))` 코드가 실행된다

Main Thread에 적용되기 전 postValue()가 여러 번 호출된다면 가장 최신 값이 적용된다.     
다만 postValue() 이후에 getValue()를 바로 하게 되면 Handler를 통해서 Main Thread에 값이 전달되는 시간이 필요하기 때문에 값을 제대로 가져오지 못한다.

### `setValue()`
Main Thread에서 LiveData의 값을 변경한다        
MainThread에서 바로 값 변경을 진행하기 때문에 setValue() 호출 뒤에 바로 getValue()로 변경 값을 가져올 수 있다       
중요한 것은 setValue()는 MainThread 값을 dispatch하기 때문에 백그라운드에서 setValue()를 호출한다면 오류가 발생한다     

### 결론
즉 LiveData 값을 즉시 변경해야 하는 상황이라면 setValue를 쓰되 Main Thread에서 호출해야 하며, background thread에서 호출하지 않게 조심해야 한다     
또한 백그라운드에서 진행한 작업의 value를 적용시키기 위해서라면 보통 suspend이기 때문에 postValue를 통해 값을 변경해야 한다