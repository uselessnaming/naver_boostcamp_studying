# Thread
프로세스 내에서 실제로 작업을 수행하는 실행의 흐름 및 최소 작업 단위를 의미합니다. 하나의 프로세스는 최소 1개의 스레드를 가지고, 필요에 따라 여러 스레드를 생성해 여러 작업을 처리할 수 있습니다.  

> 참고 자료  
> [object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html)  
> [kotlin thread](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.concurrent/thread.html)  

## 특징
- Thread는 프로세스에 할당된 Heap, 데이터, 코드 영역 등의 메모리를 같은 프로세스 내의 Thread 끼리 공유합니다.  
- 또한 각 Thread는 고유한 Stack과 레지스터를 따로 가져 각각의 함수 호출 및 실행 순서를 유지합니다.  

## 장점
- 메모리를 공유하므로 데이터를 주고 받기 쉽고 프로세스에 비해 생성, 전환 비용이 낮습니다.  
- Multi Thread를 활용해 사용자가 웹 페이지를 스크롤 하면서 동시에 이미지가 로딩되게 만드는 등의 작업이 가능합니다.

## 단점
- 여러 Thread가 동시에 같은 자원에 접근하면 충돌 혹은 데이터 무결성 문제가 발생할 수 있습니다 (race condition)
- 실행 순서 예측이 어려워 에러를 잡기 어렵습니다.

## Thread의 상태
- `NEW` : Thread 객체가 생성되었으나 `start()`가 호출되지 않은 상태
- `RUNNABLE` : JVM에서 실행 중이거나 CPU 실행 대기(Ready) 상태인 경우
- `WAITING` : synchronized 에서 Lock을 기다리는 상태를 의미  
- `BLOCKED` : monitor lock 대기이며, Lock을 기다리는 상태
- `TIMED_WAITING` : `sleep()` 혹은 타임 아웃이 있는 `wait()`이나 `join()`과 같이 특정 시간 동안 대기하고 있는 상태
- `TERMINATED` : run()이 끝났거나 예외로 인해 Thread가 종료된 상태

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
`Thread(runnable).run()`을 하게 되면 해당 thread 내부 작업을 현재 thread에서 진행합니다.  

예를 들어, 기존 main thread인 상태에서 threadA에서 task1, task2, task3을 진행하도록 정의했다고 가정합니다. 여기서 main thread 내부에서 threadA.run()을 하게 되면 main thread인 상태에서 task1, task2, task3이 진행됩니다.

### start()
`Thread(runnable).start()`는 새로운 thread를 생성하여 해당 thread 위에서 task들을 실행시킵니다. 
즉, `run()`과 다르게 main thread에서 `start()`호출하게 된다면 main thread가 아닌 또 다른 thread가 생성되며 여기서 task들이 실행됩니다.  

단, 이미 `start()`를 호출한 thread에 `start()`를 재호출하게 되면 `IllegalThreadStateException`이 발생하게 됩니다. 따라서 재호출에는 주의해야 합니다.

### join()
`Thread(runnable).join()`을 하게 되면 현재 thread에서 해당 thread의 작업이 끝나고 종료할 때까지 대기하게 됩니다.  
이 때 main thread는 blocking되고, blocking 되는 동안은 어떠한 작업도 할 수 없습니다.

## My Thread 구현
[MyThread 구현](https://github.com/uselessnaming/cs-kotlin/blob/main/src/thread_coroutine/MyThread.kt)   

Thread는 기본적으로 상태 값을 가지고 있으며, 진행 상황에 따라 상태를 변경해줘야 합니다.  

state는 lock으로 race-condition을 방지해줘야 합니다. 이 때 state는 main thread와 internal thread, 2개의 thread에서 접근이 가능하기 때문에 lock으로 관리되어야 합니다.

synchronized() 함수를 사용하면 동시성 문제를 해결할 수 있습니다. lock의 `wait()`를 호출해 lock을 반납하고, 깨어난 뒤 lock을 다시 얻으면 이후 코드가 실행됩니다. 이제 상태를 변경한 thread가 `lock.notifyAll()`을 호출해 상태가 변경되었음을 알립니다.  

```kotlin
while(state != State.TERMINATED) { lock.wait() }
```

여기서 `if(state != State.TERMINATED)` 가 아닌 `while(state != State.TERMINATED)`를 사용하게 됩니다. 공식 문서를 살펴보면 허위 기상이라 하여 `notify()`가 오지 않았음에도 thread가 기상할 수 있다고 합니다. 따라서 while을 통해 반복적으로 조건을 확인해줘야 합니다.

## race condition
별도의 조치 없이 여러 thread에서 하나의 공유 자원에 접근하려고 했을 경우 충돌 혹은 예상치 못한 동작이 이뤄질 수 있습니다.  

![alt text](<race condition 발생 결과.png>)  
실제로 count를 공유 자원으로 마련하고 여러 thread에서 해당 count에 접근하도록 했을 때 예상 값은 500000인 것에 비해 실제 계산 값은 210284였음을 알 수 있습니다.  
이를 방지하기 위해서는 접근하고자 하는 공유 자원 역시 lock을 통해 동기화해줘야 함을 알 수 있습니다.  

결과적으로 race-condition을 방지하기 위해서는 병렬로 처리되던 작업을 순차 실행하여 한 번에 하나의 thread가 자원에 접근할 수 있도록 합니다.  

```kotlin
var count = 0
val lock = Object()

val runnable1 = Runnable {
    synchronized(lock) {
        repeat(100000) {        
            count++
        }
    }
}

val runnable2 = Runnable {
    repeat(100000) {`
        synchronized(lock) {
            count++
        }
    }
}
```
runnable1과 runnable2 중 어떤 것이 더 적합한 코드일까요?  
이 경우, runnable1이 성능적으로 더 우월합니다. runnable2가 세세한 조절이 가능하여 더 좋아보이지만 시간을 측정해보면 그렇지 않습니다.   
synchronized()를 사용하게 되면 lock을 점유하고, 놓는 것에도 비용이 발생하게 됩니다. runnable1 같은 경우에는 해당 비용이 1번 발생하는 것이지만, runnable2 같은 경우 반복하며 100000번 비용이 발생하기 때문에 처리 속도에서 큰 차이가 발생하게 됩니다.

### Q1 왜 Object에 notifyAll()과 wait() 코드가 포함되는가?
synchronized()를 사용하면서 lock을 사용하게 됩니다. 여기서 대부분의 코드를 확인해보면 lock을 Object()로 선언합니다. 이 Object는 java에서 모든 class의 기반이 되는 class입니다.  
`notifyAll()`과 `wait()` 모두 java에서의 최상위 클래스인 Object 함수로, 이를 상속받는 하위 클래스들은 모두 사용할 수 있습니다.  
> kotlin에서는 `Any()`가 최상위 클래스로 여기서는 `notifyAll()`, `wait()`를 사용할 수 없습니다. `Any()`는 JVM 전용이 아니라서 `toString()` / `hashCode` / `equals`만 갖고 있습니다.

다른 클래스들의 역할에 `notifyAll()`과 `wait()`이 필요 없는데, 왜 Object에 해당 코드가 들어있는 지 의문이 들었습니다. 

Java의 최초 설계 당시에 모든 객체들은 synchronized()를 위한 내장 모니터를 가지게 설계되었다고 합니다.  

결과적으로 Object에 포함되어 있던 부분을 `Lock`과 `Condition`이라는 별도의 타입으로 락 기능을 분리해서 관리할 수 있게 되었습니다. 물론 이미 `notifyAll()`, `wait()`을 너무 오래 사용해왔기 때문에 없애지는 못한 상황입니다.  

> [lock과 관련된 참고 자료](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/package-summary.html)

`Lock`과 `Condition`이 기존 Object로 사용하던 lock이 락 획득, 조건 대기, 신호가 명시적인 메서드로 분리돼 의도가 잘 드러난다고 생각되어 Object로 구현했던 lock을 `Lock`과 `Condition`을 사용하는 방식으로 변경했습니다.