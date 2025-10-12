# Thread 종류
## MainThread
애플리케이션이 Android에서 시작되면 `MAIN Thread`라고 하는 첫 번째 실행 스레드가 생성된다       
적절한 사용자 인터페이스 위젯에 이벤트를 전달하고 Android UI Toolkit의 구성 요소와 통신하는 역할을 한다     
애플리케이션의 응답성을 유지하려면 `MAIN Thread`를 차단할 수 있는 작업을 수행하지 않도록 주의해야 함        
> 네트워크 작업 및 데이터베이스 작업        
만약 메인 스레드에서 호출되게 된다면, 동기적으로 호출되게 되는데 이 때 작업이 완료될 때까지 UI가 완전히 응답하지 않는 상태로 유지된다       
따라서 별도의 Thread에서 수행하는 것으로 `UI Thread` 차단을 방지한다

`MAIN Thread`에서 UI 관련된 작업들도 처리한다       
> `MAIN Thread`에서는 UI 작업을 처리할 수 없도록 되어있다       

안드로이드 컴포넌트의 생명주기 메서드와 그 내부 메서드 호출은 모두 `MAIN Thread`에서 처리한다

Activity 외에도 BroadcastReceiver, Service, Application에서 UI와 관련 없더라도 `MAIN Thread`에서 작업 처리된다

## UI Thread
애플리케이션의 기본 실행 스레드     
여기에서 대부분의 애플리케이션 코드가 실행된다      
모든 Application 구성 요소(Activity, Service, ContentProvider, BroadcastReceiver)는 이 스레드에서 생성되고 해당 구성 요소에 대한 모든 시스템 호출은 해당 Thread에서 수행됨

백그라운드 작업을 수행하게 된다면 이 코드는 UI Thread에서 실행되지 않고, 만약 백그라운드 스레드에서 수행해야 하는 경우에 `runOnUiThread`를 사용하여 변경한다

## Worker Thread
백그라운드 스레드       
UI Thread 외에 별도로 생성되는 Thread들     

## Any Thread
해당 주석이 달린 메소드는 모든 Thread 내에서 호출이 가능함을 말함

## Binder Thread
Binder Thread는 프로세스 간 통신을 위해 Binder 시스템에서 사용되는 Thread