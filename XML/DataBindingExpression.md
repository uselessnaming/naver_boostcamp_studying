# DataBinding Expression

+ `<layout>` 태그를 root로 하기
+ `<data>` 태그 내에 변수와 import 작성하기

## Data 태그
`<variable name="user" type="com.example.User"/>`와 같이 선언하면 User 클래스 타입을 가지는 user 변수에 접근할 수 있다      
Type에 외부 클래스를 사용한다면 import 문을 반드시 필요로 한다      
Type에 내가 정의한 클래스, 즉 Custom Class를 사용하게 될 때에는 해당 클래스의 path를 모두 적어야 한다

## Data 태그 사용
`<TextView android:text="@{user.name}"/>`과 같이 사용할 수 있다

**주의 사항**       
DataBinding Expression은 최대한 작고 간결하게 유지해야 한다     
unit test를 할 수 없고, IDE 또한 지원이 제한적이기 때문에 Custom `Binding Adapter`를 활용하여 최대한 간결하게 작성해야 한다

> Q 그럼 BindingAdapter는 어떻게 테스트할까?

## Binding Data
해당 layout xml 파일에 `DataBinding`을 적용하게 되면 (layout 태그로 변환하면) Binding Class가 생성된다      

다음과 같이 Activity에서 binding 설정이 가능하다     

```kotlin
// DataBindingUtil 클래스 사용
override fun onCreate(savedInstanceState: Bundle?){
    super.onCreate(savedInstanceState)
    val binding: XXXBinding = DataBindingUtil.setContentView(this, R.layout.xxx)
}

// binding에 직접 inflate
val binding = XXXBinding.inflate(getLayoutInflater())
```

만약 `Fragment`, `ListView`, `RecyclerView`에서 `inflate()`가 필요하다면 이때 역시 `DataBindingUtil`을 쓰거나 Binding 클래스에 직접 접근하거나 둘 중 선택할 수 있다

```kotlin
// DataBidningUtil 클래스 사용
val xxxBinding = DataBindingUtil.inflate(layoutInflater, R.layout.xxx, viewGroup, false)

// binding 클래스에 직접 접근
val xxxBinding = XXXBinding.inflate(layoutInfalter, viewGroup, false)
```

## Expression Langauge
+ Mathematical: `+ - / * %`
+ String concatenation: `+`
+ Logical: `&& ||`
+ Binary: `& | ^`
+ Unary: `+ - ! ~` (단항 연산자)
+ Shift: `>> >>> <<`
+ Comparison: `== > < >= <=` (&lt)
+ `instanceof`: 클래스 타입에 대한 정보가 필요할 때
+ grouping: `()`
+ Literals: `characters`, `String`, `numeric`, `null`
+ Cast
+ Method calls
+ Field access
+ Array access: `[]`
+ Ternary operator: `?:` (삼항 연산자)

## Missing Operations
+ this
+ super
+ new

## Null coalescing operator
`??` 을 사용해서 null일 떄 값을 정해줌      
`user ?? admin.name` user가 null이라면 admin.name 값을 갖도록 한다

## Avoid NPE
DataBinding에서는 NPE를 자동적으로 배제한다     
값이 null인 경우 자동적으로 null로 할당되며, int의 경우에는 0으로 default value가 잡힌다