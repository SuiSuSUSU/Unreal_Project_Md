---
Status: ACTIVE
Scope: Current Implementation
Last Updated: 2026-07-06
Source of Truth: Yes
---

# CurrentImplementation.md

## 1. Document Purpose

This document is the current implementation snapshot for the project.

Use this document to check what is actually implemented before changing code, Blueprint setup, data assets, or design documents.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/CurrentImplementationHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Project Entry Points

| Area | Current Path / Owner | Notes |
|---|---|---|
| Main level | `/Game/TopDown/Lvl_TopDown` | Current playable test map. |
| GameMode | `/Game/Blueprints/Core/BP_CombatGameMode` | Current game flow entry. |
| Player character | `/Game/Blueprints/Player/BP_PlayerCharacter` | Current playable pawn. |
| Player controller | `/Game/Blueprints/Player/BP_PlayerController` | Current player input controller. |
| Player stats data | `/Game/Data/Player/DA_PlayerStats_Default` | Runtime systems read this first, then use code defaults as fallback. |

Older TopDown template assets and sample classes can still exist in the project, but they should not be treated as the current main gameplay path unless the map or Blueprint references prove it.

## 3. Current Playable Loop

The current playable loop is a single-map Stage/Room MVP.

```text
Player enters an enabled Room
-> RoomCombatActor starts combat
-> Room locks local boundary / doors
-> Room spawns or activates enemies
-> Player defeats all tracked enemies
-> RoomCombatActor marks Room cleared
-> StageFlowManager receives progression-ready
-> StageFlowManager enables the next Room
-> Final Room clears the current Stage
-> Stage Clear reward appears
-> Reward is selected
-> Temporary restart fallback appears
```

Long-term Run and Stage Map flow is documented in `Docs/StageRunStructure.md`. The current project does not have the final Stage Map UI or next-Stage selection yet.

## 4. Player Implementation

| Feature | Current Owner | Status |
|---|---|---|
| WASD movement | `PlayerKeyboardMovementSubsystem` | Implemented. |
| Mouse wheel camera zoom | `MouseWheelCameraZoomSubsystem` | Implemented. |
| Intro weapon draw | `PlayerKeyboardMovementSubsystem` | Implemented. |
| Weapon socket swap | `PlayerKeyboardMovementSubsystem` | Implemented. |
| Space dodge | `PlayerKeyboardMovementSubsystem`, `PlayerStatsComponent` | Implemented with runtime stats. |
| Left-click 1/2/3 combo | `PlayerMeleeAttackSubsystem` | Implemented as manual melee combo. |
| Attack damage timing | `PlayerMeleeAttackSubsystem`, `AnimNotify_PlayerMeleeHit` | Pending-hit path exists; notify timing is preferred, fallback delay remains. |
| Attack cone damage | `PlayerMeleeAttackSubsystem` | Implemented. |
| Hit blood VFX | `PlayerMeleeAttackSubsystem` | Implemented on confirmed hit. |
| Hit distortion / Lightning hit VFX | `PlayerMeleeAttackSubsystem` | Neutral uses hit distortion; Lightning state uses `NS_Hit_Lightning_once` on confirmed hit. |
| Sword trail | `WeaponTrailComponent`, `AnimNotifyState_WeaponTrail` | Implemented as component/notify path; editor tuning still matters. |
| Lightning debug state | `PlayerStatsComponent`, `PlayerKeyboardMovementSubsystem` | L toggles `Neutral` / `Lightning`; log-only debug. |
| Chain Lightning MVP | `PlayerMeleeAttackSubsystem` | Implemented: Lightning state, confirmed hit, one nearby extra target, and temporary chain VFX. |
| Health orb HUD | `PlayerHealthSubsystem`, `PlayerHealthOrbWidget` | Implemented. |
| Player death | `PlayerHealthSubsystem` | Implemented. |
| Player hit reaction | `PlayerHealthSubsystem`, enemy attack request | Light/Heavy MVP implemented. |
| Slow motion debug | `PlayerKeyboardMovementSubsystem` | F10 toggles debug slow motion. |

Current player combat is still built around the existing katana combo. Final skill trees, full elemental reward cards, and final player VFX growth are not implemented.

## 5. Room / Stage Implementation

| Feature | Current Owner | Status |
|---|---|---|
| Room entry trigger | `RoomCombatActor` | Implemented. |
| Room lock / unlock | `RoomCombatActor` | Implemented through local doors and auto boundary blockers. |
| Room enemy spawn | `RoomCombatActor` | Implemented through fallback fields or `RoomEncounterDataAsset`. |
| Placed enemy activation | `RoomCombatActor`, `EnemyBase` | Implemented through placed enemy references and enemy activation state. |
| Alive enemy tracking | `RoomCombatActor` | Implemented. |
| Room Clear detection | `RoomCombatActor` | Implemented when tracked enemies are dead. |
| Progression-ready signal | `RoomCombatActor.OnRoomProgressionReady` | Implemented. |
| Ordered Room progression | `StageFlowManager` | Implemented for one active Stage id. |
| Stage Clear reward timing | `StageFlowManager` | Implemented as current MVP fallback. |
| Temporary restart after Stage Clear | `StageFlowManager` | Implemented until Stage Map exists. |

Manager-owned Rooms should not grant build reward cards on every Room Clear. `StageFlowManager` disables Room Clear rewards for registered Rooms and opens the existing reward flow at Stage Clear.

Standalone Room Clear rewards remain only as an MVP test path for rooms not owned by `StageFlowManager`.

## 6. Encounter / Spawn Implementation

| Feature | Current Owner | Status |
|---|---|---|
| Per-Room encounter data | `RoomEncounterDataAsset` | Implemented as first data asset path. |
| Initial spawn entries | `RoomEncounterDataAsset.InitialSpawnEntries` | Implemented. |
| Wave spawn entries | `RoomEncounterDataAsset.WaveSpawnEntries` | Implemented. |
| Spawn delay | `RoomEncounterDataAsset.SpawnDelay` or `RoomCombatActor` fallback | Implemented. |
| Spawn points | `RoomCombatActor.SpawnPoints` | Implemented. |
| Placed enemies | `RoomCombatActor.PlacedEnemyActors` | Implemented. |
| Clear condition enum | `RoomEncounterDataAsset.ClearCondition` | Prepared / first implementation. |
| Global test spawn | `EnemySpawnSubsystem` | Exists but should remain disabled by default. |

`EnemySpawnSubsystem` is not the main game spawn loop. It is a global test/temporary validation path.

## 7. Reward Implementation

| Feature | Current Owner | Status |
|---|---|---|
| Reward pickup actor | `RewardActor` | Implemented. |
| Reward card UI | `RewardSelectionWidget` | Implemented as temporary UI. |
| Card count | `RewardActor` | Supports 1 to 4 cards; current MVP starts with 2. |
| Reward data struct | `FRewardCardOption` in `RewardTypes.h` | Implemented temporary bridge. |
| Apply type enum | `ERewardApplyType` | Instant / Run / Permanent implemented as type split. |
| Runtime Run stat modifiers | `PlayerStatsComponent` | Implemented for temporary Run rewards. |
| Run reward reset | `PlayerHealthSubsystem`, `PlayerStatsComponent` | Implemented on death/restart path. |
| Permanent upgrade persistence | None | Not implemented. |

Long-term reward definitions, rarity, weighted pools, Luck, enemy drop items, and permanent upgrade save/load are not implemented.

## 8. Enemy Implementation

| Feature | Current Owner | Status |
|---|---|---|
| Common enemy base | `EnemyBase` | Implemented. |
| Health / death | `HealthComponent`, `EnemyBase` | Implemented. |
| Chase and melee attack | `EnemyBase` | Implemented. |
| Enemy separation | `EnemyBase`, `EnemyStatsDataAsset` | Implemented. |
| Dormant / Active / Dead state | `EnemyBase` | Implemented. |
| Spawn intro | `EnemyBase` | Implemented for supported enemies. |
| Weak / strong attack slots | `EnemyBase` | Implemented. |
| Little Demon | `BP_Enemy_LittleDemon` | Implemented MVP enemy. |
| Tank / Butcher | `BP_Enemy_Tank`, `ABP_Butcher` | Implemented MVP enemy. |
| Fast / DemonHeavy | `BP_Enemy_Fast`, dedicated AnimBP path | Implemented MVP enemy. |
| Universal enemy hit react | Future system | Not implemented. |

The removed shared white hit-flash should not be reintroduced as the enemy feedback path.

## 9. VFX / Animation Implementation

| Area | Current Owner | Status |
|---|---|---|
| Weapon trail timing | `AnimNotifyState_WeaponTrail` | Implemented. |
| Confirmed hit timing | `AnimNotify_PlayerMeleeHit` | Implemented. |
| Ground impact spark | `AnimNotify_GroundImpactVfx` | Implemented. |
| Ground contact spark window | `AnimNotifyState_GroundContactImpactVfx` | Implemented. |
| Ground scrape sparks | `AnimNotifyState_GroundScrapeVfx` | Implemented. |
| Blood and hit distortion | `PlayerMeleeAttackSubsystem` | Implemented on confirmed hit. |
| Lightning hit burst | `PlayerMeleeAttackSubsystem` | Implemented on confirmed direct hit while Lightning state is active. |
| Lightning sword aura / trail | `WeaponTrailComponent` | Implemented with `NS_Sword_Lightning` as the weapon aura and `NS_SlashTrail_Lightning` as the Lightning-state attack trail override. Movement afterimage/trail VFX is intentionally removed for now; final Niagara color wiring still requires editor-side check. |
| Chain Lightning VFX | `PlayerMeleeAttackSubsystem` | Implemented as temporary Source enemy -> Target enemy arc using `NS_EletricLightning`. |
| WeaponImpactVfxComponent | Future component | Not implemented. |
| Enemy HitReactComponent | Future component | Not implemented. |

Exact visual quality still requires Unreal Editor testing. Do not assume Niagara color, scale, location, rotation, or socket alignment is final from code alone.

## 10. Not Implemented Yet

- Stage Map UI.
- Stage Node data assets.
- Returning to Stage Map after Stage Clear.
- Next Stage candidate selection UI.
- Mid Boss Stage implementation.
- Boss Stage / Final Boss / Run Clear.
- Final reward database, rarity, weighted pool, Luck roll, and Prism roll logic.
- Enemy drop item system.
- Permanent upgrade save/load and lobby upgrade UI.
- Full skill tree, weapon tree, item synergy, or full elemental status framework.
- Final tuned Lightning VFX chain arcs.
- Final hit reaction system for enemies.
- Final camera feedback system.

## 11. Source of Truth Routing

- `Docs/StageRunStructure.md`: Run, Stage Map, Stage, Room hierarchy.
- `Docs/RoomCombatSystem.md`: Room combat, StageFlowManager handoff, encounter execution.
- `Docs/RewardSystem.md`: reward timing, reward source split, apply types.
- `Docs/ElementalRewardSystem.md`: Lightning reward planning and Chain Lightning MVP.
- `Docs/VFXSystem.md`: weapon trail, hit VFX, ground sparks, Lightning VFX ownership.
- `Docs/AnimationSystem.md`: animation timing, notifies, player/enemy animation rules.
- `Docs/10_ProjectClassMap.md`: class and ownership map.

## 12. Editing Rules

- Do not treat archived dated logs as current implementation rules.
- Do not use `CANDIDATE`, `DEPRECATED`, or `Archive` documents as current implementation truth unless the user explicitly asks.
- Before changing code, confirm the current Blueprint/map connection if the change touches `Config`, `Content`, or `Source`.
