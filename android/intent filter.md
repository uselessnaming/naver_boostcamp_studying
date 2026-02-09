# Intent Filter
intent-filter는 Activity가 어떤 intent를 받을 수 있는지, 어떤 진입점이 될 수 있는지를 시스템에 알려주는 선언부

기본 생성되는 Activity의 Manifest intent-filter

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

action.MAIN의 경우 해당 Activity가 앱의 메인 진입점이라는 의미로, 앱 시작 시 처음 열릴 수 있는 화면을 의미한다.

category.LAUNCHER의 경우 앱 아이콘을 눌렀을 때 이 Activity를 띄워도 된다는 허용의 의미다.

Intent-filter를 활용하여 암묵적 Intent로 실행이 가능하고, 외부 앱 또한 exported 속성 값 변경을 통해 실행할 수 있다.  

Intent-filter가 없을 경우에는 명시적인 Intent로만 처리할 수 있다