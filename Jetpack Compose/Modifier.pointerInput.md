# PointerInput
Modifier의 확장함수 중 하나로, 여러 상호작용에 대한 처리를 위해 사용한다.       
`pointerInput` 확장 함수는 
```kotlin
fun Modifier.pointerInput(key1: Any?, block: PointerInputEventHandler): Modifier =
    this then SuspendPointerInputElement(key1 = key1, pointerInputEventHandler = block)
```
위와 같이 구성되어 있다.        

`SuspendPointerInputElement` 타입의 내부로 들어가보면 이 클래스는 `ModifierNodeElement<SuspendingPointerInputModifierNodeImpl>`을 상속받고 있다     

따라서 최종적으로 `SuspendingPointerInputModifierNodeImpl` 클래스로 들어가보면 `pointerInputJob`이라는 Job 객체를 보유하고 있다. 이는 `PointerEvent`에 따른 동작을 처리하기 위한 Job이다. `onPointEvent()` 함수를 확인해보면 null일 경우에 새로운 CoroutineScope의 launch 블록 내부에서 동작을 실행하고 이렇게 생성된 Job 객체를 `pointerInputJob`에 할당하는 것을 확인할 수 있다.      
또한 할당할 때 
```kotlin
private val pointerHandlersLock = makeSynchronizedObject(pointerHandlers)
``` 
lock을 활용해 해당 이벤트가 처리되는 동안 새로운 이벤트가 등록되거나 기존의 이벤트가 제거되지 않게 lock하는 작업이 포함되어 있다     