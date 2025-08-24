# 버튼 View에서 발생하는 오류

### Buttons in button bars should be borderless
use style="?android:attr/buttonBarButtonStyle"  
이라는 오류가 생성됨

이는 안드로이드의 UI/UX 가이드라인에 의거하는 것으로, 강조될 필요 없는 곳에서는 borderless button 을 사용하는 것을 권장하고 있다

주요 Action Button의 경우에는 Raised Button을 사용      
그 외 Secondary Button의 경우 borderless를 권장