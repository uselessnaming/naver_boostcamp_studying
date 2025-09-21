# ViewModel에서 init 블록을 통한 상태 초기화

[참고 블로그(Launched Effect vs ViewModel)](https://proandroiddev.com/loading-initial-data-in-launchedeffect-vs-viewmodel-f1747c20ce62)

대부분의 프로젝트, 심지어 안드로이드 코드랩과 같은 것들을 봐도 view model 내에서 init 블록을 사용하고 있다      

이것이 뭔가 잘못된 부분이 있을까? 에 대한 정리다        

우선 ViewModel에서 init으로 데이터를 불러오는 등 상태를 초기화하게 되면 어떤 문제가 있는지 알아보자

## View Model init 시 발생하는 문제점
### ViewModel 생성과 강한 응집도
데이터를 가져오는 타이밍이 ViewModel의 생성자에 포함된다.       
따라서 새로고침과 같은 사용자 상호작용이나 기타 이벤트를 통한 데이터 로딩 시점 제어가 어렵다

### 테스트 어려움
ViewModel이 생성되자마자 데이터 로드가 시작된다.        
이렇게 되면 특정 기능 단위로 Test를 진행할 때 반드시 Init에 있는 코드가 실행되면서 단독 테스트가 불가능하고, 이와 관련되어 설정하는 것이 복잡해진다.        
따라서 View Model init 블록 내에서 상태를 초기화하면 테스트에서 어려움이 생긴다

### 리소스 낭비의 가능성
앱이나 화면에서 진입과 동시에 데이터를 필요로 하는 경우가 아니라면, 쓸모없는 데이터를 계속 들고 있어야 하므로 리소스가 낭비된다

### UI 응답성
init 블록은 최대한 가벼워야 한다        
여기서 메인 thread를 차단하는 작업이 들어간다면 UI 응답성에 문제가 발생할 수 있다

**주의**        
왠만하면 사용하지 않는 것이 좋지만, 흐름상 사용해야만 한다면 사용해도 된다

## 해결
`Lazy Observing` 키워드     
핵심 개념은 init 시 데이터를 들게 하는 것이 아니라 Flow나 LiveData를 관측할 때 혹은 수집할 때 데이터를 발생시키는 것이다.       
Flow로 관리한다면 init 블록 내에서 데이터 흐름을 받아서 