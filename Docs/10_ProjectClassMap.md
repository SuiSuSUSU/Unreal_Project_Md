---
Status: ACTIVE
Scope: Class Map
Last Updated: 2026-07-06
Source of Truth: Yes
---

# 10_ProjectClassMap.md

## 1. Document Purpose

This document maps the current gameplay classes, Blueprint owners, and responsibility boundaries.

Use this before moving responsibility between systems or before editing `Source`, `Content`, or `Config`.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/ProjectClassMapHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Runtime Flow

```text
/Game/TopDown/Lvl_TopDown
-> /Game/Blueprints/Core/BP_CombatGameMode
-> /Game/Blueprints/Player/BP_PlayerCharacter
-> /Game/Blueprints/Player/BP_PlayerController
```

Current Stage/Room gameplay is driven by placed `RoomCombatActor` Blueprint instances and one `StageFlowManager` for the active Stage test.

## 3. High-Level Ownership Map

| System | Current Owner | Responsibility |
|---|---|---|
| Player input/movement | `PlayerKeyboardMovementSubsystem` | WASD, dodge, intro, weapon socket transition, debug inputs. |
| Camera zoom | `MouseWheelCameraZoomSubsystem` | Mouse wheel camera distance. |
| Player melee combo | `PlayerMeleeAttackSubsystem` | Left-click combo, hit cone, damage, hit VFX, Chain Lightning MVP. |
| Player stats/runtime modifiers | `PlayerStatsComponent` | Runtime stat values, Run reward modifiers, combat element state. |
| Player stat defaults | `PlayerStatsDataAsset` | Editable baseline player numbers. |
| Player health/death/HUD | `PlayerHealthSubsystem`, `PlayerHealthOrbWidget` | Health component hookup, health orb, death, player hit reaction. |
| Weapon trails/aura | `WeaponTrailComponent` | Combo trail slots, AnimNotify trail playback, Lightning sword aura. |
| Room combat | `RoomCombatActor` | Local room combat lock, spawn/activation, alive tracking, Room Clear signal. |
| Stage room order | `StageFlowManager` | Ordered Rooms, next-Room activation, Stage Clear reward fallback. |
| Encounter data | `RoomEncounterDataAsset` | Per-Room spawn entries, wave entries, placed enemy policy, clear condition. |
| Rewards | `RewardActor`, `RewardSelectionWidget`, `RewardTypes.h` | Temporary reward pickup/card UI/effect bridge. |
| Enemy common behavior | `EnemyBase` | Health, chase, melee attack, activation, separation, spawn intro, death. |
| Enemy simple animation | `EnemySimpleAnimationComponent` | Fallback idle/move animation path. |
| Enemy AnimBP data | `EnemyAnimInstance` | Shared AnimBP variables for dedicated enemy AnimBPs. |
| Global test spawn | `EnemySpawnSubsystem`, `EnemySpawnDataAsset` | Disabled-by-default global test spawn path. |

## 4. Room / Stage Classes

| Class | Status | Current Responsibility | Should Not Own |
|---|---|---|---|
| `ARoomCombatActor` | Implemented | One Room's entry trigger, lock/unlock, enemy spawn or activation, alive enemy tracking, Room Clear, progression-ready signal. | Full Run routing, Stage Map, next Stage selection, reward pool generation. |
| `AStageFlowManager` | Implemented MVP | One active Stage id, ordered Rooms, next-Room activation, final-Room Stage Clear reward/restart fallback. | Enemy composition, enemy AI, attack logic, final Stage Map UI, Run Clear. |
| `URoomEncounterDataAsset` | Implemented first data path | Initial spawn entries, wave entries, spawn delay, placed enemy activation policy, clear condition. | Stage Map route data, reward card database, permanent progression. |
| Future `StageMapWidget` | Not implemented | Visual Stage node map and connected next Stage choices. | Room combat logic. |
| Future `StageNodeDataAsset` | Not implemented | Stage type, risk/reward hint, connected node data, theme. | Per-Room spawn execution. |

`RoomCombatActor.NextRoomActors` and `bAutoFindNextRoomByOrder` remain fallback/setup fields. New manager-owned Stage work should prefer `StageFlowManager`.

## 5. Player / Combat Classes

| Class | Status | Responsibility |
|---|---|---|
| `UPlayerKeyboardMovementSubsystem` | Implemented | WASD, dodge, intro, socket transition, debug slow motion, debug element toggle. |
| `UMouseWheelCameraZoomSubsystem` | Implemented | Camera zoom. |
| `UPlayerMeleeAttackSubsystem` | Implemented | Manual melee combo, pending hit timing, cone hit, confirmed damage, hit blood/distortion VFX, Chain Lightning MVP. |
| `UPlayerStatsComponent` | Implemented | Runtime player stat access, Run reward modifiers, combat element state. |
| `UPlayerStatsDataAsset` | Implemented | Editable baseline player stat values. |
| `UHealthComponent` | Implemented | Shared health, damage, healing, death event, invulnerability. |
| `UPlayerHealthSubsystem` | Implemented | Player health setup, health orb, death flow, Run reward reset, player hit reaction. |
| `UPlayerHealthOrbWidget` | Implemented | Runtime circular health orb HUD. |
| `UWeaponTrailComponent` | Implemented | Combo trail slots, trail activation, reward visual hook, Lightning aura. |

`PlayerMeleeAttackSubsystem` still has a historical name, but it is the current manual left-click melee attack owner.

## 6. Reward Classes

| Class / Struct | Status | Responsibility |
|---|---|---|
| `ARewardActor` | Implemented | Temporary reward carrier and card UI entry point. |
| `URewardSelectionWidget` | Implemented | Temporary card UI and reward application path. |
| `FRewardCardOption` | Implemented temporary bridge | Runtime/UI reward card shape. |
| `ERewardApplyType` | Implemented | Instant / Run / Permanent category split. |
| `ERewardCardEffect` | Implemented | Temporary hardcoded reward effect id. |
| Future `RewardDataAsset` | Not implemented | Final reward definitions, grades, weights, requirements. |
| Future reward pool/resolver | Not implemented | Card candidate selection, Luck influence, Stage filtering. |

Current `Permanent` rewards are structural placeholders only. No save/load, currency, or lobby upgrade UI exists yet.

## 7. Enemy Classes

| Class / Asset | Status | Responsibility |
|---|---|---|
| `AEnemyBase` | Implemented | Common enemy movement, chase, melee attack, activation state, separation, spawn intro, death. |
| `UEnemyStatsDataAsset` | Implemented | Enemy health, speed, attack, range, separation values. |
| `UEnemySimpleAnimationComponent` | Implemented | Simple fallback idle/move playback. |
| `UEnemyAnimInstance` | Implemented | AnimBP variables such as movement/death state. |
| `UEnemySpawnSubsystem` | Implemented test path | Disabled global test spawn. Not main room spawn. |
| `UEnemySpawnDataAsset` | Implemented test path | Global test spawn data. |
| `ASurvivorSquareEnemy` | Legacy/test | Old simple enemy test actor. |
| `BP_Enemy_LittleDemon` | Implemented MVP | Little Demon enemy. |
| `BP_Enemy_Tank` | Implemented MVP | Butcher/Tank enemy using dedicated AnimBP/Montage path. |
| `BP_Enemy_Fast` | Implemented MVP | DemonHeavy/Fast enemy using dedicated AnimBP/Montage path. |

Do not hardcode monster mesh, skeleton, or animation assets in `EnemyBase` for new enemy types. Configure those through Blueprint/assets unless a code-level abstraction is explicitly needed.

## 8. Animation / VFX Helper Classes

| Class | Status | Responsibility |
|---|---|---|
| `UAnimNotifyState_WeaponTrail` | Implemented | Opens/closes weapon trail windows on attack animations. |
| `UAnimNotify_PlayerMeleeHit` | Implemented | Executes queued melee hit timing from animation. |
| `UAnimNotify_GroundImpactVfx` | Implemented | One-frame weapon-ground spark. |
| `UAnimNotifyState_GroundContactImpactVfx` | Implemented | Contact window for weapon-ground impact. |
| `UAnimNotifyState_GroundScrapeVfx` | Implemented | Repeated scrape sparks while weapon drags across floor. |
| Future `WeaponImpactVfxComponent` | Not implemented | Contact-point VFX after confirmed hit. |
| Future `HitReactComponent` | Not implemented | Target-side body reaction/stagger decision. |

Ground spark notifies should use `WeaponVfx_GroundContact` on `BP_PlayerCharacter`, not static-mesh socket auto-editing.

## 9. Data / Asset Ownership

| Data | Status | Owner Rule |
|---|---|---|
| `PlayerStatsDataAsset` | Implemented | Baseline player numbers only. Do not store VFX assets here. |
| `EnemyStatsDataAsset` | Implemented | Enemy stat values. |
| `EnemySpawnDataAsset` | Implemented test path | Global test spawn only. |
| `RoomEncounterDataAsset` | Implemented | Per-Room enemy composition and placed enemy policy. |
| Future `RewardDataAsset` | Not implemented | Reward definitions and card metadata. |
| Future `StageNodeDataAsset` | Not implemented | Stage Map node and route metadata. |

## 10. Deprecated / Fallback / Legacy Notes

- `EnemySpawnSubsystem` is not the main spawn system for current gameplay. Treat it as global test spawn.
- `RoomCombatActor.NextRoomActors` and `bAutoFindNextRoomByOrder` are fallback/setup fields for old or standalone flows.
- Standalone Room Clear rewards remain only for rooms not owned by `StageFlowManager`.
- The shared enemy white hit-flash is removed and should not be restored as the default hit feedback path.
- Old dated implementation notes are archived and should not override this active class map.

## 11. Related Documents

- `Docs/CurrentImplementation.md`: current implementation snapshot.
- `Docs/RoomCombatSystem.md`: Room combat and StageFlowManager handoff.
- `Docs/StageRunStructure.md`: Run / Stage / Room terminology.
- `Docs/RewardSystem.md`: reward timing and ownership.
- `Docs/VFXSystem.md`: VFX ownership.
- `Docs/AnimationSystem.md`: animation ownership.
- `Docs/EnemySystem.md`: enemy behavior direction.
