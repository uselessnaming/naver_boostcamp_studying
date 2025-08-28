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