# Navigation 2
화면 이동과 관련된 액션을 쉽게 구현하도록 도와주는 Jetpack 라이브러리 중 하나

XML에서는 Fragment 간 전환 코드를 작성 시 FragmentManager를 활용할 때가 많았으나, Fragment 백스택 관리, 전달 파라미터 등 많은 요소들을 코드로 직접 구현함에 있어 컴포넌트의 코드 양이 늘어나고 난잡해지는 경우가 많았다

여기서 Jetpack Navigation이 등장하였고, 화면 이동, 스택 관리, 데이터 전송까지 손쉽게 구현할 수 있게 되었다

[참고 자료1](https://algosketch.tistory.com/190)
[참고 자료2](https://medium.com/hongbeomi-dev/compose-navigation-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-8f9c63426eca)

## 장점
+ Fragment 트랜잭션 처리
+ 손쉬운 화면 이동 처리
+ 손쉬운 애니메이션 구현 지원
+ 딥 링크 구현 및 처리 지원
+ Safe args와 함께 안전한 데이터 전달 지원
+ Bottom Navigation, Navigation Drawer와 같은 요소들을 손쉽게 구현할 수 있도록 지원
+ view model을 활용하여 그래프 내 컴포넌트 간 UI 관련 데이터를 공유할 수 있도록 지원
+ 안드로이드 스튜디오의 Navigation Editor를 통해 네비게이션 그래프를 한 눈에 보고 쉽게 편집할 수 있도록 지원

## 구성
### Graph
목적지를 배열 형태로 관리하여, 앱 내의 모든 탐색 목적지와 목적지가 서로 연결되는 방식을 정의하는 데이터 구조 (NavGraph 클래스로 구현)

자식 노드를 가질 수 있으나 다른 NavDestination의 경우에는 자식 노드를 가질 수 없다

그리고 NavGraph는 화면에 출력될 수 없기에 startDestination을 필요로 하며, NavGraph로 navigate되면 즉시 startDestination으로 navigate된다

composable과 dialog는 빌더의 확장 함수로 NavDestination을 만든다        
NavHost 및 navigation의 내부에서는 빌더에서 생성한 NavDestination을 addDestination() 메소드를 호출해 부모 노드의 자식 노드로 추가한다     

### Host
목적지를 포함하고 있는 UI 요소  
Compose에서는 NavHost 클래스가 이를 관장하고 있다

composable 함수나 확장 함수를 통해 쉽게 트리를 구성할 수 있다

NavGraph는 NavHost 혹은 navigation 확장 함수를 통해 생성되며 ComposeNaviagtor.Destination은 composable 확장 함수로 만들어진다

이 때 그래프의 root는 NavGraph가 되고, NavGraph는 자식 노드를 가지게 된다

navigation같은 부분은 중첩 그래프를 표현할 때 사용되며, 추가로 NavHost는 NavController에 완성된 그래프를 등록하는 역할도 담당하며, route 정보가 필요하지 않다

생성된 NavGraph를 제공받은 NavController의 graph에 등록

### Controller
NavHost 내 앱 목적지 간의 탐색, 백스택, 딥링크 처리를 관리한다      
Compose에선 NavController 클래스를 상속받는 NavHostController로 구현되어 있다

전체 Navigator에 대한 백스택을 종합해 전체 백스택을 관리한다

navigate 함수를 구현하고 있으며, 내부적으로 적절한 Navigator를 가져와 Navigator의 navigate를 처리한다

rememberNavController 호출 시 createNavController 함수를 통해 NavController를 생성한다      
기본적으로 3종류의 Navigator를 추가하며, 커스텀 Navigator도 추가할 수 있는 구조다

### Destination
nav graph의 노드        
host가 해당 노드의 콘텐츠를 표시한다        
NavDestination 클래스로 구현

![alt text](image-4.png)

navigation 그래프의 각 노드를 구성하는 클래스이며, ComposeNavigator.Destination, NavGraph, DeialogNavigator.Destination 세가지 하위 클래스가 기본적으로 제공된다

위 사진은 예시 트리로 각 노드들은 NavDestination을 상속받는 클래스의 객체이다

### Route
목적지와 목적지에 필요한 데이터를 고유하게 식별하며, Route를 통해 탐색이 가능하다

route를 설정하는 부분은 단순히 route 값을 설정하는 setter가 아닌 커스텀 setter를 호출하게 된다      
내부적으로 createRoute 메소들르 통해 scheme을 추가해 deepLink 형태로 생성하고, deepLinks 목록에 추가한다

이를 통해 navigate의 파라미터로 route를 넘기는 것과 deepLink를 넘기는 것을 동일하게 처리할 수 있다      
즉 route와 deepLink는 본질적으로 거의 동일하다

## 알아둬야 할 클래스
### NavBackStackEntry
navigate 될 때마다 백스택에 추가된다        
NavDestination과 전달받은 arguments 정보를 가지고 있다      
NavDestination에서 화면에 출력해야 할 content를 가져온다

### Navigator
navigate를 담당하는 클래스로, 한 가지 NavDestination에 대한 백스택을 관리하고 있다      
Navigator의 타입 파라미터로 NavDestination을 받는다     
앞서 NavDestination의 종류 세 가지가 있다고 얘기했었는데, Navigator도 크게 3가지 구현체가 존재한다

navigation, composable, dialog로 만든 노드로 navigate 할 때의 로직을 구현한다

## 동작 원리
동작이 매우매우매우매우 복잡하니 핵심 로직을 보자

backStack은 Navigator에서 StateFlow로 관리되며, NavController에서 구독하고 있다

NavHost는 backStack 최상단에 있는 Entry를 화면에 출력하며, navigate()가 동작되면 Entry를 추가한다       
navigate() 메소드는 route로 호출하는 것과 deeplink로 호출하는 방법이 있다       
하지만 route를 통해 호출해도 deeplink 형태로 변환하는 과정이 있기 때문에 두 메소드는 동일하게 처리된다      
내부적으로는 navigation graph 순회하면서 uri 형태가 일치하는 destination을 찾게 된다

navigate의 인자로 넘겨준 route 혹은 deepLink는 request의 uri라는 프로퍼티 형태로 존재한다

순회 과정에 uri를 파싱하고, arguments를 찾는 과정이 포함되어 있다. matchDeepLink()의 반환 값인 DeepLinkMatch의 matchingArgs가 uri에 포함된 arguments이다

이렇게 destination과 arguments를 활용해 NavBackStackEntry를 생성해 backStack에 추가하는 구조다

destination 종류마다 처리할 처리할 수 있는 Navigator가 다르다       
이는 navigatorProvider에 저장되어 있어, 적절한 Navigator를 찾아 navigate() 메소드를 호출한다

composable 함수를 통해 생성된 destination은 ComposeNavigator가 처리하고, navigation 함수를 통해 생성된 destination은 NavGraphNavigator가 처리한다       
NavGraph로 navigate할 경우 곧바로 startDestination으로 navigate하는 메소드를 호출한다       
이 때 startDestination도 NavGraph일 수 있기에 재귀적으로 처리된다

NavController.kt 파일 내부에 addEntryToBackStack()함수가 있고, NavController의 백스택에 entry를 추가하는 코드다.        
이 로직은 람다 형태로 아래 메소드에 전달하게 된다       

addEntryToBackStack()는 NavController의 백스택에 navigate 대상 entry를 추가하는 함수다      
이 entry는 최종 목적지이기 때문에 중간 목적지가 NavGraph인 경우, node와 람다의 파라미터로 들어온 entry와 destination이 다를 수 있다     
따라서 최종 목적지의 조상들을 먼저 백스택에 추가하고, 마지막으로 entry를 추가한다

<img width="845" height="463" alt="image" src="https://github.com/user-attachments/assets/3bfbcbea-1e1d-4e3b-91c8-a28209079e34" />

말로만 하니 어렵지만 위 구조를 보면 좀 더 이해가 쉽다    
A화면에서 B화면으로 이동하고자 한다면?에 대한 설명이다    

각 화면들은 NavDestination에 매칭된다    
그리고 NavDestination은 화면 전달을 요청하기 위한 NavDirection을 가지고 있다    

A화면에서 B화면 이동을 위해 NavDirection을 NavController에게 요청하면 내부적으로 stack을 처리하고, 다음 NavDirections를 실행할 때 arguments가 있다면 전달한다

### Navigator 구조
Navigator는 state라는 이름으로 backStack을 관리하며, NaviatorState는 backStack이라는 프로퍼티이다

NavigatorState는 추상 클래스이며, push라는 메소드를 가지고 있다     
이는 backStack에 push하는 역할을 하고 있다

NavController는 이 추상 클래스를 inner class 형태로 상속받는다.     
여기서 push 메소드를 오버라이드하며, 외부 클래스인 NavController의 handler를 실행하게 된다      
이 핸들러는 navigateInternal()에서 초기화된 핸들러이다      

따라서 push 메소드는 navigator의 backStack에 push도 하지만, NavController의 백스택에도 push한다

NavController의 백스택은 backQueue라는 이름으로 존재하며 이는 Deque를 활용한다

navigate 대상이 NavGraph일 경우 Navigator 내에서 재귀적으로 navigate()가 호출되는데, NavController에서는 이를 감지할 방법이 필요하다        
그 방법으로 NavigatorState를 NavController에서 구현해 사용한 것으로 보인다

> Navigator의 backStack을 NavController가 구독하는 형태라고 했는데, 왜 2개에 모두 push할까
**NavController**는 앱의 전역적인 Navgation 상태를 관리하며, 자체적으로 NavBackStackEntry 객체들의 리스트인 BackStack을 갖고 있다        
**Navigator**는 실제 화면 단위에 대한 구체적 네비게이션 로직을 담당한다고 한다      
자체적으로 NavigatorBackStack을 유지하며, push/pop 로직을 수행한다
그리고 Navigator의 경우에는 NavController에 자신이 발생시킨 변화를 알려준다

여기서 NavController와 Navigator는 각각 BackStack을 갖고 있고, 서로 독립적이지만 항상 동기화가 되어야 한다

Navigator의 BackStack은 실제 화면 표시/제거를 위한 로컬 스택이다. 그리고 Fragment 트랜잭션 혹은 Compose Navigation 상태와 직접 연결을 담당한다

NavController의 BackStack은 앱 전역 Navigation 상태를 관리하고, 여러 Navigator을 관리 시 통합된 BackStack이 필요하다.

즉, Navigator 혼자 제거하게 되면 화면은 사라지지만 NavController는 상태를 모르게 되어 앱 전체 Navigation에 혼란이 발생한다.     
반대로 NavController에서만 제거하게 되면 화면 상태와 UI가 불일치하게 되어 사용자 입자에서는 화면이 그대로 남아있는 오류가 생긴다.

### pop back stack 동작 원리
NavHost에서는 내부적으로 BackHandler를 구현하고 있다        
ComposeNavigator의 backStack의 사이즈가 2 이상일 때만 동작한다

popBackStackInternal() 메소드는 목표 destination까지 백스택에서 제거한다        
여기서 목표 destination은 보통 composable이다.
composable을 제거하면 백스택의 최상단에 NavGraph가 남을 수 있는데, NavGraph는 content를 갖지 않기에 백스택의 최상단에 존재해도 보여줄 수 있는 화면이 없다       
따라서 자식 entry가 모두 제거되었다면, dispatchOnDestinationChanged() 메서드에서 NavGraph를 제거하게 된다

backStack에서 pop하는 로직 역시 push할 떄와 마찬가지로 Navigator의 backStack을 감지하여 backQueue에서도 pop해준다
