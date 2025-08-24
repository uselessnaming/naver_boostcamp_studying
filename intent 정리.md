# INTENT
Intent란 다른 앱 구성 요소에서 작업을 요청하는 데 사용할 수 있는 메시지 객체다        

일반적으로 3가지의 기본 사용 사례가 있음     

1. 활동 시작
Intent를 startActivity에 전달하여 새로운 Activity를 시작할 수 있다      
단순 Intent를 활용하여 화면 전환도 가능하며, 이 때 Intent에 extra를 추가하여 데이터를 전달할 수 있다        
여기서 활동 완료될 때 결과를 수신하기 위해서는 startActivityForResult()를 호출하고, onActivityResutl() 콜백에서 결과를 별도의 Intent 객체로 수신   

2. 서비스 시작
startService()에 전달하여 일회성 작업을 실행하는 작업을 실행할 수 있다 (파일 다운로드와 같은 것들)   
시작할 서비스를 설명하고 필요한 데이터를 전달한다   
bindService()에 intent를 전달하여 다른 구성 요소에서 서비스에 바인딩할 수 있다   

3. 브로드캐스트 전송
시스템은 시스템이 부팅되거나 기기가 충전을 시작하는 경우와 같은 시스템 이벤트에 대해서 다양한 브로드캐스트를 전송한다   
sendBroadcast() 혹은 sendOrderedBroadcast()에 전달해 다른 앱에 브로드캐스트를 전달할 수 있다

이번 과제에서는 활동 시작, intent를 활용한 activity 전환을 주 목적으로 하여 다른 것들은 다음에 더 공부해보기로 하자

## Intent의 종류
+ 명시적 인텐트
```kotlin
val intent = Intent(this, Service::class.java).apply{ data = Uri.parse(fileUrl) }
startService(intent)
```
위의 코드와 같이 특정 앱의 활동이나 서비스와 같은 특정 앱 구성 요소를 실행하는 데 사용되는 인텐트이다.   

+ 암시적 인텐트
작업 실행이 가능한 기기의 앱을 호출할 수 있는 작업을 지정한다   
사용자가 사용할 앱을 선택할 수 있다   

```kotlin
val sendIntent = Intent().apply{
    action = Intent.ACTION_SEND
    putExtra(Intent.EXTRA_TEXT, textMessage)
    type = "text/plain"
} 
startActivity(sendIntent)
```

## 명시적 vs 암시적
쉽게 생각해서 명시적이란 A 앱을 지정해서 이거해! 라고 직접 지정하는 것이고,   
암시적이란 "이 기능" 수행 가능한 앱들 나와봐! 라고 이해하면 될 듯 하다.

## registerForActivityResult
해당 기능을 사용하여 부모 Activity에서 자식 Activity를 생성하고 콜백 받을 수 있다