# USQLite Unreal Engine Plugin Documentation

## Overview

`USQLite` is a powerful Unreal Engine plugin designed to handle **runtime data persistence** with **SQLite databases**, without requiring the developer to write SQL queries or deal with Blueprint complexity. It uses reflection to automatically generate SQL commands from object properties and supports complex features like multithreaded operations, data versioning, object reference serialization, and custom load screens.

---

# [🎬 >> QUICK TUTORIAL #1](https://www.youtube.com/watch?v=902oSGdhSdY)
# [🎬 >> QUICK TUTORIAL #2](https://www.youtube.com/watch?v=eXiVu5TyPWc)

# [📼 >> DOWNLOAD PROJECT HERE!](https://www.dropbox.com/scl/fi/70hajotbf3nhyzn7k7m2q/HKH_USQLite_Demo_UE5.6.zip?rlkey=utp8tf1b59j6layq3zp3m3k66&st=h7rdih2k&dl=0)  

<br>

---

## Key Features

- ✅ **No SQL Required** – Auto-generate SQL from UObjects, Actors, or Components.
- 🚀 **Multithreaded Execution** – Run save/load operations in background threads.
- 🔄 **Blueprint-Friendly** – Expose all functionality to Blueprints.
- 💾 **Data Versioning** – Store historical versions of objects for compatibility.
- 🔗 **Reference Saving** – Supports saving and restoring UObject, Actor, and Component references.
- 🎞 **Custom Load Screens** – Built-in splash screen, blur, or movie support during loading.
- 📈 **Progress Feedback** – Real-time progress bar system and events.
- 🔄 **Runtime & Editor Integration** – Create/edit SQLite database assets directly in Unreal Editor.
- 🎮 **Game-Ready** – Ensures thread-safe operations while gameplay continues.

---

## Class Overview

### USQLite_Settings

A configuration class to control default behavior of the SQLite system.

```cpp
UPROPERTY(EditDefaultsOnly, config)
bool DeepLogs;
```

Enables verbose logging of SQL operations.

---

### USQLite

The core class managing database operations and schema generation.

#### Properties
- `DB_VERSION`, `DB_VERSIONS`: Table versioning system.
- `DB_MODE`: Runtime threading mode (sync/threaded).
- `DB_FILE`: Absolute path to the SQLite database.
- `DB_SelectCondition`, `DB_DeleteCondition`: Optional WHERE clauses.
- `DBS_QUEUE`, `DBL_QUEUE`: SQL command queues for save/load.
- `DB_REDIRECTORS`: Maps old property names to new ones for backwards compatibility.
- `LoadScreenMode`: Enum for selecting the load screen type.
- Visual customization: `SplashImage`, `ProgressBarTint`, `FeedbackFont`, etc.

#### Blueprint Events
- `EVENT_OnBeginDataSAVE`, `EVENT_OnFinishDataSAVE`
- `EVENT_OnBeginDataLOAD`, `EVENT_OnFinishDataLOAD`

These events allow users to bind custom logic during save/load events.

#### Blueprint Functions

**Command Queue API:**
```cpp
void DB_EnqueueSAVE(FString SQL);
void DB_EnqueueLOAD(FString SQL);
```

**Immediate Execution API:**
```cpp
void DB_OBJ_ImmediateSAVE(UObject*);
void DB_OBJ_ImmediateLOAD(UObject*);
```

**Update Column API:**
Updates single or arrays of:
- Bool, Byte, Enum, Int, Long, Float
- Name, Text, String, Date, Color, Vector2D, Vector3D, Rotator
- UObject* and arrays of UObject*

**Select API:**
Read values from columns or rows, supports all types as above.

**Generate SQL API:**
Generate SQL queries for objects, actors, or components:
```cpp
FString DB_GenerateSQL_Object_INSERT(UObject*);
```

**Versioning API:**
```cpp
bool DB_SetVersion(FString);
bool DB_AddVersion(FString);
bool DB_HasVersion(FString);
TArray<FString> DB_GetVersionList();
```

**Threading API:**
```cpp
void DB_GetThreadSafety(ESQLThreadSafety&, FSQLite_ThreadChanged);
```

---

### Serializable Types

These are base classes designed to be extended to enable automatic SQL serialization.

- `USQLSerializable_OBJ` (Base UObject)
- `USQLSerializable_CMP` (Actor Component)
- `ASQLSerializable` (Actor)

Each includes overrideable events:

```cpp
void DB_PrepareToSave(USQLite* Database, ESQLSaveMode Mode);
void DB_OnFinishLoad(USQLite* Database, bool Success);
```

---

## Load Screen System

Customize user experience during save/load operations.

### Modes:
- `BackgroundBlur`
- `SplashScreen`
- `MoviePlayer`
- `NoLoadScreen`

### Configuration:
- `BackBlurPower`, `SplashImage`, `SplashMovie`
- `ProgressBarOnMovie`, `PauseGameOnLoad`
- `FeedbackSAVE`, `FeedbackLOAD`, `FeedbackFont`

---

## Threaded Operations

Async workers automatically manage background database operations using Unreal’s `FNonAbandonableTask`:

```cpp
DBS_ExecuteQueue_Task
DBL_ExecuteQueue_Task
DBS_ImmediateSave_Task
DBL_ImmediateLoad_OBJ_Task
```

---

## Example Use Cases

- **Save Player Progress**: Store player stats, inventory, position, and level state.
- **Versioned Game Saves**: Maintain backward compatibility across game versions.
- **Persistent Multiplayer World**: Serialize and deserialize the game world without stalling the server.
- **Custom Editor Tools**: Create in-editor asset databases that persist metadata or gameplay tuning parameters.

---

# USQLite Usage Examples

## 🔹 Saving and Loading Data

### ✅ Blueprint Example

#### Save All Data

```plaintext
[Event Begin Play]
    ↓
[Call Function: DB_Save]
    - Context: Self
    - Mode: SaveAll
```

#### Load All Data

```plaintext
[Event Begin Play]
    ↓
[Call Function: DB_Load]
    - Context: Self
```

### ✅ C++ Example

#### Save All Data

```cpp
USQLite* MyDatabase = ...; // Reference to the database asset
MyDatabase->DB_Save(MyDatabase, ESQLSaveMode::SaveAll);
```

#### Load All Data

```cpp
MyDatabase->DB_Load(MyDatabase);
```

## 🔹 Saving an Object Immediately

### ✅ Blueprint

```plaintext
[Call Function: DB_OBJ_ImmediateSAVE]
    - Object: TargetObject
```

### ✅ C++

```cpp
UObject* TargetObject = ...;
MyDatabase->DB_OBJ_ImmediateSAVE(TargetObject);
```

## 🔹 Saving/Loading Actors & Components

### ✅ Blueprint: Save Actor

```plaintext
[Call Function: DB_ACT_ImmediateSAVE]
    - Actor: TargetActor
```

### ✅ C++

```cpp
AActor* Actor = ...;
MyDatabase->DB_ACT_ImmediateSAVE(Actor);
```

## 🔹 Custom Table Versioning

### ✅ Blueprint

```plaintext
[Call Function: DB_SetVersion]
    - Table: "PlayerData_V2"
```

### ✅ C++

```cpp
bool success = MyDatabase->DB_SetVersion("PlayerData_V2");
```

## 🔹 Custom SQL Generation

### ✅ C++

```cpp
FString SQL_Insert = MyDatabase->DB_GenerateSQL_Object_INSERT(MyObject);
FString SQL_Update = MyDatabase->DB_GenerateSQL_Object_UPDATE(MyObject);
FString SQL_Select = MyDatabase->DB_GenerateSQL_Object_SELECT(MyObject);
```

## 🔹 Selecting & Updating Columns

### ✅ Blueprint

```plaintext
[Call Function: DB_SELECT_Integer]
    - RowID: "Player_001"
    - ColumnName: "Health"
```

```plaintext
[Call Function: DB_UPDATE_Integer]
    - RowID: "Player_001"
    - ColumnName: "Health"
    - Value: 75
```

### ✅ C++

```cpp
int32 CurrentHealth = MyDatabase->DB_SELECT_Integer("Player_001", "Health");

ESQLResult Result = MyDatabase->DB_UPDATE_Integer("Player_001", "Health", 75);
```

## 🔹 Handling Save/Load Events

### ✅ Blueprint Events

Bind to:
- `On Begin Data Save`
- `On Finish Data Save`
- `On Begin Data Load`
- `On Finish Data Load`

### ✅ C++ Overrides

```cpp
void AMyActor::DB_OnBeginSave_Implementation(USQLite* Database)
{
    UE_LOG(LogTemp, Log, TEXT("Saving started"));
}

void AMyActor::DB_OnFinishLoad_Implementation(USQLite* Database, bool Success)
{
    if (Success)
    {
        UE_LOG(LogTemp, Log, TEXT("Loading completed successfully"));
    }
}
```

## 🔹 Using Load Screens

### ✅ Blueprint Settings

- Set `LoadScreenMode` to `SplashScreen`, `MoviePlayer`, or `BackgroundBlur`
- Configure `SplashImage`, `BackBlurPower`, `FeedbackSAVE`, etc.

### ✅ Runtime Behavior

When loading data, the system automatically shows the configured UI and tracks progress via:

```plaintext
[Call Function: DB_LaunchLoadScreen]
```

---

## Summary

USQLite is designed to be easy to integrate with both **Blueprints** and **C++**, giving you powerful tools to persist data without dealing with SQL manually. Whether you're saving a simple player inventory or synchronizing thousands of object states across a streamed world, USQLite has you covered.