---
Status: ACTIVE
Scope: Player Stats
Last Updated: 2026-07-06
Source of Truth: Yes
---

# PlayerStatsSystem.md

## 1. Document Purpose

This document defines the current player stat ownership rules.

Use this before changing player health, basic attack numbers, combo values, dodge values, Run reward stat modifiers, or the temporary combat element state.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/PlayerStatsSystemHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Direction

Player stat values are split into two layers.

| Layer | Owner | Meaning |
|---|---|---|
| Baseline editable stats | `PlayerStatsDataAsset` | Default values that can be tuned in the editor. |
| Runtime stat provider | `PlayerStatsComponent` | Reads the data asset, applies runtime Run modifiers, and exposes final values to gameplay systems. |

Rules:

- Do not modify `PlayerStatsDataAsset` at runtime for temporary rewards.
- Run rewards should apply through `PlayerStatsComponent` runtime modifiers.
- VFX assets, sockets, Niagara timing, and visual offsets should not live in player stat data.
- Combat presentation belongs to `WeaponTrailComponent`, animation notifies, VFX systems, or future presentation components.

## 3. Current Files and Assets

| Area | Path / Class | Status |
|---|---|---|
| Player stat data asset class | `Source/MyProject/Combat/PlayerStatsDataAsset.h/.cpp` | Implemented. |
| Player stat runtime component | `Source/MyProject/Combat/PlayerStatsComponent.h/.cpp` | Implemented. |
| Default player stat asset | `/Game/Data/Player/DA_PlayerStats_Default` | Current intended data asset path. |
| Current player Blueprint | `/Game/Blueprints/Player/BP_PlayerCharacter` | Should own or receive `PlayerStatsComponent`. |
| Runtime fallback hookup | `PlayerHealthSubsystem` | Ensures the player has a stats component if missing. |

## 4. Current Baseline Stat Groups

`PlayerStatsDataAsset` and `PlayerStatsComponent` currently cover these baseline groups.

### Health

| Field | Meaning |
|---|---|
| `MaxHealth` | Player maximum health. |
| `DebugDamage` | Debug damage amount used by health testing. |

### Basic Attack

| Field | Meaning |
|---|---|
| `BasicAttackDamage` | Base melee attack damage. |
| `BasicAttackRange` | Cone attack range. |
| `BasicAttackAngleDegrees` | Cone attack angle. |
| `BasicAttackCooldown` | Minimum attack input cooldown. |
| `ComboResetWindow` | Time before combo step resets. |
| `MaxComboStep` | Maximum combo step, currently clamped to 1-3. |
| `Combo1ForwardStepDistance` | Forward movement distance for combo 1. |
| `Combo2ForwardStepDistance` | Forward movement distance for combo 2. |
| `Combo3ForwardStepDistance` | Forward movement distance for combo 3. |
| `ComboForwardStepDuration` | Duration of the attack forward step. |
| `ComboTransitionRatio` | Timing ratio for combo transition. |
| `BasicAttackAnimationSlotName` | Animation slot name used by attack playback. |

### Dodge

| Field | Meaning |
|---|---|
| `DodgeDistance` | Dodge travel distance. |
| `DodgeDuration` | Dodge movement duration. |
| `DodgeCooldown` | Dodge cooldown. |
| `DodgeInvulnerabilityDuration` | Invulnerability time during dodge. |

### Debug Flags

| Field | Meaning |
|---|---|
| `bShowHealthDebug` | Shows health debug feedback when enabled. |
| `bShowComboDebug` | Shows combo debug feedback when enabled. |
| `bShowBasicAttackDebug` | Shows basic attack debug visuals when enabled. |
| `bShowChainLightningDebug` | Shows the intended Chain Lightning transfer line when enabled. |

## 5. Runtime Run Reward Modifiers

`PlayerStatsComponent` currently supports runtime-only Run modifiers.

| Modifier | Method | Affects |
|---|---|---|
| Max health bonus | `AddRunMaxHealthBonus` | Final max health returned by `GetConfiguredMaxHealth`. |
| Basic attack damage bonus | `AddRunBasicAttackDamageBonus` | Final attack damage returned by `GetBasicAttackDamage`. |
| Dodge cooldown bonus | `AddRunDodgeCooldownBonus` | Final dodge cooldown returned by `GetDodgeCooldown`. |
| Move speed bonus | `AddRunMoveSpeedBonus` | Owner character movement speed. |
| Reset | `ResetRunRewardModifiers` | Clears all runtime Run modifiers. |

Run modifier rules:

- Runtime modifiers must not write back to `PlayerStatsDataAsset`.
- Non-finite reward values are ignored by a guard in `PlayerStatsComponent`.
- Run modifiers reset on player death through `PlayerHealthSubsystem`.
- Run modifiers also reset naturally when the level/session recreates the component.
- `RunMoveSpeedBonus` temporarily edits the owner character movement speed and restores from the cached base speed on reset.

## 6. Runtime Combat Element State

`PlayerStatsComponent` currently owns a temporary combat element state.

| State | Meaning |
|---|---|
| `Neutral` | Default state. No elemental attack rule is active. |
| `Lightning` | Debug Lightning state used by the current Chain Lightning MVP. |

Current rules:

- L toggles between `Neutral` and `Lightning` through `PlayerKeyboardMovementSubsystem`.
- The debug element toggle logs to Output Log only. Persistent on-screen debug text is not used.
- `WeaponTrailComponent` listens to combat element changes and can attach the Lightning sword aura.
- `PlayerMeleeAttackSubsystem` checks `IsLightningElementActive()` before applying Chain Lightning MVP damage.
- `ResetRunRewardModifiers()` also resets the combat element to `Neutral`.

This is a runtime state hook, not the final elemental reward system. Reward cards, Prism unlocks, Shock, and final Lightning VFX are documented in `Docs/ElementalRewardSystem.md`.

## 7. Current Consumers

| Consumer | Uses |
|---|---|
| `PlayerMeleeAttackSubsystem` | Basic attack damage, range, angle, cooldown, combo values, debug flags, Lightning state. |
| `PlayerKeyboardMovementSubsystem` | Dodge distance, duration, cooldown, invulnerability, debug element toggle. |
| `PlayerHealthSubsystem` | Max health, runtime Run reward reset, player stat component guarantee. |
| `RewardSelectionWidget` | Applies temporary Run stat rewards through `PlayerStatsComponent`. |
| `WeaponTrailComponent` | Listens to combat element changes for Lightning aura. |

## 8. Current Non-Goals

- Do not store Niagara systems, VFX colors, socket names, or VFX transforms in `PlayerStatsDataAsset`.
- Do not apply permanent upgrades through this component yet.
- Do not implement Luck-based reward rolls here yet.
- Do not make `PlayerStatsComponent` own reward pool selection.
- Do not make `PlayerStatsComponent` own Stage progression, Room progression, or reward UI.
- Do not treat `Neutral` / `Lightning` debug state as the final element reward ownership model.

## 9. Future Candidates

The following are planning candidates, not implemented final systems.

- Luck stat for reward grade or Prism chance.
- Technique levels for weapon or element growth.
- Dedicated Run reward state container if reward complexity grows.
- Separate weapon stat data if multiple weapons become real choices.
- Separate elemental build state if multiple elements become real choices.
- Debug UI that shows final calculated stat values.

## 10. Related Documents

- `Docs/CurrentImplementation.md`: current implementation snapshot.
- `Docs/RewardSystem.md`: reward type and runtime reward rules.
- `Docs/ElementalRewardSystem.md`: Lightning and future element reward planning.
- `Docs/BasicAttackSystem.md`: basic melee combo behavior.
- `Docs/PlayerControlSystem.md`: movement, dodge, and debug input behavior.
- `Docs/VFXSystem.md`: VFX ownership and combat presentation boundaries.
