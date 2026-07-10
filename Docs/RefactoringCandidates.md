---
Status: REFERENCE
Scope: Non-Visual Refactoring Candidates
Last Updated: 2026-07-02
Source of Truth: No
---

# RefactoringCandidates.md

## 1. Document Purpose

This document is a non-visual responsibility audit.

It records which gameplay classes are starting to own too many responsibilities, which splits are likely useful later, and which refactors should not be done yet.

This document is not the current implementation source of truth. Use `Docs/CurrentImplementation.md`, `Docs/10_ProjectClassMap.md`, and the domain documents first.

## 2. Inputs Reviewed

- `Docs/00_ReadFirst.md`
- `Docs/05_DecisionLog.md`
- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- `Docs/04_CodexWorkflow.md`
- `Docs/RoomCombatSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/RewardSystem.md`
- `Docs/EnemySystem.md`
- `Docs/EnemyArchitecture.md`
- `Docs/BasicAttackSystem.md`
- `Docs/VFXSystem.md`
- `Source/MyProject/Rooms/RoomCombatActor.h`
- `Source/MyProject/Rooms/StageFlowManager.h`
- `Source/MyProject/Rooms/RoomEncounterDataAsset.h`
- `Source/MyProject/Enemies/EnemyBase.h`
- `Source/MyProject/Combat/PlayerMeleeAttackSubsystem.h`
- `Source/MyProject/UI/RewardSelectionWidget.h`
- `Source/MyProject/Combat/WeaponTrailComponent.h`

## 3. Current Refactoring Rule

Do not refactor only because a class is large.

Prefer this order:

1. Keep the current gameplay loop stable.
2. Extract data first when repeated editor setup becomes painful.
3. Add clean handoff signals before moving ownership.
4. Move one responsibility at a time.
5. Remove old fallback fields only after editor maps and Blueprints are verified.

The current project is still in a fast MVP-to-structure transition. Refactors should happen when a new feature touches the same area, not as isolated cleanup with no gameplay benefit.

## 4. Responsibility Size Snapshot

| Area | File | Current signal |
| --- | --- | --- |
| Room combat | `RoomCombatActor.cpp` | Large room-local executor with trigger, spawn, tracking, clear, reward fallback, boundary, door, and progression handoff logic. |
| Stage flow | `StageFlowManager.cpp` | Medium manager now owns current Stage room order, next-room activation, Stage Clear reward timing, and temporary restart fallback. |
| Enemy runtime | `EnemyBase.cpp` | Large enemy base with movement, activation, attack, animation presentation, spawn intro, death, stats, and reaction hooks. |
| Player melee | `PlayerMeleeAttackSubsystem.cpp` | Medium-large subsystem with combo state, hit detection, animation, movement lock, forward step, and trail request logic. |
| Reward UI | `RewardSelectionWidget.cpp` | UI currently also applies reward effects. This is fine for MVP but should not grow much more. |
| Weapon trail | `WeaponTrailComponent.cpp` | Currently a focused presentation component. Keep it narrow. |

## 5. Priority Candidates

### P1 - Reward Effect Application

Current owner:

- `RewardSelectionWidget`

Current responsibilities:

- Display reward card choices.
- Handle selection.
- Apply instant rewards.
- Apply run rewards.
- Log permanent reward TODO behavior.

Why it is a candidate:

- The UI is starting to own gameplay effect logic.
- Future reward grades, Luck, technique levels, permanent upgrades, and reward pools would make this class grow quickly.

Recommended split:

- Keep `RewardSelectionWidget` as display and selection UI.
- Move reward application to a future `RewardEffectApplier`, `RewardRuntimeEffectSystem`, or similar small gameplay owner.
- Keep `RewardDataAsset`, weighted pools, rarity tables, currency, and save/load out of the first split.

When to do it:

- Before adding real reward grades, Luck-based reward rolling, or permanent upgrades.

Do not do yet:

- Do not build a full reward database only to support the current two-card MVP.

### P1 - RoomCombatActor Responsibility Boundary

Current owner:

- `RoomCombatActor`

Current responsibilities:

- Detect player room entry.
- Lock and unlock room boundary.
- Control optional room doors.
- Start room combat.
- Spawn enemies from direct fields or `RoomEncounterDataAsset`.
- Register placed enemies.
- Track alive enemies.
- Decide Room Clear.
- Emit progression-ready signal.
- Keep standalone reward/progression fallback paths.
- Keep editor debug and preview helpers.

Why it is a candidate:

- The class is still the core local Room executor, but it also contains old MVP setup fields and new EncounterData paths at the same time.
- This is acceptable during migration, but it should not become the Stage, reward, and encounter data owner again.

Recommended split:

- Keep `RoomCombatActor` as the local room runtime executor.
- Keep reusable enemy composition in `RoomEncounterDataAsset`.
- Keep Stage sequencing in `StageFlowManager`.
- Later consider small helpers only if the class keeps growing:
  - `RoomEnemyTracker`
  - `RoomBoundaryController`
  - `RoomDoorController`
  - `RoomEncounterExecutor`

When to do it:

- After `StageFlowManager` and `RoomEncounterDataAsset` are stable in editor.
- When more room types, placed enemies, elite rooms, or special clear rules are added.

Do not do yet:

- Do not remove direct Room fields until existing map instances and Blueprints are checked.
- Do not make `RoomEncounterDataAsset` mandatory for every test room yet.

### P2 - EnemyBase Split Candidates

Current owner:

- `EnemyBase`

Current responsibilities:

- Health component setup and death handling.
- Dormant/active state.
- Chase movement.
- Separation.
- Attack timing and damage.
- Attack animation presentation.
- Strong attack selection for Tank/Butcher style enemies.
- Spawn intro and follow-up idle/turn behavior.
- Stats data application.
- Future hit reaction decision hooks.

Why it is a candidate:

- Enemy behavior, combat rules, and presentation rules are concentrated in one base class.
- This is useful for fast iteration, but risky once more enemy types, bosses, and hit reactions are added.

Recommended split:

- Keep the base class stable for now.
- Add future splits only around real repeated needs:
  - `EnemyMovementComponent` or `EnemyChaseComponent`
  - `EnemyAttackComponent`
  - `EnemyPresentationComponent`
  - `HitReactComponent`

When to do it:

- When the second or third enemy type needs different movement or reaction logic that cannot be handled cleanly with data.
- When HitReact becomes visually validated and more than one enemy needs custom reaction policy.

Do not do yet:

- Do not extract everything at once.
- Do not make `HitReactComponent` before the first damage-based reaction rule is proven in play.

### P2 - PlayerMeleeAttackSubsystem Boundary

Current owner:

- `PlayerMeleeAttackSubsystem`

Current responsibilities:

- Read attack input.
- Manage combo step and combo timing.
- Resolve cone hit detection.
- Apply melee damage.
- Trigger attack animation.
- Lock movement during attack.
- Allow dodge cancel.
- Apply forward step movement.
- Request weapon trail playback.
- Provide debug display.

Why it is a candidate:

- Player attack gameplay, presentation, and movement side effects are in the same subsystem.
- This will become more important when skills, impact VFX, hit reactions, elemental effects, and reward-modified attacks are added.

Recommended split:

- Keep it as the current attack coordinator.
- Later move confirmed-hit handling into a smaller resolver:
  - `PlayerMeleeHitResolver`
  - `PlayerAttackEffectRouter`
  - `WeaponImpactVfxComponent`
- Keep `WeaponTrailComponent` as trail playback only.

When to do it:

- When adding real hit impact VFX, lightning chain attacks, bleed, or damage-based HitReact.

Do not do yet:

- Do not split combo timing before the current animation and AnimNotify trail path is stable.

### P2 - StageFlowManager Scope Guard

Current owner:

- `StageFlowManager`

Current responsibilities:

- Find and sort rooms in one active Stage.
- Receive room progression-ready signals.
- Activate next room by `StageId + RoomOrder`.
- Detect final room.
- Open Stage Clear reward using the existing reward actor/widget path.
- Show temporary restart/menu fallback until Stage Map exists.

Why it is a candidate:

- It is now the correct owner for current Stage progression.
- It can become too large if Run, Stage Map, Stage choice UI, boss logic, reward generation, and save state are all added here.

Recommended split:

- Keep `StageFlowManager` focused on one active Stage.
- Later add separate owners:
  - `RunFlowManager` for whole-run state.
  - `StageMapController` for node map selection.
  - `StageRewardResolver` for Stage Clear reward candidates.

When to do it:

- When the real Stage Map is implemented.
- When the player can choose between multiple next Stage nodes.

Do not do yet:

- Do not turn `StageFlowManager` into the full Run manager now.

### P3 - WeaponTrailComponent Scope Guard

Current owner:

- `WeaponTrailComponent`

Current responsibilities:

- Store combo trail slots.
- Attach Niagara trails to the weapon.
- Support fallback playback.
- Support `AnimNotifyState_WeaponTrail` timing.
- Receive reward-related visual scaling hooks.

Why it is a candidate:

- This component is currently well scoped.
- It should stay limited to trail playback.

Recommended split:

- Keep sword trail playback here.
- Put contact sparks, blood, ground hit sparks, lightning contact, or slash impact bursts in a future `WeaponImpactVfxComponent`.

Do not do:

- Do not put damage, reward selection, enemy reaction, or hit validation into `WeaponTrailComponent`.

## 6. Recommended Refactoring Order

1. Reward effect application split.
2. Player confirmed-hit routing split.
3. Enemy HitReact hook or component after one visual rule is tested.
4. RoomCombatActor helper splits only after EncounterData adoption grows.
5. StageFlowManager split only when Stage Map work begins.

This order keeps the most likely upcoming feature pressure in mind:

- Stage Clear rewards will expand soon.
- Hit impact VFX and damage-based reactions are likely next combat feedback work.
- Room encounter data already exists, so RoomCombatActor can wait until more room data is actually used.
- Stage Map is planned but not implemented yet.

## 7. Do Not Refactor Yet

- Do not remove `RoomCombatActor` standalone fallback behavior yet.
- Do not remove direct Room enemy fields until map instances are audited.
- Do not build `RewardDataAsset` until reward grade, pool, and Luck rules are clearer.
- Do not create a full Run manager before Stage Map exists.
- Do not create a full HitReact animation system before one enemy reaction is verified.
- Do not merge trail VFX and impact VFX into the same component.

## 8. Next Smallest Implementation Candidates

If implementation resumes later, the safest small candidates are:

1. Add a minimal reward effect applier and keep `RewardSelectionWidget` as UI-only.
2. Add a confirmed-hit event/request path from player melee into future impact VFX and HitReact.
3. Add a default no-op enemy HitReact hook, then enable one simple reaction rule on one enemy type.
4. Move more Room enemy setup into `RoomEncounterDataAsset` only after the current editor test stage is stable.

Each candidate should be implemented and editor-tested separately.

