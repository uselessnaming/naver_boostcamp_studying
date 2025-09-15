# JSON
JavaScript Object Notation의 약자       
JavaScript 객체 문법으로 구조화된 데이터를 표현하기 위한 문자 기반의 표준 포맷

JavaScript 객체 문법과 매우 유사하나 JavaScript에서만 사용하는 것이 아니고, 널리 사용하고 있다

## Serialization 직렬화
객체를 binary 형식으로 변환하는 것        

## Deserialization 역직렬화
binary 형식을 객체로 변환하는 것

## Kotlin에서 Json 파싱을 위한 라이브러리
### Kotlin-Serialization
kotlin 공식 라이브러리로 `kotlinx-serialization-json` 의존성을 추가해야 한다        
Multiplatform을 지원해준다      
다양한 종류의 포멧에 대한 변환을 지원해준다     
(CBOR, protobuf, HOCON, properties 등 지원해주나, 아직 실험적 API이므로 주의해야 한다)      

필요한 부분에서 `@Serializable` 어노테이션을 붙이면 컴파일 타임에서 serializer 코드가 생성된다

Retrofit과 호환되도록 Converter를 추가해줬기 때문에 Converter Factory를 설정해주는 것으로 Retrofit과도 함께 사용할 수 있다

내부 코드를 타고 들어가면 
```kotlin
public val SerialDescriptor.elementDescriptors: Iterable<SerialDescriptor>
    get() = Iterable {
        object : Iterator<SerialDescriptor> {
            private var elementsLeft = elementsCount
            override fun hasNext(): Boolean = elementsLeft > 0

            override fun next(): SerialDescriptor {
                return getElementDescriptor(elementsCount - (elementsLeft--))
            }
        }
    }
```
이 코드를 볼 수 있다            
완전 분해는 어렵지만 예전에 구현했던 경험을 생각해보면 순회 구조를 통해서 재귀적으로 값들을 파싱하는 것을 볼 수 있다        
다만 여기서 재귀적인 구조가 
SerialDescriptor에는 serialName, kind, isNullable과 같은 필드 값을 가지고 있다

```kotlin
@Serializable
data class Reactions(
    @SerialName("+1") val plus: Int,
    @SerialName("-1") val minus: Int,
    val confused: Int,
    val eyes: Int,
    val heart: Int,
    val hooray: Int,
    val laugh: Int,
    val rocket: Int,
    val totalCount: Int,
    val url: String,
)
```
이렇게 클래스 선언 시 `@Serializable` 어노테이션을 활용해 자동으로 Serialization 해주는 코드를 생성할 수 있다       
그리고 Field 명을 변경해야 한다면 `@SerialName`으로 변경이 가능하다

[kotlin 문서](https://kotlinlang.org/docs/serialization.html#add-plugins-and-dependencies)      

[github library - serialization](https://github.com/Kotlin/kotlinx.serialization)

[github library - serialization converter](https://github.com/JakeWharton/retrofit2-kotlinx-serialization-converter)

### GSON
코틀린의 default value 문법을 무시한다      
primitive 값은 0, reference 타입의 필드는 null 값을 default로 한다      
GSON은 코틀린의 null-safety를 준수하지 않기 때문에 고려하지 않으면 Crash가 발생하게 된다        
따라서 Nullable하게 설계해야 한다       

GSON은 리플렉션을 많이 사용하기 때문에 문제를 일으키는 경우가 많은 직렬화 라이브러리다      
따라서 성능 저하가 발생할 가능성이 있다

또한 Android Gradle 파일로 이동하면 
```kotlin
release {
    isMinifyEnabled = false
    proguardFiles(
        getDefaultProguardFile("proguard-android-optimize.txt"),
        "proguard-rules.pro"
    )
}
```
위와 같은 코드를 마주한다

release 버전일 때 proguard를 활용한 난독화를 시행하게 되는데, 이 때 Gson을 사용해 파싱할 때 제대로 되지 않는 현상이 발생한다.       
> ProGuard는 앱의 용량을 줄이고 코드의 보안을 강화하며 성능을 최적화하기 위함이다

이 같은 것을 고치려면 
```kotlin
-keep class 모델경로 { *; }
```

이 코드를 활용하여 이 경로 내부를 난독화를 하지 않도록 수정해야 한다

[공식 문서](https://developer.android.com/topic/performance/app-optimization/choose-libraries-wisely?hl=ko)     

### Moshi
Square 에서 개발한 JSON 파서 라이브러리     
GSON과 유사하나 성능이 더 빠르고 코틀린과의 호환성이 더 높은 특징이 있다        
Moshi는 자바 및 안드로이드 앱과 함께 사용할 수 있다

#### 장점
+ Gson보다 더 빠름
+ 코틀린과의 호환성이 더 높음
+ 직렬화 및 역직렬화를 보다 쉽게 할 수 있다
+ 유연성이 높아 사용자 형식을 처리하기 쉽다
+ 오류 발생 시 예외를 Throw한다

```kotlin
@JsonClass(generateAdapter = true)
data class Reactions(
    @Json("+1") val plus: Int,
    @Json("-1") val minus: Int,
    val confused: Int,
    val eyes: Int,
    val heart: Int,
    val hooray: Int,
    val laugh: Int,
    val rocket: Int,
    val totalCount: Int,
    val url: String,
)
```
위와 같이 클래스 선언 시 `@JsonClass` 어노테이션을 활용해 생성하면 adpater를 자동으로 생성해준다        
그리고 만약 필드 값을 다르게 해야 한다면 혹은 위와 같이 사용할 수 없는 필드 값일 경우 `@Json`을 사용해 변경해줄 수 있다

```kotlin
private fun logMoshi(json: String) {
    val moshi = Moshi.Builder()
        .add(KotlinJsonAdapterFactory())
        .build()

    val type = Types.newParameterizedType(List::class.java, Issue::class.java)
    val issueAdapter: JsonAdapter<List<Issue>> = moshi.adapter(type)
    val issue = issueAdapter.fromJson(json)

    Log.d("TEST JSON", "${issue}")
}
```
위와 같이 moshi로 적용할 수 있다        
다만 json이 Object가 아닌 Array로 되어 있다면 Moshi의 Types.newParameterizedType에 접근하여 별도 처리가 필요하다

내부 코드를 확인하면 자체 Stack을 구현해 Serialization을 구현하는 것을 볼 수 있다

## 차이에 대한 견해
GSON 라이브러리의 경우 null safety 적용이 되지 않고, Default Value 또한 무시하는 문제가 있다        
리플렉션을 사용하여 속도가 저하되는 문제가 있다        

따라서 Moshi와 Kotlinx Serialization 중에서 선택하게 되었다     

Moshi는 Serialization 시 자체 Stack을 활용하여 Serialization을 진행한다     

Kotlinx-Serialziation 같은 경우 재귀적으로 진행한다     
> 정확히는 모르겠으나 내부적으로 inline 함수를 적절히 활용하여 실제 함수 호출의 오버헤드를 최소화한다고 학습했다      

결과적으로 Moshi와 Kotlinx-Serialization 두 라이브러리의 차이를 보면 된다       

[Moshi vs KotlinX Serialization 성능 비교](https://medium.com/@kacper.wojciechowski/moshi-vs-kotlinx-serialization-the-ultimate-benchmark-a7ed776a46c0)         
위 자료를 보면 각각 일반적인 모델, 크기가 큰 모델, Sealed class로 구분지어 각각의 직렬화, 역직렬화에 대한 시간을 측정하고 분석한 결과 모두 Kotlinx-Serialization이 더 빠름을 확인했다      
전반적으로 kotlinx-serialziation이 성능이 더 빠르며, Moshi는 코틀린에 덜 친화적이기 때문에 몇몇 버전에서는 사용할 수 없는 이슈도 있다고 한다