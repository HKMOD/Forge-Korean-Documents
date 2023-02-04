레벨 저장 데이터
================

레벨 저장 데이터는 (또는 World Saved Data, WSD) 캐패빌리티 대신 사용할 수 있는 시스템으로 각 레벨에다가 데이터를 부착시킬 수 있습니다.

선언하기
-----------

월드의 부착시킬 데이터들은 `SavedData` 클래스의 하위 클래스여야만 합니다. 이때 주목해야 할 메서드가 3개가 있는데:

* `save`: NBT 데이터를 레벨에 작성합니다.
* `setDirty`: 레벨의 데이터가 수정된 이후, 게임에 레벨의 데이터가 변경되었음을 알리기 위해 무조건 호출되어야 하는 메서드입니다. 만약 이 메서드를 호출하지 않는다면 `#save` 메서드는 호출되지 않을 것이고, 변경된 데이터는 유지되지 않을 것입니다.

레벨에 추가하기
----------------------

모든 `SavedData` 는 동적으로 불러와지고 레벨에 추가됩니다. 그러다보니 레벨에 추가된 적이 없는 데이터는 존재하지 않습니다.

`SavedData` 는 `DimensionDataStorage` 에서 생성하고 불러옵니다, 이는 `ServerChunkCache#getDataStorage` 또는 `ServerLevel#getDataStorage` 를 통해 접근할 수 있습니다. `DimensionDataStorage#computeIfAbsent` 를 호출하여 레벨 저장 데이터에 접근 또는 생성할 수 있습니다, 이 메서드는 데이터가 존재하지 않으면 생성하고, 존재한다면 그 데이터를 반환합니다.

`DimensionDataStorage#computeIfAbsent` 는 세개의 인자를 받는데, NBT 데이터를 레벨 저장 데이터로 불러오고 반환하는 함수, 새로운 레벨 저장 데이터의 인스턴스를 생성할 `Supplier`, 레벨의 `data` 폴더에 저장될 `.dat` 파일 이름입니다.

예를 들어, "example" 이라는 이름의 데이터를 네더에 추가한다면, `./<level_folder>/DIM-1/data/example.dat` 에 새로운 파일이 생성됩니다, 그리고 코드 구현은 다음과 같이 합니다:

```java
// 어떤 클래스 안에서
public ExampleSavedData create() {
  return new ExampleSavedData();
}

public ExampleSavedData load(CompoundTag tag) {
  ExampleSavedData data = this.create();
  // Load saved data
  return data;
}

// 데이터를 다른곳에서 불러올때
netherDataStorage.computeIfAbsent(this::load, this::create, "example");
```

여러 레벨에서 접근 가능한 레벨 저장 데이터를 만들려면, 단순히 오버월드에 데이터를 추가하시면 됩니다, `MinecraftServer#overworld` 를 사용해 오버월드의 인스턴스를 얻을 수 있습니다. 오버월드는 유일하게 완전히 언로드되는 일이 없는 레벨로, 언제든지 안전하게 접근할 수 있습니다.
