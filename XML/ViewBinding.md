# View Binding
android layout resource 폴더 중 **layout**폴더의 xml에 대해 자동적으로 binding 클래스를 생성해주는 기능이다     

이름과 같이 ViewBinding을 사용하면 해당 layout을 가지고 object 객체를 생성하고, 이를 기반으로 내부 View들에 직접 접근이 가능해진다.

[공식 문서](https://developer.android.com/topic/libraries/view-binding?hl=ko)       
[참고 블로그](https://medium.com/androiddevelopers/use-view-binding-to-replace-findviewbyid-c83942471fc)

## 설정
모듈 수준의 `build.gradle` 파일에서 `viewBinding` 옵션을 `true`로 설정해야 한다

```kotlin
android{
    buildFeatures{
        viewBinding true
    }
}
```

만약 Layout 중 binding을 무시하고자 한다면 root view에 `tools:viewBindingIgnore="true"` 속성을 추가하면 된다고 한다

## 사용
기존에는 `findViewById<class명>(resource id)`형식으로 관련 View를 참조한 후에 속성에 접근하여 제어했다      

ViewBinding을 활용하게 되면 일일이 `findViewById`로 접근하지 않고 `onCreate()`에서 참조할 binding을 설정하면 된다     
설정하게 되면 이후에는 `binding.지정 id` 를 통해 접근이 가능하다        

## binding object 생성
binding object를 생성하기 위해서는 3가지 방법이 존재한다        

### `inflate(inflater)`
해당 함수는 `Activity`의 `onCreate()`함수에서 사용한다      
이는 binding object를 넘길 parent view가 존재하지 않을 경우 사용하게 된다       

### `inflate(inflater, parent, attachToParent)`
해당 함수는 `Fragment`나 `RecyclerView`의 `Adapter` 혹은 `ViewHolder`와 같이 binding object를 parent ViewGroup에 전달해야 할 때 사용한다

### `bind(rootView)`
해당 함수는 이미 inflate된 View를 사용하고 있고, 단지 `findViewById()`함수를 사용하지 않기 위해서 접근할 때 사용한다        

## `findViewById`와의 차이점
### Null-Safety
View Binding은 View에 대한 직접 참조를 생성하기 때문에 View의 id를 잘못 작성하거나 `NPE`가 발생할 위험이 없다       

### Type-Safety
binding 클래스의 필드 Type은 XML 파일에서 참조하는 뷰의 Type과 일치하기 때문에 변환 예외가 발생할 일이 없다

예를 들어
```kotlin
<TextView 
    android:id="@+id/tvTextView"
    android:text="..."
    .../>
```
이런 `TextView`를 생성하고, 이에 대해 `findViewById()`를 사용한다고 가정하자

```kotlin
val tv = findViewById<TextView>(R.id.tvTextView)
```
이와 같이 접근하게 된다면 문제가 발생하지 않을 것이다       

하지만 실수로 type을 잘못 지정한다고 생각해보면
```kotlin
val tv = findViewById<Button>(R.id.tvTextView)
```
이렇게 선언해도 컴파일 시점에서는 잘못된 것을 알 수 없다        

`findViewById()`함수는 런타임에 이를 발견하기 때문에 문제가 될 가능성이 크다        
빌드는 문제 없이 되지만 실제로 해당 화면에 가서는 잘못된 캐스팅으로 인해 비정상 종료될 것이다 (`ClassCastException`)

그에 비해 binding은 컴파일 시에 이를 발견하기 때문에 미리 방지할 수 있다

> 이 장점 + 보일러 코드 제거가 가장 크게 느껴진다