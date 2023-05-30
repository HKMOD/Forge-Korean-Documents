시점
===========

[`BakedModel`][bakedmodel]을 아이템으로 렌더링할 때, 아이템을 렌더링하는 시점이 어느 시점이냐에 따라 다르게 렌더링할 수도 있습니다. 이때 "시점"은 모델을 렌더링하는 맥락을 의미하는데, 마인크래프트에서 사용하는 시점들은 `ItemTransforms$TransformType`에 열거되어 있습니다. 시점을 다루는 데는 두 가지 방법이 있는데: `BakedModel#getTransform`, `ItemTransforms`, 그리고 `ItemTransform`을 활용하는 마인크래프트 로직을 그대로 활용하시는 것과, 이를 대체하는 포지의 `IForgeBakedModel#handlePerspective`를 사용하는 것 입니다. 기존 마인크래프트 코드는 가능하면 `handlePerspective`를 대신 사용하도록 패치되었습니다.

`TransformType`
---------------

`NONE` - 시점 처리 안 함.

`THIRD_PERSON_LEFT_HAND`/`THIRD_PERSON_RIGHT_HAND`/`FIRST_PERSON_LEFT_HAND`/`FIRST_PERSON_RIGHT_HAND` - `FIRST_PERSON_*`은 플레이어가 일인칭에서 손에 아이템을 직접 들고 있을 때, `THIRD_PERSON_*`은 해당 아이템을 삼인칭으로 관찰할 때 사용됩니다.

`HEAD` - 플레이어가 아이템을 머리에 착용하고 있는 것을 삼인칭으로 관찰할 때 사용합니다(에: 호박).

`GUI` - 아이템을 `Screen` 내에서 렌더링할 때 사용합니다.

`GROUND` - 아이템을 `ItemEntity`를 통해 레벨에 렌더링할 때 사용합니다.

`FIXED` - 아이템 액자에 렌더링할 때 사용합니다.

바닐라 로직
---------------

마인크래프트는 `BakedModel#getTransforms`을 통해 시점을 관리합니다. 이 메서드는 `ItemTransforms`를 반환하는데, 이는 여러 `ItemTransform`을 `public final` 필드로 저장하는 객체입니다. `ItemTransform`은 회전, 이동, 크기 등 모델에 적용할 시점 변환 행렬을 표현합니다. `ItemTransforms`는 `NONE`을 제외한 각 시점마다 알맞은 `ItemTransform`을 저장합니다, 만약 `#getTransform`에 인자`NONE`을 전달하신다면 아무런 변형도 가하지 않는 `ItemTransform#NO_TRANSFORM`이 반환됩니다. 

마인크래프트의 시점 관리 시스템은 전체적으로 포지에 의해 대체되었으며, `BakedModel`은 `#getTransforms`로 `ItemTransforms#NO_TRANSFORMS`를 반환해야 합니다. 그 대신 `#handlePerspective`를 사용하여 시점 처리를 해야 합니다.

포지 로직
-------------

포지에선 시점 처리를 `#handlePerspective`에서 하는데, 이 메서드는 `BakedModel`에 새로 주입된 메서드이며, 기존의 `#getTransforms`를 대체합니다.

#### `BakedModel#handlePerspective`

`TransformType`과 `PoseStack`을 인자로 받고, 렌더링할 `BakedModel`을 반환하는 메서드 입니다. 이때 반환하는 `BakedModel`은 완전히 새로운 모델이어도 되는데, 이를 통해 기존 마인크래프트 로직보다 더 유연한 시점 처리를 수행할 수 있습니다(예: 손에 들면 평평하지만, 땅에 던지면 구겨지는 종이를 구현할 수 있음). 

### `PerspectiveMapWrapper`

`BakedModel`을 감싸는 wrapper입니다. `#handlePerspective`를 제외한 모든 메서드 호출을 `BakedModel`로 위임합니다, 그리고 `Map<TransformType, Transformation>`을 활용해 `#handlePerspective`를 처리합니다, 또한 아래와 같이 여러 유용한 정적 메서드들을 제공합니다.

#### `getTransforms`와 `getTransformsWithFallback`

`ItemTransforms`, 또는 `ModelState`로부터 `ImmutableMap<TransformType, Transformation>`을 구성하는 메서드 입니다. `ModelState`가 전달되었을 땐, 각 `TransformType`을 `#getPartTransformation`의 인자로 호출하여 알맞은 `Transformation`을 구성합니다.

일반적으로 모델들은 `ModelState`를 이 메서드들에 활용하여 시점 처리를 해야 합니다. `UnbakedModel#bake`에서 이 메서드를 호출하고, 그렇게 생성한 `Map<TransformType, Transformation>`을 `#bake`의 결과인 `BakedModel`과 합쳐 `PerspectiveMapWrapper`를 생성하여, 자동으로 `#handlePerspective`를 처리할 수 있습니다.
!!! note
    여기서 말하는 `getTransforms`는 `BakedModel#getTransforms`와 다른 것입니다, 여기에서는 정적 메서드 `PerspectiveMapWrapper#getTransforms`에 대해 다루고 있고, `BakedModel#getTransforms`는 이전에도 말씀드린 것처럼 포지 로직으로 대체된 마인크래프트 렌더링 로직입니다.

#### `handlePerspective`

`ModelState` 또는 `Map<TransformType, Transformation`, `BakedModel`, `TransformType`, 그리고 `PoseStack`을 인자로 전달받으면, 해당 시점에 알맞은 `Transformation`을 `BakedModel`, 또는 `ModelState`로 부터 찾은 이후, 해당 행렬을 전달한 `PoseStack`에 적용합니다. 이때 `ModelState`의 경우 위와 동일하게 `#getPartTransformation`을 사용하여 `Transformation`을 찾습니다. 이 메서드는 간단하게 `BakedModel#handlePerspective`를 구현하는 데 사용할 수 있습니다.

[bakedmodel]: ./bakedmodel.md
