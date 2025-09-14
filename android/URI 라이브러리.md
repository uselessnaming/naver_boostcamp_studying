# URI 라이브러리
Android에서는 uri를 정의하는 2가지 방법이 있다

Android 는 Comparable<Uri>, Parcelabel을 상속받고,
Java는 Comparable<URI>, Serializable을 상속받는다

두 개 중 더 좋은 방법은 명확히 정해져있지 않다      
다만 파싱 시 엄격함이 조금 다르며, Java의 URI는 쿼리 파라미터 값을 직접 받아오는 메소드가 없다      
반면 android의 Uri는 구형 안드로이드 버전에서 IPv6 주소를 파싱할 수 없다

> Android에서 일반적으로 Parcelable이 속도가 더 빠른 것으로 알고 있는데 그러면 Android의 Uri를 사용하는 것이 더 좋지 않나?
> 근데 unit test에서는 안드로이드 프레임워크가 별도로 존재하지 않기 때문에 시스템에 의존하는 객체는 Mocking 작업이 별도로 필요하다