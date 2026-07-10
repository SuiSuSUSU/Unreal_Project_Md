---
Status: REFERENCE
Scope: VFX System History
Last Updated: 2026-07-06
Source of Truth: No
---

# VFXSystemHistory.md

This archive preserves the previous long-form VFX implementation notes and dated update history.

Do not use this file as the current implementation standard unless explicitly requested.

For current VFX rules, read `Docs/VFXSystem.md`.

## 1. Document Purpose

This document defines the current VFX ownership and direction.

Use this document before changing attack trails, sword waves, hit feedback VFX, reward-driven VFX growth, Niagara setup, or animation-timed visual effects.

## 2. Current VFX Direction

Combat VFX should support readability and build identity.

The player should eventually see that the build is changing through attack presentation, such as larger trails, sword waves, lightning chains, bleed effects, or dodge-triggered effects.

VFX should not be hardcoded into stat data. Stats own numbers; presentation systems own visuals.

## 3. Current Implementation

- The old basic combo slash VFX path was removed.
- `WeaponTrailComponent` is the prepared owner for player sword-trail presentation.
- `AnimNotifyState_WeaponTrail` is the prepared animation-side timing hook.
- `PlayerMeleeAttackSubsystem` calls the trail component for attack-start fallback playback.
- When `bUseAnimNotifyTrailTiming` is enabled, attack animations should control trail start/stop through AnimNotify windows.
- `RewardSelectionWidget` can notify `WeaponTrailComponent` when reward cards are selected.
- Reward-driven trail growth currently has a prepared hook only; final reward-specific VFX profiles are not implemented.
- `BP_PlayerCharacter.WeaponTrailComponent.ComboTrailSlots` currently use `/Game/SlashTrail_SoftTofu/Niagara/Distortion_Only/NS_SlashTrail_Distortion_Only` for the player sword trail.
- The current sword trail was adopted from the SlashTrail SoftTofu Kwang demo reference, specifically the trail used by `/Game/SlashTrail_SoftTofu/Demo/ParagonKwang/Characters/Heroes/Kwang/Animations/Ability_R_Distortion_Only`.
- The trail asset selection lives in `ComboTrailSlots`; the start/end timing should still be controlled by `AnimNotifyState_WeaponTrail` windows on the player attack animations.
- `WeaponTrailComponent.ComboTrailSlots` can optionally override a Niagara user color parameter, defaulting to `User.Color`. This is the preferred way to tune player weapon-trail palette variants such as P_Trail-like lightning color without directly modifying third-party Niagara assets.
- `WeaponTrailComponent` also owns the current element weapon aura. When the player combat element is `Lightning`, it attaches `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning` to `EquippedOtachi` as a sword aura. The current runtime aura writes the temporary Lightning palette `(0.38, 0.68, 1.0, 1.0)` into several Niagara user color candidates such as `User.Color`, `User.Tint`, `User.LightningColor`, and `User.AuraColor`. If the visible asset color still does not change, the Niagara system itself must connect one of those user parameters into `Particles.Color` or its dynamic material color path inside the Niagara editor. When the element returns to `None`, the aura is removed.
- `AnimNotify_GroundImpactVfx` is implemented for one-frame animation-timed ground spark VFX.
- `AnimNotifyState_GroundContactImpactVfx` is implemented for a short contact window that waits until the weapon trace point is close enough to the floor before spawning one impact spark.
- `AnimNotifyState_GroundScrapeVfx` is implemented for short scrape windows where a blade drags across the floor.
- The default ground spark Niagara is `/Game/MechanicalDamageFX/FX/NS_Sparks_Small`.
- Ground spark VFX is presentation-only. It does not apply damage, camera shake, stagger, Lightning, or shockwave gameplay.
- Ground spark timing should prefer the contact-window notify state when the exact contact frame is hard to tune.
- Ground spark position now uses a player Blueprint trace-point component named `WeaponVfx_GroundContact`, instead of editing the weapon static mesh asset directly.
- Ground spark Niagara components are spawned as world-space presentation effects. Animation notifies decide when to emit sparks, but spawned sparks should not remain visually attached to the player animation after they appear.
- Ground spark shutdown should stop emission first and allow existing particles to fade before forced component cleanup. Do not destroy spark components immediately unless a specific effect is known to require it.
- Scrape sparks should not be attached to the character or weapon after spawn. If a scrape effect needs direction, use the weapon trace point's world-space movement direction as input rather than keeping the spawned effect attached.
- Already spawned Impact / Contact spark effects should keep playing and naturally fade out.
- If an active Scrape window is interrupted, stop new emission first, keep the spark alive until at least the minimum emission time has passed, allow remaining particles to fade, then force cleanup later.
- `BladeTipTracePoint` remains useful as a blade-tip marker or legacy fallback, but ground spark placement should prefer `WeaponVfx_GroundContact`.
- If `WeaponVfx_GroundContact` is missing, the current C++ notify can create a runtime-only trace point under `EquippedOtachi` using a tunable relative transform. This does not modify Blueprint or Static Mesh assets.
- Spark reflection/bloom readability is currently tuned through `/Game/MechanicalDamageFX/Materials/M_Sparks` and the unbound `PP_SparkReflectionTuning` PostProcessVolume in `/Game/TopDown/Lvl_TopDown`.

## 2026-07-04 Update: Player Hit Blood VFX MVP

Player melee attacks now spawn a small blood Niagara when damage is actually applied to an enemy.

Implemented path:

```text
PlayerMeleeAttackSubsystem
-> DamageEnemiesInAttackCone
-> HealthComponent::ApplyDamage succeeds
-> spawn one random NS_PreShadedBlood_* Niagara near the enemy body
```

Current source folder:

`/Game/sA_SplatterofbloodFx/Fx/PreShaded_Blood_Alternatives`

Current rule:

- Blood VFX is confirmed-hit presentation only.
- Blood VFX must not decide hit detection, damage, knockback, hit react, or enemy death.
- Current MVP picks randomly from `NS_PreShadedBlood_1` through `NS_PreShadedBlood_10`.
- Spawn location is approximated on the enemy body surface facing the player, using the enemy mesh bounds/capsule radius.
- If a later enemy needs custom hit sockets or hit-zone-specific blood, add a separate hit-impact/VFX component instead of expanding basic attack damage logic too far.

## 2026-07-05 Update: Player Hit Distortion VFX MVP

Player melee attacks now also spawn a short hit-distortion Niagara when damage is actually applied to an enemy.

Implemented path:

```text
PlayerMeleeAttackSubsystem
-> DamageEnemiesInAttackCone
-> HealthComponent::ApplyDamage succeeds
-> spawn NS_Hit_Distortion near the enemy body impact side
```

Current Niagara:

`/Game/SlashTrail_SoftTofu/Niagara/Distortion_Only/NS_Hit_Distortion`

Current rule:

- Hit distortion is confirmed-hit presentation only.
- It does not apply damage, knockback, hit reaction, stun, Lightning, bleed, camera shake, or enemy death.
- Player melee damage is now delayed until the queued hit timing instead of firing immediately when the attack starts.
- The fallback hit timing uses a short combo-based delay, but exact timing should use `AnimNotify_PlayerMeleeHit` on the attack animation.
- Blood VFX and hit-distortion VFX are spawned from the same confirmed-hit timing path so they stay aligned with the hit frame.
- Current basic attack hit detection does not provide an exact bone/socket hit point, so the spawn point is approximated on the enemy body side facing the player.
- If exact hit zones become important, move hit-impact placement toward a dedicated hit-impact VFX owner that can use sockets, traces, or future attack contact data.

## 2026-07-06 Update: Lightning Sword Aura Color Pending

`NS_Sword_Lightning` is currently attached to the sword during the debug Lightning element state, but its visible color is not yet confirmed to follow runtime user color parameters.

Current confirmed state:

```text
WeaponTrailComponent
-> attaches NS_Sword_Lightning to EquippedOtachi
-> writes Lightning color candidates at runtime
   - User.Color
   - User.Tint
   - User.LightningColor
   - User.AuraColor
```

Current limitation:

- The Niagara asset appears to keep using its own internal color or material color path.
- Changing runtime user parameters is not enough unless the Niagara system connects one of those user parameters to `Particles.Color` or Dynamic Material Parameters.
- Do not keep spending code time on this color issue until the Niagara graph can be visually inspected in the editor.

Next editor-side check:

1. Open `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning`.
2. Inspect the `Sword` and `Lightning` emitters.
3. Find the color-setting modules such as `Initialize Particle`, `NMS_Color_Core`, or `Dynamic Material Parameters`.
4. Connect `User.Color` or `User.LightningColor` into the actual visible color path.
5. Save and test the F9 debug Lightning state in PIE.

Until this is done, treat Lightning sword aura color as visually pending, not as a code bug.

## 2026-07-05 Update: Ground Spark World-Space Shutdown Rule

Ground sparks should behave like floor/world effects after they are spawned, even though animation notifies decide when to start them.

Current rule:

```text
Impact / Contact spark already spawned
-> keep playing
-> stop emission only after its configured lifetime
-> let remaining particles fade
-> force cleanup after a short safety delay

Scrape spark while the animation window is active
-> repeatedly sample WeaponVfx_GroundContact
-> spawn sparks in world space
-> do not attach spawned sparks to the character or weapon
-> if needed, use world-space trace-point movement direction only

Scrape window interrupted or ended
-> stop spawning new sparks
-> do not immediately destroy existing sparks
-> keep emission until the minimum lifetime is satisfied
-> let remaining particles fade
-> force cleanup after 1 second
```

Current minimum Scrape emission lifetime:

| Setting | Current Rule |
|---|---|
| Minimum scrape emission lifetime | 0.14 seconds |
| Forced cleanup delay after emission stop | 1.0 second |

## 2026-07-02 Update: Ground Impact Spark MVP

Ground impact spark playback now has a small C++ AnimNotify path.

Implemented path:

```text
Attack animation frame
-> AnimNotify_GroundImpactVfx
-> trace downward from WeaponVfx_GroundContact
-> spawn NS_Sparks_Continuous at ground hit
-> stop emission after a short lifetime
-> allow remaining particles to fade before forced cleanup
```

Current default setup:

| Setting | Default |
|---|---|
| Niagara | `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` |
| Trace point component | `WeaponVfx_GroundContact` by default |
| Runtime trace point parent | `EquippedOtachi` |
| Runtime trace point relative location | `115, 0, 2` |
| Deprecated socket fallback | `EquippedOtachi` / `BladeTip`, only if Trace Point Component Name is None |
| Trace distance | 320 |
| Effect lifetime | 0.25 seconds |

Editor tuning rule:

- Place the notify on the exact frame where the weapon visually touches the ground.
- If the exact contact frame is hard to place, prefer `Ground Contact Impact VFX` instead.
- Add a SceneComponent such as `WeaponVfx_GroundContact` under `EquippedOtachi` in `BP_PlayerCharacter`, then adjust its position visually against the weapon-ground contact point in animation.
- Do not use fixed attack-start delays for this effect.
- Do not use owner-local fallback offsets for final spark placement.
- If the effect looks too continuous or too long, lower `EffectLifetime` or try another Niagara from the same pack.

## 2026-07-04 Update: Ground Contact Impact Window

`AnimNotifyState_GroundContactImpactVfx` is the preferred path when a one-frame ground impact notify is too hard to align.

Implemented path:

```text
Attack animation contact window
-> AnimNotifyState_GroundContactImpactVfx starts
-> trace downward from WeaponVfx_GroundContact every tick
-> wait until trace point is close enough to the floor hit
-> spawn one NS_Sparks_Small impact
-> stop checking after the first spawn or when the window ends
```

Current default setup:

| Setting | Default |
|---|---|
| Niagara | `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` |
| Trace point component | `WeaponVfx_GroundContact` |
| Runtime trace point parent | `EquippedOtachi` |
| Runtime trace point relative location | `115, 0, 2` |
| Trace distance | 320 |
| Max contact distance | 45 |
| Effect lifetime | 0.25 seconds |

Editor tuning rule:

- Use a NotifyState window that begins shortly before the blade should touch the floor and ends shortly after the contact moment.
- Do not make the window cover the whole attack unless debugging.
- Green debug sphere means the contact condition was met and the spark can spawn.
- Orange debug sphere means the trace hit the floor, but the trace point is still farther than `MaxContactDistance`.
- If the window ends without a spark, either move/extend the notify-state window, adjust `WeaponVfx_GroundContact`, or slightly raise `MaxContactDistance`.
- Keep `Ground Impact VFX` only when a single exact contact frame is easy to identify.

## 2026-07-02 Update: Ground Scrape Spark MVP

Ground scrape spark playback now has a C++ AnimNotifyState path for effects that should appear repeatedly while the blade scrapes the floor.

Implemented path:

```text
Attack animation scrape window
-> AnimNotifyState_GroundScrapeVfx
-> repeatedly trace downward from WeaponVfx_GroundContact while the notify window is open
-> spawn NS_Sparks_Small at each ground hit
-> stop emission after each spark lifetime
-> allow remaining particles to fade before forced cleanup
```

Current default setup:

| Setting | Default |
|---|---|
| Niagara | `/Game/MechanicalDamageFX/FX/NS_Sparks_Small` |
| Trace point component | `WeaponVfx_GroundContact` by default |
| Runtime trace point parent | `EquippedOtachi` |
| Runtime trace point relative location | `115, 0, 2` |
| Deprecated socket fallback | `EquippedOtachi` / `BladeTip`, only if Trace Point Component Name is None |
| Spawn interval | 0.08 seconds |
| Effect lifetime | 0.07 seconds |
| Effect scale | 0.45 |
| Jitter radius | 2.5 |

Editor tuning rule:

- Use `Ground Impact VFX` for one contact burst.
- Use `Ground Scrape VFX` for a short range on the timeline where the blade visually drags across the ground.
- Make the notify-state window cover only the scrape frames, not the whole attack animation.
- Tune `Trace Point Component Name`, `Spawn Interval`, `Effect Lifetime`, `Effect Scale`, and `Rotation Offset` per animation.
- If sparks appear in one spot, verify the trace-point component is attached under the moving weapon component and not to a static character/bone position.
- If sparks look too noisy, increase `Spawn Interval` or lower `Effect Lifetime`.
- Even if a scrape notify uses a very short `EffectLifetime`, the runtime keeps a small minimum spark emission lifetime so animation cuts do not hard-cut the visible particles.

## 2026-07-03 Update: Weapon Trace-Point Ground Spark Trace

The owner-local trace offset path and direct static-mesh socket editing path are no longer the preferred ground spark placement paths.

Current rule:

```text
Attack animation notify
-> find WeaponVfx_GroundContact scene component on the player actor
-> trace downward from that component location
-> spawn spark at the floor hit point
```

Default editor setup:

| Field | Default |
|---|---|
| Trace Point Component Name | `WeaponVfx_GroundContact` |
| Create Missing Trace Point At Runtime | `true` |
| Runtime Trace Point Parent Component Name | `EquippedOtachi` |
| Runtime Trace Point Relative Location | `115, 0, 2` |
| Deprecated Trace Component Name | `EquippedOtachi` |
| Deprecated Trace Socket Name | `BladeTip` |

`WeaponVfx_GroundContact` should be a SceneComponent on `BP_PlayerCharacter`, attached under `EquippedOtachi` and placed near the weapon-ground contact point for the current attack animation. If it does not exist, the notify creates a runtime-only component under `EquippedOtachi` with the relative transform above. If a different contact point is needed for a specific animation, add another trace-point component such as `WeaponVfx_EdgeMid`, `WeaponVfx_Tip`, or `WeaponVfx_GroundContact_Heavy` and set the notify to that component.

Current setup note:

- Do not auto-edit `/Game/Weapon/Katana/Meshes/StaticMeshes/SM_Otachi_1` to add sockets. An automated socket test caused Static Mesh Editor crashes and was reverted.
- Use Blueprint-owned or runtime-created trace-point components for this MVP path.
- Runtime lookup accepts the Blueprint component instance name or its generated variable-style name, such as `WeaponVfx_GroundContact_GEN_VARIABLE`, before falling back to a runtime-created trace point.
- For backward compatibility, old notify instances that still request `BladeTipTracePoint` will use `WeaponVfx_GroundContact` first if it exists, then fall back to `BladeTipTracePoint`.
- While `bLogMissingSetup` is enabled, ground spark notifies log the trace point, hit actor/component, impact point, and final spawn location to help diagnose misplaced sparks.
- `bDrawTraceDebug` is disabled by default. Enable it only while tuning trace placement.
- While `bDrawTraceDebug` is enabled, ground spark notifies draw a red sphere at the trace point, a green sphere at the floor hit point, and a yellow line between them. This is for visual placement tuning and should be disabled after the marker is verified.

If the trace-point component cannot be found and runtime creation is disabled or fails, the notify should not spawn sparks. This is intentional; inaccurate owner-local fallback placement should not hide a missing setup.

## 2026-07-03 Update: Spark Reflection Tuning

Runtime spark readability can be affected by bloom, Lumen reflections, and shiny floor materials.

Current tuning:

| Area | Current Rule |
|---|---|
| Spark material | `/Game/MechanicalDamageFX/Materials/M_Sparks` should not cast ray traced shadows or contribute dynamic emissive area lighting. |
| Map post process | `/Game/TopDown/Lvl_TopDown` contains `PP_SparkReflectionTuning` as an unbound tuning volume. |
| Bloom | Lowered for spark readability. |
| Reflection | Screen-space and Lumen reflection intensity/quality are reduced for the current test map. |
| Motion blur | Disabled in the tuning volume while evaluating fast attack VFX. |

This is a visual tuning layer for the current map, not a final art direction. If the floor material changes later, prefer tuning the actual floor material roughness/specular values before making the spark Niagara weaker.

## 2026-07-02 Update: Lightning VFX Planning Boundary

Lightning is the first planned elemental VFX branch for the Katana build.

This is a planning direction, not an implemented visual system.

Lightning VFX should be separated by purpose:

| VFX Purpose | Preferred Owner |
|---|---|
| Sword trail shape while the weapon moves | `WeaponTrailComponent` |
| Confirmed hit contact burst | Future `WeaponImpactVfxComponent` |
| Chain line/arc between enemies | Future elemental VFX request path |
| Shock mark on enemy body | Enemy-specific presentation or future status VFX path |
| Prism-scale lightning burst or storm effect | Future reward/VFX profile data |

Important boundary:

- Chain Lightning should be requested after gameplay confirms a valid hit and valid chain target.
- Do not fake Chain Lightning only by stretching the sword trail.
- Do not hardcode Lightning Niagara assets into `PlayerStatsDataAsset`.
- Do not merge weapon trail timing, hit impact bursts, target Shock marks, and chain arcs into one component.
- Early implementation may use logs or temporary VFX until the chain targeting and damage feel good.

First visual test candidates:

1. Small electric hit burst on the first struck enemy.
2. Simple line or arc to one nearby chained enemy.
3. Slightly larger 3rd-hit Chain Lightning effect.
4. Shock mark placeholder on affected enemies.
5. `폭풍검` Prism-scale lightning strike/explosion after core gameplay works.

## 2026-07-02 Update: WeaponImpactVfxComponent Structure Design

This section is a design target only. Do not treat `WeaponImpactVfxComponent` as implemented until a C++ component and actual editor-tuned VFX settings exist.

`WeaponImpactVfxComponent` should own hit-impact presentation for player weapon attacks. It should not replace `WeaponTrailComponent`; the trail is the moving weapon/sword-arc effect, while the impact VFX is the burst, spark, blood, slash contact, lightning contact, or ground contact effect that appears when an attack actually connects.

Recommended responsibility:

| Area | Example Fields / Inputs |
|---|---|
| Impact request | attacker, hit target, hit location, hit normal, combo step, attack id |
| VFX selection | impact profile id, Niagara system, target type, surface type, reward/build tags |
| Spawn transform | location mode, rotation mode, relative offset, relative scale |
| Attachment | world-space spawn, attach to target, attach to socket, detach after spawn |
| Runtime tuning | scale multiplier, color/intensity params, max effects per attack, debug toggle |
| Presentation handoff | optional sound id, camera feedback id, decal id as later hooks only |

Recommended non-responsibility:

- Do not calculate attack hit detection.
- Do not apply damage or status effects.
- Do not mutate `PlayerStatsDataAsset` or enemy stat data.
- Do not play or time weapon trails; `WeaponTrailComponent` and `AnimNotifyState_WeaponTrail` own that path.
- Do not own enemy hit reaction animations. Hit reaction should stay enemy-specific or move into a future `HitReact` system.
- Do not reintroduce the removed generic enemy white hit-flash path.
- Do not directly own camera shake logic. It may emit or reference a camera feedback id later.

Recommended data candidates:

| Candidate | Purpose |
|---|---|
| `FWeaponImpactVfxRequest` | Runtime request generated when an attack confirms a hit. |
| `FWeaponImpactVfxProfile` | Editor-facing impact VFX setup for a combo step, reward tag, target type, or surface type. |
| `EWeaponImpactLocationMode` | Use target actor location, hit point, target mesh socket, or ground trace point. |
| `EWeaponImpactRotationMode` | Face attack direction, face hit normal, face camera, or use fixed profile rotation. |
| `EWeaponImpactAttachMode` | Spawn in world, attach to hit target, attach to target socket, or attach then detach. |

Recommended first implementation path:

1. Keep all current damage and cone-hit logic in `PlayerMeleeAttackSubsystem`.
2. After a confirmed enemy hit, create a small impact request containing target, approximate hit location, attack direction, and combo step.
3. Let `WeaponImpactVfxComponent` choose and spawn one configured impact Niagara effect.
4. Keep the component disabled or asset-empty by default so no visual behavior changes until editor tuning.
5. Add reward/build tags later, after a real reward effect needs a different impact style.
6. Add ground-spark support later through animation notifies or a ground trace request, not through a fixed attack delay.

First useful VFX cases:

| Case | Example |
|---|---|
| Basic enemy hit | Small slash contact burst on enemy body. |
| Heavy third hit | Larger impact burst or short spark at the contact point. |
| Ground contact | Spark/dust when a heavy animation notify says the weapon touches the floor. |
| Lightning reward | Chain/lightning contact VFX after the damage system confirms a valid target. |
| Bleed reward | Blood/bleed contact VFX when a bleed-applying reward is active. |

## 2026-07-02 Update: HitReact and VFX Boundary

`HitReact` and impact VFX should be coordinated but not merged.

- `WeaponImpactVfxComponent` owns the visual effect at the contact point.
- `HitReact` owns the target body's response, such as flinch, stagger, brief lock, or immunity.
- A single confirmed hit may request both impact VFX and hit reaction.
- Either side may be disabled independently. For example, a boss can show impact sparks while ignoring flinch.
- Do not use generic white flashes as a replacement for hit reaction or impact VFX.
- Camera shake, sound, and decals are later feedback hooks and should not be hidden inside `HitReact`.

## 4. Weapon Trail Rule

Formal weapon-trail timing should come from animation notifies when possible.

```text
Attack animation
-> AnimNotifyState_WeaponTrail window opens
-> WeaponTrailComponent starts selected combo trail
-> Notify window closes
-> WeaponTrailComponent stops trail
```

Fallback `ActivationDelay` and `Duration` values are only for testing when notify timing is disabled.

## 5. VFX Ownership Boundaries

| Area | Owner |
|---|---|
| Player weapon trail slots and Niagara assignment | `WeaponTrailComponent` |
| Exact animation timing for trail playback | `AnimNotifyState_WeaponTrail` |
| Combo attack gameplay timing and damage | `PlayerMeleeAttackSubsystem` |
| Runtime stat modifiers that may influence VFX scale/intensity | `PlayerStatsComponent` / reward runtime state |
| Reward-specific VFX identity | Future reward/VFX profile data |
| Player weapon hit-impact VFX | Future `WeaponImpactVfxComponent` |
| Target body hit reaction | Enemy Blueprint, AnimBP, or future `HitReactComponent` |
| Enemy-specific hit/death/spawn VFX | Enemy Blueprint, AnimBP, or future enemy presentation component |

## 6. Current Non-Goals

- Do not restore the old generic combo slash VFX path.
- Do not put VFX asset references into `PlayerStatsDataAsset`.
- Do not treat a Fab demo Blueprint as final player-weapon VFX without editor tuning.
- Do not add generic white hit-flash back into `EnemyBase`.
- Do not assume Niagara scale, location, rotation, or socket alignment is correct without visual PIE testing.
- Do not merge weapon trail playback and hit-impact playback into one component.
- Do not merge target body hit reaction into weapon impact VFX.

## 7. Future VFX Candidates

- 3rd combo sword wave.
- Wider 3rd combo trail after reward.
- Lightning chain on hit.
- Bleed hit effect.
- Dodge afterimage explosion.
- Ground spark when a heavy weapon hits the floor. First MVP notify path is implemented; final timing and visual tuning remain editor-side.
- Enemy spawn VFX.
- Enemy death dissolve or burst.
- Camera shake on heavy hit.

These are candidates until assets, timing, and gameplay hooks are verified.

## 8. Related Documents

- `Docs/AnimationSystem.md`: animation notifies, Montages, and timing ownership.
- `Docs/BasicAttackSystem.md`: basic combo attack behavior.
- `Docs/AttackSkillSystem.md`: attack/skill reward direction.
- `Docs/RewardSystem.md`: reward-driven VFX growth direction.
- `Docs/EnemyAnimationRetargeting.md`: enemy animation asset constraints.
