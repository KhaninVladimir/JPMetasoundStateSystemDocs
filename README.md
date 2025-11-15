# Metasound State Machine Plugin

A lightweight and powerful **Enum-driven State System** for Unreal Engine MetaSounds.  
Supports both **Global** and **Component-Scoped** state logic, dynamic Enum-based MetaSound nodes, and RTPC-style global values.  
Includes simple debugging tools.

---

## Features

- **Global State**: One shared Enum state for the entire game (music, UI, ambience).
- **Component State**: Per-`UAudioComponent` Enum state; perfect for weapons, characters, objects.
- **Dynamic MetaSound Nodes**: Router, Selector, Trigger Switcher, GetGlobalValue.
- **Global Custom Values**: Float/int/bool values stored by string key (RTPC-like).
- **Blueprint Integration**: Set/Read global and component states from Blueprint.
- **Debugging**:  
  - `jp.audio.debug.log`  
  - `jp.audio.debug.print`

---

## Table of Contents

1. [Concept Overview](#concept-overview)  
2. [Blueprint Nodes](#blueprint-nodes)  
   - [Set Global State](#1-set-global-state)  
   - [Set Component State](#2-set-component-state)  
   - [Set/Get Global Value](#3-setget-global-value)  
3. [MetaSound Nodes](#metasound-nodes)  
   - [Router](#1-router)  
   - [Selector](#2-enum-selector)  
   - [Trigger Switcher](#3-trigger-switcher)  
   - [GetGlobalValue](#4-getglobalvalue)  
4. [Quick Tutorials](#quick-tutorials)  
   - [Global Music State](#global-music-state)  
   - [Per-Component Weapon State](#per-component-weapon-state)  
5. [Debugging](#debugging)

---

## Concept Overview

The plugin provides a unified system for managing audio states inside MetaSounds using **Enums**.

### Global State

- Defines one Enum value shared by the whole project.
- All MetaSounds listening to the same Enum see the same value.
- Ideal for music states, UI pages, ambience modes.

### Component State

- State is linked to a specific `UAudioComponent`.
- Each sound source can have an independent state.
- Perfect for characters, weapons, interactable objects.

### Global Values

- Non-Enum values stored by a string key.
- Behave like RTPC parameters.
- Useful for continuous control (volume, filters, intensity, etc.).

---

## Blueprint Nodes

### 1. Set Global State

Set Enum value in **global storage**.

---

### 2. Set Component State

Set Enum value **for a specific Audio Component**.
> Only affects MetaSounds executed by this specific `UAudioComponent`.

---

### 3. Set/Get Global Value

RTPC-style value storage for MetaSounds.

- **Set Metasound Global Value**  
  Stores float/int/bool under a string key.

- **Get Metasound Global Value**  
  Reads the stored value.

---

## MetaSound Nodes

### 1. Router

Listens for Enum changes and fires trigger outputs.

**Behavior:**

- `GateIn` starts monitoring.
- When state changes → fires the output matching the new Enum value.
- Can operate in Global or Component scope (depending on node variant or settings).

**Use Cases:**

- Music layer switching  
- UI theme changes  
- Dynamic ambience states  

---

### 2. Enum Selector
(float, FTime, bool, int32, FWaveAsset)
Outputs a value based on current Enum state when triggered.

**Use Cases:**

- Selecting mix levels  
- Picking filter/resonance per state  
- Mapping state → numeric parameter  

---

### 3. Trigger Switcher

MetaSound equivalent of “Switch on Enum”.

**Behavior:**

- On `GateIn`, fires exactly **one** output that matches the current Enum.

**Use Cases:**

- One-shot stingers  
- Selective SFX triggers  
- State-driven transitions  

---

### 4. GetGlobalValue

Reads a float/int/bool stored by `Set Metasound Global Value`.

**Use Cases:**

- Continuous gameplay-driven parameters  
- Intensity curves  
- Player-dependent mix changes  

---

## Quick Tutorials

### Global Music State

1. Create enum `EGameMusicState` with:  
   `Menu`, `Exploration`, `Combat`.

2. In Blueprint:

   - On menu:  
     `Set Global Metasound State → Menu`
   - On gameplay:  
     `Set Global Metasound State → Exploration`
   - On combat:  
     `Set Global Metasound State → Combat`

3. In MetaSound:

   - Add **Router** with `EGameMusicState`.
   - Connect `On_Menu`, `On_Exploration`, `On_Combat` to the appropriate music sections.
   - Trigger `GateIn` on start.

---

### Per-Component Weapon State

1. Create enum `EWeaponState`:  
   `Idle`, `Firing`, `Reload`.

2. In weapon Blueprint:

   - Call **Set Component Metasound State**.
   - Pass the weapon’s `AudioComponent`.
   - Set values based on weapon logic (Idle, Firing, Reload).

3. In the weapon’s MetaSound:

   - Use Router/Selector bound to **Component** state.
   - Each weapon instance behaves independently.

---

## Debugging

Two debugging modes are available:

### `jp.audio.debug.log`

Prints detailed state changes to the Output Log.

### `jp.audio.debug.print`

Shows on-screen debug messages with current state and updates.

Use them to verify:

- Global and Component state changes  
- Router and Selector reactions  
- Current values and keys
