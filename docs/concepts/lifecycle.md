모드 생명주기
==============

모드를 불러오는 과정에서 여러 생명주기 이벤트들이 모드별 버스에 방송됩니다. 여러 초기화 작업들이 이 이벤트들 도중에 이루어 집니다. 이때 진행되는 초기화로는, [객체 등록하기][등록], [데이터 생성 준비][데이터생성], 또는 [다른 모드와의 통신][모드통신] 등이 있습니다.

이벤트 리스너들은 `@EventBusSubscriber(bus = Bus.MOD)` 어노테이션을 사용하거나 모드의 생성자에서 등록되어야 합니다:

```Java
@Mod.EventBusSubscriber(modid = "mymod", bus = Mod.EventBusSubscriber.Bus.MOD)
public class MyModEventSubscriber {
    @SubscribeEvent
    static void onCommonSetup(FMLCommonSetupEvent event) { ... }
}

@Mod("mymod")
public class MyMod {
    public MyMod() {
        FMLModLoadingContext.get().getModEventBus().addListener(this::onCommonSetup);
    } 
  
    private void onCommonSetup(FMLCommonSetupEvent event) { ... }
}
```

!!! warning
    대부분의 생명주기 이벤트들은 병렬적으로 방송됩니다: 모든 모드들은 동시에 같은 이벤트에 반응합니다.
    
    모드들은 다른 모드의 API 또는 마인크래프트 시스템에 접근할 때, 스레드 안전성을 준수해야만 합니다. `ParallelDispatchEvent#enqueueWork` 를 이용해 코드 실행을 예약하여 나중에 실행되도록 하세요.

레지스트리 이벤트
---------------

`RegistryEvent` 들은 모드 인스턴스 초기화 이후에 방송됩니다. 이 이벤트들의 종류는 `NewRegistry` 그리고 `Register` 가 있습니다. 이 이벤트들은 모드들을 불러올때 동기적으로 방송됩니다.

`RegistryEvent$NewRegistry` 이벤트는 모드 개발자들이 직접 만든 레지스트리를 `RegistryBuilder` 를 사용해 등록할 수 있도록 해줍니다.

`RegistryEvent$Register<?>` 이벤트는 [객체들을 레지스트리에 등록할 때][등록] 사용합니다. 이 이벤트는 각 레지스트리당 한번씩 방송됩니다, 이는 개발자가 직접 등록한 레지스트리도 포함됩니다.

데이터 생성
---------------

만약 게임이 [데이터 생성기][데이터생성]를 사용하도록 설정되어 실행되었다면, `GatherDataEvent` 가 가장 마지막에 방송됩니다. 이 이벤트는 모드들의 데이터 제공자를 데이터 생성기에 등록할때 사용합니다. 또한 이 이벤트는 동기적으로 방송됩니다.

일반 초기화
------------

`FMLCommonSetupEvent` 이벤트를 사용하여 클라이언트, 그리고 서버 사이드에 공통적으로 수행될 코드를 작성할 수 있습니다. 그 예로 [캐패빌리티][캐패빌리티]등록이 있습니다.

사이드 초기화
-----------

사이드 초기화 이벤트는 각각 알맞는 [물리 사이드][사이드]에 방송됩니다: `FMLClientSetupEvent` 는 물리 클라이언트에, `FMLDedicatedServerSetupEvent`는 전용 서버에 사용됩니다. 이 이벤트를 사용하여 키바인드 등록과 같은 사이드 전용 초기화를 진행할 수 있습니다.

InterModComms
-------------

이 생명주기 단계에서는 모드끼리 메세지를 보내 모드간 호환성을 유지할 수 있도록 합니다. 이때 방송되는 이벤트는 `InterModEnqueueEvent` 와 `InterModProcessEvent` 가 있습니다.

`InterModComms` 의 역할은 모드 통신에 사용되는 메세지를 보유하는 것입니다. 그리고 이 클래스의 메서드들은 `ConcurrentMap` 을 사용하기 때문에 병렬적으로 방송되는 생명주기 이벤트에서 호출하여도 안전합니다.

`InterModEnqueueEvent` 도중 `InterModComms#sendTo` 를 사용해 다른 모드에 메세지를 전송할 수 있습니다, 또한 `InterModProcessEvent` 도중 `InterModComms#getMessages` 를 사용해 받은 모든 메세지들을의 흐름(Stream)을 받을 수 있습니다.

!!! note
    그외에도 모드 인스턴스 생성 직후, 레지스트리 이벤트가 방송되기 이전에 방송되는 `FMLConstructModEvent` 생명주기 이벤트와 `InterModComms` 이후에 방송되어 모드를 완전히 불러왔음을 알리는 `FMLLoadCompleteEvent` 생명주기 이벤트가 있습니다.

[등록]: registries.md#객체-등록하기
[캐패빌리티]: ../datastorage/capabilities.md
[데이터생성]: ../datagen/intro.md
[모드통신]: lifecycle.md#intermodcomms
[사이드]: sides.md
