# Composable 수명 주기
Compose에서 composable은 UI의 상태와 변경 사항을 효율적으로 관리하기 위해서 특정한 수명 주기를 따른다

초기 Composition 시, Compose는 처음으로 composable을 실행하고, UI를 구성하기 위해 호출된 composable을 추적한다

앱 상태 변경 시 Recomposition을 예약하며 상태 변경 사항을 반영하고 UI 업데이트를 하게 된다

### Composition
+ composable 함수가 처음 호출되고 UI트리를 구성할 때 발생
+ composable 함수는 composition 과정에서 호출되어 UI 요소를 생성

### Recomposition
+ UI 상태 변경 시 composable 함수가 다시 호출되어 UI를 업데이트 하는 과정
+ State가 변경되면 Compose는 관련된 composable 함수들을 다시 호출하여 UI를 업데이트 한다

### Decomposition
+ composable 함수가 UI 트리에서 제거될 때 발생
+ composable이 더 이상 필요하지 않는다면 메모리에서 제거

## Smart Recomposition
Compose에서는 상태가 변경되면 변경된 부분만 다시 렌더링하여 UI를 업데이트한다       
이 과정에서 Skippable, Restartable 등의 개념을 통해 Smart Recomposition을 수행한다
> 이 과정을 통해 불필요한 Recomposition을 최소화할 수 있고, 성능을 최적화할 수 있다

### Compose의 안전성
Compose는 컴파일 시점에서 객체들의 Stability(안정성)을 확인한다     
어떤 composable의 함수의 모든 인자가 Stable이라면, 그 composable 함수는 Skippable(생략 가능)하다        
이는 Recomposition이 발생했을 때, 모든 함수 인자가 Stable이고, 변하지 않았다면 Recomposition을 생략한다     

restartable의 경우는 composable 함수의 인자가 변경되거나 상태가 변경되면 해당 함수는 다시 실행되어야 함을 의미한다

**stable**      
기본적인 Primitive 타입과 이를 활용한 함수 타입 등이 해당된다       
그리고 각 인자들이 val로 이뤄져있어야 한다

### Stable Marker
Compose에서는 unstable한 타입을 안정적으로 사용할 수 있도록 하는 기능이 존재하며, 이를 Stable Marker라고 한다       
여기에는 @Stable과 @Immutable이 존재한다

하지만 컴파일러에게 약속하는 기능이며, 실제 검사를 수행하지 않기 때문에 예상치 못한 결과를 발생시킬 수 있다

+ @Stable
객체가 변경될 가능성이 있으나, 변경이 UI에 영향을 주지 않거나 변경이 발생해도 효율적으로 감지할 수 있을 때 사용한다

```kotlin
@Stable
class User(var name: String)
```
위의 코드 같은 경우 name 인자가 var로 선언되어 unstalbe이지만, @Stable을 통해 해당 객체를 안정적으로 관리한다고 판단한다        
따라서 해당 객체가 변경되지 않는다면 Recomposition을 생략할 수 있다

+ @Immutable
객체가 완전한 불변임을 나타내며, 객체의 모든 속성이 초기화된 후에 변경되지 않음을 의미한다      

```kotlin
@Immutable
data class Address(val street: String, val city: String)
```
위의 코드 같은 경우 모든 속성이 val이기 때문에 초기화 이후 변경되지 않으며, Recomposition 생략이 가능하다

결국 Immutable하다는 것은 stable하다는 것을 의미한다