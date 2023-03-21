파티클
=========

파티클은 게임의 시각적 효과 중 하나로 몰입감을 증진하는 데 이용됩니다. 하지만 파티클은 유용한 만큼 사용하실 때 주의를 기울이셔야 합니다.

파티클 만들기
-------------------

파티클은 유저에게 보여주기 위한 [**클라이언트 전용**][사이드] 코드와 서버에서 데이터를 동기화 및 조회하기 위한 공용 코드로 나누어져 있습니다.

| 클래스              | 사이드   | 설명                                                                      |
|:---------------- |:-----:|:----------------------------------------------------------------------- |
| ParticleType     | 양쪽    | 양측에서 파티클의 정의를 참조할 때 사용되는 레지스트리 객체                                       |
| ParticleOptions  | 양쪽    | 파티클의 정보를 클라이언트들과 동기화할 때 사용되는 데이터 꾸러미                                    |
| ParticleProvider | 클라이언트 | `ParticleType` 에서 등록한 `ParticleOptions` 로 `Particle` 의 인스턴스를 만들어주는 팩토리. |
| Particle         | 클라이언트 | 클라이언트에서 유저에게 파티클을 보여주기 위한 렌더링 코드.                                       |

### ParticleType

`ParticleType` 은 파티클의 종류를 정의하는 [레지스트리] 객체로, 양측 사이드에 파티클의 정의에 대한 참조를 제공합니다.

각 `ParticleType` 은 두 개의 파라미터가 있는데, 파티클을 거리에 상관없이 렌더링하도록 하는 `overrideLimiter`, 그리고 서버에서 보낸 `ParticleOptions`를 읽는 `ParticleOptions$Deserializer` 가 있습니다. `ParticleType` 은 추상 클래스이고, `#codec` 함수를 정의하셔야 합니다. 이는 파티클의 `ParticleOptions`를 인코딩하고 디코딩하는 데 사용됩니다.

!!! note
    `ParticleType#codec` 은 바닐라 마인크래프트의 바이옴의 `codec` 에서만 사용됩니다.

일반적인 상황에선 파티클만의 데이터를 클라이언트에 전송할 필요가 없을 겁니다. 이럴 땐 대신 `SimpleParticleType` 의 인스턴스를 만드시는 것을 추천드립니다: `ParticleType` 과 `ParticleOptions` 의 구현 중 하나로 파티클 종류 정보 이외의 데이터를 전송하지 않습니다. 바닐라 마인크래프트의 파티클들은 레드스톤 가루 등을 제외하곤 `SimpleParticleType`을 사용합니다.

!!! important
    클라이언트에서만 참조할 파티클은 `ParticleType`을 사용하지 않고 구현할 수 있지만, 이런 경우 직접 `ParticleEngine`을 응용하여 직접 렌더링하셔야 합니다.

### ParticleOptions

`ParticleOptions` 는 각 파티클에 들어가는 데이터를 담고 있습니다. 또한 서버에서 클라이언트에 파티클의 정보를 보낼 때도 사용됩니다. 파티클을 생성하는 모든 메서드들은 `ParticleOptions`를 인자로 받아 생성할 파티클의 종류와 데이터를 받아옵니다.

`ParticleOptions` 는 메서드 세 개로 분석할 수 있습니다:

| 메서드            | 설명                              |
|:-------------- |:------------------------------- |
| getType        | 파티클의 종류(`ParticleType`)를 반환합니다. |
| writeToNetwork | 클라이언트에 전송할 파티클 데이터를 버퍼에 작성합니다.  |
| writeToString  | String 에 파티클 데이터를 작성합니다.        |

`ParticleOptions` 의 인스턴스는 는 필요할 때 마다 생성하셔도 되고, 아니면 `SimpleParticleType` 처럼 singleton 패턴을 따르셔도 됩니다.

#### ParticleOptions$Deserializer

클라이언트에서 `ParticleOptions`를 해석하거나, 작성 중인 커맨드의 파티클의 데이터를 참조하기 위해선 파티클의 데이터를 무조건 `ParticleOptions$Deserializer`를 이용해 해석하셔야 합니다. `ParticleOptions$Deserializer` 에 정의된 각 메서드들은 `ParticleOptions` 의 인코딩 메서드들과 짝을 이룹니다:

| 메서드         | ParticleOptions 의 인코더 | 설명                                         |
|:----------- |:---------------------:|:------------------------------------------ |
| fromCommand | writeToString         | 문자열에 작성된 파티클 데이터를 해독합니다, 일반적으로 명령어에 사용됩니다. |
| fromNetwork | writeToNetwork        | 버퍼에 작성된 파티클 데이터를 해독합니다.                    |

`Deserializer` 는 `ParticleType` 의 생성자의 인자로 사용됩니다.

### Particle

`Particle` 은 전송된 데이터를 활용해 파티클을 화면에 그리는데 필요한 렌더링 코드를 제공합니다. 새 `Particle`을 정의하기 위해선 아래 두 메서드들을 구현해야 합니다:

| 메서드           | 설명                                  |
|:------------- |:----------------------------------- |
| render        | 파티클을 화면에 그립니다.                      |
| getRenderType | 파티클을 그릴 때 사용할 `RenderType` 을 반환합니다. |

그저 파티클의 텍스쳐를 그리기만 하면 되는 `Particle` 들은 `TextureSheetParticle`를 사용할 수 있습니다. `#getRenderType` 은 여전히 재정의해야 하지만, `#render` 는 지정된 텍스쳐 스프라이트를 알맞은 위치에 그리도록 미리 구현되어 있습니다.

#### ParticleRenderType

`ParticleRenderType` 은 `RenderType` 과 유사하게, 존재하는 모든 파티클의 렌더링 사전 설정과 정리를 담당합니다. 이후 `Tesselator`를 거쳐 화면에 한 번에 전송합니다. 바닐라 마인크래프트는 아래 여섯 개의 `ParticleRenderType`을 정의합니다:

| Render Type                | 설명                                                                  |
|:-------------------------- |:------------------------------------------------------------------- |
| TERRAIN_SHEET              | 텍스쳐가 블록인 파티클을 렌더링할 때 사용합니다.                                         |
| PARTICLE_SHEET_OPAQUE      | 텍스쳐가 불투명한 파티클을 그릴 때 사용합니다.                                          |
| PARTICLE_SHEET_TRANSLUCENT | 투명도가 있는 텍스쳐를 사용하는 파티클을 그릴 때 사용합니다.                                  |
| PARTICLE_SHEET_LIT         | `PARTICLE_SHEET_OPAQUE` 와 동일하지만 파티클 전용 쉐이더를 사용하지 않습니다.              |
| CUSTOM                     | 블렌딩과 깊이 마스크는 사전 설정해 주지만 그 외에는 `Particle#render` 를 통해 구현해 제공하지 않습니다. |
| NO_RENDER                  | 아예 렌더링하지 않습니다.                                                      |

새로운 종류의 `ParticleRenderType`을 구현하는 것은 여러분들의 과제로 남겨드리죠. ;)

### ParticleProvider

`Particle` 의 인스턴스는 일반적으로 `ParticleProvider`를 통해 생성됩니다. 이 인터페이스는 메서드 `#createParticle`을 정의하여 `ParticleOptions`, 레벨, 위치, 그리고 속도를 이용해 `Particle` 의 인스턴스를 만듭니다. `Particle` 은 특정 파티클의 종류에 종속된 것이 아니기 때문에 원하신다면 재사용하셔도 됩니다.

`ParticleProvider` 는 무조건 모드 버스에서 방송되는 `ParticleFactoryRegisterEvent` 가 방송될 때 `ParticleEngine#register(ParticleType, ParticleProvider)`를 호출하여 등록되어야 합니다.

!!! important
    `ParticleFactoryRegisterEvent` 는 클라이언트 전용 코드입니다! 그렇기에 `DistExecutor` 또는 `@EventBusSubscriber`를 통해 등록하세요!

!!! important
    `ParticleProvider`를 `ParticleEngine#register(ParticleType, ParticleProvider)`를 호출하여 직접 등록하시는 것은 텍스쳐를 추가하지 않을 때 이용합니다. 텍스쳐를 추가하신다면 아래 섹션을 충분히 숙지하신 후 `ParticleEngine#register(ParticleType, SpriteParticleRegistration)`을 대신 호출하세요!

#### ParticleDescription, SpriteSet, SpriteParticleRegistration

`PARTICLE_SHEET_OPAQUE`, `PARTICLE_SHEET_TRANSLUCENT`, 그리고 `PARTICLE_SHEET_LIT` 은 위에서 기술한 방법으로 등록한 `ParticleProvider` 에서 쓰는 텍스쳐를 이용할 수 없습니다, 왜냐하면 셋 모두 `ParticleEngine` 이 불러오는 스프라이트들을 바로 사용하기 때문입니다. 그렇기에 사용하실 텍스쳐를 따로 등록하셔야 합니다. 바닐라 마인크래프트에서는 오직 `TextureSheetParticle` 만이 파티클용 텍스쳐를 사용하기 때문에 당신께서 만드시는 파티클이 `TextureSheetParticle` 의 자식 클래스라고 가정하겠습니다.

파티클에 텍스쳐를 추가하기 위해선 `assets/<modid>/particles` 에 새 JSON 파일을 추가하셔야 합니다. 이 파일은 `ParticleDescription` 으로 비직렬화 됩니다. 이 파일의 이름은 `ParticleType` 의 레지스트리 이름과 동일해야 합니다. 각 JSON 파일은 `ResourceLocation` 들을 담고 있는 `textures` 라는 이름의 배열을 저장합니다. 이때, 각 `ResourceLocation` 들은 텍스쳐 이미지 파일 `assets/<modid>/textures/particle/<path>.png`를 가리킵니다.

```js
{
  "textures": [
    // 가리키는 대상:
    // assets/mymod/textures/particle/particle_texture.png
    "mymod:particle_texture",
    // 여기서 정의하는 텍스쳐들은 그리는 순서에 따라 작성되어야 합니다.
    // particle_texture를 먼저 그리고, 그다음 particle_texture2, 이런 식으로요
    "mymod:particle_texture2"
  ]
}
```

`TextureSheetParticle` 의 자식 클래스에서 파티클의 텍스쳐를 사용하기 위해선 `SpriteSet`, 또는 `SpriteSet` 에서 얻은 `TextureAtlasSprite` 의 참조가 필요합니다. `SpriteSet` 은 각 `ParticleDescription` 에서 등록한 텍스쳐들의 리스트를 보유합니다. `SpriteSet` 은 `TextureAtlasSprite`를 반환하는 메서드 2개를 정의하는데, 첫 번째 메서드는 두 개의 정수를 인자로 받습니다. 이는 텍스쳐가 시간에 따라 변화할 수 있도록 하기 위함입니다. 두 번째 메서드는 무작위 텍스쳐를 반한하는 것으로, `Random` 의 인스턴스를 인자로 받습니다. `TextureSheetParticle` 은 인자로 전달된 `SpriteSet` 에서 무작위 텍스쳐를 가져와 반환하는 `#pickSprite` 와, `Particle` 의 나이와 수명으로 텍스쳐를 가져와 반환하는 `#setSpriteFromAge`를 제공합니다.

파티클의 텍스쳐들을 등록하기 위해선, `SpriteParticleRegistration` 의 인스턴스를 `ParticleEngine#register(ParticleType, SpriteParticleRegistration)` 에 전달하여 호출해 주세요. `SpriteParticleRegistration` 은 `SpriteSet`을 받아 `ParticleProvider` 의 인스턴스를 만드는 메서드 `#create`를 정의하는 함수형 인터페이스 입니다. 이때 `ParticleProvider` 의 하위 클래스의 생성자가 `SpriteSet`을 인자로 받도록 한 다음, `#create` 메서드에서 이를 전달하여 `ParticleProvider` 의 인스턴스를 만들고 반환하는 것이 가장 간단합니다.

파티클 소환하기
-------------------

파티클은 사이드에 따라 소환하는 방법에 차이가 있습니다. `ClientLevel` 은 `#addParticle` 또는 (엄청 멀리 나가도 보이도록)`#addAlwaysVisibleParticle` 을, `ServerLevel` 은 `#sendParticles`을 호출하여 파티클을 소환할 수 있습니다. `ClientLevel` 용 메서드를 서버에서 호출하면 아무런 일도 일어나지 않습니다.

[사이드]: ../concepts/sides.md
[레지스트리]: ../concepts/registries.md
