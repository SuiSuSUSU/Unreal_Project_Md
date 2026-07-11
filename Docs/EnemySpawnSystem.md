---
Status: ACTIVE
Scope: Enemy Spawn
Last Updated: 2026-07-06
Source of Truth: Yes
---

# EnemySpawnSystem.md

## 1. Document Purpose

This document defines the current enemy spawn and encounter rules.

Use this document before changing Room enemy composition, spawn queues, placed enemy activation, `RoomEncounterDataAsset`, or the old global test spawn path.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/EnemySpawnSystemHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Direction

The current main spawn direction is Room/Stage based.

```text
Player enters enabled Room
-> RoomCombatActor starts combat
-> Room spawns enemies or activates placed enemies
-> RoomCombatActor tracks alive enemies
-> tracked enemies die
-> Room Clear
-> StageFlowManager opens the next Room or handles Stage Clear
```

Current rules:

- `RoomCombatActor` executes the runtime Room encounter.
- `RoomEncounterDataAsset` is the preferred reusable data source for Room enemy composition.
- Direct fields on `RoomCombatActor` remain valid for quick one-off tests and fallback setup.
- `EnemySpawnSubsystem` remains a disabled-by-default global test spawn path.
- Infinite global spawning is not the current main gameplay loop.
- Stage/Room progression belongs to `StageFlowManager` and `RoomCombatActor`, not to `EnemySpawnSubsystem`.

## 3. Current Implementation Files

| Area | File | Current Role |
|---|---|---|
| Room encounter execution | `Source/MyProject/Rooms/RoomCombatActor.h/.cpp` | Starts Room combat, spawns/activates enemies, tracks alive enemies, marks Room Clear. |
| Encounter data | `Source/MyProject/Rooms/RoomEncounterDataAsset.h` | Defines reusable Room enemy entries, wave entries, spawn delay, placed enemy policy, and clear condition. |
| Stage order | `Source/MyProject/Rooms/StageFlowManager.h/.cpp` | Orders Rooms by `StageId` / `RoomOrder`, activates the next Room, handles Stage Clear fallback. |
| Enemy behavior | `Source/MyProject/Enemies/EnemyBase.h/.cpp` | Owns one enemy's health, activation state, chase, attack, separation, spawn intro, and death. |
| Enemy stats | `Source/MyProject/Enemies/EnemyStatsDataAsset.h/.cpp` | Defines one enemy type's gameplay stats. |
| Global test spawn | `Source/MyProject/Enemies/EnemySpawnSubsystem.h/.cpp` | Disabled global test spawn loop around the player. |
| Global test spawn data | `Source/MyProject/Enemies/EnemySpawnDataAsset.h/.cpp` | Tuning data for the global test spawn loop. |

## 4. Ownership Boundaries

| System | Owns | Does Not Own |
|---|---|---|
| `EnemyBase` | One enemy's runtime behavior. | Room composition, Stage order, reward timing. |
| `EnemyStatsDataAsset` | One enemy type's stats. | Spawn count, Room waves, visual assets. |
| `RoomEncounterDataAsset` | One Room's reusable enemy composition data. | Runtime combat execution, Stage Map route data. |
| `RoomCombatActor` | One Room's combat execution and enemy tracking. | Full Run routing, next Stage choice, final reward database. |
| `StageFlowManager` | Current Stage Room order and Stage Clear fallback. | Enemy AI, enemy stats, per-Room composition. |
| `EnemySpawnSubsystem` | Global test spawn verification only. | Current Stage/Room encounter flow. |

Do not put enemy health, movement, attack damage, or animation assets in spawn data. Those belong to enemy stats, enemy Blueprint setup, or animation/VFX documents.

## 5. RoomCombatActor Spawn Flow

Current Room spawn flow:

```text
StartCombat()
-> BuildRuntimeEncounterQueues()
-> Prepare/Register placed enemies
-> Spawn initial enemies
-> Apply placed enemy activation policy
-> Spawn wave/sequential enemies as needed
-> Track spawned and placed enemies
-> Check clear condition
-> Broadcast OnRoomProgressionReady
```

Important details:

- If `EncounterData` is assigned, Room spawn entries are read from the data asset first.
- If `EncounterData` is not assigned, direct `RoomCombatActor` fields are used.
- Initial enemies are intended to appear together at Room start.
- The first queued Wave enemy is scheduled when Room combat starts and can join after `SpawnDelay` while Initial enemies are still alive.
- Pending or alive Wave enemies continue to block Room Clear.
- Spawned enemies and placed enemies are both registered for clear checks when the clear condition requires it.
- `RoomCombatActor` reports progression-ready; `StageFlowManager` decides the next manager-owned Room.

## 6. RoomEncounterDataAsset

`RoomEncounterDataAsset` is the preferred path for reusable Room enemy composition.

Current fields:

| Field | Meaning |
|---|---|
| `InitialSpawnEntries` | Enemy classes and counts that spawn together when the Room starts. |
| `WaveSpawnEntries` | Enemy classes and counts that spawn as delayed follow-up entries. |
| `SpawnDelay` | Delay between wave/sequential spawn entries. |
| `PlacedEnemyActivationPolicy` | How already-placed enemies become active. |
| `PlacedEnemyDetectionRadius` | Detection radius for placed enemies that activate on player detection. |
| `ClearCondition` | Which tracked enemies must be defeated for Room Clear. |

Current enums:

| Enum | Values |
|---|---|
| `EPlacedEnemyActivationPolicy` | `ActivateOnRoomStart`, `ActivateOnPlayerDetection` |
| `ERoomClearCondition` | `AllTrackedEnemiesDefeated`, `SpawnedEnemiesDefeated`, `PlacedEnemiesDefeated` |

Use `RoomEncounterDataAsset` when:

- multiple Rooms reuse similar enemy compositions;
- a Stage needs stable authored encounter data;
- placed enemies and spawned enemies need a clear policy;
- direct per-actor setup is becoming hard to maintain.

## 7. RoomCombatActor Fallback Fields

Direct `RoomCombatActor` fields remain useful for quick tests.

| Field | Meaning |
|---|---|
| `EnemyClass` | Fallback enemy class if arrays and encounter data are empty. |
| `EnemiesToSpawn` | Fallback count used with `EnemyClass`. |
| `InitialSpawnEnemyClasses` | Enemy classes spawned together at Room start. |
| `SpawnEnemyClasses` | Ordered sequential/wave enemy class queue. |
| `SpawnPoints` | Optional actor locations used as spawn positions. |
| `SpawnDelayBetweenEnemies` | Fallback delay between sequential spawns. |
| `PlacedEnemyActors` | Already-placed enemies controlled by this Room. |

Guideline:

- For throwaway tests, direct fields are fine.
- For real Stage rooms that will survive longer, prefer `EncounterData`.
- Do not remove these fields abruptly because existing placed Rooms and Blueprints may reference them.

Current Start Stage stabilization exception:

- The old `StageFlowManager` runtime 3-Room encounter override remains as a fallback but is disabled by default.
- Start Stage `CANDIDATE v0.1` uses three authored `RoomEncounterDataAsset` assets instead of the runtime override.
- Exact candidate entries and delays are documented in `Docs/StartStageV01.md`.
- Do not treat this profile as implemented until the DataAssets and Room instances are verified in PIE.

## 8. Spawned Enemies

Spawned enemies are created by `RoomCombatActor` when Room combat starts or when a wave entry becomes active.

Current spawned enemy rules:

- Spawned enemy classes must derive from `EnemyBase`.
- Spawn transform comes from `SpawnPoints` when possible.
- If there are fewer spawn points than enemies, `RoomCombatActor` falls back to temporary positions around the Room actor.
- Spawned enemies are registered for death tracking.
- Spawned enemies can count toward Room Clear depending on `ClearCondition`.
- Spawn collision handling may adjust placement, but spawn point quality still matters for readable combat.

## 9. Placed Enemies

Placed enemies are already in the level before Room combat starts.

Current placed enemy rules:

- Place an `EnemyBase`-derived enemy in the level.
- Add it to `RoomCombatActor.PlacedEnemyActors`.
- The Room prepares it as Dormant before combat when appropriate.
- On Room start, it either activates immediately or waits for player detection.
- Placed enemies are tracked separately from spawned enemies.
- Placed enemies can count toward Room Clear depending on `ClearCondition`.

Activation policy:

| Policy | Behavior |
|---|---|
| `ActivateOnRoomStart` | Placed enemies become active when Room combat starts. |
| `ActivateOnPlayerDetection` | Placed enemies stay Dormant until the player enters detection radius. |

Placed enemies are useful for rooms where enemies should be visible before combat, wake up from the environment, or be positioned very precisely by hand.

## 10. Clear Conditions

Room Clear depends on `ERoomClearCondition`.

| Condition | Meaning |
|---|---|
| `AllTrackedEnemiesDefeated` | Spawned and placed tracked enemies must all be dead. |
| `SpawnedEnemiesDefeated` | Only tracked spawned enemies are required. |
| `PlacedEnemiesDefeated` | Only tracked placed enemies are required. |

Default guideline:

- Use `AllTrackedEnemiesDefeated` for normal combat Rooms.
- Use `SpawnedEnemiesDefeated` when placed enemies are decorative, optional, or not part of the clear goal.
- Use `PlacedEnemiesDefeated` for ambush or pre-authored enemy rooms where no spawned waves are required.

## 11. EnemySpawnSubsystem

`EnemySpawnSubsystem` is not the current main gameplay spawn system.

Current purpose:

- Global test spawn around the player.
- Quick enemy class verification.
- Temporary debugging when a Room setup is not needed.

Activation requirements:

- Console variable `MyProject.EnemySpawn.GlobalTestSpawnEnabled` must be `1`.
- `/Game/Data/Enemy/Spawn/DA_EnemySpawn_Default` must exist and have `bSpawnEnabled = true`.

Current `EnemySpawnDataAsset` fields:

| Field | Meaning |
|---|---|
| `EnemyClass` | Fallback enemy class. |
| `SpawnEnemyEntries` | Weighted enemy class entries with optional time range. |
| `bSpawnEnabled` | Enables the data asset side of global test spawning. |
| `MaxAliveEnemies` | Maximum active test enemies. |
| `SpawnCountPerBatch` | Enemies spawned per batch. |
| `InitialSpawnDelay` | Delay before first test spawn. |
| `SpawnInterval` | Delay between test spawn batches. |
| `SpawnDistance` | Spawn distance around the player. |

Do not use `EnemySpawnSubsystem` as the default Room, Stage, or Run spawn system.

## 12. StageFlowManager Relationship

`StageFlowManager` does not own enemy composition.

It owns:

- one active `StageId`;
- ordered Rooms in that Stage;
- next-Room activation;
- Stage Clear fallback until Stage Map exists.

It does not own:

- which enemies appear in a Room;
- enemy stats;
- enemy activation behavior;
- wave timing;
- spawn point placement.

When a Room is manager-owned, `RoomCombatActor` still handles spawn and clear tracking, then emits `OnRoomProgressionReady`.

## 13. Editor Setup Checklist

For a normal manager-owned Room:

1. Place `BP_RoomCombatActor` in the level.
2. Set `StageId` to match the current `StageFlowManager`.
3. Set `RoomOrder`.
4. Assign `EncounterData` if the Room should use reusable encounter data.
5. If no `EncounterData` is assigned, fill direct fallback fields.
6. Add `SpawnPoints` for readable enemy placement.
7. Add `PlacedEnemyActors` only for enemies already placed in the level.
8. Confirm the first Room is enabled and later Rooms are gated by `StageFlowManager`.
9. Play and check Output Log for Room clear and StageFlowManager progression logs.

For quick isolated spawn tests:

1. Use direct `RoomCombatActor` fields, or
2. temporarily enable `EnemySpawnSubsystem` using the console variable and data asset.

The second option is not the current main gameplay route.

## 14. Folder Guidelines

Current or intended data locations:

| Folder | Purpose |
|---|---|
| `/Game/Data/Enemy/Stats` | Enemy stat data assets. |
| `/Game/Data/Enemy/Spawn` | Global test spawn data assets. |
| `/Game/Data/Rooms` | Room encounter data assets. |
| `/Game/Blueprints/Rooms` | Room actors, door actors, reward actors, and room helpers. |
| `/Game/Blueprints/Enemies` | Enemy Blueprint classes. |

If a folder is missing, treat it as an intended organization guideline, not proof that a system is unimplemented.

## 15. Not Implemented Yet

The following are not final systems yet:

- Full authored encounter database for every Stage.
- Stage Map route-based encounter selection.
- `StageNodeDataAsset`.
- Final reward pool, rarity, Luck, and Stage-type reward weighting.
- Elite/boss encounter tables.
- Procedural Room generation.
- Spawn VFX policy per enemy type.
- Large-wave crowd/performance manager.
- Room-specific music, camera, or environment pacing rules.

## 16. Current Do / Do Not

Do:

- Use `RoomEncounterDataAsset` for reusable Room enemy composition.
- Keep `RoomCombatActor` as the runtime executor for one Room.
- Keep `StageFlowManager` focused on Room order and Stage Clear.
- Keep enemy stats in `EnemyStatsDataAsset`.
- Keep visual and animation quality in Blueprint/assets and animation/VFX documents.

Do not:

- Treat `EnemySpawnSubsystem` as the main spawn loop.
- Put Room wave counts into `EnemyStatsDataAsset`.
- Put enemy animation assets into encounter data.
- Move Stage Map or reward pool responsibility into `RoomCombatActor`.
- Reintroduce vampire-survivor-style infinite spawning as the default progression path.
- Use Archive documents as current source of truth.

## 17. Related Documents

- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- `Docs/RoomCombatSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/EnemySystem.md`
- `Docs/EnemyArchitecture.md`
- `Docs/RewardSystem.md`
- `Docs/VFXSystem.md`
- `Docs/AnimationSystem.md`
