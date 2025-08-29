# Surface
영어 이름을 보면 알 수 있듯이 표면, 즉 껍데기의 의미이다.       

UI를 그리는 Layer라고 생각을 해도 된다

```kotlin
modifier: Modifier = Modifier,
shape: Shape = RectangleShape,
color: Color = MaterialTheme.colorScheme.surface,
contentColor: Color = contentColorFor(color),
tonalElevation: Dp = 0.dp,
shadowElevation: Dp = 0.dp,
border: BorderStroke? = null,
content: @Composable () -> Unit
```
이런 파라미터 값들을 가지고 있다        

파라미터 값을 보면 알 수 있듯이     
모양, 색상, 내부 색상, elevation, 테두리 의 값을 지정할 수 있다     

**Q 이거 왜 써야 할까?**
알아보면서 느낀 점은 Surface를 사용하는 것과 Row, Column, Box를 사용해서 구성하는 것과 다른 점이 뭘까?      
결국 Surface의 내부 값을 modifier로 전달하여 처리하는 데, 이러면 기본 Layout들을 사용하고, 여기에 modifier 설정하면 되는 것 아닐까? 하는 의문이 생겼다

## 사용 이유
> 나의 질문과 연결되어 생각하면 좋을 것 같다

Material Design 규칙을 얼마나 자동으로 지켜주는 가! 에 대한 답이다.     

Elevation, 색상 체계, shape, clip, onClick 모두 Surface를 처리하면 기존 Layout보다 훨씬 Material3의 디자인을 따르도록 도와주는 부분이 많다

예를 들면 elevation을 설정하게 되면 기본적으로 modifier에서 설정해야 하는 elevation과 tonalElevation 등을 설정해야 한다. 그리고 다크 모드면 살짝 밝게 보이도록 overlay를 준다.      

즉 Surface를 적절히 사용하면, Material Design이 제공하는 일관성을 UI에 적용하기 쉽고, Configuration Change에 대한 대응과 같은 부분에서도 상당히 편리하다