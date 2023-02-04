BlockEntityWithoutLevelRenderer
=======================

`BlockEntityWithoutLevelRenderer` 는 아이템을 동적으로 렌더링할 수 있도록 해주는 클래스입니다. 이 시스템은 이전에 `ItemStack` 을 기반으로 만든  구시스템보다 간단한데, 그때는 `BlockEntity` 를 사용했어야만 했었고, `ItemStack` 에 접근하는 것은 불가능했었습니다.

BlockEntityWithoutLevelRenderer 쓰기
--------------------------

BlockEntityWithoutLevelRenderer(또는 BWELR) 는 아이템을 `public void renderByItem(ItemStack itemStack, TransformType transformType, PoseStack poseStack, MultiBufferSource bufferSource, int combinedLight, int combinedOverlay)` 를 사용해 렌더링할 수 있도록 합니다.

BEWLR 을 사용하기 위해선, `Item` 의 모델이 `BakedModel#isCustomRenderer` 에서 true 를 반환하도록 해야 합니다, 이를 위한 방법은 여러가지가 있는데, 아이템의 모델이 JSON 으로 정의되어 있다면 다음 줄을 추가하시면 됩니다.

```json
"parent":"builtin/entity"
```

어떻게든 위 메서드에서 true 를 반환하도록 만드셨다면, 게임이 아이템을 렌더링할 때 BEWLR 를 사용하게 됩니다. 만약 아이템이 사용할 BEWLR 을 따로 지정하지 않으셨다면, `ItemRenderer#getBlockEntityRenderer` 를 대신 사용할 것입니다.  

아이템이 사용할 BEWLR 을 지정하기 위해선, `IItemRenderProperties` 의 익명 인스턴스를 `Item#initializeClient` 에서 사용하셔야 합니다. 이때 사용하는 익명 인스턴스에서 `IItemRenderProperties#getItemStackRenderer` 메서드를 재정의해서 커스텀 BEWLR 을 반환하도록 해야 합니다. 

```java
// 아이템 클래스 내부
@Override
public void initializeClient(Consumer<IItemRenderProperties> consumer) {
  consumer.accept(new IItemRenderProperties() {

    @Override
    public BlockEntityWithoutLevelRenderer getItemStackRenderer() {
      return myBEWLRInstance;
    }
  });
}
```

!!! important
    각 모드는 하나의 BEWLR 만 사용하실 수 있습니다.

BEWLR 을 사용하기 위한 이것 이외의 추가적인 작업은 필요하지 않습니다.
