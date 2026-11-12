# Coroutine
`Coroutine`은 light-weight threads라고도 합니다. Thread를 차단하지 않고 실행을 정지할 수 있어, 하나의 Thread에서 여러 개의 `Coroutine`을 실행할 수 있습니다.

Coroutine은 Thread에 묶이지 않습니다. 한 Thread에서 실행하다 중단되었다가 다른 Thread에서 재개될 수 있습니다.

> Q. 하지만 기존 Thread로 돌아오도록 Dipatch하는 것을 권장하는데, 이를 특징으로 얘기해도 되는건가?

코드는 기본적으로 순차적으로 진행되고, 동시성은 명시적으로 생성합니다. (async, launch)

## 중단과 재개
`Coroutine`에서 `delay()`, `await()`과 같은 함수를 호출하게 되면 `Coroutine`이 중단됩니다.  
중단될 때 `Coroutine`에서 가지고 있던 지역 변수나 상태 번호 정보들이 `Continuation` 객체에 저장됩니다.  
이후 다른 Thread에서 `resume(value)`를 호출하면 dispatcher가 지정한 Thread에서 `Coroutine`이 다시 재개됩니다.

> Q `Continuation` 객체에 저장하고 난 후에 어떤 과정이 지난 후에 다른 Thread에서 `resume(value)`을 호출하게 되는 것일까

Coroutine 내에 중단점이 없는 경우에는 해당 Coroutine 작업이 Thread를 계속 선점합니다. 만약 오래 걸리는 작업을 하게 될 경우에는 루프 내에 중단점을 집어넣어 양보하도록 하는 방법도 있습니다.  
CPU 집약 작업은 단일 Thread가 아닌 다중 Thread Pool인 `Dispatchers.Default`를 사용합니다.

> Q `yield()` 함수는?

## 빌더
- launch : `Job`을 반환합니다.
- async : `Deferred`을 반환하며 `await()`을 통해 결과를 받습니다. 만약 `start = LAZY`라면 `await/start`에서 시작합니다.
- runBlocking : 현재 Thread를 막고 완료까지 대기합니다. 이는 `main`, 테스트, blocking과 suspend의 연결용입니다. 기본 dispatcher는 호출 Thread의 이벤트 루프입니다.
- coroutineScope { } : 자식이 모두 끝날 때까지 완료되지 않는 suspend 함수

> Q 이는 main, 테스트, blocking과 suspend의 연결용이다 라는 말이 무슨 말일까

## 구조적 동시성
새 `Coroutine`은 `CoroutineScope`에서만 띄울 수 있습니다. 그리고 `CoroutineScope`은 `Coroutine`의 수명을 제한합니다. `Scope`는 `CoroutineContext`만 가진 껍데기입니다.  
`Coroutine`의 `Job`은 `Scope` context의 `Job`을 부모로 갖는 자식 `Job`입니다. `Job`들은 트리 구조를 가집니다.  
부모는 자식이 모두 끝나야 완료되며, 취소와 예외가 계층을 따라 전파됩니다.  

## Coroutine과 Thread의 차이점
### 중단 방식 차이
Thread의 경우에는 OS가 언제든 강제로 작업을 끊을 수 있습니다. Thread의 중단 시점에서는 실행 상태를 저장 및 복원하게 됩니다.

Coroutine은 스스로 중단점에서 양보합니다. Coroutine의 중단 시점에는 Cotinuation 객체에 지역 변수와 상태 번호를 저장하고 이를 통해 상태를 복원합니다.

> Thread의 강제로 작업을 끊는 것은 왜 비교되는 부분인거지?

> Thread의 중단 시점에서는 값들을 어디에 저장하고 복원하는지?

### 대기 방식 차이
Thread의 경우에는 `WAITING` 혹은 `TIMED_WAITING` 상태가 되며 기다리는 동안 해당 Thread는 다른 동작을 하지 않습니다. `nofity`/`signal`, `join`등과 같은 방식으로 Thread가 lock을 다시 경쟁해 얻고 이어서 실행하도록 할 수 있습니다.

Coroutine의 경우에는 멈추게 되면 Coroutine이 돌아가던 Thread가 풀려나게 되며, 이 Thread는 아까의 Coroutine 대신 다른 Coroutine 작업을 실행할 수 있습니다. Coroutine이 `COROUTINE_SUSPEND`를 반환하게 되면 Continuation 객체에 지역 변수와 상태 번호가 필드에 저장되고, 실행해야 할 다른 Coroutine들이 있다면 이를 실행합니다. `resume()`이 호출되어 다시 재개할 수 있으며, Continuation 객체에 저장했던 정보를 바탕으로 이어서 실행합니다.

## 중단점
코드 안에서 `suspend 함수`를 호출하는 지점으로, `delay()`, `await()`, `Job.join()`등이 있습니다.

중단점은 세 가지 성질이 있음
1. 중단될 수 있는 지점이지, 항상 중단되는 지점은 아님 (함수를 중단하고 반환하고, 아니면 결과를 그대로 받아 계속 진행)
2. 일반 함수 호출은 중단점이 아님 (suspend 함수가 일반 함수를 호출하는 건 제한이 없고, 일반 코드에는 중단점이 없음)
3. 컴파일러가 중단점마다 상태를 하나씩 만들어 (중단점이 N개인 람다는 중단점 하나당 하나의 상태가 생성됨, 그 상태 번호와 지역 변수가 continuation 객체에 저장되고, 재개할 때 그 상태부터 이어서 실행)

Coroutine 내에서 A Coroutine 작업 도중 await()으로 B Coroutine 작업의 값을 불러올 때, 만약 B Coroutine 작업이 완료되어 있다면, 값을 받아올 때 중단되지 않습니다.  
하지만 B Coroutine 작업이 완료되지 않았다면, 값을 받기까지 `COROUTINE_SUSPEND` 마커와 함께 대기하게 됩니다.  
대기 기간 동안 다른 Coroutine 작업을 진행하고, 다른 Thread에서 `resume(value)`를 호출했을 때 dispatcher에 의해서 지정된 Thread에서 Coroutine 작업을 이어가게 됩니다.

중단점이 0개인 suspend 함수도 종종 있습니다. 기본적으로 중단점이 없다면 suspend로 두지 않는 것이 좋습니다. 하지만 suspend를 요구로 하는 시그니처가 존재하거나 앞으로 중단점이 생길 수 있을 경우는 사용하기도 합니다. 혹은 테스트 코드를 작성할 때에도 종종 이런 경우들이 있습니다.   
예를 들어, 안드로이드에서 Network 작업을 진행하는 repository라고 한다면 함수들이 suspend로 존재합니다. 이에 대한 테스트 코드를 작성한다고 했을 때에는, suspend 함수에 중단점을 넣지 않은 채로 구현하여 테스트를 진행할 것입니다.  
위와 같은 몇몇 경우들이 아닌 경우에는 보통 중단점 없는 suspend 함수는 사용하지 않습니다.

## CoroutineContext
Coroutine 관련 요소들의 불변 집합입니다. Scope는 CoroutineContext를 가지고 있습니다.  
Scope에서 Builder로 Coroutine을 생성하여 실행하게 되면 그 Coroutine은 Scope의 CoroutineContext를 받아 자신의 Job을 자식으로 갖는 새로운 CoroutineContext를 가지게 됩니다.

## Dispatcher
CoroutineContext 안에 Dispatcher에 대한 정보가 있습니다. Dispatcher는 Coroutine의 작업을 어떤 Thread에서 실행하거나 재개할지를 결정합니다.

## CPS 변환
CPS란 Continuation-Passing Style의 약자입니다. 함수 결과를 return 하는 대신 결과를 받아서 할 일을 인자로 받아 이를 호출해 넘기는 방식입니다.   
그 다음에 할 일을 객체로 들고 다니기 대문에 함수가 중간에 멈췄다가 나중에 다시 객체를 호출해서 이어갈 수 있습니다.  

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

kotlin의 모든 suspend 함수는 컴파일 시 컴파일러가 `Continuation<T>` 인자로 추가하고 반환 타입을 `Any?`로 변경합니다.  

ex)
```kotlin
//컴파일 전
suspend fun foo(a: Int): String

//컴파일 후
fun foo(a: Int, cont: Continuation<String>): Any?
```

이 때 CPS 변환 후 Any 타입이 되는 것은 중단하지 않았을 경우에는 기존 String 값을 반환하지만, 중단하게 되는 경우 `COROUTINE_SUSPEND` 마커를 반환해야 하기 때문입니다.  

> CPS 변환 과정을 실제로 돌려보고 결과 내리는 것도 재밌겠다

## Continuation 객체
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

## GlobalScope

> GlobalScope의 정의 와 개념 먼저 정리

`GlobalScope`는 `Job`이 없어서 수명을 책임질 주체가 구조에 없습니다. 따라서 문서는 `async / launch`에 쓰는 걸 권장합니다. 앱 수명 전체에서 도는 최상위 작업 같은 드문 경우에만 인정합니다.  

GlobalScope의 coroutineContext는 `EmptyCoroutineContext`를 반환합니다. `Job`이 없으니 여기서 새로운 `Job`을 생성할 시 부모가 없는 `Job`이 발생하게 됩니다. 
기존 Coroutine의 경우 부모가 있기 때문에 작업을 실행하다 예외가 발생할 시 자식부터 부모까지 예외가 전파되며 종료됩니다.  
하지만 GlobalScope에서는 부모가 없는 `Job`이므로 작업이 중단되어도 `Job`이 계속 실행됩니다. 따라서 자원이 낭비될 가능성이 존재합니다.

`GlobalScope`는 앱 전체 수명에서 사용하는 용도로, 여기서 생성한 `Job`은 취소하거나 기다릴 주체가 구조에 존재하지 않기 때문에 `Job`이 관리될 수 없습니다.   
따라서 예외 상황이 생긴다면 취소하거나 이를 관리할 수 없기 때문에 특수한 상황이 있는 것이 아니라면 추천하지 않습니다.

그렇지만 `GlobalScope`에서 `Job`을 실행할만한 상황 또한 있습니다. 라이브러리 혹은 저수준의 코드일 때 `Dispatchers.Unconfined`에서 도는 연산자일 경우입니다. 혹은 애플리케이션 전체에서 도는 최상위 Coroutine의 경우에도 이를 허용하지만, 세밀한 검토를 통해 필요한 지 확인해야 하며 확인 후에도 `@OptIn`을 붙여서 확인했음을 의도적으로 표시하도록 합니다. 

Q1-1. 하나의 긴 작업을 하는 Coroutine이 단일 dispatcher 환경에서 돌아가고 있다면, 다른 Coroutine은 어떻게 돌아갈까?
-> 직접 코드로 확인해보고 싶다
**실행 방법**
- 단일 thread dispatcher를 만들고 Coroutine A, B를 띄우기
- A는 중단점 없이 약 2초간 while로 CPU 소모
- B는 시작 시각을 출력해. B가 A보다 늦게 출발하는지, 몇 ms에 시작하는지 확인
- 그다음 A 루프에 `yield()`를 넣고 B의 시작 시간이 어떻게 달라지는 지 확인해보기

Q1-2. `yield()` 함수 내부 코드 확인하고, 어떤 동작을 하는 함수인지 정리해보기