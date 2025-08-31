# . vs ::

## . 연산자
객체의 멤버 접근 및 호출 시 사용한다       
호출 시 함수가 즉시 실행되며, 결과가 반환된다

```kotlin
fun greet(name: String) = "$name Hi"

val msg = greet("alice")
println(msg)
```
위 코드처럼 사용하곤 한다

## :: 연산자
함수 참조를 만드는 용도로 사용된다      
즉시 실행이 아니고, 함수 자체를 값으로 다룬다       

```kotlin
fun greet(name: String) = "$name Hi"

val greeter: (String) -> String = ::greet
println(greeting("alice"))
```
위 코드처럼 사용한다

함수 타입 변수에 저장하거나, 고차 함수를 전달할 때 사용하는 방법이다

## Compose에서의 장점
Compose에서 클릭 혹은 이벤트 처리에서는 람다 전달이 더 효율적이다       
람다를 직접 만들게 되면 상위 Compsoable이 recomposition될 때마다 새로운 람다 객체가 생성되기 때문이다       
따라서 람다 객체가 바뀌었다고 판단외서 하위 Composable 또한 같이 recomposition될 수 있다        

그래서 ::연산자로 전달하게 된다면 함수 참조이므로 매번 새 객체가 생성되지 않는다        
따라서 Compose는 stable로 판단하여 Skippable할 수 있다

. 연산 vs :: 연산
```kotlin
// .연산
Button(onClick = { stateHolder.onClickItem() }){
    Text("click")
}

// ::연산
Button(onClick = { stateHolder::onClickItem }){
    Text("click)
}
```

그리고 remember와 사용하면 안정적인 참조를 유지할 수 있다