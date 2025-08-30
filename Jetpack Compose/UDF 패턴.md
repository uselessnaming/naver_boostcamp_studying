# UDF 패턴 (Undirectional Data Flow)
> 왜 Undirectional일까.. 이건 단방향 데이터 흐름이 아니라 무방향 데이터 흐름 아닌감?

이는 Compose에서 말하는 데이터 흐름이다     
상태는 아래로, 이벤트는 위로 이동하는 디자인 패턴이다       
단방향 데이터 흐름을 따라 UI에 상태를 표시하는 composable를 저장하고 변경하는 앱 부분을 서로 분리할 수 있다

## 흐름
1. 이벤트 : UI의 일부가 이벤트를 생성하여 위쪽으로 전달하거나 앱의 다른 레이어에서 이벤트가 전달됨
2. 상태 업데이트 : 이벤트 핸들러가 상태를 변경할 수 있다
3. 상태 표시 : 상태 홀더가 상태를 아래로 전달하고 UI가 상태를 표시한다

<img width="429" height="527" alt="image" src="https://github.com/user-attachments/assets/5d419762-c8b2-4af8-a047-e8fe082c1d98" />

단방향 데이터 흐름도        

## 장점?
+ 테스트 가능성: 상태와 상태를 표시하는 UI를 분리해 격리 상태에서 더 쉽게 테스트할 수 있다
+ 상태 캡슐화: 상태는 한 곳에서만 업데이트할 수 있고 composable의 상태에 관한 정보 소스가 하나 뿐이므로 일관되지 않은 상태로 인해 버그를 만들 가능성이 작다
* UI 일관성: 관찰 가능한 상태 홀더(StateFlow 혹은 LiveData)를 사용함으로써 모든 상태 업데이트가 UI에 즉시 반영

## Composable의 매개 변수
재사용성 혹은 분리를 유도할 때 각 composable에는 가능한 한 최소한의 정보를 포함해야 한다        

```kotlin
@Composable
fun Header(title: String, subtitle: String) {
    // Recomposes when title or subtitle have changed.
}

@Composable
fun Header(news: News) {
    // Recomposes when a new instance of News is passed in.
}
```

이렇게 되어있다고 하자!     
news 클래스에 속성 값이 title, subtitle만 있는 것이 아니라고 하자       
그러면 Header와 관련 없는 데이터를 갱신하여 업데이트해도 Header가 recomposition된다     

하지만 매개변수의 수가 너무 많다면 함수의 인체공학적 특성이 감소하므로 이 경우 매개변수를 클래스로 그룹화하는 것이 좋다
