## 📦 Hytale EnergyStorage

Energy Storage API is a lightweight developer‑focused library for Hytale mods. It provides a fully‑featured, component‑based energy system that other mods can easily integrate into their machines, blocks, entities, or custom gameplay systems. This mod does **not** add gameplay content by itself. Instead, it exposes a clean and extensible API built around Hytale’s official `ComponentRegistryProxy`, `ComponentType`, and `BuilderCode`c systems, ensuring maximum compatibility and stability across mods. Use this library when you want to give your machines the ability to store, receive, or extract energy in a predictable and standardized way.

***

## ⚡ Features

*   `IEnergyStorage` interface with full simulation support
*   `EnergyStorageComponent` using Hytale’s `&nbsp;BuilderCodec`
*   Automatic serialization and validation
*   Configurable capacity, max receive, and max extract rates
*   Safe and deterministic energy transfer logic
*   Easy integration with any `EntityStore`
*   Clean `EnergyModule` registry for cross‑mod compatibility
*   Designed as a shared dependency for other mods

***

## 🧩 Usage Examples

**1\. Adding an Energy Component to a Block Entity**

**Java:**

```
EnergyStorageComponent energy = new EnergyStorageComponent(
    0,        // initial energy
    50000,    // max capacity
    2000,     // max receive per tick
    2000      // max extract per tick
);

blockEntity.addComponent(
    EnergyModule.get().getEnergyComponentType(),
    energy
);
```

**Json:**

```
{
    "components": {
        "energystorage:energy": {
            "EnergyStored": 2500
            "MaxEnergy": 10000,
            "MaxReceive": 1000,
            "MaxExtract": 500
        }
    }
}
```

**2\. Receiving Energy**

```
long inserted = energy.receiveEnergy(1000, false);
System.out.println("Inserted: " + inserted);
```

Simulation mode (no mutation):

```
long possible = energy.receiveEnergy(1000, true);
```

**3\. Extracting Energy**

```
long extracted = energy.extractEnergy(500, false);
System.out.println("Extracted: " + extracted);
```

**4\. Checking Storage State**

```
if (energy.isFull()) {
    System.out.println("Storage is full!");
}

if (energy.isEmpty()) {
    System.out.println("Storage is empty!");
}

float ratio = (float) energy.getEnergyStored() / energy.getMaxEnergyStored();
```

**5\. Accessing the Component Type (for other mods)**

```
ComponentType<EntityStore, EnergyStorageComponent> type =
        EnergyModule.get().getEnergyComponentType();
```

***

## 🛠️ Who Is This For?

This library is intended exclusively for developers who want to:

*   Build machines that store or transfer energy
*   Create automation systems
*   Implement generators, batteries, cables, or power networks
*   Share a common energy standard across multiple mod

If your mod needs energy, this API gives you a clean, stable foundation.
