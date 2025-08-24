# 각 View에 대해 새로운 속성들 정리

## Image View
#### contentDesciprtion
해당 속성은 왜 필요할까?    
해당 속성을 지정하지 않으면 

```kotlin
missing contentDescriprtion
```
ImageView에서 warning을 남긴다      

그렇다고 ImageView의 content decription이 어디서 보일까? 보이지 않는다      
그래서 의미를 찾아봤다      

image view나 image button, check box와 같은 시각적으로 정보를 전달하는 뷰에 대해서 해당 속성을 설정하도록 하는데, 이는 시각장애와 같이 시각 정보를 제대로 처리할 수 없을 경우에 이를 대체할 설명이 필요하여 설정하는 것이다.        

VoiceOver와 같은 TTS가 ImageView를 클릭할 경우에, content description으로 설정한 text를 읽는다고 한다