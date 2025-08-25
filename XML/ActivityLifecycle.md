# Activity Lifecycle
생명주기?       
Activity를 사람이라고 생각하고 태어나 죽기까지의 인생을 표현하는 것       
Activity가 생성되고 삭제되기까지의 전반적인 주기를 말한다     

## 기본적인 Lifecycle
<img width="723" height="817" alt="image" src="https://github.com/user-attachments/assets/eed6e3db-f039-41a6-9405-92858d0c4d3a" />

#### onCreate
액티비티 전체 수명 주기 동안 단 한 번만 실행되는 메소드로 초기화 및 시작 로직을 설정해 둘 수 있다       
예를 들면, 초기 데이터 세팅, 데이터 바인딩 등등..

여기서 setContentView()는 onCreate()에 종속적인 메소드로 반드시 해당 콜백 메소드 안에 구현해야 한다        
이 때 setContentView()함수는 XML 레이아웃 파일을 넘겨주는 것으로 화면에 해당 레이아웃을 그린다      

그리고 onCreate()는 savedInstanceState 매개변수 객체를 반환하며, 활동에 존재하지 않을 경우 Bundle 객체의 값이 null로 전달된다       
```kotlin
var state: String? = null

override fun onCreate(savedInstanceState: Bundle?){
    super.onCreate(savedInstanceState)
    
    state = savedInstanceState?.getString("~~~")
}
```
와 같이 onCreate 내에서 사용할 수 있다      

savedInstanceState는 Bundle 객체로 액티비티 이전 상태 정보를 가지고 있다        
일시 중지, 중단 후에 다시 시작될 때 savedInstanceState를 활용해 이전 상태를 복원할 수 있다       

onCreate()호출 후 액티비티 종료! 는 아니고 액티비티가 Started 상태에 들어가 onStart(), onResume()을 호출한다

#### onStart()
onStart()가 호출되면 앱의 활동이 foreground로 들어가 대화형이 되도록 준비한다        

onStart()는 매우 빨리 실행되며, 액티비티가 RESUMED 상태에 진입함과 동시에 onResume() 메소드를 호출하게 된다       

#### onResume()
foreground에 액티비티가 표시되고, 앱이 사용자와 상호작용할 수 있는 상태가 된다       
어떤 방해 이벤트 혹은 interrupt가 발생하여 focus가 없어지지 않는다면 계속 RESUMED상태에 머무른다        
(전화가 오거나, 홈으로 나가거나 등등..)

여기서 실제 interrupt가 발생한다거나, 방해 이벤트가 발생하게 되면, 액티비티가 PAUSED 상태에 돌입하여 시스템이 onPause() 메소드를 호출한다       

#### onPause()
시스템은 사용자가 떠난다는 첫 번째 신호로 해당 메소드를 호출한다.       
사용자가 잠시 액티비티를 떠났을 때, 즉 다른 액티비티에 포커스를 뒀을 때 호출되는 콜백이다.        
= 해당 액티비티가 foreground에 있지 않음

멀티 윈도우 혹은 스플릿 등 한 화면에 두 개 이상의 앱을 두는 경우 포커스를 옮기는 경우에도 onPause가 호출됨       

onPause()의 경우 onStart()처럼 잠깐 실행되기 때문에 데이터를 저장하는 작업을 하기에는 시간이 부족할 수 있다       
따라서 onPause()에서는 사용자 데이터 저장, 네트워크 호출, DB 트랜잭션 등을 실행해서는 안된다      
부하가 있는 작업의 경우 onPause()보다는 onStop()에서 수행해야 한다

#### onStop()
액티비티가 사용자에게 더 이상 표시되지 않으면 STOPPED 상태에 진입하고 시스템이 onStop() 콜백 메소들르 호출     
새로 시작된 액티비티가 화면 전체를 차지할 경우 혹은 액티비티 실행이 완료되어 종료될 시점에 onStop()을 호출 할 수 있음     

이 메소드에서는 필요하지 않은 리소스 해제를 하거나 조정해야 함
ex) DB에 데이터 저장, UI 이벤트 중지 등등

만약 사용자가 다시 액티비티로 돌아오게 된다면 STOPPED 상태에서 다시 시작되어 onRestart() >> onStart() >> onResume() 이 연달아 호출되며 RESUMED 상태로 변화하여 다시 foreground 액티비티로 전환된다      

#### onDestroy()
액티비티가 완전히 소멸되기 전 이 콜백 메소드가 호출됨      
1. finish()가 호출되거나 사용자가 앱을 종료해 액티비티가 종료되는 경우
2. 화면 구성이 변경되어 일시적으로 액티비티를 소멸시키는 경우

만약 해당 메소드 호출까지 해제되지 않은 리소스가 있다면 여기서 모두 해제해야 함       
하지 않는다면 메모리 누수의 위험이 존재한다        

#### Configuration Change
안드로이드에서는 Configuration Change라는 것이 존재한다.        
Configuration Change란 시스템의 구성이 변경되는 것을 말한다.

이 때 Configuration Change가 발생하면 생명 주기는 onDestory()를 호출하고, 그 이후로 onCreate()로 새로운 인스턴스를 생성한다       
위에서 화면을 회전하거나, 다크모드 >> 라이트모드로 모드 전환을 할 경우가 해당되며 이는 onDestory까지 호출되어 인스턴스를 삭제하고 onCreate로 새로운 인스턴스를 생성한다.

**그렇다면 Configuration Change가 발생한다면 내부에 존재하는 데이터는 어떻게 유지될까?**        
이 때 savedInstanceState를 활용하여 보존해야 할 데이터를 보존한다       
그래서 onCreate를 확인해보면 savedInstanceState를 파라미터로 받아서 사용하는 것을 볼 수 있다