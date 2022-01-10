포지 모드 개발 시작하기
==========================

이 페이지는 간단한 모드를 만들 수 있도록 해주는 가이드 입니다. 그렇기에 이 문서의 다른 페이지들을 보시기 전에 이 페이지를 먼저 읽는 것을 권장드립니다.

첫 모드 개발 해보기
--------------------

1. 먼저, Java 8 개발 키트(JDK)와 64비트 Java 가상 머신(JVM)을 다운받으세요. 마인크래프트와 포지는 둘다 Java 8로 컴파일이 가능하기에 이를 사용하시는 것이 좋습니다. 32비트를 사용하시면 아래 명시된 Gradle 작업 수행도중 문제가 발생할 수 있습니다. [AdoptOpenJDK][jdk]에서 다운받으실 수 있습니다.
2. [포지 사이트][files]에서 모드 개발 키트(MDK)를 다운받으세요.
3. 다운받은 MDK 의 압축을 를 비어있는 폴더에 해제하도록 하세요. 그러면 여러 파일들과 `src/main/java` 에 있는 예제 모드가 보이실텐데, 모드 개발할때, 아래 명시된 파일들은 건드실 필요가 있는 경우가 많지 않으며 여러 모드 프로젝트에서 재사용하셔도 괨찮습니다:
    * `build.gradle`
    * `gradlew.bat`
    * `gradlew`
    * `gradle` 폴더
4. 위 파일들을 새로운 폴더로 옮기세요. 그 새 폴더는 모드 프로젝트 폴더가 될 것입니다.
5. IDE 를 고르세요:
    * 포지에서 명시적으로 지원되는 IDE 는 Eclipse 가 유일합니다. 그러나 Intellij IDEA 나 Visual Studio Code 에서 개발하기 위해 추가적인 run 작업들 또한 제공합니다. 사실 어떤 IDE 든 모드 개발에 사용할 수 있도록 만들 수 있긴 합니다.
    * Eclipse 와 Intellij IDEA 는 Gradle 을 관리해 주기 때문에 초기 개발환경 설정을 간단하게 하실 수 있습니다. 그 초기 개발환경 설정에는, 모장 또는 포지와 같은 여러 소프트웨어 공유 사이트에서 패키지를 받는 것이 포함됩니다. VSCode 는 `Gradle Tasks` 플러그인을 사용해 초기 개발환경 설정을 관리하도록 할 수 있습니다.
    * 대부분, 또는 모든 상황에선, build.gradle 을 수정하고 이를 적용하기 위해서는 Gradle 을 호출하여 프로젝트를 다시 처리하도록 해야 합니다. 위에서 언급한 두 IDE 는 Gradle 패널의 새로고침 버튼으로 프로젝트를 다시 처리할 수 있습니다.
7. IDE 실행 설정 생성:
    * 이클립스: `runEclipseRuns` 를 실행하세요(`gradlew genEclipseRuns`). 이는 게임 실행 설정을 생성하고 게임에 필요한 에셋들을 다운받습니다. 이 작업이 끝난 이후 프로젝트를 새로고치세요.
    * Intellij: `genIntellijRuns` 를 실행하세요(`gradlew genIntellijRuns`). 이는 게임 실행 설정을 생성하고 게임에 필요한 에셋들을 다운받습니다. 만약 "module not specified" 와 같은 오류가 난다면 설정을 수정하여 "main" 모듈을 선택하시거나 `ideaModule` 속성을 사용해 모듈을 선택하실 수 있습니다.
    * VSCode: `genVSCodeRuns` 를 실행하세요(`gradlew genVSCodeRuns`). 이는 게임 실행 설정을 생성하고 게임에 필요한 에셋들을 다운받습니다.

모드 정보 수정하기
--------------------------------

`build.gradle` 파일을 수정해 모드가 어떻게 빌드되어야 할지 설정할 수 있습니다(모드 파일 이름, 버전, 등).

!!! important

    `buildscript {}` 부분을 **절때 수정하지 마세요.** 기본적으로 적혀있는 내용은 ForgeGradle 이 동작하기 위해 필요합니다.

`// Only edit below this line, the above code adds and enables the necessary things for Forge to be setup.` 아래 있는 내용들은 거의 수정하셔도 괜찮은 것들입니다.

### 간단한 `build.gradle` 설정

이 설정들은 모든 프로젝트에 권장됩니다.

* 빌드되는 파일의 이름을 변경하시려면 - `archivesBaseName` 의 값을 수정하세요.
* "maven 코디네이트"를 변경하시려면 - `group` 의 값을 수정하세요.
* 모드의 버전을 변경하시려면 - `version` 의 값을 수정하세요.
* 실행 설정에서 mod id를 바꾸시려면 - `examplemod` 를 전부 모드의 id 로 변경하세요.

### 모장 공식 매핑으로 변경하기.

1.16.5 부터 Forge 는 모장의 공식 매핑(MojMaps)을 사용할 것입니다. 공식 매핑은 사용할 모든 필드와 메서드 이름, 그리고 1.17 에서 사용하는 클래스 이름을 포함하고 있습니다. 이때 Javadoc 이랑 함수 인자 이름은 포함되어 있지 않습니다. 현재로썬 이 매핑을 사용하는 것이 법적으로 완전히 안전한지 확실하진 않으나 모장측에서 사용하기를 바라니 포지에서는 이를 사용하기로 결졍하였습니다. 포지팀의 입장은 [이곳][mojmap]에서 더 알아보실 수 있습니다.

만약 이 매핑을 사용하시고 싶지 않으시다면 공식 매핑 이전에 사용하던 MCP 를 대신 이용하실 수 있습니다. MCP 는 일부 메서드, 필드, 그리고 함수 인자 이름과 Javadoc 을 포함하고 있습니다. 다음 MCP 버전은 아마도 마지막 MCP 매핑의 버전이 될 것인데, 왜냐하면 이는 더이상 관리하에 있지 않기 때문입니다.

```groovy
minecraft {
    mappings channel: 'snapshot', version: '20210309-1.16.5'
}
```

모드 빌드 및 테스트
-----------------------------

1. 모드를 빌드하기 위해서는 `gradlew build` 를 실행하세요. 이는 `build/libs` 에 빌드된 Jar 파일을 이름 `[archivesBaseName]-[version],jar` 로 출력할 것입니다. 이 파일은 포지가 설치된 마인크래프트 폴더의 `mods` 에 넣어 실행할 수 있습니다.
2. 모드를 테스트할때 가장 쉬운 방법은, 프로젝트를 설정할때 생성된 실행 설정 파일을 사용하여 게임을 실행하는 것입니다. 이때 게임은 `<runDir>` 에 실행되며, 이때 실행 설정에 정의된 모드 소스 코드들과 함께 실행됩니다. MDK 는 기본으로 `main` 소스셋을 포함하니, `src/main/java` 에 작성하신 코드들은 게임 실행시 사용할 코드에 포함됩니다.
3. 또한 `gradlew runServer` 를 통해 전용 서버를 실행하실 수도 있습니다. 전용 서버를 이렇게 실행하시면 GUI 와 함께 마인크래프트 서버를 실행합니다. 최초 실행시 EULA 동의를 위해 서버가 바로 꺼지는데, `run/eula.txt` 에 동의하신 이후에는 서버가 정상적으로 실행될 것입니다. 이후 주소 `localhost` 를 통해 접속하실 수 있습니다.

!!! note

    만약 모드가 전용 서버에서도 사용되어야 한다면, 전용 서버에서도 모드를 테스트해보는 것이 늘 권장됩니다.
    
[files]: https://files.minecraftforge.net "포지 파일 배포 사이트"
[jdk]: https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=hotspot "AdoptOpenJdk 8 다운로드 페이지"
[mojmap]: https://github.com/MinecraftForge/MCPConfig/blob/master/Mojang.md
