# Preview를 위한 구조
Compose에서는 미리보기를 위한 Preview를 지원한다        

Preview에서는 widthDp, heightDp, uiMode 등 다양한 속성을 조절하여 미리보기를 할 수 있다

## 간단한 구조
```kotlin
@Composable
fun Header(){
    Text(text = "Header")
}

@Composable
fun Title(title: String){
    Text(text = title)
}

@Preview
@Composable
fun PreviewScreen(){
    Column{
        Header()
        Title("title")
    }
}
```
위 코드와 같이 선언된 Composable을 Preview에서 선언하는 것으로 미리보기를 확인할 수 있다

## 대규모 데이터 셋
Composable Preview에 대규모 데이터 세트를 전달해야 하는 경우도 꽤나 존재한다.

이럴 때 @PreviewParameter 주석을 사용하여 Composable Preview에 샘플 데이터를 전달할 수 있다     

```kotlin
@Preview
@Composable
fun PreviewScreen(
    @PreviewParameter(UserPreviewParameterProvider::class) user: User
){
    UserProfile(user)
}

class UserPreviewParameterProvider: PreviewParameterProvider<User> {
    override val values = sequenceOf(
        User("A"),
        User("B"),
        User("C"),
    )
}
```
이렇게 작성하게 되면 sequence의 각 항목별로 Preview를 보여주게 된다

## 제한 사항
만능같은 Preview도 안되는 것들이 존재한다       

#### Network에 엑세스 불가
#### 파일 엑세스 불가
#### 일부 Context API를 완전히 사용하지 못할 가능성이 있음

#### ViewModel 사용

여기서 핵심은 View Model 사용이다       

ViewModel을 사용하면 Preview 지원이 되지 않기 때문에 화면을 그릴 때 Preview를 고려한다면 ViewModel을 사용하지 않도록 해야 한다      


하지만 잘 생각해보면 화면을 구성할 때 대부분 ViewModel을 구현하고 있다      

그럼 어떻게 해야할까?

Android Developer에서 권장하는 방법은 ViewModel에서 전달할 인자들을 받는 Composable을 새로 생성하는 것이다      

무슨 말인가 싶지만, 사실 코드를 보면 별거 없다

```kotlin
@Composable
fun AScreen(viewModel: AViewModel = viewModel()){
    AScreen(
        name = viewModel.name,
        user = viewModel.user
    )
}

@Composable
fun AScreen(
    name: String,
    user: User
){
    // UI 구성
}

@Preview
@Composable
fun PreviewAScreen(){
    val testName = ""
    val testUser = User()
    AScreen(
        name = testName,
        user = testUser
    )
}
```
위 구조처럼 ViewModel을 사용하는 Composable과 동일한 내용의 Composable을 갖지만 데이터 타입으로 받아 ViewModel을 확인하지 않아도 됩니다