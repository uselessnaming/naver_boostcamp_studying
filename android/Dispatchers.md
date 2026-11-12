# Dispatchers
Coroutine들을 적절하게 Thread 혹은 Thread pool에 할당하여 실행시키는 Scheduler의 역할을 하는 CoroutineContext의 일부

[참고 블로그] (https://jtm0609.tistory.com/263)

+ 주말에 추가 정리하기

## Dispatchers.Default
코루틴의 기본 Dispatcher        

> runBlocking의 경우 내부적으로 자체 Dispatcher를 사용      
> 기본적으로 자신을 호출한 Thread를 기본 Dispatcher로 활용      
> ViewModelScope의 기본 Dispatcher는 Dispatchers.MAIN

해당 Dispatcher는 CPU Intensive 작업을 위해 설계됩니다      
CPU 코어 수와 동일한 개수의 Thread Pool을 사용하며, 최소 설정 값은 2개입니다      
이론상 스레드를 효율적으로 사용한다고 가정하며 최적의 Thread수는 CPU 코어 수

### Limiting the default dispatcher
무거운 작업이 발생해 Dispatchers.Default의 모든 Thread를 독점하게 된다면?       
이런 경우 같은 Dispatchers를 사용하는 다른 Coroutine들은 해당 Thread를 사용하지 못하고 starve한 상태가 됩니다     

Dispatcher 내 Thread의 개수를 limitedParallelism을 통해 제한할 수 있습니다

limitedParallelism을 사용하면 특정 프로세스에 할당된 Thread 수를 제한할 수 있습니다     
단일 프로세스가 너무 많은 Thread의 독점을 막을 수 있습니다      
당연히 Dispatchers.Default의 limitedParallelism는 CPU 코어 수를 넘겨서 설정할 수 없고, 만약 넘겨서 설정하게 되면 무시됩니다

만약 Dispatchers.Default에서 무거운 작업을 하게 되면 해당 Thread는 작업이 완료될 때까지 다른 Coroutine이 사용할 수 없습니다     
그런데 Default는 CPU의 코어 수를 최대 개수로 Thread 수가 정해져있습니다     
따라서 무거운 작업을 Dispatcher.Default에서 진행하게 될 경우 그 동안 다른 Coroutine은 사용하지 못하며, 이로 인한 지연이 상당히 발생할 수 있습니다

이런 배경에서 IO Dispatcher가 등장했습니다

## Dispatchers.IO
파일 읽기, DB 접근, Network 요청과 같은 Blocking IO 작업을 처리하도록 설계      
최소 64개의 Thread 제한이 있어 동시 IO 작업에 최적화되어있습니다

해당 Dispatcher는 외부 리소스를 기다리는 작업에 적합하지만 CPU intensive 작업을 위한 것은 아닙니다

내부적으로 Thread 할당 시 UnlimitedIoScheduler를 사용합니다     
Dispatchers.IO는 제한 없는 ThreadPool 크기를 가지고 있고, 초기 상한은 64개로 되어 있습니다

### Dispatchers.IO와 Dispatchers.Default
두 Dispatcher는 Thread Pool을 공유합니다      
따라서 작업 간 Thread를 변경하기 위한 Dispatching이 자주 필요하지 않습니다        
그렇기 때문에 Thread를 새로 할당하는 오버헤드가 줄고, 불필요한 ContextSwitching이 줄어듭니다      

하지만 각각의 Dispatcher의 제한은 독립적으로 동작합니다       
즉 한 쪽 Thread를 많이 사용한다고 다른 쪽의 Thread가 방해받는 일은 없습니다

### Dispatchers.IO에서 limitedParalleliism의 값을 크게 한다고 다 좋을까?
너무 많은 수를 설정하게 된다면 Dispatcher가 필요보다 더 많은 Thread를 사용하고, 병렬 처리를 시도합니다        
이 때 Thread가 너무 많이 생성되고, 그에 따라 ContextSwitching의 빈도가 증가합니다     
그로 인해 오버헤드가 발생하여 성능이 저하될 수 있습니다

또한 시스템의 메모리를 고갈시켜 OOM이 발생할 수 있고, Race Condition 문제 또한 발생할 수 있습니다

### Dispatchers.IO에서 CPU-instensive 작업을 왜 하면 안되나?
CPU intensive 작업에서는 Thread pool이 너무 커지면 CPU가 오히려 비효율적으로 동작합니다       
Thread가 너무 많아지면 CPU가 이 Thread들을 전환하는 데 시간을 낭비하게 됩니다     
즉 실제 연산에 필요한 시간보다 Thread를 교체하는 데에 비용을 더 많이 쓰게 되어 효율적이지 못하게 됩니다

## Dispatchers.Main
UI와 interact하는 Thread        
여기서는 IO 연산 혹은 blocking function을 하지 않는 것이 좋습니다       
전체 애플리케이션이 frozen되는 것을 방지하기 위해서

기본적으로 코루틴은 비동기 실행입니다. 코루틴이 실행되면 작업 큐에 등록되고, 등록된 것을 실행할 시간이 되면 코루틴이 실행되는 식으로 동작합니다     

만약 즉각적으로 처리하고 싶은 것이 있다면 `Dispatchers.Main.Immediate`를 사용합니다   

`Dispatchers.Main.Immediate`를 사용하면 작업 큐에 등록하지 않고 동기적으로 바로 실행됩니다

Dispatcher는 `isDispatchNeeded()`함수를 제공하여 Dispatch가 발생해야 하는 여부를 결정하게 되는데, 여기서 Dispatch를 건너 뛰게 되면 작업 큐에 등록되지 않고 바로 실행되는 것입니다     
Disaptchers.Main.Immediate는 Main Thread에 있는 동안 isDispatchNeeded()에 대해서 false를 반환하기에 바로 실행되는 것입니다

`ViewModelScope`와 `LifecycleScope`는 기본적으로 Default가 `Dispatchers.Main.Immediate`로 되어있습니다

## Dispatchers.Unconfined
해당 Dispathcer는 다른 Dispatcher와 달린 어떤 Thread도 변경하지 않습니다      
이 Dispatcher의 코루틴이 실행되면 시작한 Thread에서 실행되고, resume되면 resume한 Thread에서 실행됩니다

Thread Switching이 없기 때문에 가장 저렴합니다.       
따라서 만약 어디 Thread에서 실행되도 상관 없다면 고려해볼만 하지만, 일반 코드에서는 사용하지 않는 것을 권장하며, 고려해야 할 것이 있다면 사용하지 않아야 합니다.   
사용할 수 있는 경우는 CPU를 쓰지 않고, 특정 Thread에 묶인 공유 데이터도 건드리지 않는 Coroutine일 경우 혹은 나중에 dispatch하는 것이 필요없거나 원하지 않는 부작용을 낳는 코너 케이스가 없는 경우 (어떤 작업을 즉시 해야 할 경우)

Unconfined가 문제가 되는 이유는 dispatch 과정이 넘어가기 때문입니다. 이런 dispatch 과정은 해당 Coroutine이 어떤 Thread에서 작업하게 될 지를 결정하는 단계이지만, 이 단계를 넘어가게 되면서 Unconfined로 지정된 Coroutine은 어떤 Thread에서 작업하게 될 지 추적이 어려워집니다.  
게다가 만약 특정 Thread에 묶인 데이터를 건드릴 경우, race condition이 발생할수도 있고, 허용된 Thread가 아닌 Thread에서 자원에 접근하는 일이 발생할 수 있기 때문입니다.

## newSingleThreadContext
Thread 하나에 Coroutine을 가두는 Dispatcher를 생성하는 함수입니다.