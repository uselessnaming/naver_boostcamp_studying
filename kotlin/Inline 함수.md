# Inline 함수
> 고차함수를 생각해야 한다!

**Lambda를 사용한다고 생각해보자**

```kotlin
fun someMethod(a: Int, doSomething: () -> Unit): Int{
    doSomething()
    return 2*a
}
```
위 코드를 컴파일하면 어떤 Java 코드가 생성될까?

```java
public final class LambdaFunctions {
   public static final LambdaFunctions INSTANCE;

   public final int someMethod(int a, @NotNull Function0 doSomething) {
      doSomething.invoke();
      return 2 * a;
   }

   @JvmStatic
   public static final void main(@NotNull String[] args) {
      int result = INSTANCE.someMethod(2, (Function0)null.INSTANCE);
      System.out.println(result);
   }

   static {
      LambdaFunctions var0 = new LambdaFunctions();
      INSTANCE = var0;
   }
}
```

확인해보면 doSomething에 람다식을 전달해주도록 구현한 부분에 대해서 새로운 instance를 생성하고, 이를 넘겨주도록 변환됨을 확인할 수 있다

이렇게 Lambda를 사용할 때 무의미한 객체를 생성하여 메모리를 차지하고, 내부적으로 연쇄적인 함수 호출을 하게 되어 오버헤드로 인해 성능이 떨어질 수 있다

따라서 람다식 내부의 실행문들을 컴파일 시 람다를 호출하는 부분에 주입하는 방식을 사용하는 것이 **Inline Function**의 개념이다

```kotlin
inline fun someMethod(a: Int, doSomething: () -> Unit): Int {
    doSomething()
    return 2 * a
}
```
이렇게 동일한 코드에 inline 키워드를 추가하면 

```java
public final class InlineFunctions {

   @JvmStatic
   public static final void main(@NotNull String[] args) {
	     int a = 2;
       int var5 = false;
       String var6 = "ㅇㅅㅇ";
       System.out.println(var6);
       int result = 2 * a;
       System.out.println(result);
   }

}
```
컴파일 시 람다 부분에 내부 코드가 그대로 복사됨을 확인할 수 있다

컴파일되는 바이트 코드 양은 늘지만, 객체 생성 혹은 함수를 재호출하는 등 비효율적인 행동을 하지 않아서 일반적인 함수보다 성능이 좋다

### noinline keyword
전달 받은 함수 중 일부는 다른 함수로 넘겨줘야 한다면?   
모든 인자를 inline 처리하면 안되는 경우가 생긴다

이를 위해 필요한 키워드가 noinline이다.

```kotlin
inline fun firstMethod(a: Int, func1: () -> Unit, noinline func2: () -> Unit) {
    func1()
    secondMethod(10, func2)
}

fun secondMethod(a: Int, func: () -> Unit): Int {
    func()
    return 2 * a
}

fun main() {
    firstMethod(2, {
        println("Just some dummy function")
    }, {
        println("can't pass function in inline functions")
    })
}
```

이렇게 first method의 3번째 인자로 2번째 function을 받고, 이를 second method에 넘겨줘야 하므로 여기서 inline이 발생하면 안된다

해당 코드를 자바로 변환하게 되면 

```java
public final void firstMethod(int a, @NotNull Function0 func, @NotNull Function0 func2) {
   func.invoke();
   this.secondMethod(10, func2);
}

public final int secondMethod(int a, @NotNull Function0 func) {
   func.invoke();
   return 2 * a;
}

@JvmStatic
public static final void main(@NotNull String[] args) {
   String var6 = "Just some dummy function";
   System.out.println(var6);
   this_$iv.secondMethod(10, func2$iv);
}
```

위와 같이 전환되며, func()는 기존 방식대로 객체를 생성하여 호출하게 되는 것을 확인할 수 있다

> func2$iv 로 표현된 부분이 객체 생성 부분으로, decompiler가 보여주는 코드가 단순화되어 있어 new 구문이 전부 보이지 않을 뿐이다

## inline 함수는 항상 옳은가??

본문에서 얘기했듯이, 바이트 코드가 늘어나는 단점은 존재한다.

따라서 많은 코드를 갖고 있는 람다를 inline 처리하게 된다면 바이트 코드의 양이 훨씬 많아지게 되며, 이는 성능 악화를 초래할 수 있다

**결과적으로 inline 처리는 1~3줄 정도의 길이를 권장하고 있다**

## 안드로이드의 JetpackCompose Row, Column은 왜 inline 일까?

Row, Column의 content 부분은 보통 람다로 제공된다

여기서 UI를 계속 갱신하는 프레임워크에서 inline 함수를 사용하지 않게 되면 오버헤드가 엄청나게 쌓이게 된다

그렇기 때문에 성능 최적화를 위해 inline 함수를 호출하고 있다

---

대부분의 Composable이 상태를 가지게 된다 >> 여기서 상태를 관리하는 측면에서 이득이 있고, 오버헤드가 적어져서 사용하고 있다