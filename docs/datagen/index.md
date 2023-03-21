데이터 생성기
===============

데이터 생성기는 코드를 사용해 모드에 필요한 에셋과 데이터를 만드는 방법입니다.  이를 통해 에셋과 데이터 파일들의 내용을 코드에서 정의하고 문법이나 규격에 상관없이 편하게 생성할 수 있도록 해줍니다.

데이터 생성 시스템은 메인 클래스 `net.minecraft.data.Main` 에서 불러옵니다. 게임 실행 명령 인수를 바꿔 어떤 모드의 데이터를 수집할지, 어떤 파일들을 고려할지 등을 설정하실 수 있습니다. 데이터 생성을 관리하는 클래스는 `net.minecraft.data.DataGenerator` 입니다.

MDK 의 `build.gradle` 은 기본적으로 `runData`를 추가하여 데이터 생성기를 실행할 수 있도록 해줍니다.

이미 존재하던 파일
--------------

데이터 생성을 통해 만들어지지 않은 모든 텍스쳐 및 데이터들을 참조하려면, 일단 그 데이터들이 시스템에 존재하긴 하는지를 확인하여야 합니다, 이는 이러한 참조들이 유효함을 확인하기 위함이며, 레지스트리 이름 등에 발생한 오타를 찾기 편하게 해줍니다.

`ExistingFileHelper` 는 해당 데이터들이 존재하는지 확인하는 클래스입니다. 이 클래스의 인스턴스는 `GatherDataEvent#getExistingFileHelper`를 통해 받을 수 있습니다.

명령 인수 `--existing <folderpath>` 는 데이터 생성중 지정된 폴더, 그리고 그 하위 폴더들에 있는 데이터들을 사용할 수 있도록 합니다. 추가적으로, 명령 인수 `--existing-mod <modid>` 는 데이터 생성중 지정된 모드의 리소스들을 사용할 수 있도록 합니다. 이러한 인수들을 지정하지 않았다면 오직 마인크래프트 기본 데이터팩과 리소스팩만이 `ExistingFileHelper` 에 등록됩니다.

생성 모드
---------------

데이터 생성기는 4가지 종류의 데이터를 생성할 수 있습니다, 어떤 종류의 데이터를 생성할 지는 게임 실행 명령 인수를 통해 설정할 수 있고, 어떤 데이터를 생성하려고 하는지는 `GatherDataEvent#include***` 메서드로 확인하실 수 있습니다.

* __클라이언트 에셋__
  * `asset` 에 들어가는 클라이언트 전용 파일들을 생성합니다: 블록/아이템 모델, BlockState 정의 JSON 파일들, 언어 파일 등.
  * __`--client`__, `#includeClient`
* __서버 데이터__
  * `data` 안에 들어가는 서버 전용 데이터를 생성합니다: 조합법, 도전과제, 태그 등.
  * __`--server`__, `#includeServer`
* __개발 도구__
  * 개발 도구들을 실행합니다: SNBT를 NBT 로 바꾸기, 그 반대로 바꾸기 등.
  * __`--dev`__, `#includeDev`
* __보고서 작성__
  * 등록된 모든 블록, 아이템, 명령어등을 덤프합니다.
  * __`--reports`__, `#includeReports`

모든 종류의 데이터를 생성하려면 `-all`을 사용하세요.

데이터 생성하기
--------------

데이터를 생성하기 위해서는, 데이터 제공자가 필요한데, 이는 어떤 데이터가 생성되고 제공될 지를 정의하는 클래스 입니다. 모든 데이터 제공자들은 `DataProvider`를 구현해야 합니다. 마인크래프트는 대부분의 에셋과 데이터에 사용하기 위한 데이터 제공자를 추상화 클래스로 제공하고 있습니다, 그렇기에 모드 개발자들은 정해진 메서드 몇개만 재정의하면 됩니다.
`GatherDataEvent` 는 데이터 생성기가 초기화 될 때 모드 버스에 방송됩니다, `DataGenerator` 인스턴스를 이 이벤트에서 접근하실 수 있습니다. `DataGenerator#addProvider`를 사용해 데이터 제공자를 만들고 등록하세요.

### 클라이언트 에셋

* `net.minecraftforge.common.data.LanguageProvider` - 언어 문자열 생성: `#addTranslations`를 재정의하세요
* `ModelProvider<?>` - 모든 모델 제공자들의 기반이 되는 클래스
  * _이 클래스들은 `net.minecraftforge.client.model.generators` 패캐지에 있습니다_
  * `ItemModelProvider` - 아이템 모델 생성; `#registerModels`를 재정의하세요
  * `BlockStateProvider` - BlockState 정의, 블록 모델, 아이템 모델 생성: `#registerStatesAndModels`를 재정의하세요
  * `BlockModelProvider` - 블록 모델 생성; `#registerModels`를 재정의하세요

### 서버 데이터

* [`net.minecraftforge.common.data.GlobalLootModifierProvider`][glmgen] - [전체 전리품 수정 데이터 생성][glm]; `#start`를 재정의하세요
* _아래 클래스들은 `net.minecraft.data` 패키지에 있습니다_
* [`LootTableProvider`][loottablegen] - [컨테이너 전리품 테이블 생성][loottable]; `#getTables`를 재정의하세요
* [`RecipeProvider`][recipegen] - [조합법][recipes]과 조합법이 해금하는 도전과제 데이터 생성; `#buildCraftingRecipes`를 재정의하세요
* [`TagsProvider`][taggen] - [태그 데이터 생성][tags]; `#addTags`를 재정의하세요
* [`AdvancementProvider`][advgen] - [도전과제 데이터 생성][advancements]; `#registerAdvancements`를 재정의하세요

[langgen]: ./client/localization.md
[lang]: https://minecraft.fandom.com/wiki/Language
[soundgen]: ./client/sounds.md
[sounds]: https://minecraft.fandom.com/wiki/Sounds.json
[modelgen]: ./client/modelproviders.md
[models]: ../resources/client/models/index.md
[itemmodelgen]: ./client/modelproviders.md#itemmodelprovider
[blockmodelgen]: ./client/modelproviders.md#blockmodelprovider
[blockstategen]: ./client/modelproviders.md#block-state-provider
[glmgen]: ./server/glm.md
[glm]: ../resources/server/glm.md
[loottablegen]: ./server/loottables.md
[loottable]: ../resources/server/loottables.md
[recipegen]: ./server/recipes.md
[recipes]: ../resources/server/recipes/index.md
[taggen]: ./server/tags.md
[tags]: ../resources/server/tags.md
[advgen]: ./server/advancements.md
[advancements]: ../resources/server/advancements.md
