# Thread

## Thread 생성 방법
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

## Thread 관련 함수
### run()
`thread{ }.run()`을 하게 되면 해당 thread 내부 작업을 현재 thread에서 진행합니다.  

예를 들어, 기존 main 스레드인 상태에서 threadA에서 task1, task2, task3을 진행하도록 정의했다고 가정합니다. 여기서 main 스레드 내부에서 threadA.run()을 하게 되면 main 스레드인 상태에서 task1, task2, task3이 진행됩니다.

### join()
`thread{ }.join()`을 하게 되면 현재 thread에서 해당 thread의 작업이 끝나고 종료할 때까지 대기하게 됩니다.  
이 때 main thread는 blocking되고, blocking 되는 동안은 어떠한 작업도 할 수 없습니다.