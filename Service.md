# Service
Service는 안드로이드 주요 컴포넌트 중 하나로 사용자와 상호 작용하지 않고 백그라운드에서 오래 실행되는 작업을 수행할 수 있는 애플리케이션의 구성 요소이다       
> 해당 설명을 보고 백그라운드에서 동작하는 스톱워치를 구현할 때 사용해봐야지! 라고 생각했다

## 종류
#### Foreground Service
서비스가 수행하는 동작을 사용자에게 알릴 때 사용하며 Notification을 통해 서비스 실행을 알린다      
활성화 된 Activity와 동일한 우선순위를 가져 메모리 부족에서도 살아남을 확률이 높다고 한다      

ex) 음악 플레이어의 경우 현재 재생 중인 음악 정보를 Notification에 표시

#### Background Service
사용자에게 보이지 않는 Background 작업을 수행하며, 시스템 리소스가 부족하다면 강제 종료될 가능성이 높다     
API 25부터는 앱이 Foreground에 있지 않을 때는 백그라운드 서비스를 강제 종료한다

여기서 foreground service와 background service의 큰 차이는 사용자에게 서비스의 존재를 인식시키는 것과 시스템 리소스 우선 순위다.       
앞에서 동작, 뒤에서 동작이라고 모호하게 생각할 수 있다     
하지만 Foreground Service는 유저가 인식할 수 있는 Service로 실제 Background 작업이어도 사용자에게 알림 등을 통해 사용자가 인식하고 있다면 이는 Foreground Service이다      
그리고 Background Service는 유저가 인식할 수 없고, 화면 혹은 알림으로 유저가 인식할 수 없고, 시스템이 동작하도록 하는 서비스이다

즉 이번 프로젝트 스톱워치 구현에서는 유저가 데이터 변동을 계속 인식해야 하므로 foreground 서비스를 선택했다

#### Bound Service
IBinder라는 이넡페이스로 서버-클라이언트 관계처럼 서비스와 상호작용한다      
여러 프로세스에서 같은 서비스에 바인딩하여 작업 수행이 가능하며, IPC 또한 수행할 수 있다        
Bound Service는 bidning된 컴포넌트들이 전부 해제되면 소멸하게 된다

## 생명 주기
<img width="565" height="662" alt="image" src="https://github.com/user-attachments/assets/fed95587-6987-417d-aa61-8c034600f054" />

#### onCreate()
서비스 첫 생성 시 호출되며, onStartCommand() 혹은 onBind() 전에 호출     
서비스가 실행 중이라면 호출되지 않는다       

#### onStartCommand()
시스템에서 액티비티와 같이 다른 컴포넌트에서 startService()를 호출한다면 이 메소드가 실행되고 서비스가 시작된다        
이 메소드를 구현한 후에 서비스를 중단하기 위해 stopSelf() 혹은 stopService() 메소드를 호출해야 한다     

만약 binding만 할 경우에는 메소드 구현이 필요하지 않다

**return 값 종류**
+ START_NOT_STICKY: 시스템이 서비스를 onStartCommand() 반환 후 중단시키면 서비스를 재생성하지 x -> 불필요하게 서비스가 여러 개 생성되는 것을 방지
+ START_STICKY: 시스템이 onStartCommand() 반환 후에 서비스 중단하면 서비스를 자동으로 다시 생성, 마지막 인텐트를 전달하지 않음
+ START_REDELIVER_INTENT: 시스템이 onStartCommand() 반환 후에 서비스를 중단할 경우, 서비스를 재생성하고 이 서비스에 전달된 마지막 인텐트로 onStartCommand()를 호출하면 모든 보류 인텐트가 차례로 전달

#### onBind()
안드로이드의 구성 요소가 서비스에 바인딩 하고자 하는 경우 호출

해당 메소드의 구현체에서는 IBinder를 return 하여 클라이언트와 서비스가 통신할 수 있는 인터페이스를 제공해야 한다 (null이라도 반환해야 함)

## 트러블
<img width="1513" height="330" alt="image" src="https://github.com/user-attachments/assets/a2526732-56c5-4702-b894-c1e2f11c59ed" />

Service를 생성하고 사용할 때 기본적으로 Manifest에 등록해야 한다     

그리고 특히나 Foreground Service를 사용하는 경우에는 권한이 별도로 필요하다

Foreground Service를 manifest에 추가할 때 기본적으로 foreground service type을 지정해줘야 하는데, 이에 맞는 권한을 허용해줘야 한다.
추가적으로 Foreground Service에 대한 permission 또한 허용해줘야 한다.

위의 설정들을 설정하지 않고 Service, 특히 Foreground Service를 사용하려고 하면 위와 같은 오류가 발생한다.