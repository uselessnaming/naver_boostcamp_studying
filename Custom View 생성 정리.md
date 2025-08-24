# Custom View
알림 목록 관리하는 View를 Custom으로 생성해보기!        

Custom View는 어떻게 만들 수 있을까?      

## 설계
얼핏 학습 했을 때 View를 상속받아 그려주면 되지 않을까?! 라고 단순히 생각했다     
> 역시 실제로 해보면 너무 다르다

우선 View 구조를 생각해봐야 했다        
```
MainFragment
|-- AlarmListView
|    |-- AlarmListItem
```
이렇게 Main Fragment 에 AlarmListView를 생성하고, ListView 내부에 들어갈 Item을 표현해야 한다

## 구현
우선 모두 View를 상속받아 처리하는 것보다 item은 xml로 생성한 후 binding하여 inflate하고, 그 이후로 데이터와 연결해주고자 했다        

그리고 xml로 view를 구성할 때 단순히 하나의 View를 구성하는 것이 아니기 때문에 ViewGroup을 상속받아야 했고, ViewGroup를 상속받는 ConstraintLayout 클래스를 사용했다

AlarmListView는 추가된 데이터를 넘겨받고, 받은 데이터를 기준으로 AlarmListItem을 생성하여 나의 view에 추가하는 동작을 진행해야 한다

#### 생성자
View의 기본적인 생성자는 4개가 존재한다        
1. View(context: Context)
이 생성자는 프로그래밍적으로 View를 생성할 때 호출되는 것으로 코드에서 직접 View를 생성할 떄 사용

2. View(context: Context, attrs: AttributeSet)
XML 레이아웃 파일에서 Custom View를 정의할 때 호출되는 것으로 XML에 정의된 속성들을 처리하기 위해서 사용

3. View(context: Context, attrs: AttributeSet, defStyleAttr: Int)
XML에서 스타일 지정 시 호출되는 생성자로 XML에서 style 속성을 사용해 기본 스타일을 정의할 때 이 생성자가 호출

4. View(context: Context, attrs: AttributeSet, defStyleAttr: Int, defStyleRes: Int)
API 21이상에서 스타일 리소스를 지정할 때 호출되는 것으로 defStyleAttr은 스타일 속성을, defStyleRes는 해당 스타일 리소스 id를 전달


<img width="1714" height="80" alt="image" src="https://github.com/user-attachments/assets/f44db9d2-e319-436e-aa82-309be42916a5" />

단순히 하나의 생성자만 선언하면 문제 없이 동작할 수 있을 것 같지만 사실 xml을 inflate하는 과정에서 맨 마지막 생성자를 제외하고는 모두 실행된다      
따라서 선언하지 않았던 나는 위의 오류를 마주하게 되었다     

CustomView를 생성하면서 3개의 생성자를 별도로 지정한다면 오류를 마주하지 않을 수 있다       

다만 이런 과정이 번거롭고 까다롭기 때문에 사용하는 것이 @JvmOverloads 어노테이션이다       
해당 어노테이션은 생성자 오버로딩을 자동으로 생성해 주는 어노테이션으로 이를 사용하면 생성자 선언을 별도로 하지 않아도 괜찮아서 코드를 줄일 수 있다

해당 어노테이션을 사용하면 inflate 과정에서 생성자로 발생하는 문제를 없앨 수 있다

#### addView 함수
ViewGroup에는 addView함수가 존재한다     
해당 함수는 나의 View에 view를 추가하는 것으로 자식을 낳는다!라고 생각하면 이해가 쉬웠다      

여기서 조심해야 할 것은 alarm 데이터를 전달받았으나 아직 AlarmListItem에 전달되지 않았으니 
```kotlin
val itemView = AlarmItemView(context).apply{
    id = View.generateViewId()
    bind(alarm)
}
```
위의 코드처럼 별도로 구현한 bind 함수를 통해서 데이터를 전달하여 view를 구성해야 한다

그리고 id = View.generateViewId() 와 같이 id를 지정해야 한다     

<img width="961" height="51" alt="image" src="https://github.com/user-attachments/assets/9dc963ad-5f35-498a-9a89-607aea624b22" />


만약 별도 지정을 하지 않는다면 위와 같은 오류를 마주하게 될 것이다      

해당 오류는 constraint layout을 사용할 때, constraint set을 활용하여 제약을 설정하게 된다.      
이 때 id를 지정하지 않으면 어떤 뷰와 연결해야 할 지 알 수 없기 때문에 오류가 발생하게 된다      
View.generateViewId()는 임의로 id 를 붙여주도록 하는 코드이다.      

**그렇다면 XML에서 ConstraintLayout을 사용하며 id 값을 지정하지 않는다면 오류가 발생할까?**

그건 아니다!     
이것에 대해서는 찾아본 결과 XML의 ConstraintLayout을 사용하면 Constraint Set을 사용하지 않고 처리하기 때문에 오류가 발생하지 않는다

즉, 결과적으로 Constraint Set을 활용하여 View들의 위치를 지정할 때 id가 지정되지 않으면 발생하는 오류인 것이다

#### ConstraintSet()을 통한 제약 설정
Constraint Layout에서는 Constraint Set이라는 것이 존재한다.

Constraint Set을 사용하여 제약을 코드로 설정한다

Constraint Set의 clone()에 Constraint Layout을 전해주면 해당 Constraint Layout의 제약 조건을 복사하게 되고,      
이를 전달받은 ConstraintSet 객체를 ConstraintLayout의 자식 View들을 제어할 수 있게 된다

ConstraintSet.connect() 함수를 활용하여 동적으로 설정한다       
connect 함수는 5개의 파라미터를 가지고 있다        
+ startID: 제약을 설정할 View의 ID
+ startSide: 제약을 연결할 View의 어떤 면 (TOP, BOTTOM, START, END)
+ endID: 연결할 대상 View의 ID
+ endSide: 연결할 대상 View의 면
+ margin: 마진 (px)

쉽게 생각하면 XML에서 Constraint Layout의 app:layout_constraintLayoutTop_toTopOf="parent" 라면
ConstraintSet.connect(내 View id, ConstraintSet.TOP, ConstraintSet.PARENT_ID, ConstraintSet.TOP) 이렇게 전환할 수 있다        

이렇게 Constraint를 설정하면 된다

설정이 완료되면 applyTo로 전달받은 ConstraintLayout에 제약 조건을 적용하게 된다 
