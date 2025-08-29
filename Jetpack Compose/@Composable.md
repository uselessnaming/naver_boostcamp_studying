# @Composable
Jetpack Compose로 사용자 인터페이스를 생성하기 위한 특수 코틀린 함수

@Composable 을 이용해 선언하며 Composable이 호출되면 앱 내에서 해당 영역이 이미지로 변환될 때 사용자에게 동작되는 방식을 정의하는 데이터와 프로퍼티의 집합을 전달

코틀린의 일반 함수와 같은 방식으로 결과값은 반환하지 않고, 런타임으로 UI 요소들을 전달하면 컴포즈 런타임을 통해 렌더링이 진행된다

> 일반적으로 composable의 경우에는 Pascal Case를 따른다

**Q 그럼 예외 Case에는 어떤 것들이 있을까?**

### Composable 헬퍼 함수
흔히 접하는 rememberScrollState(), rememberCoroutine()같은 함수들은 camel case를 따른다

상태나 Utility를 반환하는 역할로 UI가 아닌 state/logic provider 같은 경우 camel case를 적용한다

### Modifier의 확장 함수
Modifier로 제약 설정 혹은 세세한 속성 설정을 하는 경우
ex) fillMaxSize(), clickable(), padding() 등등

속성을 체이닝하는 함수이므로 camel case를 적용한다

### 안드로이드 Compose 내부 low-level API
drawBehind, onGloballyPositioned와 같이 UI를 직접 그리는, 혹은 위치를 계산하는 등, 개발자가 직접적으로 사용하지 않고 노출되는 API를 통해 처리하는 작업들의 경우
내부적으로 camel case가 적용되어 동작하고 있다

## 결론
직접 UI와 관련된 함수들은 모두 pascal case를 적용한다!      
그리고 State를 관리하거나 utility 성격의 composable, Modifier 확장과 같은 경우에서는 camel case를 적용한다