# `@OptIn`
어노테이션이 연결된 파일, 선언, 식에서 Opt-in API를 사용할 수 있다

```kotlin
@OptIn(ExperimenatlMaterial3Api::class)
@Composable
fun A(){

}
```
위와 같이 설정하면 A 선언에 대해 실험적인 Material3 Api를 사용할 수 있도록 해주는 어노테이션이다

실험적인 API를 사용하기 위해서 해당 API가 정의된 패키지에서 @OptIn 어노테이션을 사용해 선언해야 한다

## 고민
@ExperimentalMaterial3Api 어노테이션을 직접적으로 붙이는 경우도 꽤 있었다       

**그러면 `@ExperimentalMaterial3Api`과 `@OptIn(ExperimentalMaterial3Api::class)`는 무슨 차이일까?**     
ExperimentalMaterial3Api 어노테이션은 개발하는 부에서 붙이는 어노테이션이다!        
즉, 라이브러리 쪽에서 붙이는 어노테이션이다.

"이 API는 실험적이니 안정성을 보장하지 않아요!"라는 의미의 어노테이션이다       

OptIn(ExperimentalMaterial3Api::class) 어노테이션은 사용자가 붙이는 어노테이션이다.     
즉, 사용 함수, 클래스, 파일 단위로 붙일 수 있습니다.

"나는 실험적 API를 쓸 걸 알고 있고, 괜찮으니 경고를 없애달라"는 선언이다.


> ExperimentalMaterial3Api는 위험한 거니까 쓸 사람들은 알아둬!
> OptIn은 위험한 거 알고 있으니까 그만 알려줘!

**실수에 대한 공유**        
`@ExperimentalMaterial3Api`을 사용하면 그 함수를 호출하는 위쪽 함수에서도 모두 붙여야 했는데, 이는 해당 어노테이션을 사용함으로써 불안정한 API임을 말해주는데, 그것을 상위로 계속 올려서 사용하는 모든 범위에 어노테이션을 달아야 한다