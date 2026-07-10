---
Status: REFERENCE
Scope: Current Implementation History
Last Updated: 2026-07-06
Source of Truth: No
---

# CurrentImplementation.md

## 2026-07-02 Update: Domain Document Split

- Reward-specific rules now live in `Docs/RewardSystem.md`.
- VFX-specific rules now live in `Docs/VFXSystem.md`.
- Cross-cutting animation rules now live in `Docs/AnimationSystem.md`.
- Stage/Run hierarchy remains in `Docs/StageRunStructure.md`.
- Room execution remains in `Docs/RoomCombatSystem.md`.
- Enemy implementation remains split across `Docs/EnemySystem.md`, `Docs/EnemyArchitecture.md`, and `Docs/EnemySpawnSystem.md`.



## 2026-07-02 Update: Reward Source Planning Split

- Planning direction now separates enemy drop items from Stage Clear reward cards.
- Enemy drop items are planned for combat sustain and short timed effects, such as one-time healing, short attack buffs, short move-speed buffs, or short cooldown buffs.
- Stage Clear reward cards are planned for current-Run build growth, such as 3rd-hit sword wave, dodge-followup attack boost, attack range increase, bleed on hit, kill-heal passive, or other Run-long combat rule changes.
- `instant_heal_20` is still implemented as a temporary card option, but its long-term home is enemy drop / pickup item rather than the main Stage Clear card pool.
- `RewardSelectionWidget` should be treated as the Stage Clear card selection UI in the long-term structure.
- A future enemy drop item should use a separate pickup path. A future shared effect applier may apply both card rewards and pickup effects.
- Enemy drop item spawning, drop chance tables, timed pickup buffs, shared reward effect applier, and final reward data assets are not implemented yet.

## 2026-07-02 Update: Lightning Reward Planning Not Implemented

- Lightning is adopted as the first planned elemental reward branch for Katana/basic combo combat.
- The first planned identity test is `체인 ?�이?�닝`: confirmed 3rd combo hit -> nearby enemy chain damage.
- `?�기 ?�파?? is planned as an early/Bronze entry reward, not a Silver-only reward.
- `감전 칼날`, `과충??, and `??��검` are planned follow-up candidates, not implemented runtime cards.
- Prism unlock rules are planning only: high-risk Stage chance increase, 3+ same-element reward unlock, and later skill Lv10 unlock after a skill-level system exists.
- Prism unlock means the reward can enter the candidate pool; it does not guarantee appearance.
- Lightning reward cards, Chain Lightning targeting/damage, Shock status, elemental reward pool, Prism unlock logic, and Lightning VFX are not implemented yet.
- Detailed planning source of truth: `Docs/ElementalRewardSystem.md`.

## 2026-07-02 Update: Ground Impact Spark Notify Implemented

- `AnimNotify_GroundImpactVfx` now exists at `Source/MyProject/Combat/AnimNotify_GroundImpactVfx.h/.cpp`.
- It is a presentation-only animation notify for weapon-ground contact sparks.
- The default Niagara system is `/Game/MechanicalDamageFX/FX/NS_Sparks_Small`.
- The notify traces downward from a Blueprint trace-point component, then spawns the spark at the ground hit point.
- The default trace-point setup is `TracePointComponentName=WeaponVfx_GroundContact`. If missing, the notify can create a runtime-only trace point under `EquippedOtachi` with relative location `115, 0, 2`. Old `BladeTipTracePoint` settings are supported as a compatibility fallback, but `WeaponVfx_GroundContact` is preferred when it exists. Deprecated socket fallback remains only if the trace point name is set to None.
- Do not auto-edit `/Game/Weapon/Katana/Meshes/StaticMeshes/SM_Otachi_1` for ground spark sockets. That automated socket test caused Static Mesh Editor crashes and was reverted.
- The old owner-local fallback offset path and direct static-mesh socket editing path are deprecated for this notify because they did not follow the blade safely.
- Continuous spark effects are deactivated and destroyed after a short configurable lifetime.
- This does not apply damage, shockwave gameplay, Lightning damage, hit reaction, or camera shake.
- The notify still needs editor-side placement on the attack animation frame where the weapon visually touches the ground.
## 2026-07-04 Update: Ground Contact Impact NotifyState Implemented

- `AnimNotifyState_GroundContactImpactVfx` now exists at `Source/MyProject/Combat/AnimNotifyState_GroundContactImpactVfx.h/.cpp`.
- It is the preferred ground-impact spark path when a single-frame notify is too hard to align.
- The notify state opens a short contact window, traces downward from `WeaponVfx_GroundContact` every tick, and spawns one spark only when the trace point is close enough to the floor hit.
- `MaxContactDistance` controls the contact threshold. If the trace hits the floor but the trace point is still too high, no spark is spawned yet.
- Debug draw uses a red sphere for the trace point, a green sphere for accepted contact, an orange sphere for floor hit without contact, and a yellow trace line.
- Existing `AnimNotify_GroundImpactVfx` remains available for old one-frame contact setups.
## 2026-07-02 Update: Ground Scrape Spark NotifyState Implemented

- `AnimNotifyState_GroundScrapeVfx` now exists at `Source/MyProject/Combat/AnimNotifyState_GroundScrapeVfx.h/.cpp`.
- It is a presentation-only animation notify state for weapon-ground scrape sparks.
- The default Niagara system is `/Game/MechanicalDamageFX/FX/NS_Sparks_Small`.
- While the notify-state window is active, it repeatedly traces downward from a Blueprint trace-point component and spawns small sparks at ground hit points.
- The default trace-point setup is `TracePointComponentName=WeaponVfx_GroundContact`. If missing, the notify can create a runtime-only trace point under `EquippedOtachi` with relative location `115, 0, 2`. Old `BladeTipTracePoint` settings are supported as a compatibility fallback, but `WeaponVfx_GroundContact` is preferred when it exists. Deprecated socket fallback remains only if the trace point name is set to None.
- Do not auto-edit `/Game/Weapon/Katana/Meshes/StaticMeshes/SM_Otachi_1` for ground spark sockets. That automated socket test caused Static Mesh Editor crashes and was reverted.
- The old owner-local start/end offset path and direct static-mesh socket editing path are deprecated for this notify state because they did not follow the blade safely.
- This does not apply damage, shockwave gameplay, Lightning damage, hit reaction, or camera shake.
- The notify-state still needs editor-side placement over the frames where the weapon visually drags across the ground.## 2026-07-02 Update: RoomEncounterDataAsset First Implementation

- `URoomEncounterDataAsset` now exists at `Source/MyProject/Rooms/RoomEncounterDataAsset.h/.cpp`.
- `RoomCombatActor.EncounterData` can now provide structured encounter spawn data before the older direct RoomActor fields.
- `RoomEncounterDataAsset.InitialSpawnEntries` expands into enemies spawned together when Room combat starts.
- `RoomEncounterDataAsset.WaveSpawnEntries` expands into the existing delayed sequential queue after tracked enemies are defeated.
- `RoomEncounterDataAsset.SpawnDelay` owns the DataAsset-driven delay between wave entries.
- `RoomEncounterDataAsset.PlacedEnemyActivationPolicy` can activate placed enemies on Room start or keep them dormant until player-detection range.
- `RoomEncounterDataAsset.ClearCondition` is prepared for all-tracked, spawned-only, or placed-only Room clear checks.
- `RoomCombatActor.PlacedEnemyActors` is the editor list for enemies already placed in the level.
- Existing direct fields remain fallback: `InitialSpawnEnemyClasses`, `SpawnEnemyClasses`, `SpawnDelayBetweenEnemies`, and `EnemyClass + EnemiesToSpawn`.
- `EnemyBase` now exposes `EnemyActivationState`, `ActivateEnemy()`, `DeactivateEnemy()`, and `IsAlive()` for spawned and placed enemy workflows.


## 2026-07-01 Update: Weapon Trail AnimNotify Timing Cleanup

- Formal player weapon-trail timing now belongs to `AnimNotifyState_WeaponTrail` windows on the attack animation assets.
- `WeaponTrailComponent.ComboTrailSlots` remain responsible for Trail System, attach component/socket, relative location, relative rotation, and relative scale.
- `ActivationDelay` and `Duration` remain as fallback values only and are displayed in the editor as `Fallback Activation Delay` and `Fallback Duration`.
- If `bUseAnimNotifyTrailTiming` is enabled, attack-start fallback playback is skipped and AnimNotify windows control trail start/stop.
- If `bUseAnimNotifyTrailTiming` is disabled, `AnimNotifyState_WeaponTrail` ignores notify playback so the fallback path can be tested without double-triggering trails.
## 2026-07-01 Update: Run / Stage Planning Boundary

- The current playable implementation is still a single-map Room chain MVP driven mainly by `RoomCombatActor`.
- The current room chain should be interpreted as one Stage's internal Room progression, not as the full Run structure.
- The long-term Run structure is `Stage Map -> choose Stage -> clear Rooms inside Stage -> Stage Clear -> reward card selection -> apply reward -> return to Stage Map -> choose next Stage`.
- Stage Map UI, Stage Node data, route/path selection, return to Stage Map, Mid Boss Stage, Boss Stage, Final Boss, and Run Clear are not implemented yet.
- The long-term reward cadence is Stage Clear, not every Room Clear.
- The planned Stage Clear order is reward selection first, then Stage Map return, then connected next Stage selection.
- `StageFlowManager` disables Room Clear rewards for manager-owned Rooms and uses the existing `RewardActor` / `RewardSelectionWidget` flow at Stage Clear.
- The current last-room restart overlay is an MVP fallback shown after the Stage Clear reward is claimed, not the final Stage Map or Run Clear UI.
- Detailed planning source of truth: `Docs/StageRunStructure.md`.

## 2026-07-01 Update: StageFlowManager First MVP Plan

- The first implementation should move only Room sequencing and Stage Clear fallback out of `RoomCombatActor`.
- First MVP scope: one active Stage id, ordered Rooms, current Room index, next-Room activation, and temporary Stage Clear fallback.
- It should not own Stage Map UI, next Stage selection, final reward data/grade/pool generation, Luck stat, technique levels, Mid Boss Stage, Boss Stage, Final Boss, or Run Clear yet.
- `RoomCombatActor` should keep local Room combat: player entry, doors/boundaries, enemy spawning, alive enemy tracking, Room Clear detection, optional standalone MVP RewardActor flow, and a progression-ready handoff signal.
- Existing `RoomCombatActor` fields such as `bAutoFindNextRoomByOrder`, `NextRoomActors`, and the last-room fallback should remain until the manager path is verified in editor.

## 2026-07-01 Update: StageFlowManager Stage Progression Implemented

- `AStageFlowManager` now exists at `Source/MyProject/Rooms/StageFlowManager.h/.cpp`.
- `RoomCombatActor` now exposes `OnRoomProgressionReady`.
- `RoomCombatActor.CompleteRoomProgression()` broadcasts `OnRoomProgressionReady` after Room Clear, reward wait, progression delay, and local door/boundary unlock.
- `StageFlowManager` can auto-find `RoomCombatActor` instances with the same `StageId`, sort them by `RoomOrder`, bind to their progression-ready signal, log the received signal, compare the old handoff path, and activate the next Room.
- `StageFlowManager` now disables `RoomCombatActor` legacy progression handoff and Room Clear rewards for registered Rooms.
- `StageFlowManager` now owns the temporary final Room -> Stage Clear reward and restart fallback until Stage Map exists. `NextRoomActors` and `bAutoFindNextRoomByOrder` remain fallback/setup data.

## 2026-07-01 Update: Next Room Opening Rule

- Long-term rule: defeating all tracked monsters in the current Room clears the Room and opens the path to the next Room.
- Long-term rule: Room Clear should not grant a build reward card.
- Current rule: build reward cards are granted at Stage Clear for manager-owned Stages.
- Long-term rule: Stage Clear reward selection happens before returning to the Stage Map.
- Long-term rule: next Stage candidates become selectable after the selected Stage Clear reward is applied.
- Current exception: the old Room Clear reward card flow remains available only for standalone rooms not owned by `StageFlowManager`.
- `StageFlowManager` disables Room Clear rewards at runtime for registered Rooms and opens the existing reward card UI at Stage Clear.

## 2026-07-01 Update: NextRoomActors / Auto-Link Deprecation Plan

- `NextRoomActors` and `bAutoFindNextRoomByOrder` are not deprecated yet.
- They remain as fallback/setup data while `StageFlowManager` owns the primary next-Room handoff in the current Stage test.
- Gate 0: confirm `StageFlowManager received room progression-ready` in Output Log.
- Gate 1: let `StageFlowManager` calculate the next Room candidate without activating it. Implemented as a validation log.
- Gate 2: verify that the manager candidate matches the current `RoomCombatActor` handoff. Implemented as a validation log.
- Gate 3: let `StageFlowManager` activate the next Room. Implemented; `RoomCombatActor` legacy handoff is disabled for registered manager-owned Rooms.
- Gate 4: let `StageFlowManager` handle final Room -> Stage Clear. Temporary Stage Clear reward and restart fallback are now manager-owned until Stage Map exists.
- Do not remove the existing UPROPERTY fields abruptly because placed rooms or Blueprints may still reference them.

## 2026-07-01 Update: StageFlowManager Next Implementation Units

- Step A now calculates and logs the next Room candidate from `StageId` and `RoomOrder + 1`.
- Step B now compares the manager candidate with the existing `RoomCombatActor` handoff path and logs match/mismatch.
- Step A and Step B are validation logs and must not activate Rooms or alter playable progression.
- Step C now moves next-Room activation into `StageFlowManager`.
- Step D temporary final Room -> Stage Clear reward and restart fallback are also manager-owned while Stage Map is not implemented.
- `RoomCombatActor.NextRoomActors` and `bAutoFindNextRoomByOrder` remain as fallback/setup data, not the active opener in manager-owned Stages.

## 2026-06-30 Update: WeaponTrailComponent Structure Prep

- `WeaponTrailComponent` was added as a separate player weapon-trail presentation component.
- The old combo slash VFX path remains removed. `PlayerStatsDataAsset` and `PlayerStatsComponent` still do not own attack VFX assets or VFX offsets.
- `PlayerMeleeAttackSubsystem` now ensures the player pawn has a `WeaponTrailComponent` and calls `PlayAttackTrail(ComboStep)` when a basic combo attack starts.
- `WeaponTrailComponent` has three prepared combo trail slots for attack 1, attack 2, and attack 3.
- Default playback is disabled through `bEnableWeaponTrailPlayback=false`, and slots have no Niagara asset by default. This keeps the current game visually unchanged until editor tuning.
- `AnimNotifyState_WeaponTrail` was added so attack animations can later own the exact weapon-trail on/off window.
- `WeaponTrailComponent.bUseAnimNotifyTrailTiming` can disable the attack-start fallback trail and let the notify state control trail timing.
- `RewardSelectionWidget` notifies `WeaponTrailComponent` when a reward card is selected.
- The current implemented visual-growth hook only records reward IDs and lets `BasicAttackDamage` Run rewards increase a runtime trail intensity/scale value.
- `PlayerHealthSubsystem` resets the runtime weapon-trail visual state on player death, matching the existing Run reward reset rule.
- Actual Niagara assignment, sword socket alignment, animation notify placement on attack assets, projectile sword waves, and reward-specific VFX profiles are not implemented yet.

## 2026-06-29 Update: 기본 공격 콤보 VFX ?�거

- ?�전 ?�레?�어 기본 공격 콤보 slash VFX 경로???�재 구현 기�??�서 ?�거?�다.
- `PlayerMeleeAttackSubsystem`?� ???�상 좌클�?1/2/3?� 공격 �?Niagara slash VFX�??�성?��? ?�는??
- `PlayerStatsDataAsset`�?`PlayerStatsComponent`?????�상 콤보 slash VFX ?�셋, VFX Point fallback offset, delay, scale, lifetime ?�드�??�공?��? ?�는??
- ?�재 기본 공격?� ?�니메이?? ?�진 ?�텝, 부채꼴 ?��?지 ?�정, 공격 범위 ?�버�??�시�??��??�다.
- 추후 무기 ?�레?? 검�? 검�?보상 ?�출?� ???�계/구현 경로�??�룬??

## 2026-06-28 Update: Tank Butcher AnimBP Attack Path

- `BP_Enemy_Tank.AttackAnimation` now uses `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Weak_Attack1`.
- `BP_Enemy_Tank.StrongAttackAnimation` now uses `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Strong_Attack3`.
- The weak Montage is generated from `/Game/Demons_Big_Pack/Butcher/Animation/Anim_Butcher_attack1`.
- The strong Montage is generated from `/Game/Demons_Big_Pack/Butcher/Animation/Anim_Butcher_attack3`.
- `BP_Enemy_Tank.StrongAttackEveryNthAttack` is set to `2`, so the Tank alternates weak attack1 and strong attack3 on successful melee attacks.
- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/ABP_Butcher` as its dedicated AnimBP.
- `ABP_Butcher` uses the Butcher skeleton, `Anim_Butcher_idle1`, `Anim_Butcher_walk2`, and a `DefaultSlot` for attack playback.
- Tank attack playback now uses explicit AnimMontage assets through the `EnemyBase` AnimBP/Montage path instead of the SingleNode `PlayAnimation` fallback.
- `EnemyBase` now supports an optional strong attack slot through `StrongAttackAnimation` and `StrongAttackEveryNthAttack`.
- `EnemyBase.AttackAnimationLockDurationScale` controls how long attack presentation locks movement/facing after an attack animation starts.
- The default value is `1.0`, preserving existing enemy behavior.
- `BP_Enemy_Tank` currently uses `AttackAnimationLockDurationScale = 0.9` so chase/run can resume slightly before the full Butcher attack1 tail finishes.
- `EnemyBase.AttackAnimationPlayRate` controls per-enemy attack animation playback speed.
- The default value is `1.0`, preserving existing enemy behavior.
- `BP_Enemy_Tank` currently uses `AttackAnimationPlayRate = 0.75`, so Butcher weak/strong attacks read as heavier Tank attacks without making attack3 as slow as the earlier long-lock test.
- `EnemyBase.PostAttackTurnToPlayerDuration` adds an optional short post-attack preparation turn before chase resumes.
- The default value is `0.0`, preserving existing enemy behavior.
- `BP_Enemy_Tank` currently uses `PostAttackTurnToPlayerDuration = 0.7` so the Tank does not snap directly from attack recovery into chase-facing rotation.
- Tank weak/strong Montages currently use `0.28` seconds of BlendOut to soften the return from attack Slot to locomotion.
- `ABP_Butcher` currently uses `Anim_Butcher_walk2` at `0.9` speed for Tank movement because `Anim_Butcher_run1` and the earlier `walk1` setup produced visible loop stutter during chase.
- `/Game/Blueprints/Enemy/Tank/BS_Butcher_Tank_Locomotion` has been created as a prepared Tank locomotion BlendSpace using idle/walk/heavy-walk samples.
- Unreal Python can configure the BlendSpace asset but cannot automatically replace the existing AnimGraph `SequencePlayer` locomotion nodes with a `BlendSpacePlayer`, so the ABP graph connection is still an editor-side step.
- `EnemyAnimInstance` uses separate start/stop movement speed thresholds plus a short stop delay, so small velocity dips do not repeatedly cut the Tank run animation back to idle.
- `EnemySimpleAnimationComponent` is no longer the Tank idle/run driver; it remains the fallback for enemies without a dedicated AnimBP.
- The previous Tank attack issue was caused by using an attack animation that did not match the current Tank/Butcher visual setup, then by the SingleNode path cutting directly between locomotion and attack.
- Tank still reuses the existing `EnemyBase` melee attack timing, facing lock, and soft separation logic.
- This changes shared enemy presentation only enough to support a configured strong attack and optional post-attack turn; room spawning flow and reward flow were not changed.
- Other Butcher attack candidates exist as `Anim_Butcher_attack1` through `Anim_Butcher_attack8`, but random attack selection remains out of the MVP until each asset is tested for clean transitions.
- Actual attack1-to-attack3 feel still needs in-editor visual review. If the transition still feels cut, the next formal step is a single combo Montage with sections and later AnimNotify-based damage timing.

## 2026-07-01 Update: Enemy White Hit Flash Removed

- The enemy white hit-flash MVP has been removed from the current implementation direction.
- `EnemyBase` no longer listens to `HealthComponent.OnHealthChanged` only to trigger a white overlay flash.
- The exposed hit-flash settings `bEnableHitFlash`, `HitFlashOverlayMaterial`, `HitFlashColor`, `HitFlashStrength`, `HitFlashDelay`, and `HitFlashDuration` are no longer current implementation 기�?.
- Enemy damage, death events, attack animation playback, spawn intro logic, movement, and room clear tracking remain unchanged.
- Little Demon, Fast, and Tank should not use a shared white hit-flash reaction in the current MVP.
- If hit feedback is needed later, redesign it as a separate presentation feature such as weapon impact VFX, sound, animation reaction, camera feedback, or enemy-specific effects.

## 2026-06-27 Update: Stage 1 Room Auto-Link MVP

- `RoomCombatActor` now has Stage MVP fields: `StageId`, `ExpectedStageRoomCount`, `bAutoFindNextRoomByOrder`, and `bShowStageRoomProgressInDebugMessages`.
- If `NextRoomActors` does not contain a valid actor, a room can automatically find the next `RoomCombatActor` in the same `StageId` with `RoomOrder + 1`.
- This lets Stage 1 test rooms be placed as `RoomOrder` `0~6` or `1~7` without manually wiring every `NextRoomActors` reference.
- If the same Stage has a room with `RoomOrder = 0`, the auto-link helper treats the stage as zero-based and displays progress as `1/7` through `7/7`.
- Existing manual `NextRoomActors` links still take priority.
- The final room is handled by `StageFlowManager`: no valid next room means Stage Clear reward, then the temporary restart overlay.
- This is not a full Run manager. Stage branching, next Stage selection UI, reward grade rolls, technique level tracking, and Luck stat remain deferred.

## 2026-06-27 Update: Stage/Reward Planning Not Yet Implemented

- Planning direction now treats one Stage as a set of multiple Rooms. Stage 1 is currently planned around a 7-Room structure, with the last Room or a separate boss Room ending the Stage.
- Planning direction also includes Stage Clear -> next Stage candidate selection, where choices can differ by enemy composition, danger level, reward tendency, recovery chance, and special conditions.
- Planning direction separates reward grade from technique level: grade describes card quality, while technique level describes accumulated growth on a skill/technique axis.
- Bronze/Silver/Gold/Prism are candidate grade names, not final enum names.
- Luck is a candidate future stat that may affect high-grade reward appearance, especially Prism-style rare rewards.
- Stage 1 room auto-link remains as `RoomCombatActor` fallback/setup data, but current manager-owned Stages use `StageFlowManager` for next-room handoff, Stage Clear reward timing, and the temporary restart fallback.
- `StageFlowManager`, next Stage selection UI, real reward grade rolls, technique level tracking, Luck stat, and Prism reward logic remain deferred.

## 2026-06-26 Update: Reward Design Direction Not Yet Implemented

- Stage 1 reward direction has been adopted in planning docs: short combat rooms, changing reward card candidates per cleared room, and reward-grade tiers. Bronze/Silver/Gold/Prism are current candidate names.
- Higher-tier reward candidates now favor build-changing combat effects such as 3rd-hit sword wave, 3rd-hit damage/range increase, 3rd-hit kill heal, 3rd-hit bleed, dodge-followup attack boost, and afterimage explosion.
- This is not implemented yet. Current runtime reward code still uses the existing MVP card options and `Instant`/`Run`/`Permanent` apply type structure.
- `RewardDataAsset`, rarity weights, reward pool tables, complex synergy, save/load, currency, and permanent upgrade UI remain deferred.

## 2026-06-24 Update: RoomCombatActor Editor Setting Cleanup

- `RoomCombatActor` now has editor-facing room identity fields: `RoomId`, `RoomOrder`, and `RoomDisplayName`.
- Debug screen messages can be toggled per room with `bShowRoomDebugMessages`.
- Room message durations are exposed through `DefaultDebugMessageDuration`, `RoomClearedMessageDuration`, and `ProgressionDebugMessageDuration`.
- Spawn-position debug preview duration is exposed through `SpawnPreviewDuration`.
- `ProgressionReadyDelay` can optionally delay next-room opening or stage-clear handling after reward selection. The default is `0.0`, so the current MVP flow remains unchanged.
- `NextRoomActors` validity checks now use `IsValid` so destroyed or invalid actors do not count as real next-room links.

## 2026-06-24 Update: Current MVP Snapshot

This pass is a small data-structure preparation step for rewards, room settings, and future stage/encounter separation.

Current playable MVP loop:

1. Player enters a `RoomCombatActor` trigger.
2. The room locks doors/boundaries and starts combat.
3. Room enemies spawn through `InitialSpawnEnemyClasses`, `SpawnEnemyClasses`, or the older `EnemyClass + EnemiesToSpawn` fallback.
4. `EnemyBase` enemies chase, separate from each other, attack, and report death through `HealthComponent`.
5. When all tracked enemies are dead, manager-owned Rooms emit progression-ready without spawning a Room Clear reward.
6. The player touches the reward pickup and chooses one card in `RewardSelectionWidget`.
7. The room unlocks progression after the reward is claimed.
8. If `NextRoomActors` exist or the Stage MVP auto-link finds the next room by `RoomOrder`, the next room path opens.
9. If there are no valid `NextRoomActors`, the MVP treats the stage as cleared and shows a restart button.

Implemented MVP systems:

- Player control: WASD movement, left-click 1/2/3 combo, Space dodge, intro weapon socket flow.
- Player health/UI: `HealthComponent`, `PlayerHealthSubsystem`, circular `PlayerHealthOrbWidget`.
- Room flow: `RoomCombatActor` trigger, room boundary lock, door actor support, progression-ready signal, and `StageFlowManager` next-room activation by `StageId + RoomOrder`.
- Enemy flow: `EnemyBase`, `EnemyStatsDataAsset`, Little Demon, Fast, Tank, spawn intro support, attack1 playback, soft enemy separation.
- Stage rewards: `StageFlowManager` opens `RewardActor` / `RewardSelectionWidget` at Stage Clear. `ERewardApplyType` supports `Instant`, `Run`, `Permanent`.
- Run rewards: runtime-only max health, attack damage, move speed, and dodge cooldown modifiers on `PlayerStatsComponent`.

Not final / deferred:

- Full Run/Stage Map ownership is still later work. The current `StageFlowManager` owns one active Stage room chain, Stage Clear reward timing, and temporary restart fallback only.
- `RoomEncounterDataAsset` is still a later structure for reusable enemy waves, room metadata, clear rules, and special-room reward hooks if needed.
- `RewardDataAsset`, rarity, weights, reward pools, equipment, skill synergy, save/load, lobby upgrades, and currency are intentionally not implemented yet.
- Permanent rewards are placeholder-only and should not change save data yet.
- Fast now uses the DemonHeavy mesh with dedicated `ABP_DemonHeavy_Fast` and `AM_DemonHeavy_Fast_Attack1` Montage playback. Tank currently uses the modified Butcher mesh and dedicated `ABP_Butcher` path.
- Stage clear UI and reward UI are functional MVP Slate widgets, not final UI design.
- Enemy animation polish, hit reactions, death cleanup, boss rooms, elite rooms, and final room variety are still future work.

## 2026-06-24 Update: Data Structure Preparation

- `FRewardCardOption` now has `RewardId` as a stable temporary identity for each reward card.
- Current default cards assign ids such as `instant_heal_20`, `run_attack_damage_5`, and `run_move_speed_80`.
- `RewardId` prepares the path for a future `RewardDataAsset` or reward table, but no reward pool, rarity, weight, save data, or currency system has been added.
- `RoomCombatActor` editor-facing settings are grouped by responsibility: stage MVP, activation, encounter fallback, initial wave, sequential queue, spawn points, reward, progression, doors, and boundary.
- `StageFlowManager` now owns the current Stage room chain, Stage Clear reward timing, and temporary restart fallback. It does not own Stage Map branching yet.
- `RoomEncounterDataAsset` now exists as the first reusable Room encounter data class. It is still early and should be introduced gradually, while direct `RoomCombatActor` fields remain fallback/editor override data.

Future split direction:

- `RoomCombatActor`: keep room trigger, local combat lock, local enemy tracking, room clear signal, and optional standalone reward spawn for tests.
- `StageFlowManager`: own current Stage room order, next-room activation, final clear reward timing, and temporary restart/menu flow; later add Stage Map branching.
- `RoomEncounterDataAsset`: later own reusable enemy lists, wave timing, spawn rules, room tags, clear rules, and special-room reward hooks if needed.
- `RewardDataAsset` or reward table: later own reward definitions after card rules stabilize.

Safe next tasks without visual editor access:

- Review and clean debug/on-screen messages.
- Consolidate documentation around room/reward/enemy ownership.
- Add small compile-safe guards or logs.
- Sketch `RoomEncounterDataAsset` fields in docs only.
- Review code paths for restart, reward selection, and run reward reset behavior.

Tasks that should wait for editor/play testing:

- UI layout polish and final card visuals.
- Collision/boundary feel.
- Enemy animation feel and mesh offsets.
- Spawn point placement and room spacing.
- Door visuals and reward pickup visuals.

## 2026-06-24 Update: Code Safety Stabilization Pass

- `RoomCombatActor` now removes invalid, destroyed, dead, or untrackable enemies from its alive-enemy list before deciding room clear.
- If a spawned room enemy has no `HealthComponent`, it is logged, destroyed, and not tracked. This prevents a room from staying uncleared forever because of an unreportable enemy.
- `RewardActor` now removes an active reward selection widget during `EndPlay`, so level restart or actor destruction does not leave a stale reward UI reference.
- `RewardActor` no longer blocks the reward UI only because the player health component is missing at overlap time. Health-specific reward effects still validate `HealthComponent` when the card is actually selected.
- `RewardSelectionWidget` skips reward application if the player actor is missing or the reward value is non-finite.
- `PlayerStatsComponent` ignores non-finite run reward bonuses so temporary runtime stat modifiers cannot be polluted by invalid values.
- `PlayerHealthSubsystem` re-checks `PlayerStatsComponent` even when the same pawn and health component are cached, keeping reward stat lookup safer.

## 2026-06-23 Update: Stage Clear Restart Button MVP

- When the last room is cleared and `RoomCombatActor` has no valid `NextRoomActors`, the stage is treated as cleared.
- The MVP now shows a simple `?�테?��? ?�리?? overlay with a `?�시 ?�작` button.
- Pressing the button reloads the current level with `UGameplayStatics::OpenLevel`, so room state, spawned enemies, rewards, and runtime run rewards reset with the level.
- The restart button clears current viewport widgets before reloading so the temporary stage-clear overlay does not remain after restart.
- This is a temporary MVP restart path, not a final stage-clear menu or game-over flow.

## 2026-06-23 Update: Korean Reward UI Text

- The reward card MVP now displays Korean UI text for card titles, card descriptions, reward apply type labels, effect values, and reward selection debug messages.
- This is only a text/localization pass; reward categories, reward effects, room progression, and reward selection flow were not changed.

## 2026-07-04 Update: Fast DemonHeavy AnimBP Path

- `BP_Enemy_Fast` now uses `/Game/Demons_Big_Pack/DemonHeavy/Mesh/SK_DemonHeavy`.
- Fast idle/run are now driven by `/Game/Blueprints/Enemy/Fast/ABP_DemonHeavy_Fast` using `Anim_DemonHeavy_idle1` and `Anim_DemonHeavy_run1`.
- Fast attack presentation now uses `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Attack1`, generated from `Anim_DemonHeavy_attack1`, through the `EnemyBase` AnimBP/Montage path. Death presentation remains configured with `Anim_DemonHeavy_death1`.
- The current Fast Montage uses `DefaultSlot`, `BlendIn=0.08`, `BlendOut=0.14`, and `AttackAnimationPlayRate=2.0`.
- Fast strong attack now uses `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Strong_Attack3`, generated from `Anim_DemonHeavy_attack3`; `StrongAttackEveryNthAttack=3`.
- `EnemyBase` now exposes `StrongAttackDamageMultiplier` and `StrongAttackRangeMultiplier`; `BP_Enemy_Fast` currently uses `1.4` damage and `1.15` range for its strong attack.
- This is a Blueprint/animation asset connection only; `EnemyBase`, room spawning, Stage flow, and reward flow were not changed.
## 2026-06-23 Update: Fast/Tank Mesh Orientation Fix

- `BP_Enemy_Fast` previously used a Manny placeholder mesh with the standard character mesh relative rotation `(Pitch=0, Yaw=-90, Roll=0)`.
- `BP_Enemy_Tank` has since moved to the modified Butcher mesh and `ABP_Butcher` path.
- They previously used `Pitch=-90`, which made the enemies appear rotated sideways and partially sunk into the ground while moving.
- This was a Blueprint mesh transform fix only; `EnemyBase` movement and room spawning logic were not changed.

## 2026-06-23 Update: Reward Apply Type MVP

- Reward cards now separate reward behavior with `ERewardApplyType`: `Instant`, `Run`, and `Permanent`.
- `Heal +20` is an `Instant` reward and applies immediately through `HealthComponent::Heal`.
- `Max Health +10` is treated as a `Run` reward, not as a `PlayerStatsDataAsset` edit.
- Test run rewards now include attack damage up, move speed up, and dodge cooldown down.
- Run rewards are stored only as transient runtime modifiers on `PlayerStatsComponent`.
- `PlayerHealthSubsystem` resets run reward modifiers when the player dies; level restart also resets them because components are recreated.
- `Permanent` reward type exists only as a placeholder path for later meta progression and currently logs/TODOs instead of saving data.
- No `RewardDataAsset`, rarity, weighted pool, equipment, skill, synergy, save/load, lobby upgrade UI, or currency system has been added yet.
- The reward card UI still starts with `2` visible cards. The first card remains the heal test, and the second card rotates through the other temporary reward candidates for MVP testing.

## 2026-06-23 Update: Fast/Tank Enemy MVP and Initial Room Wave

- `RoomCombatActor` now supports `InitialSpawnEnemyClasses`.
- `InitialSpawnEnemyClasses` spawns all listed `EnemyBase` subclasses together when room combat starts.
- If `InitialSpawnEnemyClasses` is empty, the existing `SpawnEnemyClasses` sequential queue still works as before.
- Room1 currently uses `InitialSpawnEnemyClasses` for a first test wave containing `BP_Enemy_Fast` and `BP_Enemy_Tank`.
- Room1's sequential `SpawnEnemyClasses` queue is currently empty so the first test focuses on the two new enemy types.
- `BP_Enemy_Fast` and `BP_Enemy_Tank` are temporary MVP enemies based on the existing `EnemyBase` flow.
- Fast now uses the DemonHeavy dedicated AnimBP/Montage setup; Tank uses the modified Butcher setup. Both keep separate `EnemyStatsDataAsset` values.
- Final enemy theme, models, animations, and exact balance are not decided yet.

## 2026-06-22 Update: Room Enemy Class Array

- `RoomCombatActor` now has `SpawnEnemyClasses`.
- When the array is filled, room combat spawns one listed `EnemyBase` subclass at a time in order.
- The next enemy in the array is spawned only after the current tracked enemy dies and `SpawnDelayBetweenEnemies` has elapsed.
- `SpawnDelayBetweenEnemies` is currently set to `0.5` seconds as a temporary test value.
- The array mode spawns one enemy per entry, so `EnemiesToSpawn` is only used by the fallback mode.
- If `SpawnEnemyClasses` is empty, the existing `EnemyClass` plus `EnemiesToSpawn` behavior is unchanged.
- This allows Room1, Room2, and later test rooms to mix enemy classes without introducing `RoomEncounterDataAsset` yet.

## 2026-06-22 Update: Room Encounter Setting Direction

- Current phase uses per-room direct settings on each `RoomCombatActor`.
- For example, Room1 and Room2 can each set their own `EnemyClass`, `EnemiesToSpawn`, `SpawnPoints`, `RewardSpawnPoint`, and `NextRoomActors` in the editor.
- This is the current Stage 1 room-setting method and is valid for the MVP.
- Later, when room count, enemy variety, waves, or reward rules grow, the plan is to move encounter definitions into `RoomEncounterDataAsset`.
- `RoomEncounterDataAsset` should become the Stage 2 data-driven structure for enemy groups, wave timing, room metadata, and special room rules. Main build rewards should move toward Stage Clear reward data instead of per-room reward candidates.
- The current direct `RoomCombatActor` settings can remain as a fallback for quick test rooms even after the Data Asset structure exists.

## 2026-06-22 Update: Stage Clear Fallback MVP

- `RoomCombatActor` now treats a cleared room with valid `NextRoomActors` as a next-room handoff.
- If a room has no valid `NextRoomActors`, room progression now prints `Stage Cleared` instead of `Next Room Open`.
- This keeps the two-room test extensible: Room1 can point to Room2, Room2 can end the stage, and later Room2 can point to Room3 without changing the core flow.
- This is still an MVP fallback, not the final `StageFlowManager` structure.

## 2026-06-21 Update: Next Room Progression MVP

- `RoomCombatActor` now has a room progression-ready state after room clear.
- If a standalone Room Clear reward is spawned, doors and auto boundary blockers wait until the player chooses a reward card. In manager-owned Stages, Room Clear rewards are disabled and reward selection happens at Stage Clear.
- After Room progression-ready, `StageFlowManager` opens the next Room. On the final Room, it opens the Stage Clear reward and then the temporary restart fallback.
- `NextRoomActors` are disabled at BeginPlay and enabled only when room progression becomes ready.
- To test two rooms, place a second `BP_RoomCombatActor` in the level and add it to the first room's `NextRoomActors` array.
- If no reward is spawned, the room opens immediately after clear as a fallback.
- This is the current MVP handoff from one cleared room to the next room area.

## 2026-06-21 Update: Reward Card Selection MVP

- `RewardActor` now opens a reward card selection UI instead of immediately applying `Heal +20`.
- The current MVP starts with `2` visible reward cards.
- The reward card count is designed to support up to `4` cards through `RewardActor.MaxRewardCardCount`.
- The first test cards are `Heal +20` and `Max Health +10`.
- Choosing a card applies the selected effect, closes the UI, restores game input, and removes the reward pickup.
- This is not a slot-machine animation flow. It is a direct card choice UI.

## 2026-06-21 Update: Room Reward Pickup MVP

- `RoomCombatActor` can still spawn a room-clear reward for standalone tests, but manager-owned Stages disable that path at runtime.
- The current reward class is `RewardActor`.
- `RewardActor` is a temporary pickup actor: when the player overlaps it, it opens the reward card selection UI.
- Reward effects are applied only after the player chooses a card.
- `RoomCombatActor.RewardSpawnPoint` can be assigned in the editor. If it is empty, the reward spawns near the `RoomCombatActor` using `RewardFallbackOffset`.
- This is only the first reward pipeline test. Final reward categories, selection UI, item synergy, and permanent progression are not decided yet.

## 2026-06-21 Update: Little Demon Spawn Intro

- `EnemyBase` now supports an optional `SpawnIntroAnimation`.
- While the spawn intro is playing, the enemy does not chase or attack the player.
- After the spawn intro animation finishes, the enemy returns to the existing idle/run chase and melee attack flow.
- `BP_Enemy_LittleDemon.SpawnIntroAnimation` is set to `/Game/Demons_Big_Pack/Little_Demon/Animation/Anim_Little_Demon_rage`.
- `BP_Enemy_LittleDemon.SpawnIntroMeshRelativeOffset` is currently `(0, 0, -50)` to keep the rage intro grounded, then the mesh location is restored after the intro.
- `BP_Enemy_LittleDemon.SpawnIntroFollowupAnimation` is set to `/Game/Demons_Big_Pack/Little_Demon/Animation/Anim_Little_Demon_idle2` for a 1.0 second pause after rage.
- `BP_Enemy_LittleDemon.SpawnIntroTurnToPlayerDuration` is set to `0.45` seconds, so the enemy turns toward the player smoothly before chase starts.
- This is the current MVP way to make newly spawned Little Demons present themselves before chasing the player.

## 2026-06-20 Update: Debug Slow Motion Toggle

- `PlayerKeyboardMovementSubsystem` now handles an `F10` debug toggle for global time dilation.
- Pressing `F10` switches gameplay to `0.1x`; pressing `F10` again restores `1.0x`.
- This is intended for visual animation testing, especially enemy attack transition checks.

## 2026-06-20 Update: Enemy Attack Animation State

- `EnemyAnimInstance` now exposes `bIsAttacking`.
- `EnemyBase` sets `bIsAttacking=true` when attack animation playback starts and clears it when attack playback ends or the enemy dies.
- While `bIsAttacking` is true, `EnemyAnimInstance` keeps `bIsMoving=false` so the locomotion base pose does not flip between idle/run under the attack Slot.
- `BP_Enemy_LittleDemon` has been moved back to the SingleNode/SimpleAnimation path for the MVP because the temporary AnimBP + runtime Dynamic Montage path produced visible T-pose/flicker artifacts between attacks.
- Attack animation presentation duration is now based on the played animation duration only; attack cooldown remains a separate gameplay timer through `AttackCooldownRemaining`.

## 2026-06-20 Update: Little Demon Attack Animation Path

- `BP_Enemy_LittleDemon.AttackAnimation` points to the original `/Game/Demons_Big_Pack/Little_Demon/Animation/Anim_Little_Demon_attack1` sequence.
- `BP_Enemy_LittleDemon` currently uses `AnimationSingleNode` plus `EnemySimpleAnimationComponent` for idle/run and direct sequence playback for attack1.
- `BP_Enemy_LittleDemon.Mesh.BoundsScale` is set to `8.0`, `ComponentUseFixedSkelBounds=true`, and `VisibilityBasedAnimTickOption` is set to always refresh bones to avoid attack-pose culling/flicker during MVP tests.
- `EnemyBase` wraps sequence attack assets with `PlaySlotAnimationAsDynamicMontage()` only when the mesh is using a valid Animation Blueprint and Slot.
- If no valid AnimBP is active, `EnemyBase` now directly plays the configured sequence instead of creating a transient Dynamic Montage.
- The temporary `/Game/Blueprints/Enemy/LittleDemon/AM_LittleDemon_Attack1` asset was removed because the current MVP path should avoid malformed or stale Montage assets.
- `/Game/Blueprints/Enemy/LittleDemon/ABP_LittleDemon` still exists as a transition asset, but it is not the current Little Demon runtime path.
- The next animation refinement is adding a dedicated death state and later AnimNotify-based attack damage timing.

## 2026-06-19 Update: Little Demon AnimBP Transition Start

- `EnemyAnimInstance` is now the shared C++ parent class for enemy Animation Blueprints.
- It exposes `GroundSpeed`, `bIsMoving`, and `bIsDead` for AnimBP graphs.
- `/Game/Blueprints/Enemy/LittleDemon/ABP_LittleDemon` has been created for `SK_Little_Demon_Skeleton` with `EnemyAnimInstance` as parent.
- The AnimBP graph is wired for idle/run locomotion plus `DefaultSlot` attack playback.
- Historical note: `ABP_LittleDemon` was created as a transition asset, but the current Little Demon MVP runtime path later moved back to SingleNode/SimpleAnimation for stability.
- `EnemySimpleAnimationComponent` remains the current Little Demon idle/run driver unless a stable dedicated AnimBP is deliberately reassigned later.

## 2026-06-19 Update: Enemy Attack Facing Stabilization

- While an enemy attack animation is playing, `EnemyBase` now keeps the attack-start facing rotation instead of continuously re-facing the player every tick.
- Attack-time enemy separation no longer uses `AddMovementInput`; it applies a small swept world offset so separation does not fight `bOrientRotationToMovement`.
- During attack playback, `EnemyBase` temporarily disables movement-oriented rotation and forces attack animation root motion to `IgnoreRootMotion`, then restores the previous values when the attack ends.
- Little Demon attack playback has been simplified to a single `AttackAnimation` asset, currently attack1, to avoid spin-like presentation issues from mixed attack assets.
- Small animation hard cuts should be tested again through the new Little Demon AnimBP/Slot-based attack path.

## 2026-06-15 Update: Little Demon Dynamic Montage Attack Blend

- `EnemyBase` now plays attack animation assets through Dynamic Montage only when a valid Animation Blueprint class and Slot path are available.
- When an enemy has no dedicated AnimBP class, `EnemyBase` now directly plays sequence attack assets through the mesh SingleNode path.
- Direct `USkeletalMeshComponent::PlayAnimation` remains the final fallback, so existing enemies do not lose attack animation playback if Montage wrapping is unavailable.
- Attack transition tuning is exposed through `AttackAnimationSlotName`, `AttackAnimationBlendInTime`, and `AttackAnimationBlendOutTime`.
- `BP_Enemy_LittleDemon` currently uses the single sequence `AttackAnimation` rule through the SingleNode/SimpleAnimation path. The AnimBP/DefaultSlot path remains a later transition option.
- While an attack animation is playing, `EnemyBase` continues applying enemy separation movement so nearby enemies do not stack again around the player.
- Death blending is still the longer-term missing part of the Little Demon AnimBP.

## 2026-06-14 Update: Player Health Orb UI MVP

- `PlayerHealthSubsystem` now creates a runtime `PlayerHealthOrbWidget` for the local player.
- `PlayerHealthOrbWidget` displays the player's `HealthComponent` value as a bottom-left circular red health orb.
- The frame source image is `Content/UI/Textures/T_UI_HealthOrb_Frame.png`.
- The old `Player HP` on-screen debug text has been removed; the health orb is the current player health HUD.

## 2026-06-14 Update: Enemy Separation MVP

- `EnemyBase` now applies soft separation from nearby `EnemyBase` actors while chasing the player.
- This is used because Pawn collision remains overlap-only, so enemies should not physically block or push the player.
- The tunable values are `SeparationRadius`, `SeparationStrength`, and `MaxSeparationNeighbors` on `EnemyStatsDataAsset`.
- Separation is updated at a short interval and interpolated before use to reduce jitter when enemies partially overlap.
- Current room tests can use this to keep three Little Demon enemies from stacking into a single visual cluster.
- `DA_EnemyStats_LittleDemon.SeparationStrength` is currently reduced to `0.22` so enemies do not visibly push each other too strongly.
- Attack animation lock uses `AttackSeparationStrengthMultiplier=0.35`, so attack-time separation is much weaker than normal chase separation.

## 2026-06-14 Update: Little Demon Attack Animation MVP

- `EnemyBase` now exposes a single `AttackAnimation` asset slot.
- `BP_Enemy_LittleDemon` uses `/Game/Demons_Big_Pack/Little_Demon/Animation/Anim_Little_Demon_attack1` as its only current attack animation.
- The earlier random attack pool idea is removed from `EnemyBase` for the MVP.
- During the attack animation, `EnemySimpleAnimationComponent` is paused and the enemy will not start another melee attack until the attack animation lock finishes.
- `EnemyBase` turns to face the player immediately before the attack and keeps facing the player while the attack animation lock is active.
- Damage timing is still the existing distance/cooldown instant hit; animation notify based timing is a later refinement.

## 2026-06-12 Update: Auto Room Boundary MVP

- `RoomCombatActor` now includes automatic boundary blocker components for room combat tests.
- `TriggerBox` remains overlap-only for room entry detection.
- Boundary preview is refreshed in the editor when boundary settings change.
- Boundary blockers are disabled before combat, enabled on combat start, and disabled again when the room is cleared.
- This is the current MVP way to prevent leaving the combat room without manually placing cube doors.
- Long term, this logic can move to `CombatLockVolume` or `RoomBoundaryActor`, while `RoomDoorActor` handles visible door presentation.

## 2026-06-12 Update: Room Door MVP

- `RoomCombatActor` now supports a `DoorActors` array for the first room door open/close test.
- When room combat starts, registered door actors are shown and their collision is enabled.
- When the room is cleared, registered door actors are hidden and their collision is disabled.
- This keeps the current MVP small: no dedicated `RoomDoorActor`, door animation, lock state, reward gate, or next-room flow yet.

## 1. 문서 목적

?�재 ?�로?�트???�제�?구현?�어 ?�는 기능�??�시 MVP 구조�??�리?�다.

Codex???�음 ?�업 ?�에 ??문서�?먼�? ?�고, ?�제�??�용 중인 Blueprint?� C++ 구조�??�인?????�정?�야 ?�다.

## 2. ?�재 ?�제 ?�용 ?�름

| 구분 | ?�재 �?| 비고 |
|---|---|---|
| 기본 �?| `/Game/TopDown/Lvl_TopDown` | `DefaultEngine.ini` 기�? |
| 기본 GameMode | `/Game/Blueprints/Core/BP_CombatGameMode` | ?�재 TopDown 맵에???�용 |
| ?�레?�어 Character | `BP_PlayerCharacter` | ?�제 ?�레??Pawn |
| PlayerController | `BP_PlayerController` | ?�제 ?�력 Controller |
| 주의 ?�래??| `MyProjectCharacter`, `MyProjectPlayerController` | ?�일?� ?��?�??�재 기본 ?�레???�름?�서 직접 부모로 ?��? ?�는 것으�??�인??|

## 3. ?�재 구현??주요 기능

| 기능 | 구현 ?�치 | ?�재 ?�태 | 비고 |
|---|---|---|---|
| 마우????�?| `MouseWheelCameraZoomSubsystem` | ?�작 ?�인 | ?�재 Pawn??`SpringArm`??찾아 카메??거리 조절 |
| WASD ?�동 | `PlayerKeyboardMovementSubsystem` | ?�작 ?�인 | 기존 ?�릭 ?�동 Mapping Context ?�거 ??직접 ?�동 ?�력 ?�용, ?�레?�어 Capsule??Pawn 채널 Overlap ?��? |
| ?�클�??�동 ?�거 | `PlayerKeyboardMovementSubsystem`, IMC ?�정 | ?�작 ?�인 | ?�재 ?�동?� WASD 기�? |
| ?�작 ?�트�??�니메이??| `PlayerKeyboardMovementSubsystem` | 구현??| `Katana_Blade_Intro` ?�생 |
| ?�트�?�?무기 ?�켓 ?�환 | `PlayerKeyboardMovementSubsystem` | 구현??| 검??`SheathSocket_Back`?�서 ?�작 ??`HandGrip_R`�??�동 |
| Space ?�피 | `PlayerKeyboardMovementSubsystem` | 구현??| 거리, 지?�시�? 쿨�??? 무적 ?�간?� Player Stats 기�? |
| 공격 �??�피 캔슬 | `PlayerKeyboardMovementSubsystem`, `PlayerMeleeAttackSubsystem` | 구현??| WASD 캔슬?� 막고, Space ?�피로만 공격 취소 |
| 공통 체력 | `HealthComponent` | 구현??| ?�레?�어/??공통 ?�용 ?�보 |
| ?�레?�어 체력 ?�결 | `PlayerHealthSubsystem` | 구현??| ?�레??�?Pawn??체력 컴포?�트 보장 |
| ?�레?�어 체력 ?�브 HUD | `PlayerHealthSubsystem`, `PlayerHealthOrbWidget` | 구현??| 좌측 ?�단 ?�형 체력 ?�브. ?�레?��? `Content/UI/Textures/T_UI_HealthOrb_Frame.png` ?�용 |
| H ???�스???��?지 | `PlayerHealthSubsystem` | 구현??| ?�버그용 체력 감소 |
| ?�레?�어 ?�망 처리 | `PlayerHealthSubsystem` | 구현??| ?�동/?�력 비활?? ?�망 ?�니메이?? `/Game/SuperVFXPack/NiagaraSystem/NS_BPDissolve` ?�망 VFX |
| ?�레?�어 ?�탯 관�?| `PlayerStatsComponent`, `PlayerStatsDataAsset` | 구현??| 체력, 공격, 콤보, ?�피 ?�치 관�?|
| 좌클�?근접 콤보 공격 | `PlayerMeleeAttackSubsystem` | 구현??| ?�재 ?�름?� Auto지�??�제 ??��?� ?�동 근접 공격 MVP |
| 공격 범위 ?�정 | `PlayerMeleeAttackSubsystem` | 구현??| 부채꼴 범위 ?�의 모든 ?�에�??��?지 |
| 공격 범위 ?�버�??�시 | `PlayerMeleeAttackSubsystem` | 구현??| �?��??미리보기, 주황???�제 공격 ?�시 |
| 콤보 ?�버�??�시 | `PlayerMeleeAttackSubsystem` | 구현??| 좌측 ?�단 ?�재 콤보 ?�시 |
| 공격 ?�격�?MVP | `PlayerMeleeAttackSubsystem`, `EnemyBase` | ?�거??| ???�백�??�트?�톱?� ?�재 ?�용?��? ?�음 |
| ??공통 기반 | `EnemyBase` | 구현??| 체력, 추적, 근거�?공격, ?�망 처리, Pawn?�리 막�? ?�는 Collision 처리 |
| ??간단 ?�니메이??| `EnemySimpleAnimationComponent` | 구현??| ?�용 AnimBP가 ?�는 ?�의 Idle/Move ?�니메이???�시 ?�환 |
| ?�스????| `SurvivorSquareEnemy` | 구현??| `EnemyBase` ?�속 ?�스????|
| Little Demon ??| `BP_Enemy_LittleDemon`, `DA_EnemyStats_LittleDemon` | 구현??| `/Game/Demons_Big_Pack/Little_Demon` ?�셋 기반 ?�규 ?? attack1/?�망 ?�니메이???�용, ?�망 ???�아 ?�음 |
| ???�폰 | `EnemySpawnSubsystem` | 구현??| ?�역 ?�스???�폰/?�시 검증용?�며 기본 OFF. `MyProject.EnemySpawn.GlobalTestSpawnEnabled=1`�?DataAsset `bSpawnEnabled=true`???�만 ?�용 |
| �??�투 MVP | `RoomCombatActor` | 구현??| Trigger ?�장 ?????�폰, �?�?경계 ?�금, ?�성?????�망 ?�벤??추적, ?�멸 ??`Room Cleared` 출력 ??�?경계 ?�제 |
| �??�투 Blueprint | `BP_RoomCombatActor` | ?�성??| `/Game/Blueprints/Rooms/BP_RoomCombatActor` |
| ???�폰 ?�이??| `EnemySpawnDataAsset` | 구현??| ?�재???�역 ?�스???�폰 ?�정. ?��? �??�폰 ?�이?�로 ?�사??가??|
| ???�력�??�이??| `EnemyStatsDataAsset` | 구현??| 체력, ?�동?�도, ?��? 거리, 캡슐 겹침 ?�유거리, 공격?? 공격 ?�거�? 공격 쿨�???관�?|
| ???�니메이??리�?�?기�? | `Docs/EnemyAnimationRetargeting.md` | 문서?�됨 | 캐릭?�별 ?�켈?�톤 ?��?, IK Rig / IK Retargeter ?�선 |

## 4. ?�재 조작 규칙

| ?�력 | ??�� | 비고 |
|---|---|---|
| WASD | ?�동 | 공격 중에???�동 불�? |
| Mouse Wheel | 카메??�????�웃 | Subsystem?�서 처리 |
| Left Mouse Button | 근접 콤보 공격 | 1?�/2?�/3?� |
| Space | ?�피 | 공격 중에??발동 가?�하�?공격??캔슬 |
| H | ?�버�??��?지 | ?�레?�어 체력 감소 ?�스??|

## 5. ?�재 공격 구조

- ?�재 기본 공격?� ?�동 ?��????�니??좌클�??�동 근접 콤보 공격?�다.
- `PlayerMeleeAttackSubsystem` ?�름?� 과거 ?�동 ?��? MVP?�서 ???�름?�라 ?�제 ??���??�름???�긋???�다.
- ?�재 공격?� 마우??커서 방향??기�??�로 캐릭?��? ?�전????부채꼴 범위???��?지�?준??
- 공격 중에??WASD ?�동???�긴??
- 공격 �?Space ?�피???�용?�며, 공격 ?�니메이?�과 ?�진 ?�텝???�는??
- 공격 ?�진 ?�텝?� �?충돌?� ?��??�되, ??Actor???�동 충돌 ?�?�으�?무시?�다.
- 공격 ?�치?� 콤보 ?�?�밍?� `PlayerStatsComponent`?� `PlayerStatsDataAsset`???�선 참고?�다.
- ?�재 기본 공격?� 별도 콤보 VFX�??�성?��? ?�는??

## 6. ?�재 무기/검�?구조

| ??�� | ?�재 �?| ?�정 ?�치 |
|---|---|---|
| ?�착 검 컴포?�트 | `EquippedOtachi` | `BP_PlayerCharacter` |
| 검 ?�위 메시 컴포?�트 | `Otachi_Handle` | `BP_PlayerCharacter` |
| 검�?컴포?�트 | `EquippedOtachiSheath` | `BP_PlayerCharacter` |
| ???�켓 | `HandGrip_R` | `SKM_Manny_Simple` Skeleton Tree |
| 검�??�켓 | `SheathSocket_Back` | `SKM_Manny_Simple` Skeleton Tree |
| 검 ?�기 | `EquippedOtachi` Scale | `BP_PlayerCharacter` |
| 검�??�기 | `EquippedOtachiSheath` Scale | `BP_PlayerCharacter` |
| ?�에 ??검 ?�치/방향 | `HandGrip_R` ?�켓 Transform | `SKM_Manny_Simple` |
| ?�에 �?검�??�치/방향 | `SheathSocket_Back` ?�켓 Transform | `SKM_Manny_Simple` |

주의:

- ?�재 ?�트�??�작 ??코드가 검/검집을 ?�켓???�시 붙인??
- `EquippedOtachi`, `Otachi_Handle`, `EquippedOtachiSheath`?� �??�위 컴포?�트???��??�에??`NoCollision`�?Physics Off�??��??�다.
- ?�라???�치?� ?�전?� Blueprint 컴포?�트 Transform보다 ?�켓 Transform ?�향?????�다.
- ?�기??`SnapToTargetNotIncludingScale`???�용?��?�?컴포?�트 Scale???�선 조정?�다.

## 7. ?�재 ?�이???�셋 방향

| ?�이??| ?�??| 기본 경로/비고 |
|---|---|---|
| ?�레?�어 ?�탯 | `PlayerStatsDataAsset` | `/Game/Data/Player/DA_PlayerStats_Default` ?�용 방향 |
| ??개체 ?�력�?| `EnemyStatsDataAsset` | `/Game/Data/Enemy/Stats` ?�더?�서 ???�?�별 ?�성 방향 |
| ???�폰 ?�정 | `EnemySpawnDataAsset` | `/Game/Data/Enemy/Spawn/DA_EnemySpawn_Default` ?�용 방향 |

?�레?�어 ?�탯 ?�이?�에??체력, 공격?? 공격 범위, 콤보 ?�?�밍, ?�피 거리/?�간/쿨�????�이 ?�함?�다.

??개체 ?�력치�? ???�폰 규칙?� 분리?�서 관리한?? `EnemyBase`??`StatsDataAsset`??지?�되???�으�?Data Asset 값을 ?�선 ?�용?�고, ?�으�?기존 C++ 기본값을 fallback?�로 ?�용?�다. ?�의 ?�레?�어 ?�근 거리??`StopDistance`?� `PersonalSpacePadding`?�로 조절?�며, ?�제 ?��? 거리???�레?�어/??캡슐 반�?름을 ?�께 고려?�다.

?�레?�어 Capsule�???Capsule?� Pawn 채널??Overlap?�로 처리?�다. ?�레?�어가 ?�에�??�러?�여 ?�동 불�?가 ?�거??공격 ?�진 ?�텝?�로 ?�을 밀?�내??것을 막기 ?�한 기�??�며, ???��?지??Collision Block???�니??`EnemyBase`??거리/쿨�????�정?�로 처리?�다.

`EnemySpawnDataAsset`?� ?�재 `SpawnEnemyEntries` 배열�??�러 ??Blueprint Class�??�간/가중치 기�??�로 ?�어 ?�폰?????�다. ?�성 Entry가 ?�으�?`EnemyClass`�?fallback?�로 ?�용?�고, �?값도 ?�으�?기존 ?�스?�용 `/Game/Blueprints/Enemy/Basic/BP_Enemy_Imp`�?fallback?�로 ?�용?�다.

?�재 게임 방향?� 무한 ?�역 ?�폰보다 �??�장 ??SpawnPoint 기�??�로 ?�해�??�을 ?�성?�는 ?�테?��? ?�리?�형 구조?? ?�라??`EnemySpawnSubsystem`?� ??��?��? ?�되 ?�역 ?�스???�폰/?�시 검증용?�로 보고, ?�식 진행 구조??`RoomCombatActor` ?�는 Stage/Wave 구조�??�환?�다.

## 8. ?�재 ???�니메이??방향

- ?�의 공통 로직?� `EnemyBase`???�다.
- ?�의 메시, ?�켈?�톤, Animation Blueprint??캐릭?�별 Blueprint?�서 관리하??방향?�다.
- `EnemyBase`?????�상 Imp 메시?� Imp AnimBP�?C++?�서 직접 ?�드코딩?��? ?�는??
- ?�용 AnimBP가 ?�는 ?��? `EnemySimpleAnimationComponent`�?Idle/Move ?�니메이?�을 ?�시 ?�환?????�다.
- ?�재 `BP_Enemy_LittleDemon`?� `SK_Little_Demon`, `Anim_Little_Demon_idle1`, `Anim_Little_Demon_run1`, `Anim_Little_Demon_attack1`, `Anim_Little_Demon_death1`???�용?�다.
- `EnemyBase`??`DeathAnimation`??지?�되???�으�??�망 ???�당 ?�니메이?�을 1???�생?�다.
- `BP_Enemy_LittleDemon`?� `bDestroyOnDeath=false`�??�정?�어 죽�? ??즉시 ?�거?��? ?�는??
- Imp 같�? 몬스?�는 ?�본 ?�켈?�톤???��??�고, UE5 Manny ?�니메이?�이 ?�요?�면 IK Rig / IK Retargeter�?변?�해???�용?�다.
- Imp ?�체�?UE5 Manny ?�켈?�톤?�로 강제 변?�하??방식?� ?�재 추천?��? ?�는??
- ?�세??기�??� `Docs/EnemyAnimationRetargeting.md`�??�른??

## 9. ?�시 MVP ?�는 ?�름 ?�리 ?�요 ??��

| ??�� | ?�유 | 추천 ?�리 방향 |
|---|---|---|
| `PlayerMeleeAttackSubsystem` | ?�름?� ?�동 공격?��?�??�재??좌클�??�동 콤보 공격 ?�당 | 추후 `BasicAttackComponent` ?�는 `PlayerMeleeAttackComponent` ?�보 |
| `SurvivorSquareEnemy` | ?�스?�용 ?�모 ??| 추후 `BP_EnemyBase` 기반 ?�으�??�리 |
| `EnemySpawnSubsystem` | ?�역 ?�스???�폰/?�시 검증용, 기본 OFF | 추후 `RoomCombatActor` ?�는 Stage/Wave ?�스?�으�?�??�위 ?�폰 ?�환 |
| ?�버�?UI | ?�면 메시지 기반 | ?�식 체력 ?�시???�브 UI�??�동 �? ?��? ?�버�?문자???�요 ???�거 ?�보 |

## 10. ?�테?��? ?�리?�형 ?�환 ?�정 구조

?�재 구현???�투/조작/체력/??구조???��??�고, ?�폰�?진행 ?�름�?�??�위�??�환?�다.

1. �??�장 Trigger
2. `RoomCombatActor` ?�투 ?�작
3. `SpawnPoints` 기�? ???�성
4. `AliveEnemies` 추적
5. ?�멸 ??`Room Cleared` ?�버�?출력
6. �??�힘/?�림 ?�는 보상 ?�성
7. ?�음 �??�동

초기 MVP???�덤 ?�전 ?�이 같�? 맵에 ?�수??고정 방을 배치?�는 방식?�로 검증한?? Stage 1???�기 목표 구조??7�?Room ?�후?� �?보스방으�??�장?�는 방향?�다.

## 11. ?�음 ?�업 ??주의?�항

- ?�레?�어/?�력/카메???�업 ??`BP_PlayerCharacter`, `BP_PlayerController`, `BP_CombatGameMode` ?�용 ?��?�??�인?�다.
- 공격 관???�업 ??`Docs/BasicAttackSystem.md`?� `Docs/AttackSkillSystem.md`�??�인?�다.
- 조작/?�피/?�트�?관???�업 ??`Docs/PlayerControlSystem.md`�??�인?�다.
- ??관???�업 ??`Docs/EnemySystem.md`, `Docs/EnemyArchitecture.md`, `Docs/EnemySpawnSystem.md`�??�인?�다.
- �??�리?�형 ?�투 ?�업 ??`Docs/RoomCombatSystem.md`�??�인?�다.
- ???�니메이??리�?�??�업 ??`Docs/EnemyAnimationRetargeting.md`�??�인?�다.
- 무기 ?�착/?�켓 관???�업?� `SKM_Manny_Simple`???�켓�?`BP_PlayerCharacter` 컴포?�트 ?�름???�께 ?�인?�다.
- C++ ?�??추�?/??�� ?�는 UPROPERTY 변경�? ?�디??종료 ???�체 빌드?�다.


> Archive note: This file preserves pre-restructure dated notes. It may contain legacy or corrupted text and must not be used as current implementation truth unless explicitly requested.
