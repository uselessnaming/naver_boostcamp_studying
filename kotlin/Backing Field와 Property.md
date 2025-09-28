# Backing Field와 Property
[공식 문서](https://kotlinlang.org/docs/properties.html#changing-visibility-or-adding-annotations)

## 용어
+ Field? 클래스 내에서 선언된 변수
+ Property? Field에 접근하기 위해 사용

다만 kotlin에서는 바로 `Field`에 접근하지 않고 `Property`를 통해서 `Field`에 접근한다

Kotlin에서 `Property`를 정의하게 되면 컴파일 과정에서 자동으로 `Field`를 private하게 선언하며 이에 접근할 수 있는 getter와 setter 메소드를 정의한다

Kotlin에서 `val`은 변경 불가 `Property`를 정의하고, `var`은 변경 가능한 `Property`를 정의하는 것을 알 수 있다       
이는 곧 `val`은 `getter`만 정의해주고, `var`은 `getter`와 `setter`를 정의해줌을 알 수 있다

## Backing Field
이는 자바에서는 존재하지 않는 개념이다      
`Property` 정의 시 자동으로 `Field`를 `private`으로 선언한다
이 때 `Field`를 `Backing Field`라고 한다

`Property`를 정의하면 얻을 수 있는 `getter`와 `setter` 메소드를 `Property` 선언 과정에서 커스텀하게 정의할 수 있다

이 때 사용하는 것이 `get()`, `set()` 함수다

### 사용 예시
**Test Class**      
```kotlin
class TestClass {
    var a: String = ""
        get() = "var"
        set(value) {
            if (value.length >= 2) {
                field = value
            }
        }

    val b: String = "val"
}
```

이 클래스를 통해 테스트 해보자

다만 `val`로 정의한 변수에서 `set()`을 사용하면 컴파일 오류가 발생한다.     
물론 당연히 `value`로 불변이기 때문에 `set()`함수는 정의되지 않는다.        
따라서 이를 정의하고자 하면 오류가 발생한다

그리고 `val`에서는 `get()`을 정의하는 것도 컴파일 오류가 발생한다.      
`Initializer is prohibited here because this property has no backing field.`라는 문구와 함께 이 `Property`가 `BackingField`를 가지지 않기 때문에 초기화가 금지된다는 오류가 생긴다.     

decompile한 Java 파일을 확인해보자
```java
public final class TestClass {
   @NotNull
   private String a = "";
   @NotNull
   private final String b = "val";

   @NotNull
   public final String getA() {
      return "var";
   }

   public final void setA(@NotNull String value) {
      Intrinsics.checkNotNullParameter(value, "value");
      if (value.length() >= 2) {
         this.a = value;
      }

   }

   @NotNull
   public final String getB() {
      return this.b;
   }
}

```
이를 확인하면 `val` 은 `final` 키워드로 지정되고 `setB()`는 별도로 정의되지 않고 `getB()`만 정의되는 것을 알 수 있다.       
그에 비해 `var`은 `final` 키워드가 없이 정의되며 `getA()`와 `setA()`가 정의되었다       
또한 custom setter가 설정되어 `setA()`함수 내에서 확인할 수 있었다

그리고 만약 `custom setter`를 설정하면 클래스 밖에서는 수정할 수 없는 Property가 된다       

## Backing Property
이는 Backing Field에 적용한 개념들을 넓혀서 Property에 적용한 것이다

```kotlin
class TestClass{
    private var _a: String = ""
    val a: String
        get() = _a
}
```
이렇게 Backing Property를 정의할 수 있다        

이를 decompile 하면 
```java
public final class TestClass {
   @NotNull
   private String _a = "";

   @NotNull
   public final String getA() {
      return this._a;
   }
}

```
이렇게 java 파일이 생성된다

코드를 확인해보면 a에 대한 필드가 생성되지 않음을 확인할 수 있다        
이는 a 필드가 전혀 사용되지 않기 때문에 a 필드를 굳이 생성하지 않는다

만약 `Backing Property`를 사용하지 않고 `val a: String = _a`를 사용한다면, 코틀린 컴파일러는 자동으로 생성한 `get()`메소드에서 `Backing Field`를 반환해야 하기 때문에 기존 불필요한 Field를 만들고 값을 넣어줘야 한다

> 따라서 보일러 코드를 줄일 수 있는 장점이 있다

## 목적
`Backing Property`와 `Backing Field`를 활용하면, 클래스 내부에서 데이터를 안전하게 관리할 수 있다       
다른 클래스에서는 `Property`를 통해 값을 읽을 수 있지만, `Property`의 `setter`를 제한하거나 특정 조건에서만 수정 가능하도록 하는 등 제어할 수 있다      
이를 통해서 데이터의 캡슐화를 강화하고, 잘못된 접근을 방지할 수 있다

## 차이점?
```kotlin
class Test1 {
    private var _word = "test"
    val word: String
        get() = _word
}

class Test {
    var word = "test"
        private set
}
```

이 두 코드의 차이가 뭘까?        
이 차이는 MutableList나 List를 활용하여 확인할 수 있다      

```kotlin
class Test1 {
    private val _words = mutableListOf<String>()
    val words: List<String>
        get() = _words
}

class Test2 {
    var words = mutableListOf<String>()
        private set
}
```
이런 예시를 사용해본다면 디컴파일시 첫 번째 코드는 final 키워드가 적용되고, 두 번째 코드는 final 키워드가 적용되지 않는다

## 결론
`Backing Field`를 활용한 복잡한 로직 등이 필요하다면 첫 번째 방법을 사용하되 그렇지 않은 경우에서는 간단한 두 번째 방법을 활용하는 것이 좋을 것이다

따라서 이런 차이를 확인하고 사용해야 한다