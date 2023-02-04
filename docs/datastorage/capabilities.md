캐패빌리티
=====================

캐패빌리티는 여러 인터페이스 구현하느라 고생할 필요 없이 동적이고 유연하게 여러 기능들을 노출시킬 수 있도록 해줍니다. 이 시스템은 특정 객체에 캐패빌리티를 추가하거나 노출시켜, 동적으로 기능을 추가하거나 데이터를 추가할 수 있도록 합니다.

일반적으로, 캐패빌리티는 규격을 정의하는 *인터페이스*, 그 인터페이스의 *기본 구현*, 그리고 *데이터 핸들러* 이 3가지로 이루어져 있습니다. 캐패빌리티의 인터페이스를 구현하는 클래스는 여러개 있을 수 있으나, 무조건 하나 이상은 있어야 하기에 기본 구현이 요구됩니다. 그리고 데이터 핸들러는 기본 구현만 지원할 수도 있습니다.

포지는 BlockEntity, Entity, ItemStack, Level, 그리고 LevelChunk 가 기본적으로 캐패빌리티 시스템을 지원하도록 합니다, 그렇기에 이 객체들에는 캐패빌리티를 이벤트를 통해서 추가하거나, 객체들을 구현하며 노출시킬 수 있습니다. 이에 대해서는 아래 더 자세히 다루도록 하겠습니다.

포지에서 제공하는 캐패빌리티
---------------------------

포지에서는 `IItemHandler`, `IFluidHandler`, 그리고 `IEnergyStorage` 캐패빌리티를 제공합니다.
`IItemHandler` 는 인벤토리 슬롯을 관리하는 캐패빌리티의 규격입니다. 이 캐패빌리티는 BlockEntity, Entity, 또는 ItemStack 에 사용할 수 있습니다. `IInventory` 와 `ISidedInventory` 대신 사용하세요.

`IFluidHandler` 는 액체 저장 공간을 관리하는 캐패빌리티의 규격입니다. 이 캐패빌리티는 BlockEntity, Entity, 또는 ItemStack 에 적용할 수 있습니다. 

`IEnergyStorage` 는 에너지 저장 공간을 관리하는 캐패빌리티의 규격입니다. 이는 BlockEntity, Entity, 또는 ItemStack 에 적용할 수 있습니다. TeamCoFH 의 RedstoneFlux API 를 기반으로 하여 만들어 졌습니다.

여담으로, 모드 개발자들 사이에선 캐패빌리티 인터페이스를 캐패빌리티 그 자체인것 처럼 표현하기도 합니다, `IItemHandler` 캐패빌리티라고 하면, `IItemHandler` 인터페이스를 규격으로 사용하는 캐패빌리티를 말하는 것입니다.

이미 존재하는 캐패빌리티 사용하기
----------------------------

이미 이전에 말했다싶이, `BlockEntity`, `Entity`, 그리고 `ItemStack` 은 캐패빌리티 시스템을 지원합니다. `ICapabilityProvider` 인터페이스를 구현하는 것으로 캐패빌리티 시스템을 지원할 수 있습니다. 이 인터페이스를 구현하는 객체들은 캐패빌리티 제공자라고 합니다. 이 인터페이스는 `#getCapability` 메서드를 정의하는데, 이 메서드는 제공자가 가진 캐패빌리티 인터페이스 구현의 인스턴스를 감싸는 `LazyOptional` 이 반환합니다. 이에 대해서는 아래에서 더 다루도록 하겠습니다.

한 제공자가 여러 캐패빌리티를 가지고 있을 수 있다 보니, 그중 하나만 선택하여 참조하기 위해서는, 해당 캐패빌리티의 인스턴스가 필요합니다. `IItemHandler` 의 경우 `CapabilityItemHandler#ITEM_HANDLER_CAPABILITY` 에 인스턴스가 할당되어 있지만, 다른 캐패빌리티들은 `CapabilityManager#get` 를 사용하여 참조할 수 있습니다

```Java
static Capability<IItemHandler> ITEM_HANDLER_CAPABILITY = CapabilityManager.get(new CapabilityToken<>(){});

static Capability<IItemHandler> STATIC_REFERENCE = CapabilityItemHandler#ITEM_HANDLER_CAPABILITY;
// 위 두가지 다 동일한 캐패빌리티의 인스턴스를 반환합니다
```

`CapabilityManager#get` 은 호출되었을 경우 null 이 아닌 요청된 타입의 캐패빌리티 인스턴스를 반환합니다. `CapabilityToken` 익명 클래스는 포지가 강제적인 의존성을 부여하는 것을 피하면서, 올바른 캐패빌리티 인스턴스를 반환하기 위해 필요한 제너릭 타입 인자에 대한 정보를 제공받을 수 있도록 합니다.

!!! important
    null 이 아닌 캐패빌리티의 인스턴스의 참조를 언제나 얻을 수 있긴 하나, 그렇게 얻은 캐패빌리티를 언제나 사용할 수 있다는 것은 아닙니다. 캐패빌리티가 사용가능한지 확인하려면 `Capability#isRegistered` 를 사용하세요.

`#getCapability` 메서드는 `Direction` 를 두번째 인자로 받는데, 이는 방향(또는 면)에 따라 다른 캐패빌리티를 사용할 수 있도록 해줍니다. 만약 방향으로 `null` 이 전달되었다면, 블록 안쪽에서 캐패빌리티를 요청하였거나, 맥락상 방향이 필요가 없어 방향을 신경쓰지 않는 범용적인 캐패빌리티를 요청하였다고 볼 수 있습니다. `#getCapability` 가 반환하는 타입은 전달된 캐패빌리티의 인스턴스의 타입을 감싸는 `LazyOptional` 입니다. 예를 들어, `IItemHandler` 캐패빌리티가 전달되었을 경우, `LazyOptional<IItemHalder>` 가 반환됩니다. 만약 `#getCapability` 메서드가 요청받은 캐패빌리티를 반환할 수 없다면, 비어있는 `LazyOptional` 이 대신 반환됩니다.

캐패빌리티 노출하기
---------------------

캐패빌리티를 노출하기 위해서는 첫번쨰로 캐패빌리티 인터페이스의 구현의 인스턴스가 필요합니다. 하나 알아두셔야 할 점은, 각 구현의 인스턴스들은 각자 다른 데이터를 가질 수 있다는 것입니다, 이를 활용한 예로, 캐패빌리티 인터페이스의 구현의 각기 다른 인스턴스들을 여러 엔티티에  각각 할당하여 각각의 엔티티가 다른 데이터를 가지도록 할 수 있습니다. 캐패빌리티를 이런식으로 활용하는 것은 보편적이며 권장됩니다.

`IItemHandler` 의 경우, 기본 구현은 `ItemStackHandler` 클래스입니다, 이 클래스의 생성자는 인벤토리 슬롯 갯수를 인자로 받으며, 기본값은 1입니다. 그렇지만 이렇게 캐패빌리티가 기본적으로 제공하는 구현을 사용하는 것은 권장하지 않습니다, 캐패빌리티 시스템의 목적은 게임을 불러오는 도중, 몇몇 캐패빌리티가 존재하지 않을 때 발생할 수 있는 오류들을 최소화하는 것이기 때문입니다, 그렇기에 기본 구현의 인스턴스를 만드는 것은 해당 캐패빌리티가 이미 존재하는지 확인한 이후 진행하셔야만 합니다. (이전 섹션의 `CapabilityManager#get` 에 관한 내용을 참고하세요).

이제 캐패빌리티 인터페이스 구현의 인스턴스가 있으니, 캐패빌리티 시스템 사용자들에게 새로운 캐패빌리티를 노출한다고 알리고 `LazyOptional` 을 통한 참조를 제공해야 합니다. `#getCapability` 메서드를 재정의하고, 인자로 전달받은 캐패빌리티 인스턴스를 노출하고자 하는 캐패빌리티와 비교하세요. 만약 해당 제공자가 방향에 따라 다른 캐패빌리티를 반환해야 한다면 전달받은 `side` 인자를 사용하실 수 있습니다, 사용 예시로는 방향에 따라 다른 인벤토리 슬롯을 가지는 기계가 있습니다. 엔티티와 아이템 스택의 경우 `side` 를 무시할 수 있으나, 플레이어 갑옷 (`Direction#UP` 는 플레이어의 투구 슬롯의 캐패빌리티를 노출함) 등과 같은 용도에 사용하실 수도 있습니다. `super` 메서드 호출을 하는 것을 잊지 마세요, 그렇지 않으면 이미 부착된 캐패빌리티를 제공할 수 없게 됩니다.

캐패빌리티는 제공자의 생명주기가 끝나면 무조건 `LazyOptional#invalidate` 를 호출하여 무효화 하여야만 합니다, 엔티티와 블록 엔티티의 경우 월드에서 제거될 때 `#invalidateCaps` 를 자동으로 호출하기 때문에, 만약 직접 이들을 구현하고 계신다면 `#invalidateCaps` 를 재정의하여 캐패빌리티를 무효화 시킬 수 있습니다. 그외 다른 제공자들은 캐패빌리티를 무효화시키는 `Runnable` 을 `AttachCapabilitiesEvent#addListener` 에 전달해야 합니다.

```java
// 블록 엔티티 자식 클래스 어딘가에 아래 내용을 추가하세요
LazyOptional<IItemHandler> inventoryHandlerLazyOptional;

// inventoryHandlerSupplier 는 캐패빌리티 인터페이스 구현을 반환하는 Supplier 입니다 (예: () -> inventoryHandler).
// 개으른 초기화를 하도록 하여 꼭 필요할 때만 초기화 되도록 합니다.
inventoryHandlerLazyOptional = LazyOptional.of(inventoryHandlerSupplier);

@Override
public <T> LazyOptional<T> getCapability(Capability<T> cap, Direction side) {
  // cap 은 캐패빌리티 인스턴스
  if (cap == CapabilityItemHandler.ITEM_HANDLER_CAPABILITY) {
    return inventoryHandlerLazyOptional.cast();
  }
  // super 를 호출하지 않으면 부착된 다른 캐패빌리티를 사용할 수 없게 됩니다!!
  return super.getCapability(cap, side);
}

@Override
public void invalidateCaps() {
  super.invalidateCaps();
  inventoryHandlerLazyOptional.invalidate();
}
```

`Item` 들은 조금 특이하게 다루어야 하는데, 캐패빌리티 제공자들을 `Item` 이 아니라 `ItemStack` 에 부착하기 때문입니다. 그렇기 때문에 `Item#initCapabilities` 에서 새로운 제공자들을 부착하여야 합니다. 이때 부착된 캐패빌리티들은 그 아이템 스택의 생명주기가 끝나면 무효화 됩니다.

캐패빌리티 요청은 매 틱마다, 수십번씩 발생할 수 있으니, 캐패빌리티 제공자는 매우 빨라야만 합니다, 그렇지 않으면 게임의 전반적인 성능을 저해할 수 있습니다. 그렇기 때문에 외부 자료구조 등을 사용하는 것은 권장되지 않습니다.

캐패빌리티 부착하기
----------------------

이미 이전에 언급했듯이, 캐패빌리티를 이미 존재하는 `Level` 이나 `LevelChunk` 와 같은 제공자에 부착하는 것은 `AttachCapabilitiesEvent` 를 통해 이루어집니다. 이 이벤트는 모든 캐패빌리티 제공자에 사용됩니다. `AttachCapabilitiesEvent` 는 5개의 제너릭 타입이 있는데, 이는 다음과 같습니다:

* `AttachCapabilitiesEvent<Entity>`: 엔티티에 등록할 때만 방송됨.
* `AttachCapabilitiesEvent<BlockEntity>`: 블록 엔티티에 등록할 때만 방송됨.
* `AttachCapabilitiesEvent<ItemStack>`: 아이템 스택에 등록할 때만 방송됨.
* `AttachCapabilitiesEvent<Level>`: 월드에 등록할 때만 방송됨.
* `AttachCapabilitiesEvent<LevelChunk>`: 청크에 등록할 때만 방송됨.

이 제너릭 타입들은 위에 지정된 타입들보다 더 구체적일 수는 없습니다, 예를 들어, `Player` 에다가 캐패빌리티를 부착시키고 싶다면 `AttachCapabilitiesEvent<Event>` 이벤트를 구독한 이후, 전달된 제공자 객체가 `Player` 인지 확인하여야 합니다.

이 이벤트는 `#addCapability` 메서드가 있는데, 이 이벤트는 대상 제공자 객체에다가 새로운 캐패빌리티를 부착하는데 사용할 수 있습니다. 리스트에 캐패빌리티 그 자체를 추가하기 보다는 캐패빌리티 제공자를 추가합니다, 즉 이미 존재하는 제공자에다가 또 다른 제공자를 추가하는 것이지요. 이때, 추가되는 제공자들은 보편적으로 익명클래스입니다. 또한, 추가되는 제공자들은 방향에 따라 다른 캐패빌리티를 반환할 수도 있습니다. 캐패빌리티 제공자는 `ICapabilityProvider` 만 구현하면 되지만, 만약 캐패빌리티의 데이터가 유지되어야 할 경우, `ICapabilitySerializable<T extends Tag>` 를 대신 구현할 수도 있습니다, 이 인터페이스는 캐패빌리티를 제공하는 메서드 뿐만 아니라 NBT를 저장하고 불러오는 메서드 또한  가지고 있습니다.

`ICapabilityProvider` 구현에 관해서는 [캐패빌리티 노출시키기][expose] 섹션을 참고하세요.

캐패빌리티 직접 만들기
----------------------------

일반적으로, 캐패빌리티는 모드 버스에 방송되는 `RegisterCapabilitiesEvent` 이벤트에 `#register` 를 호출하여 등록됩니다.

```java
@SubscribeEvent
public void registerCaps(RegisterCapabilitiesEvent event) {
  event.register(IExampleCapability.class);
}
```

LevelChunk 와 BlockEntity 캐패빌리티 데이터 유지시키지
--------------------------------------------

레벨, 엔티티, 아이템 스택과 다르게, 레벨 청크와 블록 엔티티들은 그 데이터가 수정되었다고 표기되었을 경우에만 디스크에 써집니다. 그렇기 때문에 레벨 청크 또는 블록 엔티티에 사용할 캐패빌리티의 데이터를 올바르게 유지시키지 위해서는 캐패빌리티의 데이터가 변경되었을 때 해당 객체의 데이터가 수정되었다고 표기하여야만 합니다.

블록 엔티티에 인벤토리를 추가하는 `ItemStackHandler` 는 `void onContentsChanged(int slot)` 이라는 메서드를 가지고 있습니다. 이 메서드는 블록 엔티티의 데이터가 수정되었다고 표기하기 위해 사용됩니다.

```java
public class MyBlockEntity extends BlockEntity {

  private final IItemHandler inventory = new ItemStackHandler(...) {
    @Override
    protected void onContentsChanged(int slot) {
      super.onContentsChanged(slot);
      setChanged();
    }
  }

  // ...
}
```

클라이언트와 데이터 동기화 하기
-------------------------------

기본적으로, 캐패빌리티의 데이터는 클라이언트에게 전송되지 않습니다. 그렇기 때문에 모드들은 스스로 패킷을 사용해서 데이터를 직접 동기화 하여야 합니다.

데이터를 동기화해야 하는 상황은 크게 아래 3가지가 있습니다, 이러한 상황들을 처리하는 것은 선택사항입니다.

1. 엔티티가 레벨에 스폰될때나, 블록이 설치되는 경우. 이럴땐 클라이언트들에 초기 값을 보내볼 수 있습니다.
2. 저장된 데이터가 수정되는 경우, 이경우 데이터를 주시하고 있는 클라이언트들에 데이터를 보내볼 수 있습니다.
3. 클라이언트가 특정 엔티티나 블록을 처다보기 시작할 때, 이 경우 이미 존재하는 데이터를 보내볼 수 있습니다.

[네트워킹][network]를 참고하여 네트워크 패킷을 구현하는 방법에 대해 자세히 알아보세요.
Refer to the [Networking][network] page for more information on implementing network packets.

플레이어가 죽어도 데이터 유지시키기
-------------------------------

기본적으로, 캐패빌리티의 데이터는 사망하면 다 사라집니다. 그렇기 때문에 플레이어가 사망시 캐패빌리티의 데이터를 리스폰 과정에서 수동으로 복사하여야만 합니다.

`PlayerEvent$Clone` 의 이벤트를 구독하여 이를 구현할 수 있는데, 죽기 전 플레이어 엔티티로 부터 데이터를 읽어 와 새로운 플레이어 엔티티에 데이터를 작성하는 것입니다. 이 이벤트는 `#isWasDead` 메서드를 제공하여, 플레이어가 진짜 죽은 것인지, 아니면 엔드에서 돌아오는 것인지 구분할 수 있도록 해줍니다. 이 필드가 중요한 이유는, 엔드에서 돌아올 때는 데이터가 그대로 남아있기 때문에 중복을 방지해야 하기 때문입니다.

IExtendedEntityProperties 로 부터 이전하기
---------------------------

!!! warning
    이부분은 번역가가 익숙하지 않은 부분이라 번역이 완전하지 않을 수 있습니다, IEEP 를 캐패빌리티로 업그레이드해야 할 필요가 없다면 넘어가셔도 됩니다!

캐패빌리티 시스템은 이미 IEEP 보다 모든 방면에서 우월하지만, 이 두 시스템은 1대1로 대응되진 않습니다, 이 섹션에서는 IEEP 를 캐패빌리티로 바꾸는 방법에 대해 다루겠습니다.

아래는 IEEP와 캐패빌리티에서 동일한 개념들입니다:

* 속성 이름/아이디 (`String`): 캐패빌리티 키 (`ResourceLocation`)
* 등록 (`EntityConstructing`): 부착 (`AttachCapabilitiesEvent<Entity>`), 캐패빌리티는 `FMLCommonSetupEvent` 이벤트 도중 등록합니다.
* Tag 읽기/쓰기: 데이터 읽기/쓰기는 자동으로 일어나지 않습니다. `ICapabilitySerializable` 를 이벤트 핸들러에서 부착하고, `serializeNBT`/`deserializeNBT` 에서 읽기/쓰기 메서드들을 호출하세요.

IEEP 를 캐패빌리티로 변환하는법:

1. IEEP의 키/아이디 문자열을 `ResourceLocation` 으로 바꾸세요. (modid 를 네임 스페이스로 사용하세요).
2. 데이터 핸들러에 캐패빌리티의 인스턴스를 참조하는 필드를 만드세요.
3. `EntityConstructing` 이벤트를 `AttachCapabilityEvent` 로 바꾸세요, 그리고 IEEP 를 바로 등록하는 대신 `ICapabilityProvider` 를 부착하세요(데이터를 디스크에 저장할 필요가 있다면 `ICapabilitySerializable` 을 대신 부착하세요).
4. IEEP를 사용하셨다면, 아마 이벤트 핸들러를 등록하는 `register()` 라는 메서드를 IEEP 에 정의해 두셨을 텐데, 없다면 하나 만드세요. 이 메서드에서는 캐패빌리티를 등록하는 메서드를 호출하세요 (그리고 다시 한번 말씀들이지만, 이 메서드를 `FMLCommonSetupEvent` 도중 호출하는 것을 잊지 마세요).

[expose]: #캐패빌리티-노출하기
[network]: ../networking/index.md
