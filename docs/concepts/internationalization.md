국제화와 현지화
=====================================

국제화(Internationalization, i18n라고 하기도 합니다.)는 다양한 언어를 지원할때 코드상 변경이 필요하지 않도록 하는 설계 방법입니다. 현지화는 표시되는 글자들을 유저의 언어에 맞게 바꾸는 과정입니다.

국제화는 _번역 키값_ 을 통해 구현됩니다. 번역 키값은 언어에 상관없이 표시 가능한 텍스트를 식별하는 문자열 입니다. 예를 들어, `block.minecraft.dirt`는 흙 블록의 이름을 뜻하는 문자열 입니다. 이를 통해 표시 가능한 텍스트들을 언어에 관계없이 참조할 수 있습니다. 이때 코드 변경은 필요하지 않습니다.

현지화는 게임의 언어 파일을 통해 이루어 집니다. 마인크래프트는 언어 설정을 통해 사용할 언어 파일을 지정합니다. 서버 환경에서는 오직 `en_us`(영어)만이 지원됩니다. 지원되는 언어들은 [마인크래프트 위키][언어]에서 확인하실 수 있습니다.

언어 파일
--------------

언어 파일들은 `asset/[네임스페이스]/lang/[언어 코드].json` 에 위치합니다. (예: `examplemod`의 영어 파일은 `assets/examplemod/lang/en_us.json`에 위치합니다.) 이 파일은 번역 키값을 실제 텍스트로 사상해주는 간단한 json 파일 입니다. 언어 파일들의 인코딩은 무조건 UTF-8이어야 합니다. 구버전 .lang 파일들은 [변환기][변환기]를 통해 .json으로 변환될 수 있습니다.

```json
{
  "item.examplemod.example_item": "예시 아이템 이름",
  "block.examplemod.example_block": "예시 블록 이름",
  "commands.examplemod.examplecommand.error": "예시 커맨드 오류!"
}
```

블록과 아이템에서 사용하기
---------------------------

블록, 아이템, 그리고 다른 몇몇의 마인크래프트 클래스들은 이름을 표시하기 위해 번역 키값이 내장되어 있습니다. 이 번역 키값들은 `#getDescriptionId`를 오버라이딩하여 지정됩니다. 또, 아이템은 `#getDescriptionId(ItemStack)` 메서드도 있는데, 이를 오버라이딩 하여 아이템의 NBT에 따라 다른 번역 키값을 사용하실 수 있습니다.

`#getDescriptionId`는 기본값으로 겹점(:)이 점으로 대체된 레지스트리 이름(예: "examplemod:example_item" -> "examplemod.example_item") 앞에 `block.` 이나 `item.`가 붙어진 값을 반환합니다(예: "item.examplemod.example_item"). `BlockItem`은 이 메서드를 오버라이딩하여 자신이 상징하는 블록의 번역 키값을 사용하도록 되어 있습니다. 예를 들어 `examplemod:example_item`를 레지스트리 이름으로 가지는 아이템에 이름을 부여하는 방법은 언어 파일에 다음과 같은 줄을 추가하는 것입니다:

```json
{
  "item.examplemod.example_item": "예시 아이템 이름"
}
```

!!! note
    번역 키값은 국제화를 위한 것일 뿐입니다, 이를 이용한 객체 식별과 같은 로직 구현은 권장되지 않습니다. 이때 레지스트리 이름을 대신 사용하세요!

현지화 관련 메서드들
--------------------

!!! warning
    많은 모드 개발자들이 자주 하는 실수중 하나는 서버에서 클라이언트의 현지화를 대신 하는 것 입니다. 서버는 오직 자신의 언어 설정에 따른 현지화만 할 수 있으며, 이 설정은 접속한 클라이언트들의 설정과 다를 수 있습니다.

    그렇기에, 서버는 `TranslationTextComponent`와 번역 키값을 사용하여 클라이언트가 스스로 언어 설정에 맞는 텍스트를 표현하도록 하여야 합니다.

### `net.minecraft.client.resources.I18n` (클라이언트 전용)

**이 클래스는 클라이언트에만 존재합니다!** 이는 클라이언트에서만 실행되는 코드에서 사용하고자 만든 것입니다. 이를 서버에서 사용하려고 하면 예외와 함께 튕깁니다.

- `get(String, Object...)` 포매팅을 사용하여 번역 키값의 텍스트를 불러옵니다. 첫번째 인자는 번역 키값이고, 나머지는 `String.format(String, Object...)`의 포매팅 인자와 동일합니다.

### `TranslationTextComponent`
`TranslationTextComponent`는 현지화와 포매팅이 개으르게 진행되는 `ITextComponent` 입니다. 플레이어에게 메세지를 보낼 때 매우 유용한데, 클라이언트에서 언어 설정에 맞는 현지화를 스스로 진행하기 때문입니다.

`TranslationTextComponenet(String, Object...)`의 첫번째 인자는 번역 키값입니다. 그리고 뒤에 오는 것들은 포매팅에 사용됩니다. 지원되는 서식 지정자는는 `%s`랑 `%1$s`, `%2$s`, `%3$s` 등이 있습니다. 포매팅에 사용되는 인수에 다른 `ITextComponent`를 사용할 수 있으며, 이는 서식 지정자와 같은 속성들을 유지하며 기존 텍스트에 삽입됩니다.

### `TextComponentHelper`
!!! note
    이 부분은 문서 개선이 필요하니 원문과 함께 올려놓겠습니다. 

- `createComponentTranslation(ICommandSource, String, Object...)` 는 포매팅과 현지화가 이미 이루어진 `TextComponent`를 수신자에 맞춰 생성합니다. 만약 수신자가 바닐라 클라이언트면 즉각적으로 초기화가 이루어 지나, 그렇지 않다면 `TranslationTextComponent`를 통해 개으르게 이루어 집니다. 이는 서버가 클라이언트의 접속을 허락하여야 할 때만 유용합니다. (문서 수정이 필요함, 원문은 아래)
- `createComponentTranslation(ICommandSource, String, Object...)` creates a localized and formatted `TextComponent` depending on a receiver. The localization and formatting is done eagerly if the receiver is a vanilla client. If not, the localization and formatting is done lazily with a `TranslationTextComponent`. This is only useful if the server should allow vanilla clients to connect.

[언어]: https://minecraft.fandom.com/ko/wiki/%EC%96%B8%EC%96%B4
[변환기]: https://tterrag.com/lang2json/
