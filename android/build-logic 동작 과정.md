# build-logic 동작 과정
## build-logic 사용 계기
멀티 모듈을 사용하는 과정에서 모듈을 추가하게 되면 공통적으로 build.gradle 파일을 관리하게 됩니다.  모듈의 개수가 많아지면 많아질수록 각 build.gradle 파일들의 공통 설정에 대한 중복적인 코드가 늘어나고, 버전 코드 등 여러 설정 값들이 개별 관리될 가능성이 있습니다.  따라서 build-logic을 활용하는 것으로 이런 불편함을 해소할 수 있습니다.

## build-logic 적용 방법
root project에 build-logic 경로 추가합니다.

```kotlin
class AndroidApplicationConventionPlugin: Plugin<Project> {
    override fun apply(target: Project) {
        ...
    }
}
```
Plugin 클래스를 생성합니다.  

생성된 클래스를 build.gradle 파일 내 `gradlePlugin{ }` 블록에 등록합니다.  

root project의 setting.gradle 파일 내 `pluginManagement{ }` 블록 내에서 `includeBuild("buildlogic")` 코드를 통해 독립 build 추가합니다.  
> includeBuild() 사용 시 root project bulid 시 독립적인 build가 일어납니다. 이후 buildlogic 독립 build되어 compile되고, 이를 사용하게 됩니다.

[!WARNING]
setting.gradle 파일에서 `pluginManager{ }` 블록은 가장 최전방에 위치해야 합니다. `pluginManagent{ }` 블록은 plugin의 해석에 대한 정보를 가지고 있고, 코드는 위에서 아래로 실행되기 때문에 다른 작업들 보다도 가장 우선순위를 갖게 됩니다. 만약 `pluginManagement{ }` 블록이 최상단이 아닐 경우 `pluginManagement{} block must appear before any other statements` 오류가 발생하게 됩니다.

[!WARNING]
build-logic에서 settings.gradle 파일이 없다면 compile build로 인식하지 못하기 때문에 반드시 settings.gradle 파일을 추가해줘야 합니다.

## build-logic 추가 후 동작 과정
root project를 build할 시 **Gradle**이 **settings.gradle**을 실행하면서 `Settings 객체`가 생성됩니다.  

스크립트 가장 위에 선언되어 있는 `pluginManagement{ } 블록`이 실행됩니다. 내부 블록 중 `includeBuild()`가 실행됩니다.  

root project가 진행되기 전에, build-logic 자체가 Initialization -> Configuration -> Execution 라이프사이클이 진행되어 `compileKotlin`,`jar` 같은 task가 진행됩니다.  

build-logic의 build.gradle에 선언된 plugin에 대한 정보를 기반으로 jar 파일의 META-INF 안에 id와 클래스 매핑 정보가 저장됩니다.    

build-logic 빌드 이후 산출물은 메인 build의 plugin 클래스 패스에 편입됩니다.   

이후 메인 build되면서 필요한 plugin이 보이면 등록했던 해당하는 클래스를 찾고, 해당 클래스의 `apply(target: Project)` 함수가 실행됩니다.  

```kotlin
gradlePlugin {
    plugins {
        register("android-application") {
            id = "androidApplication"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
    }
}
```
> 위 코드를 보면 등록하는 id와 클래스 패스 등을 등록하는 것을 확인할 수 있습니다.