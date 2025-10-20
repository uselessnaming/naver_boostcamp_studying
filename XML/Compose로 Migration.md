# Compose로 Migration
XML에서 Compose로 Migration하기 위해서는 순차적으로 진행된다

1. XML에서 사용하는 내부 코드들을 점차적으로 Compose로 변환한다
2. Compose로 migration이 전부 끝나면 navigation 또한 Jetpack Compose Navigation으로 이전한다

## ViewCompositionStrategy
`ComposeView`라는 브리지 View를 활용하여 XML에서 일부 코드를 Compose로 구현하여 처리한다

`ComposeView`의 `setViewCompositionStrategy()` 함수를 사용하여 View에서 해당 Compose를 언제 dispose 해줄지를 정한다     

Compose와 관련된 UI에서 상태를 관리할 때 `SlotTable`과 같은 클래스에서 상태 정보를 내부적으로 가지게 된다       
이 때 dispose를 통해서 이런 데이터를 언제 비워줄 지를 정하는 전략이다

종류에는 4개가 존재한다

### DisposeOnDetachedFromWindow
이는 ComposeView의 기본적인 전략으로 View가 window에서 분리될 때마다 dispose하는 전략이다       
사용자가 `AbstractComposeView.createComposition()`를 별도로 호출하지 않는 한 안전한 리소스 정리를 보장한다      
Composition이 dispose되고 난 후에는 GC가 남은 메모리를 자동으로 처리하기에 신경쓰지 않아도 괜찮다

단 `AbstractComposeView.createComposition()`을 호출한 경우, 그 View가 나중에 attach되지 않고 파기된다면 개발자가 직접 `disposeComposition()`을 명시적으로 호출해야 메모리 누수를 막을 수 있다

### DisposeOnDetachedFromWindowOrReleasedFromPool
해당 전략은 RecyclerView와 같은 재사용 가능성이 있는 View에서 사용하는 전략이다.        
만약 RecyclerView의 ViewHolder와 같이 재사용을 해야하는 View에서 `DisposeOnDetachedFromWindow()`를 사용하게 되면 ViewHolder가 View 범위 밖으로 나가게 되어 소멸하게 되면 detached 되어버려 다시 사용할 때 `setContent{}`를 재호출해야 하기 때문에 비효율적이다

따라서 이와 같은 `pooling container`를 사용할 떄는 이 전략을 사용하도록 한다

### DisposeOnLifecycleDestroyed
해당 전략은 `Activity`나 `Fragment`처럼 `lifecycle`이 `destroyed`가 될 때 해제하는 전략이다.        
따라서 Fragment의 View가 파괴될 때 dispose되므로 Fragment에 View가 재할당 시 중복 생성을 방지할 수 있다

이 전략은 다른 전략과 달리 object가 아닌 class로 선언되었다     
왜?     

런타임 내에 다른 `lifecycle` 인스턴스를 받아야 하기 때문에 class로 선언된 것이다

### DisposeOnViewTreeLifecycleDestroyed
해당 전략은 Tree 내에서 가장 가까운 lifecycle을 찾고, 그 lifecycle에 따라 destoryed되면 자동적으로 dispose 해주는 전략이다.      
즉 명시적으로 Lifecycle을 전달하지 않아도 View 계층 내에서 가장 가까운 lifecycle을 자동으로 연결해주는 전략이야.        
따라서 Fragment에서 view의 lifecycle을 따르는 ComposeView의 경우에 사용하게 되는 전략이야