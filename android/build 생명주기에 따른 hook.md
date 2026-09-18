# build 생명주기에 따른 hook

[참고자료](https://docs.gradle.org/current/userguide/build_lifecycle.html)

<img width="2900" height="2100" alt="image" src="https://github.com/user-attachments/assets/9faf941e-eaee-41fe-afe8-23f79476eddf" />      

위 그림과 같이 build 시 기본 생명주기는 **`Initialization`** -> **`Configuration`** -> **`Execution`** 단계로 진행됩니다.

<details>
<summary><b>Initialization 단계</b></summary>

Initialization 단계에서 사용할 수 있는 hook 종류

- **Init Script** : 전역 변수 설정
- `gradle.beforeSettings` : settings 파일이 실행되기 전에 실행
- `pluginManagement{ ... }` : 반드시 처음으로 실행되어야 하며, plugin repo와 버전, rules를 정의
- `plugins{ ... }` : settings 객체에 plugin을 적용 (e.g.. Build Scans)
- **Settings Script Body** : 정의된 build 구조를 평가
- `gradle.settingsEvaluated` : Settings가 진행되며 Project 객체들이 생성됨
- `gradle.projectsLoaded` : 모든 Project 객체들이 존재는 하지만 설정되지 않은 상태

</details>

<details>
<summary><b>Configuration 단계</b></summary>

Configuration 단계에서 사용할 수 있는 hook 종류

- `gradle.lifecycle.beforeProject` : 각 project들이 평가되기 전 발생 (추천x)
- `project.beforeEvaluate` : 특정 project의 build script가 평가되기 전 실행
- `buildscript { ... }` : build script에서 classpath를 설정
- `plugins{ ... }` : plugins을 적용하고 DSL과 Tasks를 추가
- **Build Script Body** : task들을 등록하고 property들을 설정
- `project.afterEvaluate` : 특정 project가 평가된 후 실행. 또 다른 프로젝트의 configuration을 확인할 때 유용 (추천x)
- `gradle.lifecycle.afterProject` : 각 프로젝트의 평가가 끝난 후 실행
- `gradle.projectsEvaluated` : 모든 project script가 실행 (Task Graph를 변경할 수 있는 마지막 기회)
- **Task Graph Construction** : gradle이 DAG을 기반으로 요청된 task들을 계산
- `gradle.taskGraph.whenReady` : Task Graph가 채워지고 난 후에 실행

</details>

<details>
<summary><b>Execution 단계</b></summary>

Execution 단계에서 사용할 수 있는 hook 종류

- **Task Input Snapshotting** : 각 task들이  넘어가야 할 지 혹은 실행되어야 할지를 결정  
> `task.onlyIf` : onlyIf 이후 블록의 내용이 true일 경우 task를 실행
- **Dependency Graph Resolution** : 선언된 dependency들을 가지고 dependency graph 구축
- **Artifact Resolution** : repository들로부터 dependency들을 다운로드
- `task.doFirst` : task 실행 시작 전에 실행
- **Task Execution** : Task 실행
- `task.doLast` : task 실행 종료 이후 실행

> [!WARNING]
> Task Input Snapshotting은 Task별로 반복  
> 그 이후 과정은 실제로 필요해지는 시점에 실행됨

</details>

## 요약
<img width="860" height="1300" alt="image" src="https://github.com/user-attachments/assets/7d7b12e9-4896-485b-a3fe-87a3d8369b0c" />    

Initialization 단계에서 전역 변수를 설정하고, 정의된 build 구조를 평가합니다.   
Configuration 단계에서는 task들을 등록하고 property들을 설정합니다.  
Execution 단계에서는 설정된 정보를 기반으로 실행합니다.   


각 단계 사이사이에 hook이 존재하고, hook을 통해 각 단계 전후로 여러 설정할 수 있습니다.