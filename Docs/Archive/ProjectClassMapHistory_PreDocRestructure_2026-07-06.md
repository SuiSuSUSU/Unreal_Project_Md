---
Status: REFERENCE
Scope: Class Map History
Last Updated: 2026-07-06
Source of Truth: No
---

# 10_ProjectClassMap.md

## 2026-07-02 Update: Documentation Responsibility Map

- `StageRunStructure.md`: Run, Stage Map, Stage, Room, Encounter, Wave terminology and flow.
- `RoomCombatSystem.md`: RoomCombatActor, RoomEncounterDataAsset, Room Clear, local room execution.
- `RewardSystem.md`: Stage Clear rewards, reward apply types, reward grade direction, future reward data ownership.
- `ElementalRewardSystem.md`: elemental reward branches, Lightning reward candidates, Prism unlock direction.
- `VFXSystem.md`: WeaponTrail, Niagara, reward-driven visual growth, hit/spawn/death VFX direction.
- `AnimationSystem.md`: AnimBP, Montage, BlendSpace, AnimNotify, cross-cutting animation ownership.
- `EnemyAnimationRetargeting.md`: enemy skeleton, retargeting, and enemy-specific animation asset notes.


## 2026-07-02 Update: WeaponImpactVfxComponent Design Boundary

- `WeaponImpactVfxComponent` is a planned player weapon hit-impact presentation component, not an implemented runtime class yet.
- It should own impact VFX requests after a weapon attack confirms a hit, such as slash contact bursts, sparks, blood contact effects, lightning contact effects, or ground-contact effects.
- It should not replace `WeaponTrailComponent`. `WeaponTrailComponent` owns moving sword trails; `WeaponImpactVfxComponent` should own contact effects at hit or ground-impact points.
- `PlayerMeleeAttackSubsystem` should remain the owner of current cone hit detection and damage application. It may later send a lightweight impact request after a confirmed hit.
- `WeaponImpactVfxComponent` should not apply damage, calculate attack ranges, mutate player/enemy stats, own enemy hit-reaction animations, or reintroduce the removed generic white hit flash.
- Ground sparks should eventually use animation notifies or explicit ground-trace requests, not fixed delay guesses.
- Reward-driven impact changes should later come from reward/VFX profile data, not from `PlayerStatsDataAsset`.

## 2026-07-02 Update: GroundImpactVfx AnimNotify

- `AnimNotify_GroundImpactVfx` exists at `Source/MyProject/Combat/AnimNotify_GroundImpactVfx.h/.cpp`.
- It is an animation-timed presentation notify for weapon-ground contact sparks.
- It traces downward from a Blueprint trace-point component, then spawns `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` by default.
- Default trace-point setup is `TracePointComponentName=WeaponVfx_GroundContact`.
- Old notify instances that still request `BladeTipTracePoint` use `WeaponVfx_GroundContact` first when it exists, then fall back to `BladeTipTracePoint`.
- If the trace point is missing, it can create a runtime-only component under `EquippedOtachi` using the configured relative transform.
- Deprecated socket fallback remains only if `TracePointComponentName` is set to None.
- It is presentation-only and does not apply damage, hit reaction, Lightning, shockwave, camera shake, or gameplay effects.
- Final timing, socket, scale, rotation, and lifetime should be tuned per animation in the editor.

## 2026-07-04 Update: GroundContactImpactVfx NotifyState

- `AnimNotifyState_GroundContactImpactVfx` exists at `Source/MyProject/Combat/AnimNotifyState_GroundContactImpactVfx.h/.cpp`.
- It is the preferred ground-impact spark path when a one-frame notify is too hard to place exactly.
- It opens a short animation window, traces downward from `WeaponVfx_GroundContact` every tick, and spawns one spark only when the trace point is close enough to the floor hit.
- `MaxContactDistance` controls how close the trace point must be before the impact spark is allowed to spawn.
- It is presentation-only and does not apply damage, hit reaction, Lightning, shockwave, camera shake, or gameplay effects.
- Existing `AnimNotify_GroundImpactVfx` remains available for simple one-frame contact bursts and old animation setups.

## 2026-07-02 Update: GroundScrapeVfx AnimNotifyState

- `AnimNotifyState_GroundScrapeVfx` exists at `Source/MyProject/Combat/AnimNotifyState_GroundScrapeVfx.h/.cpp`.
- It is an animation-timed presentation notify state for weapon-ground scrape sparks.
- It repeatedly traces downward from a Blueprint trace-point component while the notify-state window is active, then spawns `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` by default.
- Default trace-point setup is `TracePointComponentName=WeaponVfx_GroundContact`.
- Old notify-state instances that still request `BladeTipTracePoint` use `WeaponVfx_GroundContact` first when it exists, then fall back to `BladeTipTracePoint`.
- If the trace point is missing, it can create a runtime-only component under `EquippedOtachi` using the configured relative transform.
- Deprecated socket fallback remains only if `TracePointComponentName` is set to None.
- The older owner-local start/end offset fallback is deprecated because it did not follow the blade accurately.
- It is presentation-only and does not apply damage, hit reaction, Lightning, shockwave, camera shake, or gameplay effects.
- Final timing, socket, interval, scale, rotation, and lifetime should be tuned per animation in the editor.


## 2026-07-02 Update: HitReact Design Boundary

- `HitReact` / `HitReactComponent` is a planned target-side hit response structure, not an implemented runtime class yet.
- It should decide whether a damaged enemy visually reacts, briefly flinches, staggers, ignores the hit, or uses a boss-specific response by comparing hit strength against target resistance.
- `EnemyBase` may later emit a lightweight hit-react request after confirmed damage, but it should not force all enemies into one common reaction.
- Enemy Blueprint, AnimBP, or Montage assets should own character-specific reaction presentation.
- `WeaponImpactVfxComponent` and `HitReact` are separate: impact VFX is contact-point presentation, while HitReact is target body response.
- Boss and MiniBoss reactions should default to `Immune` or special windows, not generic flinch.
- `HitReact` should not apply damage, calculate attack hits, decide Room Clear, own death handling, or restore the removed white hit-flash behavior.
- Preferred future inputs: `DamageAmount`, `HitReactPower`, attack type, combo step, reward modifiers, and enemy `HitReactResistance`.
- Enemy type differences should mostly come from resistance and immunity policy, not hardcoded "this enemy always flinches" rules.


## 2026-07-02 Update: RewardDataAsset Design Boundary

- `RewardDataAsset` is a planned reward card definition owner, not an implemented runtime class yet.
- Current runtime rewards still use `RewardActor`, `RewardSelectionWidget`, `RewardTypes.h`, and `FRewardCardOption`.
- Future `RewardDataAsset` should own reward identity, display data, apply type, effect type/id, grade, values, tags, eligibility, and implementation status.
- Future `RewardDataAsset` should feed `FRewardCardOption` or a replacement UI transfer struct.
- Reward application should remain in runtime systems such as `RewardSelectionWidget`, `PlayerStatsComponent`, `HealthComponent`, and later dedicated reward effect handlers.
- Reward definitions should not directly mutate `PlayerStatsDataAsset`, write save data, control Stage progression, or hardcode Niagara/animation timing.
- Reward pool selection, grade weighting, Luck influence, and Stage-type filtering should remain separate from individual reward definitions. A future reward pool asset, reward table, or Stage reward resolver can own that policy.


## 2026-07-02 Update: RoomEncounterDataAsset First Implementation

- `URoomEncounterDataAsset`: reusable per-Room encounter data for initial spawns, wave spawns, spawn delay, placed enemy activation policy, and clear condition.
- `ARoomCombatActor`: Room-level runtime owner. It reads optional `EncounterData`, tracks spawned and placed enemies, and keeps existing direct spawn settings as fallback.
- `AEnemyBase`: now has `EnemyActivationState`, `ActivateEnemy()`, `DeactivateEnemy()`, and `IsAlive()` so placed enemies can remain dormant until a Room wakes them.
- `StageFlowManager` remains the Stage-level Room sequencing owner; it does not own per-Room enemy composition.
- `EnemySpawnSubsystem` remains global test spawn only.


## 2026-07-01 Update: Weapon Trail Timing Ownership

- `WeaponTrailComponent` owns weapon-trail asset selection and attachment settings per combo step.
- `AnimNotifyState_WeaponTrail` owns formal attack-animation trail timing when `bUseAnimNotifyTrailTiming` is enabled.
- `Fallback Activation Delay` and `Fallback Duration` are not the formal timing controls; they exist only for fallback attack-start playback when notify timing is disabled.
- Notify timing and fallback timing are now mutually exclusive, so the editor setting should be interpreted as a mode choice rather than two active timing layers.
## 2026-07-01 Update: Run / Stage / Stage Map Ownership Candidate

- `RoomCombatActor` remains the Room-level owner for combat lock, enemy spawning, alive tracking, and room clear. In manager-owned Stages, reward timing and next-room handoff are owned by `StageFlowManager`.
- A Stage Map node represents one `Stage`, not one `Room`.
- A normal Stage contains multiple Rooms. The current planning baseline is around Room 0~6, but later Stages may vary by type.
- `StageFlowManager` should later own current Run Stage state, ordered Rooms inside the chosen Stage, Stage Clear, return to Stage Map, next Stage candidate generation, Mid Boss Stage, Boss Stage, and Run Clear.
- First `StageFlowManager` MVP should be smaller: one active Stage, ordered Rooms, next-Room activation, and Stage Clear fallback only.
- Stage Map return, next Stage candidate generation, Mid Boss Stage, Boss Stage, and Run Clear should stay deferred until the first manager path is stable.
- `StageMapWidget` is the future UI candidate for showing Stage nodes, paths, selectable next Stages, and locked/skipped branches.
- `StageNodeDataAsset` is a future data candidate for Stage type, danger, reward tendency, connected next nodes, theme, and special rules.
- `RoomEncounterDataAsset` should own Room/Encounter/Wave data inside a Stage, not the whole Run route.
- `RewardDataAsset` or a reward table should later own Stage Clear reward definitions, grade, weight, prerequisite rules, and effect ids.
- Future Stage Clear flow should be ordered as reward card selection, reward application, Stage Map display, then connected next Stage selection.
- `StageMapWidget` should not show selectable next Stage candidates until the Stage Clear reward selection has completed.
- The existing `RewardActor` / `RewardSelectionWidget` flow is now reused for Stage Clear reward timing in manager-owned Stages. It is still not the final reward data/grade/pool system.

## 2026-07-01 Update: StageFlowManager Initial Signal Bridge

- `StageFlowManager` exists as `Source/MyProject/Rooms/StageFlowManager.h/.cpp`.
- `RoomCombatActor` exposes `OnRoomProgressionReady` as the first Stage-layer handoff signal.
- The initial bridge verified that the manager can auto-register Rooms by `StageId`, sort them by `RoomOrder`, receive the signal, and rebroadcast it for Blueprint or future C++ use.
- This initial bridge status has been superseded by the progression ownership transition below.
- Current manager-owned Stages use `StageFlowManager` for next-Room activation and the temporary final-Room Stage Clear reward/restart fallback.
- `RoomCombatActor.NextRoomActors`, `RoomCombatActor.bAutoFindNextRoomByOrder`, and standalone Room Clear rewards remain fallback/setup paths, not the primary manager-owned Stage path.

## 2026-07-01 Update: Progression Ownership Transition

- Current primary path: `StageFlowManager` activates the next Room after Room progression-ready and opens Stage Clear rewards on the final Room.
- Current fallback/setup path: `RoomCombatActor.NextRoomActors`, `RoomCombatActor.bAutoFindNextRoomByOrder`, and standalone Room Clear rewards remain available but are not primary in manager-owned Stages.
- Do not remove `NextRoomActors` or `bAutoFindNextRoomByOrder` because placed Rooms and fallback tests may still reference them.
- `NextRoomActors` is now fallback/manual override data while `StageFlowManager` is the primary opener in manager-owned Stages.
- `bAutoFindNextRoomByOrder` remains setup/fallback data until the current Stage is verified end to end.
- UPROPERTY fields should remain until old maps and placed Blueprints no longer depend on them.

Next implementation unit candidates:

- Step A: `StageFlowManager` calculates next Room candidate only.
- Step B: `StageFlowManager` compares its candidate with `RoomCombatActor`'s current handoff path. Implemented as a validation log.
- Step C: `StageFlowManager` activates the next Room after A/B are verified. Implemented as the primary opener.
- Step D: `StageFlowManager` handles final Room Stage Clear reward and restart fallback after C is verified. Temporary fallback is manager-owned until Stage Map exists.

Step A and Step B are validation steps. Step C changes next-Room ownership but keeps the same playable progression target.

## 2026-06-30 Update: Weapon Trail Ownership Candidate

- `WeaponTrailComponent` was added as the prepared owner for player sword-trail presentation.
- It is separate from `PlayerStatsDataAsset` and `PlayerStatsComponent`; player stats still own numbers, not VFX assets.
- `PlayerMeleeAttackSubsystem` only calls the trail component with the current combo step.
- `AnimNotifyState_WeaponTrail` is the prepared animation-side timing hook for opening and closing a weapon-trail window.
- `RewardSelectionWidget` notifies the trail component when a reward is selected so later reward cards can change attack presentation without rewriting the attack damage path.
- The current component is intentionally inactive by default and needs editor-side Niagara assignment/tuning before visible testing.

## 2026-06-27 Update: Stage 1 Auto-Link Ownership

- `RoomCombatActor` now owns a small Stage 1 MVP auto-link helper.
- It can use `StageId` and `RoomOrder` to find the next `RoomCombatActor` when `NextRoomActors` has no valid manual link.
- Manual `NextRoomActors` links remain the first priority.
- `ExpectedStageRoomCount` and progress labels are debug/editor helpers, not final stage progression data.
- This auto-link helper is now fallback/setup data for manager-owned Stages.
- `StageFlowManager` is the current primary owner for one active Stage's ordered Room sequencing and temporary Stage Clear fallback. Branching, next Stage candidate selection, and final Stage UI remain later work.

## 2026-06-27 Update: Stage Selection and Growth Ownership Candidate

- Stage planning treats a Stage as multiple Rooms, with the current baseline around a 7-Room structure.
- `RoomCombatActor` should remain focused on one Room's combat lock, enemy tracking, reward spawn, and next-room handoff for the current MVP.
- `StageFlowManager` currently owns one active Stage's ordered Room sequencing and temporary Stage Clear fallback. It should later expand to next Stage candidate generation and chosen Stage transition.
- `RoomEncounterDataAsset` now owns the first reusable room/encounter data path for enemy composition, wave timing, placed enemy activation policy, and clear condition. Room tags and special-room reward metadata remain later candidates if needed.
- `RewardDataAsset` or a reward table should later own reward grade, technique target, effect value, unlock condition, and weight.
- A future player growth component or extension of `PlayerStatsComponent` may own technique levels such as `?�두르기`, `?�피`, and `?�의 검??.
- Luck is a candidate future player stat for high-grade reward appearance and should not be treated as implemented yet.

## 2026-06-26 Update: Reward Tier Ownership Candidate

- Stage planning now uses changing Stage Clear reward candidates and reward-grade tiers. Bronze/Silver/Gold/Prism are current candidate names.
- This is a planning direction, not an implemented data system.
- Current MVP reward ownership remains `RewardActor` + `RewardSelectionWidget` + `FRewardCardOption`.
- Later, `RewardDataAsset` or a reward table should own reward tier, display text, effect id, candidate pool, weight, and prerequisite rules.
- Later, Stage-level data or reward data should own Stage Clear reward candidates, reward card count, and reward tier policy. `RoomEncounterDataAsset` may keep special-room reward metadata only if a specific Room needs it.

## 2026-06-24 Update: Current MVP Ownership Map

- `RoomCombatActor` owns room trigger, enemy spawn, alive tracking, and room clear. `StageFlowManager` now owns progression, Stage Clear reward timing, and the temporary last-room restart overlay in manager-owned Stages.
- `RewardActor` is the temporary reward carrier and creates the reward card widget. It is reused for Stage Clear rewards in manager-owned Stages.
- `RewardSelectionWidget` is the temporary runtime card UI and applies the selected reward directly.
- `RewardTypes.h` holds `ERewardCardEffect`, `ERewardApplyType`, and `FRewardCardOption`.
- `FRewardCardOption.RewardId` is the temporary stable identity for reward cards until reward definitions move to a data asset or table.
- `PlayerStatsComponent` owns runtime run reward modifiers and does not mutate `PlayerStatsDataAsset`.
- `PlayerHealthSubsystem` creates the health orb, handles player death, and clears run reward modifiers on death.
- `EnemyBase` owns common enemy movement, chase, separation, melee damage timing, spawn intro, attack playback, and death handling.
- `EnemySimpleAnimationComponent` is still the fallback idle/run animation driver for enemies without a dedicated AnimBP path.

Later ownership candidates:

- `StageFlowManager`: stage order, final clear, restart/menu flow, branching, and stage-level state.
- `RoomEncounterDataAsset`: reusable per-room enemy waves, spawn timing, special-room reward hooks, elite/boss metadata, and special rules.
- `RewardDataAsset` or reward table: reward pools, rarity, weights, descriptions, and effect definitions after the MVP reward rules settle.
- Dedicated UI widgets/assets: final reward cards, stage clear menu, and player HUD polish.

## 2026-06-24 Update: Data Structure Candidate Map

Current direct `RoomCombatActor` settings:

- Stage MVP: `StageId`, `ExpectedStageRoomCount`, `bAutoFindNextRoomByOrder`, and progress debug labels.
- Identity: `RoomId`, `RoomOrder`, and `RoomDisplayName` for editor organization and debug labels.
- Activation: player overlap start behavior.
- Encounter fallback: single class plus count for old tests.
- Initial wave: enemies that spawn together at combat start.
- Sequential queue: enemies spawned one at a time after prior tracked enemies die.
- Spawn points: optional placed transforms for room-specific spawn positions.
- Reward/progression: reward pickup class, reward spawn transform, next room actors, and last-room fallback UI.
- Debug: per-room debug message toggle, message durations, and spawn preview duration.

Current `StageFlowManager` responsibilities:

- One active Stage id.
- Ordered `RoomCombatActor` list for that Stage.
- Next Room activation after Room progression-ready.
- Temporary Stage Clear reward/restart fallback when no next Room exists.

Later `StageFlowManager` candidates:

- Later: Stage start/reset across multiple Stage nodes.
- Later: Stage Map return and branching route decisions.
- Later: Mid Boss Stage, Boss Stage, Final Boss, and Run Clear.

Current `RoomEncounterDataAsset` responsibilities:

- Initial spawn entries.
- Wave spawn entries.
- Spawn delay.
- Placed enemy activation policy.
- Placed enemy detection radius.
- Clear condition.

Later `RoomEncounterDataAsset` candidates:

- Encounter identity and room type tags.
- Spawn point policy or spawn group names.
- Reward candidates and reward count policy only if a special Room needs it.

Direct `RoomCombatActor` fields remain fallback/editor override data while encounter assets are created gradually.

## 2026-06-23 Update: Reward Runtime Classes

- `RewardTypes.h` defines the current reward card effect and apply type enums.
- `ERewardApplyType` separates rewards into `Instant`, `Run`, and `Permanent`.
- `RewardActor` owns the temporary reward carrier/card options and is reused for Stage Clear rewards in manager-owned Stages.
- `RewardSelectionWidget` applies the selected reward based on its apply type.
- `PlayerStatsComponent` owns current-run stat modifiers and does not mutate `PlayerStatsDataAsset`.
- `PlayerHealthSubsystem` clears run reward modifiers on player death.
- `Permanent` rewards are placeholder-only until save/load, lobby upgrade UI, and currency systems are designed.

## 2026-06-23 Update: Fast/Tank MVP Enemy Classes

- `BP_Enemy_Fast` exists at `/Game/Blueprints/Enemy/Fast/BP_Enemy_Fast`.
- `BP_Enemy_Tank` exists at `/Game/Blueprints/Enemy/Tank/BP_Enemy_Tank`.
- Both are temporary Blueprint enemies that reuse the `EnemyBase` C++ behavior.
- `DA_EnemyStats_Fast` and `DA_EnemyStats_Tank` hold their current gameplay stat differences.
- Fast now uses `/Game/Demons_Big_Pack/DemonHeavy/Mesh/SK_DemonHeavy` with `/Game/Blueprints/Enemy/Fast/ABP_DemonHeavy_Fast`.
- Fast attack playback uses `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Attack1` through `DefaultSlot`, with `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Strong_Attack3` as every third strong attack. Fast strong attack also uses `StrongAttackDamageMultiplier=1.4` and `StrongAttackRangeMultiplier=1.15`. Tank currently uses the modified Butcher mesh with `ABP_Butcher`, but enemy art is still subject to later direction changes.
- `RoomCombatActor.InitialSpawnEnemyClasses` is now the first-wave simultaneous spawn field for rooms.
- Room1 currently uses `InitialSpawnEnemyClasses = [BP_Enemy_Fast, BP_Enemy_Tank]`.

## 2026-06-22 Update: RoomCombatActor Enemy Array

- `RoomCombatActor` supports `SpawnEnemyClasses` for ordered per-room enemy class spawning.
- Array mode uses the list as a sequential queue, spawning one listed `EnemyBase` subclass at a time.
- The next listed enemy spawns after the currently tracked enemy dies and `SpawnDelayBetweenEnemies` has elapsed.
- The current temporary spawn delay is `0.5` seconds.
- Empty array mode falls back to `EnemyClass` plus `EnemiesToSpawn`, preserving the existing Little Demon room setup.
- This is a Stage 1 direct actor setting and can later feed into or be replaced by `RoomEncounterDataAsset`.

## 2026-06-22 Update: Room Encounter Data Direction

- `RoomCombatActor` can still own room-specific MVP settings directly per placed room actor.
- `RoomEncounterDataAsset` now exists as the first data-driven encounter class.
- The current transition is: direct `RoomCombatActor` settings remain fallback/editor override data, while reusable Room enemy groups and wave rules move into `RoomEncounterDataAsset` over time.
- Reward candidates and special room metadata should stay Stage/reward-owned unless a specific Room needs a special-case hook later.

## 2026-06-22 Update: Stage Clear Fallback

- `StageFlowManager` now owns the temporary last-room fallback for manager-owned Stages.
- When room progression completes, `StageFlowManager` calculates the next Room; no valid next Room means Stage Clear reward/restart fallback.
- Full Stage Map branching and final reward data remain later ownership.

## 2026-06-21 Update: Room Progression Ready State

- `StageFlowManager` now owns the MVP next-room progression gate for manager-owned Stages.
- Standalone Room rewards can still wait for `RewardActor.OnRewardClaimed`; manager-owned Stages disable Room rewards and wait for Stage Clear reward only at the final Room.
- `NextRoomActors` remains fallback/setup data when the room is ready for next-room movement.
- `StageFlowManager` drives the current manager-owned Stage flow from the progression-ready signal.

## 2026-06-21 Update: Reward Card Selection MVP

- `RewardSelectionWidget` is the first runtime reward card UI.
- `RewardActor` opens `RewardSelectionWidget` when triggered by Stage Clear or when the player touches a standalone test pickup.
- The first implementation starts with `2` reward cards and supports up to `4`.
- Current card effects are health-focused test effects.

## 2026-06-21 Update: Room Reward Pickup MVP

- `RewardActor` is the first temporary reward carrier class.
- `RoomCombatActor` can still spawn `RewardActor` after room clear for standalone tests; `StageFlowManager` spawns/opens it at Stage Clear for manager-owned Stages.
- The current reward flow opens card selection and applies only the selected card effect.
- This is a temporary card pipeline, not the final reward data design.

## 2026-06-21 Update: Enemy Spawn Intro

- `EnemyBase` owns optional spawn intro playback through `SpawnIntroAnimation`.
- While `SpawnIntroAnimation` is active, `EnemyBase` locks chase and melee attack logic, then restores normal movement after the animation finishes.
- `BP_Enemy_LittleDemon` currently assigns `Anim_Little_Demon_rage` to this spawn intro slot.

## 2026-06-20 Update: Debug Slow Motion Input

- `PlayerKeyboardMovementSubsystem` owns the `F10` debug slow motion toggle.
- The toggle sets global time dilation to `0.1x` and restores it to `1.0x` on the next press.

## 2026-06-20 Update: Little Demon Attack Animation Path

- `BP_Enemy_LittleDemon.AttackAnimation` references the original `Anim_Little_Demon_attack1` sequence.
- `EnemyBase` handles sequence attack assets through runtime Dynamic Montage playback when a valid AnimBP Slot is available.
- `BP_Enemy_LittleDemon` currently uses the SingleNode/SimpleAnimation path for MVP stability.
- `ABP_LittleDemon` exists as a transition asset, but it is not the current Little Demon runtime path.

## 2026-06-28 Update: Tank Butcher AnimBP Path

- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/ABP_Butcher`.
- `ABP_Butcher` uses `EnemyAnimInstance` as its parent class and targets the Butcher skeleton.
- `BP_Enemy_Tank.Mesh` uses the modified Butcher source-skeleton mesh and `AnimationBlueprint` mode.
- Tank weak attack1 is configured through `EnemyBase.AttackAnimation` with `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Weak_Attack1`.
- Tank strong attack3 is configured through `EnemyBase.StrongAttackAnimation` with `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Strong_Attack3`.
- `BP_Enemy_Tank.StrongAttackEveryNthAttack = 2`, so the second successful melee attack uses attack3 as the strong attack.
- Tank attack playback now uses `DefaultSlot` Montage playback for both weak and strong attack assets.
- Tank movement currently uses `Anim_Butcher_walk1` in `ABP_Butcher`; `Anim_Butcher_run1` is avoided for now because it produced visible loop stutter.
- `EnemySimpleAnimationComponent` is still present as a fallback component type, but its Tank idle/run assignments are cleared because Tank is now an AnimBP-driven enemy.

## 2026-06-19 Update: Enemy AnimBP Foundation

- `EnemyAnimInstance` is the C++ parent class for dedicated enemy Animation Blueprints and stabilizes run/idle switching with separate movement start/stop thresholds.
- It provides `GroundSpeed`, `bIsMoving`, and `bIsDead` to AnimBP graphs.
- `ABP_LittleDemon` exists at `/Game/Blueprints/Enemy/LittleDemon/ABP_LittleDemon`.
- `EnemySimpleAnimationComponent` remains the fallback for enemies without a dedicated AnimBP, but disables itself when the mesh already uses an AnimBP class. Fast and Tank are now AnimBP-driven; Little Demon remains on the fallback path for MVP stability.

## 2026-06-15 Update: Enemy Attack Dynamic Montage Path

- `EnemyBase` now attempts Dynamic Montage playback for attack animations only when a valid AnimBP class and Slot path are available.
- Enemies without an AnimBP class play configured sequence assets directly through the mesh SingleNode path.
- `AttackAnimationSlotName`, `AttackAnimationBlendInTime`, and `AttackAnimationBlendOutTime` tune attack transitions.
- Enemy attack playback is currently based on a default `AttackAnimation` plus an optional configured `StrongAttackAnimation`; the earlier random attack selection idea remains removed from the MVP.

## 2026-06-14 Update: Player Health Orb UI MVP

- `PlayerHealthOrbWidget` is the current C++ runtime widget for the player's circular health orb.
- `PlayerHealthSubsystem` owns creation and updates for this widget.
- The frame image is loaded from `Content/UI/Textures/T_UI_HealthOrb_Frame.png`.
- This is the first player HUD path; the old `Player HP` on-screen debug text has been removed.

## 2026-06-14 Update: Enemy Separation MVP

- `EnemyBase` now owns soft separation movement between nearby enemies.
- `EnemyStatsDataAsset` exposes `SeparationRadius`, `SeparationStrength`, and `MaxSeparationNeighbors`.
- Runtime separation stores target and smoothed separation directions in `EnemyBase` to reduce frame-to-frame jitter.
- This is a movement behavior, not a room spawn rule, so `RoomCombatActor` and `EnemySpawnSubsystem` do not own it.

## 2026-06-19 Update: Little Demon Single Attack Animation MVP

- `EnemyBase` owns single attack animation playback through `AttackAnimation`.
- The previous `AttackAnimations` random pool has been removed from `EnemyBase`.
- `BP_Enemy_LittleDemon` should use `Anim_Little_Demon_attack1` as the current MVP attack animation.

## 2026-06-12 Update: Auto Room Boundary MVP

- `RoomCombatActor` owns four automatic boundary blocker components for the current room-lock MVP.
- `BoundaryBlocker_North/South/East/West` block Pawn collision only while room combat is active.
- `OnConstruction()` refreshes the boundary preview in the editor when boundary settings change.
- `LockRoomBoundary()` enables the blockers, and `UnlockRoomBoundary()` disables them.
- A future `CombatLockVolume` or `RoomBoundaryActor` can take over this responsibility when room shapes become more complex.

## 2026-06-12 Update: Room Door MVP

- `RoomCombatActor` owns the current door open/close MVP through `DoorActors`.
- Door close means `SetActorHiddenInGame(false)` and `SetActorEnableCollision(true)`.
- Door open means `SetActorHiddenInGame(true)` and `SetActorEnableCollision(false)`.
- `RoomDoorActor` remains a later candidate for animated or stateful doors.

## 1. 문서 목적

?�재 ?�로?�트?�서 Codex가 ??�??�상 ?�인?�거???�정??C++ ?�래?�의 ??��???�리?�다.

?�래?��? 존재?�다�??�서 ?�재 ?�레?�에???�제�??�용 중이?�는 ?��? ?�니?? ?�제 ?�용 ?��???`Config/DefaultEngine.ini`, ?�재 맵의 GameMode, Blueprint 부�??�래?? ?��???Subsystem ?�결???�께 ?�인?�야 ?�다.

## 2. ?�재 ?�제 ?�레???�름

```text
/Game/TopDown/Lvl_TopDown
-> /Game/Blueprints/Core/BP_CombatGameMode
-> /Game/Blueprints/Player/BP_PlayerCharacter
-> /Game/Blueprints/Player/BP_PlayerController
```

주의:

- `MyProjectCharacter`?�?`MyProjectPlayerController`??기본 ?�플�?C++ ?�래?��?�? ?�재 기본 ?�레??Blueprint가 직접 부모로 ?��? ?�는 것으�??�인?�었??
- ?�재 ?�심 기능 ?�수???�제 Blueprint 부모�? 바꾸지 ?�기 ?�해 `WorldSubsystem` 방식?�로 붙어 ?�다.

## 3. ?�심 ?��????�래??
| ?�래???�일 | ??�� | ?�제 ?�용 ?��? | 비고 |
|---|---|---|---|
| `MouseWheelCameraZoomSubsystem.*` | 마우????�?처리 | ?�용 �?| ?�재 Pawn??`SpringArm`??찾아 거리 조절 |
| `PlayerKeyboardMovementSubsystem.*` | WASD ?�동, ?�릭 ?�동 ?�거, ?�트�??�니메이?? 무기 ?�켓 ?�환, Space ?�피 | ?�용 �?| 조작 관???�심 Subsystem |
| `PlayerMeleeAttackSubsystem.*` | 좌클�?근접 콤보 공격, 부채꼴 ?��?지, 공격 ?�버�? 공격 �??�동 ?? ?�피 캔슬 ?�동 | ?�용 �?| ?�름?�?Auto지�??�재???�동 공격 ?�당 |
| `WeaponTrailComponent.*` | ?�레?�어 검 궤적 VFX ?�롯�?보상 기반 ?�출 ?�태 준�?| 구조 준비됨 | 기본 비활?? Niagara/?�켓/?�?�밍 ?�닝?�??�디???�인 ?�요 |
| `AnimNotifyState_WeaponTrail.*` | 공격 ?�니메이?�에??검 궤적 Trail 구간??켜고 ?�는 NotifyState | 구조 준비됨 | ?�제 ?�니메이???�셋 배치???�직 ?�디???�업 ?�요 |
| `HealthComponent.*` | 공통 체력, ?��?지, ?�복, ?�망 ?�벤?? 무적 ?�태 | ?�용 �?| ?�레?�어/??공통 체력 컴포?�트 |
| `PlayerHealthSubsystem.*` | ?�레?�어 체력 ?�결, 체력 ?�브 HUD ?�성/갱신, H ?�스???��?지, ?�레?�어 ?�망 처리, ?�망 ?�니메이??VFX | ?�용 �?| ?�재 ?�레?�어 Pawn???��???보장 |
| `PlayerHealthOrbWidget.*` | 좌측 ?�단 ?�형 ?�레?�어 체력 ?�브 HUD | ?�용 �?| `Content/UI/Textures/T_UI_HealthOrb_Frame.png` ?�레?�과 코드 ?�성 ?�체 ?�스�??�용 |
| `PlayerStatsComponent.*` | ?�레?�어 ?�탯 ?�공 | ?�용 �?| Data Asset???�으�?�?값을 ?�선 ?�용 |
| `PlayerStatsDataAsset.*` | ?�레?�어 ?�탯 ?�이???�셋 ?�??| ?�용 �?| 체력, 공격, 콤보, ?�피 ?�치 |
| `EnemyBase.*` | ??공통 기반 Actor | ?�용 �?| 체력, 추적, 근거�?공격, ?�망 처리 |
| `EnemyAnimInstance.*` | ???�용 AnimBP 부�??�래??| ?�용 ?�보/?�환 �?| AnimBP??`GroundSpeed`, `bIsMoving`, `bIsDead` ?�공 |
| `EnemySimpleAnimationComponent.*` | ?�용 AnimBP가 ?�는 ?�의 ?�시 Idle/Move ?�니메이???�환 | ?�용 �?| Little Demon ?�에???�용 |
| `EnemySpawnSubsystem.*` | ?�역 ?�스???�폰 관�?| ?�용 �?| 기본 OFF. 콘솔 변?��? `EnemySpawnDataAsset`??명시?�으�?켰을 ?�만 ?�작?�는 ?�시 검증용 ?�폰 |
| `EnemySpawnDataAsset.*` | ???�폰 규칙 ?�이???�셋 ?�??| ?�용 �?| ?�역 ?�스???�폰 ?�정. ?��? �??�폰 ?�이?�로 ?�사??가??|
| `EnemyStatsDataAsset.*` | ??개체 ?�력�??�이???�셋 ?�??| ?�용 �?| 체력, ?�동?�도, ?��? 거리, 캡슐 겹침 ?�유거리, 공격?? 공격 ?�거�? 공격 쿨�???|
| `SurvivorSquareEnemy.*` | ?�스?�용 ?�모 ??| ?�용 �?| `EnemyBase` ?�속, ?�시 ?�스????|
| `RoomCombatActor.*` | �??�투 MVP Actor | ?�용 �?| Trigger ?�장, ???�폰, �?�?경계 ?�금, ?�망 ?�벤??추적, `Room Cleared` 출력, �?경계 ?�제 |
| `RewardActor.*` | �??�리??보상 ?�업 MVP Actor | ?�용 �?| `RoomCombatActor`가 �??�리?????�성. ?�레?�어가 ?�으�?`RewardSelectionWidget`???�고, 카드 ?�택 ???�과 ?�용 |

## 4. ?�레?�어 조작 관??
### `Source/MyProject/Input/PlayerKeyboardMovementSubsystem.h/.cpp`

??��:

- 기존 TopDown ?�릭 ?�동 Mapping Context ?�거
- WASD ?�동 ?�용
- 게임 ?�작 ??`Katana_Blade_Intro` ?�생
- ?�트�?�??�동 ?�금
- ?�트�?�?검??`SheathSocket_Back`?�서 `HandGrip_R`�??�환
- Space ?�피 처리
- ?�피 �?무적 ?�간 처리
- 공격 �?Space ?�피 캔슬???�해 `PlayerMeleeAttackSubsystem`�??�동

주의:

- ?�동/?�피/?�트�?무기 ?�켓 관???�정?�????�일??먼�? ?�인?�다.
- 공격 ?�문???�동 ?�력???�긴 경우?�도 Space ?�피???�외?�으�??�용?�다.

### `Source/MyProject/Camera/MouseWheelCameraZoomSubsystem.h/.cpp`

??��:

- ?�재 ?�레?�어 Pawn??`USpringArmComponent`�?찾아 마우???�로 카메??거리 조절
- ?�정 Character/Controller ?�속 구조???�존?��? ?�음

## 5. 공격 관??
### `Source/MyProject/Combat/PlayerMeleeAttackSubsystem.h/.cpp`

??��:

- 좌클�?근접 공격 ?�력 감�?
- 1/2/3?�?콤보 ?�계 관�?- 콤보 ?�력 버퍼�?- 마우??커서 방향?�로 캐릭???�전
- 부채꼴 범위 ?�의 모든 `EnemyBase`?�게 ?��?지
- 공격 �?짧�? ?�진 ?�텝
- 공격 ?�니메이???�생
- 공격 �??�동 ?�력 ?�금
- 공격 �?Space ?�피 캔슬 처리 지??- ?�재 콤보 ?�계??맞춰 `WeaponTrailComponent`??검 궤적 ?�생 ?�청
- ?�중 ??짧�? ?�트?�톱 처리
- 공격 범위/콤보 ?�버�??�시

주의:

- ?�름?�?`PlayerMeleeAttackSubsystem`?��?�??�재???�동 공격???�니??
- ?�재 ??��?�??�좌?�릭 ?�동 근접 콤보 공격 MVP?�이??
- 추후 ?�식 구조?�서??`BasicAttackComponent` ?�는 `PlayerMeleeAttackComponent`�???�� ?�보?�다.

### `Source/MyProject/Combat/HealthComponent.h/.cpp`

??��:

- `MaxHealth`, `CurrentHealth` 관�?- ?��?지/?�복 처리
- ?�망 ?��? 관�?- ?��?지 ?�벤?��? ?�망 ?�벤???�공
- 무적 ?�태 처리

주의:

- 공격/?�킬/?�트 ?��?지????컴포?�트???��?지 ?�수�??�출?�는 방향???��??�다.
- Character마다 체력 코드�??�로 만들지 ?�는??

## 6. ?�레?�어 체력/?�망 관??
### `Source/MyProject/Combat/PlayerHealthSubsystem.h/.cpp`

??��:

- ?�제 ?�레??Pawn??`HealthComponent`가 ?�으�??��??�에??붙임
- H ?�로 ?�버�??��?지 ?�용
- 좌측 ?�단 ?�형 체력 ?�브 HUD ?�성/갱신
- 좌측 ?�단 ?�버�?체력 ?�시
- 체력 0 ?�하 ???�망 처리
- ?�동/?�력 비활?�화
- `/Game/Sword_and_shield/Animations_UE5/Death/RM_death_01` ?�망 ?�니메이???�생
- Niagara ?�망 VFX ?�생

주의:

- ?�식 구조?�서???�레?�어 Blueprint??직접 `HealthComponent`�?붙이??방식?�로 ?�리?????�다.
- ?�재 체력 HUD??`Source/MyProject/UI/PlayerHealthOrbWidget.h/.cpp`???�으�? Blueprint ?�젯?�??�직 ?�용?��? ?�는??

## 7. ?�레?�어 ?�탯 관??
### `Source/MyProject/Combat/PlayerStatsComponent.h/.cpp`

??��:

- ?�레?�어 ?�탯 값을 ?��??�에 ?�공
- `StatsDataAsset`??지?�되???�으�?Data Asset 값을 ?�선 ?�용
- Data Asset???�으�?컴포?�트 기본�??�용

관�?�?

- 최�? 체력
- 기본 공격 ?��?지
- 공격 범위
- 공격 각도
- 공격 쿨�???- 콤보 리셋 ?�간
- 최�? 콤보 ?�계
- 콤보�??�진 거리
- 콤보 ?�진 ?�간
- 콤보 ?�환 비율
- 공격 ?�니메이??Slot ?�름
- ?�피 거리/?�간/쿨�???무적 ?�간
- ?�버�??�시 ?��?

### `Source/MyProject/Combat/PlayerStatsDataAsset.h/.cpp`

??��:

- ?�디?�에??조절 가?�한 ?�레?�어 ?�탯 ?�이???�셋 ?�??- ?�재 기본 ?�셋 ?�보: `/Game/Data/Player/DA_PlayerStats_Default`

## 8. ??관??
### `Source/MyProject/Enemies/EnemyBase.h/.cpp`

??��:

- ??공통 기반 Actor
- `HealthComponent` 보유
- `EnemySimpleAnimationComponent` 보유
- ?�레?�어 ?�색/추적
- ?�레?�어/??캡슐 반�?름을 고려???��? 거리 관�?- 근거�?공격 ?��?지/?�거�?쿨�???관�?- `EnemyStatsDataAsset`??지?�되???�으�?체력/?�도/?��? 거리/겹침 ?�유거리/공격???�거�?쿨�???값을 ?�선 ?�용
- `DeathAnimation`??지?�되???�으�??�망 ???�당 ?�니메이?�을 1???�생
- `bDestroyOnDeath`�??�망 ??즉시 ?�거 ?��?�??�어
- ?�정 ?�의 Mesh/AnimBP??C++?�서 ?�드코딩?��? ?�고 ??Blueprint?�서 관�?- ?�격 ??짧고 부?�러???�백 ?�수 보유, ?�재 기본 공격?�서???�출?��? ?�음
- 기본값�? ???�망 ??Destroy 처리, Little Demon?�?Blueprint?�서 Destroy�??�고 ?�망 ?�니메이?????��?

주의:

- 빠른 ?? ?�커, ?�거�????��? ??구조�?기�??�로 ?�장?�는 방향???�선?�다.

### `Source/MyProject/Variant_Survivor/SurvivorSquareEnemy.h/.cpp`

??��:

- ?�스?�용 ?�모 ??- ?�재??`EnemyBase`�??�속?�는 간단???�스????- ?�제 게임 ??모델/?�니메이?�이 ?�정?�기 ?�까지 ?�시 ?�으�?본다.

### `Source/MyProject/Enemies/EnemySpawnSubsystem.h/.cpp`

??��:

- ?�역 ?�스???�폰/?�시 검증용?�로 ?�레?�어 주�??????�성
- 기본 ?�태?�서???�동 ?�폰?��? ?�음
- `MyProject.EnemySpawn.GlobalTestSpawnEnabled=1` 콘솔 변?��? `EnemySpawnDataAsset.bSpawnEnabled=true`가 모두 ?�요
- ?�스??�?최�? ?�아?�는 ?????��?
- ?�스?�용 ?�작 지?? ?�폰 간격, ?�폰 거리 관�?- `EnemySpawnDataAsset` 값을 ?�선 ?�용
- `EnemySpawnDataAsset.SpawnEnemyEntries`?�서 ?�재 ?�간???�성?�된 ?�을 가중치 ?�덤?�로 ?�성
- ?�성 Entry가 ?�으�?`EnemySpawnDataAsset.EnemyClass` ?�는 기본 Imp Blueprint�?fallback?�로 ?�성

주의:

- ?�재 게임 방향?�?�??�리?�형 ?�테?��? ?�파 구조??
- `EnemySpawnSubsystem`?�???��?��? ?��?�??�식 진행 구조??중심?�로 보�? ?�는??
- ?�으�?�??�위 ???�성?�?`RoomCombatActor`, `RoomSpawnPoint`, `RoomEncounterDataAsset` ?�보 구조�??�환?�다.

### `Source/MyProject/Enemies/EnemySpawnDataAsset.h/.cpp`

??��:

- ???�폰 규칙 ?�이???�셋 ?�??- ??개체 ?�력치�? ?�니???�폰 규칙�?관리한??
- ?�러 ??Blueprint Class�??�간/가중치 기�??�로 ?�택?????�다.

관�?�?

- ?�폰 ?�성???��?
- ?�간/가중치 기반 ?�폰 목록
- fallback ?�폰 ?�래??- 최�? ????- ??번에 ?�폰????- 초기 ?�폰 지??- ?�폰 간격
- ?�폰 거리

### `Source/MyProject/Enemies/EnemySimpleAnimationComponent.h/.cpp`

??��:

- ?�용 Animation Blueprint가 ?�직 ?�는 ?�을 ?�한 ?�시 ?�니메이??컴포?�트
- BP?�서 `IdleAnimation`, `MoveAnimation`??지?�하�??�동 ?�도???�라 ?�일 ?�니메이?�을 ?�환
- ?�정 몬스???�셋 경로�?C++???�드코딩?��? ?�는??

?�재 ?�용:

- `BP_Enemy_LittleDemon`

주의:

- ?�기?�으로는 ?�용 AnimBP ?�는 Montage 구조�??�격?????�다.

### `Source/MyProject/Enemies/EnemyStatsDataAsset.h/.cpp`

??��:

- ??개체 ?�력�??�이???�셋 ?�??- `EnemyBase`??`StatsDataAsset` ?�롯??지?�해???�용
- Data Asset??지?�되�?기존 C++ 기본값보???�선 ?�용

관�?�?

- ??고유 ID
- ???�름
- 최�? 체력
- ?�동?�도
- ?��? 거리
- 캡슐 겹침 ?�유거리
- 공격??- 공격 ?�거�?- 공격 쿨�???
## 9. 기본 ?�플�??�재 비핵???�래??
| ?�래???�일 | ??�� | ?�재 ?�단 |
|---|---|---|
| `MyProject.h/.cpp` | ?�로?�트 공통 모듈/로그 | 공통 |
| `MyProjectGameMode.*` | 기본 ?�플�?GameMode | ?�재 TopDown BP ?�름?�서 직접 ?�심 ?�정 ?�치 ?�님 |
| `MyProjectCharacter.*` | 기본 ?�플�?Character | ?�재 ?�제 ?�레??Character 부모로 보�? ?�음 |
| `MyProjectPlayerController.*` | 기본 ?�플�?PlayerController | ?�재 ?�제 ?�레??Controller 부모로 보�? ?�음 |
| `Variant_Strategy/*` | Unreal ?�플�?변??코드 | ?�재 게임 ?�심 ?�름 ?�님 |
| `Variant_TwinStick/*` | Unreal ?�플�?변??코드 | ?�재 게임 ?�심 ?�름 ?�님 |

## 9.5 �??�투 관??
### `Source/MyProject/Rooms/RoomCombatActor.h/.cpp`

??��:

- `TriggerBox` Overlap?�로 ?�레?�어 �??�장 감�?
- `EnemyClass` 기�? ???�성
- `SpawnPoints` 배열 기�? ?�치 지??- `EnemiesToSpawn` ?�만?????�성
- ?�성???�의 `HealthComponent.OnDeath` ?�벤??추적
- 모든 ???�망 ??`Room Cleared` ?�버�?메시지 출력

주의:

- �??�힘/?�림, ?�동 �?경계, 보상 ?�성, 보상 ?�택 ?��? `NextRoomActors` 기반 ?�음 �?진행, 마�?�?�??�테?��? ?�리???�시?��? `RoomCombatActor`/`RewardActor` MVP??구현?�어 ?�다.
- `EnemySpawnSubsystem`???��???��?��? ?�고, �??�투 MVP??별도 Actor�??�다.
- Little Demon???�려�??�디?�에??`EnemyClass`??`BP_Enemy_LittleDemon`??지?�한??

## 10. ?�셋/Blueprint ?�결 메모

| ??�� | ?�치 | 비고 |
|---|---|---|
| ?�레?�어 Blueprint | `/Game/Blueprints/Player/BP_PlayerCharacter` | ?�제 ?�레??캐릭??|
| 검 컴포?�트 | `EquippedOtachi` | `BP_PlayerCharacter` |
| 검�?컴포?�트 | `EquippedOtachiSheath` | `BP_PlayerCharacter` |
| ???�켓 | `HandGrip_R` | `SKM_Manny_Simple`?�서 조정 |
| 검�??�켓 | `SheathSocket_Back` | `SKM_Manny_Simple`?�서 조정 |
| 카�????�셋 | `/Game/Weapon/Katana` | 무기 ?�셋 |
| 공격 ?�니메이??| `/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/2_Attacks/0__3Combos` | 1/2/3?�?|
| ?�트�??�니메이??| `/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/1_Movements/0__Intro` | ?�작 ?�출 |
| ?�레?�어 ?�망 ?�니메이??| `/Game/Sword_and_shield/Animations_UE5/Death/RM_death_01` | ?�망 ???�생 |
| ?�망 VFX | `/Game/SuperVFXPack/NiagaraSystem/NS_BPDissolve` | Niagara ?�망 ?�출 |
| �??�투 Blueprint | `/Game/Blueprints/Rooms/BP_RoomCombatActor` | `RoomCombatActor` 기반 �??�투 MVP |

## 11. ?�음 ?�리 ?�보

- `PlayerMeleeAttackSubsystem` ?�름�???�� ?�리
- `PlayerKeyboardMovementSubsystem`???�동/?�피/?�트�??�착?�로 분리?��? 검??- ?�스???�을 Blueprint 기반 `BP_EnemyBase` 구조�??�장
- ?�식 `BasicAttackComponent` ?�는 `PlayerMeleeAttackComponent` ?�입 ?��? 결정

## 12. ?�테?��? ?�리?�형 ?�환 ?�보 ?�래??
?�래 ?�래??블루?�린?�는 ?�직 구현 ?�정 ?�일???�니??�??�리?�형 MVP�??�한 ?�보 구조??

| ?�보 | ??�� | ?�선?�위 |
|---|---|---|
| `RoomCombatActor` | �??�장 감�?, ???�폰, ?�아?�는 ??추적, �??�리???�버�?출력 | C++ MVP 구현??|
| `RoomEncounterDataAsset` | 방별 ??조합, ?�이�? 보상 ?�보 관�?| 초기?�는 ?�보, 구조 ?�정 ??분리 |
| `RoomDoorActor` | ?�투 ?�작 ??�??�힘, �??�리????�??�림 | ?�선 구현 ?�보 |
| `RewardActor` | �??�리????보상 ?�성/?�득 처리 | MVP 구현??|
| `StageFlowManager` | ?�작 �? ?�투 �? ?�리??보스�? ?�테?��? ?�리???�름 관�?| 추후 ?�장 ?�보 |

초기 MVP?�서???�덤 ?�전 ?�이 같�? 맵에 ?�수??고정 ?�스??방을 배치?�고, `RoomCombatActor`가 `EnemyClass` 배열�?`RoomSpawnPoint` 배열??직접 ?�고 ?�어???�다. Stage 1 목표 구조???�후 7�?Room ?�후?�?�?보스방으�??�장?�는 방향?�다.


> Archive note: This file preserves pre-restructure dated notes. It may contain legacy or corrupted text and must not be used as current implementation truth unless explicitly requested.
