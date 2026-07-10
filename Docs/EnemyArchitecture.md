---
Status: ACTIVE
Scope: Enemy Architecture
Last Updated: 2026-07-06
Source of Truth: Yes
---

# EnemyArchitecture.md

## 1. Document Purpose

This document defines the current enemy architecture boundaries.

Use this before changing `EnemyBase`, enemy Blueprint structure, enemy stats, enemy animation ownership, enemy hit reaction direction, or enemy-related responsibility splits.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/EnemyArchitectureHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Direction

Enemy architecture should separate common gameplay logic from enemy-specific presentation.

Current direction:

- `EnemyBase` owns common enemy runtime behavior.
- Enemy Blueprint assets own mesh, skeleton, AnimBP, Montage, and visual setup.
- `EnemyStatsDataAsset` owns gameplay numbers for one enemy type.
- `RoomEncounterDataAsset` owns Room enemy composition.
- `RoomCombatActor` executes one Room encounter.
- `StageFlowManager` sequences Rooms inside the current Stage.

Do not make `EnemyBase` a place for every monster-specific asset, VFX, animation, reward, or Stage rule.

## 3. Current Architecture Map

```text
EnemyBase
-> BP_Enemy_LittleDemon
-> BP_Enemy_Fast
-> BP_Enemy_Tank
-> future BP_Enemy_Ranged / Elite / Boss variants

EnemyStatsDataAsset
-> enemy health, speed, attack, range, separation values

RoomEncounterDataAsset
-> Room enemy entries, waves, placed enemy activation policy, clear condition

RoomCombatActor
-> spawn / activate / track / clear one Room

StageFlowManager
-> order Rooms and handle current Stage Clear fallback
```

## 4. Current Implemented Enemy Types

| Enemy | Current Structure | Notes |
|---|---|---|
| `BP_Enemy_LittleDemon` | `EnemyBase` + Little Demon assets | Stable MVP enemy. Uses the SingleNode / simple animation path for now. |
| `BP_Enemy_Fast` | `EnemyBase` + DemonHeavy assets + dedicated AnimBP/Montage path | Fast pressure enemy. Uses weak/strong attack setup. |
| `BP_Enemy_Tank` | `EnemyBase` + Butcher assets + dedicated AnimBP/Montage path | Slower durable enemy. Uses weak/strong attack setup. |

New enemy types should first be tested as `EnemyBase` Blueprint variants with different stats and assets. Add new C++ enemy subclasses only when shared `EnemyBase` settings and Blueprint configuration are no longer enough.

## 5. EnemyBase Responsibility

`EnemyBase` owns common gameplay behavior for one enemy actor.

Current responsibilities:

- health component hookup;
- enemy stat data application;
- player target search;
- chase movement;
- stop distance and personal space handling;
- soft enemy separation;
- Dormant / Active / Dead activation state;
- spawn intro lock and follow-up;
- melee attack distance and cooldown logic;
- weak/strong attack playback request;
- attack-facing and post-attack turn support;
- death handling and death event response.

`EnemyBase` should not own:

- Room enemy composition;
- Stage or Run progression;
- reward timing or reward generation;
- specific monster mesh paths;
- specific monster skeleton or AnimBP paths;
- all hit reaction presentation;
- weapon impact VFX;
- boss phase logic.

## 6. Blueprint and Asset Responsibility

Enemy-specific Blueprint or assets own presentation and per-character setup.

Blueprint/assets should own:

- Skeletal Mesh;
- Skeleton;
- Animation Blueprint;
- Montage and animation sequence assignment;
- mesh relative transform fixes;
- character-specific VFX and SFX hooks;
- death presentation assets;
- enemy-specific reaction assets;
- `EnemyStatsDataAsset` assignment.

Rules:

- Do not hardcode Little Demon, DemonHeavy, Butcher, or future monster asset paths in `EnemyBase`.
- Keep monster-specific orientation, scale, and skeleton decisions in Blueprint or asset setup.
- Use the original monster skeleton when possible.
- Use IK Rig / IK Retargeter only when sharing animation across skeletons is actually needed.

## 7. EnemyStatsDataAsset Boundary

`EnemyStatsDataAsset` owns gameplay-facing enemy numbers.

Current stat responsibilities:

- enemy id and display name;
- max health;
- move speed;
- stop distance;
- personal space padding;
- separation radius and strength;
- max separation neighbors;
- attack damage;
- attack range;
- attack cooldown.

Do not put these into `EnemyStatsDataAsset`:

- spawn counts;
- wave timing;
- Room clear conditions;
- reward data;
- Niagara assets;
- animation assets;
- mesh or skeleton references.

If future HitReact becomes gameplay-facing, resistance values such as `HitReactResistance` can be considered here. Animation or VFX assets for that reaction should still stay outside stat data.

## 8. Spawn and Encounter Boundary

Enemy architecture does not own Room enemy composition.

Current split:

| Area | Owner |
|---|---|
| One enemy's behavior | `EnemyBase` |
| One enemy type's stats | `EnemyStatsDataAsset` |
| One Room's enemy composition | `RoomEncounterDataAsset` |
| Runtime spawn/activation/tracking | `RoomCombatActor` |
| Room order and Stage Clear fallback | `StageFlowManager` |
| Old global test spawn | `EnemySpawnSubsystem` |

`EnemySpawnSubsystem` is a disabled global test path. It is not the current Room/Stage spawn architecture.

## 9. Animation Boundary

Common animation playback requests can start in `EnemyBase`, but animation quality belongs to enemy-specific AnimBP, Montage, BlendSpace, and asset tuning.

Current rules:

- Little Demon remains on its stable simple animation path until a dedicated AnimBP path is visually verified.
- Fast and Tank use dedicated AnimBP/Montage paths.
- Random attack animation swapping is not a default rule because it previously caused visible transition issues.
- If multiple attacks are needed, prefer verified Montage sections or weak/strong Montage slots over random sequence playback.
- Attack damage timing is still primarily distance/cooldown logic, not enemy attack notify timing.

For detailed animation and retargeting rules, use:

- `Docs/AnimationSystem.md`
- `Docs/EnemyAnimationRetargeting.md`

## 10. HitReact Boundary

Enemy HitReact is a future structure candidate, not a completed shared system.

Current rules:

- The old generic white hit-flash is removed and should not be restored as the default feedback path.
- `EnemyBase` should not force every hit to play the same flinch, flash, stagger, or knockback.
- Future reaction decisions should compare attack strength with enemy resistance.
- Boss and mini-boss enemies may have special immunity or stagger windows.

Recommended future decision model:

```text
HitReactScore = DamageAmount + AttackTypeBonus + ComboStepBonus + RewardBonus

if HitReactScore >= EnemyHitReactResistance:
    request allowed reaction
else:
    no reaction
```

Recommended ownership:

| Area | Preferred Owner |
|---|---|
| Hit detection and damage application | Attacker combat system / `HealthComponent` |
| Emitting a reaction request | `EnemyBase` or a receiver bridge |
| Deciding whether reaction happens | Enemy Blueprint or future `HitReactComponent` |
| Playing flinch/stagger animation | Enemy AnimBP, Montage, or Blueprint |
| Contact VFX | Future `WeaponImpactVfxComponent` or enemy-specific VFX owner |
| Boss immunity windows | Boss-specific Blueprint or AI state |

## 11. Collision and Separation Rule

Enemies should avoid visually stacking into one body, but they should not physically trap or shove the player as a core mechanic.

Current rules:

- `EnemyBase` handles soft enemy-to-enemy separation.
- Separation is tuned through `EnemyStatsDataAsset`.
- Separation is movement behavior, not a spawn rule.
- Character/Blueprint collision setup should avoid enemies physically pinning the player.
- Larger waves may later need crowd avoidance, spatial partitioning, or a dedicated movement/avoidance component.

## 12. Not Implemented Yet

- Shared `HitReactComponent`.
- Final enemy stagger/poise system.
- Enemy attack damage notifies.
- Final death presentation for every enemy family.
- Enemy-specific spawn/death/hit VFX policy.
- Ranged, elite, mini-boss, and boss enemy architecture.
- Large-wave crowd/performance manager.
- Full authored encounter database for every Stage.

## 13. Modification Rules

- Keep common enemy logic in `EnemyBase`.
- Keep enemy stats in `EnemyStatsDataAsset`.
- Keep Room composition in `RoomEncounterDataAsset`.
- Keep enemy assets in Blueprint or asset setup.
- Do not add monster-specific hardcoded paths to C++ shared classes.
- Do not mix hit reaction, impact VFX, damage application, and death handling into one shared method.
- Do not remove fallback paths abruptly while placed Blueprints may still reference them.

## 14. Related Documents

- `Docs/EnemySystem.md`: enemy gameplay behavior and current enemy types.
- `Docs/EnemySpawnSystem.md`: spawn, placed enemies, and encounter data rules.
- `Docs/RoomCombatSystem.md`: Room combat execution.
- `Docs/AnimationSystem.md`: animation ownership, notifies, hit reaction direction.
- `Docs/EnemyAnimationRetargeting.md`: enemy skeleton and retargeting rules.
- `Docs/VFXSystem.md`: enemy-related VFX and hit presentation boundaries.
- `Docs/HealthSystem.md`: health, damage, death, and hit reaction boundaries.
- `Docs/10_ProjectClassMap.md`: current class responsibility map.
