포지 모드 개발 시작하기
==========================

이 페이지는 간단한 모드를 만들 수 있도록 도와주는 가이드입니다. 그렇기에 이 문서의 다른 페이지들을 보시기 전에 이 페이지를 먼저 읽어 보시는걸 권장합니다.

나의 첫 모드 개발하기
--------------------

1. 먼저, Java 17 개발 키트(JDK) 와 64비트 Java 가상 머신 (JVM)을 설치하세요. 마인크래프트와 포지는 둘 다 Java 17로 컴파일 됨으로 이를 사용하시는 것이 좋습니다. [Eclipse Adoptium][jdk]에서 JDK를 다운 받으실 수 있습니다. 32비트를 사용하시면 아래 명시된 Gradle 작업 수행중 문제가 발생할 수 있습니다.
2. [포지 사이트][files]에서 모드 개발 키트 (MDK)를 다운하세요.
3. 다운받은 MDK의 압축을 비어있는 폴더에 해체하도록 하세요. 그러면 여러 파일들과 `src/main/java`에 있는 예제 모드가 있습니다. 모드를 개발할 때 아래 명시된 파일들은 건드실 필요가 있는 경우가 많지 않으며, 여러 모드 프로젝트에서 재사용 하셔도 괜찮습니다.
    * `build.gradle`
    * `gradlew.bat`
    * `gradlew`
    * `settings.gradle`
    * `gradle` 폴더
4. 위 파일들을 새로운 폴더로 옮기세요. 그 폴더는 모드 프로젝트의 폴더가 될 것입니다.
5. IDE를 선택하세요:
    * 포지에서 명시적으로 지원하는 IDE는 Eclipse가 유일합니다. 그러나 Intellij IDEA 또는 Visual Studio Code에서 개발하기 위해 추가적인 run 작업들 또한 제공합니다. 사실 어떤 IDE든 모드 개발에 사용할 수 있도록 만들 수 있긴 합니다.
    * Eclipse와 Intellij IDEA는 Gradle을 관리해 주기 때문에 초기 개발환경 설정을 보다 간편하게 하실 수 있습니다. 그 초기 개발환경 설정에는, 모장 또는 포지와 같은 여러 소프트웨어 공유 사이트에서 패키지를 받는 것이 포함됩니다. VSCode는 `Gradle Tasks` 플러그인을 사용해 초기 개발환경 설정을 관리하도록 할 수 있습니다.
    * 대부분의 상황에선 `build.gradle`을 수정하고 이를 적용하기 위해서는 Gradle을 호출하여 프로젝트를 다시 처리하도록 해야 합니다. 위에서 언급한 두 IDE는 Gradle 패널의 새로고침 버튼으로 프로젝트를 다시 처리 할 수 있습니다.
6. IDE 실행 설정 생성하기:
    * Eclipse: `genEclipseRuns`를 실행하세요. (`gradlew genEclipseRuns`). 이후 프로젝트를 새로고치시면 됩니다.
    * IntelliJ: `genIntellijRuns`를 실행하세요. (`gradlew genIntellijRuns`). 만약 "module not specified" 와 같은 오류가 발생한다면 설정을 수정하여 "main" 모듈을 선택하거나 `ideaModule` 속성을 사용해 직접 메인 모듈을 선택하실 수 있습니다.
    * VSCode: `genVSCodeRuns`를 실행하세요. (`gradlew genVSCodeRuns`). 

내 모드 정보 수정하기
--------------------------------

프로젝트의 `build.gradle`파일을 수정하여 모드가 어떻게 빌드 되어야 하는지 설정할 수 있습니다. (빌드된 모드의 파일 이름, 버전 등). `build.gradle`의 대부분의 내용은 지우거나 수정하셔도 됩니다.

!!! important
    확신 없이면 **절대로** `settings.gradle`을 수정하지 마세요. ForgeGradle 플러그인 적용을 위해 필수적인 내용이 적혀 있습니다. 

### 간단한 `build.gradle` 설정

이 설정들을 모든 프로젝트에 사용하길 매우 권장합니다.

* 빌드된 결과물의 파일 이름을 변경 하시려면 - `archivesBaseName`의 값을 수정하세요.
* "maven coordinates"(메이븐 코디네이트)를 변경 하시려면 - `group`의 값을 수정하세요.
* 버전을 변경 하시려면 - `version`의 값을 수정하세요.
* 모드의 id를 바꾸시려면 - `examplemod`를 전부 모드의 id로 변경하세요.

### 모장 공식 매핑으로 변경하기

포지는 모장의 공식 매핑(*MojMaps으로 불리기도 합니다.*)을 사용합니다. 공식 매핑은 클래스 메서드 및 필드 이름을 제공합니다. 하지만 Javadoc, 함수 인자의 이름은 포함되어 있지 않습니다. 현재 이 매핑을 사용하는 것이 법적으로 안전한지 확실하진 않으나 모장측에서 사용하기를 바라니 포지에서는 이를 사용하기를 결정 하였습니다. [이곳][mojmap]에서 포지의 입장을 읽을 수 있습니다.

내 모드 빌드하고 테스트하기
-----------------------------

1. 모드를 빌드하기 위해서 `gradlew build`. 를 실행하세요. 이는 `build/libs`에 빌드된 Jar 파일을 `[archivesBaseName]-[version].jar`과 같은 형태로 출력 할 것입니다. 이 파일은 포지가 설치된 마인크래프트 폴더의 `mods`에 넣어 실행할 수 있습니다.
2. 모드를 테스트하기 가장 쉬운 방법은 프로젝트를 설정할때 생성된 실행 설정 파일들을 사용하여 게임을 실행하는 것입니다. 또는 `gradlew runClient`를 실행하세요. 그러면 실행 설정에 지정된 `<runDir>`의 위치에서 모드 코드와 함께 마인크래프트가 시작됩니다.
3. 또한 서버 실행 설정을 사용하거나 `gradlew runServer`를 통해 GUI와 함께 마인크래프트 서버가 시작됩니다. 첫 번째 실행 후 서버는 `run/eula.txt`를 수정하여 마인크래프트의 이용약관(EULA)에 동의할 때까지 즉시 종료됩니다. 수락시 서버가 로드되고 `localhost`로 서버에 연결할 수 있습니다.

!!! note
    만약 모드가 전용 서버에서 작동되어야 한다면 전용 서버에서 또한 모드를 테스트 해보십시오.
    
[files]: https://files.minecraftforge.net "포지 파일 배포 사이트"
[jdk]: https://adoptium.net/temurin/releases?version=17 "Temurin 17 JDK 다운받는 곳"
[mojmap]: https://github.com/MinecraftForge/MCPConfig/blob/master/Mojang.md
