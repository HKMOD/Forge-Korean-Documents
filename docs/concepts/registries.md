레지스트리
==========

레지스트리 등록은 모드에서 추가하는 객체를 게임이 사용할 수 있도록 하는 과정입니다. 객체를 등록하는 것은 매우 중요한데, 만약 객체를 등록하지 않는다면 게임은 이 객체들의 존재를 알지 못하며 예기치 못한 동작을 하거나 튕길 수 있습니다.

대부분 등록되어야 하는 객체들은 포지 레지스트리에서 관리합니다. 레지스트리는 Map<K, V>와 비슷한 객체입니다. 포지는 [리소스 위치][ResourceLocation]를 Key로 사용하는 레지스트리를 사용하여 객체들을 등록합니다. 이를 통해 `ResourceLocation`을 객체의 레지스트리 이름처럼 사용할 수 있습니다. 객체의 레지스트리 이름은 `#getRegistryName`/`#setRegistryName`를 통해 접근할 수 있습니다. 이때 `#setRegistryName`은 오직 한번만 호출될 수 있습니다; 두번 호출할 시 예외가 발생하니 하지 마세요.

모든 등록될 수 있는 객체들은 알맞는 레지스트리가 존재합니다. 포지에서 지원하는 모든 레지스트리를 보려면 `ForgeRegistries` 클래스를 참조하세요. 레지스트리에 등록된 모든 객체들의 레지스트리 이름은 무조건 고유해야 합니다. 그러나 서로 다른 레지스트리에 있는 객체들은 같은 이름을 가지고 있어도 됩니다. 예를 들어, `Block` 레지스트리와 `Item` 레지스트리는 각각 `example:thing` 이라는 레지스트리 이름을 가진 객체를 가지고 있어도 이름이 겹치는 문제가 발생하지 않습니다. 그러나 `Block` 2개, 또는 `Item` 2개가 같은 레지스트리 이름으로 등록될 경우, 두번째로 등록된 것이 첫번째로 등록된 것을 덮어씌울 것입니다.

객체 등록하기
------------------

객체를 올바르게 등록하는 방법에는 2가지가 있는데, `DeferredRegister` 클래스와 `RegistryEvent$Register` 생명주기 이벤트 입니다.

### DeferredRegister

`DeferredRegister` 는 객체를 등록하는 새로운, 그리고 문서화된 방식입니다. 이는 간단하게 정적 초기화를 사용할 수 있게 해줍니다.

`DeferredRegister` 는 레지스트리에 등록시킬 객체들을 공급하는 공급자들을 리스트에 저장하고, 알맞은 `RegistryEvent$Regiser` 이벤트 도중에 등록합니다.

모드에서 블록을 등록하는 예시입니다:

```java
//블록들을 등록하는 DeferredRegister 인스턴스 만들기
private static final DeferredRegister<Block> BLOCKS = DeferredRegister.create(ForgeRegistries.BLOCKS, MODID);

//DeferredRegister 인스턴스에 블록 객체를 공급하는 공급자 등록하기
public static final RegistryObject<Block> ROCK_BLOCK = BLOCKS.register("rock", () -> new Block(BlockBehaviour.Properties.of(Material.STONE)));

public ExampleMod() {
    //DeferredRegister 가 RegistryEvent$Register 에 반응할 수 있도록 알맞은 이벤트 버스 설정하기
    BLOCKS.register(FMLJavaModLoadingContext.get().getModEventBus());
}
```

### `Register` 이벤트

`RegistryEvent` 들은 객체를 등록하는 또 다른 방법으로, 더욱 유연하게 객체들을 등록할 수 있습니다. 이 [이벤트][이벤트] 들은 모드들의 메인 클래스 인스턴스 생성할때와 설정 파일을 불러올때 사이에 방송됩니다.

객체를 등록할때 사용하는 이벤트는 `RegistryEvent$Register<T>` 입니다. 타입 매개변수 `T`는 등록되는 객체들의 타입이어야 합니다. 이벤트에서 `#getRegistry`를 호출하시면, `#register`나 `#registerAll`로 객체를 등록할 수 있는 레지스트리가 반환됩니다.

등록 예시(이 이벤트 핸들러는 무조건 *모드 버스*에 등록되어야 합니다):

```java
@SubscribeEvent
public void registerBlocks(RegistryEvent.Register<Block> event) {
    event.getRegistry().registerAll(new Block(...), new Block(...), ...);
}
```

### 포지 레지스트리가 아닌 레지스트리들

바닐라코드의 몇몇 특성 때믄에, 포지가 모든 레지스트리를 감쌀 순 없습니다. `RecipeType` 과 같은 정적이라서 사용해도 안전한 레지스트리나, `ConfiguredFeature` 와 같은 동적 레지스트리, 또는 월드젠 관련된 레지스트리들이 있습니다, 이 레지스트리들은 보통 JSON 을 통해 관리 및 등록을 합니다. 이러한 레지스트리에 등록하는 것은 다른 레지스트리 객체가 필요로 할때만 하여야 합니다. `DeferredRegister#create` 는 동명의 메서드, 또는 오버로드가 여러개 있는데, 그중 `RegistryObject` 들이 등록될 바닐라 레지스트리의 레지스트리 키를 받는 것도 있습니다. 이를 통해 만든 `DeferredRegister` 의 인스턴스는 다른 인스턴스들과 동일한 방식으로 객체를 등록하거나 모드 버스에 등록할 수 있습니다. 

```java
private static final DeferredRegister<RecipeType<?>> REGISTER = DeferredRegister.create(Registry.RECIPE_TYPE_REGISTRY, "examplemod");

// RecipeType 은 인터페이스이기 때문에, 익명 클래스를 생성하여 등록합니다
// 바닐라에서는 RecipeType 의 #toString 메서드를 재정의하는데, 이는 디버그 용도이기 때문에 이 예제에서는 건너뛰도록 하겠습니다
// 예제 ExampleRecipe
public static final RegistryObject<RecipeType<ExampleRecipe>> EXAMPLE_RECIPE_TYPE = REGISTER.register("example_recipe_type", () -> new RecipeType<>() {});
```

!!! note
    몇몇 클래스들은 레지스트리에 등록될 수 없습니다. 그 대신, 그 클래스들의 종류를 상징하는 `*Type` 클래스가 대신 등록되어야 하며, 이 `*Type` 클래스는 전자의 클래스의 생성자에서 사용되어야 합니다. 그 예시로, [`BlockEntity`][블록엔티티]는 `BlockEntityType`를, 그리고 `Entity`는 `EntityType`을 대신 등록하여야 합니다. 이 `*Type` 클래스들은 알맞는 클래스의 인스턴스를 생성하는 팩토리입니다.

    이 팩토리들은 `*Type$Builder` 클래스를 이용해 생성됩니다, 아래 `REGISTER` 는  `DeferredRegister<BlockEntityType>` 입니다.
    ```java
    public static final RegistryObject<BlockEntityType<ExampleBlockEntity>> EXAMPLE_BLOCK_ENTITY = REGISTER.register(
        "example_block_entity", () -> BlockEntityType.Builder.of(ExampleBlockEntity::new, EXAMPLE_BLOCK.get()).build(null)
    );
    ```

등록된 객체 참조하기
------------------------------

객체를 등록하실때 이를 특정 필드에 저장하면 안됩니다, 알맞는 `RegistryEvent$Register<T>` 이벤트가 방송될때 마다 새로 만들어 지고 등록되어야 하기 때문입니다. 이는 추후에 포지에서 동적으로 모드를 활성화/비활성화하기 위함입니다.

등록되는 객체들은 무조건 `RegistryObject` 또는 `@ObjectHolder` 어노테이션이 있는 필드를 통하여 참조되어야만 합니다.

### RegistryObject 로 참조하기

`RegistryObject`들은 등록된 객체들의 참조를 반환합니다. 등록되지 않은 객체에는 사용하실 수 없습니다. `DeferredRegister` 를 통해 등록하시면 `RegistryObject` 의 인스턴스의 참조가 반환됩니다. `@ObjectHolder` 어노테이션과 `RegistryObject` 들은 알맞는 `RegistryEvent$Register<T>` 가 방송될때 갱신됩니다.

`RegistryObject` 인스턴스의 참조를 다른 방법으로 얻기 위해서는 `RegistryObject#create` 를 등록된 객체의 `ResourceLocation`과 레지스트리의 `IForgeRegistry` 를 사용하여 호출하세요. 커스텀 레지스트리는 등록될 객체들의 클래스를 공급하는 공급자를 이용하여 `RegistryObject`에 접근할 수 있습니다. `RegistryObject`를 `public static final` 필드를 이용해 저장하고 `#get` 함수를 이용해 등록된 객체에 접근하세요.

`RegistryObject` 사용 예시:

```java
public static final RegistryObject<Item> BOW = RegistryObject.create(new ResourceLocation("minecraft:bow"), ForgeRegistries.ITEMS);

// 아래 예시에서는 'neomagicae:mana_type' 은 레지스트리이며, 'neomagicae:coffeinum' 이 해당 레지스트리에 등록된 객체라 가정합니다
public static final RegistryObject<ManaType> COFFEINUM = RegistryObject.create(new ResourceLocation("neomagicae", "coffeinum"), new ResourceLocation("neomagicae", "mana_type"), "neomagicae");
```

### @ObjectHolder 사용하기

`@ObjectHolder` 어노테이션을 클래스 또는 필드에 달고 `ResourceLocation`을 추론해 내기 충분한 정보를 제공하여,  `public static` 필드에 레지스트리의 등록된 객체들을 주입할 수 있습니다.

`@ObjectHolder` 를 사용하는 방법은 다음과 같습니다:

* 만약 클래스에 `@ObjectHolder` 어노테이션이 있다면, 어노테이션의 value 를 그 클래스의 필드들의 네임스페이스 기본값으로 사용합니다.
* 만약 클래스에 `@Mod` 어노테이션이 있다면, `modid` 를 그 클래스의 네임스페이스 기본값으로 사용합니다.
* 필드가 다음 조건들을 충족시킨다면 주입이 이루어 집니다:
  * 최소한 `public static` 키워드가 있을때;
  * 다음중 하나라도 참일때:
    * 필드를 **감싸는 클래스**에 `@ObjectHolder` 어노테이션이 있으며 필드가 `final` 이라면:
      * 필드의 이름을 리소스 위치의 경로로 사용합니다.
      * 리소스 위치의 네임 스페이스는 클래스의 `ObjectHolder`의 value 를 상속받아 사용합니다.
      * _만약 네임 스페이스를 찾을 수도, 상속 받을 수도 없다면 예외가 발생합니다._
    * **필드**에 `@ObjectHolder`가 있을때:
      * 리소스 위치의 경로는 필드의 어노테이션의 value 값을 사용합니다.
      * 리소스 위치의 네임 스페이스는 어노테이션에서 지정하지 않았다면 클래스의 어노테이션에서 상속 받습니다.
  * 필드의 타입이나, 필드의 타입의 조상중 일치하는 레지스트리가 있다면(예: `Item` 또는 `ArrowItem` 은 `Item` 레지스트리에 일치한다);
  * _만약 필드에 일치하는 레지스트리가 없다면 예외가 발생합니다._
* _만약 추론된 `ResourceLocation` 이 미완성이거나, 문제가 있다면 (허용되지 않은 문자가 있다면) 예외가 발생합니다._
* 만약 문제가 없거나, 예외가 발생하지 않았다면 필드에 등록된 객체가 주입될 것입니다.
* 만약 위 조건중 하나라도 만족하지 않는다면 아무 일도 일어나지 않습니다. (그리고 이에 관한 메세지가 출력될 수 있습니다.)

위 조건을 충족하는 `@ObjectHolder` 주입 대상 필드는 그에 알맞는 레지스트리의 `RegistryEvent$Register` 이벤트가 방송될때 객체가 주입됩니다. 또한 `RegistryObject` 들도 이때 갱신 됩니다.

!!! note
    만약 추론된 리소스 위치를 통해 객체를 참조하여 이를 필드에 주입하려고 할 때, 그 객체가 존재하지 않는다면 이에 관한 메세지가 출력될 것이며 필드에 값이 주입되지 않습니다.

위에 기술된 규칙들이 복잡해 보이실 수 있으니, 몇가지 예제를 준비해 보았습니다:

```java
@ObjectHolder("minecraft") // 상속 가능한 네임 스페이스: "minecraft"
class AnnotatedHolder {
  public static final Block diamond_block = null; // 어노테이션 없음, [public static final] 키워드가 있어야함.
                                                    // Block 은 이에 맞는 레지스트리가 있음: [Block]
                                                    // 리소스의 경로는: "diamond_block"
                                                    // 네임 스페이스는 명시적으로 정의되어 있지 않음.
                                                    // 그렇기에, 네임 스페이스는 클래스 어노테이션으로 부터 상속받음: "minecraft"
                                                    // 주입 대상: [Block] 레지스트리의 "minecraft:diamond_block" 

    @ObjectHolder("ambient.cave")
  public static SoundEvent ambient_sound = null;  // 어노테이션 있음, [public static] 키워드가 있어야함.
                                                    // SoundEvent 는 이에 맞는 레지스트리가 있음: [SoundEvent]
                                                    // 리소스의 경로는 어노테이션의 value 임: "ambient.cave"
                                                    // 네임 스페이스는 명시적으로 정의되어 있지 않음.
                                                    // 그렇기에, 네임 스페이스는 클래스 어노테이션으로 부터 상속받음: "minecraft"
                                                    // 주입 대상: [SoundEvent] 레지스트리의 "minecraft:ambient.cave"

  // [ManaType] 레지스트리가 존재한다고 가정합니다.          
  @ObjectHolder("neomagicae:coffeinum")
  public static final ManaType coffeinum = null;  // 어노테이션 있음, [public static] 키워드가 있어야함. [final] 은 선택 사항.
                                                    // ManaType 은 이에 맞는 레지스트리가 있음: [ManaType] (커스텀 레지스트리)
                                                    // 리소스 위치가 명시적으로 정의됨: "neomagicae:coffeinum"
                                                    // 주입 대상: [ManaType] 의 "neomagicae:coffeinum"

  public static final Item ENDER_PEARL = null;    // 어노테이션 없음, [public static final] 키워드가 있어야함.
                                                    // Item 은 이에 맞는 레지스트리가 있음: [Item].
                                                    // 필드의 이름이 리소스의 경로임: "ENDER_PEARL" -> "ender_pearl"
                                                    // !! ^ 사용 가능한 필드 이름, 소문자로 자동으로 변환됨.
                                                    // 네임 스페이스는 명시적으로 정의되어 있지 않음.
                                                    // 그렇기에, 네임 스페이스는 클래스 어노테이션으로 부터 상속받음: "minecraft"
                                                    // 주입 대상:  [Item] 레지스트리의 "minecraft:ender_pearl"

  @ObjectHolder("minecraft:arrow")
  public static final ArrowItem arrow = null;     // 어노테이션 있음, [public static] 키워드가 있어야함. [final] 은 선택 사항.
                                                    // ArrowItem 은 이에 맞는 레지스트리가 없음.
                                                    // ArrowItem 의 조상 Item 은 이에 맞는 레지스트리가 있음: [Item]
                                                    // 리소스 위치가 명시적으로 정의됨: "minecraft:arrow"
                                                    // 주입 대상: [Item] 레지스트리의 "minecraft:arrow"                                                   

  public static Block bedrock = null;             // 어노테이션 없음, [public static final] 키워드가 있어야함.
                                                    // 그러나 없음, 그렇기에 이 필드는 무시됨.

  public static final ItemGroup group = null;     // 어노테이션 없음, [public static final] 키워드가 있어야함.
                                                    // ItemGroup 은 이에 맞는 레지스트리가 없음.
                                                    // ItemGroup 의 조상중 맞는 레지스트리가 없음.
                                                    // 그렇기에, 이는 예외를 발생시킴.
}

class UnannotatedHolder { // 이 클래스는 @ObjectHolder 가 없음.
  @ObjectHolder("minecraft:flame")
  public static final Enchantment flame = null;   // 어노테이션 있음, [public static] 키워드가 있어야함. [final]  은 선택 사항..
                                                    // Enchantment 는 이에 맞는 레지스트리가 있음:: [Enchantment].
                                                    // 리소스 위치가 명시적으로 정의됨: "minecraft:flame"
                                                    // 주입 대상: [Enchantment]의 "minecraft:flame"

  public static final Biome ice_flat = null;      // 클래스에 @ObjectHolder 어노테이션이 없음.
                                                    // 그렇기에 이 필드는 무시됨.

  @ObjectHolder("minecraft:creeper")
  public static Entity creeper = null;            // 어노테이션 있음, [public static] 키워드가 있어야함.
                                                    // Entity 는 이에 맞는 레지스트리가 없음.
                                                    // Entity 의 조상중 맞는 레지스트리가 없음.
                                                    // 그렇기에, 이는 예외를 발생시킴.

  @ObjectHolder("levitation")
  public static final Potion levitation = null;   // 어노테이션 있음, [public static] 키워드가 있어야함. [final]  은 선택 사항..
                                                    // Potion 은 이에 맞는 레지스트리가 있음: [Potion].
                                                    // 리소스 위치의 경로는 어노테이션의 value "levitation" 임
                                                    // 네임 스페이스는 명시적으로 정의되어 있지 않음.
                                                    // 클래스에 @ObjectHolder 어노테이션이 없음.
                                                    // 그렇기에, 이는 예외를 발생시킴.
}
```

커스텀 포지 레지스트리 만들기
--------------------------------

모드에서 커스텀 레지스트리를 만들 때 Map<K, V> 를 사용하여 만드는 경우는 꽤나 흔합니다; 그러나 이는 강제적으로 레지스트리에 의존성을 부여하며 사이드간의 데이터 동기화를 수동으로 시켜 주어야 한다는 단점이 있습니다. 커스텀 포지 레지스트리는 강제적인 의존성을 피하고, 자동으로 동기화를 해 주며(설정시 변경 가능) 관리또한 잘 해주는 참 좋은 대안입니다. 등록되는 객체들도 포지 레지스트리 시스템을 사용하니 등록 과정이 동일합니다.

커스텀 포지 레지스트리는 `NewRegistryEvent` 또는 `DeferredRegister` 를 통해 `RegistryBuilder` 를 사용하여 만듭니다. `RegistryBuilder` 는 레지스트리의 이름, 레지스트리에 등록될 객체들의 클래스, 여러 이벤트들에 반응할 콜백들 등 여러가지 설정 가능한 속성들이 있습니다. 새롭게 만들어진 레지스트리들은 `RegistryManager` 에 `NewRegistryEvent` 가 끝나고 등록됩니다. 

레지스트리에 등록될 객체의 클래스는 무조건 `IForgeRegisterEntry` 인터페이스를 구현하여야 합니다, 이 인터페이스는 `#setRegistryName` 과 `#getRegistryName` 메서드를 정의합니다. 이 인터페이스를 바로 구현하는 것 보단 `ForgeRegistryEntry` 를 상속하는 것이 권장됩니다, 이 클래스는 `IForgeRegistryEntry` 인터페이스의 기본적인 구현을 제공합니다. `#setRegistryName(String)` 이 문자열을 인자로 받아 호출될 때, 만약 그 문자열에 네임 스페이스가 포함되어 있지 않다면, 이는 자동으로 해당 모드의 아이디로 추론됩니다.

모든 새로 만들어진 레지스트리들 또한 [이 방법들][등록]을 통해 객체들을 등록하여야 합니다.

### NewRegistryEvent 쓰기

`NewRegistryEvent` 를 사용할 때, `#create` 를 `RegistryBuilder` 를 인자로 호출하면 `Supplier` 로 감싸진 레지스트리가 반환됩니다. 이 감싸진 레지스트리는 모드 버스에 방송된 `NewRegistryEvent` 가 완전히 끝난 이후 접근하실 수 있으며, 그 이전에 접근하실 경우 `null` 이 대신 반환됩니다.

### DeferredRegister 쓰기

`DeferredRegister` 는, 다시 말하지만, 위 이벤트를 사용하는 wrapper 입니다. `DeferredRegister` 를 레지스트리 이름과 모드 아이디를 인자로 받는 오버로드 메서드 `#create` 를 통해 생성하고 이를 상수 필드에 할당한 이후, `DeferredRegister#makeRegistry` 를 통해 레지스트리를 생성하실 수 있습니다. 이 메서드는 레지스트리에 등록될 객체의 클래스와 생성될 레지스트리의 속성 및 설정값들을 담고 있는 `RegistryBuilder` 를 감싸는 `Supplier` 를 인자로 받습니다. `DeferredRegister#makeRegistry` 는 자동으로 `RegistryBuilder#setName` 과 `RegistryBuilder#setType` 을 호출합니다. 해당 메서드는 언제든지 반환될 수 있기 때문에 `IForgeRegistry` 를 `Supplier` 로 감싼 것을 대신 반환합니다. `NewRegistryEvent` 가 끝나기 이전 커스텀 레지스트리에 접근하려고 하면 `null` dㅣ 대신 반환됩니다.

!!! important
    `DeferredRegister#makeRegistry` 는 무조건 `DeferredRegister#register` 가 호출되어 모드 버스에 등록되기 이전에 호출되어야만 합니다. 또한, `#makeRegistry` 는 `#register` 를 통해 등록 되어야만 `NewRegistryEvent` 이벤트 도중 레지스트리를 만들 수 있습니다. 그러니 `#makeRegistry` 를 먼저 하시고 `#register` 를 이후에 하는 것을 잊지 마세요.

누락된 항목 처리하기
------------------------

가끔씩, 모드가 업데이트되거나 제거되었을 때 레지스트리의 객체가 갑자기 없어질 수도 있습니다. 갑자기 없어진 항목을 처리하는 방법을 세번째 레지스트리 이벤트인 `RegistryEvent$MissingMappings` 이벤트로 지정할 수 있습니다. 이 이벤트 도중에 `getMappings` 를 통해 해당 모드의 누락된 매핑 리스트를 받아올 수 있으며, `#getAllMappings` 를 통해 어떤 모드에서 이 오류가 발생했는가와 관련 없이 모든 매핑들을 받아올 수 있습니다.

각 누락된 항목마다, 이 4가지 방법중 하나를 선택하여 어떻게 처리할지 선택할 수 있습니다:

| 동작     | 설명                           |
|:------:|:---------------------------- |
| IGNORE | 누락된 항목을 무시하고 레지스트리 매핑을 버립니다. |
| WARN   | 로그에 경고를 띄웁니다.                |
| FAIL   | 월드를 불러오는 것을 막습니다.            |
| REMAP  | 다른 항목으로 대체합니다.               |

만약 누락된 항목을 처리하는 방법이 지정되지 않았다면 플레이어에게 이에 대해 알리는 기본 동작을 수행하게 됩니다. 플레이어는 월드를 불러오고 싶을 것이기에, REMAP 을 제외한 다른 동작의 경우. 누락된 항목이 다시 추가될 수 있으니 레지스트리의 다른 객체들이 누락된 항목을 대체하는 것을 막습니다.

[ResourceLocation]: ./resources.md#resourcelocation
[등록]: #객체-등록하기
[이벤트]: ./events.md
[블록엔티티]: ../blockentities/index.md