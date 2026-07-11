---
Status: ACTIVE
Scope: Reward System
Last Updated: 2026-07-06
Source of Truth: Yes
---

# RewardSystem.md

## 1. Document Purpose

This document defines the current reward-system direction and ownership boundaries.

Use this before changing reward cards, reward timing, reward effects, reward UI, reward grades, reward drops, or reward-related player growth.

Detailed dated history before this cleanup was moved to:

- `Docs/Archive/RewardSystemHistory_PreDocRestructure_2026-07-06.md`

## 2. Current Reward Direction

The long-term main reward cadence is Stage Clear based.

```text
Clear all required Rooms in selected Stage
-> Stage Clear
-> Reward card selection
-> Apply selected reward
-> Return to Stage Map
-> Choose one connected next Stage
```

Current rules:

- Main build rewards should be granted at Stage Clear, not after every Room Clear.
- Room Clear should normally open the next Room only.
- The old Room Clear reward flow may remain as a standalone MVP test path for Rooms not owned by `StageFlowManager`.
- Stage Clear rewards should change the current Run's build identity.
- Enemy drop items should handle moment-to-moment sustain or short buffs.
- Permanent upgrades are deferred until save/load, currency, and lobby upgrade UI exist.

## 3. Reward Source Split

The reward system should separate reward source, timing, and duration.

| Reward Source | Timing | Duration | Purpose | Examples |
|---|---|---|---|---|
| Enemy Drop Item | Enemy death or combat event. | Instant or short timed effect. | Sustain and moment-to-moment tempo. | Heal pickup, short attack buff, short move-speed buff. |
| Stage Clear Reward Card | After Stage Clear. | Current Run duration. | Build direction and combat-style growth. | 3rd-hit sword wave, dodge-followup boost, bleed on hit, kill-heal passive. |
| Permanent Upgrade | Outside the current Run. | Permanent after save/load exists. | Long-term account/player growth. | Base attack level, max health level, move speed level. |

Important distinction:

- One-time `Heal +20` belongs long-term to enemy drop / pickup item logic.
- "Heal slightly whenever an enemy is killed" is a Stage Clear build reward.
- A short timed `Attack Up` pickup is an enemy drop item.
- A Run-long attack, combo, dodge, bleed, lightning, or sword-wave rule is a Stage Clear reward card.

## 4. Current Implementation State

| Feature | Current Owner | Status |
|---|---|---|
| Stage Clear reward timing | `StageFlowManager` | Implemented as current MVP timing. |
| Temporary reward carrier | `RewardActor` | Implemented. |
| Temporary reward card UI | `RewardSelectionWidget` | Implemented. |
| Temporary reward structs/enums | `RewardTypes.h` | Implemented. |
| Reward apply type split | `ERewardApplyType` | Instant / Run / Permanent. |
| Runtime Run stat modifiers | `PlayerStatsComponent` | Implemented. |
| Run reward reset | `PlayerHealthSubsystem`, `PlayerStatsComponent` | Implemented on death/restart path. |
| Permanent upgrade persistence | None | Not implemented. |

Current reward data is still temporary. It is not the final reward database.

For Start Stage `CANDIDATE v0.1`, code support exists to request three fixed Stage Clear options by reward id: `run_attack_damage_5`, `run_lightning_current_slash`, and `run_dodge_cooldown_015`. This candidate setup is not considered verified until the Stage reward actor is checked in PIE.

## 5. Current Temporary Rewards

| Current Reward Id | Apply Type | Long-Term Home | Status |
|---|---|---|---|
| `instant_heal_20` | Instant | Enemy drop / pickup item. | Implemented temporary card. |
| `run_max_health_10` | Run | Stage Clear reward. | Implemented temporary card. |
| `run_attack_damage_5` | Run | Stage Clear reward or future timed drop variant. | Implemented temporary card. |
| `run_move_speed_80` | Run | Stage Clear reward or future timed drop variant. | Implemented temporary card. |
| `run_dodge_cooldown_015` | Run | Stage Clear reward. | Implemented temporary card. |
| `run_lightning_current_slash` | Run | Stage Clear Lightning starter reward. | Implemented temporary card; enables Lightning chain MVP for the current Run. |
| Permanent placeholder | Permanent | Future permanent upgrade system. | Placeholder only. |

Guideline:

- Keep hardcoded temporary cards until Stage Clear flow is stable.
- Do not treat current hardcoded card options as the final reward pool.
- Do not mutate `PlayerStatsDataAsset` for temporary Run rewards.
- Runtime changes should go through runtime state such as `PlayerStatsComponent`.

## 6. Reward Apply Types

| Type | Meaning | Current Rule |
|---|---|---|
| Instant | Applied immediately when selected or picked up. | Good for healing and short one-off effects. |
| Run | Applies only during the current Run or current level session. | Reset on death or restart. |
| Permanent | Long-term growth type. | Structure only for now; no save/load or meta progression. |

Current MVP still shows some Instant rewards as card choices. Long-term, Instant sustain should usually move to enemy drop items.

## 7. Reward Grade Direction

Reward grades are a planning direction, not a final implemented data system.

Candidate grade names:

| Grade | Planning Meaning |
|---|---|
| Bronze | Small numeric improvement. |
| Silver | Medium numeric improvement or simple build hook. |
| Gold | Strong numeric improvement or meaningful build hook. |
| Prism | Rare transformative effect. |

Current Prism direction:

- Unlocking Prism means it can enter the candidate pool; it does not guarantee appearance.
- High-risk Stage selection can increase Prism chance.
- Owning 3 or more rewards from the same element can unlock that element's Prism candidates.
- Skill-level based Prism unlocks are deferred until a skill-level system exists.
- Normal Runs should usually see 0-2 Prism rewards; very lucky or risky Runs may reach 2-3.

Luck is not implemented as a final reward-roll system yet.

## 8. Elemental Reward Branching

Elemental rewards are a major Stage Clear reward branch.

The first element to test is Lightning with the current Katana/basic combo combat.

```text
Katana + Lightning
-> confirmed melee hit as the first MVP trigger
-> Chain Lightning as the first identity test
-> 3rd combo hit upgrades as later reward payoffs
-> Shock, dodge overcharge, lightning sword wave, and Prism payoffs later
```

Reward choice should support different risk profiles:

- Stable rewards raise the Run's low point.
- Scaling rewards become stronger with more investment.
- Synergy rewards become strong when the player already owns related rewards.
- Gamble rewards can create a high ceiling but may be weaker without support.
- Prism rewards are rare transformative payoffs.

Lightning reward card candidates and detailed Prism planning live in `Docs/ElementalRewardSystem.md`.

## 9. Reward Content Direction

More attractive rewards should change how combat feels, not only increase numbers.

Current candidate directions:

- Add sword wave to the 3rd combo hit.
- Empower the next attack after dodge.
- Heal slightly after killing an enemy.
- Increase attack range.
- Add short bleed on hit.
- Trigger a shadow explosion after dodge.
- Add Lightning chain or shock behavior to confirmed hits.

These are design directions. Do not assume they are implemented until code and assets prove it.

## 10. Future RewardDataAsset Direction

`RewardDataAsset` is a design target, not implemented yet.

It should define reward cards. It should not directly apply rewards, mutate player data assets, save permanent upgrades, or control Stage progression.

Recommended responsibility:

| Area | Example Fields |
|---|---|
| Identity | `RewardId`, display name, description, icon/card art reference. |
| Category | `ApplyType`, effect type, grade, family, tags. |
| Value | Primary value, secondary value, flat/percent flag, stack policy. |
| Eligibility | Minimum Stage index, allowed Stage tags, prerequisites, exclusions. |
| Roll metadata | Base weight, grade modifier, Luck scaling candidate. |
| Presentation hook | Visual profile id, sound id, camera feedback id. |
| Implementation state | Implemented, test, placeholder. |

Recommended non-responsibility:

- Do not directly modify `PlayerStatsDataAsset`.
- Do not directly apply healing, stat changes, VFX, camera shake, or save data.
- Do not own Stage Map routing, next Stage selection, or Room progression.
- Do not hardcode Niagara systems, animation timing, or weapon sockets inside reward stat data.
- Do not become the whole reward pool resolver by itself.

The current `FRewardCardOption` can remain as the temporary UI/runtime bridge. A future implementation can build it from reward definitions first, then replace the bridge only if the UI shape becomes limiting.

## 11. Ownership Boundaries

| Area | Owner |
|---|---|
| Reward timing in manager-owned Stages | `StageFlowManager` |
| Temporary reward pickup / card UI entry point | `RewardActor` |
| Temporary card UI | `RewardSelectionWidget` |
| Temporary reward type structs/enums | `RewardTypes.h` |
| Runtime Run stat modifiers | `PlayerStatsComponent` |
| Player healing and death reset hook | `PlayerHealthSubsystem` |
| Future enemy drop item spawning | Enemy death flow, drop table, or dedicated drop spawner |
| Future enemy drop item pickup behavior | Dedicated pickup actor, not `RewardSelectionWidget` |
| Future shared effect application | Future `RewardEffectApplier` or reward runtime system |
| Future reward definitions and roll metadata | Future `RewardDataAsset` or reward table |

## 12. Not Implemented Yet

- Final reward card art/layout.
- Enemy drop item system.
- Drop chance table.
- Short timed pickup buffs.
- Reward rarity roll table.
- `RewardDataAsset`.
- Weighted reward pool.
- Luck-based reward grade calculation.
- Permanent save/load.
- Lobby upgrade UI.
- Currency system.
- Full skill, weapon, item, or synergy system.
- Full elemental reward pool.
- Full Lightning reward card pool.
- Prism unlock or Prism roll logic.
- Skill-level based Prism unlocks.

## 13. Modification Rules

- Keep Stage Clear reward timing separate from Room Clear progression.
- Keep enemy drop items separate from Stage Clear reward cards.
- Keep temporary rewards small until Stage Clear and Stage Map flow are stable.
- Do not store temporary Run rewards in `PlayerStatsDataAsset`.
- Do not turn `RewardSelectionWidget` into the owner of all reward gameplay logic.
- Do not treat `Permanent` rewards as implemented until save/load and meta progression exist.

## 14. Related Documents

- `Docs/StageRunStructure.md`: Stage Clear timing and Stage Map return order.
- `Docs/RoomCombatSystem.md`: Room Clear and Stage Clear handoff.
- `Docs/PlayerStatsSystem.md`: runtime stat ownership.
- `Docs/AttackSkillSystem.md`: attack and skill reward direction.
- `Docs/VFXSystem.md`: reward-driven combat VFX growth.
- `Docs/ElementalRewardSystem.md`: element reward branches, Lightning rewards, and Prism unlock direction.
