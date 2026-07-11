---
Status: ACTIVE
Scope: Elemental Reward System
Last Updated: 2026-07-06
Source of Truth: Yes
---

# ElementalRewardSystem.md

## 1. Document Purpose

This document defines the current planning direction for elemental build rewards.

Use this document before changing element rewards, elemental synergy, Prism unlock rules, Lightning rewards, or weapon/element build identity.

This is a design source of truth, not an implementation claim. Check `Docs/CurrentImplementation.md` before assuming any element reward is implemented.

## 2. Core Direction

The game should let the player's attack style be shaped mainly by two large axes:

| Axis | Meaning |
|---|---|
| Weapon | How the player attacks: rhythm, range, combo shape, speed, risk, and feel. |
| Element | What extra rules are attached to attacks: chain, bleed, lifesteal, slow, explosion, shadow, or other build effects. |

Current first axis:

```text
Weapon: Katana
First Element: Lightning
First Build Focus: confirmed melee hit + small Lightning chain
```

Do not build every element at once. The first goal is to make one element feel good, then use that pattern for later elements.

## 3. Reward Philosophy

Reward choice should create tension between safe growth and high-risk high-ceiling growth.

| Reward Style | Meaning | Example |
|---|---|---|
| Stable | Always useful, raises the low point. | Attack damage, max health, dodge cooldown reduction. |
| Scaling | Starts small but grows over the Run. | More Lightning chance, higher chain count, technique level growth. |
| Synergy | Strong when combined with previous choices. | Lightning chain from sword wave, Shock spread after kill. |
| Gamble | Risky or conditional but can create a high ceiling. | Prism chance up, high-risk Stage, chance-based double chain. |
| Prism | Rare transformative reward that changes the build's shape. | Storm Sword, lightning afterimage, large electric shockwave. |

Good reward choices should ask the player:

- Do I take the safe card now?
- Do I commit deeper into Lightning?
- Do I gamble for a high-grade or Prism payoff?
- Do I pick a Stage that fits my current build?

## 4. Lightning Build Identity

Lightning should feel like fast spread, chain damage, and high-roll moments.

Core Lightning verbs:

- Chain to nearby enemies.
- Mark or briefly Shock enemies.
- Trigger from confirmed attack hits.
- Later rewards can make 3rd combo hits stronger.
- Connect dodge into the next attack.
- Turn late-run hits into dramatic multi-target bursts.

Initial growth flow:

```text
Early:
Any confirmed melee hit can chain weak Lightning through up to 2 nearby enemies for the current test baseline.

Mid:
Chain count increases.
Short Shock marks are added.
Dodge can empower the next lightning attack.
3rd combo hits can gain higher chain count, higher damage, or special finish effects.

Late:
Shocked enemy kills spread Shock.
Last chain target can receive lightning strike damage.
Prism rewards complete the Storm Sword fantasy.
```

## 5. Lightning Reward Candidates

### Bronze / Early Entry

`전기 스파크` was originally discussed as Silver, but it should be treated as an early entry or Bronze-level Lightning starter because it is a simple low-probability small lightning hit.

| Card | Effect |
|---|---|
| 전기 스파크 | On attack hit, low chance to deal small Lightning damage. |

### Silver

| Card | Effect |
|---|---|
| 전류 베기 | On 3rd combo hit, deal Lightning damage to 1 nearby enemy. |
| 번개 전이 | On 3rd combo hit, chain Lightning to 2 nearby enemies. |
| 감전 칼날 | Enemies damaged by Lightning receive a short Shock mark. |
| 과충전 | After dodge, the next attack deals bonus Lightning damage. |

### Gold

| Card | Effect |
|---|---|
| 체인 라이트닝 | On 3rd combo hit, chain Lightning to up to 3-4 enemies. |
| 감전 폭발 | Hitting a Shocked enemy with the 3rd combo hit causes an electric explosion nearby. |
| 전류 순환 | Killing an enemy with Lightning damage reduces dodge cooldown. |
| 번개 검기 | The 3rd combo sword wave becomes Lightning-element. |
| 낙뢰 마무리 | The last target in a Chain Lightning sequence receives extra lightning strike damage. |

### Prism

| Card | Effect |
|---|---|
| 폭풍검 | On 3rd combo hit, Lightning chains to up to 5 enemies, then the last target causes a lightning strike explosion. |
| 뇌신의 잔상 | The afterimage left by dodge causes an electric explosion. |
| 전류 지배 | When a Shocked enemy dies, Shock spreads to nearby enemies. |
| 번개 난무 | Each enemy hit by the 3rd combo sword wave emits small lightning branches. |
| 폭뢰참 | The 3rd combo hit creates a strong electric shockwave at the impact point. |

## 6. First Test Priority

The first implementation should not try to build the entire Lightning tree.

Recommended test order:

| Priority | Card | Reason |
|---|---|---|
| 1 | 체인 라이트닝 | Core identity of the Lightning element. |
| 2 | 전기 스파크 | Simple entry card and easy feedback test. |
| 3 | 과충전 | Connects dodge and attack. |
| 4 | 감전 칼날 | Minimal status mark foundation for later synergy. |
| 5 | 폭풍검 | Representative Prism payoff after the basic chain is fun. |

First MVP target:

```text
Player owns Lightning reward
-> any melee hit confirms damage
-> find up to 2 nearby additional enemies
-> apply each transfer after a short 0.03-0.06 second delay
-> apply small Lightning damage
-> show log or temporary VFX
```

Do not start with:

- Full status-effect framework.
- Elemental resistance/weakness.
- Every Lightning card at once.
- Full Prism reward pool.
- Final Niagara tuning.

## 7. Prism Unlock Rules

Prism should be rare and should feel like a high-ceiling moment.

Important rule:

```text
Unlocked does not mean guaranteed.
Unlocked means the Prism reward can enter the candidate pool.
```

Current Prism unlock candidates:

| Condition | Meaning | Status |
|---|---|---|
| High-risk Stage selected | Choosing a dangerous Stage increases Prism appearance chance. | Adopted planning direction. |
| 3+ rewards from the same element | Owning 3 or more Lightning rewards unlocks Lightning Prism candidates. | Adopted planning direction. |
| Specific skill reaches Lv10 | Reaching a skill/technique milestone unlocks related Prism candidates. | Deferred until skill-level system exists. |

Run target:

- A normal Run may see 0-2 Prism rewards.
- A lucky or high-risk Run may reach 2-3 Prism rewards.
- Prism should not appear so often that it feels like a normal Gold reward.

Future Luck stat direction:

- Luck may increase high-grade or Prism appearance chance.
- Luck should not guarantee Prism by itself.
- Luck should interact with Stage risk, build commitment, and reward pool eligibility.

## 8. Synergy Direction

Lightning should connect with later reward families.

Example future synergies:

| Pairing | Possible Result |
|---|---|
| Lightning + Sword Wave | Sword wave can trigger or carry Lightning chain. |
| Lightning + Dodge | Dodge can overcharge the next attack or leave an electric afterimage. |
| Lightning + Shock | Shocked enemies can explode, spread Shock, or receive bonus chain damage. |
| Lightning + Kill Reward | Lightning kills can reduce dodge cooldown or start another chain. |
| Lightning + Multi-enemy Stage | Chain effects become more valuable against many weak enemies. |

These are candidates until implemented and tested.

## 9. Implementation Boundaries

Current implementation status:

- Player runtime combat element state now has a debug-only `None` / `Lightning` toggle on L.
- This debug state lives on `PlayerStatsComponent` as runtime state and resets with run reward modifiers.
- The current debug element toggle is recorded through logs only. Persistent top-left on-screen debug text is removed.
- `WeaponTrailComponent` listens to that runtime combat element state and attaches `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning` to the player sword while Lightning is active.
- Chain Lightning MVP is implemented as gameplay feedback with temporary VFX. When the player is in Lightning state and a melee attack actually applies damage, `PlayerMeleeAttackSubsystem` can chain small Lightning damage through up to 2 nearby additional active enemies.
- Current Chain Lightning MVP excludes enemies directly hit by the original melee cone, queues at most one chain sequence per attack, delays each transfer by 0.03-0.06 seconds, and can transfer through up to 2 nearby additional active enemies.
- Each transfer spawns a temporary `/Game/EnergyBeam/NS/NS_ChainLightning_Controlled` world Beam after chain damage is applied, plus a short attached target electric hit VFX on the chained target.
- `EnemyBase` now supports Electric Stack and Shock state. Direct Lightning hits add 2 stacks, Chain Lightning hits add 1 stack, 5 stacks trigger a 2 second Shock state, and stacks expire after 4 seconds without more electric hits.
- Shock currently has no extra damage, slow, explosion, propagation, or final Niagara VFX.
- A first temporary Stage Clear Lightning reward card, `run_lightning_current_slash`, is implemented. It enables Lightning state for the current Run and lets the existing Chain Lightning MVP run from confirmed melee hits.
- Prism unlock logic is not implemented yet.
- Chain Lightning reward scaling is not implemented yet.
- Lightning VFX is not final; the current chain arc is an MVP visual.
- Lightning sword aura color is currently pending Niagara editor-side graph wiring. Runtime code already sends color candidates, but the Niagara asset must visually use them before this is treated as final.

Expected ownership when implemented:

| Area | Preferred Owner |
|---|---|
| Reward identity, grade, tags, and prerequisites | Future reward definition data or reward table |
| Stage Clear card timing | `StageFlowManager` |
| Reward card display and selection | `RewardSelectionWidget` |
| Reward effect application | Future `RewardEffectApplier` or reward runtime system |
| Basic attack hit confirmation | `PlayerMeleeAttackSubsystem` |
| Runtime stat or reward state | `PlayerStatsComponent` or future run reward state |
| Chain/impact VFX | Future `WeaponImpactVfxComponent` or elemental VFX component |
| Sword trail playback | `WeaponTrailComponent` |
| Shock target-side reaction | Enemy-specific logic or future status/HitReact system |

## 10. No-Editor Work Mode

When the Unreal Editor cannot be visually checked, avoid work that depends on final appearance, animation timing, or Niagara material tuning.

Good no-editor tasks:

- Define Lightning gameplay rules.
- Prepare small code hooks that can be verified later through logs.
- Document reward/card boundaries.
- Review ownership between `PlayerMeleeAttackSubsystem`, `PlayerStatsComponent`, `WeaponTrailComponent`, and future elemental VFX logic.
- Keep all new visual behavior disabled, placeholder, or log-first until PIE can be checked.

Avoid while away from the editor:

- Final Niagara color tuning.
- Sword aura scale/location/rotation tuning.
- Animation notify timing adjustments.
- Mesh, socket, material, and Blueprint visual placement edits.

Recommended next no-editor implementation target:

```text
Chain Lightning MVP, log-first
-> only when player combat element is Lightning
-> from any confirmed melee hit
-> find up to 2 nearby additional alive enemies
-> delay each transfer by 0.03-0.06 seconds
-> apply small Lightning damage
-> log source target, chained target, and damage
-> leave final chain VFX for editor-side tuning
```

This gives the Lightning build a real gameplay rule without depending on final VFX quality.

## 11. Pending Editor Verification

The following checks are deferred until the Unreal Editor can be opened and tested in PIE.

### Chain Lightning MVP

Test setup:

```text
PIE
-> press L to switch player combat element to Lightning
-> enter a room with at least 2 active enemies
-> hit only 1 enemy directly
-> keep another active enemy nearby but outside the melee cone
-> check Output Log
```

Expected Lightning log:

```text
Debug combat element changed: Lightning
Manual melee combo 1 hit 1 enemies for ...
Chain Lightning triggered: Combo=1 Source=... Target=... Damage=... Applied=...
```

Expected Neutral behavior:

```text
Neutral state
-> melee hit still damages directly hit enemies
-> no Chain Lightning triggered log
```

Expected no-target behavior:

```text
Lightning state
-> direct melee hit succeeds
-> no nearby extra active enemy exists
-> Chain Lightning skipped: ... Reason=No additional active target ...
```

Important test note:

- Enemies directly hit by the original melee cone are excluded from the chain target search.
- For the cleanest test, place or lure one enemy inside the melee cone and one nearby enemy outside the cone.

### Lightning Sword Aura Color

This remains separate from Chain Lightning gameplay.

- Confirm whether the sword aura appears when L switches to Lightning.
- Confirm whether the aura color follows the intended blue palette.
- If the aura appears but color does not change, inspect `NS_Sword_Lightning` in the Niagara editor and connect `User.Color` or `User.LightningColor` into the actual visible color path.

## 12. Related Documents

- `Docs/RewardSystem.md`: reward timing, reward sources, reward grade direction.
- `Docs/AttackSkillSystem.md`: attack, skill, and build-changing reward direction.
- `Docs/VFXSystem.md`: weapon trail, impact VFX, and elemental VFX ownership.
- `Docs/StageRunStructure.md`: Stage Clear timing and high-risk Stage context.
- `Docs/PlayerStatsSystem.md`: runtime reward stat ownership and future Luck stat candidate.
