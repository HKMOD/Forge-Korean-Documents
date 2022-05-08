월드 저장 데이터
================

월드 저장 데이터 (또는 World Saved Data, WSD) 캐패빌리티 대신 사용할 수 있는 시스템으로 각 월드에다가 데이터를 부착시킬 수 있습니다.

선언하기
-----------

각 WSD 구현들은 `WorldSavedData` 클래스의 하위 클래스여야만 합니다. 이때 주목해야 할 메서드가 3개가 있는데:

* `save`: NBT 데이터를 월드에 작성합니다.
* `load`: 이전에 저장된 NBT 데이터를 읽어들입니다.
* `setDirty`: 월드의 데이터가 수정된 이후, 게임에 월드의 데이터가 변경되었음을 알리기 위해 무조건 호출되어야 하는 메서드입니다. 만약 이 메서드를 호출하지 않는다면 `#save` 메서드는 호출되지 않을 것이고, 변경된 데이터는 유지되지 않을 것입니다.

The constructor of the class also requires a `String`. This is the name of the `.dat` file stored within the `data` folder for the implemented world. For example, if a WSD was named "example" within the Nether, then a file would be created at `./<level_folder>/DIM-1/data/example.dat`.

Attaching to a World
----------------------

Any `WorldSavedData` is loaded and/or attached to a world dynamically. As such, if one is never created on a world, then it will not exist. 

`WorldSavedData`s are created and loaded from
the `DimensionSavedDataManager`, which can be accessed by either `ServerChunkProvider#getDataStorage` or `ServerWorld#getChunkSource`. From there, you can get or create an instance of your WSD by calling `DimensionSavedDataManager#computeIfAbsent`. This will attempt to get the current instance of the WSD if present or create a new one and load all available data.

To persist a WSD across worlds, a WSD should be attached to the Overworld dimension, which can be obtained from `MinecraftServer#overworld`. The Overworld is the only dimension that is never fully unloaded and as such makes it perfect to store multi-world data on.
