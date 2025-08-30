# Compose의 단계

Compose에는 크게 3개의 단계로 나눠집니다.

<img width="1348" height="148" alt="image" src="https://github.com/user-attachments/assets/1e6bfa71-6b5b-4d97-8891-d8e8436bd62a" />

### Composition
표시할 UI로 Compose는 구성 가능한 함수를 실행하고 UI 설명을 만든다      

Compose 런타임은 @Composable 함수를 실행하고 UI를 나타내는 트리 구조를 출력한다

코드의 각 @Composable 함수는 UI 트리의 단일 레이아웃 노드에 Mapping된다

### Layout
UI를 배치할 위치로 측정과 배치라는 두 단계로 구성된다    
Layout 요소는 Layout 트리에 있는 각 노드의 Laout요소 및 모든 하위 요소를 2D 좌표로 측정하고 배치

Composition 단계에서 생성된 UI 트리를 사용하며, 각 Layout 노드는 2D 공간에서 크기와 위치를 결정하는 데 필요한 모든 정볼ㄹ 포함한다

+ 하위 요소 측정: 노드가 하위 요소를 측정
+ 자체 크기 결정: 이런 측정치를 기반으로 노드가 자체 크기를 결정
+ 하위 요소 배치: 각 하위 노드는 노드의 자체 위치를 기준으로 배치

> 이 단계 종료 시 각 레이아웃 노드는 할당된 width, height와 x, y 좌표를 얻는다

<img width="1014" height="506" alt="image" src="https://github.com/user-attachments/assets/7ffa46b1-e7a6-4ea8-b903-8c95e54f88a8" />

위와 같은 경우 과정은 다음과 같다
1. Row는 하위 요소인 Image 및 Column을 측정
2. Image가 측정 / 하위 요소가 없으므로 자체 크기를 결정하고 이를 Row에 다시 보고
3. 다음으로 Column이 측정되며, 먼저 자체 하위 요소를 측정
4. 첫 번째 Text가 측정된다. 하위 요소가 없기 때문에 자체 크기를 측정하고 Column에 다시 보고 (두번째 Text도 동일 반복)
5. Column은 하위 측정 값을 사용해 자체 크기를 결정한다. 최대 하위 요소 너비와 하위 요소의 높이 합계를 사용한다
6. Column은 하위 요소를 자신에 상대적으로 배치해 서로 아래에 배치
7. Row는 하위 측정 값을 사용해 자체 크기를 결정 / 최대 하위 요소 높이와 하위 요소 너비의 합계를 사용
8. 자식 배치

### Drawing
UI를 렌더링 하는 방법으로 UI 요소는 일반적으로 기기 화면인 캔버스에 그려진다

일반적으로는 단계의 순서가 동일하다     
> 여기서 BoxWithConstraints, LazyColumn, LazyRow는 중요한 예외로, 하위 요소의 컴포지션이 상위 요소의 Layout 단계에 따라 달라진다

Layout 단계의 예제를 동일하게 사용한다고 할 시
1. Row는 배경 색상과 같은 콘텐츠를 그림
2. Image가 자체적으로 그림
3. Column이 자체적으로 그림
4. 각 Text는 자체를 그림

### 상태 읽기 최적화
Compose는 상태를 적절한 단계에서 읽도록 추적하여 불필요한 작업을 줄이도록 한다

```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        // Non-optimal implementation!
        Modifier.offset(
            with(LocalDensity.current) {
                // State read of firstVisibleItemScrollOffset in composition
                (listState.firstVisibleItemScrollOffset / 2).toDp()
            }
        )
    )

    LazyColumn(state = listState) {
        // ...
    }
}
```
위와 같은 코드가 있다고 해보자      
> 해당 코드는 오프셋 수정자로 최종 레이아웃 위치를 오프셋하여 사용자가 스크롤 시 시차 효과가 발생하는 Image다

여기서 firstVisibleItemScrollOffset 상태 값을 Composition 단계에서 읽어 Modifier.offset()에 전달하게 된다       
스크롤할 때마다 전체 UI가 Recomposition된다.        
그 결과 불필요한 측정과 배치가 발생해 성능이 저하될 수 있다

이를 방지하기 위해서 상태 값을 레이아웃 단계에서 읽도록 수정해야 한다       
Modifier.offset(offset: Density.() -> IntOffset)과 같은 람다 기반 오프셋 수정자를 사용하면, Layout단계만 재시작되도록 최적화할 수 있다.

```kotlin
Box {
    val listState = rememberLazyListState()

    Image(
        // ...
        Modifier.offset {
            // State read of firstVisibleItemScrollOffset in Layout
            IntOffset(x = 0, y = listState.firstVisibleItemScrollOffset / 2)
        }
    )

    LazyColumn(state = listState) {
        // ...  
    }
}
```
수정자의 람다 블록이 레이아웃 단계에서 실행되어 firstVisibleItemScrollOffset 상태를 컴포지션 중에 읽지 않기 때문에 효율적이다.      
따라서 상태 변경 시 Layout과 Drawing 단계만 재시작 된다

람다 매개변수를 사용하는 것은 약간의 비용이 들지만, 상태 읽기를 Layout 단계로 제한하여 불필요한 Recomposition을 방지하는 이점이 더 크다

> 기존에서는 listState.firstVisibleItemScrollOffset을 즉시 계산하여 IntOffset으로 넣고 있으며 이 과정은 Composition 시점에 해당하므로 Composable이 이 상태를 구독하게 된다 (즉 스크롤마다 Composition 단계부터 재구성된다)
>
> 하지만 람다 사용시, 람다 자체를 modifier에 전달하기 때문에 Compose는 이 람다를 저장하고 Layout 단계에서 호출하여 사용하게 된다 (즉 Layout 엔진이 해당 상태를 읽게 되어 Layout단계만 다시 실행되어 위치 계산부터 시작하게 됨) 

**최적화를 위해서는 Recomposition을 최소화하는 방법을 고려해야 한다**

