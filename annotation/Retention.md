# Retention 어노테이션
어노테이션의 생명 주기를 결정하는 어노테이션이다        
여기서 지정할 수 있는 타입은 `RUNTIME`, `BINARY`, `SOURCE`로 총 3가지 타입이 존재한다.

## SOURCE
컴파일 되면 어노테이션이 완전히 사라짐      
`.class` 파일에는 없게 됨       
`RUNTIME`에서는 영향이 없게 됨

**예시**        
+ `@Composable`
+ `@Suppress`
+ Lint 전용 어노테이션

즉, 컴파일러에게 힌트만 주고 끝난다

## BINARY
`.class` 파일에 포함된다        
단 RUNTIME 시 리플렉션은 불가능하다     
컴파일러 / 도구는 인식 가능

**예시**        
+ `@DelicateCoroutineApi`       
+ `@ExperimentalCoroutinesApi`
+ `@MetaData`

즉, 도구와 컴파일러용 정보를 제공

## RUNTIME
RUNTIME에도 유지됨      
리플렉션으로 조회 가능      
성능 비용 존재

**예시**
+ `@Inject`
+ `@SerialName`
+ `@Entity`
+ `@Test`

프레임워크가 동작하면서 읽는다

## 단계별 차이
.kt 파일을 컴파일 하는 과정에서 정보를 제공하고 이후 BINARY 파일부터는 보여주지 않도록 할 경우에는 `SOURCE` 전략이다        
`@Suppress`와 같은 경우 개발자에게만 노출되고 이후에는 노출할 필요가 없으므로, 이 경우 `SOURCE`로 처리하는 것이 적합하다        

BINARY 파일까지 유지되도록 하되 RUNTIME에서는 보이지 않도록 할 경우는 `BINARY`를 사용하게 된다          
`@DelicateCoroutineApi`나 `@ExperimentalCoroutineApi`와 같은 어노테이션들의 경우에 `BINARY`로 처리하고 있다     
이 때 이런 것들을 `SOURCE`로 하면 안되는 이유는 뭘까?       
이에 대한 대답은 라이브러리와 관련하여 생각하면 좋다. 라이브러리를 생성할 떄 바이너리 파일로 만들어 배포하게 된다. 이 때 `SOURCE`로 해뒀다면 이런 어노테이션 정보들이 모두 사라지기 때문에 사용하는 입장에서는 제작자의 의도를 알 수가 없다.        

앱을 실행하면서 계속 어노테이션 정보를 사용해야 하는 경우 RUNTIME 전략을 활용한다       
`@Entity`, `@Inject` 어노테이션과 같이 RUNTIME에서도 정보가 필요한 경우에는 해당 전략을 활용한다        
이 전략은 프로그램 실행 중 자기 자신의 구조를 파악해야 할 때 주로 사용한다. 예를 들면 객체의 생성 방법, 어떤 메소드를 자동으로 호출할지, 어떤 필드를 자동으로 채울지, 어떤 규칙을 동적으로 적용할지 등을 실행 중에 결정해야 할 때로 말할 수 있다        
실제 사용 예시 중에는 Spring에서 사용하는 `@Controller`나 Android의 `@Parcelize`, `@Serialize`와 같은 직렬화 어노테이션 등이 있다