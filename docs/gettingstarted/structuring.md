모드 구조 짜기
====================

한번 모드 파일들을 어떻게 나누고 정리하는지, 그리고 어떤 파일이 무슨 역할을 하는지 살펴보도록 하죠.

패캐지 이름 쓰기
---------

패키지 이름으로는 고유한 이름을 사용하세요, 만약 프로젝트와 연관된 URL 이 있다면 최상위 패키지로 사용하실 수 있습니다. 예를 들어, "example.com" 을 소유하고 계신다면 `com.example` 을 최상위 패키지로 사용하실 수 있습니다. 최상위 패캐지를 만드는 것은 선택사항입니다.

!!! important
    소유하고 계시지 않은 도메인을 최상위 패캐지로 사용하지 마세요. 고유하기만 하다면 도메인 말고도 이메일이나, 이름 또는 닉네임을 사용하실 수도 있습니다.

그 다음으론, 모드만의 고유한 이름을 뒤에 붙입니다, 예를 들어 고유한 이름이 `examplemod` 라면 최종적으로 패캐지는 `com.example.examplemod` 가 될 것입니다.

`mods.toml` 파일
-------------------

!!! important

    마인크래프트 1.16 부터 mods.toml 에 라이센스가 정의되어 있지 않다면 모드가 실행되지 않습니다. 이곳을 참조하여 라이센스를 선택하세요.: https://choosealicense.com/
    모든 라이센스 리스트는 이곳에 나와 있습니다: https://www.olis.or.kr/license/compareGuide.do

이 파일은 모드의 메타데이터를 정의합니다. 이 메타데이터의 일부는 메인 화면의 'Mods' 탭에 표시됩니다. 또한 파일 하나로 모드 여러개를 기술할 수 있습니다. 

`mods.toml` 파일은 [TOML][] 형식으로 작성되어 있습니다, MDK 에 들어있는 예제 `mods.toml` 파일은 파일의 내용을 기술하는여러 주석들이 작성되어 있습니다. 이 파일은 `src/main/resources/META-INF/mods.toml` 에 위치하고 있습니다. 모드 1개를 기술하는 간단한 `mods.toml` 은 다음과 같이 작성하실 수 있습니다:
```toml
    # 사용할 모드로더의 이름입니다. - 대다수의 포지 @Mod 모드들은 javafml 을 사용합니다.
    modLoader="javafml"
    # 사용할 모드 로더의 버전 범위를 지정합니다 - 대다수의 포지 @Mod 모드들은 포지 버전을 사용합니다.
    # 1.16.5 포지 버전은 36 입니다.
    loaderVersion="[36,)"
    # 모드의 라이센스입니다. 라이선스 설정은 필수적이며, 재배포 속성과 관련 권리를 제 3자가 쉽게 이해할 수 있도록 해줍니다.
    # 라이선스 선택은 이곳을 참고하실 수 있습니다: https://choosealicense.com/ 저작권은 기본으로 저작자가 모든 권리를 보유합니다(All rights reserved), 그렇기에 포지에서도 이를 기본값으로 사용합니다.
    license="All Rights Reserved"
    # 모드에 문제가 발생할 시 문제 제기를 할 URL 입니다.
    issueTrackerURL="github.com/MinecraftForge/MinecraftForge/issues"
    # 이 파일에 정의된 모드들이 별도의 리소스팩으로 표시되어야 할지를 정의합니다.
    showAsResourcePack=false

    [[mods]]
      modId="examplemod"
      version="1.0.0.0"
      displayName="예제 모드"
      updateJSONURL="minecraftforge.net/versions.json"
      displayURL="minecraftforge.net"
      logoFile="logo.png"
      credits="저희 부모님에게 감사드리고 싶습니다.."
      authors="Author"
      description='''
      흙을 다이아몬드로 조합할 수 있도록 해줍니다. 이 예제모드는 영겁의 시간동안 이어져 내려온 고대의 전통입니다. 신성한 노치께서 시작하셨고 젭이 환성적인 무지개를 달았으며, 디너본이 뒤집었고...(생략)
      '''

      [[dependencies.examplemod]]
        modId="forge"
        mandatory=true
        versionRange="[36,)"
        ordering="NONE"
        side="BOTH"

      [[dependencies.examplemod]]
        modId="minecraft"
        mandatory=true
        versionRange="[1.16.5,1.17)"
        ordering="NONE"
        side="BOTH"
```

만약 문자열이 `${file.jarVersion}` 처럼 작성되었다면 포지는 런타임 도중 이 문자열을 manifest 의 **Implementation Version** 으로 대체합니다. 모드 개발 환경에서는 manifest 를 읽어들일 모드 jar 가 없기 때문에 `NONE` 이 대신 표시됩니다. 그렇기에 이 필드는 그대로 냅두는 것이 권장됩니다. 아래 표에는 모드가 가질 수 있는 속성들이 표시되어 있습니다, `필수` 로 표시되어 있는 속성은 기본값이 없기 때문에 무조건 정의되어야 합니다. 그렇지 않으면 오류가 날 것입니다.

|            속성 |   타입   | 기본값  | 설명                                                                 |
|--------------:|:------:|:----:|:-------------------------------------------------------------------|
|         modid | string |  필수  | 이 모드의 아이디.                                                         |
|       version | string |  필수  | 모드의 버전, 여러 숫자가 마침표로 분리된 형태여야 합니다, [버전 명명][버전명명] 규칙을 따른다면 더 좋습니다.   |
|   displayName | string |  필수  | 읽기 쉬운 모드 이름.                                                       |
| updateJSONURL | string | `""` | [버전 JSON][자동업데이트] URL.                                             |
|    displayURL | string | `""` | 모드의 홈페이지 URL.                                                      |
|      logoFile | string | `""` | 모드의 로고 파일 이름. 로고 이미지 파일은 리소스 폴더 최상위에 존재하여야 하며, 그 하위 폴더에 존재하면 안됩니다. |
|       credits | string | `""` | 언급하고 싶은 합의사항 또는 공로.                                                |
|       authors | string | `""` | 모드의 저자.                                                            |
|   description | string |  필수  | 모드 설명.                                                             |
|  dependencies | [list] | `[]` | 모드의 종속성 목록.                                                        |

\* 모든 버전 범위는 [Maven 버전 범위 규악][mvr]을 따릅니다.

모드 파일
------------

일반적으로 패키지 안에 이름이 모드 이름인 파일을 만들고 거기서부터 시작합니다. 이 파일은 모드의 *시작 지점*이며, 따로 특수하게 표시를 해 두어 이를 알립니다.

`@Mod` 는 무엇인가요?
-------------

이 어노테이션은 포지 모드 로더에게 클래스가 모드의 시작 지점임을 나타내기 위해 사용합니다. 이 어노테이션의 value 는 `src/main/resources/META-INF/mods.toml` 에 정의된 mod id 와 동일해야 합니다.

하위 패캐지를 사용하여 코드 말끔하게 정리하기
------------------------------------------

모든걸 패키지 하나에, 클래스 하나에 다 밀어넣는 것 보다 하위 패캐지를 만들어 분리하는 것이 권장됩니다.

일반적으로, `common` 과 `client` 하위 패캐지를 만들어 코드를 분리합니다, 이때 `common` 은 서버/클라이언트에서 실행될 코드, `client` 는 클라이언트에서만 실행될 코드를 포함합니다. `common` 패캐지에는 아이템, 블록, 타일 엔티티 입니다(각각 별도의 패캐지에 들어갈 수도 있음). 렌더러나 UI 들은 `client` 패캐지에 들어갈 것입니다.

!!! note

    패캐지를 이렇게 나누는 것이 일반적이나 완전히 선택사항 입니다, 패캐지는 마음대로 나누셔도 됩니다.

하위 패캐지를 사용하여 코드를 나눔으로써 모드를 조금더 정돈하고 조직화하여 만들 수 있습니다.

클래스 이름 규칙
--------------------

클래스 이름 규칙은 클래스가 어디 사용되는지 해석하기 쉽도록 만들어 주며, 뭐가 어디있는지 찾기 쉽도록도 해줍니다.

에를 들어:

* `PowerRing` 이란 `Item` 은 `item` 패캐지에 `PowerRingItem` 이란 이름으로 있을 것입니다.
* `NotDirt` 라는 `Block` 은 `block` 패캐지에 `NotDirtBlock` 이란 이름으로 있을 것입니다.
* `SuperChewer` 라는 `TileEntity` 는 `tile` 또는 `tileentity` 라는 패캐지에 `SuperChewerTile` 이라는 이름으로 있을 것입니다.

클래스 이름 뒤에 클래스의 *종류*를 써놓으시면 나중에 무슨 클래스인지 식별하기 쉽도록 해줍니다.

[라이센스]: https://choosealicense.com/
[TOML]: https://github.com/toml-lang/toml
[버전명명]: ../conventions/versioning.md
[자동업데이트]: autoupdate.md
[mvr]: https://maven.apache.org/enforcer/enforcer-rules/versionRanges.html
