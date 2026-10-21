# Thread
프로세스 내에서 실제로 작업을 수행하는 실행의 흐름 및 최소 작업 단위를 의미합니다. 하나의 프로세스는 최소 1개의 스레드를 가지고, 필요에 따라 여러 스레드를 생성해 여러 작업을 처리할 수 있습니다.  

> 참고 자료  
> [object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)  
> [kotlin thread](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.concurrent/thread.html?utm_source=chatgpt.com)  

## 특징
- Thread는 프로세스에 할당된 Heap, 데이터, 코드 영역 등의 메모리를 같은 프로세스 내의 Thread 끼리 공유합니다.  
- 또한 각 Thread는 고유한 Stack과 레지스터를 따로 가려 각각의 함수 호출 및 실행 순서를 유지합니다.  

## 장점
- 메모리를 공유하므로 데이터를 주고 받기 쉽고 응답 속도가 빠릅니다.
- Multi Thread를 활용해 사용자가 웹 페이지를 스크롤 하면서 동시에 이미지가 로딩되게 만드는 등의 작업이 가능합니다.

## 단점
- 여러 Thread가 동시에 같은 자원에 접근하면 충돌 혹은 데이터 무결성 문제가 발생할 수 있습니다 (race condition)
- 실행 순서 예측이 어려워 에러를 잡기 어렵습니다.

## Thread의 상태
- `NEW` : Thread 객체가 생성되었으나 `start()`가 호출되지 않은 상태
- `RUNNABLE` : JVM에서 실행 중이거나 CPU 실행 대기(Ready) 상태인 경우
- `BLOCKED` : 동기화 락(synchronized)를 얻기 위해 다른 Thread의 해제를 기다리는 상태
- `TIME_WAITING` : `sleep()`, `wait()`을 통해 특정 시간 동안 대기하고 있는 상태
- `TERMINATED` : 메서드가 정상 종료되었거나 예외로 인해 Thread가 종료된 상태

## Thread 생성 방법
### Thread를 상속받는 클래스 구현
```kotlin
class MyThread: Thread() {
    ...
}
```
Thread()를 상속받는 클래스를 구현하여 사용합니다. 이 때 상속받은 run()을 구현하여 동작시킵니다.

### Runnable 전달
```kotlin
val runnable = Runnable {
    task()
}
Thread(runnable).start()
```
Thread에서 담당해야 할 runnable을 정의한 뒤 runnable을 전달하는 방식으로, Thread는 자체 상태 관리와 runnable을 동작시키는 것으로 책임 분리를 할 수 있습니다.

### `thread { }` 블록 사용
`val t = thread { }`와 같이 선언할 수 있습니다.  
`thread{ }`의 내부 코드를 확인해보면 

```kotlin
public fun thread(
    start: Boolean = true,
    isDaemon: Boolean = false,
    contextClassLoader: ClassLoader? = null,
    name: String? = null,
    priority: Int = -1,
    block: () -> Unit
): Thread {
    val thread = object : Thread() {
        public override fun run() {
            block()
        }
    }
    if (isDaemon)
        thread.isDaemon = true
    if (priority > 0)
        thread.priority = priority
    if (name != null)
        thread.name = name
    if (contextClassLoader != null)
        thread.contextClassLoader = contextClassLoader
    if (start)
        thread.start()
    return thread
}
```  

이렇게 정의되어 있습니다. 별도의 설정이 없다면 `thread{ }` 블록은 선언할 때 `start()`되는 것을 확인할 수 있습니다.  
그래서 정의해두고 원하는 시점에 thread를 실행시키려면 start 옵션을 false로 설정해둬야 합니다.

### Executor 사용
```kotlin
val executor = Executors.newFixedThreadPool(1)
executor.submit { task() }
executor.shutdown()
```  
Executor를 사용하면 ThreadPool을 관리할 수 있습니다. 사용할 Thread의 개수를 정하고 그 안에서만 동작할 수 있도록 관리를 도와주는 역할을 합니다. task를 등록하면 관리되는 Thread 중 사용 가능한 Thread에 할당하여 처리하게 됩니다.

## Thread 관련 함수
### run()
`thread{ }.run()`을 하게 되면 해당 thread 내부 작업을 현재 thread에서 진행합니다.  

예를 들어, 기존 main thread인 상태에서 threadA에서 task1, task2, task3을 진행하도록 정의했다고 가정합니다. 여기서 main thread 내부에서 threadA.run()을 하게 되면 main thread인 상태에서 task1, task2, task3이 진행됩니다.

### start()
`thread{ }.start()`는 새로운 thread를 생성하여 해당 thread 위에서 task들을 실행시킵니다. 
즉, `run()`과 다르게 main thread에서 `start()`호출하게 된다면 main thread가 아닌 또 다른 thread가 생성되며 여기서 task들이 실행됩니다.

### join()
`thread{ }.join()`을 하게 되면 현재 thread에서 해당 thread의 작업이 끝나고 종료할 때까지 대기하게 됩니다.  
이 때 main thread는 blocking되고, blocking 되는 동안은 어떠한 작업도 할 수 없습니다.

## My Thread 구현
(MyThread 구현)[https://github.com/uselessnaming/cs-kotlin/blob/main/src/thread_coroutine/MyThread.kt]   

Thread는 기본적으로 상태 값을 가지고 있으며, run, start, join 등에 따라 상태를 변경해줘야 합니다.  

state는 lock으로 race-condition을 방지해줘야 합니다. 이 때 state는 main thread와 internal thread, 2개의 thread에서 접근이 가능하기 때문에 lock으로 관리되어야 합니다.

synchronized() 함수를 사용하면 동시성 문제를 해결할 수 있습니다. lock의 `wait()`을 통해 대기하게 됩니다. 대기하다가 lock이 `nofityAll()`을 통해 변경이 있음을 알려줍니다. 해당 알림을 통해 `wait()` 이후 코드가 실행됩니다.

```kotlin
while(state != State.TERMINATED) { lock.wait() }
```

여기서 `if(state != State.TERMINATED)` 가 아닌 `while(state != State.TERMINATED)`를 사용하게 됩니다. 공식 문서를 살펴보면 허위 기상이라 하여 `nofity()`가 오지 않았음에도 thread가 기상할 수 있다고 합니다. 따라서 while을 통해 반복적으로 조건을 확인해줘야 합니다.

### Q1 왜 Object에 notifyAll()과 wait() 코드가 포함되는가?
synchronized()를 사용하면서 lock을 사용하게 됩니다. 여기서 대부분의 코드를 확인해보면 lock을 Object()로 선언합니다. 이 Object는 java에서 모든 class의 기반이 되는 class입니다.  
`notifyAll()`과 `wait()` 모두 Object 클래스의 함수로, 이를 상속받는 하위 클래스들은 모두 사용할 수 있습니다.  
> kotlin에서는 Any()가 최상위 클래스로 여기서는 `nofityAll()`, `await()`은 사용하지 못하도록 막아뒀습니다.

다른 클래스들의 역할에 `notifyAll()`과 `wait()`이 필요 없는데, 왜 Object에 해당 코드가 들어있는 지 의문이 들었습니다. 

Java의 최초 설계 당시에 모든 객체들은 synchronized()를 위한 내장 모니터를 가지게 설계되어있습니다. 이에 따라서 개발자들이 너무 많은 책임을 Object가 가진다고 비판했습니다.  

이런 비판 속에서 이를 분리하여 `Lock`과 `Condition`이라는 별도의 타입으로 락 기능을 분리해서 관리할 수 있습니다. 물론 이미 `notifyAll()`, `wait()`을 너무 오래 사용해왔기 때문에 없애지는 못한 상황입니다.  

> [lock과 관련된 참고 자료](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/package-summary.html)

따라서 MyThread의 Lock을 `Object`에서 `Lock`과 `Condition`으로 교체해서 사용했습니다.