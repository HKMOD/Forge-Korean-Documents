다국어 지원
=====================================

마인크래프트는 다국어를 국제화(Internationalization, i18n라고 하기도 합니다.)와 현지화를 통해 지원합니다. 국제화는 다양한 언어를 지원할때 코드의 변경이 필요하지 않도록 하는 설계 방법입니다. 현지화는 표시되는 글자들을 유저의 언어에 맞게 바꾸는 과정입니다.

국제화와 현지화는 문자열 치환을 통해 이루어집니다. 국제화는 _번역 키값_을 사용하는데, 번역 키값은 표시 가능한 텍스트로 치환 가능한 문자열입니다. 예를 들어, 흙 블록의 이름 "Dirt" 는 `block.minecraft.dirt` 라는 번역 키값을 표시 가능한 텍스트로 치환하여 얻을 수 있습니다. 이를 통해 표시 가능한 텍스트들을 사용자의 언어에 관계없이 참조할 수 있으며, 새로운 언어를 지원하기 위해 게임의 코드를 수정할 필요가 없습니다.

이때 번역 키값을 표시 가능한 텍스트로 치환하는 과정을 현지화라고 합니다. 이는 게임의 언어 설정을 따라 진행됩니다. 클라이언트에서는 언어 설정에서 다른 국가의 언어를 사용할 수 있지만, 전용 서버에서는 오직 `en_us` (영어)만이 지원됩니다. 지원되는 언어들은 [마인크래프트 위키][언어]에서 확인하실 수 있습니다.

언어 파일
--------------

마인크래프트가 지원하는 언어들은 그 언어용 언어 파일이 각각 존재하고, 이 언어 파일들은 `asset/[네임 스페이스]/lang/[언어 코드].json` 에 위치합니다. (예: `examplemod`를 모드 아이디로 가지는 모드의 한국어 언어 파일은 `assets/examplemod/lang/ko_kr.json` 에 위치합니다.) 이 파일은 번역 키값을 실제 텍스트로 변환해주는 간단한 json 파일 입니다. 언어 파일들의 인코딩은 무조건 UTF-8이어야 합니다. 구버전 .lang 파일들은 [변환기][변환기]를 통해 .json으로 변환할 수 있습니다. 각 줄은 `<번역 키값>`:`<텍스트>` 형식으로 작성되어 있습니다.

```js
{
  "item.examplemod.example_item": "예시 아이템 이름",
  "block.examplemod.example_block": "예시 블록 이름",
  "commands.examplemod.examplecommand.error": "예시 커맨드 오류!"
}
```

블록과 아이템에서 사용하기
---------------------------

블록, 아이템, 도전과제와 같은 게임의 여러 요소들은 이름의 형태가 정해져있는데, 아이템은 `item.<네임 스페이스>.<경로>`, 블록은 `block.<네임 스페이스>.<경로>` 와 같은 형태를 가지고 있습니다. 이러한 형태는 각 클래스의 `#getDescriptionId` 메서드에서 정합니다. 또, 아이템은 `#getDescriptionId(ItemStack)` 메서드도 있는데, 이 메서드를 재정의하여 아이템의 NBT에 따라 다른 번역 키값을 사용하실 수도 있습니다.

`#getDescriptionId` 는 대개 레지스트리 이름의 겹점(:)을 점으로 대체한 것(예: "examplemod:example_item" -> "examplemod.example_item") 앞에 `block.` 이나 `item.` 이 붙은 문자열을 번역 키값으로 반환합니다(예: "item.examplemod.example_item"). `BlockItem` 은 `Item#getDescriptionId`를 재정의하여 자신이 상징하는 블록의 번역 키값을 대신 사용하도록 되어 있습니다.

아이템 `examplemod:example_item` 의 이름은 다음처럼 지정하실 수 있습니다:

```json
{
  "item.examplemod.example_item": "예시 아이템 이름"
}
```

!!! note
    번역 키값은 텍스트 식별을 위한 것일 뿐입니다, 이를 레지스트리 객체를 구분하는데 사용하지 마세요, 꼭 레지스트리 이름을 대신 사용하세요!

현지화 관련 메서드들
--------------------

!!! warning
    많은 모드 개발자들이 자주 하는 실수중 하나는 서버에서 번역 키값을 미리 텍스트로 치환한 다음 클라이언트에 전송하는 것 입니다.

    서버의 언어 설정은 클라이언트와 다를 수 있기 때문에, 서버는 `TranslatableComponent` 와 번역 키값을 사용하여 클라이언트가 스스로 언어 설정에 맞는 텍스트를 표현할 수 있도록 하여야 합니다.

### `net.minecraft.client.resources.language.I18n` (클라이언트 전용)

**이 클래스는 클라이언트에만 존재합니다!** 이를 서버에서 사용하려고 하면 예외가 발생하고 게임이 충돌합니다.

- `get(String, Object...)` 는 번역 키값으로 치환한 텍스트에 포매팅까지 적용하는 메서드입니다. 첫번째 인자는 번역 키값이고, 나머지는 `String.format(String, Object...)` 의 `Object` 가변 인자와 동일합니다.

### `TranslatableComponent`

`TranslatableComponent` 는 문자열 치환과 포매팅이 나중에 이루어지는 `Component` 입니다. 플레이어에게 메세지를 보낼 때 매우 유용한데, 클라이언트에서 직접 언어 설정에 맞는 문자열로 번역 키값을 수행하도록 할 수 있기 때문입니다.

`TranslatableComponent(String, Object...)` 도 `String.format(String, Object...)` 와 비슷합니다; 첫번째 인자는 번역 키값, 뒤에 오는 `Object` 가변인자는 포매팅에 사용됩니다. 지원되는 서식 지정자는 `%s`랑 `%1$s`, `%2$s`, `%3$s` 등이 있습니다. 포매팅에 사용되는 인수에 또 다른 `Component`를 사용할 수 있으며, 이는 해당 `Component` 가 가지고 있는 서식 지정자와 같은 속성들을 유지하며 대상 텍스트에 삽입합니다.

### `TextComponentHelper`

- `createComponentTranslation(CommandSource, String, Object...)` 는 포매팅과 문자열 치환이 이미 이루어진 `BaseComponent`를 생성합니다. 이때, 만약 `CommandSource` 가 포지가 설치되지 않은 클라이언트면 즉시 영문 텍스트를 생성하지만, 그렇지 않다면 `TranslatableComponent`를 통해 나중에 텍스트를 생성하도록 합니다. 이는 서버에 포지가 설치되지 않은 클라이언트가 있을 때 말고는 유용하지 않습니다. 이 메서드의 두번째

[언어]: https://minecraft.fandom.com/ko/wiki/%EC%96%B8%EC%96%B4
[변환기]: https://tterrag.com/lang2json/
