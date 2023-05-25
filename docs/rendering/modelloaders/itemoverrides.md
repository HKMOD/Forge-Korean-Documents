`ItemOverrides`
==================

`ItemOverrides`는 [`BakedModel`][baked]이 `ItemStack`의 상태에 따라 새로운 `BakedModel`을 형성할 수 있도록 해 줍니다; 그리고 이렇게 새로 형성한 모델은 기존 모델 대신 렌더링에 사용됩니다. 이 시스템은 아이템 오버라이드라고 불리며, `ItemOverrides`라는 클래스로 표현됩니다. 이는 당기면 활시위 모양이 바뀌는 활처럼 동적인 모델을 구현할 때 유용합니다.

### `ItemOverrides()`

`List<ItemOverride>`를 전달받으면 `ImmutableList<BakedOverride>`로 변환합니다, `#getOverrides`를 호출하여 해당 `ImmutableList<BakedOverride>`에 접근할 수 있습니다. 

### `resolve`

`BakedModel`, `ItemStack`, `ClientLevel`, `LivingEntity`, 그리고 `int`를 전달받아 렌더링할 때 또 다른 `BakedModel`을 생성합니다. 아이템 속성에 따라 외관을 바꾸는 작업을 여기서 합니다.

이때 전달되는 레벨은 변경하지 마세요.

### `getOverrides`

`ItemOverrides`가 사용하는 모든 [`BakedOverride`][override]를 포함하는 불변 리스트를 반환합니다. 쓰는 것이 없다면 빈 리스트를 반환할 수도 있습니다.

## 바닐라 마인크래프트에서의 아이템 오버라이드
바닐라 마인크래프트는 아이템의 외간에 영향을 끼칠 수 있는 아이템의 속성을 실숫값 하나로 표현하도록 하여 모델 JSON에 정의된 조건에 따라 다른 모델을 선택할 수 있도록 합니다. 예를 들어 활시위를 당긴 시간에 따라 모델을 바꾼다고 한다면, 활시위를 당긴 시간이 해당 실숫값이 될 수 있을 것이고, 아이템의 내구도에 따라 모델이 바뀐다고 한다면 내구도가 해당 실숫값이 될 수 있습니다. 어떻게든 결과로 실숫값이 나오기만 한다면 무엇이 되든 상관없습니다. 이후 마인크래프트는 이 값이 모델 JSON에 정의한 값보다 큰지 비교하며, 만약 JSON의 값보다 더 크다면 다른 모델을 사용합니다.

## `ItemPropertyFunction`
`ItemPropertyFunction`는 함수 `(ItemStack, ClientLevel, LivingEntity, int)` -> `float`를 표현하는 함수형 인터페이스입니다. 아이템의 여러 가지 속성값을 활용해서 하나의 실숫값을 만듭니다. 모든 `ItemPropertyFunction`는 사전에 `ItemProperties#register`를 통해 등록되어야 합니다.

## `BakedOverride`
이 클래스는 아이템의 속성을 `ItemPropertyFunction`를 통해 실수로 변환한 값들과, 조건 만족시 대신 사용할 모델을 담습니다. 해당 클래스의 인스턴스는 `ItemStack`마다 다릅니다. 

아래는 아이템 오버라이드를 활용하는 모델의 예시입니다
```js
{
  // 모델 JSON 파일 내부
  "overrides": [
    // 아래 객체는 ItemOverride가 됩니다 
    {
      // predicate는 런타임에서 Map<ResourceLocation, Float>를 구성합니다.
      "predicate": {
        // predicate는 다수의 ResourceLocation과 실숫값의 쌍으로 이루어져 있습니다. 
        // 여기서 사용되는 ResourceLocation은 게임이 사전에 등록한 ItemPropertyFunction의 레지스트리 이름입니다
        // 이 ResourceLocation으로 지정한 ItemPropertyFunction은 이 모델 파일을 사용하는 아이템의 여러 속성들을 하나의 실숫값으로 변환합니다
        // 그리고 그 실숫값이 0.5보다 크다면 다른 모델을 사용합니다, 여기선 "example1:item/model"을 사용합니다
        "example1:prop": 0.5,
      },
      // 위에서 정의한 조건을 만족하면 사용할 모델을 지정합니다
      "model": "example1:item/model"
    },
    // 아래 객체 또한 ItemOverride가 됩니다
    {
      "predicate": {
        "example2:prop": 1,
        // 이때 여러 개의 ItemPropertyFunction을 지정할 수 있습니다
        // predicate에 정의된 모든 ItemPropertyFunction을 만족해야만 다른 모델을 사용합니다
        "example2:prop1": 0.7,
        "example2:prop2": 0.8,
      },
      // 위에서 정의한 3개의 조건을 다 만족하면 사용할 모델을 지정합니다
      "model": "example2:item/model"
    }
  ]
}
```

[baked]: ./bakedmodel.md
[override]: #bakedoverride
