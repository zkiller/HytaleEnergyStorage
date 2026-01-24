## 📦 Hytale EnergyStorage

[EnergyStorage Curse Project](https://www.curseforge.com/hytale/mods/energystorage)

Energy Storage API is a lightweight developer‑focused library for Hytale mods. It provides a fully‑featured, component‑based energy system that other mods can easily integrate into their machines, blocks, entities, or custom gameplay systems. This mod does **not** add gameplay content by itself. Instead, it exposes a clean and extensible API built around Hytale’s official `Component` systems [ECS](https://hytalemodding.dev/en/docs/guides/plugin/block-components), ensuring maximum compatibility and stability across mods. Use this library when you want to give your machines the ability to store, receive, or extract energy in a predictable and standardized way.

***

## ⚡ Features

*   `EnergyStorageBlockComponent` using Hytale’s official `Component` systems [ECS](https://hytalemodding.dev/en/docs/guides/plugin/block-components)
*   `EnergyStorageEntityComponent` using Hytale’s official `Component` systems [ECS](https://hytalemodding.dev/en/docs/guides/plugin/block-components)
*   Automatic serialization and validation
*   Configurable capacity, max receive, and max extract rates
*   Safe and deterministic energy transfer logic
*   Designed as a shared dependency for other mods

***

## 🧩 Usage Examples

**1\. Adding an Energy Component to a Block Entity**
* **EnergyStored** - The amount of energy initially stored in the machine when it is created.
* **MaxEnergy** - The maximum energy capacity the machine can hold.
* **MaxReceive** - - The maximum amount of energy the machine can receive per tick (set to 0 to disable receiving).
* **MaxExtract** - - The maximum amount of energy the machine can extract per tick (set to 0 to disable extracting).


**Adding Component**
```json
"Components": {
    "EnergyStorageComponent": {
    "EnergyStored": 0,
    "MaxEnergy": 80000,
    "MaxReceive": 0,
    "MaxExtract": 1000
  }
}
```

**SolarGenerator.json**
```json
{
  "TranslationProperties": {
    "Name": "items.Solar_Generator.name"
  },
  "ItemLevel": 10,
  "MaxStack": 100,
  "Icon": "Icons/ItemsGenerated/Solar_Generator_Block.png",
  "Categories": [
    "Blocks.Rocks"
  ],
  "PlayerAnimationsId": "Block",
  "Set": "SolarGenerator",
  "BlockType": {
    "State": {},
    "Material": "Solid",
    "DrawType": "Cube",
    "Group": "Stone",
    "Flags": {},
    "VariantRotation": "NESW",
    "Gathering": {
      "Breaking": {
        "GatherType": "Rocks",
        "ItemId": "SolarGenerator"
      }
    },
    "Interactions":{
      "Use": {
        "Interactions": []
      }
    },
    "BlockEntity": {
      "Components": {
        "EnergyStorageComponent": {
          "EnergyStored": 0,
          "MaxEnergy": 80000,
          "MaxReceive": 0,
          "MaxExtract": 1000
        },
        "examplemod:solar_generator": {
          "ProductionPerTick": 80,
          "RequiresDaylight": true
        }
      }
    },
    "BlockParticleSetId": "Stone",
    "ParticleColor": "#737055",
    "BlockSoundSetId": "Stone",
    "Aliases": [
      "stone",
      "stone00"
    ],
    "BlockBreakingDecalId": "Breaking_Decals_Rock",
    "Textures": [
      {
        "Up": "BlockTextures/SolarGenerator/top.png",
        "Side": "BlockTextures/SolarGenerator/side.png",
        "Down": "BlockTextures/SolarGenerator/bottom.png"
      }
    ]
  },
  "ResourceTypes": [
    {
      "Id": "Rock"
    },
    {
      "Id": "Rock_Stone"
    }
  ],
  "Tags": {
    "Type": [
      "Rock"
    ]
  },
  "ItemSoundSetId": "ISS_Blocks_Stone"
}
```
**ExampleMod.java**
```java
package dev.zkiller.exemplemod;

import com.hypixel.hytale.component.ComponentType;
import com.hypixel.hytale.logger.HytaleLogger;
import com.hypixel.hytale.server.core.plugin.JavaPlugin;
import com.hypixel.hytale.server.core.plugin.JavaPluginInit;
import com.hypixel.hytale.server.core.universe.world.storage.ChunkStore;
import dev.zkiller.machinefactory.generators.solar.SolarGeneratorComponent;
import dev.zkiller.machinefactory.generators.solar.SolarGeneratorSystem;

import javax.annotation.Nonnull;

public class ExampleMod extends JavaPlugin {

    private static ExampleMod instance;
    private static final HytaleLogger LOGGER = HytaleLogger.forEnclosingClass();

    private ComponentType<ChunkStore, SolarGeneratorComponent> solarGeneratorType;

    public ExampleMod(@Nonnull JavaPluginInit init) {
        super(init);
        instance = this;
    }

    @Override
    protected void setup() {

        this.solarGeneratorType = this.getChunkStoreRegistry().registerComponent(
                SolarGeneratorComponent.class,
                "examplemod:solar_generator",
                SolarGeneratorComponent.CODEC
        );

        this.getChunkStoreRegistry().registerSystem(
                new SolarGeneratorSystem(this.solarGeneratorType)
        );

        LOGGER.atInfo().log("Solar Generator component and system registered successfully!");
    }

    public static MachineFactory getInstance() {
        return instance;
    }

    public ComponentType<ChunkStore, SolarGeneratorComponent> getSolarGeneratorType() {
        return solarGeneratorType;
    }
}
```

**SolarGeneratorComponent.java**
```java
package dev.zkiller.examplemod.generators.solar;

import com.hypixel.hytale.codec.Codec;
import com.hypixel.hytale.codec.KeyedCodec;
import com.hypixel.hytale.codec.builder.BuilderCodec;
import com.hypixel.hytale.codec.validation.Validators;
import com.hypixel.hytale.component.Component;
import com.hypixel.hytale.server.core.universe.world.storage.ChunkStore;

import javax.annotation.Nonnull;
import javax.annotation.Nullable;

public final class SolarGeneratorComponent implements Component<ChunkStore> {

    public long productionPerTick = 80;

    @Nonnull
    public static final BuilderCodec<SolarGeneratorComponent> CODEC =
            BuilderCodec.builder(SolarGeneratorComponent.class, SolarGeneratorComponent::new)

                    .append(new KeyedCodec<>("ProductionPerTick", Codec.LONG),
                            (c, v) -> c.productionPerTick = v,
                            c -> c.productionPerTick)
                    .addValidator(Validators.greaterThanOrEqual(0L))
                    .documentation("Energy produced per tick.")
                    .add()
                    .build();

    public SolarGeneratorComponent() {}

    public SolarGeneratorComponent(SolarGeneratorComponent other) {
        this.productionPerTick = other.productionPerTick;
    }

    @Nullable
    @Override
    public Component<ChunkStore> clone() {
        return new SolarGeneratorComponent(this);
    }
}
```

**SolarGeneratorSystem.java**
```java
package dev.zkiller.exemplemod.generators.solar;

import com.hypixel.hytale.component.*;
import com.hypixel.hytale.component.query.Query;
import com.hypixel.hytale.component.system.tick.EntityTickingSystem;
import com.hypixel.hytale.server.core.universe.world.storage.ChunkStore;
import dev.zkiller.energystorage.components.EnergyStorageBlockComponent;

import javax.annotation.Nonnull;

public final class SolarGeneratorSystem extends EntityTickingSystem<ChunkStore> {

    private final ComponentType<ChunkStore, SolarGeneratorComponent> solarType;

    public SolarGeneratorSystem(ComponentType<ChunkStore, SolarGeneratorComponent> solarType) {
        this.solarType = solarType;
    }

    @Nonnull
    @Override
    public Query<ChunkStore> getQuery() {
        // Le système ne tick que les entités ayant un SolarGeneratorComponent + EnergyStorageBlockComponent
        return Query.and(EnergyStorageBlockComponent.getComponentType(), this.solarType);
    }

    @Override
    public void tick(
            float dt,
            int index,
            @Nonnull ArchetypeChunk<ChunkStore> chunk,
            @Nonnull Store<ChunkStore> store,
            @Nonnull CommandBuffer<ChunkStore> cmd
    ) {
        SolarGeneratorComponent solar = chunk.getComponent(index, this.solarType);
        EnergyStorageBlockComponent energy = chunk.getComponent(index, EnergyStorageBlockComponent.getComponentType());
        if (solar == null || energy == null) return;

        long produced = solar.productionPerTick;
        
        long inserted = energy.receiveEnergyInternal(produced, false);
        
        // Console Debug
        System.out.println("[MachineFactory] Inserted: " + inserted +
                " FE | Stored: " + energy.getEnergyStored() +
                "/" + energy.getMaxEnergyStored());
    }
}
```
***

## 🛠️ Who Is This For?

This library is intended exclusively for developers who want to:

*   Build machines that store or transfer energy
*   Create automation systems
*   Implement generators, batteries, cables, or power networks
*   Share a common energy standard across multiple mod

**If your mod needs energy, this API gives you a clean, stable foundation.**
