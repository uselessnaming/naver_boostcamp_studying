# LazyColumn, LazyRow

현재 아이템만 구성, 배치하는 세로 스크롤 리스트이다.    
LazyListScope에서 item을 사용해 단일 아이템을 지정하고, items를 사용해 아이템 list를 추가할 수 있다

> XML에서 사용하던 RecyclerView와 동일한 동작을 한다고 생각해도 좋다

Lazy의 경우 화면에 보이는 View만 생성하고, 스크롤 시 화면 밖으로 사라지는 View는 삭제하고 새로 생성하는 방식으로 많은 수의 항목을 효율적으로 보여누는 방식이다.

Lazy인 이유는 필요할 때 지연(Lazy) 방식으로 필요한 항목을 생성해 성능을 최적화하기 때문에 Lazy 키워드를 넣었다



### state
LazyListState 형이며, 리스트 상태를 제어하고, 관찰할 떄 사용하는 상태 값이다.   
offset을 가져오는 property, 드래그 이벤트를 가져오는 property 등을 사용할 수 있다

### contentPadding
PaddingValues 형이고, 내부 padding 값 조절에 사용한다

### reverseLayout
Boolean 값으로 true면 아이템을 역순 배치한다

### flingBehavior
플링 동작을 설명하는 로직

> fling: 스크롤 가능한 디자인에서 사용자가 손가락으로 화면을 빠르게 튕기면 속도와 방향에 따라 객체가 일정 시간동안 마찰력을 받아 점차 느려지다 멈추는 이벤트

### userControlEnabled
Boolean 값으로 사용자의 제스처 혹은 접근성 작업을 통해 스크롤이 허용되는 지에 대한 여부를 말한다.   
false여도 상태를 사용해 프로그래밍 방식으로 스크롤 할 수 있다


## 효율적인 Lazy 설정
### 복잡한 연산 위치
```kotlin
LazyColumn(){
    items(list.sorted()){ item ->
        Text(item.name)
    }
}
```
위오 같은 코드가 있다고 하자        

과연 효율적일까?

답은 NO이다!        

LazyColumn은 지연 방식으로, 화면에 보이는 새로운 View를 생성하고, 사라지는 View는 삭제하는데, 이런 작업이 진행되면서 계속 sorted 연산을 해야 하므로 성능이 좋지 못하다

그래서 이런 코드는
```kotlin
val sortedList = remember(contracts, comparator){
    contracts.sortedWith(comparator)
}

LazyColun(){
    items(sortedList){ item ->
        Text(item.name)
    }
}
```
이렇게 변경해주는 것이 좋다

> 실제로는 list의 정렬은 state를 관리하는 곳에서 정리하고, UI는 보여주기만 하는 것이 가장 best

### Key 사용
Lazy에서 item의 요소로 key를 받을 수 있다   
이 key를 사용하면 불필요한 recomposition을 막는 역할을 한다
key 타입은 any지만, bundle에 담을 수 있는 primitive type 혹은 Parcelable, Enum을 사용해야 한다

해당 key는 마치 DB의 primary key 처럼 Lazy 안에서 unique해야 한다

key가 없다면 item의 position이 key로 사용된다
이 경우 list에 특정 아이템이 추가/삭제되는 경우, 해당 아이템 이후 item들에 대해서 composition tree가 재생성되어야 하므로 recomposition의 부담이 크게 된다

여기서 구분 가능한 unique 키를 지정하여 item list의 중간 값 혹은 처음/마지막 값 등 어떤 리스트가 삭제/추가 되어오 기존 composition tree를 유지하되 변경된 item을 tree에 추가 혹은 삭제할 수 있다

### ContentType 사용
여러 다른 형태를 갖는 item을 리스트에 표시하는 경우, 각 item의 type을 지정하면 LazyList 혹은 Grid에서 성능을 높일 수 있다   

만약 서로 다른 형태의 A, B type의 item들이 번갈아서 연속될 경우, Compose는 구조가 다른 type임을 인지하고 composition 재사용을 시도하지 않는다.

> 아! 동일한 구조의 경우 똑같은 구조니까 값만 바꿔서 줘야지 ~ 하고 재사용한다고 함

하지만 만약 여러 타입을 사용하지만 ContentType이 없는 경우?         
재사용이 되지 않아서 Composable 트리를 재생성해야 하는데, 이런 과정이 반복되면 불필요한 Recomposition과 할당으로 인해 성능이 안좋아진다

```kotlin
LazyColumn{
    items(
        contentType = { item ->
            when(item){
                is A -> "TYPE_A"
                is B -> "TYPE_B"
            }
        }
    )
}{ item ->
    if (item is A){
        typeAItem(item.text)
    } else if(item is B) {
        typeBItem(item.text, onClick = {})
    }
}
```
위 코드와 같이 사용할 떄 A에서 B로 바뀌면 당연히 재생성되지만, TYPE_A끼리는 구조가 같다고 보고, 재사용할 수 있다

ContentType이 없다면 아이템의 구조들을 비교하는 것에서 매번 리소스를 많이 사용하게 됨 

### 결과
+ 여러 아이템을 사용한다면 content type을 지정해주기
+ key를 반드시 지정하여 불필요한 Recomposition을 줄이기
+ items를 건네줄 때 복잡한 연산 수행하지 않기