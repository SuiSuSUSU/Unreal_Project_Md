---
Status: REFERENCE
Scope: Room Combat History
Last Updated: 2026-07-06
Source of Truth: No
---

# RoomCombatSystem.md

## 2026-07-02 Update: Room Responsibility Boundary

- `RoomCombatSystem.md` is the Room-level source of truth.
- `RoomCombatActor` owns one Room's entry trigger, combat lock, local doors/boundaries, enemy tracking, Room Clear, and progression-ready signal.
- `RoomEncounterDataAsset` owns reusable encounter data for a Room, not Stage Map route data.
- Stage-level sequencing and Stage Clear reward timing belong to `StageFlowManager` and `Docs/StageRunStructure.md`.
- Reward rules should be checked in `Docs/RewardSystem.md` before changing reward timing or card effects.


## 2026-07-02 Update: RoomEncounterDataAsset First Implementation

- `RoomCombatActor` still owns one Room's trigger, combat lock, doors, boundary, enemy tracking, Room Clear, and progression-ready signal.
- `RoomCombatActor.EncounterData` is the new optional data-driven encounter source.
- If `EncounterData` has valid entries, `InitialSpawnEntries` and `WaveSpawnEntries` are expanded into runtime spawn queues.
- If `EncounterData` is empty or not assigned, direct RoomActor settings remain fallback.
- `PlacedEnemyActors` is the list for enemies already placed in the level before combat starts.
- Placed enemies are deactivated before Room combat and then activated by the encounter policy when combat starts.
- `ActivateOnRoomStart` wakes placed enemies immediately when Room combat begins.
- `ActivateOnPlayerDetection` keeps placed enemies dormant until the player enters their detection radius.
- `AliveEnemies` tracks every enemy that can block Room Clear; spawned and placed enemies are also tracked separately for future clear-rule options.


## 2026-07-01 Update: Room / Stage / Stage Map Boundary

- `RoomCombatActor` is a Room-level combat actor, not the owner of the whole Run or Stage Map.
- A Stage Map node represents one `Stage`, not one `Room`.
- A normal Stage contains multiple Rooms. The current baseline is around `Room 0~6`, but this count can vary later.
- Long-term reward timing is `Stage Clear -> reward selection`.
- The existing `RoomCombatActor -> RewardActor -> RewardSelectionWidget` room-clear reward flow is a temporary MVP test pipeline.
- Current room rewards can remain for testing reward UI/effects, but they should not be treated as the final reward cadence.
- `StageFlowManager` should later own Stage-level Room order, Stage Clear, Stage reward timing, Stage Map return, Mid Boss Stage, Boss Stage, and Run Clear.
- Detailed source of truth: `Docs/StageRunStructure.md`.

Current implementation boundary:

```text
RoomCombatActor
-> one Room's trigger/combat/spawn/alive enemy tracking/room clear
```

Long-term planning boundary:

```text
StageMap
-> choose Stage Node
-> StageFlowManager
-> RoomCombatActor chain inside selected Stage
-> Stage Clear
-> Stage Clear reward selection
-> apply selected reward
-> return to Stage Map
-> choose next connected Stage
```

## 2026-07-01 Update: StageFlowManager First Cut

`StageFlowManager` should be split out in small steps, not as a full Run manager on day one.

Current implementation status:

- The first Stage-owned Room progression path is implemented.
- `RoomCombatActor` broadcasts `OnRoomProgressionReady`.
- `StageFlowManager` receives that signal, calculates the next Room by `RoomOrder`, and activates it.
- `StageFlowManager` disables `RoomCombatActor` legacy progression handoff and Room Clear rewards for registered Rooms.
- `StageFlowManager` handles the temporary final Room -> Stage Clear reward/restart fallback until Stage Map exists.

Next Room opening condition:

- If all tracked monsters in the current Room are defeated, the Room is cleared.
- Room Clear opens the path to the next Room.
- Room Clear should not grant a build reward card in the long-term structure.
- Build reward cards should be granted at Stage Clear.

Current implementation exception:

- The existing Room Clear reward card flow remains only as an MVP test pipeline for rooms not owned by `StageFlowManager`.
- In manager-owned Stages, `StageFlowManager` disables Room Clear rewards at runtime and opens the reward card flow at Stage Clear.

Stage Clear reward and Stage Map handoff rule:

```text
Final Room progression-ready
-> StageFlowManager marks Stage Clear
-> Stage Clear reward card UI opens
-> player selects one reward
-> selected reward is applied to the current Run
-> reward UI closes
-> Stage Map is shown
-> connected next Stage candidates become selectable
```

- Stage Clear reward selection happens before Stage Map route selection.
- The next Stage selection UI should appear only after the Stage Clear reward choice is complete.
- This order lets the player choose the next Stage based on the updated build.
- Until Stage Map exists, the current Stage Clear restart overlay remains as an MVP fallback after the Stage Clear reward is claimed.
- The current implementation reuses `RewardActor` and `RewardSelectionWidget` for Stage Clear rewards instead of creating a final reward data system.

First MVP scope:

- Own the ordered Room list for one active Stage.
- Open the first Room.
- Listen for a Room progression-ready signal.
- Open the next Room by `RoomOrder`.
- Treat the final Room as Stage Clear.
- Keep the existing temporary Stage Clear fallback until Stage Map exists.

What `StageFlowManager` should not own yet:

- Stage Map UI or next Stage node selection.
- Reward grades, reward pools, Luck stat, or technique levels.
- Mid Boss Stage, Boss Stage, Final Boss, or Run Clear.
- Enemy spawning, enemy stats, enemy movement, player stats, room doors, or room boundaries.

What stays in `RoomCombatActor` for now:

- Player overlap entry trigger.
- Local room combat start.
- Doors and boundary lock/unlock.
- Enemy spawn and alive enemy tracking.
- Room Clear detection.
- Current MVP `RewardActor` spawn and reward selection wait only for rooms that are not manager-owned.
- Progression-ready event after the Room is safe to leave.

Migration rule:

- Do not delete `bAutoFindNextRoomByOrder`, `NextRoomActors`, or the current last-room fallback until the manager path is tested in editor.
- The first code step should add a clean handoff point, then move next-room activation, then move Stage Clear fallback.

## 2026-07-01 Update: NextRoomActors / Auto-Link Transition Gates

`NextRoomActors` and `bAutoFindNextRoomByOrder` are not bad or broken features. They are the current MVP path for opening the next Room from `RoomCombatActor`.

They should become fallback/test-only only after `StageFlowManager` proves that it can safely own the same responsibility.

Current status:

- `StageFlowManager` is the active next-Room handoff path for manager-owned Stages.
- `RoomCombatActor.NextRoomActors` and `RoomCombatActor.bAutoFindNextRoomByOrder` remain fallback/setup data.
- `StageFlowManager` observes `OnRoomProgressionReady`, logs the `RoomOrder + 1` next Room candidate, compares it with the old handoff path, and activates the next Room.

Transition gates:

| Gate | Condition | Ownership |
|---|---|---|
| Gate 0 | Output Log confirms `StageFlowManager received room progression-ready` after Room Clear. | No ownership change. `RoomCombatActor` still opens the next Room. |
| Gate 1 | `StageFlowManager` can find and sort same-`StageId` Rooms by `RoomOrder` in editor. | Implemented as a next Room candidate log. Manager does not activate it yet. |
| Gate 2 | Manager-calculated next Room matches the Room that `RoomCombatActor` would open. | Implemented as a match/mismatch log. `NextRoomActors` stays active; manager is validation-only. |
| Gate 3 | `StageFlowManager` activates the next Room successfully in editor. | Implemented as the primary next-Room opener. `RoomCombatActor` legacy handoff is disabled when the manager owns progression. |
| Gate 4 | `StageFlowManager` handles final Room -> Stage Clear fallback successfully. | `bAutoFindNextRoomByOrder` becomes deprecated for new work and should default OFF later. |
| Gate 5 | Stage Clear reward and Stage Map return are implemented and stable. | Old Room-level progression fallback can be considered for removal or editor-only testing. |

Deprecation meaning:

- Deprecated does not mean delete immediately.
- It means the feature is no longer the primary design path for new work.
- Because placed Blueprints and maps may still reference these fields, do not remove the UPROPERTY fields abruptly.

Rules before Gate 3:

- Do not turn off `NextRoomActors`.
- Do not turn off `bAutoFindNextRoomByOrder`.
- Do not move Stage Clear fallback out of `RoomCombatActor`.
- Do not make `StageFlowManager` mandatory for playable room progression.

Rules after Gate 3:

- New progression work should prefer `StageFlowManager`.
- `NextRoomActors` should remain available as fallback/manual override.
- Existing rooms can keep their current setup until the manager path is verified across the whole Stage.

Rules after Gate 4:

- `bAutoFindNextRoomByOrder` can become default OFF for newly placed rooms.
- It should remain available temporarily for old maps and quick tests.
- Documentation should stop recommending it as the main progression path.

## 2026-07-01 Update: StageFlowManager Next Implementation Units

Current status note:

- Gate 0 through Gate 4 are complete for manager-owned Stages.
- `StageFlowManager` is now the primary next-Room opener and temporary final-Room Stage Clear reward/restart fallback owner for manager-owned Stages.
- Step A and Step B below are historical validation steps, kept to explain why the transition was made gradually.

The code work was split into small validation steps.

Step A: next Room candidate calculation only

- Trigger: `StageFlowManager` receives `OnRoomProgressionReady`.
- Find the current Room's `RoomOrder`.
- Find a same-`StageId` Room with `RoomOrder + 1`.
- Log the candidate Room name and order.
- Do not activate the next Room.
- Do not disable `NextRoomActors`.
- Do not change Stage Clear behavior.
- Implementation status: done as a validation log in `StageFlowManager`.

Expected log direction:

```text
StageFlowManager candidate next room: Current=Room1 Order=0, Next=Room2 Order=1
```

Step B: compare manager candidate with current `RoomCombatActor` handoff

- Historical validation state: `RoomCombatActor` was kept as the active progression owner while the manager candidate was compared.
- Current manager-owned Stages use `StageFlowManager` as the primary progression owner after Step C.
- Compare the manager-calculated next Room with valid `NextRoomActors` or the auto-linked next Room.
- If they match, log an info message.
- If they do not match, log a warning.
- Do not activate the next Room from the manager yet.
- Implementation status: done as a validation log in `StageFlowManager`.

Expected log direction:

```text
StageFlowManager candidate matches RoomCombatActor handoff: Room2
```

or:

```text
StageFlowManager candidate mismatch: Manager=Room2, RoomCombatActor=Room4
```

Step C: manager-owned next Room activation

- Started after Step A and Step B were verified in editor.
- `StageFlowManager` activates the calculated next Room.
- `StageFlowManager` disables `RoomCombatActor` legacy progression handoff for registered Rooms.
- `NextRoomActors` remains available as fallback/manual override data, but it is not the primary opener while the manager owns progression.
- `StageFlowManager` also owns the temporary final Room -> Stage Clear restart fallback until the real Stage Map flow exists.
- `StageFlowManager` disables Room Clear rewards for registered Rooms and opens a Stage Clear reward using the existing `RewardActor`/`RewardSelectionWidget` path.
- `bAutoFindNextRoomByOrder` remains enabled as setup/fallback data until the current Stage is verified end to end.

Step D: manager-owned Stage Clear fallback

- Current manager-owned Stages use this path.
- If there is no next Room candidate, `StageFlowManager` handles the temporary Stage Clear reward/restart fallback.
- `bAutoFindNextRoomByOrder` can become deprecated for new work and default OFF later, but should remain available for old maps and quick tests.

What not to do yet:

- Do not add Stage Map UI.
- Do not build the final Stage Clear reward data system yet.
- Do not remove `NextRoomActors`.
- Do not remove `bAutoFindNextRoomByOrder`.
- Do not make `StageFlowManager` required for old test rooms until the current Stage works end to end.

## 2026-06-27 Update: Stage 1 Room Auto-Link MVP

`RoomCombatActor`??Stage 1??7�?Room ?�스?��? ?�게 ?�기 ?�한 ?�동 ?�결 MVP�?가진다.

???�정:

- `StageId`: 같�? Stage???�한 Room??묶는 ID?? ?�재 기본값�? `Stage1`?�다.
- `RoomOrder`: Stage ?�에?�의 Room ?�서?? Stage 1 ?�스?�에?�는 `0~6` ?�는 `1~7`???�용?????�다.
- `ExpectedStageRoomCount`: ?�버그용 ?�상 Room ?�다. Stage 1 기본 ?�스??기�??� 7?�다.
- `bAutoFindNextRoomByOrder`: `NextRoomActors`???�효??Actor가 ?�으�?같�? `StageId`??`RoomOrder + 1` Room???�동?�로 찾아 ?�결?�다.
- `bShowStageRoomProgressInDebugMessages`: ?�면 ?�버�?메시지??`RoomName (1/7)` 같�? 진행?��? 붙인??

?�재 규칙:

- ?�동?�로 ?��? `NextRoomActors`가 ?�으�?�??�결???�선?�다.
- ?�동 ?�결???�을 ?�만 `StageId + RoomOrder` ?�동 ?�결???�도?�다.
- 같�? Stage ?�에 `RoomOrder = 0`??방이 ?�으�?`0~6`??7�?방으�??�단?�고, ?�면 ?�버�??�기??`1/7`부??보여준??
- `RoomOrder`가 0부???�작?��? ?�으�?기존처럼 `1~7`??7�?방으�??�단?�다.
- ?�동 ?�결 MVP?�서??�?방이 ?�닌 Room???�리거�? 처음부??비활?�화?�다.
- ?�전 Room???�리?�되�?보상 ?�택까�? ?�나??`NextRoomActors`�??�결???�음 Room???�리거�? ?�성?�된??
- ?�라???�레?�어가 물리?�으�??�쪽 �??�치??먼�? 가?�라?? ?�서가 ?�리지 ?��? Room?� ?�투�??�작?��? ?�아???�다.
- 마�?�?Room?�거???�음 Room??찾�? 못한 Room?� 기존처럼 `Stage Cleared`?� ?�시??버튼?�로 ?�어진다.
- ??기능?� 7�?�?MVP ?�결 보조 기능?�며, ?�식 `StageFlowManager`???�직 만들지 ?�는??

## 2026-06-27 Update: Stage Progression Direction

Stage???�러 Room??묶음?�로 본다. ?�기 기획?�서??Stage ?�의 모든 ?�심 Room???�리?�하�?Stage Clear가 ?�고, 주요 빌드 보상 ?�택?� Stage Clear ?�점??발생?�다.

Stage 1 기�? 방향:

- Stage 1?� 7�?Room 진행??목표 구조�?검?�한??
- �?Room?� 짧�? ?�투?� ?�음 Room ?�동?�로 빠르�??�어진다.
- �?Room??모든 몬스?��? 처치?�면 ?�음 Room?�로 가??길이 ?�린??
- 주요 빌드 보상 ?�택?� Stage Clear ?�점??발생?�다.
- Room Clear마다 보상 카드�?주�? ?�는??
- 마�?�?Room?� 보스방이거나 보스�?직전???�심 Room???????�다.
- ?�확??Room ?? 보스�??�치, ?�복/?�벤??Room ?�함 ?��????�벨 ?�자???�스????조정?�다.

기본 ?�름:

```text
Stage 1 ?�작
-> Room 1 ?�투
-> ?�음 Room ?�동
-> Room 2 ?�투
-> ?�음 Room ?�동
-> ...
-> Room 7 ?�는 Boss Room ?�리??-> Stage 1 Clear
-> Stage Clear 보상 ?�택
-> ?�택??보상 ?�용
-> Stage Map 복�?
-> ?�음 Stage ?�보 ?�택
-> ?�택???�음 Stage 진입
```

Stage Clear ???�음 Stage ?�택 방향:

- Stage�??�리?�하�??�음 Stage ?�보 2~3개�? 보여주는 구조�??�기 방향?�로 ?�다.
- �??�보????구성, ?�험?? 보상 경향, ?�복 가?�성, ?�수 조건??간단??보여준??
- ?�택?� ?�재 빌드?� 체력 ?�태�?기�??�로 ?�단?�게 ?�다.

Stage ?�보 ?�시:

| Stage ?�보 | ?�징 | ??맞는 빌드 |
|---|---|---|
| ?�염???�수�?| ?�수 ??중심 | 3?� 검�? 공격 범위, 처치 보상 |
| 버려�?병기�?| 강적 중심 | 출혈, 3?� 강화, ?�혈 |
| 붉�? ??| 빠른 ??중심 | ?�피 강화, 반격, ?�동?�도 |
| ?�복 ?�테?��? | ?�험????��, ?�복 기회 ?�음 | 체력????? ?�태 |
| 고위???�테?��? | ?�이???�음, ?��? ?�급 보상 ?�률 증�? | 강하�??�성??빌드 |

?�재 구현 주의:

- ?�재 MVP??`RoomCombatActor.NextRoomActors`??같�? Stage ?�의 ?�음 Room ?�결?�으�?본다.
- Stage 1 MVP?�서??`NextRoomActors` ?�동 ?�결??비어 ?�으�?`StageId + RoomOrder` 기�??�로 ?�음 Room???�동 ?�색?????�다.
- Stage ?�체 ?�서, Stage Clear ???�보 ?�시, ?�음 Stage ?�택?� ?�직 구현?��? ?�는??
- ??구조가 ?�요?��????�점?�는 `StageFlowManager`가 Stage ?�서, ?�재 Stage/Room ?�태, Stage ?�보 ?�택, Stage Clear 처리�?맡는 방향??좋다.
- 초기 구현 ?�선?�위??Stage 1 ?�에??7�?Room ?�후??짧�? ?�투/보상 루프?� �?보스방을 ?�성?�는 것이??

## 2026-06-26 Update: Stage 1 Reward Direction

Stage 1?� 짧�? ?�투방을 ?�속?�로 ?�파???? Stage Clear 보상?�로 ?�투 방식??조금??바뀌는 구조�??�선?�다.

Room 방향:

- �??�나?�나???�데?�처??짧�? ?�투 ?�위�??�계?�다.
- �?방�? 빠르�??�장, ?�투, ?�리?? ?�음 �??�동?�로 ?�어?�야 ?�다.
- Stage Clear 보상 ?�보 카드???�택??Stage ?�?�과 ?�재 Run ?�태???�라 바뀌어???�다.
- 초기 카드 ?�는 2?�을 ?��??�되, 구조??최�? 4?�까지 ?�장?????�게 ?�다.

보상 ?�급 방향:

Bronze, Silver, Gold, Prism?� ?�재 ?�보 명칭?�며, 최종 ?�급 ?��? ?�름?� ?�직 ?�정?��? ?�는??

| ?�급 | ??�� |
|---|---|
| Bronze | ?��? ?�치 ?�승 ?�는 ?�전??기본 보상 |
| Silver | ?�레??감각??체감?�는 ?�투 보너??|
| Gold | 빌드 방향???�아주는 강한 ?�투 보상 |
| Prism | 가??강한 ?�급???�심 보상, ?�투 방식 변?��? ??|

보상 ?�보 방향:

- `3?� 공격??검�?추�?`
- `?�피 ???�음 공격 강화`
- `??처치 ??체력 ?�량 ?�복`
- `공격 범위 증�?`
- `공격 ?�중 ??짧�? 출혈`
- `?�피 ???�상 ??��`

?��? ?�급 보상 ?�보:

| 보상 | ?�과 |
|---|---|
| ?�연�?검�?| 3?� 공격 ???�방 검�?발사 |
| 마무�??�격 | 3?� 공격 ?��?지 증�? |
| ?��? 마무�?| 3?� 공격 범위 증�? |
| ?�혈 마무�?| 3?��???처치 ??체력 ?�복 |
| 출혈 베기 | 3?� ?�중 ??출혈 부??|

?�재 주의:

- ???�용?� 채택??기획 방향?�며, ?�직 구현 ?�료 ?�태가 ?�니??
- `RewardActor`/`RewardSelectionWidget`???�재 MVP ?�름?� ?��??�다.
- `RewardDataAsset`, ?��???가중치, 보상 ?� ?�이�? ?�??로드, ?�화 ?�스?��? ?�직 만들지 ?�는??
- ?�중??보상 ?�이?��? ?�요?��?�?Stage Clear 보상 ?�보, 카드 ?? ?�급 ?�장 ?�책?� `RewardDataAsset` ?�는 Stage-level data�???��??
- `RoomEncounterDataAsset`?� ?�수 방에?�만 ?�요??추�? 보상 메�??�이?��? 가�????�다.

## 2026-06-25 Update: Stage Flow Terminology

?�테?��? 진행 구조???�래 ?�어�?기�??�로 ?�리?�다.

| ?�어 | ?��? |
|---|---|
| `Stage` | ?�나????구역, 챕터, ?�마 |
| `Room` | `Stage` ?�에???�투가 벌어지??개별 �?|
| `Encounter` | `Room` ?�에???�제�??�오????조합 |
| `Wave` | `Encounter` ?�에???�차?�으�??�오????묶음 |
| `Run` | 게임 ?�작부??죽거???�리?�할 ?�까지???�체 ?�레??|

기본 계층?� `Run > Stage > Room > Encounter > Wave`�?본다.

?�재 MVP?�서??`RoomCombatActor`가 `Room`, `Encounter`, `Wave` ?��? 책임??직접 ?�고 ?�어???�다. �??��? ??조합???�어?�면 `StageFlowManager`가 `Stage` ?�름??맡고, `RoomEncounterDataAsset`??`Encounter`?� `Wave` ?�이?��? 맡는 방향?�로 분리?�다.

## 2026-06-24 Update: RoomCombatActor Editor Setting Cleanup

- `RoomCombatActor` now exposes simple room identity fields for placed rooms: `RoomId`, `RoomOrder`, and `RoomDisplayName`.
- These fields are for editor organization, logs, and debug messages. They do not yet create a full `StageFlowManager` room registry.
- `ProgressionReadyDelay` can delay opening the next room or stage-clear flow after a reward is claimed or after a room clears without a reward.
- Room debug output is grouped under `Room Combat|Debug` with `bShowRoomDebugMessages`, message durations, and `SpawnPreviewDuration`.
- Default values preserve the current MVP behavior: immediate progression after reward selection, visible debug messages, and the existing 40 second progression message.

## 2026-06-24 Update: MVP vs Later Room Structure

Current Room owner:

- `RoomCombatActor` owns room entry detection, combat start, enemy spawning, placed enemy activation, alive enemy tracking, and Room Clear detection.
- In manager-owned Stages, `StageFlowManager` owns next-room handoff, Stage Clear reward timing, and the temporary last-room restart fallback.
- Standalone or legacy test rooms can still use `RoomCombatActor` reward/progression fallback behavior for isolated testing.

Current room enemy configuration:

- `InitialSpawnEnemyClasses`: enemies spawned together at combat start.
- `SpawnEnemyClasses`: ordered sequential queue for enemies spawned one at a time after previous tracked enemies die.
- `EnemyClass + EnemiesToSpawn`: fallback for older single-class tests.
- `SpawnPoints`: optional placed actor transforms; fallback positions are generated near the room actor.

Current reward configuration:

- In manager-owned Stages, Room Clear rewards are disabled at runtime and the existing `RewardActor` / `RewardSelectionWidget` flow opens at Stage Clear.
- Standalone or legacy test rooms can still spawn `RewardActor` after Room Clear for reward UI/effect testing.
- `RewardSelectionWidget` applies one selected card in both paths.

Current/later split:

- `StageFlowManager` currently owns one active Stage's room order, next-room activation, Stage Clear reward timing, and temporary restart/menu fallback. Branching and Stage Map return remain later work.
- `RoomEncounterDataAsset` currently owns the first reusable enemy group/wave timing path. Elite/boss flags, room tags, and special room metadata remain later candidates.
- Stage Clear reward candidates should live in Stage-level reward data or a future reward table, not in normal Room encounter data by default.
- A future `RoomDoorActor` or `CombatLockVolume` can take over visual doors and non-rectangular room locks.
- Final stage-clear UI should move out of the temporary restart fallback when Stage Map exists.

## 2026-06-24 Update: Data Structure Preparation

Current `RoomCombatActor` setting groups:

- Stage MVP: `StageId`, `ExpectedStageRoomCount`, `bAutoFindNextRoomByOrder`, and progress debug labels.
- Activation: `bStartOnlyOnPlayerOverlap`.
- Encounter / Fallback: `EnemyClass`, `EnemiesToSpawn`.
- Encounter / Initial Wave: `InitialSpawnEnemyClasses`.
- Encounter / Sequential Queue: `SpawnEnemyClasses`, `SpawnDelayBetweenEnemies`.
- Encounter / Spawn Points: `SpawnPoints`.
- Reward: `bSpawnRewardOnRoomCleared`, `RewardActorClass`, `RewardSpawnPoint`, `RewardFallbackOffset`.
- Progression: `bWaitForRewardBeforeOpeningRoom`, `bEnableNextRoomActorsOnProgressionReady`, `NextRoomActors`.
- Doors / Boundary: `DoorActors`, room boundary extents, and auto boundary blockers.

Reward structure:

- `FRewardCardOption.RewardId` is the current stable reward identity for temporary reward cards.
- `RewardId` is not a final reward database key yet, but it prepares the path for a later `RewardDataAsset` or reward table.
- `ERewardApplyType` remains the current MVP split: `Instant`, `Run`, `Permanent`.
- `Permanent` rewards still do not write save data.

Current `StageFlowManager` responsibility:

- Own one active Stage id, current room order, final Stage Clear reward timing, temporary restart/menu fallback, and future branching entry point.
- Decide when the next Room becomes active in manager-owned Stages.
- Keep stage-wide state out of individual room actors once room count grows.
- It should not own enemy movement, enemy stats, reward effect application, or player runtime stats.

Current `RoomEncounterDataAsset` fields and later candidates:

- Identity: `EncounterId`, display name, debug note.
- Room tags: normal, elite, boss, event, reward, tutorial.
- Enemy setup: initial enemy classes, sequential enemy entries, optional wave entries.
- Spawn setup: spawn delay, spawn point policy, fallback spread.
- Clear condition: all enemies dead, boss dead, timer survived, interact complete.
- Special reward setup: optional special-room reward candidates, reward count, reward policy.
- Progression setup: next room tag or next room rule, not direct actor references.

Current rule:

- Active Room sequencing and temporary Stage Clear reward/restart fallback are owned by `StageFlowManager` for manager-owned Stages.
- `RoomEncounterDataAsset` exists and can be used for reusable Room encounter data.
- Direct per-room settings on placed `BP_RoomCombatActor` instances remain useful as fallback/editor override data while encounter assets are introduced gradually.

## 2026-06-24 Update: Room and Reward Safety Rules

- A room enemy must have a valid `HealthComponent` to be tracked by `RoomCombatActor`.
- If a spawned enemy cannot report death through `HealthComponent`, `RoomCombatActor` logs the issue, destroys the enemy, and keeps the room clear check moving.
- `RoomCombatActor` cleans invalid or already-dead enemy references before checking whether the room is cleared.
- Reward selection UI cleanup belongs to `RewardActor` for the current MVP. If the reward actor ends play while the UI is open, it removes the widget reference.
- Reward card selection can open even if the player health component is temporarily missing, but each reward effect still validates the component it needs before applying.

## 2026-06-23 Update: Stage Clear Restart MVP

- If a cleared room has no valid `NextRoomActors`, it is currently treated as the final room of the MVP stage.
- Final room clear now displays a simple stage-clear overlay with a `?�시 ?�작` button.
- The button reloads the current level and resets the MVP run state through normal level restart behavior.
- Before reloading, the button clears current viewport widgets so the temporary restart overlay does not persist into the restarted level.
- This is not the final stage-clear UI design; it is only a functional test button.

## 2026-06-23 Update: Korean Reward Card Text

- Reward card titles, descriptions, apply type labels, and effect value labels are now shown in Korean.
- The current reward card UI remains the same direct-choice MVP flow.
- This change does not add localization tables yet; strings are still defined directly in the current C++ MVP.

## 2026-06-23 Update: Room Reward Type Split

- Standalone Room-clear reward tests can still use the existing flow: room clear -> `RewardActor` spawn -> player touches pickup -> `RewardSelectionWidget` opens -> one card is selected.
- Manager-owned Stages disable Room-clear rewards and open the existing reward card flow at Stage Clear instead.
- Reward cards now have an apply type: `Instant`, `Run`, or `Permanent`.
- `Instant` is for one-time effects such as `Heal +20`.
- `Run` is for temporary current-run stat changes such as max health, attack damage, move speed, or dodge cooldown changes.
- `Permanent` is a future meta-progression placeholder only; permanent save/load and lobby upgrades are intentionally not part of the current MVP.
- The current card UI remains a small direct-choice UI and is not a finalized reward design.

## 2026-06-23 Update: Initial Simultaneous Enemy Wave

- `RoomCombatActor.InitialSpawnEnemyClasses` is the current MVP field for enemies that should appear together as the first room wave.
- All valid entries in `InitialSpawnEnemyClasses` spawn immediately when combat starts.
- If `InitialSpawnEnemyClasses` is filled, the room does not immediately spawn the first `SpawnEnemyClasses` entry.
- After the initial enemies are defeated, any remaining `SpawnEnemyClasses` entries can still be used as the delayed sequential queue.
- If `InitialSpawnEnemyClasses` is empty, the previous `SpawnEnemyClasses` sequential queue and `EnemyClass` plus `EnemiesToSpawn` fallback remain unchanged.
- Room1 currently uses this for a two-enemy first wave: Fast + Tank.

## 2026-06-22 Update: Ordered Enemy Class Array

- `RoomCombatActor` now supports `SpawnEnemyClasses`.
- If `SpawnEnemyClasses` has entries, the room treats the array as a sequential spawn queue.
- The first entry spawns when room combat starts, and the next entry spawns after the currently tracked enemy is defeated.
- `SpawnDelayBetweenEnemies` controls the delay before the next queued enemy appears; the current temporary default is `0.5` seconds.
- `SpawnPoints` still decide spawn transforms by index; if there are fewer spawn points than enemy entries, spawn points wrap by index.
- If `SpawnEnemyClasses` is empty, the existing `EnemyClass` plus `EnemiesToSpawn` fallback remains active.
- This keeps the current Little Demon single-class setup working while allowing per-room mixed enemy lists in the editor.

## 2026-06-22 Update: Room-Specific Encounter Settings

- Stage 1 uses direct per-actor settings on `RoomCombatActor`.
- Each placed room can independently set `EnemyClass`, `EnemiesToSpawn`, `SpawnPoints`, reward location, and `NextRoomActors`.
- This is the current MVP approach for making Room1, Room2, Room3, and boss rooms behave differently without adding a larger data system too early.
- `RoomEncounterDataAsset` now exists as the data-driven structure for reusable enemy lists, counts, waves, activation policy, and clear rules.
- Direct `RoomCombatActor` settings should remain fallback values or simple override values for test rooms.
- Stage Clear reward candidates should move to Stage-level reward data or a future reward table instead of normal Room encounter data.

## 2026-06-22 Update: Stage Clear Fallback MVP

- Historical MVP fallback: Room progression originally branched by `NextRoomActors`.
- In manager-owned Stages, `StageFlowManager` now opens the next Room by `StageId + RoomOrder` and handles final Room -> Stage Clear reward/restart fallback.
- `NextRoomActors` remains fallback/manual override data for old maps and standalone tests.

## 2026-06-21 Update: Next Room Progression MVP

- Room progression now waits for reward choice when a room reward exists.
- Flow: room clear -> reward spawns -> player chooses one card -> doors open -> boundary unlocks -> `Next Room Open`.
- `RoomCombatActor.NextRoomActors` is an optional list for actors that should become visible/collidable when the next room is ready.
- `NextRoomActors` are hidden and collision-disabled at BeginPlay, then enabled after reward selection.
- For a two-room MVP test, put the second room's `BP_RoomCombatActor` into the first room's `NextRoomActors` array.
- If no reward actor is spawned, the room opens immediately after clear.
- This keeps next-room progression tied to reward completion without introducing a full stage manager yet.

## 2026-06-21 Update: Reward Card Selection Rule

- Room rewards now use a card selection MVP.
- The first test version shows `2` cards.
- The structure allows up to `4` reward cards later.
- The pickup still comes from `RoomCombatActor` spawning `RewardActor` after room clear.
- `RewardActor` opens the card UI when touched, and the selected card applies its effect.
- The current initial card effects are `Heal +20` and `Max Health +10`.

## 2026-06-21 Update: Room Clear Reward Pickup MVP

- `RoomCombatActor` now owns the first room-clear reward spawn flow.
- On room clear, `RoomCombatActor` can spawn `RewardActor`.
- `RewardActor` is currently a simple overlap pickup that opens the reward card selection UI.
- When the player chooses a card, the selected temporary effect is applied and the pickup is removed.
- This confirms the pipeline: room clear -> reward appears -> player touches reward -> card UI opens -> selected effect runs.
- This does not finalize the long-term reward design. Reward choices, rarity, item pools, and room-specific rewards are still future work.

## 2026-06-14 Update: Spawn Debug Preview

- `RoomCombatActor` now draws purple debug boxes at the actual enemy spawn transforms when combat starts.
- The labels `Spawn 1`, `Spawn 2`, `Spawn 3` appear briefly above the debug boxes.
- If `SpawnPoints` are assigned, the preview uses those actor transforms.
- If no `SpawnPoints` are assigned, the preview uses the same fallback spawn locations as the spawned enemies.
- This is only a spawn-position preview and does not change spawn rules.

## 2026-06-12 ?�데?�트: ?�동 �?경계 MVP

- `RoomCombatActor`???�동 �?경계 ?�스?�용 `BoundaryBlocker_North/South/East/West` 컴포?�트�?가진다.
- `TriggerBox`???�장 감�?�??�당?�고, ?�레?�어 ?�탈 방�???별도 Boundary Blocker가 ?�당?�다.
- `RoomBoundaryHalfExtent`, `BoundaryWallThickness`, `bShowBoundaryBlockersForDebug` 값�? ?�디??배치 ?�면?�서??즉시 경계 미리보기??반영?�다.
- ?�투 ?�작 ??`LockRoomBoundary()`�?Boundary Blocker Collision??켜서 ?�레?�어가 �?밖으�??��?지 못하�??�다.
- �??�리????`UnlockRoomBoundary()`�?Boundary Blocker Collision??꺼서 ?�음 구역?�로 ?�동?????�게 ?�다.
- MVP?�서??`RoomCombatActor` ?��? ?�동 경계�?검증한??
- ?�식 구조?�서??`CombatLockVolume` ?�는 `RoomBoundaryActor`�?분리?�고, `RoomDoorActor`???�제 �?메시/?�니메이???�운???�금 ?�태�??�당?�는 방향??좋다.

## 2026-06-12 ?�데?�트: �??�림/?�힘 MVP

- `RoomCombatActor`??�?�??�스?�용 `DoorActors` 배열??가진다.
- ?�투 ?�작 ???�록??�?Actor�?보이�??�고 Collision??켜서 문을 ?�는??
- �??�리?????�록??�?Actor�??�기�?Collision??꺼서 문을 ?�다.
- ?�재??MVP 검�?구조?�며, �??�니메이???�금/?�운??개별 ?�태가 ?�요?��?�?별도 `RoomDoorActor`�?분리?�다.
- ?�디?�에?�는 Static Mesh Actor, Cube, BlockingVolume ??��??Actor�?`DoorActors`???�어 ?�스?�할 ???�다.

## 1. 문서 목적

??문서???�테?��? ?�리?�형 ?�다???�션 MVP??�??�위 ?�투 구조�??�리?�다.

Codex가 기존 ?�역 ?�스???�폰 구조?� ??�??�리?�형 진행 구조�??�동?��? ?�도�?기�????�공?�다.

## 2. ?�재 방향

- 게임 방향?� ?�터 ??건전처럼 ?��? �?구역???�나???�리?�하???�테?��? ?�파???�다???�션 MVP??
- 기존 WASD ?�동, 좌클�?1/2/3?� 콤보, Space ?�피???��??�다.
- `HealthComponent`, `EnemyBase`, `EnemyStatsDataAsset`, Little Demon ??구조???��??�다.
- 무한???�이 계속 ?�오??구조??보류?�다.
- ?��? �??�장 ??SpawnPoint 기�??�로 ?�해�??�량 ?�는 ?�이브로 ?�성?�다.
- 초기 검증�? ?�덤 ?�전 ?�이 같�? 맵에 ?�수??고정 방을 배치??진행?�다.
- Stage 1 목표 구조???�후 7�?Room ?�후�??�장?�는 방향??검?�한??

## 3. MVP ?�름

?�재 MVP 검�??�름:

1. ?�작 �?2. ?�투 �?1
3. Room Clear 보상 ?�스???�는 ?�음 �??�동
4. ?�투 �?2
5. Room Clear 보상 ?�스???�는 ?�음 �??�동
6. ?�리??보스 ?�보 �??�는 마�?�?Room
7. ?�테?��? ?�리??
?�기 기획?�서??주요 빌드 보상??Room마다 주�? ?�고, Stage ?�의 ?�요??Room??모두 ?�리?�한 ??Stage Clear ?�점??지급한?? ?�재 Room Clear 보상?� 보상 카드 UI?� ?��????�과 ?�용??검증하�??�한 MVP ?�이?�라?�이??

�??�나??기본 ?�름:

1. ?�레?�어가 방에 ?�어간다.
2. �??�장 Trigger가 `RoomCombatActor`�??�성?�한??
3. 문이 ?�힌??
4. `RoomSpawnPoint` ?�치???�이 ?�성?�다.
5. ?�아?�는 ??목록??추적?�다.
6. 모든 ?�이 ?�망?�면 �??�리???�태가 ?�다.
7. 문이 ?�리거나 보상???�성?�다.
8. ?�레?�어가 ?�음 방으�??�동?�다.

## 4. ?�보 ?�래??블루?�린??
| ?�보 | ??�� | 초기 ?�단 |
|---|---|---|
| `RoomCombatActor` | �??�장 감�?, ?�투 ?�작, ???�폰, AliveEnemies 추적, �??�리??처리 | C++ MVP 구현??|
| `RoomSpawnPoint` | �??�에???�이 ?�성???�치 | ?�용 ?�래???�보, ?�재???�반 Actor `SpawnPoints` 배열 ?�용 |
| `RoomDoorActor` | ?�투 ?�작 ??�??�힘, ?�리????�??�림 | ?�용 ?�래???�보, ?�재??`DoorActors` 배열 MVP ?�용 |
| `RewardActor` | ?�재 MVP??Room Clear 보상 ?�스???�업 | ?�시 보상 ?�이?�라??|
| `RoomEncounterDataAsset` | 방별 ??조합, ?�이�? ?�수 �?보상 메�??�이??관�?| 구조 ?�정 ??분리 ?�보 |

## 5. 초기 구현 범위

초기?�는 ?�래 범위�??�한?�다.

- ?�덤 ?�전 ?�음
- ?�점 ?�음
- ?�쇠 ?�음
- 복잡???�이???�너지 ?�음
- ?�수??고정 ?�스??방에???�작?�고, ?�후 Stage 1 목표 Room ?�로 ?�장
- ?�동 배치 `RoomSpawnPoint` ?�용
- �?구현?� `RoomCombatActor`??`EnemyClass` 배열�?`SpawnPoint` 배열??직접 ?�출?�도 ??- ?�재 Room Clear 보상?� ?�순 Actor ?�성 ?�는 ?�버�?로그�??�작 가?�하?? ?�기 주요 보상?� Stage Clear ?�점?�로 ?�다.

## 6. ?�재 구조?�??관�?
| 기존 구조 | ?��? ?��? | �??�투?�서????�� |
|---|---|---|
| WASD ?�동 | ?��? | �????�동 |
| 좌클�?1/2/3?� 콤보 | ?��? | �??�투 기본 공격 |
| Space ?�피 | ?��? | �??�투 ?�피/?�출 |
| `HealthComponent` | ?��? | ?�레?�어/???��?지?� ?�망 처리 |
| `EnemyBase` | ?��? | 방에???�성?�는 ?�의 공통 기반 |
| `EnemyStatsDataAsset` | ?��? | ??개체 ?�력�?관�?|
| Little Demon ??구조 | ?��? | 초기 �??�투 ???�보 |
| `EnemySpawnSubsystem` | ?��??�되 ?�치 ??�� | ?�역 ?�스???�폰/?�시 검증용 |
| `BP_RoomCombatActor` | 추�???| `/Game/Blueprints/Rooms/BP_RoomCombatActor` |

## 7. RoomCombatActor ?�보 책임

- �??�투가 ?��? ?�작?�는지 ?�인
- �??�투가 ?��? ?�리?�됐?��? ?�인
- ?�레?�어 ?�장 ???�투 ?�작
- ?�록??SpawnPoint?�서 ???�성
- ?�아?�는 ??목록 관�?- ???�망 ?�벤???�신
- 모든 ?�이 ?�망?�면 �??�리??처리
- ?�면??`Room Cleared` ?�버�?메시지 출력

?�재 구현??MVP 범위:

- `Source/MyProject/Rooms/RoomCombatActor.h`
- `Source/MyProject/Rooms/RoomCombatActor.cpp`
- `/Game/Blueprints/Rooms/BP_RoomCombatActor`
- `TriggerBox` Overlap?�로 ?�투 ?�작
- `EnemyClass`, `SpawnPoints`, `EnemiesToSpawn`�?Details ?�널?�서 지??- SpawnPoint가 부족하�?지?�된 SpawnPoint�??�환 ?�용
- SpawnPoint가 ?�으�?`RoomCombatActor` ?�치 근처??fallback ?�폰
- ?�투 ?�작 ??`DoorActors`�??�고, ?�리?????�시 ?�다.
- ?�투 ?�작 ???�동 �?경계 Blocker�?켜고, ?�리?????�다.
- ?�디?�에??`RoomBoundaryHalfExtent`�?�?경계 미리보기�??�인?????�다.
- ?�투 ?�작 ???�제 ?�폰 ?�치??보라???�버�?박스�??�시 ?�시?�다.

?�직 구현?��? ?��? 범위:

- 보상 ?�성
- ?�음 �??�동
- ?�용 `RoomSpawnPoint` ?�래??- ?�용 `RoomDoorActor` ?�래??- `RoomEncounterDataAsset` ?�동

## 8. RoomEncounterDataAsset ?�보

구조가 커�?�??�래 ?�이?��? Data Asset?�로 분리?�다.

| ??�� | ?�명 |
|---|---|
| EncounterID | �??�투 고유 ID |
| EnemyClasses | ?�성?????�래??목록 |
| EnemyCounts | ?�별 ?�성 ?�량 |
| SpawnDelay | ?�투 ?�작 ???�폰 지??|
| WaveCount | ?�이�???|
| SpecialRoomRewardCandidates | ?�수 방에?�만 ?�요??추�? 보상 ?�보 |
| bIsEliteRoom | ?�리?�방 ?��? |
| bIsBossRoom | 보스�??��? |

초기 MVP?�서????Data Asset??바로 만들지 ?�아???�다. 먼�? `RoomCombatActor`가 직접 배열???�고 검증한??

## 9. 구현 ?�서 ?�보

1. `RoomCombatActor` ?�계 ?�료
2. 고정 �?1개에??Trigger ?�장 감�? 구현 ?�료
3. SpawnPoint 기�? ???�성 구현 ?�료
4. ?�성???�의 ?�망 ?�벤??추적 구현 ?�료
5. 모든 ???�망 ??�??�리???�버�?출력 구현 ?�료
6. �??�힘/?�림 MVP 구현 ?�료
7. ?�동 �?경계 MVP 구현 ?�료
8. ?�폰 ?�치 ?�버�??�시 구현 ?�료
9. `RoomSpawnPoint` 별도 ?�래???�계
10. 간단??`RewardActor` ?�성
11. ?�수??고정 ?�스??방으�??�장
12. Stage 1 목표 Room ?��? 보스�?구성?�로 ?�장
12. ?�요 ??`RoomEncounterDataAsset` 분리

## 10. Codex 구현 주의?�항

- 구현 ??`AGENTS.md`, `Docs/CurrentImplementation.md`, `Docs/EnemySpawnSystem.md`, `Docs/EnemyArchitecture.md`�?먼�? ?�는??
- 기존 ?�투/조작/체력/??구조�?갈아?��? ?�는??
- `EnemySpawnSubsystem`????��?��? ?�는??
- �??�투 구현?� `RoomCombatActor`부???��? ?�위�??�작?�다.
- ?�덤 ?�전, ?�점, ?�쇠, 복잡???�이???�너지??초기 MVP???��? ?�는??
- 구현 ???�정 ?�일, ?�성 ?�일, ?�결??Blueprint, ?�스??방법???�약?�다.


> Archive note: This file preserves pre-restructure dated notes. It may contain legacy or corrupted text and must not be used as current implementation truth unless explicitly requested.
