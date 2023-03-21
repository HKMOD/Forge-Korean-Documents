언어 생성
===================

모드의 [언어 파일들][lang]은 `LanguageProvider` 의 하위 클래스를 만들고 `#addTranslation`을 구현하는 것으로 자동으로 생성할 수 있습니다. 각 `LanguageProvider` 하위 클래스들은 언어 한개의 언어 파일들을 형성합니다. 하위 클래스를 구현한 이후, `DataGenerator` 에 [등록][datagen]되어야 합니다.

`LanguageProvider`
------------------

`LanguageProvider` 의 하위 클래스들은 궁극적으로는 각 번역 키들을 언어에 맞는 문자열로 바꾸는 `Map` 입니다. 각 번역 키들은 `#add`를 사용해 추가할 수 있습니다. 이뿐만 아니라 `Block`, `Item`, `ItemStack`, `Enchantment`, `MobEffect`, 그리고 `EntityType` 의 번역 키 값을 쓰는 메서드들도 있습니다.

```java
// LanguageProvider#addTranslations
this.addBlock(EXAMPLE_BLOCK, "예제 블록");
this.add("object.examplemod.example_object", "예제 오브젝트");
```

[lang]: ../../concepts/internationalization.md
[locale]: https://minecraft.fandom.com/wiki/Language#Languages
[datagen]: ../index.md#데이터-생성하기
