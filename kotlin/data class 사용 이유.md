# Data class
Data Class를 사용하면 보일러플레이트 코드를 자동으로 생성하여, 개발 생산성을 높일 수 있고, 코드의 가독성을 향상시킬 수 있다

### 1. 보일러플레이트 코드 감소
Data Class에서 toString(), equals(), 'hashCode()'와 같은 메소드들을 자동으로 생성해준다     
일반 클래스에서는 이런 메소드들을 직접 구현해야 하지만, data class는 이를 자동 제공하여 개발자가 반복적으로 작성해야 하는 코드를 줄여준다

copy() 메소드는 기존 객체의 값을 복사하면서 일부 프로퍼티 값을 변경할 수 있게 해주기 때문에 불변 관리에 유용합니다

### 2. 데이터 동등성 비교
equals() 메서드가 자동으로 구현되며, 서로 다른 객체여도 모든 프로퍼티 값이 같다면 true를 반환한다
-> == 연산으로 데이터의 동등성을 쉽게 비교할 수 있다

### 3. Destructuring Declarations 지원
data class는 내부적으로 componentN()메소드를 생성하여 구조 분해를 지원해준다

**Q 그러면 내가 어떤 클래스를 생성하든 component 연산자를 선언하면 구조 분해 선언이 가능할까?**
가능하다!

```kotlin
class A{
    var name = ""
    operator fun component1() = name
    operator fun component2(): (String) -> Unit = {name = it}
}

val (name, setName) = A()
```
위와 같이 선언해도 정상 동작한다!

### 4. 가독성 향상
데이터를 저장하거나 전달하기 위한 용도로 설계된 클래스임을 명시적으로 보여주므로, 코드의 의도를 정확히 파악할 수 있다
