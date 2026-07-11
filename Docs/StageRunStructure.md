---
Status: ACTIVE
Scope: Run / Stage / Room Structure
Last Updated: 2026-07-06
Source of Truth: Yes
---

# StageRunStructure.md

## 1. Purpose

This document defines the current planning direction for `Run`, `Stage Map`, `Stage`, `Room`, `Encounter`, and `Wave`.

Codex should use this document to avoid confusing a Stage Map node with a single combat Room.

This is the source of truth for Run/Stage/Room flow terminology. Reward timing details should link to `Docs/RewardSystem.md`, Room combat execution should link to `Docs/RoomCombatSystem.md`, and enemy composition inside a Room should link to `Docs/EnemySpawnSystem.md`.

## 2. Core Direction

The game is a stage-clear top-down action roguelite.

A Run starts on a Stage Map. The Stage Map is a left-to-right node map. Each node is one Stage, not one Room.

When the player chooses a Stage node, the player enters that Stage and clears the Rooms inside it in order. After the Stage is cleared, the Stage Clear reward selection appears first. After the player chooses and applies one reward, the player returns to the Stage Map and chooses one of the connected next Stage candidates.

The Run continues through multiple selected Stages until the player reaches the Boss Stage. Defeating the Final Boss in the Boss Stage causes Run Clear.

## 3. Hierarchy

```text
Run
-> Stage Map
   -> Stage Node
      -> Stage
         -> Room
            -> Encounter
               -> Wave
```

Important rule:

- A Stage Map node means one Stage.
- A Stage contains multiple Rooms.
- A Room is the individual combat space inside a Stage.

## 4. Terms

| Term | Meaning |
|---|---|
| Run | Full play session from game start until player death or Final Boss clear. |
| Stage Map | Left-to-right node map showing the Run route. |
| Stage Node | One circular node on the Stage Map. It represents one Stage, not one Room. |
| Line / Path | Connection between Stage Nodes. After a Stage Clear, only connected next Stages can be chosen. |
| Stage | A larger zone made of multiple Rooms. |
| Room | Individual combat room inside a Stage. |
| Encounter | Enemy composition that appears inside a Room. |
| Wave | Sequential enemy group inside an Encounter. |
| Stage Clear | State after all required Rooms in the selected Stage are cleared. |
| Mid Boss Stage | A special Stage around the middle of the Run that contains a mid-boss or short mid-boss sequence. |
| Boss Stage | Final node on the far right of the Stage Map, centered on the Final Boss. |
| Run Clear | State after defeating the Final Boss in the Boss Stage. |

## 5. Run Flow

```text
Run Start
-> Stage Map
-> Choose one connected Stage
-> Enter selected Stage
-> Clear Room 0
-> Clear Room 1
-> Clear Room 2
-> Clear remaining required Rooms
-> Stage Clear
-> Stage Clear reward selection
-> Apply selected reward to the current Run
-> Return to Stage Map
-> Show connected next Stage candidates
-> Choose next connected Stage
-> Repeat through multiple Stages
-> Reach Boss Stage
-> Defeat Final Boss
-> Run Clear
```

## 6. Stage and Room Rules

- A normal Stage contains multiple Rooms.
- Earlier tests used `Room 0~6`, roughly seven Rooms.
- The seven-Room count is not a final rule.
- The current 3-month stabilization pass uses a smaller Start Stage baseline of 3 active Rooms.
- The exact Start Stage 3-Room Encounter composition is `CANDIDATE v0.1` in `Docs/StartStageV01.md` until editor and PIE verification are complete.
- Room count can change after playtesting based on Stage type, Run pacing, difficulty, or special rules.
- A Stage Type must affect the Rooms or Encounters inside that Stage.
- A Stage Node should never be treated as a single Room.
- Room order inside a Stage can remain linear at first.
- Stage Map progression between Stages is branching and selective.

## 7. Reward Timing Rule

Next Room opening rule:

- If all tracked monsters in the current Room are defeated, the Room is cleared.
- When a Room is cleared, the path to the next Room can open.
- The player should not receive a build reward card after every Room Clear in the long-term structure.
- Build reward cards should be granted at Stage Clear.

Long-term reward timing:

- Main build rewards should be granted at Stage Clear.
- The player clears the Rooms inside a Stage, then chooses a reward when the Stage is cleared.
- Stage Clear reward selection happens before returning to the Stage Map.
- The selected reward is applied to the current Run before the next Stage candidates are shown.
- This lets the player choose the next Stage with the updated build state already active.
- Room-clear rewards are not the long-term default reward cadence.

Stage Clear post-clear order:

```text
Final required Room cleared
-> Stage Clear state
-> Stage Clear reward card selection
-> Player selects one reward
-> Apply selected reward to the current Run
-> Close reward UI
-> Return to / show Stage Map
-> Show connected next Stage candidates
-> Player chooses next Stage
-> Enter selected Stage
```

Timing rule:

- Reward cards come before Stage Map route choice.
- Stage Map return happens after reward selection is complete.
- The next Stage selection UI appears on the Stage Map after the selected reward has been applied.
- Do not show next Stage candidates before the Stage Clear reward decision.

Current MVP exception:

- The current `RewardActor` / `RewardSelectionWidget` flow remains the MVP card UI implementation.
- In manager-owned Stages, Room Clear reward cards are disabled at runtime and the existing reward card flow opens at Stage Clear.
- Standalone Room Clear reward pickup remains available only as an MVP test pipeline for rooms not owned by `StageFlowManager`.
- Future reward data should move toward Stage Clear reward candidates and Stage-type reward tendencies.

## 8. Stage Map Rules

- Stage Map is not a fully linear route.
- The player does not visit every Stage in one Run.
- After clearing the current Stage, the player chooses one connected next Stage candidate.
- The next Stage candidates should become selectable only after Stage Clear reward selection is finished.
- The player should see the route choice with the post-reward build state.
- Unchosen Stage candidates are skipped for that Run route.
- This gives strategic route choice based on current build, health, and risk tolerance.

Example next Stage candidates:

| Candidate | Meaning |
|---|---|
| Many-enemy Stage | Many weak enemies. Good for range, sword wave, chain lightning, or kill rewards. |
| Strong-enemy Stage | Fewer durable enemies. Good for bleed, 3rd-hit damage, or lifesteal finishers. |
| Recovery Stage | Lower danger with better recovery or stable reward chance. |

## 9. Stage Type Examples

| Stage Type | Direction |
|---|---|
| Normal Combat Stage | Standard enemy mix and baseline reward tendency. |
| Many-enemy Stage | Many weak enemies; favors area attacks, sword waves, chain lightning, and kill triggers. |
| Strong-enemy Stage | Tank or Elite-centered; favors bleed, 3rd-hit upgrades, and lifesteal finishers. |
| Fast-enemy Stage | Fast enemy-centered; favors dodge upgrades, shadow-style rewards, and slow/control effects. |
| Recovery Stage | Lower danger; higher chance for healing or stable survival rewards. |
| High-risk Stage | Higher danger; higher chance for high-grade rewards. |
| Element Stage | Higher chance for a specific reward theme, such as lightning, blood/bleed, shadow, or cold. |
| Mid Boss Stage | Mid-run rhythm change with a mid-boss or short mid-boss sequence. |
| Boss Stage | Final Stage Map node centered on the Final Boss. |

## 10. Boss Rules

- A Boss does not appear at the end of every normal Stage by default.
- A Mid Boss can appear around the middle of the Run as a special Stage or special Stage ending.
- The Final Boss appears in the Boss Stage on the far right of the Stage Map.
- Boss Stage is separate from normal Stage structure and is centered on the Final Boss fight.
- Boss Stage does not need to follow the current seven-Room test baseline.

Possible Mid Boss Stage structure:

```text
Mid Boss Stage
-> Short combat Room
-> Recovery or preparation Room
-> Mid Boss Room
-> Stage Clear reward selection
-> Apply selected reward
-> Return to Stage Map
```

## 11. Implementation Boundary

Current implementation:

- `RoomCombatActor` handles Room entry, combat start, enemy spawning, alive enemy tracking, room clear, reward pickup, and next-room handoff.
- Current Stage 1 MVP can chain multiple `RoomCombatActor` instances by `StageId` and `RoomOrder`.
- Current Start Stage stabilization uses `StageFlowManager.ActiveStageRoomCount = 3`, so only the first three ordered Rooms count for Stage Clear.
- Current last-room fallback shows Stage Clear / Restart as a temporary MVP behavior.
- Current room-clear reward pickup is implemented for reward flow testing.
- `RoomCombatActor` now emits `OnRoomProgressionReady` as the first handoff signal.
- `StageFlowManager` now receives the signal, logs the `RoomOrder + 1` next Room candidate, compares it with the old Room handoff, and activates the next Room when progression is ready.
- `StageFlowManager` disables `RoomCombatActor` legacy progression handoff for registered Rooms.
- Until the real Stage Map flow exists, `StageFlowManager` also owns the temporary final Room -> Stage Clear restart fallback.

Not implemented yet:

- Stage Map UI.
- Stage Node data.
- Route/path selection.
- Returning to Stage Map after Stage Clear.
- Final Stage Clear reward data/grade/pool system.
- Next Stage candidate UI timing after reward selection.
- Mid Boss Stage structure.
- Boss Stage / Final Boss / Run Clear.
- Full Run/Stage Map ownership beyond the current one-Stage `StageFlowManager` MVP.
- Final `RewardDataAsset` implementation.
- Full authored encounter database for all Stages. `RoomEncounterDataAsset` exists as the first implementation path, but not all Rooms/Stages are data-driven yet.

## 12. StageFlowManager First MVP Boundary

`StageFlowManager` should be introduced gradually. The first implementation should not try to own the whole Run, Stage Map, rewards, bosses, and room combat at once.

First MVP goal:

- Move only Stage-level Room sequencing out of `RoomCombatActor`.
- Keep the current playable Room combat loop intact.
- Make `RoomCombatActor` report "this Room is ready to progress" instead of deciding the whole Stage by itself.

First MVP responsibilities:

1. Own one active Stage id.
2. Find or receive the `RoomCombatActor` list for that Stage.
3. Sort Rooms by `RoomOrder`.
4. Activate the first Room.
5. Receive a Room progression-ready signal after all tracked monsters in the Room are defeated.
6. Activate the next Room in order.
7. If no next Room exists, mark the current Stage as cleared.
8. Trigger the temporary Stage Clear fallback until the real Stage Map flow exists.

Responsibilities that should stay out of the first MVP:

- Stage Map UI.
- Choosing between next Stage nodes.
- Returning to Stage Map after Stage Clear.
- Reward grade rolls, reward pool weights, Luck stat, or technique levels.
- Mid Boss Stage routing.
- Boss Stage, Final Boss, or Run Clear.
- Enemy movement, enemy attack logic, enemy stats, or player stats.
- Room doors, room boundaries, enemy spawning, and alive enemy tracking.

Responsibilities that remain in `RoomCombatActor` for now:

- Player entry trigger.
- Local room lock/unlock through doors and boundaries.
- Enemy spawning for that Room.
- Alive enemy tracking.
- Room Clear detection.
- Current MVP `RewardActor` spawning and reward selection wait only when a Room is not owned by `StageFlowManager`.
- Emitting a progression-ready signal when the Room can hand off to the Stage layer.

Migration order:

1. Keep the existing `RoomCombatActor` auto-link and last-room fallback as the baseline.
2. Add a clean progression-ready signal from `RoomCombatActor`. Done.
3. Add `StageFlowManager` in observation mode first, where it can read Stage rooms without changing behavior. Done.
4. Move next-Room activation from `RoomCombatActor` to `StageFlowManager`.
5. Move last-Room Stage Clear fallback from `RoomCombatActor` to `StageFlowManager`.
6. Only after this is stable, reduce or deprecate `bAutoFindNextRoomByOrder` in `RoomCombatActor`.

Do not remove existing `RoomCombatActor` progression fields until the manager path is verified in editor.

NextRoomActors / auto-link transition:

- Before Output Log confirms the manager signal bridge, `NextRoomActors` and `bAutoFindNextRoomByOrder` remain the active MVP path.
- After the manager can observe signals, it may calculate the next Room candidate without changing gameplay.
- After the manager can activate the next Room in editor, `NextRoomActors` becomes fallback/manual override.
- After the manager owns final Room -> Stage Clear fallback, `bAutoFindNextRoomByOrder` can become deprecated for new work.
- Deprecated means "not the main path for new work", not immediate deletion.
- Do not remove these UPROPERTY fields abruptly because existing Blueprints and maps may reference them.

Next implementation units:

- Step A: `StageFlowManager` calculates the next Room candidate from `StageId` and `RoomOrder + 1`, then logs it. No gameplay change.
- Step B: `StageFlowManager` compares its candidate with the current `RoomCombatActor` handoff path and logs match/mismatch. No gameplay change. Implemented as a validation log.
- Step C: after editor verification, `StageFlowManager` activates the next Room. Implemented as the primary next-Room opener.
- Step D: after next-Room activation is stable, `StageFlowManager` handles final Room -> Stage Clear fallback. Temporary fallback is currently owned by the manager while Stage Map is not implemented.
- Step E: `StageFlowManager` disables Room Clear rewards for manager-owned Rooms and opens the existing reward card flow at Stage Clear. Implemented as an MVP timing correction, not as the final reward data system.

Step A and Step B are implemented as validation logs. Step C is implemented and should be verified across Room 0 -> Room 1 -> Room 2 before further Stage Map work.

## 13. Future Ownership Direction

| System | Future Responsibility |
|---|---|
| `RoomCombatActor` | One Room's combat lock, spawn, alive enemy tracking, room clear signal, local doors/boundaries. |
| `StageFlowManager` | Current Stage state, ordered Rooms, Stage Clear, return to Stage Map, Mid Boss/Boss Stage handling. |
| `StageMapWidget` | Visual node map, connected next Stage choices, route selection UI. |
| `StageNodeDataAsset` | Stage node type, display name, risk/reward hints, connected next node data. |
| `RoomEncounterDataAsset` | Room enemy composition, waves, spawn timing, boss/elite flags, clear rules. |
| `RewardDataAsset` or reward table | Reward grade, effect id, technique target, values, weight, and unlock rules. |

## 14. Superseded Planning

The following older ideas should not be treated as current long-term direction:

- Stage Map node equals one Room.
- Every Room gives a full build reward by default.
- Every normal Stage ends with a boss.
- Stage order is always fixed and fully linear.
- Vampire-survivor-style infinite enemy spawning is the main progression loop.
