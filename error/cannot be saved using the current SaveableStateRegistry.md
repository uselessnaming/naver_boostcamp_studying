<img width="1596" height="87" alt="image" src="https://github.com/user-attachments/assets/47c75a04-e880-4031-b335-70e5a10f2241" />

### java.lang.IllegalArgumentException: com.test.studymate.ui.screens.newplan.NewPlanStateHolder@51f0da cannot be saved using the current SaveableStateRegistry. The default implementation only supports types which can be stored inside the Bundle. Please consider implementing a custom Saver for this class and pass it to rememberSaveable().

Custom Class가 rememberSaveable에 저장될 때 발생하는 오류이다.

rememberSaveable은 savedInstancState에 데이터를 저장하게 되는데, bundle에 저장한다

### 해결
하지만 Custom Class가 Bundle에 저장할 수 없는 구조이기 때문에 직렬화 및 역직렬화 할 수 있는 구조로 해야 한다

> Serializable 혹은 Parcelable 객체를 사용하도록 한다
