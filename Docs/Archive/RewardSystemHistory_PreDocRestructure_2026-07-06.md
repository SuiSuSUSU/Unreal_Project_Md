---
Status: REFERENCE
Scope: Archived Reward System History
Last Updated: 2026-07-06
Source of Truth: No
---

# RewardSystem.md

## 1. Document Purpose

This document defines the current reward-system direction and responsibility boundaries.

Use this document before changing reward cards, reward timing, reward effects, reward UI, reward grades, or reward-related player growth.

## 2. Current Reward Direction

The long-term reward cadence is Stage Clear based.

```text
Clear all required Rooms in the selected Stage
-> Stage Clear
-> Reward card selection
-> Apply selected reward
-> Return to Stage Map
-> Choose one connected next Stage
```

Room Clear should not grant a main build reward card in the long-term structure.

The old Room Clear reward flow may remain as a standalone MVP test path for rooms not owned by `StageFlowManager`.

## 2026-07-02 Update: Reward Source Split

The reward system should separate combat drop items from Stage Clear build rewards.

Recommended high-level split:

| Reward Source | Timing | Duration | Purpose | Examples |
|---|---|---|---|---|
| Enemy Drop Item | Dropped by enemies with a chance on death. | Instant or short timed effect. | Combat sustain and moment-to-moment tempo. | Heal pickup, short attack buff, short move-speed buff, short cooldown buff. |
| Stage Clear Reward Card | Offered after the selected Stage is cleared. | Current Run duration. | Build direction and combat-style growth. | 3rd-hit sword wave, dodge-followup attack boost, attack range increase, bleed on hit, kill-heal passive. |
| Permanent Upgrade | Outside the current Run, usually lobby/meta progression. | Permanent after save/load exists. | Long-term account/player growth. | Base attack level, max health level, move speed level. |

Important distinction:

- A one-time `Heal +20` pickup is an `Enemy Drop Item`.
- A reward that says "heal slightly whenever an enemy is killed" is a `Stage Clear Reward Card` because it changes the current Run's build rules.
- A short `Attack Up` or `Move Speed Up` buff from a dropped item is an `Enemy Drop Item`.
- A Run-long attack upgrade, combo upgrade, dodge upgrade, bleed rule, lightning chain rule, or sword-wave rule is a `Stage Clear Reward Card`.

Long-term direction:

- Stage Clear cards should focus on Run-long build identity, not basic sustain pickups.
- Enemy drops should cover momentary recovery, short buffs, and combat pacing items.
- Permanent upgrades remain deferred until save/load, currency, lobby upgrade UI, and meta progression are designed.

Implementation boundary:

- `RewardSelectionWidget` should be treated as the Stage Clear card selection UI.
- Future enemy drop items should use a separate pickup path, not the Stage Clear card UI.
- A future `RewardEffectApplier` or similar gameplay owner may apply both card effects and pickup effects, but the UI/pickup actor should not own complex gameplay rules.
- `RewardDataAsset` or reward tables should eventually define Stage Clear card rewards. Enemy drop definitions can use separate item/drop data if they grow beyond simple MVP values.

## 2026-07-02 Update: Elemental Reward Branching

Elemental rewards are adopted as a major Stage Clear reward branch.

The first element to test is Lightning, paired with the current Katana/basic combo combat.

```text
Katana + Lightning
-> confirmed melee hit as the first MVP trigger
-> Chain Lightning as the first identity test
-> 3rd combo hit upgrades as later reward payoffs
-> Shock, dodge overcharge, lightning sword wave, and Prism payoffs later
```

Reward choice should support both safe and high-ceiling decisions:

- Stable rewards raise the Run's low point.
- Scaling rewards become stronger with more investment.
- Synergy rewards become strong only when the player already has related rewards.
- Gamble rewards can create a high ceiling but may be weaker if the Run does not support them.
- Prism rewards are rare transformative payoffs.

Lightning reward details and card candidates are tracked in `Docs/ElementalRewardSystem.md`.

Prism rule summary:

- Prism unlock does not guarantee appearance.
- Unlocking a Prism means it can enter the candidate reward pool.
- High-risk Stage selection can increase Prism chance.
- Owning 3 or more rewards from the same element can unlock that element's Prism candidates.
- Skill-level based Prism unlocks are deferred until a skill-level system exists.
- Normal Runs should usually see 0-2 Prism rewards; very lucky or risky Runs may reach 2-3.

## 3. Current Implementation

- `StageFlowManager` owns the current manager-owned Stage Clear reward timing.
- `RewardActor` is the temporary reward carrier.
- `RewardSelectionWidget` is the temporary reward card UI.
- `RewardTypes.h` defines the temporary reward card data and apply type enums.
- `PlayerStatsComponent` owns runtime stat modifiers for Run rewards.
- `PlayerHealthSubsystem` clears Run rewards on death or level restart.

Current reward data is still temporary code/data, not a final reward database.

## 2026-07-02 Update: RewardDataAsset Structure Design

This section is a design target only. Do not treat `RewardDataAsset` as implemented until a C++ asset class and actual assets exist.

`RewardDataAsset` should become the source of reward card definitions. It should not become the system that directly applies rewards, mutates player data assets, saves permanent upgrades, or controls Stage progression.

Recommended responsibility:

| Area | Example Fields |
|---|---|
| Identity | `RewardId`, display name, description, icon/card art reference |
| Category | `ApplyType`, `EffectType`, grade, reward family, reward tags |
| Value | primary value, secondary value, flat/percent flag, stack policy, max stack count |
| Eligibility | minimum Stage index, allowed Stage tags, prerequisites, exclusions, unique-per-run flag |
| Roll Metadata | base weight, grade weight modifier, Luck scaling candidate |
| Presentation Hook | visual profile id, sound id, camera feedback id |
| Implementation State | implemented/test/placeholder marker |

Recommended non-responsibility:

- Do not directly modify `PlayerStatsDataAsset`.
- Do not directly apply stat changes, healing, VFX, camera shake, or save data.
- Do not own Stage Map routing, next Stage selection, or Room progression.
- Do not hardcode Niagara systems, animation timings, or weapon sockets inside reward stat data.
- Do not become a full reward pool table by itself. Reward definitions and reward pool/roll policy should stay separable.

Recommended enum/data candidates:

| Candidate | Purpose |
|---|---|
| `ERewardGrade` | Bronze, Silver, Gold, Prism candidate tier names. |
| `ERewardEffectType` | Stable effect id such as Heal, AttackDamage, MoveSpeed, DodgeCooldown, ThirdHitSwordWave, BleedOnHit. |
| `ERewardStackPolicy` | Allow stack, upgrade existing, unique per Run, or replace lower grade. |
| `FRewardValue` | Numeric reward payload such as amount, percent flag, optional secondary amount. |
| `FRewardCardDefinition` | Runtime/UI transfer shape generated from a reward asset or table entry. |

The current `FRewardCardOption` can remain as the temporary UI/runtime bridge. A future implementation can build `FRewardCardOption` from `RewardDataAsset` first, then replace the bridge only if the current UI shape becomes limiting.

Current temporary reward mapping:

| Current Reward | Future Category | Future Effect Type | Status |
|---|---|---|---|
| `instant_heal_20` | Instant | Heal | Implemented temporary reward. |
| `run_max_health_10` | Run | MaxHealth | Implemented temporary reward. |
| `run_attack_damage_5` | Run | AttackDamage | Implemented temporary reward. |
| `run_move_speed_80` | Run | MoveSpeed | Implemented temporary reward. |
| `run_dodge_cooldown_015` | Run | DodgeCooldown | Implemented temporary reward. |
| Permanent placeholder | Permanent | PermanentUpgrade | Placeholder only; no save/load. |

Long-term placement note:

- `instant_heal_20` is implemented as a temporary card option now, but its long-term home is enemy drop / pickup item, not the main Stage Clear card pool.
- `run_attack_damage_5`, `run_move_speed_80`, and `run_dodge_cooldown_015` are acceptable temporary Stage Clear cards while the MVP is small.
- If attack, movement, or cooldown buffs become short timed boosts, they should move to enemy drop items.
- If they last for the current Run and change build direction, they can stay in the Stage Clear reward card family.

Recommended migration order:

1. Keep the current hardcoded reward options while the Stage Clear loop stabilizes.
2. Move one-time heal and short timed buffs conceptually into an enemy-drop item path.
3. Add a minimal Stage Clear reward definition path with only Run-long card rewards.
4. Convert `RewardActor` or a small reward resolver to generate card options from reward definitions.
5. Add reward grade and weight policy after Stage type selection is real.
6. Add Luck-based grade influence only after the basic reward pool works.
7. Add reward-driven VFX presentation profiles after the combat VFX path is stable.

## 4. Reward Apply Types

Current reward apply types:

| Type | Meaning | Current Rule |
|---|---|---|
| Instant | Applied immediately when selected. | Example: heal. |
| Run | Applies only during the current Run or current level session. | Reset on death or restart. |
| Permanent | Long-term growth type. | Structure only for now; no save/load or meta progression yet. |

Do not mutate `PlayerStatsDataAsset` for temporary Run rewards. Runtime changes should go through runtime stat state such as `PlayerStatsComponent`.

## 5. Reward Grade Direction

Reward grades are a planning direction, not a final implemented data system.

Current candidate grade names:

| Grade | Planning Meaning |
|---|---|
| Bronze | Small numeric improvement. |
| Silver | Medium numeric improvement. |
| Gold | Strong numeric improvement or simple build hook. |
| Prism | Rare transformative effect. |

Prism rewards should be rare and may later depend on a Luck stat or Stage type. Luck is not implemented as a final reward-roll system yet.

## 6. Reward Content Direction

More attractive rewards should change how combat feels, not only increase numbers.

Examples:

- Add sword wave to the 3rd combo hit.
- Empower the next attack after dodge.
- Heal slightly after killing an enemy.
- Increase attack range.
- Add short bleed on hit.
- Trigger a shadow explosion after dodge.

These are design directions. Do not assume they are implemented until code and assets prove it.

## 7. Ownership Boundaries

| Area | Owner |
|---|---|
| Reward timing in manager-owned Stages | `StageFlowManager` |
| Temporary reward pickup / card UI entry point | `RewardActor` |
| Temporary card UI | `RewardSelectionWidget` |
| Temporary reward type structs/enums | `RewardTypes.h` |
| Runtime Run stat modifiers | `PlayerStatsComponent` |
| Player healing / death reset hook | `PlayerHealthSubsystem` |
| Future enemy drop item spawning | Enemy death flow, drop table, or dedicated drop spawner |
| Future enemy drop item pickup behavior | Dedicated pickup actor, not `RewardSelectionWidget` |
| Future shared effect application | Future `RewardEffectApplier` or reward runtime system |
| Future reward definitions, grade, weights, prerequisites | Future `RewardDataAsset` or reward table |

## 8. Not Implemented Yet

- Final reward card art/layout.
- Enemy drop item system.
- Drop chance table.
- Short timed pickup buffs.
- Reward rarity roll table.
- RewardDataAsset implementation.
- Weighted reward pool.
- Luck-based reward grade calculation.
- Permanent save/load.
- Lobby upgrade UI.
- Currency system.
- Full skill, weapon, item, or synergy system.
- Elemental reward pool.
- Lightning reward cards.
- Prism unlock or Prism roll rules.
- Skill-level based Prism unlocks.

## 9. Related Documents

- `Docs/StageRunStructure.md`: Stage Clear timing and Stage Map return order.
- `Docs/RoomCombatSystem.md`: Room Clear and Stage Clear handoff.
- `Docs/PlayerStatsSystem.md`: runtime stat ownership.
- `Docs/AttackSkillSystem.md`: attack and skill reward direction.
- `Docs/VFXSystem.md`: reward-driven combat VFX growth.
- `Docs/ElementalRewardSystem.md`: element reward branches, Lightning rewards, and Prism unlock direction.
