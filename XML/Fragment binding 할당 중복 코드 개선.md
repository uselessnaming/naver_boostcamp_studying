# Fragment binding 관련 중복 코드 개선 방안
Fragment마다 binding에 null로 초기화한 후 onCreateView에서 할당, onDestroyView에서 null로 값 해제하는 과정을 반복하는 것은 번거롭다

이를 해결하기 위해 어떤 방법을 사용해야 할까

## Base Fragment 생성하여 상속해서 사용하기
```kotlin
typealias Inflate<T> = (LayoutInflater, ViewGroup?, Boolean) -> T

abstract class BaseFragment<VB: ViewBinding>(
    private val inflate: Inflate<VB>
): Fragment(){
    private var _binding: VB? = null
    val binding get() = _binding!!

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View?{
        _binding = inflate.invoke(inflater, container, false)
        return binding.root
    }

    override fun onDestroyView(){
        super.onDestroyView()
        _binding = null
    }
}
```
위와 같이 Fragment를 상속받는 BaseFragment를 생성하고, 여기서 반복되는 코드를 관리하도록 하는 방법이다

```kotlin
class MainFragment(): BaseFragment<BaseFragmentBinding>(BaseFragmentBinding::inflate){
    override fun onViewCreated(view: View, savedInstanceState: Bundle?){
        super.onViewCreated(view, savedInstaceState)

        binding.title.text = "짠"
    }
}
```
이렇게 상속받아 사용할 수 있다

가장 흔하게 사용하는 코드 방법이라고 한다

## Kotlin Delegated Properties 사용하기
[구글 샘플 앱 소스 코드](https://github.com/android/architecture-components-samples/blob/master/GithubBrowserSample/app/src/main/java/com/android/example/github/util/AutoClearedValue.kt)에서는 AutoClearedValue를 만들어 위임 패턴을 사용한다

binding 객체에 Delegated properties를 사용하는 방법이다


```kotlin
private var binding by autoCleared<BaseFragmentBinding>()

override fun onCreateView(
    inflater: LayoutInflater,
    container: ViewGroup?,
    savedInstanceState: Bundle?
): View?{
    binding = BaseFragmentBinding.inflate(inflater, container, false)
    return binding.root
}

override fun onViewCreated(view: View, savedInstanceState: Bundle?){
    super.onViewCreated(view, savedInstanceState)
    binding.title.text = "제목"
}
```

링크에 들어가면 있는 클래스를 사용하여 binding 관리를 넘기는 방법이다

## 외부 라이브러리 사용
[ViewBindingProperty 위임 라이브러리](https://github.com/androidbroadcast/ViewBindingPropertyDelegate)

위 라이브러리를 사용할 수도 있다        

## 결과
Fragment binding 과 관련된 중복 코드를 줄이기 위한 방법들이 위 3개 말고도 상당히 많다       
각각 방안들에 장단을 살펴서 적용해야 할 것 같다