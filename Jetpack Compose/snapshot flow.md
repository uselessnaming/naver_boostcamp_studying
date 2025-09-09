# Snapshot Flow
`SnapshotFlow`는 수집될 때 Block을 실행하고 그 결과를 보내는 Flow를 생성하며, 접근한 스냅샷 상태를 기록한다     
수집이 되는 동안 Block 별로 접근한 상태를 변경하는 새로운 Snapshot이 적용되면, 플로우는 다시 Block을 실행해 접근한 스냅샷 상태를 다시 기록한다      
블록 결과가 이전과 다르면 새로운 결과를 방출한다        

다른 Flow 연산자의 사용으로 인해 명시적으로 취소되거나 제한되지 않는 한 수집은 무기한 계속된다

[공식 문서](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?_gl=1*58uffz*_up*MQ..*_ga*MTAxNDgzNjc1MS4xNzQ1NDE0NjUw*_ga_6HH9YJMN9M*MTc0NTQxNDY1MC4xLjAuMTc0NTQxNDY1MC4wLjAuNDY5NTQ4NzYx#snapshotFlow(kotlin.Function0))

## 이벤트(Event) vs 상태(State)
### Event
순간적으로 발생하며, 발생 시점의 정보만 중요하다        
발생 후에는 사라짐      
반복되는 같은 이벤트라 할지라도 의미가 있다

ex) 버튼 클릭, Toast Message, 네트워크 요청 완료 알림

### State
최신 값만 중요하다      
중간 값은 무시해도 된다     
같은 상태를 여러 번 받아도 동일한 결과를 내야 한다 (idempotent)

## Snapshot Flow 사용 시점
`snapshotFlow`는 Compose State를 Flow처럼 관찰할 때 사용한다        
ComposeState, MutableState, LazyListState 등 스냅샷 기반 상태를 Flow로 변환할 때        
UI 상태 변화에 따라 비동기 연산을 연결하고자 할 때      

## SnapshotFlow vs StateFlow
### StateFlow
항상 최신 상태를 저장       
Compose 외부에서도 사용 가능 (ViewModel / Repository)       
값 변경 시에만 알림 발생        

### SnapshotFlow
Compose의 State를 Flow로 변환       
Compose의 Snapshot 기반 상태를 관찰할 때 사용       
UI에서 상태가 바뀔 때 Flow 이벤트 변환      
Compose 내부에서만 의미가 있다      

## 결론
Snapshot Flow는 State를 Flow로 변환하여 추적할 때 사용

또한 Compose에 의존적이기 때문에 UI 단에서 사용하게 됨

UI 상태에 따라 어떤 비동기 작업을 처리하거나 연산이 필요한 경우 이용하게 됨