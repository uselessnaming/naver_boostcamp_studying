# Fragment
'Fragment는 앱 UI의 재사용 가능한 부분을 나타낸다'라고 공식 문서에 나와있다        
이는 무슨 말일까?

Activity와 비교해보면 조금 더 이해가 쉬웠다        

Activity는 앱 UI 주위에 전역 요소들을 배치하고 Fragment는 단일 화면 혹은 화면 일부의 UI를 정의하는 것이다      
이런 것들의 연장선으로 Fragment는 Activity 없이는 생성이 불가능하다       
Activity에 의존적이라고 봐야 하는데, Fragment는 Activity 위에서 동작하기 떄문이다       

Activity에 Fragment를 올려두고 어떻게 사용해야할까?        
Activity 하나에 Fragment 하나를 올리고, fragment manager를 활용하여 fragment 교체할 수 있다.      
add를 통해 Activity에 Fragment를 올릴 수도 있고, replace를 통해서 현재 올라가 있는 Fragment를 교체할 수 있다.        

Fragment Transaction 과정에서 addToBackStack()을 호출하여 fragment manager가 관리하는 back stack에 쌓이게 된다      
이는 Activity의 Back Stack과는 별도로 진행된다      
> Activity 위에서 Fragment를 사용하니 어찌보면 당연한건가 싶기도 하다

그리고 Activity Back Stack 같은 경우는 OS가 관리한다

즉 정리해보면 Activity의 Back Stack이 존재하나 해당 Activity에서 여러 Fragment의 상태에 대해 관리할 수 있다       
그리고 fragment의 back stack은 결국 fragment manager가 관리하며 이는 activity에 종속적이게 된다고 정리할 수 있다     

[공식 문서](https://developer.android.com/topic/libraries/view-binding?hl=ko#fragments)     
[참고 블로그](https://kkamsoon-developer.tistory.com/19)

## 생명 주기
<img width="511" height="638" alt="image" src="https://github.com/user-attachments/assets/a73cd5df-faf5-4425-8183-1c3902573cf8" />

android developer에서 제공하는 Fragment의 lifecycle이다      

보면 Fragment의 Lifecycle은 2개가 존재한다      
Fragment의 자체 Lifecycle과 View의 Lifecycle이다        
Fragment는 Fragment View보다 Lifecycle이 길다       
Fragment는 Fragment의 재사용을 대비해 Fragment View들을 메모리에 보관하도록 되어 있다       
그래서 Fragment의 onDestroy 호출 후 Fragment에 대한 레퍼런스는 존재하지 않지만 내부적으로 View들은 재사용을 위해 보관하고 있다      

```kotlin
private var _binding: FragmentBinding? = null
private val binding get() = _binding!!

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View? {
    _binding = FragmentBinding.inflate(inflater, container, false)
    val view = binding.root
    return view
}

override fun onDestroyView(){
    super.onDestroyView()
    _binding = null
}
```
공식 문서에서는 위와 같이 binding 객체를 null로 초기화한 후에, `onCreateView`에서 binding을 할당하고 `onDestroyView`에서 binding을 해제하는 로직을 작성해야 한다        
이 방식을 통해서 GC에서 수집해갈 수 있도록 해야 메모리 누수에서 자유로울 수 있다        

[binding 관련 보일러 코드를 제거하기 위한 방안 모색](https://github.com/uselessnaming/naver_boostcamp_studying/blob/main/XML/Fragment%20binding%20%ED%95%A0%EB%8B%B9%20%EC%A4%91%EB%B3%B5%20%EC%BD%94%EB%93%9C%20%EA%B0%9C%EC%84%A0.md)

여기서 생략된 부분이 존재한다        

#### onAttach()
해당 fragment 생명 주기에서 현재 보이지 않는데, 이건 뭘까?      
Fragment가 FragmentManager에 추가되고, 호스트 Activity에 연결될 때 호출된다       
이 시점에 Fragment가 활성되고, Fragment Manager는 Fragment의 생명 주기를 관리한다       

#### onCreate()
여기서는 Fragment만 CREATED 된 상황이다       
FragmentManager에 add되면 여기에 도달하고, onCreate() 콜백 함수를 호출한다     

여기서 Fragment View 생성이 되지 않은 상태이기 때문에 여기서는 View와 관련된 작업을 수행해서는 안된다       
Fragment를 생성하면서 념겨준 값들이 존재한다면, 여기서 변수에 넣어주면 된다      

+ onActivityCreated 지원 중단
기존에 존재했던 생명주기로 Activity의 생성이 끝나야만 처리되어야만 처리할 수 있는 로직이 Fragment에 존재할 때를 위해 있는 것이다.       
그래서 최근에는 deprecated되면서 Activity와 Fragment의 생명주기가 서로 영향을 덜 주도록 개편되었다고 한다     

현재는 이 생명 주기가 onViewCreated와 onCreate로 구분되어졌으며,          
onCreate에서는 초기화 루틴을, onViewCreated에서는 View 관련 초기화를 수행하는 것을 권장한다

#### onCreateView()
Fragment가 View를 그리기 위한 부분으로, Layout을 inflate 하는 작업을 수행하는 부분이다       
따라서 반환 값도 View이며, 아직 UI를 그리지 않았다면 null 값이 반환된다      

#### onViewCreated()
onCreateView() 생명주기에서 반환된 View 객체를 onViewCreated()의 파리미터로 전달된다      
이 시점부터는 Fragment view의 생명주기가 INITIALIZED 상태로 업데이트 된다        
여기서는 View의 초기값 설정, LiveData observing, adapter 초기화 등을 수행하는 곳이다

#### onViewStateRestored()
onViewStateRestored() 함수는 저장해둔 모든 state 값이 Fragment의 View 계층 구조에 복원됐을 때 호출된다        
View lifecycle owner는 INITIALIZED 상태에서 CREATED 상태로 변경됨을 알린다

> 이 부분은 조금 낯설고 어렵네.. 더 봐야겠다

#### onStart()
Fragment가 사용자와 상호작용 가능할 때, 대화가 가능할 때 호출되는 것으로, 해당 시점부터는 FragmentTransaction을 안전하게 수행할 수 있다      
> Activity의 onStart()와 거의 비슷하다

#### onResume()
Fragment가 보이는 상태에서 모든 Animator와 Transition 효과가 종료되고, Fragment가 사용자와 상호작용할 수 있을 때 onResume() 콜백이 호출된다        
Resumed 상태가 됐다는 것은 사용자가 Fragment와 상호작용 하기에 적절한 상태가 된 것이다

#### onPause()
사용자가 Fragment를 떠나기 시작했으나 Fragment는 여전히 visible일 때 호출

#### onStop()
Fragment가 더 이상 화면에 보이지 않게 되면 Fragment와 View의 생명주기는 CREATED 상태가 되고, onStop() 콜백 함수가 호출된다     
onStop()이 onSaveInstanceState()보다 먼저 호출되는 것으로 onStop()이 FragmentTransaction을 안전하게 수행할 수 있는 마지막 지점이 된다

+ onStop()과 onSaveInstanceState() 순서 변경
API 28 전에는 onStop()보다 onSaveInstanceState()가 먼저 호출되었으나, 28이후로는 순서가 변경되었다        
변경되는 것으로 onStop()에서도 안전하게 transaction을 안전하게 수행할 수 있게 되었다        

일반적인 생각으로는 onStop()에서 상태를 저장하게 되면 최종 상태를 저장하는 것이라고 생각한다     
하지만 실제로 API 28 이전에서는 onStop()보다 onSaveInstanceState()가 먼저 호출되어 상태를 저장되어 버리는 문제가 발생했었다       
그래서 onStop과 onSaveInstanceState() 호출 순서가 바뀌었다고 한다

#### onDestroyView()
animation 작업과 transition 작업이 모두 완료되고, Fragment가 화면으로부터 벗어나면 Fragment View의 생명주기는 DESTROYED가 되고 onDestroy()가 호출된다        
해당 시점에선느 가비지 컬렉터에 의해 수거될 수 있도록 Fragment View에 대한 모든 참조가 제거되어야 한다

#### onDestroy()
Fragment가 제거되거나 FragmentManager가 destroy 됐을 경우, Fragment의 생명주기는 DESTROYED 상태가 되고, onDestroy 콜백 함수가 호출된다     
해당 지점은 Fragment 생명 주기의 끝을 알린다       

이 콜백 이후로 onDetach()가 Activity에서 Fragment가 떨어지게 된다

## Fragment binding
```kotlin
override fun onCreateView(
    inflater: LayoutInflater, container: ViewGroup?,
    savedInstanceState: Bundle?,
): View? {
    binding = FragmentMainBinding.inflate(inflater, container, false)
    return inflater.inflate(R.layout.fragment_main, container, false)
}
```

fragment의 onCreateView에서 binding을 연결할 수 있다      
inflate()함수 제일 마지막 파라미터는 boolean 값으로 field 값은 attachToParent 이다      

이건 뭘까?      
찾아본 결과 해당 값을 true로 변경하면 부모 view에 현재 fragment를 add 하게 된다

그렇다면 문제 없는 거 아닐까? 라는 생각을 가졌다        

<img width="1710" height="82" alt="image" src="https://github.com/user-attachments/assets/94020de4-e0af-4ef8-838f-2a1b017729e5" />

true로 하면 다음과 같은 오류가 생긴다     

생기는 이유는 생각보다 간단했다       
Activity에서 Fragment Manager를 호출하여 add하였는데, attachToParent 가 true로 동작하면서 이미 추가된 view를 다시 한 번 추가하려고 하는 것으로 인해 오류가 발생하게 된다

따라서 해당 속성은 false로 해야 한다

**Q 그럼 언제 써야할까?**       
해당 속성이 존재한다는 것은 사용할 곳이 있다는 건데.. 어디에 사용해야 할까?        
찾아보니 Recycler View를 예시로 드는 곳이 있었는데, 보고 확실히 이해가 되었다.     
View를 추가해서 바로 부모에 붙이는 것의 대표적인 예시가 Recycler View로, 아이템이 추가되면 반영하여 View를 그리게 되는데, 이런 동작이 필요할 때 true로 설정해야 한다

## Fragment Transaction
+ add: fragment를 stack에 추가할 수 있다
+ replace: fragment container에 보이는 fragment를 다른 fragment로 교체한다
+ show: framgent를 띄우지만 stack에 추가하지는 않음
+ hide: show된 fragment를 숨김

그리고 모든 transaction은 commit()을 호출해야 시작된다.        
commit()과 commitNow() 2개가 존재하는 데, commit()은 비동기, commitNow()는 동기식으로 동작하지만 대부분 commit()의 사용으로 충분히 구현할 수 있다

show&hide 방식은 간단하게 fragment를 띄우고 숨길 수 있지만 메모리를 더 많이 사용하게 된다 

<img width="973" height="350" alt="image" src="https://github.com/user-attachments/assets/1d67a743-240b-4076-a907-f4bcca193b63" />

기본적으로 activity 시작 시 fragment를 붙인다고 할 때 어떤 transaction을 사용하는 것이 좋을까?     
당연히 아무것도 없는 activity fragment container에 fragment를 추가하는 것이니까 add라고 생각했다.        
하지만 화면 전환 시 fragment의 정보 또한 새로 바뀌기 때문에 fragment가 2번 쌓일 수 있다

**왜 Activity가 소멸하고 재생성되는데 Fragment가 2번 생성될까?**      
찾아본 결과 Configuration Change로 Activity 재생성 시 현재 상태를 저장헀다가 onCreate에서 불러와 상태를 보존하게 된다     
여기서 FragmentManager 또한 현재의 Fragment를 저장하고 다시 생성하게 된다        
즉, 이렇게 되면 FragmentManager에서 보존한 Fragment와 새로 add한 Fragment, 총 2개가 Activity에 존재하게 되는 것이다

위의 사진을 확인해보면 onAttach ~ onViewCreated 까지의 생명주기가 2번씩 반복되고, 그 이후로도 2번씩 같은 생명주기를 반복하는 것을 확인해볼 수 있다

따라서 초기 화면에서도 add로 진행하는 것보다 replace로 진행하여야 이런 참상을 막을 수 있다

<img width="1034" height="482" alt="image" src="https://github.com/user-attachments/assets/9aac772b-ddd9-4aae-894c-9ad575256c8e" />

해당 Log는 add로 처리했을 때 Main Fragment와 Add Alarm Fragment의 생명 주기를 확인할 수 있다       

보면 Main Fragment에서 onResume() 호출 후에 Add Alarm Fragment가 위에 덮어지는 것을 확인할 수 있다

<img width="1032" height="676" alt="image" src="https://github.com/user-attachments/assets/342245cd-e6be-4999-8f77-9c55627cd60d" />

해당 Log는 replace로 처리했을 때 Main Fragment와 Add Alarm Fragment의 생명 주기를 확인할 수 있다

여기서 확인해보면 replace로 했을 때는 add와 달리 onStop() 상태에서 교체할 fragment를 생성하고, View가 생성된 후에 기존 fragment가 destroy 된다.     
그리고 마찬가지로 다시 replace를 할 경우에 동일하게 onStop()까지 호출한 상태에서 교체할 fragment의 생성을 대기한 후에 fragment가 destroy 된다

> 왜 그럴지 조금 생각해보니 어찌보면 당연하다고 생각할 수도 있을 것 같다!   

만약 그대로 destory 후에 create가 된다면 화면에 깜빡임이 생길 수 있다. 따라서 이를 부드럽게 교체하기 위한 조절하는 것이다.

<img width="993" height="353" alt="image" src="https://github.com/user-attachments/assets/6144f932-4ef7-421f-8b4e-b658c042d03d" />

해당 Log는 show와 hide를 처리했을 때 Main Fragment와 Add Alarm Fragment의 생명 주기를 확인할 수 있다       

확인하면 정말 특이하다는 것을 알 수 있었다        

show는 기존에 add된 fragment만 띄울 수 있는데, 생명 주기로 확인해보면 두 개의 fragment가 모두 onResume()을 호출하고 대기하고 있음을 알 수 있다      

show, hide를 사용하면 속도는 빠르지만 두 개의 fragment를 모두 RESUMED 상태에 존재하기 때문에 리소스를 더 많이 사용하게 된다

#### 결론
+ 전체 화면의 교체를 원한다면 replace를 호출
+ 작은 dialog와 같은 것들을 사용할 때 add 이후 hide, show를 호출
+ 화면 일부에만 fragment를 사용할 때 add 호출 >> 이것 또한 show, hide를 통해 관리 가능
