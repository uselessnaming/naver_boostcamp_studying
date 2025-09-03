# LiveData vs Flow

[참고 자료1](https://onlyfor-me-blog.tistory.com/557)
[참고 자료2](https://velog.io/@jini_1514/LiveData-vs-flow-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5)

## Flow
단일 값만 반환하는 suspend function과 달리 여러 값을 순차적으로 보낼 수 있는 유형이다       
Flow는 비동기식으로 계산할 수 있는 데이터 스트림의 개념으로, 기본 스레드를 차단하지 않고 다음 값을 생성할 네트워크 요청을 안전하게 만들 수 있다

**생산자** : 스트림에 추가되는 데이터 생산, 코루틴 덕분에 Flow에서 비동기적으로 데이터가 생산될 수 있다     
**중개자** : 스트림에 내보내는 값이나 스트림 자체를 수정할 수 있다      
**소비자** : 스트림의 값을 사용한다

안드로이드에서 Repository는 일반적으로 UI 데이터를 생산하는 생산자 역할이다     
이 때 UI는 최종적으로 데이터를 표시하는 소비자가 된다       
만약 필요하다면, 이 사이에 다음 레이어 요구사항에 맞도록 조정하기 위해서 데이터 스트림을 수정하는 중개자가 있을 수 있다

## LiveData
식별 가능한(관찰 가능한) 데이터 홀더 클래스다       
일반 식별 가능한 클래스와 달리 LiveData는 **생명 주기를 인식**한다      
즉 Activity, Fragment, Service 등 다른 앱 구성 요소의 생명 주기를 고려한다      
그래서 결론적으로는 LiveData는 액티비티 생명 주기 상태에 있는 앱 구성 요소 관찰자만 업데이트 하게 된다  
LiveData는 Observer 클래스로 표현되는 관찰자의 생명 주기가 STARTED 혹은 RESUMED 상태일 때만 관찰자를 활성 상태로 간주하여 업데이트 정보를 알리고, 이 외 상태에는 비활성 상태로하여 정보를 전달하지 않는다

LifeCycleOwner 인터페이스를 구현하는 객체와 페어링된 관찰자를 등록할 수 있다        
이 관계를 사용하면 관잘자에 대응되는 LifeCycle 객체의 상태가 DESTORYED로 변경될 때 관찰자를 삭제할 수 있다

### 장점
+ UI와 데이터 상태 일치 보장: LiveData는 관찰자 패턴을 따르기 때문에 앱 데이터가 바뀔 때마다 관찰자가 대신 UI를 업데이트 한다
+ 메모리 누수 없음: 관찰자는 LifecCycle 객체에 결합되어 있다면 연결된 생명주기가 끝나면 자동으로 삭제된다
+ 중지된 엑티비티로 인한 비정상 종료 없음: Activity가 BackStack에 존재할 때를 비롯해 관찰자의 생명 주기가 비활성 상태에 있다면 관찰자는 데이터를 수신하지 않음

## Flow? LiveData?
LiveData는 생명 주기를 인식해 대상의 생명주기를 따른다      
LiveData를 사용해 이런 생명주기 소유자와 ViewModel 객체 등 수명이 다른 객체 간 통신할 수 있다       
ViewModel은 기본적으로 UI 관련 데이터를 불러오고 관리하므로 LiveData를 보유하는 것이 적합하다       
반대로 UI 레이어는 상태를 보유하는 것이 아니라 데이터를 표시하는 역할을 하기 때문에 LiveData를 갖는것은 적합하지 않다.      
다만 ViewModel이 가지는 LiveData를 관찰하도록 설계해야 한다

### 비동기 지원
LiveData 객체 작업에서는 비동기 데이터 스트림을 처리하도록 설계되지는 않았다        
LiveData 변환, MediatorLiveData를 활용해 LiveData 객체 작업을 진행할 수는 있지만, 데이터 스트림 결합 기능이 매우 제한적이며, 모든 LiveData 객체가 메인 스레드에서 관찰된다      

앱의 다른 레이어에서 데이터 스트림을 사용해야 한다면 Flow를 사용한 다음 asLiveData()를 사용해 ViewModel의 LiveData로 변환하는 게 좋다

### Domain Layer
클린 아키텍처를 살펴보면 Domain Layer가 존재한다        
여기서 Domain Layer는 안드로이드에 의존성을 가지지 않은 순수 자바, 코틀린으로만 구성된다        
이 때 LiveData는 안드로이드 SDK에 포함되기 때문에 DomainLayer에 적합하지 않다

### Flow의 단점
+ Flow는 안드로이드 생명 주기를 알 수 없다
+ 상태가 없어 추적이 어렵다
+ Cold Stream 방식으로, 연속적인 데이터를 처리하는 것보다 Collect 시 생성되고 값이 반환된다
(이런 방법은 여러 번 리소스 요청을 할 수 있는 좋지 않은 방법이다)

이를 보완하기 위해 StateFlow와 SharedFlow가 생성되었다

## 결론
LiveData는 구성 변경 시 안정성을 제공하고 최신 데이터를 View로 전달하는 역할을 하게 된다        
Flow는 Usecase, Repository, DataSource Layer와 긴밀하게 작동해 데이터를 수집 및 처리해서 서로 다른 코루틴 범위에서 작업을 실행한다      

따라서 ViweModel과 View 사이의 상호 작용은 LiveData     
더 깊은 레이어와 Threading 같은 더 복잡한 처리는 Flow가 처리한다