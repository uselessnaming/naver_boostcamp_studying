# Scope 함수
람다식을 이요해 호출하면 일시적인 Scope가 생기고, 이 범위 안에서 전달된 객체에 대해 it 혹은 this라는 context object를 통해서 접근하는 것

## Return value VS Context Object 참조

### this
run, with, apply는 Context Object를 "this"로 참조한다       
따라서 람다식 내에서 일반 클래스 멤버처럼 사용이 가능하다

### it
let, also는 Context Object를 "it"으로 참조한다  
따로 전달 인자명을 지정할 수 있고, 지정하지 않으면 기본적으로 it으로 접근한다

### Return value
+ Context Object 반환 : apply, also     
체인 형식으로 계속적인 호출이 가능


+ 람다식 결과 반환 : let, run, with
결과를 변수에 할당하거나, 결과에 대해 추가적인 작업 등을 수행할 때 사용할 수 있다       

## 함수 별 권장 케이스
### let
+ Context Object: it
+ Return value: lambda result

객체 결과 값에 하낭 ㅣ상의 함수를 호출할 경우 사용한다      

let은 null이 아닌 값으로만 코드 블록을 실행시킬 때 자주 사용한다        
null이 아닌 객체에 대해 작업을 수행하려면 안전한 호출 연산자를 let에 사용하도록 한다

```kotlin
val alphabet = mutableListOf("a", "b", "c")
alphabet.map{it.length}
.let{
    println(it)
    // 추가적인 함수 실행 가능
}
```

### with
+ Context object: with
+ Return value: lambda result

이미 생성된 Context object 객체를 인자로 받아서 사용하는 것이 효율적일 때 좋습니다

```kotlin
val alphabet = mutableListOf("a", "b", "c")
```

### run
+ Context object: this
+ Return value: lambda result

with와 비슷하게 이미 생성된 Context Object 객체를 사용할 때 호출하며, with와 전달받는 위치가 다르다

?을 통해 null 체크가 가능하기 때문에 with보다 run이 더 자주 사용된다

```kotlin
val width = run{
    val point = Point()
    point.x * 0.5
}
```

### apply
+ Context object: this
+ Return value: Context object

보통 객체 초기화 시 많이 사용함

```kotlin
val person = Persno("나").apply{
    age = 26
    addr = ""
}
```

### also
+ Context Object: it
+ Return value: context object

also는 기존 객체를 수정하거나 변경 없이, 디버깅을 위한 로깅 등의 추가적인 부가 작업 시 사용한다

## 결과
해당 순서도처럼 조건에 따라 scope 함수를 달리 써야 한다

<img width="890" height="440" alt="image" src="https://github.com/user-attachments/assets/ffc60cae-f99b-4142-b31a-637c46064dee" />
