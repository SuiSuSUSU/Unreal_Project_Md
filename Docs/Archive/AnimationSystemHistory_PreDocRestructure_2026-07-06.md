---
Status: REFERENCE
Scope: Archived Animation System History
Last Updated: 2026-07-05
Source of Truth: No
---

# AnimationSystem.md

## 1. Document Purpose

This document defines animation ownership and implementation direction across player combat, enemy combat, spawn intros, hit reactions, and animation-timed VFX.

Use this document before changing AnimBPs, Montages, animation sequences, attack timing, animation notifies, retargeting, or enemy animation playback.

## 2. Core Animation Direction

Animation quality is asset-specific and should be verified visually in Unreal Editor.

C++ may trigger animation playback, but smooth transitions belong mainly to AnimBP, Montage, BlendSpace, Slot, and notify setup.

Do not force all monsters onto one skeleton or one generic animation setup.

## 2026-07-05 Update: Player Hit React Type MVP

Player non-lethal hit reaction is implemented as a small MVP path in `PlayerHealthSubsystem`.

- `EPlayerHitReactType` currently supports `None`, `Light`, and `Heavy`.
- Enemy attacks explicitly request the reaction type after confirmed damage instead of letting `HealthComponent.OnHealthChanged` choose presentation.
- Enemy weak attacks request `Light`.
- Enemy strong attacks request `Heavy`.
- Light animation asset: `/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/4_Damages/1__Front/Katana_Blade_Damage_Front_Small_ver_A`.
- Heavy animation asset: `/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/4_Damages/1__Front/Katana_Blade_Damage_Front_Big_ver_C`.
- Playback method: `PlaySlotAnimationAsDynamicMontage` on `DefaultSlot`.
- The reaction is presentation-only for now. It does not lock movement, cancel attacks, add invulnerability, or change damage rules.
- A short minimum interval prevents rapid repeated damage from restarting the hit reaction every frame.
- Death animation remains separate and should take priority when health reaches zero.

If the MVP reaction feels too disruptive, prefer tuning blend/interval values, root motion settings, or moving to a dedicated player hit-react montage/slot before adding gameplay stagger.

## 2026-07-02 Update: HitReact Structure Design

This section is a design target only. Do not treat `HitReact` or `HitReactComponent` as implemented until code and enemy-specific assets exist.

Hit reaction should be enemy-specific presentation and optional gameplay interruption, not a hardcoded response inside every damage event.

Recommended direction:

- `EnemyBase` may expose a lightweight hit-react request or event after a confirmed hit.
- Enemy Blueprint, AnimBP, Montage assets, or a future `HitReactComponent` should decide the actual reaction.
- The primary decision should be based on attack damage, attack/combo hit-react power, and the enemy's hit-react resistance.
- Little Demon, Fast, Tank, MiniBoss, and Boss enemies may feel different mainly through different resistance values, immunity rules, and available reaction assets.
- Hit reaction should start as presentation-only unless a specific gameplay stagger rule is explicitly designed.
- If gameplay stagger is added later, separate it from animation playback so damage, interruption, and visual reaction remain understandable.

Recommended decision formula:

```text
HitReactScore = DamageAmount + AttackTypeBonus + ComboStepBonus + RewardBonus

if HitReactScore >= EnemyHitReactResistance:
    choose Light / Medium / Heavy reaction by threshold
else:
    None
```

`DamageAmount` and `HitReactPower` should stay conceptually separate. Damage controls health loss; hit-react power controls how much the body should react. A high-damage attack can also have high hit-react power, but the system should allow future attacks that are low damage but high stagger, or high damage but low stagger.

Recommended reaction categories:

| Category | Meaning |
|---|---|
| None | No visible reaction. Useful for tiny damage ticks or immune states. |
| Light | Small flinch without interrupting major actions. |
| Medium | Noticeable flinch, possibly brief movement lock for normal enemies. |
| Heavy | Strong stagger or knockback candidate for heavy attacks or special rewards. |
| Immune | Boss, armored phase, spawn intro, death, or uninterruptible attack state. |

Recommended request data:

| Field | Purpose |
|---|---|
| Hit source actor | Who caused the hit. |
| Hit target actor | Which enemy received the hit. |
| Hit direction | Direction for flinch, turn, or knockback candidates. |
| Hit location | Optional location for VFX or directional presentation. |
| Damage amount | Useful for reaction threshold decisions, not for reapplying damage. |
| HitReactPower | Optional attack-specific reaction force separate from raw damage. |
| Combo step / attack id | Lets 3rd-hit or heavy attacks request stronger reactions. |
| Gameplay tags / reward tags | Lets bleed, lightning, or special rewards choose different presentation later. |
| Target resistance | Enemy-side resistance threshold used to decide whether a reaction happens. |

Recommended non-responsibility:

- Do not calculate hit detection.
- Do not apply damage.
- Do not decide Room Clear or enemy death.
- Do not own weapon trail or impact VFX playback.
- Do not reintroduce the removed generic white hit-flash path.
- Do not force all enemies to share one reaction animation.

First implementation should be conservative:

1. Add only a request/event path from confirmed hits.
2. Default every enemy to no reaction until a resistance threshold is explicitly set.
3. Start with one conservative threshold rule: weak hits do nothing, stronger combo hits can request a light reaction.
4. Do not interrupt attacks, spawn intros, death animations, or boss states.
5. Add one visually verified Little Demon or Fast flinch before expanding to Tank/Boss behavior.

## 3. Current Enemy Animation Rules

- `EnemyBase` owns common runtime triggers such as chase, attack cooldown, attack playback request, spawn intro flow, and death handling.
- Enemy mesh, skeleton, AnimBP, Montages, and animation assets should be configured per enemy Blueprint.
- Little Demon currently uses the SingleNode / `EnemySimpleAnimationComponent` MVP path.
- Tank / Butcher uses a dedicated AnimBP and attack Montage path.
- Fast / DemonHeavy uses a dedicated AnimBP and a weak/strong Montage pair; its third successful melee attack uses the strong attack Montage and separate damage/range multipliers.
- Random attack selection is not a current default because unverified animation transitions caused visible issues earlier.
- If multiple attacks are needed, prefer verified Montages or a dedicated combo Montage over random sequence swapping.

## 4. Current Player Animation Rules

- Player basic attack is a 1/2/3 combo.
- Attack gameplay damage timing is currently code-driven, not fully notify-driven.
- Weapon trail timing should move toward AnimNotify windows.
- Attack movement, dodge cancel, and control rules are documented in `BasicAttackSystem.md` and `PlayerControlSystem.md`.

## 5. AnimNotify Rule

Use animation notifies when the visual timing must match a specific animation frame.

Good notify candidates:

- Weapon trail start/stop.
- Heavy weapon ground spark.
- Attack impact timing.
- Footstep VFX/SFX.
- Spawn intro VFX.
- Death VFX.

Do not rely on fixed delay values when the timing should follow animation frames.

Implemented notify paths:

| Notify | Purpose | Notes |
|---|---|---|
| `AnimNotifyState_WeaponTrail` | Opens/closes weapon trail windows. | Used for sword trail timing. |
| `AnimNotify_PlayerMeleeHit` | Executes the queued player melee hit. | Use this on the frame where the sword should visually connect so damage, blood, and hit VFX do not fire at attack start. |
| `AnimNotify_GroundImpactVfx` | Spawns a short ground spark VFX on a weapon-ground contact frame. | Uses `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` by default and traces down from a Blueprint trace-point component to the floor. |
| `AnimNotifyState_GroundContactImpactVfx` | Watches a short contact window and spawns one spark when the trace point gets close enough to the floor. | Prefer this when the exact one-frame ground contact is hard to place by hand. |
| `AnimNotifyState_GroundScrapeVfx` | Spawns repeated small ground sparks while a scrape window is active. | Use this when the weapon drags across the floor instead of hitting one frame. It follows a Blueprint trace-point component each tick. |

Ground spark notifies should only decide timing and trace location. Spawned spark Niagara components should behave as world-space effects: stop new emission when their lifetime ends, let existing particles fade, then clean up the component later.

Ground Scrape VFX should not attach spawned sparks to the character or weapon. The notify may sample the weapon trace point every tick and may use its world-space movement direction for orientation, but already spawned sparks should remain independent. If a scrape window ends or is interrupted, stop new emission first, keep existing sparks through the minimum emission lifetime, let particles fade, then force cleanup later.

## 2026-07-05 Update: Player Melee Hit Timing

Player melee damage is no longer applied at the instant the attack input starts.

Current path:

```text
Attack input
-> queue pending melee hit data
-> fallback combo hit delay OR AnimNotify_PlayerMeleeHit
-> DamageEnemiesInAttackCone
-> confirmed-hit blood and hit VFX
```

Rules:

- `AnimNotify_PlayerMeleeHit` is the preferred timing source for exact hit frames.
- The fallback delay exists so attacks still work if a combo animation has no hit notify yet.
- If a player dodge cancels the attack before the hit timing, the pending hit is cleared.
- Damage timing, blood VFX, and hit-distortion VFX should stay in the same confirmed-hit path.

## 2026-07-03 Update: Ground Spark Trace-Point Rule

Ground spark notifies should use Blueprint-owned trace-point components for location.

- `Trace Point Component Name` should usually be `WeaponVfx_GroundContact` for ground spark notifies.
- Add `WeaponVfx_GroundContact` as a SceneComponent under `EquippedOtachi` in `BP_PlayerCharacter`.
- `BladeTipTracePoint` can remain as a blade-tip marker or legacy fallback, but it should not be the main ground-contact marker.
- If the Blueprint component is missing, the current notify implementation can create a runtime-only trace point under `EquippedOtachi` so the VFX can still be tested without modifying the weapon static mesh asset.
- Owner-local offset placement and direct static-mesh socket editing are deprecated for ground sparks.
- If the trace-point component is missing and runtime creation is disabled or fails, the notify should fail visibly through setup logs rather than spawning at an approximate owner-local fallback location.

## 6. Retargeting Rule

Enemy retargeting remains asset-specific.

- Use the original monster skeleton when possible.
- Use IK Rig / IK Retargeter only when sharing animation across skeletons is actually needed.
- Validate retargeted animation in the editor before connecting it to gameplay.
- Do not hardcode mesh, skeleton, or animation asset paths in `EnemyBase`.

## 7. Current Non-Goals

- No full player locomotion overhaul.
- No final hit-reaction system. Only a future structure direction is documented.
- No final death animation system for every enemy.
- No animation-driven damage notify system yet.
- No universal enemy animation controller for every monster type.

## 8. Future Animation Candidates

- Player attack impact notifies.
- Enemy attack damage notifies.
- Dedicated hit reaction assets per enemy family.
- Dedicated death assets per enemy family.
- Boss and mini-boss animation state machines.
- BlendSpace-based locomotion for each enemy family.
- Formal spawn-intro state for placed and spawned enemies.

## 9. Related Documents

- `Docs/EnemyAnimationRetargeting.md`: enemy skeleton, retargeting, and current enemy-specific animation notes.
- `Docs/EnemySystem.md`: enemy gameplay behavior.
- `Docs/EnemyArchitecture.md`: enemy architecture boundaries.
- `Docs/BasicAttackSystem.md`: player combo attack behavior.
- `Docs/VFXSystem.md`: animation-timed visual effects.
