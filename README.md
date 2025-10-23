# JPMetasoundStateSystemDocs

Universal MetaSound state system for Unreal Engine 5 that lets you drive audio behavior using @global enum-based states** and **global key–value parameters**, controlled from **Blueprints** and read directly inside **MetaSound** graphs.

> **TL;DR**: Define an enum (Blueprint or C++). Set its value globally at runtime. In MetaSound, drop a dynamic node (built from your enum) to route/select/trigger behavior automatically as states change.

---

## Features

* **Global enum state store**

  * Accepts **Blueprint** and **C++** enums
  * Values are stored globally by **enum type name**
  * Safe to query from any MetaSound graph at runtime

* **Dynamic MetaSound nodes** (generated from your enum)

  * **Router** – keeps output linked while sound plays; reroutes automatically when the global enum value changes
  * **Selector** – outputs the value at the moment a logic Gate is triggered
  * **Trigger Switcher** – behaves like a classic switch on incoming triggers
  * **GetGlobalValue** – reads a global value by string key (non-enum key–value store)

* **Blueprint utility nodes**

  * **Set Global MetaSound State Value** – set a new value for any enum type
  * **Get/Set MetaSound Global Value** – string-keyed global value API for custom parameters

* **Works with your pipeline**

  * Designed for UE 5.6+
  * Lightweight runtime module; editor helpers optional

---

## Supported Unreal Versions & Platforms

* **Engine**: UE 5.6 (primary).
* **Platforms**: Windows, macOS, Android, iOS (runtime code is platform-agnostic; MetaSound availability may vary by platform).

> If you ship to consoles, the core pattern stays the same; confirm MetaSound feature parity on your target SDK.

---

## Modules

> Replace placeholders with your actual module names if they differ.

| Module                     | Type    | Description                                  |
| -------------------------- | ------- | -------------------------------------------- |
| `JPMetaSoundLibrary`       | Runtime | Core state system & MetaSound node factories |
| `JPMetaSoundLibraryEditor` | Editor  | (Optional) Editor utilities / asset tools    |

---

## Quick Start

### 1) Create an Enum (Blueprint or C++)

Define a gameplay state enum that will drive audio. You may use either a **Blueprint** enum or a **C++** `UENUM(BlueprintType)`.

**Example enum values**: `Exploration`, `Combat`, `Pause`, `Cinematic`.

> The plugin identifies your enum by its **type name**. Keep names stable across refactors.

### 2) Set the Global State (Blueprint)

Use the Blueprint node **Set Global MetaSound State Value**:

* **Enum Type**: pick your enum asset or C++ enum type
* **New Value**: the value you want to broadcast globally
* Call it whenever your game state changes (e.g., entering combat, pausing, etc.)

### 3) Read the State in MetaSound

Open your MetaSound graph and add one of the dynamic nodes. Each node is generated based on the **enum type** you select.

* **Router**

  * After you trigger its *Gate In*, it keeps the chosen output path active while the sound exists.
  * If the **global enum value changes**, the Router **automatically reroutes** to the new output without restarting the sound.

* **Selector**

  * When you pulse *Gate In*, it **samples** the current global enum value and outputs the matching branch **once**.

* **Trigger Switcher**

  * Works like a **classic switch** for incoming triggers. Each enum value maps to a trigger output.

* **GetGlobalValue**

  * Reads a **string-keyed** global value (non-enum). Use this for flexible parameters you want to drive from Blueprint or C++.

### 4) Custom Global Values (Non-Enum)

Use Blueprint nodes **Get/Set MetaSound Global Value** to read/write values by **key**. In MetaSound, use **GetGlobalValue** to consume them.

> Useful for RTPC-like behaviors (e.g., `MusicIntensity`, `LowPassAmount`) without tying them to enums.

---

## Usage Patterns

### Continuous State-Driven Music

* Set `MusicState` globally when gameplay shifts (Exploration → Combat).
* In your music MetaSound, use **Router** to sustain the current layer and **auto-reroute** when the state changes, crossfading or blending as needed in the graph.

### One-Shot UI Sounds

* Use **Selector** to pick the correct clip on a button press, sampling the UI mode state (e.g., `Settings`, `Store`, `Inventory`).

### Trigger Fan-Out

* With **Trigger Switcher**, feed a single trigger and branch to different SFX families depending on the state (e.g., weapon categories).

### Parameter Feeds

* Write `Stamina01` or `Heat` by key using **Set MetaSound Global Value** from gameplay code; read it in MetaSound with **GetGlobalValue** to modulate filters, gains, etc.

---

## Best Practices

* **Keep enum types stable**: The system keys the storage by **enum type name**; renaming types can break lookups.
* **Blueprint & C++ parity**: Prefer `UENUM(BlueprintType)` for C++ enums so the same enum can be selected in Blueprints and MetaSound node pickers.
* **Centralize state writes**: Encapsulate state transitions in a single subsystem or manager to avoid race conditions.
* **Threading**: MetaSound graphs may read while gameplay writes; avoid rapid oscillations without debouncing or smoothing in your graph logic.
* **Value keys**: For the string-keyed store, define constants to avoid typos (`"MusicIntensity"` vs `"musicIntensity"`).

---

## API Overview

> The exact C++ class and function names can differ per version. Blueprint usage is canonical for stability. If you expose C++ helpers, document them here.

### Blueprint Nodes

* **Set Global MetaSound State Value**

  * Inputs: `EnumType`, `NewValue`
  * Effect: Updates the global enum store for that type

* **Get MetaSound Global Value** / **Set MetaSound Global Value**

  * Inputs: `Key` (String), `Value` (variant/number/string as implemented)
  * Effect: Reads/writes a string-keyed global parameter

### MetaSound Nodes

* **Router (Enum-Driven)**
* **Selector (Enum-Driven)**
* **Trigger Switcher (Enum-Driven)**
* **GetGlobalValue (Key-Driven)**

---

## Example Blueprint Flow

1. On game state change, call **Set Global MetaSound State Value** with enum `EMusicState = Combat`.
2. In your `Music_Master` MetaSound, the **Router** node re-routes to the *Combat* branch at runtime.
3. Use crossfades or filters inside the graph for seamless transitions.

---

## Example Enum (C++)

```cpp
// Define a gameplay enum usable in Blueprints and MetaSound selections
UENUM(BlueprintType)
enum class EGamePhase : uint8
{
    Exploration UMETA(DisplayName = "Exploration"),
    Combat      UMETA(DisplayName = "Combat"),
    Pause       UMETA(DisplayName = "Pause"),
    Cinematic   UMETA(DisplayName = "Cinematic")
};
```

> Set this enum from Blueprint using **Set Global MetaSound State Value**. The MetaSound nodes generated from `EGamePhase` will then react to your changes.

---

## Replication & Networking

* The global store itself is engine-process local. For multiplayer, **replicate your authoritative state** (e.g., via a GameState/Subsystem) and write to the global store **on the client** that should hear the change.
* Keep server-authoritative logic clean: replicate enum values, not audio graph details.

---

## Performance Notes

* Reads inside MetaSound are lightweight. The **Router** listens for changes and updates its routing without tearing down the graph.
* Avoid excessively high-frequency writes to the same key/enum; consider debouncing.

---

## Limitations

* Renaming enum **types** changes the storage key; migrate carefully.
* Dynamic node menus list only **BlueprintType** enums and known C++ enums.
* String-keyed values are not type-enforced unless your project wraps accessors with typed helpers.


