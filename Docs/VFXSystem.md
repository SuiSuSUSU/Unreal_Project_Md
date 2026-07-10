---
Status: ACTIVE
Scope: VFX System
Last Updated: 2026-07-06
Source of Truth: Yes
---

# VFXSystem.md

## 1. Document Purpose

This document is the current source of truth for combat VFX ownership and rules.

Use this before changing weapon trails, hit VFX, ground sparks, Lightning VFX, reward-driven VFX growth, Niagara setup, or animation-timed visual effects.

Detailed dated history was moved to `Docs/Archive/VFXSystemHistory.md`.

## 2. Current Direction

Combat VFX should support readability and build identity.

The player should eventually see build changes through attack presentation, such as larger trails, sword waves, Lightning chains, bleed effects, or dodge-triggered effects.

Stats own numbers. Presentation systems own visuals. Do not hardcode VFX assets into stat data.

## 3. Current Owner Map

| Area | Current Owner |
|---|---|
| Player sword trail slots and Niagara assignment | `WeaponTrailComponent` |
| Exact sword trail timing | `AnimNotifyState_WeaponTrail` |
| Basic attack timing, cone hit, and damage | `PlayerMeleeAttackSubsystem` |
| Confirmed hit blood/distortion VFX | `PlayerMeleeAttackSubsystem` for now |
| Ground impact/contact/scrape spark timing | Ground VFX AnimNotify / AnimNotifyState classes |
| Runtime combat element state | `PlayerStatsComponent` |
| Lightning sword aura attachment | `WeaponTrailComponent` |
| Reward-driven trail growth hook | `RewardSelectionWidget` -> `WeaponTrailComponent` |
| Future hit-impact presentation | Future `WeaponImpactVfxComponent` |
| Target body hit reaction | Enemy Blueprint, AnimBP, or future `HitReactComponent` |

## 4. Weapon Trail Rules

- The old basic combo slash VFX path was removed.
- `WeaponTrailComponent` owns player sword-trail presentation.
- `AnimNotifyState_WeaponTrail` should control exact start/end timing on attack animations.
- `PlayerMeleeAttackSubsystem` may call fallback trail playback only when notify timing is disabled.
- `ActivationDelay` and `Duration` are fallback test values only.
- `ComboTrailSlots` should define the Niagara asset, attach component, transform, scale, and optional user color.
- After a weapon trail window ends, `WeaponTrailComponent` stops emission but keeps the Niagara component alive briefly through `TrailPostDeactivateDestroyDelay` so remaining particles can fade naturally.
- `BP_PlayerCharacter.WeaponTrailComponent.ComboTrailSlots` currently use `/Game/SlashTrail_SoftTofu/Niagara/Distortion_Only/NS_SlashTrail_Distortion_Only`.
- The current sword trail was adopted from the SlashTrail SoftTofu Kwang demo reference, especially `Ability_R_Distortion_Only`.

Preferred flow: attack animation notify opens, `WeaponTrailComponent` starts the selected trail, then the notify closes and stops the trail.

## 5. Confirmed Hit VFX

Player hit VFX must happen after confirmed damage, not at attack start.

Current path:

```text
PlayerMeleeAttackSubsystem
-> queued hit timing or AnimNotify_PlayerMeleeHit
-> cone hit detects enemy
-> HealthComponent::ApplyDamage succeeds
-> spawn hit blood and hit distortion VFX
```

Current VFX:

| VFX | Current Asset / Source |
|---|---|
| Blood | random `NS_PreShadedBlood_1` through `NS_PreShadedBlood_10` from `/Game/sA_SplatterofbloodFx/Fx/PreShaded_Blood_Alternatives` |
| Hit distortion | `/Game/SlashTrail_SoftTofu/Niagara/Distortion_Only/NS_Hit_Distortion` |
| Lightning hit burst | `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Hit_Lightning_once` |
| Chain Lightning arc | `/Game/EnergyBeam/NS/NS_ChainLightning_Controlled` |
| Chain Lightning target hit | `/Game/EnergyBeam/NS/NS_Hit_Constant_Thunder` |

Rules:

- Hit VFX is presentation-only.
- Hit VFX must not decide hit detection, damage, knockback, hit reaction, stun, Lightning, bleed, camera shake, or death.
- Current hit location is approximated on the enemy body side facing the player.
- While the player combat element is `Lightning`, confirmed direct melee hits use the Lightning hit burst instead of the neutral hit distortion.
- If exact hit zones become important, move placement toward a dedicated hit-impact owner using sockets, traces, or future contact data.

## 6. Ground Spark Rules

Ground sparks are animation-timed, world-space presentation effects.

Current notifies:

| Class | Use |
|---|---|
| `AnimNotify_GroundImpactVfx` | One-frame ground impact spark |
| `AnimNotifyState_GroundContactImpactVfx` | Contact window that waits until weapon trace point is close enough to the floor |
| `AnimNotifyState_GroundScrapeVfx` | Short scrape window that repeatedly emits floor sparks |

Current default spark Niagara:

`/Game/MechanicalDamageFX/FX/NS_Sparks_Small`

Current trace point:

`WeaponVfx_GroundContact` SceneComponent on `BP_PlayerCharacter`, preferably attached under `EquippedOtachi`.

Rules:

- Prefer contact-window notify state when the exact contact frame is hard to tune.
- Spawned spark components should behave as world effects after spawning.
- Do not keep scrape sparks attached to the character or weapon.
- On shutdown, stop emission first and allow particles to fade before forced cleanup.
- Existing impact/contact sparks should keep playing and naturally fade out.
- If a scrape window is interrupted, keep emission alive until the minimum lifetime is satisfied, then let particles fade.
- Do not auto-edit `/Game/Weapon/Katana/Meshes/StaticMeshes/SM_Otachi_1` to add sockets. A previous automated socket test caused Static Mesh Editor crashes.

## 7. Lightning VFX Rules

Lightning is the first planned elemental VFX branch for the Katana build.

Current implemented pieces:

- `PlayerStatsComponent` owns runtime combat element state: `None` / `Lightning`.
- L toggles the debug combat element.
- `WeaponTrailComponent` listens to the combat element state.
- While Lightning is active, `WeaponTrailComponent` attaches `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning` to `EquippedOtachi`.
- While Lightning is active, attack trail notifies use `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_SlashTrail_Lightning` instead of the normal `ComboTrailSlots` trail system.
- Lightning movement afterimage/trail presentation is intentionally removed for now because the tested SlashTrail/Aura assets looked awkward during movement.
- While Lightning is active, movement should not spawn or boost a separate weapon VFX. Keep the always-on sword aura and attack trail only.
- Runtime code writes the temporary Lightning palette `(0.38, 0.68, 1.0, 1.0)` into `User.Color`, `User.Tint`, `User.LightningColor`, and `User.AuraColor`.
- When Chain Lightning damage is successfully applied, `PlayerMeleeAttackSubsystem` spawns `/Game/EnergyBeam/NS/NS_ChainLightning_Controlled` from the source enemy toward the chained target enemy.
- Chain Lightning VFX now treats each transfer segment as a separate world-space Beam component.
- The current Lightning MVP can queue up to 2 transfer segments per confirmed melee attack, such as A -> B and B -> C.
- Future multi-chain rewards can raise this segment count, such as A -> B, B -> C, and C -> D.
- Each queued segment is delayed by a short 0.03-0.06 second interval so the transfer reads as a rapid electric jump instead of a single instant line.
- Beam components are spawned with `SpawnSystemAtLocation`, not attached to the player or enemies.
- Start/end points are passed as world-space `User.BeamStart` and `User.BeamEnd` before the Niagara component is activated.
- Start/end points currently use `Enemy Actor Location + Z Offset`.
- If the directly hit enemy dies from the confirming melee hit, Chain Lightning can still start from the saved hit-time source position.
- Chain Lightning target hit VFX is separated from the Beam. It is spawned only after chain damage is applied, attached to the target enemy mesh when available, falls back to the root component, and uses a short lifetime.
- Runtime code passes world-space endpoints, but the current controlled Energy Beam copy still also receives a dynamic `User._Length` fallback because the source asset currently behaves like a local-length beam until its Niagara graph is fully rewired.
- `NS_ChainLightning_Controlled` is the project-owned Chain Lightning runtime copy of the original `/Game/EnergyBeam/NS/NS_EletricLightning` reference asset.
- If the blue beam still ignores the green debug line, this is a Niagara graph wiring issue inside `NS_ChainLightning_Controlled`, not a chain target selection issue.
- A final-target sky-to-ground Lightning VFX hook exists in code, but no Niagara asset is assigned yet.
- When several enemies are directly hit, the current first chain source is chosen as the directly hit enemy that forms the nearest valid source-target chain pair, not the first actor returned by iteration order.
- A short green debug line can be drawn between the intended source and target endpoints for PIE tuning.
- This debug line is controlled by `PlayerStatsDataAsset.bShowChainLightningDebug` / `PlayerStatsComponent::ShouldShowChainLightningDebug()` and should stay off during normal playtesting.

Pending editor check:

- `NS_Sword_Lightning` visible color is not confirmed to follow those runtime user parameters.
- If the aura appears but color does not change, inspect the Niagara graph and connect `User.Color` or `User.LightningColor` into `Particles.Color` or the Dynamic Material Parameter color path.
- Treat this as Niagara graph wiring pending, not as a code bug.

Lightning VFX ownership boundaries:

| VFX Purpose | Preferred Owner |
|---|---|
| Sword aura | `WeaponTrailComponent` |
| Confirmed electric hit burst | Future `WeaponImpactVfxComponent` |
| Chain line or arc between enemies | `PlayerMeleeAttackSubsystem` for the current MVP; future elemental VFX request path later |
| Chain target electric hit | `PlayerMeleeAttackSubsystem` for the current MVP; future enemy/status VFX path later |
| Shock mark on enemy body | `EnemyBase` attaches `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning` during the current 2 second Shock MVP |
| Prism-scale lightning burst | Future reward/VFX profile data |

Do not fake Chain Lightning only by stretching the sword trail. Chain Lightning VFX should be requested after gameplay confirms a valid hit, valid chain target, and applied chain damage.

Current Chain Lightning arc is an MVP visual connection. Final color, beam width, lifetime, endpoint point, local-length fallback, final-strike asset, and Niagara `User.BeamStart` / `User.BeamEnd` graph wiring in `NS_ChainLightning_Controlled` still require PIE testing.

Required Niagara graph contract for a production Chain Lightning beam:

- The system or emitter must define `User.BeamStart` and `User.BeamEnd`.
- The beam must use world-space positions.
- Local Space should be disabled for the beam emitter.
- The actual Beam Start and Beam End inputs must read from `User.BeamStart` and `User.BeamEnd`.
- Fixed-length beam, ribbon, or mesh effects that ignore those parameters should not be used as the main chain connector.

## 8. Future WeaponImpactVfxComponent Direction

`WeaponImpactVfxComponent` is a design target, not implemented yet.

It should own contact bursts, sparks, blood, slash impacts, Lightning contact bursts, or bleed contact effects after an attack confirms a hit.

Recommended request data:

- attacker
- hit target
- hit location / hit normal
- combo step or attack id
- reward/build tags
- profile id or Niagara system

Non-responsibility:

- Do not calculate attack hit detection.
- Do not apply damage or status effects.
- Do not mutate `PlayerStatsDataAsset` or enemy stat data.
- Do not play or time weapon trails.
- Do not own enemy hit reaction animations.
- Do not own camera shake directly; it may emit a camera feedback id later.

## 9. Current Non-Goals

- Do not restore the old generic combo slash VFX path.
- Do not put VFX asset references into `PlayerStatsDataAsset`.
- Do not treat a Fab demo Blueprint as final player-weapon VFX without editor tuning.
- Do not add generic white hit-flash back into `EnemyBase`.
- Do not assume Niagara scale, location, rotation, or socket alignment is correct without visual PIE testing.
- Do not merge weapon trail playback, hit-impact playback, target hit reaction, and camera feedback into one component.

## 10. Related Documents

- `Docs/ElementalRewardSystem.md`: Lightning build, Chain Lightning gameplay, Prism direction.
- `Docs/AnimationSystem.md`: animation notifies, Montages, timing ownership.
- `Docs/BasicAttackSystem.md`: basic combo attack behavior.
- `Docs/AttackSkillSystem.md`: attack/skill reward direction.
- `Docs/RewardSystem.md`: reward-driven VFX growth direction.
- `Docs/EnemyAnimationRetargeting.md`: enemy animation asset constraints.
