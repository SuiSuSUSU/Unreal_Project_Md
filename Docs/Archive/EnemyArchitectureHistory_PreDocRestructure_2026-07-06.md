---
Status: REFERENCE
Scope: Archived Enemy Architecture History
Last Updated: 2026-07-02
Source of Truth: No
---

# EnemyArchitecture.md

## 2026-07-02 Update: Enemy Architecture Document Boundary

- This document owns enemy code/BP architecture boundaries.
- Enemy spawn composition belongs to `EnemySpawnSystem.md` and `RoomEncounterDataAsset`.
- Enemy animation quality and retargeting belong to `AnimationSystem.md` and `EnemyAnimationRetargeting.md`.
- Enemy VFX presentation belongs to `VFXSystem.md` unless a future enemy presentation component is introduced.
- Enemy stats remain separate from spawn rules and visual presentation rules.


## 2026-07-02 Update: Room Encounter Data Split

- The first `RoomEncounterDataAsset` implementation is now in C++.
- Enemy identity and stats remain on `EnemyBase` / `EnemyStatsDataAsset`.
- Room enemy composition now has an optional data source: `RoomEncounterDataAsset`.
- `RoomCombatActor` remains the runtime executor that expands encounter entries, spawns enemies, registers placed enemies, tracks alive enemies, and reports Room Clear.
- Existing direct RoomActor arrays remain fallback data until encounter assets are created for real Stage rooms.
- This split supports both room-entry spawned enemies and pre-placed Dormant enemies without moving Stage progression out of `StageFlowManager`.


## 2026-07-01 Update: Enemy Hit Flash Removed

- Common white hit-flash feedback is no longer owned by `EnemyBase`.
- The previous skeletal mesh Overlay Material hit-flash MVP is deprecated and should not be used as a current implementation 기준.
- `EnemyBase` should keep shared enemy logic focused on health ownership, chase, melee timing, spawn intro, attack playback, separation, and death handling.
- Future hit feedback should be redesigned separately through enemy-specific VFX, sound, animation reaction, camera feedback, or a dedicated presentation layer.

## 2026-07-02 Update: HitReact Architecture Boundary

- `HitReact` is a future structure candidate, not an implemented class yet.
- `EnemyBase` may eventually broadcast or call a lightweight hit-react request after confirmed damage, but it should not own all reaction presentation.
- A future `HitReactComponent` can own reaction policy and runtime state if enemy-specific Blueprint/AnimBP setup becomes too duplicated.
- Enemy Blueprint or AnimBP should still own character-specific reaction assets such as flinch Montage, stagger Montage, additive hit pose, or boss-specific reaction state.
- `EnemyStatsDataAsset` may later hold gameplay-facing resistance values such as `HitReactResistance` if stagger/poise becomes real gameplay. It should not hold animation or VFX asset references.
- `WeaponImpactVfxComponent` should own attacker-side/contact VFX requests. `HitReact` should own target-side body reaction requests.
- Do not merge hit reaction, impact VFX, damage application, and enemy death handling into one shared enemy method.
- The preferred reaction decision is attack strength versus enemy resistance, not a fixed reaction per enemy type.

Recommended decision inputs:

| Input | Owner |
|---|---|
| `DamageAmount` | Attacker combat system / `HealthComponent` result |
| `HitReactPower` | Attack definition, combo step, reward modifier, or future combat data |
| `ComboStepBonus` | Player attack/combo system |
| `HitReactResistance` | Enemy stats or enemy-specific reaction policy |
| Immunity / special windows | Enemy Blueprint, AI state, boss phase, spawn/death state |

Recommended future split:

| Area | Owner |
|---|---|
| Confirming a hit and applying damage | Attacker combat system / `HealthComponent` |
| Emitting a hit-react request | `EnemyBase` or damage receiver bridge |
| Choosing whether the enemy reacts | Enemy Blueprint or future `HitReactComponent` |
| Playing flinch/stagger animation | Enemy AnimBP, Montage, or Blueprint presentation |
| Playing contact VFX | `WeaponImpactVfxComponent` or enemy-specific VFX component |
| Boss immunity/special stagger windows | Boss-specific Blueprint/AI/presentation logic |

## 2026-06-23 Update: Fast/Tank MVP Blueprint Split

- Fast and Tank enemies are currently implemented as separate Blueprint classes derived from the existing `EnemyBase` flow.
- `BP_Enemy_Fast` and `BP_Enemy_Tank` should first differ by `EnemyStatsDataAsset` values rather than new C++ behavior.
- This keeps the first enemy-type expansion focused on movement speed, health, attack damage, attack range, attack cooldown, and separation tuning.
- The current Manny mannequin visuals are temporary placeholders.
- Final enemy-specific meshes, AnimBPs, Montages, hit reactions, and special patterns should remain character-specific asset work, not hardcoded `EnemyBase` logic.

## 2026-06-20 Update: Enemy Attack Asset Boundary

- `EnemyBase` may own generic attack playback logic for either `UAnimSequenceBase` or `UAnimMontage` assets.
- Character-specific attack quality belongs in Blueprint/AnimBP/Montage assets.
- Little Demon currently uses the original attack1 sequence through the SingleNode/SimpleAnimation MVP path. Dedicated AnimBP/Montage presentation remains a later transition option.

## 2026-06-19 Update: Attack Movement Boundary

- `EnemyBase` owns attack-start facing lock and attack-time soft separation for the current MVP.
- Chase movement may blend separation into movement input, but attack-time separation should avoid movement input so it does not trigger movement-facing rotation.
- Attack playback may temporarily ignore animation root motion and restore the previous root motion mode after the attack.
- Character-specific attack pose quality still belongs in BP/AnimBP/Montage configuration, not in common `EnemyBase` logic.

## 2026-06-15 Update: Attack Animation Playback Boundary

- Common attack animation selection remains in `EnemyBase`.
- Character-specific presentation should move toward BP/AnimBP/Montage configuration.
- `EnemyBase` now supports Dynamic Montage attack playback when a valid AnimBP class and Slot path exist, with direct `PlayAnimation` fallback.
- A dedicated Little Demon AnimBP remains the next step if idle/run/attack/death transitions still need more control.

## 2026-06-14 Update: Enemy Separation Movement

- `EnemyBase` handles current soft enemy-to-enemy separation.
- Separation is not a spawn rule; it belongs to per-enemy movement behavior and is tuned through `EnemyStatsDataAsset`.
- The MVP keeps the logic simple: each enemy checks nearby Pawn overlaps, filters to `EnemyBase`, and blends a smoothed away direction into chase movement.
- Separation uses a short update interval plus interpolation to avoid visible body jitter when enemies are close together.
- For bigger rooms or large waves, consider an update interval, spatial partitioning, Unreal crowd avoidance, or a separate movement/avoidance component.

## 1. 문서 목적

이 문서는 앞으로 게임 끝까지 가져갈 적 시스템의 기본 구조를 정리한다.

적 로직, 적 블루프린트, 적 스탯 Data Asset, 적 스폰 Data Asset의 역할을 분리해서 Codex가 나중에 구조를 섞지 않도록 하는 것이 목적이다.

## 2. 추천 최종 구조

현재 추천 구조는 아래와 같다.

```text
C++ EnemyBase
-> BP_EnemyBase
   -> BP_Enemy_Basic
   -> BP_Enemy_Fast
   -> BP_Enemy_Tank
   -> BP_Enemy_Ranged

EnemyStatsDataAsset
-> 적 개체 능력치

EnemySpawnDataAsset
-> 전역 테스트 스폰 규칙 / 일부 방 스폰 데이터 재사용 후보

RoomCombatActor
-> 방 입장 / 문 닫힘 / SpawnPoint 기준 적 생성 / 방 클리어 후보
```

적 메시/스켈레톤/애니메이션은 캐릭터별로 분리하고, 공통 로직과 섞지 않는다.
애니메이션 공유가 필요할 때는 스켈레톤 강제 변환보다 IK Rig / IK Retargeter 사용을 우선한다.
자세한 기준은 `Docs/EnemyAnimationRetargeting.md`를 따른다.

## 3. C++ EnemyBase 역할

`EnemyBase`는 모든 적이 공유하는 C++ 기반 로직을 담당한다.

- 체력 컴포넌트 보유
- 타겟 플레이어 찾기
- 기본 이동/추적 처리
- 캡슐 반지름 기반 정지 거리/겹침 방지 처리
- Pawn끼리 물리적으로 막지 않는 Collision 처리
- 피격 처리
- 사망 처리
- 적별 사망 애니메이션 재생
- 공통 이벤트
- 공통 공격 조건
- `EnemyStatsDataAsset` 값 적용

주의:
- 특정 적의 외형, 애니메이션, 개별 연출은 가능하면 `EnemyBase`에 직접 넣지 않는다.
- `EnemyBase`에는 Imp 같은 특정 몬스터의 Mesh/AnimBP 경로를 하드코딩하지 않는다.
- 특정 적 전용 로직은 하위 Blueprint 또는 별도 Component/클래스 후보로 분리한다.
- 플레이어와 적은 서로 물리적으로 밀거나 가두지 않도록 Pawn 채널을 Overlap한다. 데미지는 거리/쿨타임 판정으로 처리한다.

## 4. BP_EnemyBase 역할

`BP_EnemyBase`는 모든 적 Blueprint의 공통 부모 역할을 한다.

- 공통 Mesh/SkeletalMesh 구성
- 공통 Animation Blueprint 연결 후보
- 캐릭터별 Mesh/Skeleton/AnimBP 교체 지점
- 공통 피격/사망 이펙트 연결 후보
- 공통 Collision 설정
- 공통 EnemyStatsDataAsset 슬롯 후보
- 전용 AnimBP가 없는 적을 위한 `EnemySimpleAnimationComponent` 후보

주의:
- C++ 로직은 `EnemyBase`가 담당한다.
- 에셋 연결과 시각적 기본값은 `BP_EnemyBase`가 담당한다.
- Imp, Werewolf, 보스처럼 체형이 다른 적은 원본 스켈레톤을 유지하고, 필요한 경우 리타겟된 애니메이션을 연결한다.

## 5. 적 타입 Blueprint 후보

| Blueprint | 역할 | 설명 | 우선순위 |
|---|---|---|---|
| BP_Enemy_Basic | 기본 추적형 적 | 가장 기본적인 근거리 추격 적 | MVP |
| BP_Enemy_Fast | 빠른 돌진형 적 | 체력은 낮지만 이동속도가 빠른 압박형 | MVP |
| BP_Enemy_Tank | 고체력 압박형 적 | 느리지만 체력이 높은 적 | MVP 또는 1차 확장 |
| BP_Enemy_Ranged | 원거리 공격형 적 | 일정 거리에서 투사체로 공격 | 1차 확장 |
| BP_Enemy_LittleDemon | 빠른 소형 악마 적 | Little Demon 에셋 기반의 신규 근거리 적 | MVP 확장 |

초기에는 `BP_Enemy_Basic`부터 안정화하고, 이후 Fast/Tank/Ranged 순서로 확장한다.

## 6. EnemyStatsDataAsset 역할

`EnemyStatsDataAsset`은 적 개체 하나의 능력치를 관리하는 Data Asset 타입이다.

현재 `EnemyBase`는 `StatsDataAsset`이 지정되어 있으면 아래 값을 Data Asset에서 읽고, 지정되지 않았으면 기존 C++ 기본값을 fallback으로 사용한다.

포함 후보:

| 항목 | 설명 |
|---|---|
| EnemyID | 적 고유 ID |
| EnemyName | 적 이름 |
| MaxHealth | 최대 체력 |
| MoveSpeed | 이동속도 |
| StopDistance | 플레이어와 유지하려는 기본 정지 거리 |
| PersonalSpacePadding | 적/플레이어 캡슐이 겹치지 않도록 더하는 여유거리 |
| AttackDamage | 공격력 |
| AttackRange | 공격 범위 |
| AttackCooldown | 공격 쿨타임 |

추후 확장 후보:

| 항목 | 설명 |
|---|---|
| EnemyType | 적 타입 |
| DetectionRange | 플레이어 탐지 범위 |
| RewardEXP | 처치 시 경험치 보상 |
| RewardScrap | 처치 시 재화 보상 |

주의:
- 스폰 간격, 최대 적 수, 웨이브 비율은 여기에 넣지 않는다.
- 이 Data Asset은 “이 적이 얼마나 강한가”를 담당한다.

## 7. EnemySpawnDataAsset 역할

`EnemySpawnDataAsset`은 적을 언제, 어디서, 몇 마리 생성할지 관리한다.

현재 게임 방향에서는 `EnemySpawnDataAsset`을 전역 무한 스폰 전용으로 보지 않는다. 기존 전역 테스트 스폰 설정으로 유지하면서, 일부 방 스폰 데이터로 재사용할 수 있다.

현재 구현된 항목:

| 항목 | 설명 |
|---|---|
| EnemyClass | 활성화된 스폰 목록이 없을 때 사용할 fallback 적 Blueprint Class |
| SpawnEnemyEntries | 시간/가중치 기반으로 생성 가능한 적 목록 |
| bSpawnEnabled | 스폰 활성화 여부 |
| MaxAliveEnemies | 동시에 유지할 최대 적 수 |
| SpawnCountPerBatch | 한 번에 생성할 적 수 |
| InitialSpawnDelay | 첫 스폰까지 대기 시간 |
| SpawnInterval | 스폰 간격 |
| SpawnDistance | 플레이어 기준 스폰 거리 |

추후 확장 후보:

| 항목 | 설명 |
|---|---|
| SpawnEnemyEntries | 생성 가능한 적 종류 목록 |
| TimeRange | 특정 시간대 조건 |
| SpawnWeight | 시간대별 적 비율 또는 가중치 |
| WaveID | 웨이브 구분 |
| EliteChance | 엘리트 적 등장 확률 |

주의:
- 이 Data Asset은 “언제 어떤 적을 얼마나 생성할까”를 담당한다.
- 현재는 `SpawnEnemyEntries`로 여러 적을 시간/가중치 기준으로 섞어 스폰할 수 있다.
- `EnemyClass`는 활성 Entry가 없을 때 사용하는 fallback이다.
- 적 자체의 체력/공격력은 `EnemyStatsDataAsset`에서 관리한다.

## 8. 스테이지 클리어형 전환 기준

현재 우선 방향은 방/구역을 하나씩 클리어하는 스테이지 돌파형 탑다운 액션이다.

초기 MVP 기준:

- 랜덤 던전은 구현하지 않는다.
- 같은 맵 안에 고정 방 2~3개를 배치한다.
- 각 방에 수동으로 `RoomSpawnPoint`를 배치한다.
- 첫 구현은 `RoomCombatActor`가 `EnemyClass` 배열과 `SpawnPoint` 배열을 직접 들고 있어도 된다.
- 적 전멸 후 `RoomCleared` 상태를 만들고 문 열림 또는 보상 생성을 연결한다.

추후 구조 후보:

- 방별 적 조합과 웨이브가 늘어나면 `RoomEncounterDataAsset`으로 분리한다.
- `EnemySpawnDataAsset`은 방별 스폰 데이터의 일부로 재사용하거나, 전역 테스트 스폰 전용으로 남긴다.
- Stage/WaveSystem은 여러 방 흐름, 엘리트방, 보스방, 스테이지 클리어를 관리하는 후보로 둔다.

## 9. 추천 폴더 구조

현재 Enemy Data Asset 폴더:

```text
/Game/Data/Enemy
├─ Spawn
├─ Stats
└─ Waves
```

추후 Blueprint 폴더 후보:

```text
/Game/Blueprints/Enemy
├─ Base
├─ Basic
├─ Fast
├─ Tank
└─ Ranged
```

추후 방 전투 폴더 후보:

```text
/Game/Blueprints/Rooms
├─ Combat
├─ Doors
└─ Rewards

/Game/Data/Rooms
└─ Encounters
```

## 10. 구현 순서 후보

1. 현재 테스트 적을 `BP_Enemy_Basic` 방향으로 정리
2. `EnemyStatsDataAsset` C++ 타입 추가 완료
3. `EnemyBase`가 EnemyStatsDataAsset 값을 읽도록 연결 완료
4. `BP_EnemyBase` 또는 적 Blueprint에 EnemyStatsDataAsset 슬롯 지정
5. `BP_Enemy_LittleDemon` 추가 완료
6. `EnemySpawnDataAsset`에서 스폰할 적 클래스 선택 가능 완료
7. `EnemySpawnDataAsset`에 여러 적 종류/시간대별 비율 구조 추가 완료
8. `RoomCombatActor` 후보 설계
9. 고정 방 2~3개와 `RoomSpawnPoint` 기반 스폰 구현
10. `BP_Enemy_Fast` 추가
11. `BP_Enemy_Tank` 추가
12. `BP_Enemy_Ranged` 추가
13. `RoomEncounterDataAsset` 분리 검토
14. Stage/WaveSystem 후보 설계

## 11. Codex 구현 주의사항

- 적 관련 구현 전에는 `AGENTS.md`, `Docs/EnemySystem.md`, `Docs/EnemySpawnSystem.md`, 이 문서를 먼저 확인한다.
- 방 클리어형 전투 구현 전에는 `Docs/RoomCombatSystem.md`를 먼저 확인한다.
- 적 애니메이션, Skeleton, IK Rig, IK Retargeter 관련 작업 전에는 `Docs/EnemyAnimationRetargeting.md`를 먼저 확인한다.
- 적 개체 능력치와 스폰 규칙을 섞지 않는다.
- 여러 적 타입을 한 번에 구현하지 않는다.
- 먼저 `BP_Enemy_Basic` 기반을 안정화한다.
- 모델링과 애니메이션 연결은 개발자가 직접 판단할 수 있도록 Blueprint/Data Asset 중심으로 열어둔다.
- C++에는 공통 로직만 넣고, 개별 적의 에셋/외형/연출은 Blueprint에서 관리하는 방향을 우선한다.
