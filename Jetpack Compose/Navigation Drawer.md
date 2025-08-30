# Navigation Drawer
이 컴포넌트는 drawer로, 서랍과 같다

Component 측면에서 나오는 컴포넌트로 열었다 닫았다 하는 서랍! 이라고 할 수 있다

[공식 문서](https://m3.material.io/components/navigation-drawer/overview)

## Specs
<img width="832" height="401" alt="image" src="https://github.com/user-attachments/assets/c11c0a6d-f65d-4666-9ce7-78fd6899270b" />      

1. Container 
2. Headline
3. Label text
4. icon
5. Active indicator
6. Badge label text

## Guidelines
### 사용 권장 경우
+ 5개 이상의 top-level destination을 가진 App
+ 2개 이상의 navigation 수직 구조를 가진 App
+ 관계 없는 destination 간 빠른 이동
+ 큰 화면에서 Navigation Rail 혹은 Navigation bar을 대체할 때

### 사용을 지양하는 경우
+ 다른 navigation component와 함께 쓰지 않도록 하기 (navigation bar 같은 것)

### Standard navigation drawer VS. Modal navigation drawer

**Modal**   
화면 위에 겹쳐서 뜨는 형태      
content는 screem(반투명 배경)으로 가려져 사용자 입력 불가       
Drawer가 열려있다면 Drawer에만 집중하도록 함        

When?       
Compact / Medium 화면 크기 (폰, 작은 tablet)

**Standard**       
화면 옆에 고정 혹은 슬라이드 가능       
Modal처럼 뒤를 막지 않고 앱 content와 공존이 가능       

When?   
Large / Extra-large 화면 크기       
화면 공간이 넉넉해 Drawer를 표시 상태로 둬도 문제가 없음

## 사용 방법
### 동작 제어
```DrawerState``` : drawer의 close, open을 제어

이 때 open 및 close 함수는 suspend 함수로, CoroutineScope를 지정해야 한다

> rememberCoroutineScope를 활용해 인스턴스화 할 수 있다

