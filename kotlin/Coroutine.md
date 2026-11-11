# Coroutine

Q1. 코루틴 자신이 컨텍스트 전환을 한다는 의미를 모르겠음. 스레드가 코루틴을 전환시켜주는 것이 아니었나?

A1. Thread의 경우에는 OS가 언제든 강제로 끊을 수 있음 (선점). 그에 비해 Coroutine의 경우, 중단점에서만 스스로 양보하게 됨
Thread는 context-switch 시 레지스터와 스택 등 실행 상태를 저장 및 복원하게 됩니다. 그에 비해 Coroutine은 상태를 continuation 객체에 저장하고 하고 함수가 반환됩니다.

Q1-1. 하나의 긴 작업을 하는 Coroutine이 단일 dispatcher 환경에서 돌아가고 있다면, 다른 Coroutine은 어떻게 돌아갈까?
-> 직접 코드로 확인해보고 싶다
**실행 방법**
- 단일 thread dispatcher를 만들고 Coroutine A, B를 띄우기
- A는 중단점 없이 약 2초간 while로 CPU 소모
- B는 시작 시각을 출력해. B가 A보다 늦게 출발하는지, 몇 ms에 시작하는지 확인
- 그다음 A 루프에 `yield()`를 넣고 B의 시작 시간이 어떻게 달라지는 지 확인해보기

A1-1. 코드 내에서 중단점이 없는 Coroutine은 종료되거나 중단점을 만날 때까지 Thread를 계속 사용합니다. 따라서 다른 Coroutine 작업을 실행되지 못합니다. 이를 해결하기 위해 오래 도는 루프 안에 중단점을 넣어 스스로 양보하는 방법이며 `yield()`가 대표적으로 있습니다. 도한 CPU 집약 작업은 단일 Thread가 아닌 다중 Thread Pool인 `Dispatchers.Default`를 사용합니다.

Q1-2. `yield()` 함수 내부 코드 확인하고, 어떤 동작을 하는 함수인지 정리해보기

Q2. 중단점에서 context switch가 발생한다고 했는데, 여기서 중단점이란?

A2. 코드 안에서 `suspend 함수`를 호출하는 지점으로, `delay()`, `await()`, `Job.join()`등이 있습니다.

중단점은 세 가지 성질이 있음
1. 중단될 수 있는 지점이지, 항상 중단되는 지점은 아님 (함수를 중단하고 반환하고, 아니면 결과를 그대로 받아 계속 진행)
2. 일반 함수 호출은 중단점이 아님 (suspend 함수가 일반 함수를 호출하는 건 제한이 없고, 일반 코드에는 중단점이 없음)
3. 컴파일러가 중단점마다 상태를 하나씩 만들어 (중단점이 N개인 람다는 중단점 하나당 하나의 상태가 생성됨, 그 상태 번호와 지역 변수가 continuation 객체에 저장되고, 재개할 때 그 상태부터 이어서 실행)

Q2-1. 중단점이 중단될 수 있는 지점이다 라는 것은 결국 중단되지 않을 수 있다라는 뜻인 것 같은데 왜 이게 가능한걸까?

A2-2. MainThread에서 Coroutine A작업을 진행하고 있습니다. 이 때 B작업의 await()을 호출했는데, B작업을 실행하고 나서 완료되기 전 시점이라면 A 작업을 continuation 객체에 지역 변수와 재개할 위치(상태 번호)를 저장하며 A 작업을 담당하던 Thread가 풀려나게 됩니다. 이 때 단일 thread dispatcher 환경인 경우 A작업 대신 B작업이 진행되는 것이지만 만약 멀티 thread dispatcher환경이라면 B작업이 진행되는 동안 A 작업을 진행하던 dispatcher의 thread는 다른 작업을 진행합니다. 이후 완료되면 resume(value)를 통해 기존 A 작업의 진행 상황을 복구하고 이어가게 됩니다.

Q2-3. 중단점이 0개인 suspend 함수의 필요성은 뭘까?

A2-3. 일반적으로 중단점이 없으면 suspend를 사용하지 않는 것이 좋습니다. 다만 suspend 함수 타입을 요구하는 시그니처가 있어 이를 맞춰주기 위해서, 추후에 중단점이 생길 가능성이 있어서인 경우들에 suspend를 사용하기도 합니다. 또한 안드로이드에서 Repository를 작성할 때 네트워크 작업으로 인한 suspend 를 사용하곤 하는데, 이를 테스트하기 위해 fake repository를 생성할 떄에는 중단점을 없애 override하는 것이 더 합당합니다.

Q3. 구조적 동시성을 말하면서 새 코루틴을 특정 CoroutineScope에서만 띄울 수 있다고 했는데, 이는 소유한다는 의미일까?

A3. 네. 새로운 Coroutine은 특정 CoroutineScope에서만 띄울 수 있기 때문에, 그 Scope이 Coroutine의 수명을 한정합니다.

Q3-1. Coroutine은 Thread에 종속되는 것은 아니라고 알고 있어서 CoroutineScope에는 종속되는 것인가?

A3-1. Coroutine은 CoroutineScope의 수명에 종속된다는 말은 맞는 말입니다. 하지만 CoroutineScope은 등록된 Coroutine이 모두 끝나기 전까지 계속 진행된다라는 말은 틀린 말입니다. Scope 자체는 CoroutineContext를 들고 있는 홀더로, 자식이 끝나길 기다리는 것은 coroutineScope {} / runBlocking 이거나 CoroutineScope(Dispatchers.Default) 입니다.

Q3-2. COroutineScope과 coroutineScope{} , CoroutineScope(Dispatchers.Default)의 차이점

A3-2. CoroutineScope은 coroutineContext를 가진 인터페이스입니다. CoroutineScope(Dispatchers.Default)는 팩토리 함수로 context에 job이 없다면 기본 job을 만들어 둡니다. coroutineScope{}은 suspend 함수로써 이는 자식이 다 끝날 때까지 완료되지 않는 호출입니다.

Q3-3. Coroutine, CoroutineContext, Dispatcher, Scope의 관계

A3-3. CoroutineContext는 Coroutine 관련 요소들의 불변 집합입니다. Scope는 이를 들고 있습니다. 그 Scope에서 Builder로 Coroutine을 실행하면 Coroutine은 context를 상속하며, 자신만의 새로운 Job을 자식으로 갖습니다. context 안의 Dispatcher는 Coroutine이 어떤 Thread에서 실행하거나 재개할지를 결정합니다.

Q4. CPS란?
> CPS 변환 과정을 실제로 돌려보고 결과 내리는 것도 재밌겠다

A4. CPS는 Cotinuation-Passing Style을 의미합니다. 함수 결과를 return하는 대신 결과를 받으면 이어서 할 일을 인자로 받아 그걸 호출해 결과로 넘기는 방식입니다.

ex)
```kotlin
// 일반 코드 스타일
fun add(a: Int, b: Int): Int = a + b
val r = add(1, 2)
println(r)

// CPS
fun add(a: Int, b: Int, next: (Int) -> Unit) { next(a + b) }
add(1, 2) { r -> println(r) }
```

그 다음에 할 일을 객체로 들고 다니기 대문에 함수가 중간에 멈췄다가 나중에 다시 객체를 호출해서 이어갈 수 있습니다.

kotlin의 모든 suspend 함수는 컴파일러가 CPS로 변환합니다.

ex)
```kotlin
//컴파일 전
suspend fun foo(a: Int): String

//컴파일 후
fun foo(a: Int, cont: Continuation<String>): Any?
```

이 때 CPS 변환 후 Any 타입이 되는 것은 중단하지 않았을 경우에는 기존 String 값을 반환하지만, 중단하게 되는 경우 `COROUTINE_SUSPEND` 마커를 반환해야 하기 때문입니다.  

Q5. continuation 객체란?

A5. 중단점 이후 이어서 실행될 부분을 나타내는 객체입니다. 

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    private fun resumeWith(result: Result<T>)
}
```
와 같이 존재합니다.

`resumeWith(result)`은 Coroutine의 실행을 재개하고, 성공이나 실패 결과를 마지막 중단점의 반환 값으로 전달합니다. 중단된 Coroutine에서 보면, `await()`가 값을 받아서 돌아오는 순간이 이 호출입니다.

`context`는 해당 Coroutine의 CoroutineContext를 의미합니다.  

결과적으로 Continuation 객체는 어디까지 실행했는지, 그 당시 변수 값, 끝나면 누굴 깨울지 등에 대한 정보를 저장합니다.

Q6. 결국 Coroutine을 돌리기 위해서는 runBlocking{} 위에서 돌려야 하는 것 같은데 그럼 결국 이게 Thread의 join과 다를게 뭐지?

A6. runBlocking이 아니더라도 Coroutine을 돌리는 방식은 많습니다. 다만 runBlocking으로 진행할 경우 현재 진행 중인 작업이 모두 종료될 때까지 해당 Thread를 막게 되므로 이는 Thread의 join()과 비슷하게 동작한다고 생각할 수 있습니다.

# async / Deferred / await()
Q1. thread의 대기 방식과 coroutine의 대기 방식 차이

A1. Thread의 경우에는 `WAITING` 혹은 `TIMED_WAITING` 상태가 되며 기다리는 동안 해당 Thread는 다른 동작을 하지 않습니다. `nofity`/`signal`, `join`등과 같은 방식으로 Thread가 lock을 다시 경쟁해 얻고 이어서 실행하도록 할 수 있습니다.

Coroutine의 경우에는 멈추게 되면 Coroutine이 돌아가던 Thread가 풀려나게 되며, 이 Thread는 아까의 Coroutine 대신 다른 Coroutine 작업을 실행할 수 있습니다. Coroutine이 `COROUTINE_SUSPEND`를 반환하게 되면 Continuation 객체에 지역 변수와 상태 번호가 필드에 저장되고, 실행해야 할 다른 Coroutine들이 있다면 이를 실행합니다. `resume()`이 호출되어 다시 재개할 수 있으며, Continuation 객체에 저장했던 정보를 바탕으로 이어서 실행합니다.

Q2. GlobalScope로 async를 시작하는 것이 비추천되는 이유가 이해가 되지 않앗음

A2. GlobalScope의 coroutineContext는 `EmptyCoroutineContext`를 반환합니다. `Job`이 없으니 여기서 새로운 `Job`을 생성할 시 부모가 없는 `Job`이 발생하게 됩니다. 
기존 Coroutine의 경우 부모가 있기 때문에 작업을 실행하다 예외가 발생할 시 자식부터 부모까지 예외가 전파되며 종료됩니다.  
하지만 GlobalScope에서는 부모가 없는 `Job`이므로 작업이 중단되어도 `Job`이 계속 실행됩니다. 따라서 자원이 낭비될 가능성이 존재합니다.

Q2-1. 결국 부모 없는 `Job`은 `Job`을 생성하게 되면 생길 수 있습니다. 그렇다면 어떤 경우에 부모 없는 Job이 문제가 되는 것일까요?

A2-1. 직접 `Job`을 직접 생성하는 경우, `CoroutineScope(context)` 팩토리를 사용할 경우, 위에서 말한 `GlobalScope`를 사용한 경우 등 여러 경우에서 부모 `Job`이 생길 수 있습니다.  
이 떄 부모 `Job` 자체가 문제인 것이 아닙니다. `GlobalScope`는 앱 전체 수명에서 사용하는 용도로, 여기서 생성한 `Job`은 취소하거나 기다릴 주체가 구조에 존재하지 않기 때문에 `Job`이 관리될 수 없습니다.   
따라서 예외 상황이 생긴다면 취소하거나 이를 관리할 수 없기 때문에 특수한 상황이 있는 것이 아니라면 추천하지 않습니다.

Q2-2. 그렇다면 `GlobalScope`에서 `Job`을 실행할만한 상황에는 어떤 것이 있을까요?

A2-2. 일반 코드에서는 권장하지 않고, 라이브러리 혹은 저수준의 코드일 때 `Dispatchers.Unconfined`에서 도는 연산자일 경우가 있습니다. 혹은 애플리케이션 전체에서 도는 최상위 Coroutine의 경우에도 이를 허용하지만, 세밀한 검토를 통해 필요한 지 확인해야 하며 확인 후에도 `@OptIn`을 붙여서 확인했음을 의도적으로 표시하도록 합니다. 

# Dispatcher
Q1. Unconfined는 어떤 상황에서 쓰는 Dispatcher일가

A1. Coroutine을 다른 Thread로 넘기는 dispatch 과정 없이 지금 바로 실행해야 하는 드문 경우에 사용하는 고급 도구이며, 일반 코드에서는 사용하지 않는 것을 권장하고 있습니다.  
- CPU를 쓰지 않고, 특정 Thread에 묶인 공유 데이터도 건드리지 않는 Coroutine일 경우
- 나중에 dispatch하는 것이 필요 없거나 원하지 않는 부작용을 낳는 코너 케이스 (어떤 작업을 즉시 해야 할 때)

A1-1. dispatch 과정은 뭘까? Coroutine을 다른 Thread로 넘기는 과정이 왜 필요하지?

Q1-1. Coroutine이 중단된 동안 resume을 일으키는 Thread와 이어서 실행되어야 할 Thread가 다를 수 있다. dispatch는 Coroutine의 실행 조각을 Dispatcher에 넘겨서, 그 Dispatcher가 정한 Thread에서 실행하도록 하는 단계

Q1-2. Unconfined인 경우 문제가 되는 것은 dispatch 과정이 넘어가게 되는 것. 그래서 이게 뭐가 문제라는 거지?

A1-2. dispatcher가 Unconfined가 아니라면 delay(1000) 호출 시, Coroutine이 중단되고 1초가 지나면 이를 dispatcher의 Thread로 전달하여 기존에 실행되던 Thread에서 Coroutine 작업이 진행된다. 하지만 Unconfined로 해둔다면 이런 dispatch 과정이 사라지기 때문에 기존에 어떤 Thread에서 작업을 했던 것과 다른 Thread로 보내질 수 있고, 그에 따라 추적이 어려워지기 때문에 권장하지 않는다고 할 수 있다.
또한 만약 기존 작업에서 해당 Thread의 묶인 자원에 접근했다고 한다면, 이후 작업에서 다른 Thread로 보내지며 접근이 허용된 Thread가 아닌 Thread에서 실행될 수 있기 때문에 이렇게 사용하면 안된다.

Q2. newSingleThreadContext 또한 어떻게 사용하는 지 예시 필요

A2. 이름에서 보여지듯이 Thread 하나에 Coroutine을 가두는 Dispatcher를 생성하는 함수입니다. 
