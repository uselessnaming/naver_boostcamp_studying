# 스레드 주석
Method가 특정 유형의 Thread에서 호출되는 지 확인

+ `@MainThread`: 앱의 수명 주기와 연결된 메소드
+ `@UiThread`: 앱의 View 계층 구조와 연결된 메소드
+ `@WorkerThread`: 백그라운드 스레드
+ `@BinderThread`
+ `@AnyThread`

빌드 도구에서 `@MainThread` 및 `@UiThread` 주석을 교환 가능한 것으로 취급한다       
따라서 `@MainThread`메소드에서 `@UiThread` 메소드를 호출할 수 있고, 반대도 가능하다     
다만 서로 다른 스레드의 뷰가 여러 개 있는 시스템 앱의 경우 UI 스레드가 MAIN 스레드와 다를 가능성이 있긴 하다      