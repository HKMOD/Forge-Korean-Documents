도전 과제 생성
======================

[도전 과제][도전과제]는 `AdvancementProvider`의 `#registerAdvancements`를 오버라이드하여 생성할 수 있습니다. 도전 과제 객체를 직접 만드실 수도 있지만, `Advancement$Builder`를 이용하시는 것이 더 편리합니다.
도전 과제 생성기를 구현하셨다면, 이를 `DataGenerator`를 이용해 [등록][데이터생성]하셔야 합니다.

`Advancement$Builder`
---------------------

`Advancement$Builder`는 `Advancement`를 객체를 생성해 주는 유틸리티 입니다. 도전 과제끼리 상관 관계, 사용자에게 표시할 정보, 도전 과제의 보상, 그리고 도전 과제 해금 조건을 정의할 수 있도록 해 줍니다. 도전 과제를 정의할 땐 오직 해금 조건만 지정해 주면 됩니다.

그래도 알아두시면 좋을만한 메서드 들은 다음과 같습니다:

|     메서드 이름     | 설명                                                                                                                                                                                     |
|:--------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    `parent`    | 도전 과제 끼리의 상관 관계를 지정합니다. 도전 과제의 레지스트리 이름 또는 `Advancement`의 인스턴스를 인자로 받습니다.                                                                                                              |
|   `display`    | 채팅, 토스트 알림, 그리고 도전 과제 메뉴에서 사용자에게 표시할 정보를 지정합니다.                                                                                                                                        |
|   `rewards`    | 도전 과제 달성 보상을 지정합니다.                                                                                                                                                                    |
| `addCriterion` | 도전 과제 해금 조건을 추가합니다.                                                                                                                                                                    |
| `requirements` | 도전 과제 해금 조건을 설정합니다. 모든 조건들을 만족하도록(`RequirementsStrategy#AND`), 또는 하나만 만족해도 되도록(`RequirementsStrategy#OR`) 설정하실 수 있으며, 직접 조건 목록을 전달하거나 `RequirementsStrategy`를 구현하여 더 상세하게 설정하실 수 있습니다. |

`Advancement$Builder`를 다 설정하셨다면, `#save`를 호출하여 `Advancement` 객체를 생성하실 수 있습니다. `#save`는 도전 과제를 저장하는 `Consumer<Advancement>`와, 도전 과제의 레지스트리 이름, 그리고 다른 도전 과제들 끼리의 상관 관계를 확인하기 위한 `ExistingFileHelper`를 인자로 받습니다. `AdvancementProvider#registerAdvancements`에 전달된 것을 그대로 사용하셔도 됩니다.

```java
// AdvancementProvider#registerAdvancements(writer, fileHelper) 오버라이드 예시
Advancement example = Advancement.Builder.advancement()
  .addCriterion("example_criterion", triggerInstance) // 도전 과제 해금 조건
  .save(consumer, name, fileHelper);
}
```

[도전과제]: ../../resources/server/advancements.md
[데이터생성]: ../index.md#데이터-생성기
