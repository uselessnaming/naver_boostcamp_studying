# @res
해당 어노테이션은 메소드, 지역 변수 혹은 필드가 특정 타입의 resource id 임을 나타내는 데 사용한다

## 종류
+ @StringRes: 해당 필드나 변수가 문자열 resource id이다
+ @ColorRes: 해당 필드나 변수가 색 resource id이다
+ @DrawableRes: 해당 필드나 변수가 drawable resource id이다
+ @AnimatorRes: 해당 필드나 변수가 애니메이터 resource id이다
+ @LayoutRes: 해당 필드나 변수가 레이아웃 resource id이다
+ @FontRes: 해당 필드나 변수가 폰트 resource id이다
+ @AnyRes: 해당 필드나 변수가 어떤 타입의 resource id든 올 수 있다

## 장점
해당 어노테이션을 붙이게 되면 컴파일 시 리소스 타입이 잘못 들어가는 것을 감지하여 오류를 방지할 수 있다     
특히나 이럴 경우 Int 값이라서, 아무 int 값이 들어갈 수 있는데, 이런 것을 제한할 수 있다     

가독성 및 의도를 명확히 표현할 수 있다

## 결론
id 값을 파라미터로 사용한다면 @Res 어노테이션을 활용하여 명확하게 제한하자