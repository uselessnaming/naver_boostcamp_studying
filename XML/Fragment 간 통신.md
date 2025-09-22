# Communicates with Fragments
Fragment 간 데이터를 공유하는 방법

## Arguments를 통한 데이터 전달
```kotlin
val bundle = Bundle()
bundle.putString("key", "value")

val newFragment = NewFragment()
newFragment.argumens = bundle

parentFragmentManager.beginTransaction()
    .replace(R.id.fragment_container_view, newFragment)
    .commit()
```
위와 같이 bundle 객체를 생성하여 전달할 데이터를 저장하고, fragment의 arguments에 넣어서 처리한다

Fragment에서 직접 arguments에서 get 함수를 통해 가져올 수 있다

> Primitive 타입의 값이나 간단한 객체 정도를 전달할 떄 사용한다

## ViewModel을 활용한 데이터 공유
Activity에서 공통된 데이터를 처리하는 View Model을 선언하고, 해당 Activity 위에서 선언된 Fragment들에 해당 View Model을 공유하는 방법       

`private val viewModel: ActivityViewModel by activityViewModels()` 와 같이 Fragment에서 선언하여 사용할 수 있다

## Fragment Result API를 활용한 데이터 공유


## Jetpack Navigation을 활용한 데이터 전달
navigation 라이브러리를 사용한다면 View Model의 범위를 대상 NavBackStackEntry의 수명 주기로 지정할 수 있다

jetpack navigation에서는 화면 전환에 필요한 action과 data를 별도로 정의할 수 있다       
따라서 이를 통해 간결하게 데이터를 저달할 수 있다   

다만 데이터 전달만을 위해 navigation을 사용하는 것은 비효율적이다.      
원래 그 역할로 만들어진 것이 아니기 때문에, 이는 나중에 jetpack navigation을 활용할 때 부가 기능으로 사용하는 편이 좋다