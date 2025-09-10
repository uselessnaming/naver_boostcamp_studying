# TextField
XML에서 사용하는 EditText와 동일한 역할을 한다고 생각하면 된다

[공식 문서](https://developer.android.com/develop/ui/compose/text/user-input?textfield=state-based&hl=ko#manage-tf-state)

## 상속 구조
```
CoreTextField
 | - BasicTextField
     | - OutlinedTextField
     | - TextField
```

내부 구조를 다 보기는 아직 어려워서 대충 보면 CoreTextField는 입력 자체를 관리하는 가장 low level의 composable이다.

그리고 BasicTextField는 Custom할 수 있는 기본 API이며,
OutlinedTextField와 TextField는 이 BasicTextField를 decoration한 것이다

그래서 결과적으로는 OutlinedTextField나 BasicTextField를 사용하다가 요구사항에 의해 필요해지면 그 떄 BasicTextField를 활용하여 decoration하면 된다

## value-based VS state-based
### value-based
일반적으로 많이 사용하는 방법으로, text 입력 값을 직접 상태로 관리하는 방식

```kotlin
var text by remember{mutableStateOf("")}

TextField(
    value = text,
    onValueChange = { text = it }
)
```

여기서 text field에서 입력 값을 직접 관리한다

**문제**        
비동기 작업 시 지연이 발생

외부에서 상태를 변경하기 때문에 직접 상태 관리가 필요하며, 비동기 처리 시 지연이 발생할 수 있다

### state-based
Material3 1.4.0-alpha14 버전에서 사용할 수 있는 형태

TextFieldState 객체를 사용하여 텍스트의 입력 값, 커서 위치, 선택 영역 등 텍스트 필드의 상태를 모두 내부적으로 관리한다

**TextFieldState**      
주요 매개변수는 2가지이다.      
+ initialText: TextField의 기본 내용
+ initialSelection: 커서 혹은 선택 영역이 현재 어디에 위치하는 지에 대한 정보

TextFieldState는 Compose에 속한 클래스이지만, UI와 직접적인 의존성이 존재하지 않고, 입력창의 상태만을 관리하도록 설계되어 있어 ViewModel에서 인스턴스를 생성하여 보관하는 것을 권장한다

ViewModel에서 UI 관련 객체를 넣지 않는 것이 가장 좋지만 TextFieldState는 Compose 내부 상태 관리 시스템만 사용하기 때문에 예외적으로 ViewModel에 포함해도 괜찮다

## TextFieldValue 계산
TextField를 단순히 text를 변경하고 다루는 것이 아니라 커서 위치 조정과 같은 세세한 것들을 조절할 때는 TextFieldValue를 사용한다

> rememberTextFieldState 내부 코드를 확인하면 TextFieldValue를 사용하고 있음을 알 수 있다

### 내가 했던 고민 ?
**배경**)
일반적으로 ViewModel을 사용하고 있다면, 상태를 ViewModel이 소유하게 된다

value-based TextField를 사용한다면
```kotlin
private val _text = mutableStateOf("")
val text: State<String> = _text
```
와 같은 패턴을 많이 사용하고 있을 것이다

'state-based로 변경한다면 이런 _text를 TextFieldValue로 변경해야 할까?'라는 생각이 들었다

**고민**)
그래서 결과적으로는 TextFieldValue로 변경한다면 관리하는 위치를 Data 패키지와 ViewModel 중 어디서 관리해야 할까라는 고민이 생겼다

**시도**)
우선 data class에 적용해봤다        
TextFieldValue는 아까 말했듯, text 값만 가지는 것이 아니라 커서 위치 등 세세한 조절을 할 수 있는 클래스이다     
하지만 Data 레이어에서 이런 UI 정보를 가지고 있을 필요는 없다고 생각하여 Data class에서는 제외했다

그 다음은 VeiwModel에 적용해봤다        
TextFieldValue의 import 문을 확인해보면 `import androidx.compose.ui.text.input.TextFieldValue` 이렇다

androidx.compose의 UI 라이브러리를 받아오고 있었다

이런 Compose UI 에서 받아오는 클래스를 ViewModel에서 사용해도 괜찮을까?라는 의문이 생겼다.

**결론**)
[공식 문서](https://developer.android.com/develop/ui/compose/text/migrate-state-based?hl=ko#state-based_3)를 찾아보니 이런 고민에 대해 설명하는 부분이 있었다

공식 문서에서 설명하기를, 
```
UI 종속 항목을 ViewModel에 배치하지 않는 것은 좋은 선택이다.
일반적으로는 좋은 방법이지만 잘못 해석될 여지가 있다
특히 순전히 데이터 구조인 TextFieldState와 같이 UI를 포함하지 않는 Compose 종속 항목들은 사용해도 괜찮다
`MutableState`와 `TextFieldState`같은 경우는 Compose의 스냅샷 상태 시스템으로 지원되고 있는 간단한 상태 홀더이므로 사용하는 것은 문제가 되지 않다
```
라고 하고 있다

즉 결론적으로 말하면, TextFieldState가 비록 Compose의 UI 라이브러리 내에 위치하고 있으나, UI와 관련없는 순수 클래스 상태이므로 사용하지 않을 이유가 없다는 것이다.