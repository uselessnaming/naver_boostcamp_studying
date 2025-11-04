# CompositionLocalProvider
UI Tree에서 범위 별로 변수를 지정해 줄 수 있도록 하는 기능이다.

[공식 문서](https://developer.android.com/develop/ui/compose/compositionlocal)

Compose에서는 Component들이 Tree 형태로 구성된다. 여기서 `CompositionalLocalProvider`로 provide하게 되면 provide하는 지점보다 아래에 위치한 component, 즉 자식 component들이 이에 접근하여 사용할 수 있게 된다. 이 종류에는 `staticCompositionLocalOf`와 `compositionLocalOf` 두 가지가 있다.
`staticCompositionLocalOf`는 고정 값으로 사용하는 것이 좋다. 변경이 적어서 관리가 필요하지 않은 경우에는 이를 사용한다. 이 `staticCompositionLocalOf`를 사용하면 값 변경 시 하위에 있는 Component들이 모두 Recomposition이 일어난다. 만약 값을 변경해야 하는 경우가 생긴다면 `compositionalLocalOf`을 사용하여 이를 소비하고 있는 Component들만 재생성하도록 하는 것이 좋다.

## 사용
내가 정의한 Custom Class (Color) 를 하위에 제공하고자 할 때
```kotlin
data class CustomColor(
    val primary: Color
)

val lightColorScheme = CustomColor(
    primary = Color.Red
)

val LocalExtendedColors = staticCompositionLocalOf<CustomColor> {error("No ExtendedColor Error")}

CompositionLocalProvider(
    LocalExtendedColors provides lightColorScheme
) {
    content()
}
```
이와 같이 제공할 수 있다.

```kotlin
@Composable
fun MyCard() {
    Column{
        Text(
            text = "test",
            color = LocalExtendedColors.current.primary
        )
    }
}
```
이와 같이 접근이 가능하다