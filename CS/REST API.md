# REST API
REpresentational State Transfer 의 약자     

HTTP이 주요 저자 중 한 사람이 그 당시 웹(HTTP) 설계의 우수성에 비해 제대로 사용되지 못하고 있어 웹의 장정을 최대한 활용할 수 있는 아키텍처로 REST로 발표한 것

## REST 구성
+ 자원(Resource) - URI
+ 행위(Verb) - HTTP METHOD
+ 표현(Representations)

## REST의 특징
### 1. Uniform
Uniform interface는 URI로 지정한 리소스에 대한 조작을 통일되고 한정적인 인터페이스로 수행하는 아키텍처 스타일

### 2. Stateless
상태를 별도로 가지지 않아서 단순히 API 서버는 들어오는 요청만 처리하면 된다     
따라서 서비스의 자유도가 높아지고 서버에서 불필요한 정보를 관리하지 않아 구현이 단순해진다

### 3. Cacheable
REST의 가장 큰 특징 중 하나는 HTTP라는 기존 웹표준을 그대로 사용하기에 Last-Modified 태그나 E-Tag를 이용해 캐싱이 구현 가능하다

### 4. Self-descriptiveness
REST API 메시지만 보고도 이를 쉽게 이해할 수 있는 자체 표현 구조로 되어있음

### 5. Client-Server 구조
REST 서버는 API를 제공하고, 클라이언트는 사용자 인증 혹은 Context 등을 관리하는 구조로 서로 역할을 확실히 구분된다      
따라서 서로 의존성이 줄어들게 할 수 있다

### 6. 계층형 구조
REST 서버는 다중 계층으로 구성될 수 있고, 보안, 로드 밸런싱, 암호화 계층을 추가해 **구조상의 유연성**을 둘 수 있고, PROXY나 Gateway 같은 네트워크 기반의 중간매체를 사용할 수 있다

## REST API 디자인 가이드
### URI는 정보의 자원을 표현해야 한다
리소스 명은 동사보다 명사를 사용        
```
GET /items/delete/10
```
이렇게 delete같은 행위 표현은 넣지 않는다

### 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)로 표현
위의 잘못된 표현은
```
DELETE /items/10
```
이렇게 수정할 수 있다

## HTTP Method 역할
+ GET : 리소스 조회
+ POST : 리소스 생성
+ PUT : 리소스 수정
+ DELETE : 리소스 삭제

## URI 설계 시 주의할 점
+ 슬래시 구분자는 계층 관계를 나타내는 데 사용한다
+ URI 마지막 문자로 '/'를 사용하지 않는다
+ '-'는 URI 가독성을 높이는 데 사용
+ '_'는 URI에 사용하지 않음
+ URI 경로에는 소문자로 (대문자는 피해야 함)
+ 파일 확장자는 URI에 포함하지 않음