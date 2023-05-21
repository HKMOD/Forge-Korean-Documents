`BakedModel`
=============

`BakedModel`은 바닐라 모델의 경우 `UnbakedModel#bake`를, 커스텀 모델 로더의 경우 `IModelGeometry#bake`를 호출하면 반환되는 결괏값 입니다. 블록이나 아이템에 대한 아무런 정보가 없고 단순히 모양만 정의하는 `UnbakedModel`, 또는 `IModelGeometry`와 다르게, `BakedModel`은 추상적이지 않으며, 최적화되어 GPU에 보낼 준비가 (거의) 완료된 형태입니다. 또한 블록이나 아이템의 데이터에 따라 변형될 수도 있습니다.

일반적으로, 이 인터페이스를 직접 구현하실 필요는 없습니다, 이미 존재하는 다른 클래스들로도 충분히 여러 모델들을 만들 수 있습니다.

### `getOverrides`

해당 모델이 사용하는 [`ItemOverrides`][overrides]를 반환합니다. 이 메서드는 해당 모델을 아이템처럼 사용할 때만 사용됩니다.

### `useAmbientOcclusion`

만약 모델이 블록의 모델로 사용되어 레벨에 렌더링 되었고, 블록이 발광하지도 않으며, 이 메서드에서 `true`를 반환한다면, 모델을 [부드러운 조명 효과](ambocc)를 활용해 렌더링합니다.

### `isGui3d`

만약 모델이 인벤토리 아이템, 땅에 떨어진 아이템, 아니면 아이템 액자에 걸린 아이템 등등일 경우, 모델을 "평평하게" 만듭니다. GUI에선 조명 효과 또한 건너뜁니다.

### `isCustomRenderer`

!!! important
    정확히 무엇을 하시는지 모르신다면, `false`를 반환하도록 하세요.

해당 모델을 아이템으로 렌더링할 때, 여기서 `true`를 반환한다면, 마인크래프트는 스스로 해당 모델을 렌더링하지 않습니다, 그 대신, `BlockEntityWithoutLevelRenderer#renderByItem`를 호출합니다. 게임은 상자나 현수막의 경우 해당 메서드는 아이템의 데이터를 `BlockEntity`에 복사한 이후, `BlockEntityRenderer`를 사용해 아이템 대신 `BlockEntity`를 아이템의 자리에 렌더링 하도록 하드코딩 되어있습니다. 다른 아이템의 경우, `IItemRenderProperties#getItemStackRenderer`가 반환하는 `BlockEntityWithoutLevelRenderer`를 사용합니다. 자세히 알고 싶으시다면 [BlockEntityWithoutLevelRenderer][bewlr] 문서를 참고하세요.

### `getParticleIcon`

파티클로 어떤 텍스쳐를 사용해야 할지를 지정합니다. 블록의 경우, 엔티티가 블록 위에 낙하했을 때, 블록을 캘 때 등의 상황에 사용됩니다. 아이템의 경우, 아이템을 깨뜨리거나 먹을 때 사용합니다.

!!! important
    아무런 인자도 받지 않는 바닐라의 `getParticleIcon` 대신 `#getParticleIcon(IModelData)`를 사용하세요, 모델의 데이터에 따라서 다른 파티클 텍스쳐를 사용할 수 있기 때문입니다.

### <s>`getTransforms`</s>

이것 대신 `#handlePerspective`를 사용하세요, `#handlePerspective`를 구현하셨다면 `getTransforms`는 오버라이드 안 하셔도 됩니다. 자세한 정보는 [시점][perspective]을 참고하세요.

### `handlePerspective`

[시점][perspective]을 참고하세요.

### `getQuads`

`BakedModel`에서 가장 중요한 메서드입니다. 이 메서드는 `BakedQuad`가 들어있는 리스트를 반환합니다. `BakedQuad`는 렌더링할 때 필요한 가장 저수준의 Vertex 정보를 담고 있습니다. 만약 해당 모델을 블록 모델로 사용하신다면, `null`이 아닌 `BlockState`가 인자로 전달됩니다. 만약 해당 모델을 아이템 모델로 사용하신다면, `#getOverrides`의 결괏값인 `ItemOverrides`가 아이템의 상태를 관리할 것이고, 전달되는 `BlockState`는 `null`이 될 것입니다.

`Direction`은 컬링을 수행할 때 사용됩니다. 컬링은 특정 면을 렌더링할지, 안할지를 면의 방향에 따라 결정하는 과정인데, 만약 어떤 면의 반대쪽에 있는 또 다른 면이 불투명하다면, 해당 면은 어차피 가려질 것이니 렌더링되지 않습니다. 만약 `Direction`으로 `null`이 전달되었다면, 방향과 관계없는(그리고 컬링을 수행하지 않을) 모든 면을 반환해야 합니다.

`rand`는 난수를 생성하기 위한 `Random`의 인스턴스입니다.

해당 메서드는 `null`이 아닌 `IModelData`도 인자로 받는데, 모델을 렌더링할 때 필요할 만한 추가적인 데이터를 `ModelProperty`들을 사용해 전달할 때 사용됩니다. 그 예로, 포지의 `forge:composite` 모델 로더는 `ModelProperty`중 하나로 `CompositeModelData`를 정의하여, 하위 모델들을 전달합니다.

해당 메서드는 엄청나게 자주 호출됩니다: 지원되는 모든 블록의 렌더 레이어와 컬링으로 가려지지 않은 모든 면 사이의 모든 조합의 수만큼(0~28번), *레벨에 있는 모든 블록마다* 호출됩니다. 다시 말해서, 이 메서드는 가능한 한 가장 빠르게 만들고 최대한 많은 것을 캐싱해야 합니다.

[overrides]: ./itemoverrides.md
[ambocc]: https://en.wikipedia.org/wiki/Ambient_occlusion
[bewlr]: ../../items/bewlr.md
[perspective]: ./perspective.md
