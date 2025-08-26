# Remember와 RememberSaveable
두 함수 모두 상태 유지를 도와주는 함수이다.

### remember
+ Composable 함수가 Recomposition 될 때 값이 유지됨
+ Activity 재생성 시 값 초기화
+ 컴포저블 lifecycle 내에서만 유지되는 값

### rememberSaveable
+ Activity가 재생성되어도 값을 유지할 수 있도록 도와줌
+ SavedInstanceState를 활용해 값을 저장ㅎ나다
+ Saver를 활용하면 List나 Map같은 구조도 유지가 가능하다

### 차이 
**remember** Composable 생명 주기 내에서 값 유지가 필요할 때

**rememberSaveable**    
Activity 재생성에도 값을 유지해야 하는 경우
