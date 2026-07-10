---
Status: REFERENCE
Scope: Enemy Spawn
Last Updated: 2026-07-02
Source of Truth: No
---

# EnemySpawnSystem.md

## 2026-07-02 Update: Encounter Document Boundary

- This document owns enemy spawn and encounter data rules.
- `RoomEncounterDataAsset` is the current data structure for Room-level initial spawns, wave spawns, spawn delay, placed enemy activation policy, and clear condition.
- `RoomCombatActor` executes the runtime Room encounter.
- `StageFlowManager` does not own enemy composition; it only sequences Rooms inside the active Stage.
- Reward timing and reward content should be checked in `RewardSystem.md`, not placed inside enemy spawn rules by default.


## 2026-07-02 Update: RoomEncounterDataAsset First Implementation

- The main room spawn path is now `RoomCombatActor` plus optional `RoomEncounterDataAsset`.
- `InitialSpawnEntries` is for enemies that appear together at Room combat start.
- `WaveSpawnEntries` is for delayed follow-up enemies that use the existing sequential queue behavior.
- `SpawnDelay` is the DataAsset-owned delay between wave entries.
- `PlacedEnemyActivationPolicy` controls enemies already placed in the level.
- `ClearCondition` prepares room clear checks for all tracked enemies, spawned enemies only, or placed enemies only.
- Direct `RoomCombatActor` fields remain useful for quick one-off test rooms and as fallback data.
- `EnemySpawnSubsystem` remains a global test spawn path and is not the main Stage/Room encounter system.


## 2026-06-24 Update: Room Encounter Data Preparation

- `RoomCombatActor` direct settings remain the Stage 1 setup for room-specific spawns.
- The current direct settings are enough for hand-placed MVP rooms where the designer wants immediate control in the level.
- `InitialSpawnEnemyClasses` is for simultaneous first-wave enemies.
- `SpawnEnemyClasses` is for delayed sequential enemies.
- `SpawnPoints` is for explicit room placement when each enemy should appear at a chosen location.
- `EnemyClass + EnemiesToSpawn` stays as a fallback for simple old tests.

Current `RoomEncounterDataAsset` fields:

- `InitialSpawnEntries`
- `WaveSpawnEntries`
- `SpawnDelay`
- `PlacedEnemyActivationPolicy`
- `PlacedEnemyDetectionRadius`
- `ClearCondition`

Later field candidates should be copied from stable `RoomCombatActor` needs, not invented separately:

- `EncounterId`
- `RoomType`
- Spawn point policy or spawn group names
- Fallback enemy class/count only if direct RoomActor fallback becomes painful
- Special-room reward hooks only if a specific Room type truly needs them
- Debug notes

`EnemySpawnSubsystem` remains a global test spawn path only and should not become the main room encounter path.

## 2026-06-23 Update: Initial Wave vs Sequential Queue

- `RoomCombatActor.InitialSpawnEnemyClasses` is used when a room should spawn multiple enemies together at combat start.
- `RoomCombatActor.SpawnEnemyClasses` remains the ordered sequential queue for enemies that should appear one at a time after previous tracked enemies die.
- The current MVP rule is:
  - Use `InitialSpawnEnemyClasses` for the first simultaneous wave.
  - Use `SpawnEnemyClasses` for delayed follow-up enemies.
  - Use `EnemyClass` plus `EnemiesToSpawn` only as the fallback when both class arrays are empty.
- Room1 currently tests this by spawning `BP_Enemy_Fast` and `BP_Enemy_Tank` together as the first wave.

## 2026-06-22 Update: Room Spawn Enemy Class Array

- `RoomCombatActor.SpawnEnemyClasses` is the current Stage 1 way to define a room-specific ordered enemy list.
- If the array has entries, the room spawns one listed `EnemyBase` subclass at a time in array order.
- The next array entry spawns after the current spawned enemy dies and `SpawnDelayBetweenEnemies` has elapsed.
- The current temporary delay value is `0.5` seconds.
- If the array is empty, `RoomCombatActor` falls back to the existing single `EnemyClass` and `EnemiesToSpawn` count.
- This direct per-room setup remains fallback/editor override data now that `RoomEncounterDataAsset` exists.
- `EnemySpawnSubsystem` remains global test spawn only and should not be used as the main room encounter path.

## 2026-06-22 Update: Room Encounter Setting Direction

- Direct settings on each placed `RoomCombatActor` remain valid for quick one-off test rooms.
- Reusable room enemy composition should move toward `RoomEncounterDataAsset`.
- `RoomEncounterDataAsset` now owns the first structured encounter data path: enemy entries, counts through entry expansion, wave timing, placed enemy activation policy, and clear rules.
- Elite flags, boss flags, room tags, and special room rules remain later data candidates.
- Main build rewards should stay Stage Clear oriented unless a special Room explicitly needs its own reward hook.
- `EnemySpawnSubsystem` remains a global test spawn or temporary validation system, not the main room encounter structure.

## 2026-06-11 ?�데?�트: ?�테?��? ?�리?�형 ?�투 방향

- ?�재 게임 방향?� 무한???�이 계속 ?�오??구조가 ?�니???��? �?구역???�나???�리?�하???�테?��? ?�파???�다???�션 MVP??
- ?��? ?�레?�어 주�???계속 ?�성?�는 방식보다, �??�장 ??`RoomSpawnPoint` 기�??�로 ?�해�??�량 ?�는 ?�이브로 ?�성?�는 방향???�선?�다.
- �??�장, �??�힘, ???�폰, ???�멸, �??�리?? 보상 ?�성, ?�음 �??�동???�식 ?�름 ?�보?�다.
- `EnemySpawnSubsystem`?� ??��?��? ?�고 ?�역 ?�스???�폰/?�시 검증용 구조�??�치�???��??
- ?�식 구조??`RoomCombatActor` ?�는 Stage/WaveSystem 기반 �??�위 ?�폰?�로 ?�환?�다.
- 기존 `EnemySpawnDataAsset`?� ?�역 ?�스???�폰 ?�정?�로 ?��??�되, ?��? �??�폰 ?�이?�로 ?�사?�할 ???�다.

## 2026-05-30 ?�데?�트: ?�전 ?�스???�포??비활?�화

- `SurvivorSquareSpawner`???�전 ?�스?�용 ?�폰 Actor?�며 ?�재 공식 ?�폰 ?�스?�이 ?�니??
- `SurvivorSquareSpawner`??맵에 ?�아 ?�어?????�상 ?�을 ?�성?��? ?�도�?비활?�화?�었??
- ?�재 ?�역 ?�스???�폰?� `EnemySpawnSubsystem`�?`EnemySpawnDataAsset` 기�??�로 ?�인?�다.
- ?�이 ?�도?� ?�르�?즉시 ?�성?�면 맵에 ?��? 구형 ?�포?? `EnemySpawnSubsystem`, �??�투 ?�폰 로직???�서?��??�인?�다.

## 1. 문서 목적

??문서?????�폰 규칙???�디??관리하?��? ?�리?�다.

?�으�?Codex가 ???�폰, �??�투, ?�이�? ?�테?��? 진행??구현??????개체 ?�탯�??�폰 규칙???�동?��? ?�도�?기�????�공?�다.

## 2. ?�재 ?�정??방향

- ??개체 ?�력치�? ???�폰 규칙?� 분리?�서 관리한??
- ??개체 ?�력치는 `EnemyStatsDataAsset`?�로 관리한??
- ?�재 구현???�역 ?�스???�폰?� `EnemySpawnSubsystem`???�당?�다.
- `EnemySpawnSubsystem`?� ?�으�??�식 �??�투 구조??중심???�니???�스??검증용?�로 본다.
- ?�역 ?�스???�폰?� 기본 비활?�화 ?�태?? `MyProject.EnemySpawn.GlobalTestSpawnEnabled=1` 콘솔 변?��? `EnemySpawnDataAsset.bSpawnEnabled=true`가 모두 ?�요?�다.
- ?�식 MVP ?�폰?� �??�장 ??`RoomCombatActor`가 `RoomSpawnPoint`�?기�??�로 ?�을 ?�성?�는 방향???�선?�다.
- 초기 MVP??고정??�?2~3개�? 같�? 맵에 배치?�고, �?방에 ?�동 배치??SpawnPoint�??�용?�다.

## 3. ?�재 구현??Data Asset ?�??
| 구분 | ?�일 | ??�� |
|---|---|---|
| ?�폰 ?�정 Data Asset | `Source/MyProject/Enemies/EnemySpawnDataAsset.h` | ?�역 ?�스???�폰 ?�는 ?��? �??�폰 규칙???�디?�에??조절?�는 ?�플�?|
| ?�폰 ?�정 구현 | `Source/MyProject/Enemies/EnemySpawnDataAsset.cpp` | Data Asset C++ 구현 ?�일 |
| ???�력�?Data Asset | `Source/MyProject/Enemies/EnemyStatsDataAsset.h` | ??개체 ?�력치�? ?�디?�에??조절?�는 ?�플�?|
| ???�력�?구현 | `Source/MyProject/Enemies/EnemyStatsDataAsset.cpp` | Data Asset C++ 구현 ?�일 |
| ?�역 ?�스???�폰 ?�스??| `Source/MyProject/Enemies/EnemySpawnSubsystem.*` | 게임 �?Data Asset 값을 ?�고 ?�레?�어 주�????�을 ?�성?�는 ?�시 검증용 ?�스??|

## 4. ?�재 ?�역 ?�스???�폰 Data Asset 경로

?�재 `EnemySpawnSubsystem`?� ?�래 ?�셋??기본 ?�폰 ?�정?�로 찾는??

`/Game/Data/Enemy/Spawn/DA_EnemySpawn_Default`

???�셋???�으�?C++ 기본값을 ?�용?�다.

| ??�� | 기본�???�� |
|---|---|
| EnemyClass | `SpawnEnemyEntries` ?�성 ??��???�을 ???�용?�는 fallback ?�래??|
| SpawnEnemyEntries | ?�간/가중치 기반?�로 ?�폰????목록 |
| bSpawnEnabled | ?�역 ?�스???�폰 ?�성???��?. 콘솔 변??`MyProject.EnemySpawn.GlobalTestSpawnEnabled=1`???�께 ?�요 |
| MaxAliveEnemies | ?�시???��???최�? ????|
| SpawnCountPerBatch | ??�??�폰?????�성??????|
| InitialSpawnDelay | 게임 ?�작 ??�??�폰까�? ?��??�간 |
| SpawnInterval | ?�음 ?�폰까�? ?��??�간 |
| SpawnDistance | ?�레?�어 기�? ?�폰 거리 |

주의:

- ??구조???�재 구현???�스???�폰 구조??
- �??�리?�형 MVP?�서??방마???�른 SpawnPoint?� ??구성???�야 ?��?�?`RoomCombatActor` 중심 구조�??�환?�다.

## 5. �??�위 ?�폰 구조 ?�보

| 개념 | ??�� | 초기 MVP ?�단 |
|---|---|---|
| `RoomCombatActor` | �??�장, �??�힘, ???�폰, ?�아?�는 ??추적, �??�리??처리�??�당 | ?�선 구현 ?�보 |
| `RoomSpawnPoint` | �??�에???�이 ?�성???�치 | ?�선 구현 ?�보 |
| `RoomEncounterDataAsset` | 방별 ??목록, ?�이�? ?�수 �?보상 메�??�이?��? 관리하??Data Asset | 초기?�는 ?�보, ?�요 ??분리 |
| `DoorActor` / `RoomDoorActor` | ?�투 �?�??�힘, ?�리????�??�림 처리 | ?�선 구현 ?�보 |
| `RewardActor` | �??�리????보상 ?�성/?�득 처리 | 1�?구현 ?�보 |
| Stage/WaveSystem | ?�러 �??�름, ?�리?�방/보스�? ?�테?��? ?�리??관�?| 추후 ?�장 ?�보 |

초기 구현?� `RoomCombatActor`가 `EnemyClass` 배열�?`RoomSpawnPoint` 배열??직접 ?�고 ?�어???�다. 구조가 ?�정????`RoomEncounterDataAsset`?�로 분리?�다.

## 6. �??�리?�형 ?�폰 ?�름 ?�보

1. ?�레?�어가 �?Trigger???�어간다.
2. `RoomCombatActor`가 ?�투 ?�작 ?�태가 ?�다.
3. �?문이 ?�힌??
4. ?�록??`RoomSpawnPoint` ?�치???�해�??�이 ?�성?�다.
5. `RoomCombatActor`가 ?�아?�는 ??목록??추적?�다.
6. 모든 ?�이 ?�망?�면 `RoomCleared` ?�태가 ?�다.
7. 문이 ?�리거나 보상???�성?�다.
8. ?�레?�어가 보상???�득?�고 ?�음 방으�??�동?�다.

## 7. ?�디???�더 구조

?�재 Enemy Data Asset 관리�? ?�해 ?�래 ?�더�??�용?�다.

| ?�더 | ?�도 |
|---|---|
| `/Game/Data/Enemy/Spawn` | ?�역 ?�스???�폰, ?��? �??�폰 ?�보 ?�정 |
| `/Game/Data/Enemy/Stats` | ??개체 ?�력�?Data Asset |
| `/Game/Data/Enemy/Waves` | 추후 �??�이�?구성 Data Asset ?�보 |

추후 �??�투???�더 ?�보:

| ?�더 | ?�도 |
|---|---|
| `/Game/Data/Rooms` | `RoomEncounterDataAsset` ?�보 |
| `/Game/Blueprints/Rooms` | `RoomCombatActor`, `RoomDoorActor`, `RewardActor` ?�보 |

## 8. EnemySpawnDataAsset ??��

| ??�� | ?�명 |
|---|---|
| EnemyClass | ?�성?�된 `SpawnEnemyEntries`가 ?�을 ???�용??fallback ??Blueprint Class |
| SpawnEnemyEntries | ?�간/가중치 기반 ???�폰 목록 |
| bSpawnEnabled | ?�역 ?�스???�폰 ?�성???��? |
| MaxAliveEnemies | ?�시???��???최�? ????|
| SpawnCountPerBatch | ??�??�폰?????�성??????|
| InitialSpawnDelay | 게임 ?�작 ??�??�폰까�? ?��??�간 |
| SpawnInterval | ?�음 ?�폰까�? ?��??�간 |
| SpawnDistance | ?�레?�어 기�? ?�폰 거리 |

## 9. ?�디?�에???�역 ?�스???�폰???�용?�는 방식

1. 콘솔?�서 `MyProject.EnemySpawn.GlobalTestSpawnEnabled 1`???�력?�다.
2. Content Browser?�서 `/Game/Data/Enemy/Spawn` ?�더�??�동?�다.
3. `DA_EnemySpawn_Default`�??�다.
4. `bSpawnEnabled`�?true�?켠다.
5. `SpawnEnemyEntries` 배열???�스?�로 ?�폰????Blueprint�?추�??�다.
6. �?Entry??`EnemyClass`, `SpawnWeight`, `StartTime`, `EndTime`???�정?�다.
7. ?�폰 간격, 최�? ???? ?�폰 거리 ?�을 조절?�다.

주의:

- ??방식?� �??�리?�형 ?�식 ?�폰???�니???�투/???�??검증용?�다.
- �??�위 ?�투 구현 ?�에??`RoomCombatActor` ?�는 `RoomEncounterDataAsset` 기�??�로 ?�을 배치?�다.

## 10. Codex 구현 주의?�항

- ???�폰 관??기능???�정?�기 ?�에????문서�?먼�? ?�인?�다.
- ?�의 체력, ?�도, 공격??같�? 개체 ?�력치는 ?�폰 Data Asset???��? ?�는??
- `EnemySpawnSubsystem`????��?��? ?�는?? ?�만 ?�식 �??�투 구조??중심?�로 보�? ?�는??
- `EnemySpawnSubsystem`???�역 ?�동 ?�폰?� 기본 OFF�??��??�다.
- �??�리?�형 MVP?�서??`RoomCombatActor`, `RoomSpawnPoint`, `RoomDoorActor`, `RewardActor`�??��? ?�위�?구현?�다.
- ?�정 ?�의 Mesh?� Animation Blueprint??`EnemyBase` C++가 ?�니????Blueprint?�서 관리한??
- Data Asset???�더?�도 ?�레???�스?��? 가?�하?�록 fallback 기본값�? ?��??�다.
