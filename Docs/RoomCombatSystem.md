---
Status: ACTIVE
Scope: Room Combat
Last Updated: 2026-07-06
Source of Truth: Yes
---

# RoomCombatSystem.md

## 1. Document Purpose

This document defines the current Room combat execution rules and ownership boundaries.

Use this before changing `RoomCombatActor`, `StageFlowManager`, Room enemy spawning, Room Clear, doors, boundaries, placed enemies, or Room-to-Stage handoff.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/RoomCombatSystemHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Direction

A Room is the local combat space inside a Stage.

A Stage contains multiple Rooms. The current Stage test uses ordered Rooms controlled by `StageFlowManager`.

Current long-term rule:

```text
Room entered
-> combat starts
-> enemies spawn or activate
-> all tracked enemies die
-> Room Clear
-> next Room path opens
```

Main build reward cards should appear at Stage Clear, not after every Room Clear.

## 3. Current Room Flow

```text
Player overlaps RoomCombatActor TriggerBox
-> RoomCombatActor starts combat if the Room is enabled and not already cleared
-> local doors/boundaries lock the Room
-> encounter setup builds spawn queues and placed enemy tracking
-> initial enemies spawn or placed enemies activate
-> the first queued wave enemy is scheduled from Room combat start
-> the wave enemy joins after SpawnDelay even if initial enemies are still alive
-> AliveEnemies are tracked through HealthComponent death events
-> all tracked enemies die
-> RoomCombatActor marks Room Cleared
-> local doors/boundaries unlock when progression is ready
-> RoomCombatActor broadcasts OnRoomProgressionReady
-> StageFlowManager activates the next Room or handles Stage Clear fallback
```

## 4. Current Owner Map

| Area | Current Owner | Rule |
|---|---|---|
| Player entry trigger | `RoomCombatActor` | Local Room responsibility. |
| Room lock/unlock | `RoomCombatActor` | Doors and auto boundary blockers are local presentation/collision. |
| Enemy spawn/activation | `RoomCombatActor` | Uses `EncounterData` first, then direct fallback fields. |
| Placed enemy list | `RoomCombatActor.PlacedEnemyActors` | Editor-placed enemies can be dormant until activated. |
| Alive enemy tracking | `RoomCombatActor` | Tracks spawned and placed enemies that count for clear. |
| Room Clear detection | `RoomCombatActor` | All tracked enemies dead. |
| Next-Room activation | `StageFlowManager` | Primary path for manager-owned Stages. |
| Stage Clear fallback | `StageFlowManager` | Temporary reward/restart flow until Stage Map exists. |
| Stage Map and next Stage choice | Future Stage Map system | Not implemented yet. |

## 5. RoomCombatActor Current Responsibilities

`RoomCombatActor` should own only the local Room combat execution.

Current responsibilities:

- Detect player entry through `TriggerBox`.
- Start combat only when enabled and not cleared.
- Lock and unlock local doors or automatic boundary blockers.
- Read `EncounterData` if assigned.
- Fall back to direct room fields when `EncounterData` is not assigned.
- Spawn initial enemies.
- Spawn wave/sequential enemies with delay.
- Activate placed enemies when policy says so.
- Track alive spawned and placed enemies.
- Mark Room cleared when the configured clear condition is satisfied.
- Broadcast `OnRoomProgressionReady` after Room clear/progression wait.

`RoomCombatActor` should not own:

- Stage Map route choice.
- Next Stage candidate selection.
- Full Run state.
- Final reward pool generation.
- Boss Stage or Run Clear logic.
- Enemy AI internals.

## 6. StageFlowManager Current Responsibilities

`StageFlowManager` is the current Stage-level manager for one active Stage MVP.

Current responsibilities:

- Own one active `StageId`.
- Auto-find or use registered Rooms for that Stage.
- Sort Rooms by `RoomOrder`.
- For the current Start Stage stabilization pass, limit the active Stage to 3 ordered Rooms.
- Disable Room Clear reward cards for manager-owned Rooms.
- Listen to `RoomCombatActor.OnRoomProgressionReady`.
- Activate the next Room in order.
- If no next Room exists, mark the Stage cleared.
- Open the existing reward flow at Stage Clear.
- Show the temporary restart fallback after the Stage Clear reward is claimed.

Deferred responsibilities:

- Stage Map UI.
- Returning to Stage Map after Stage Clear.
- Connected next Stage selection.
- Mid Boss Stage routing.
- Boss Stage / Final Boss / Run Clear.
- Final reward pool and rarity calculation.

## 7. Encounter Data Rule

`RoomEncounterDataAsset` is the preferred data path for reusable Room enemy composition.

Current fields:

| Field | Meaning |
|---|---|
| `InitialSpawnEntries` | Enemies spawned together when combat starts. |
| `WaveSpawnEntries` | Enemies spawned sequentially or in later groups. |
| `SpawnDelay` | Delay from Room combat start before the first queued Wave enemy joins; later sequential behavior remains the existing queue behavior. |
| `PlacedEnemyActivationPolicy` | Whether placed enemies activate on Room start or by detection. |
| `PlacedEnemyActivationRadius` | Detection radius for dormant placed enemies. |
| `ClearCondition` | Which tracked enemies count for Room Clear. |

Fallback fields on `RoomCombatActor` remain valid for quick testing:

- `EnemyClass` + `EnemiesToSpawn`
- `InitialSpawnEnemyClasses`
- `SpawnEnemyClasses`
- `SpawnPoints`
- `SpawnDelayBetweenEnemies`
- `PlacedEnemyActors`

For new repeatable Room setups, prefer `RoomEncounterDataAsset`. For quick one-off tests, direct Room fields are acceptable.

## 8. Spawned and Placed Enemy Rules

The Room system supports two enemy entry styles.

| Style | Meaning | Current Path |
|---|---|---|
| Spawned enemy | Enemy is created when Room combat starts or a wave begins. | `RoomCombatActor` spawn queue. |
| Placed enemy | Enemy already exists in the level and wakes when the Room starts or detection policy allows. | `PlacedEnemyActors` + `EnemyBase.ActivateEnemy()`. |

`EnemyBase` supports `Dormant`, `Active`, and `Dead` states.

Dormant enemies should not chase or attack until activated.

## 9. Room Clear / Stage Clear Reward Rule

Current manager-owned Stage rule:

```text
Each Room Clear
-> open next Room only

Final Room Clear
-> Stage Clear
-> open reward card UI
-> selected reward applies
-> show temporary restart fallback
```

Standalone Room rule:

```text
Room Clear
-> optional RewardActor can spawn
-> player touches reward
-> RewardSelectionWidget opens
-> reward applies
```

Standalone Room rewards are an MVP test path only. They are not the long-term main reward cadence.

## 10. Door and Boundary Rule

Current Room lock can use:

- `DoorActors` array for simple visible/collision door actors.
- Auto boundary blockers from `RoomCombatActor` for rectangular combat lock tests.

Auto boundary blockers are good for MVP rectangular Rooms. Complex future Rooms may need a dedicated `CombatLockVolume`, `RoomBoundaryActor`, or authored collision volumes.

## 11. Editor Setup Checklist

For a manager-owned Stage test:

1. Place one `StageFlowManager` in the level.
2. Set `StageFlowManager.StageId`, usually `Stage1` for the current test.
3. Place Room actors using `BP_RoomCombatActor`.
4. Set each Room's `StageId` to match the manager.
5. Set `RoomOrder` in the intended order.
6. Assign `EncounterData` or direct fallback spawn fields.
7. Add `SpawnPoints` or use fallback spawn offsets.
8. Add `PlacedEnemyActors` only for enemies already placed in the level.
9. Confirm only the intended first Room is active at start.
10. Check Output Log if progression fails.

Expected progression logs should show the manager receiving room progression and activating the next Room.

Start Stage `CANDIDATE v0.1` profile:

| Room | Candidate role | Encounter DataAsset |
|---|---|---|
| Room 0 | Basic combat | `DA_Start_Room01_BasicMelee` |
| Room 1 | Fast pressure | `DA_Start_Room02_FastPriority` |
| Room 2 | Tank mix / Stage finish | `DA_Start_Room03_TankMixed` |

Room 2 is the candidate final required Room for this Start Stage stabilization pass. Stage Clear reward should appear only after Room 2 is cleared. Exact entries and delays are in `Docs/StartStageV01.md`; the profile remains a candidate until editor setup and PIE verification are complete.

## 12. Fallback and Deprecated Direction

Current fallback/setup fields:

- `NextRoomActors`
- `bAutoFindNextRoomByOrder`
- standalone Room Clear reward spawning
- direct spawn arrays on `RoomCombatActor`

These are not removed because placed maps and Blueprints may still reference them.

For new manager-owned Stage work, prefer:

- `StageFlowManager` for next-Room activation.
- `RoomEncounterDataAsset` for reusable enemy composition.
- Stage Clear reward timing instead of per-Room build reward cards.

## 13. Not Implemented Yet

- Final Stage Map UI.
- Stage Node data and branching route selection.
- Returning to Stage Map after Stage Clear.
- Final Stage reward data/pool/rarity system.
- Boss Stage and Run Clear flow.
- Complex non-rectangular Room lock volumes.
- Final door animation/state actor.
- Full editor-authored encounter database for every Stage.

## 14. Related Documents

- `Docs/StageRunStructure.md`: Run, Stage, Room hierarchy and Stage Clear flow.
- `Docs/RewardSystem.md`: reward timing and reward source split.
- `Docs/EnemySpawnSystem.md`: spawn and encounter data rules.
- `Docs/EnemySystem.md`: enemy behavior and activation.
- `Docs/10_ProjectClassMap.md`: class responsibility map.
- `Docs/CurrentImplementation.md`: current implementation snapshot.
