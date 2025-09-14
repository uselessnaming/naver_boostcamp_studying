# Activity Result Contracts

다른 앱의 구성 요소로 데이터를 요청하는 경우가 종종 있다        
예를 들면, 사진을 가져온다거나 파일을 가져오는 등의 동작이 그렇다

기존에는 `startActivityForResult(Intent)` 를 사용해 다른 활동으로부터 결과를 받아오고, `onActivityResult()`를 재정의해 받아온 결과에 대해 어떻게 사용할 지를 정의한다

최근 android에서는 `startActivityForResult()`, `onActivityResult()`를 사용하는 대신 androidx의 Activity와 Fragment에 도입된 `Activity Result API`를 사용하는 것을 권장하고 있다


**Q 왜?**       
기존에는 수명 주기 문제로 인해 deprecated 되었다고 한다
/** TODO 어떤 문제? */

[참고 자료](https://developer.android.com/training/basics/intents/result?hl=ko)

## 사용
Activity Result API를 사용하기 위해 `ActivityResultLauncher` 객체를 등록해야 한다       
이 객체를 생성하기 위해서는 `registerForActivityResult()`를 사용해야 한다       

이는 Activity, Fragment에서 사용할 수 있는 메소드이며, 이는 두 가지 매개변수를 갖는다       

### contracts: ActivityResultContracts
이는 다른 Activity를 실행하고 결과를 받기 위한 객체

### callback: ActivityResultCallback
callback의 자리에서는 ActivityResultCallback의 인터페이스를 람다식으로 구현하여 인자로 넘겨준다

> 시스템에서 받아온 result를 통해 어떤 액션을 할 지를 고르는 것이다

## 실행
### Activity Result API를 사용하기 위한 등록 과정
> ActivityResultLauncher 객체 생성

이는 실제 다른 Activity를 실행해 결과 값을 받아와야 한다
> 여기서 다른 Activity는 다른 앱에서의 Activity를 포함한다