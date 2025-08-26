# Remember
remember API는 메모리에 객체를 저장할 수 있다   
remember에 의해 계산된 값은 초기 Composition 과정 중 Composition에 저장되고, 저장된 값을 Recomposition 중 반환된다

> remember를 호출한 composable이 삭제되면 그 객체 자체를 잊음

# MutableState
remember와 세트로 Compose에서 상태를 관리하기 위한 인터페이스이다

mutableStateOf는 관찰가능한 ```MutableState<T>```를 생성하는데, 이는 런타임 시 Compose에 통합되어 관찰 가능한 유형이다

### MutableState 객체 선언 방법
이런 MutalbState 객체를 생성하는 방법은 3가지가 있다

+ ```val mutableState = remember{ mutableStateOf(value) }```
+ ```var value by remember{ mutableStateOf(value) }```
+ ```val (value, setValue) = remember{ mutableStateOf(value) }```

여기서 다른 것보다 by로 get과 set을 value에 위임한 경우에 조심해야 한다

by 키워드를 생성할 경우에는 반드시 value가 var로 선언되어야 하며, getValue와 setValue를 import 해야 한다

> getValue는 항상 적용되지만 종종 setValue는 적용이 안되어 직접 입력하곤 했다

**마지막 방법의 경우**
value에서는 값을 받고, setValue는 <T> -> Unit 으로 되어 있는 갱신 함수이다.

TextField의 onValueChange에서 setValue를 지정하면 간편히 사용할 수 있다

### 사용 분리?
어떤 것을 사용함에 있어 유리하다! 와 같은 것은 명확하게 있지 않다

실제로 [공식 문서](https://developer.android.com/develop/ui/compose/state?hl=ko)에서도 선언은 동일한 것이며, 용도의 상태를 사용하기 위한 Syntax Sugar로 제공됨을 알려준다