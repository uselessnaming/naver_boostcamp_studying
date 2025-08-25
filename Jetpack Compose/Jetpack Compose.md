# Jetpack Compose
구글에서 개발한 모던한 안드로이드 개발을 위한 선언형 UI Toolkit

Jetpack Compose는 선언형 프로그래밍 패러다임을 따른다       

### XML View system과의 차이는 뭘까?
기존 XML에서 뭐가 문제가 되어 Jetpack Compose를 사용하게 되었을까

1. Java/Kotlin 코드와 높은 의존성
이미 완성된 앱에서 버튼을 하나 빼고 싶다면?     
단순 xml에서 View를 지우는 것만 하지 못하고, 연결된 Java/kotlin 코드를 함께 삭제해야만 한다

2. 화면 요소의 재사용성이 낮음
5개의 입력 칸이 있는 회원 가입 화면을 만들고자 할 때?       
모두 동일해도, 같은 메소드를 가져도, 5개의 EditText가 생성되어야 한다

> BoostClock 프로젝트에서 월~일 Text를 생성할 떄에도 최대한 간략하게 하려고 노력했으나 event 설정 및 View 생성을 별개로 하는 문제가 있었다

3. 코드가 길고 보일러 플레이트가 많음
xml 언어 자체의 특성과 상술한 재사용성 문제로 인해서, 레이아웃이 복잡할수록 코드가 비대해지고 알아보기가 어려워진다

특히 RecyclerView의 경우에, Adapter, ViewHolder 등 여러 개를 연결하여 사용해야 하는데 Jetpack Compose에서는 LazyColumn 혹은 LazyRow를 활용해 간단히 정리할 수 있다

### 명령형 프로그래밍 vs 선언형 프로그래밍
**명령형 프로그래밍**은 기존 View 시스템이 사용하던 방법이다.   
어떤 변수를 참조하는 UI가 있고, 이벤트에 의해 변수가 변경되었고 UI에 반영하고자 한다.   
이 때는 변경된 내용을 갱신하도록 UI 객체에 명령해야 했다

**선언형 프로그래밍**은 화면 전체를 개념적으로 재생성한 후에 필요한 변경사항만 적용하는 방식으로 진행한다.

여기서 핵심은 State, 즉 상태라는 개념이다       
View마다 State가 존재하며, 이 state에 따라 화면을 변경하게 된다     

이벤트에 의해 변수가 변경되면, 이를 참조하고 있는 뷰는 state의 변경에 따라 뷰를 재생성하게 된다

$$
F(STATE) = VIEW
$$

위 공식은 선언형 UI에서 중요하게 다루는 공식이다.       
위젯 혹은 함수의 인자로 상태를 건네주면, 그에 맞는 View를 생성한다는 의미이다.      
함수형 프로그래밍 개념은 순수 함수처럼 같은 상태에 대해서는 항상 같은 View를 반환하게 된다

### 장점
Google에서 설명하는 Compose의 장점은 4가지가 존재한다

+ Less Code: 코드 감소
적은 수의 코드로 더 많은 작업을 하고 전체 버그 클래스를 방지할 수 있으므로 코드가 간단하며 유지 관리가 쉽다

+ Intuitive: 직관적
UI만 설명하면 나머지는 Compose에서 처리하게 된다        
앱 상태가 변경되면 UI가 자동으로 업데이트 된다

+ Accelerate Development: 빠른 개발 과정
기존의 모든 코드와 호환되기에 언제 어디서든 원하는 대로 사용이 가능하다     
Preview 및 Adnroid Studio 지원으로 빠른 개발을 지원

+ Powerful: 강력한 성능
Android Platform API에 직접 액세스하고 Material Design, Dark Theme, Animation 등을 기본적으로 지원