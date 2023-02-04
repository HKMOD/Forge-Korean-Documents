이벤트
======

포지는 이벤트 버스를 이용하여 여러 모드들이 바닐라 마인크래프트의 여러 이벤트에 반응할 수 있도록 해줍니다.

예를 들어, 막대기를 우클릭 하였을때 이벤트가 방송되고 모드는 이에 반응할 수 있습니다.

대부분의 이벤트가 방송되는 주 이벤트 버스는 `MinecraftForge#EVENT_BUS` 에 위치합니다. 또한, 각 모드별 이벤트 버스또한 존재하는데, 이는 `FMLJavaModLoadingContext#getModEventBus` 에 위치합니다. 이 버스는 오직 특정한 상황에서만 사용됩니다. 이 버스를 어디에 사용하는지는 아래 자세하게 나와있습니다.

모든 이벤트는 이 2 버스중 하나에 방송됩니다: 대부분의 이벤트는 주 포지 버스에서 방송되지만, 그중 일부는 모드별 버스에서 방송됩니다.

이벤트 핸들러는 이벤트 버스에 등록되어, 특정 이벤트에 반응하는 메서드 입니다.

이벤트 핸들러 만들기
-------------------------

이벤트 핸들러 메서드들은 결과를 반환하지 않고 인자가 하나만 있습니다. 이 메서드들은 정적이어도 되고 아니어도 됩니다.

이벤트 핸들러들은 `IEventBus#addListener` 를 사용하여 바로 등록하실 수 있습니다. 만약 이벤트가 제너릭 클래스이고, `GenericEvent<T>` 의 자식 클래스일 경우 `IEventBus#addGenericListener` 를 대신 사용하실 수 있습니다, 둘 다 전달될 메서드를 표현하는 `Consumer` 를 인자로 받습니다. 제너릭 이벤트에 반응할 핸들러들은 타입 인자또한 전달하여야 합니다. 이벤트 핸들러들은 무조건 모드의 메인 클래스의 생성자에서 등록되어야 합니다.

```java
// In the main mod class ExampleMod

// This event is on the forge bus
private void forgeEventHandler(AddReloadListenerEvent event) {
    // Do things here
}

// 이 이벤트는 모드 버스에서 방송됩니다
private static void modEventHandler(RegistryEvent.Register<RecipeSerializer<?>> event) {
    // ...
}

// 모드의 생성자
forgeEventBus.addListener(this::forgeEventHandler);
modEventBus.addGenericListener(RecipeSerializer.class, ExampleMod::modEventHandler);
```

### 어노테이션을 활용한 이벤트 핸들러

이 이벤트 핸들러는 `EntityItemPickupEvent` 에 반응합니다, 이름에서 알 수 있다싶이, `Entity` 가 아이템을 주울 때 모드 버스에 방송됩니다.

```java
public class MyForgeEventHandler {
    @SubscribeEvent
    public void pickupItem(EntityItemPickupEvent event) {
        System.out.println("아이템을 주웠습니다!!");
    }
}
```

이 이벤트 핸들러를 등록하기 위해서는 `MinecraftForge.EVENT_BUS.register(...)` 를 사용하세요. 그리고 이 메서드에 이벤트 핸들러 메서드가 있는 클래스의 인스턴스를 매개변수로 전달하세요. 만약 핸들러를 모드별 버스에 등록하고 싶다면 `FMLJavaModLoadingContext.get().getModEventBus().register(...)` 를 대신 사용하세요.

### 어노테이션을 활용한 정적 이벤트 핸들러

이벤트 핸들러를 정적으로 만들 수도 있습니다. 이 메서드에도 `@SubscribeEvent` 어노테이션이 있습니다. 위에서 사용한 인스턴스를 통한 이벤트 핸들러와의 차이점은 메서드가 정적이라는 것입니다. 정적 이벤트 핸들러를 등록하기 위해서는 클래스의 인스턴스가 아니고, 클래스 그 자체가 전달되어야 합니다. 그 예로:

```java
public class MyStaticForgeEventHandler {
    @SubscribeEvent
    public static void arrowNocked(ArrowNockEvent event) {
        System.out.println("화살 당겨짐!");
    }
}
```

이는 `MinecraftForge.EVENT_BUS.register(MyStaticForgeEventHandler.class)` 를 통해 등록합니다.

### 자동으로 정적 이벤트 핸들러 등록하기

`@Mod$EventBusSubscriber` 어노테이션은 클래스에 사용할 수 있습니다. 만약 이를 사용할 시, 그 클래스는 자동으로 `MinecraftForge#EVENT_BUS` 에 `@Mod` 클래스가 초기화될 때 등록됩니다. 이는 `MinecraftForge.EVENT_BUS.register(AnnotatedClass.class)` 구문을 `@Mod` 클래스의 생성자에서 사용하는 것과 동일합니다.

`@Mod$EventBusSubscriber` 는 아무 버스나 사용할 수 있습니다. 이를 사용할 때 모드의 아이디를 전달하는 것이 권장되는데, 이는 어노테이션만으로는 무슨 모드의 이벤트 핸들러인지 구별할 수 없기 때문입니다. 또, 이벤트를 들을 버스를 전달하는 것 또한 권장되는데, 무슨 버스의 이벤트를 듣는지 표시하기 때문입니다. 또, `Dist` 값을 지정하여 어떤 물리 사이드에서 이벤트 핸들러가 동작할 것인지를 설정하실 수 있습니다. 이를 통해 특정 물리 사이드에서는 아예 이벤트 핸들러가 등록되지 않도록 할 수 있습니다.

이를 이용한, `RenderLevelLastEvent` 이벤트에 반응하는, 클라이언트에만 존재하는 정적 이벤트 핸들러 입니다.

```java
@Mod.EventBusSubscriber(modid = "mymod", bus = Bus.FORGE, value = Dist.CLIENT)
public class MyStaticClientOnlyEventHandler {
    @SubscribeEvent
    public static void drawLast(RenderLevelLastEvent event) {
        System.out.println("월드 그리는중!");
    }
}
```

!!! note
    이를 이용하면 클래스의 인스턴스가 아닌 클래스 그 자체가 등록됩니다. 그렇기에 등록되는 모든 이벤트 핸들러는 정적이어야 제대로 동작합니다!

이벤트 취소하기
---------

만약 이벤트가 취소될 수 있다면, `@Cancelable` 어노테이션이 있을 것입니다, 그리고 메서드 `Event#isCancelable()` 는 `true` 를 반환합니다. 이벤트의 취소 여부는 `Event#setCanceled(boolean canceled` 메서드를 통해 변경될 수 있습니다, `true` 를 전달할 시 이벤트가 취소되고 `false` 를 전달할 시 이벤트의 "취소를 취소합니다". 그러나 만약에 이벤트가 취소될 수 없다면, 이 메서드를 사용하는 것 만으로 `UnsupportedOperationException` 예외가 발생합니다. 최소될 수 없는 이벤트의 취소 여부는 불변값으로 취급되기 때문입니다.

!!! important
    모든 이벤트가 취소될 수 있는 것은 아닙니다! 취소할 수 없는 이벤트를 취소하려고 할 시 `UnsuppoortedOperationException` 예외가 발생하고 게임이 튕기게 됩니다! 그렇기에 이벤트를 취소하기 전에 `Event#isCancelable()` 을 사용하는 등 이벤트가 취소될 수 있는지를 먼저 확실하게 확인하세요!

결과
-------

일부 이벤트들은 `Event$Result` 클래스를 사용합니다 결과는 이벤트를 중단하는 `DENY`, 기본 바닐라 코드를 실행하는 `DEFAULT`, 강제적으로 특정 동작을 수행하도록 하는 `ALLOW`, 이렇게 3가지가 있습니다. 이벤트의 결과는 이벤트 도중 `#setResult` 를 사용해 지정할 수 있습니다. 모든 이벤트가 결과가 있는 것은 아닙니다; 결과가 있는 이벤트는 `@HasResult` 어노테이션이 있습니다.

!!! important
    여러 이벤트들은 각자 다른 방식으로 결과를 처리할 수 있습니다, 그렇기에 이벤트의 JavaDoc 에서 이벤트가 결과를 어떻게 처리할지를 미리 숙지하도록 하세요!

우선순위
--------

(`@SubscribeEvent` 어노테이션이 있는)이벤트 헨들러 메서드에는 우선순위가 있습니다. 이벤트 핸들러의 우선순의는 `@SubscribeEvent` 의 `priority` 값을 변경하여 지정합니다. 이 우선순위는 `EventPriority` 의 열거형(`HIGHEST`, `HIGH`, `NORMAL`, `LOW`, `LOWEST`)입니다. `HIGHEST` 우선순위를 가진 이벤트 핸들러들은 가장 먼저 실행되고, `LOWEST` 우선순위를 가진 이벤트 핸들러는 가장 나중에 실행됩니다.

하위 이벤트
----------

많은 이벤트들은 조금씩 다른 번형이 존재합니다. 이 변형된 이벤트들은 역할이 조금씩 다 다르지만, 전부 하나의 기반 이벤트를 상속하여 만들어져 있거나, (예: `PlayerEvent`) 또는 하나의 이벤트를 여러 단계로 나누어 둔 것일 수 있습니다. (예: `PotionBrewEvent`, `PotionBrewEvent$Pre`와 `PotionBrewEvent$Post` 가 있음). 만약 기반이 되는 부모 이벤트에 반응하는 이벤트 핸들러가 있다면, 그 이벤트 핸들러는 그 이벤트의 모든 자식 클래스의 이벤트에 반응합니다.

모드 이벤트 버스
-------------

모드 이벤트 버스는 주로 모드의 생명주기 이벤트를 방송하는데에 사용되어 모드의 초기화를 돕는데 사용됩니다. 모드 버스에 방송되는 이벤트들은 `IModBusEvent` 인터페이스를 무조건 구현하여야 합니다. 또한 이 이벤트들은 병렬적으로 방송되어 여러 모드가 동시에 초기화 될 수 있도록 합니다. 하지만 이로 인해 모드에선 다른 모드의 코드를 직접적으로 실행할 수 없습니다. 무조건 `InterModComms` 시스템을 거쳐야만 합니다.

이 4가지는 모드 초기화가 이루어 질 때 모드 버스에서 방송되는 가장 유용한 생명주기 이벤트들 입니다.

* `FMLCommonSetupEvent`
* `FMLClientSetupEvent` & `FMLDedicatedServerSetupEvent`
* `InterModEnqueueEvent`
* `InterModProcessEvent`

!!! note
    `FMLClientSetupEvent` 와 `FMLDedicatedServerSetupEvent` 는 올바른 배포판에서만 방송됩니다!

이 4개의 생명주기 이벤트들은 모두 `ParallelDispatchEvent` 의 자식 클래스이기 때문에 병렬적으로 방송됩니다. 만약 `ParallelDispatchEvent` 또는 그 자식 클래스 이벤트가 방송되는 도중 메인 스레드에서 코드를 실행하고 싶다면 `#enqueueWork` 를 사용하세요.

생명주기 이벤트 말고, 모드 버스에서 방송되는 다른 이벤트들도 있습니다. 이 이벤트들 또한 무언가를 등록하거나, 초기화 하는 등의 용도로 쓰입니다. 이 이벤트들은 대부분 생명주기 이벤트들과 다르게 비동기적이지 않습니다. 이러한 이벤트들의 예로:

* `ColorHandlerEvent`
* `ModelBakeEvent`
* `TextureStitchEvent`
* `RegistryEvent`

등이 있습니다. 만약 이벤트가 모드 초기화 도중 방송된다면, 모드 버스에서 방송된다고 볼 수 있습니다.
