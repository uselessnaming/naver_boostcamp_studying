# Sticky Header
## Sticky Header란?
리스트 항목을 스크롤할 때 특정 헤더가 화면 상단에 고정되도록 하는 기능

긴 리스트에서 섹션별로 구분되는 항목을 쉽게 구분할 수 있도록 하는 UI 요소

## 사용 예시
+ 섹션 타이틀 고정: 각 카테고리나 날짜별로 그룹화된 항목들을 구분할 때 사용
+ 스크롤 내비게이션: 긴 리스트에서 여러 섹션을 구분해 사용자에게 목록의 흐름을 명확히 안내

## 구현 예시
Lazy Scope에서는 Sticky Header를 사용해 손쉽게 구현할 수 있다

```kotlin
@Composable
fun StickyHeaderComposable(){
    val sections = listOf(
        "Category1" to listOf("a", "b", "c" ... )
        "Category2" to listOf("A", "B", "C" ... )
        ...
    )

    LazyColumn{
        sections.forEach{ (category, items) ->
            stickyHeader{
                SectionHeader(category)
            }
            items(items){ item ->
                ItemContent(item)
            }
        }
    }
}
```
위와 같이 사용할 수 있다

## 구성
sticky header의 내부 구현을 보면
```kotlin
fun stickyHeader(
    key: Any? = null,
    contentType: Any? = null,
    content: @Composable LazyItemScope.(Int) -> Unit
){
    item(key, contentType){ content.invoke(this, 0) }
}
```
으로 구현돼 있다        

`content.invoke(this, 0)`이 부분이 어떻게 동작하는 지는 모르겠으나, item으로 처리하는 것을 볼 수 있다       
즉 sticky header도 엄밀히 따지면 item 블록 내에서 선언되는 구성요소이다

**Q 위치 고정을 어떻게 하는거지? 내부 코드에서도 따로 보이지 않는 것 같은데?**      
> `content.invoke(this, 0)` 코드로 위치 제어를 하는 것 같은데 어떻게 알 수 있을까