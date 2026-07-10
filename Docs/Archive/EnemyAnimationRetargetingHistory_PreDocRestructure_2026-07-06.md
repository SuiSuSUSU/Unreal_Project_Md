---
Status: REFERENCE
Scope: Archived Enemy Animation History
Last Updated: 2026-06-28
Source of Truth: No
---

# EnemyAnimationRetargeting.md

## 2026-07-02 Update: Animation Document Boundary

- This document remains the enemy-specific animation and retargeting source of truth.
- Cross-cutting animation rules now live in `Docs/AnimationSystem.md`.
- Use `AnimationSystem.md` for AnimNotify, Montage, BlendSpace, player/enemy timing, and animation ownership rules.
- Use this document for enemy skeleton, retargeting, Little Demon, Tank/Butcher, and monster-specific animation asset notes.

## 2026-07-04 Update: Fast DemonHeavy Setup

- `BP_Enemy_Fast` now uses `/Game/Demons_Big_Pack/DemonHeavy/Mesh/SK_DemonHeavy`.
- Fast uses the DemonHeavy skeleton and its native animation assets directly; no retargeting is required for the current setup.
- Current Fast animation assignments are `Anim_DemonHeavy_idle1`, `Anim_DemonHeavy_run1`, `Anim_DemonHeavy_attack1`, and `Anim_DemonHeavy_death1`.
- Fast currently uses Mesh relative transform `Location=(0,0,-95)`, `Rotation=(Pitch=0,Yaw=-90,Roll=0)`, and `Scale=(0.85,0.85,0.85)` to align the authored DemonHeavy forward direction with the Character capsule forward.
- Fast currently uses `AttackAnimationPlayRate=2.0`.
- `ABP_DemonHeavy_Fast` now drives Fast idle/run locomotion through `EnemyAnimInstance`.
- `AM_DemonHeavy_Fast_Attack1` is generated from `Anim_DemonHeavy_attack1` and plays through `DefaultSlot`.
- `AM_DemonHeavy_Fast_Strong_Attack3` is generated from `Anim_DemonHeavy_attack3` and is assigned as `BP_Enemy_Fast.StrongAttackAnimation`.
- `BP_Enemy_Fast.StrongAttackEveryNthAttack=3`, so attack3 plays on every third successful melee attack.
- Fast attack playback now uses the `EnemyBase` AnimBP/Montage path instead of the SingleNode fallback path.
- The current Fast Montage uses `BlendIn=0.08` and `BlendOut=0.14` for smoother Run <-> Attack transitions.
- The current Fast strong Montage uses `BlendIn=0.10` and `BlendOut=0.18`.
- Fast currently sets `StrongAttackDamageMultiplier=1.4` and `StrongAttackRangeMultiplier=1.15`, so attack3 also has stronger gameplay weight.
- `EnemySimpleAnimationComponent` remains present as a fallback component type, but its Fast idle/move assignments are cleared while the AnimBP path is active.
- If run-loop, attack recovery, or death presentation becomes visibly rough in PIE, tune `ABP_DemonHeavy_Fast` and `AM_DemonHeavy_Fast_Attack1` before changing shared `EnemyBase` logic.


## 2026-06-28 Update: Tank Butcher AnimBP Attack Path

- `BP_Enemy_Tank.AttackAnimation` is set to `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Weak_Attack1` as the weak attack Montage.
- `BP_Enemy_Tank.StrongAttackAnimation` is set to `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Strong_Attack3` as the strong attack Montage.
- `AM_Butcher_Tank_Weak_Attack1` is generated from `Anim_Butcher_attack1`.
- `AM_Butcher_Tank_Strong_Attack3` is generated from `Anim_Butcher_attack3`.
- `BP_Enemy_Tank.StrongAttackEveryNthAttack` is set to `2`, so attack3 plays on every second successful melee attack.
- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/ABP_Butcher` as its dedicated Animation Blueprint.
- `ABP_Butcher` targets `/Game/Demons_Big_Pack/Butcher/Mesh/SK_Butcher_Skeleton`.
- The Tank mesh currently assigned in BP is `/Game/Imported/Blender/Butcher_Modified/SK_Butcher_Modified_SourceSkeleton`, which uses the same Butcher skeleton.
- The AnimBP graph uses `Anim_Butcher_idle1` and `Anim_Butcher_walk2` for base locomotion, then routes through `DefaultSlot` so `EnemyBase` can play weak and strong Montage assets.
- This avoids the SingleNode `PlayAnimation` fallback, which can cut abruptly when returning from attack to locomotion.
- `AttackAnimationLockDurationScale` can release the enemy's movement/facing lock before or after the full animation length.
- Tank currently uses `0.9` to reduce the small recovery pause after `Anim_Butcher_attack1`.
- `AttackAnimationPlayRate` controls attack animation playback speed for the attack path only.
- Tank currently uses `0.75`, so both weak attack1 and strong attack3 read as heavier Tank attacks without returning to the earlier overly long attack3 lock.
- `PostAttackTurnToPlayerDuration` controls a short optional preparation turn after attack playback.
- Tank currently uses `0.7` seconds so it can face the player smoothly before chase locomotion resumes.
- Tank weak/strong Montages currently use `0.28` seconds of BlendOut for a softer return from `DefaultSlot` attack playback.
- Tank movement animation is tuned separately in `ABP_Butcher`; movement currently uses `Anim_Butcher_walk2` at play rate `0.9` because `Anim_Butcher_run1` and the earlier `walk1` setup had visible loop stutter in PIE.
- `/Game/Blueprints/Enemy/Tank/BS_Butcher_Tank_Locomotion` exists as the prepared Tank locomotion BlendSpace.
- The BlendSpace uses Speed samples `0=Anim_Butcher_idle1`, `110=Anim_Butcher_walk1`, and `260=Anim_Butcher_walk2`; it intentionally avoids `Anim_Butcher_run1/run2` until those short loops are visually verified.
- Unreal Python can create and configure the BlendSpace asset, but it cannot replace the existing AnimGraph `SequencePlayer` with a `BlendSpacePlayer`; that graph connection must be made in the editor.
- `EnemyAnimInstance` uses movement-state hysteresis for AnimBP enemies: run starts above the start threshold and only returns to idle after speed stays below the lower stop threshold briefly.
- This avoids mixing the current Tank/Butcher mesh setup with unrelated Manny attack animation assets.
- The current MVP should keep Tank on verified weak/strong slots first. If more variation is needed later, test the remaining Butcher attacks individually before adding any random attack selection.
- Attack damage timing is still handled by `EnemyBase` distance/cooldown logic, not by animation notifies yet.
- The developer should visually judge the attack1-to-attack3 transition in PIE. If it still feels cut, prefer a single combo Montage with weak/strong sections over adding random attack selection.

## 2026-06-21 Update: Little Demon Spawn Intro Animation

- `BP_Enemy_LittleDemon.SpawnIntroAnimation` is configured to use `Anim_Little_Demon_rage`.
- The spawn intro uses the current SingleNode/SimpleAnimation-friendly path and does not require assigning `ABP_LittleDemon`.
- `EnemySimpleAnimationComponent` is paused while the spawn intro plays, then reset and resumed after the intro finishes.
- `SpawnIntroMeshRelativeOffset` can be used per enemy Blueprint when a spawn intro animation appears too high or too low compared with the grounded idle/run pose.
- `BP_Enemy_LittleDemon` currently plays `Anim_Little_Demon_idle2` for 1.0 second after rage, then turns toward the player over 0.45 seconds before chase starts.
- The enemy should not chase or attack until the spawn intro animation is complete.

## 2026-06-20 Update: Little Demon Attack Sequence Path

- `BP_Enemy_LittleDemon.AttackAnimation` points to the original Little Demon `Anim_Little_Demon_attack1` sequence.
- `BP_Enemy_LittleDemon` currently uses `AnimationSingleNode` plus `EnemySimpleAnimationComponent` for the MVP.
- `EnemyBase` uses runtime Dynamic Montage playback for sequence attack assets only when a valid AnimBP and Slot are active; the no-AnimBP path plays the configured sequence directly.
- The temporary `AM_LittleDemon_Attack1` asset was removed to avoid T-pose risk from a stale or malformed Montage asset.
- `ABP_LittleDemon` still exists as a transition asset, but it is not currently assigned to `BP_Enemy_LittleDemon`.
- The temporary AnimBP + runtime Dynamic Montage setup produced visible T-pose/flicker artifacts between attacks, so it is not the current MVP runtime path.
- `EnemyAnimInstance.bIsAttacking` is set by `EnemyBase` during attack playback, and locomotion reports `bIsMoving=false` while attacking.
- Remaining target: dedicated death handling and later AnimNotify-based damage timing.

## 2026-06-19 Update: Little Demon AnimBP Transition Start

- `EnemyAnimInstance` now exists as the C++ parent for enemy AnimBPs.
- `ABP_LittleDemon` has been created for the Little Demon skeleton.
- `ABP_LittleDemon` is wired for idle/run base locomotion and `DefaultSlot` attack1 Montage playback.
- `EnemySimpleAnimationComponent` currently drives Little Demon idle/run because the AnimBP path is paused for MVP stability.

## 2026-06-15 Update: Little Demon Attack Blend Direction

- Little Demon attack playback can use Dynamic Montage once a dedicated AnimBP class and Slot path are available.
- Until a dedicated AnimBP is stable, sequence attack assets are played directly through SingleNode animation.
- SingleNode playback is the current MVP fallback and current Little Demon path.
- The next formal animation step is Little Demon death state handling and later AnimNotify damage timing.
- Random attack selection has been removed from `EnemyBase` for the MVP; Little Demon should use attack1 as the single current attack animation.

## 1. 문서 목적

이 문서는 적 캐릭터가 늘어날 때 메시, 스켈레톤, 애니메이션을 어떻게 관리할지 정리한다.

Codex가 나중에 적 애니메이션을 연결할 때 모든 몬스터를 UE5 Manny 스켈레톤으로 억지 변환하지 않도록 기준을 제공한다.

## 2. 확정된 방향

- 적의 공통 로직은 `EnemyBase`와 관련 Component/Data Asset에서 관리한다.
- 적의 외형, 메시, 스켈레톤, Animation Blueprint는 캐릭터별 Blueprint에서 관리한다.
- Imp, Werewolf, 보스, 특수 몬스터는 각자 원본 스켈레톤을 유지하는 방향을 우선한다.
- UE5 Manny/Mannequin 애니메이션을 다른 적에게 사용해야 할 때는 IK Rig와 IK Retargeter를 우선 사용한다.
- 적 애니메이션 작업은 코드 로직보다 에셋 연결과 시각 확인이 중요하므로 Unreal Editor에서 직접 테스트해야 한다.

## 3. 비추천 방향

- Imp 메시 자체를 UE5 Manny 스켈레톤으로 강제 변경하지 않는다.
- 다양한 몬스터를 하나의 스켈레톤으로 억지 통일하지 않는다.
- C++ `EnemyBase`에 특정 몬스터 전용 메시, 스켈레톤, 애니메이션 경로를 고정하지 않는다.
- 리타겟 결과를 확인하지 않고 공격/이동 애니메이션을 바로 게임 로직에 연결하지 않는다.

## 4. 추천 구조

```text
C++ EnemyBase
-> 공통 체력
-> 공통 이동/추적
-> 공통 공격 조건
-> 공통 사망 처리

BP_EnemyBase
-> 공통 Collision
-> 공통 HealthComponent 연결
-> 공통 EnemyStatsDataAsset 슬롯

BP_Enemy_Imp
-> Imp Mesh
-> Imp Skeleton
-> Imp AnimBP 또는 Retarget된 Imp 전용 AnimBP
-> Imp EnemyStatsDataAsset

BP_Enemy_Werewolf
-> Werewolf Mesh
-> Werewolf Skeleton
-> Werewolf AnimBP
-> Werewolf EnemyStatsDataAsset

BP_Enemy_LittleDemon
-> Little Demon Mesh
-> Little Demon Skeleton
-> EnemySimpleAnimationComponent로 idle/run 임시 전환
-> SpawnIntroAnimation으로 Anim_Little_Demon_rage 재생
-> AttackAnimation으로 Anim_Little_Demon_attack1 재생
-> DeathAnimation으로 death1 재생
-> bDestroyOnDeath=false로 사망 후 남김
-> Little Demon EnemyStatsDataAsset

IK Rig / IK Retargeter
-> UE5 Manny 애니메이션을 Imp/Werewolf/기타 적용 애니메이션으로 변환
```

## 5. Imp 기준 권장 방식

Imp는 현재 원본 MonsterPack 메시와 스켈레톤을 유지하는 방향을 우선한다.

권장 흐름:

1. Imp가 사용하는 Skeletal Mesh와 Skeleton을 확인한다.
2. Imp용 IK Rig가 없으면 Imp Skeleton 기준으로 IK Rig를 만든다.
3. UE5 Manny 또는 원하는 원본 캐릭터의 IK Rig를 확인한다.
4. IK Retargeter를 만들어 Manny 애니메이션을 Imp용 애니메이션으로 변환한다.
5. 변환된 애니메이션을 Imp 전용 Animation Sequence 또는 Montage로 저장한다.
6. `BP_Enemy_Imp` 또는 Imp AnimBP에서 변환된 애니메이션을 사용한다.
7. 공격 타이밍, 루트 모션, 발 위치, 손 위치가 어색하지 않은지 에디터에서 확인한다.

## 6. 난이도 기준

| 작업 | 난이도 | Codex 처리 적합도 | 비고 |
|---|---:|---|---|
| 기존 Imp 이동 AnimBP 연결 | 낮음 | 높음 | 이미 동작 확인된 구조가 있다 |
| Little Demon 단일 노드 idle/run 연결 | 낮음 | 높음 | `EnemySimpleAnimationComponent`로 임시 처리 가능 |
| Manny 애니메이션을 Imp로 리타겟 | 보통 | 보통 | 에디터에서 시각 확인 필요 |
| Imp 공격 Montage 연결 | 보통 | 높음 | 리타겟된 애니메이션이 준비되어 있으면 Codex가 연결 가능 |
| Imp 자체를 Manny 스켈레톤으로 변환 | 높음 | 낮음 | 리깅/스킨 웨이트 작업에 가깝다 |
| 여러 몬스터의 공통 리타겟 파이프라인 정리 | 보통~높음 | 보통 | 캐릭터가 늘어난 뒤 단계적으로 정리한다 |

## 7. 캐릭터가 많아질 때의 기준

- 로직은 공통화하고 외형/애니메이션은 캐릭터별로 분리한다.
- 같은 체형끼리는 리타겟 결과를 재사용할 수 있다.
- 체형이 크게 다른 적은 전용 리타겟 조정 또는 전용 애니메이션을 사용한다.
- 공격 판정은 애니메이션 파일 자체에 묶지 말고, 가능하면 코드/Notify/몽타주 타이밍으로 분리한다.
- 현재 Little Demon 공격 애니메이션은 attack1 단일 Sequence를 SingleNode/SimpleAnimation 경로로 직접 재생하며, 실제 데미지 타이밍은 아직 기존 거리/쿨타임 코드 판정이다.
- 캐릭터마다 `EnemyStatsDataAsset`을 별도로 두어 체력, 이동속도, 공격력 차이를 관리한다.
- 전용 AnimBP가 없는 신규 적은 먼저 `EnemySimpleAnimationComponent`로 빠르게 검증한 뒤, 필요하면 전용 AnimBP로 승격한다.

## 8. Codex 작업 주의사항

- 적 애니메이션 작업 전 `AGENTS.md`, `Docs/EnemyArchitecture.md`, 이 문서를 먼저 읽는다.
- 코드에서 특정 적 메시나 AnimBP 경로를 영구 고정하지 않는다.
- 임시 테스트 경로를 사용했다면 문서와 최종 요약에 “임시”라고 명시한다.
- 리타겟 작업은 결과가 자연스러운지 사람이 직접 확인해야 한다.
- 애니메이션이 재생된다고 해서 공격 판정까지 정상이라는 뜻은 아니므로, 공격 타이밍은 별도로 테스트한다.
- Skeleton, IK Rig, IK Retargeter, Animation Blueprint를 변경할 때는 기존 플레이어 Manny 설정과 섞이지 않도록 경로를 확인한다.
