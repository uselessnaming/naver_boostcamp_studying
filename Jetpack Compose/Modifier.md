# Modifier

## 제약 조건에 영향을 미치는 수정자
### size 수정자
size는 전달된 값에 맞게 수신되는 제약 조건을 조정한다.

만약 너비와 높이가 가장 작은 제약 조건 경계보다 작거나 가장 큰 제약 조건 경계보다 큰 경우 수정자는 전달된 제약 조건을 최대한 가깝게 일치시키면서도 다음에 전다로디는 제약 조건을 준수한다

> size 수정자는 연결하게 될 경우 올바르게 동작하지 않습니다.

### requiredSize 수정자
노드가 수신된 제약 조건을 재정의해야 할 경우 size를 다시 선언하지 않고, requiredSize 수정자를 통해 재정의한다

크기를 재정의할 떄 더 작아지면 사용 가능 공간 중 가운데에 위치하게 된다

### width 및 height 수정자
size 수정자는 제약 조건의 너비와 높이를 모두 조정한다       
width 수정자를 사용하면 고정 너비를 설정하되, 높이는 결정되지 않은 상태로 둘 수 있다

즉 하나만 선택해서 사용할 수 도 있고, height와 width를 다르게 설정하는 것 또한 가능하다

### sizeIn 수정자
너비와 높이의 정확한 최소 및 최대 제약 조건을 설정할 수 있다    
제약 조건을 세밀하게 제어해야 하는 경우 사용한다

### matchParentSize()
Box 크기에 영향을 미치지 않고 상위 Box와 크기가 같아지도록 하는 수정자로, Box 내에서만 사용이 가능합니다.

## Modifier에 중복 선언을 하게 되면 어떻게 될까?
```kotlin
modifier = Modifier
    .width(50.dp)
    .width(300.dp)
```
위 코드를 실행하면, 어떻게 될까?        

첫 width만 동작한다
크기를 지정하는 방법에 대해서 위에 적혀있지만 여러 제약 조건 중 width나 height를 중복을 작성하면 처음 dp만 적용된다

```kotlin
modifier = Modifier.background(color = Color.Blue.copy(alpha = 1f))
    .background(color = Color.Green.copy(alpha = 0.6f))
    .background(color = Color.White.copy(alpha = 0.1f)),
```
위 코드를 실행한다면?
> alpha는 투명도를 나타낸다

이렇게 하면 background가 차곡차곡 쌓여서 원하는 색상과는 다른 중첩된 색상이 나타나게 된다

### 결론
Modifier에서 중복을 처리할 때 체이닝이다보니 background와 같이 그리는 부분들이나 clickable과 같은 속성들은 중복되어 쌓이게 된다

다만 크기 제약자의 경우에 어떤 기준으로 정해지는 지는 정확하지 않지만 width, height / fillMaxWidth, fillMaxHeight의 경우에는 가장 처음 선언된 부분에 대해서 적용되는 것이다.

> 이유를 알아보면 좋을 것 같은데 자료를 마땅히 찾지 못헀다..