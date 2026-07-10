---
Status: REFERENCE
Scope: Enemy System History
Last Updated: 2026-07-06
Source of Truth: No
---

# EnemySystem History Archive

This archive preserves the previous EnemySystem document before the 2026-07-06 documentation cleanup.
Use `Docs/EnemySystem.md` as the active source of truth.
# EnemySystem.md

## 2026-07-02 Update: Enemy Document Boundary

- `EnemySystem.md` is the gameplay-facing enemy behavior source of truth.
- `EnemyArchitecture.md` owns enemy class structure and responsibility boundaries.
- `EnemySpawnSystem.md` owns enemy spawn, Room encounter data, and placed enemy policy.
- `EnemyAnimationRetargeting.md` and `AnimationSystem.md` must be checked before changing enemy animation playback, AnimBP, Montage, BlendSpace, or retargeting.
- `VFXSystem.md` must be checked before adding enemy hit, spawn, death, or status VFX.

## 2026-07-02 Update: Legacy Test Enemy Note

- `SurvivorSquareEnemy` and `SurvivorSquareSpawner` are old template/test assets.
- Do not treat lower historical sections that mention `SurvivorSquareEnemy` as the current enemy baseline.
- Current room-combat enemy baseline is `EnemyBase`-derived Blueprint enemies such as Little Demon, Fast, and Tank/Butcher, with room composition handled by `RoomCombatActor` and optional `RoomEncounterDataAsset`.


## 2026-07-02 Update: Spawned and Placed Enemy Activation

- Room enemies can now be spawned at combat start or placed in the level before combat begins.
- Both paths still use `EnemyBase` as the common runtime base.
- `EnemyBase.EnemyActivationState` separates alive/dead state from whether the enemy is currently chasing and attacking.
- `DeactivateEnemy()` puts an enemy into Dormant state and stops movement/attack behavior.
- `ActivateEnemy()` wakes a Dormant enemy so it can resume normal chase and attack behavior.
- `RoomCombatActor.PlacedEnemyActors` should be used for pre-placed enemies inside a Room.
- `RoomEncounterDataAsset.PlacedEnemyActivationPolicy` decides whether placed enemies wake on Room start or near-player detection.


## 2026-06-28 Update: Tank Butcher AnimBP Attack Path

- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Weak_Attack1` as its configured weak `AttackAnimation`.
- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Strong_Attack3` as its configured `StrongAttackAnimation`.
- These Montages are generated from `Anim_Butcher_attack1` and `Anim_Butcher_attack3` in `/Game/Demons_Big_Pack/Butcher/Animation`.
- `BP_Enemy_Tank.StrongAttackEveryNthAttack` is set to `2`, so the second successful melee attack uses attack3 as a strong attack.
- `BP_Enemy_Tank` now uses `/Game/Blueprints/Enemy/Tank/ABP_Butcher` as its dedicated AnimBP.
- `ABP_Butcher` uses `Anim_Butcher_idle1` and `Anim_Butcher_walk2` as the locomotion base and exposes `DefaultSlot` for attack playback.
- Tank attack playback should use the `EnemyBase` AnimBP/Dynamic Montage path rather than the SingleNode fallback.
- `EnemyBase` keeps `AttackAnimation` as the default weak attack and can optionally use `StrongAttackAnimation` every N successful melee attacks.
- `EnemyBase.AttackAnimationLockDurationScale` exists for per-enemy attack recovery tuning.
- The default is `1.0`; Tank currently uses `0.9` to reduce the visible pause after Butcher attack1.
- `EnemyBase.AttackAnimationPlayRate` exists for per-enemy attack animation speed tuning.
- The default is `1.0`; Tank currently uses `0.75` to make Butcher weak/strong attacks feel heavier without returning to the earlier overly long attack3 lock.
- `EnemyBase.PostAttackTurnToPlayerDuration` exists for per-enemy attack recovery presentation tuning.
- The default is `0.0`; Tank currently uses `0.7` seconds to turn toward the player smoothly before chase resumes.
- Tank weak/strong Montages currently use `0.28` seconds of BlendOut so attack recovery returns to locomotion less abruptly.
- `ABP_Butcher` currently uses `Anim_Butcher_walk2` at play rate `0.9` for movement, because `Anim_Butcher_run1` and the earlier `walk1` setup produced visible chase stutter.
- `/Game/Blueprints/Enemy/Tank/BS_Butcher_Tank_Locomotion` is prepared as the Tank locomotion BlendSpace with idle/walk/heavy-walk samples.
- Until `ABP_Butcher` is manually rewired from `SequencePlayer` locomotion to a `BlendSpacePlayer`, the runtime Tank locomotion still follows the existing AnimBP graph.
- Larger enemies or bosses can later raise or lower this value per Blueprint without changing shared attack logic.
- `EnemyAnimInstance` keeps run/idle state stable with separate movement start/stop thresholds and a short stop delay, so tiny chase-speed dips do not repeatedly restart locomotion.
- `EnemySimpleAnimationComponent` remains a fallback for enemies without a dedicated AnimBP, but it is not the Tank idle/run driver anymore.
- This follows the current rule that enemy-specific mesh, skeleton, idle/run animation, and attack presentation belong in the enemy Blueprint/assets, while `EnemyBase` keeps the shared chase and melee timing logic.
- The Butcher folder also contains other attack candidates, but random attack selection remains excluded until they are checked one by one for clean in-game transitions.
- If weak-to-strong chaining still feels abrupt, solve it in animation assets first: shared combo Montage sections, adjusted blend windows, and then AnimNotify timing.

## 2026-07-01 Update: Enemy White Hit Flash Removed

- The shared enemy white hit-flash reaction is no longer part of the current MVP or enemy design direction.
- `EnemyBase` should not own a generic white overlay flash on damage.
- Little Demon, Fast, and Tank enemies now rely only on the existing damage/death flow unless a new hit feedback feature is explicitly designed.
- Boss and MiniBoss hit feedback should be handled later as enemy-specific presentation, not by reusing the removed shared white flash settings.

## 2026-07-02 Update: HitReact Design Direction

- Hit reaction is a future enemy feedback layer, not part of the current implemented damage flow yet.
- `EnemyBase` should not directly force every enemy to flinch, stagger, flash, or stop attacking whenever damage is received.
- A future `HitReact` path should start from a confirmed hit request and compare attack strength against enemy resistance.
- The preferred rule is not "enemy type always reacts this way"; it is "this hit is strong enough to overcome this enemy's resistance."
- The decision should consider `DamageAmount`, attack type, combo step, optional `HitReactPower`, reward modifiers, and enemy `HitReactResistance`.
- Small enemies can have lower resistance, so normal or 3rd-hit attacks trigger reactions more often.
- Tank enemies can have higher resistance, so weak attacks do nothing while strong hits or 3rd-hit rewards can cause a reaction.
- MiniBoss and Boss enemies can use very high resistance, `Immune`, or phase-specific reaction windows.
- Spawn intro, death, and uninterruptible attack states should ignore normal hit reactions unless a special rule explicitly overrides them.
- If knockback or stagger becomes gameplay-affecting, it should be balanced as its own rule instead of hidden inside visual hit feedback.

Recommended decision concept:

```text
HitReactScore = DamageAmount + AttackTypeBonus + ComboStepBonus + RewardBonus

if HitReactScore >= EnemyHitReactResistance:
    play allowed reaction
else:
    no reaction
```

Suggested first resistance direction:

| Enemy Type | First Direction |
|---|---|
| Little Demon | Low resistance; stronger normal hits or 3rd-hit attacks can cause light flinch. |
| Fast | Low-to-medium resistance; avoid reactions so frequent that movement feels broken. |
| Tank | High resistance; weak hits do nothing, strong hits may cause a heavy reaction later. |
| MiniBoss | Very high resistance or special reaction windows only. |
| Boss | Immune by default, boss-specific reaction rules only. |

## 2026-07-04 Update: Fast DemonHeavy Visual Test

- `BP_Enemy_Fast` now uses `/Game/Demons_Big_Pack/DemonHeavy/Mesh/SK_DemonHeavy` as its current visual test mesh.
- Fast remains an `EnemyBase` Blueprint with `DA_EnemyStats_Fast`.
- Fast now uses `/Game/Blueprints/Enemy/Fast/ABP_DemonHeavy_Fast` as its dedicated AnimBP.
- Current Fast animation assignments are `Anim_DemonHeavy_idle1`, `Anim_DemonHeavy_run1`, `Anim_DemonHeavy_attack1`, and `Anim_DemonHeavy_death1`.
- Fast currently uses Mesh relative transform `Location=(0,0,-95)`, `Rotation=(Pitch=0,Yaw=-90,Roll=0)`, and `Scale=(0.85,0.85,0.85)` so the DemonHeavy visual matches the Character capsule facing.
- Fast currently uses `AttackAnimationPlayRate=2.0` because `Anim_DemonHeavy_attack1` felt too slow in PIE.
- Fast attack playback now uses `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Attack1` through the `EnemyBase` AnimBP/Montage path.
- The current Fast Montage uses `DefaultSlot`, `BlendIn=0.08`, and `BlendOut=0.14` to soften Run <-> Attack transitions.
- Fast now has a second configured attack: `/Game/Blueprints/Enemy/Fast/AM_DemonHeavy_Fast_Strong_Attack3`, generated from `Anim_DemonHeavy_attack3`.
- `BP_Enemy_Fast.StrongAttackEveryNthAttack=3`, so the third successful melee attack uses attack3 as a stronger variation.
- The current strong attack Montage uses `DefaultSlot`, `BlendIn=0.10`, and `BlendOut=0.18`.
- Strong attacks can now apply separate gameplay multipliers through `EnemyBase.StrongAttackDamageMultiplier` and `EnemyBase.StrongAttackRangeMultiplier`.
- `BP_Enemy_Fast` currently sets `StrongAttackDamageMultiplier=1.4` and `StrongAttackRangeMultiplier=1.15`, so its third attack is not only visually different but also more threatening.
- If Fast movement or attack presentation feels abrupt, tune the Fast Blueprint animation setup before changing shared `EnemyBase` logic.

## 2026-06-23 Update: Fast/Tank Mesh Orientation Fix

- `BP_Enemy_Fast` previously used a temporary Manny placeholder mesh, but now uses the DemonHeavy visual test setup above.
- `BP_Enemy_Tank` has since moved to the modified Butcher mesh and `ABP_Butcher`.
- If either enemy appears sideways or buried in the floor, first check the Blueprint Mesh transform and assigned AnimBP before changing `EnemyBase` movement logic.

## 2026-06-23 Update: Fast/Tank Enemy Type MVP

- Fast and Tank enemy types now exist as temporary MVP Blueprints.
- `BP_Enemy_Fast` is the quick chaser test type with lower health and faster movement.
- `BP_Enemy_Tank` is the durable chaser test type with higher health, slower movement, and stronger attacks.
- Both reuse `EnemyBase`, `HealthComponent`, and `EnemyStatsDataAsset`.
- Fast uses the DemonHeavy mesh with dedicated `ABP_DemonHeavy_Fast`, while Tank currently uses the modified Butcher mesh and dedicated `ABP_Butcher` presentation path.
- Room1 currently spawns Fast and Tank together through `RoomCombatActor.InitialSpawnEnemyClasses`.

## 2026-06-21 Update: Spawn Intro Lock

- `EnemyBase` can play an optional `SpawnIntroAnimation` on BeginPlay.
- During the spawn intro, the enemy's chase and melee attack logic are locked.
- When the intro animation finishes, the enemy resumes the normal player chase flow.
- `BP_Enemy_LittleDemon` uses `Anim_Little_Demon_rage` as its current spawn intro animation.
- Room-spawned Little Demons should therefore appear, play rage, and only then start chasing the player.

## 2026-06-20 Update: Little Demon Attack Sequence Path

- Little Demon now uses the original `Anim_Little_Demon_attack1` sequence as the configured single attack animation asset.
- `BP_Enemy_LittleDemon` currently uses the SingleNode/SimpleAnimation path for the MVP.
- `EnemyBase` can wrap configured sequence attack assets as runtime Dynamic Montages when an enemy has a valid Animation Blueprint and Slot, but the no-AnimBP path now plays sequences directly.
- `ABP_LittleDemon` exists as a transition asset, but it is not the current runtime path for Little Demon because the temporary AnimBP + runtime Dynamic Montage setup produced visible T-pose/flicker artifacts between attacks.
- The temporary `AM_LittleDemon_Attack1` asset was removed; the safer current MVP path is direct attack1 sequence playback.
- `EnemyBase` now informs `EnemyAnimInstance.bIsAttacking` during attack playback so locomotion does not keep switching under the attack Slot.
- `EnemyBase` no longer keeps the attack animation state alive for the whole attack cooldown; animation presentation timing and attack cooldown timing are separate.
- Random attack playback remains removed from the MVP.
- The existing fallback animation path remains for future enemies that do not yet have a dedicated AnimBP.

## 2026-06-19 Update: Little Demon AnimBP Direction

- Little Demon may later move from `EnemySimpleAnimationComponent` SingleNode playback toward a dedicated AnimBP once the graph is stable.
- `EnemyAnimInstance` provides shared runtime variables for enemy AnimBP graphs.
- `ABP_LittleDemon` is not currently assigned to `BP_Enemy_LittleDemon`.
- The current graph is idle/run locomotion plus `DefaultSlot` attack playback; death state is still a later refinement.

## 2026-06-19 Update: Attack Rotation and Separation Rule

- During attack animation playback, enemies should face the player at attack start and keep that rotation until the attack animation ends.
- Attack-time separation should not be applied through movement input, because movement-oriented rotation can fight the attack-facing lock.
- Current `EnemyBase` uses a small swept world offset for attack separation, disables movement-oriented rotation, ignores attack root motion, and clears/restores those locks when the attack animation finishes or the enemy dies.
- If more natural attack turning is needed later, move it into a dedicated AnimBP/Montage setup instead of per-tick actor rotation.

## 2026-06-15 Update: Little Demon Attack Blend MVP

- `EnemyBase` now tries to play attack animations through `PlaySlotAnimationAsDynamicMontage` only when a valid AnimBP class and Slot path are available.
- Enemies without a dedicated AnimBP class directly play the configured sequence through SingleNode animation.
- Direct SingleNode sequence playback is the current Little Demon MVP path.
- `AttackAnimationSlotName`, `AttackAnimationBlendInTime`, and `AttackAnimationBlendOutTime` are exposed for attack transition tuning.
- The current goal is stable attack playback using a single configured `AttackAnimation` asset.
- Enemy separation continues during the attack animation lock, so enemies can still spread away from nearby enemies while attacking.
- If a dedicated Little Demon AnimBP is added later, this Montage path should become the primary attack playback route.
- Damage timing is still code-based distance/cooldown logic; AnimNotify timing remains a later improvement.

## 2026-06-14 Update: Enemy Separation Movement

- `EnemyBase` now mixes a small separation direction into its player chase movement.
- Nearby `EnemyBase` actors inside `SeparationRadius` push the enemy away from crowd overlap.
- `EnemyStatsDataAsset` owns `SeparationRadius`, `SeparationStrength`, and `MaxSeparationNeighbors`.
- This keeps Pawn collision as overlap while reducing the case where spawned enemies visually stack into one enemy.
- Separation is updated on a short interval and smoothed before being applied, so enemies do not jitter from frame-by-frame direction flips.
- Little Demon currently uses a softer `SeparationStrength` of `0.22`.
- While attacking, `EnemyBase` applies only `35%` of the configured separation strength to reduce visible sliding.
- The current MVP implementation is acceptable for small room encounters; larger waves may later need batched queries, crowd avoidance, or a dedicated movement component.

## 2026-06-14 Update: Little Demon Attack Animation MVP

- `EnemyBase` now has a common single `AttackAnimation` slot.
- When a melee attack successfully damages the player, `EnemyBase` plays one attack animation.
- `BP_Enemy_LittleDemon` currently uses `Anim_Little_Demon_attack1` as its only attack animation.
- The earlier `attack1~attack6` random playback idea is removed from the MVP because some attack assets can introduce spin-like presentation issues.
- While the attack animation lock is active, `EnemyBase` does not start another melee attack and does not let idle/run animation override the attack.
- Before and during the attack animation, `EnemyBase` faces the player so the attack pose is aimed at the target.
- The attack damage timing is still the existing distance/cooldown check; animation notify timing can be added later.

## 2026-05-30 ?�데?�트: Enemy 최종 구조 방향

- ?�재 추천 구조??`C++ EnemyBase -> BP_EnemyBase -> BP_Enemy_Basic/Fast/Tank/Ranged`?�다.
- `EnemyBase`??체력, ?��??�색, ?�격, ?�망, 공통 ?�벤??같�? 로직 기반???�당?�다.
- `BP_EnemyBase`??Mesh, Animation Blueprint, 공통 ?�펙?? Collision 같�? ?�셋 ?�결??공통 부모로 ?�용?�다.
- `EnemyStatsDataAsset`?� ??개체??체력, ?�동?�도, ?��? 거리, 캡슐 겹침 ?�유거리, 공격?? 공격범위, 공격 쿨�??�을 관리하??Data Asset ?�?�으�?구현?�었??
- `EnemySpawnDataAsset`?� ?�재 ?�역 ?�스???�폰 ?�정?�로 ?��??�되, �??�위 ?�폰 ?�이???��?�??�사?�할 ???�다.
- ?�식 진행 방향?� 무한 ?�성보다 �?구역 ?�위 ?�투?�서 ?�해�??�량 ?�는 ?�이브로 ?�장?�는 구조??
- ?�의 메시/?�켈?�톤/?�니메이?��? 캐릭?�별�??��??�고, UE5 Manny ?�니메이??공유가 ?�요?�면 IK Rig / IK Retargeter�??�용?�다.
- ?�세??구조 기�??� `Docs/EnemyArchitecture.md`�??�른??
- ???�니메이??리�?�?기�??� `Docs/EnemyAnimationRetargeting.md`�??�른??

## 2026-05-30 ?�데?�트: Enemy Data Asset 관�?기�?

- ??개체 ?�력치�? ???�폰 규칙?� 분리?�서 관리한??
- ?�의 체력, ?�동?�도, ?��? 거리, 캡슐 겹침 ?�유거리, 공격?? 공격 ?�거�? 공격 쿨�??��? `EnemyStatsDataAsset`?�로 관리한??
- `EnemyBase`??`StatsDataAsset`??지?�되???�으�?Data Asset 값을 ?�선 ?�용?�고, 지?�되지 ?�았?�면 기존 C++ 기본값을 ?�용?�다.
- ?�레?�어 Capsule�???Capsule?� Pawn 채널??Overlap?�로 처리?�다. ?�레?�어가 ?�에�??�러?�여 ?�동 불�?가 ?�거??공격 ?�진 ?�텝?�로 ?�을 밀?�내???�황???�하�? ?��?지??거리/쿨�????�정?�로 처리?�다.
- ?�재 ?�역 ?�스???�폰 ?�간, 최�? ???? ??번에 ?�성???? ?�폰 거리??`EnemySpawnDataAsset`?�로 관리한??
- �??�리?�형 MVP?�서??방별 SpawnPoint, ???�량, ??조합, ?�이�??��? `RoomCombatActor` ?�는 `RoomEncounterDataAsset`?�서 관리하??방향???�선?�다.
- ?�재 ?�폰 ?�정 기본 경로??`/Game/Data/Enemy/Spawn/DA_EnemySpawn_Default`?�다.
- Enemy 관??Data Asset ?�더??`/Game/Data/Enemy/Spawn`, `/Game/Data/Enemy/Stats`, `/Game/Data/Enemy/Waves`�??�눈??
- ???�폰 관???�업 ?�에??`Docs/EnemySpawnSystem.md`�?먼�? ?�인?�다.

## 1. 문서 목적

??문서??게임???�장?????�?�과 구현 ?�선?�위�??�리?�다.
Codex가 ?�중????AI�?구현???? �??�의 ??���??�이?��? ?�동?��? ?�도�?기�????�공?�다.

?�재 ?�계?�서??코드 구현???�니???�계 기�?�??�리?�다.

## 2. ?�재 ?�정??방향

- ?��? ?�러 ?�?�으�??�누???�투???�양?�을 만든??
- 근거�?공격?? ?�거�?공격?? 빠른 ?�도???��? ?�수 ?�보�?본다.
- 체력 ?�스?�과 ?�망 ?�니메이??기능?� 기존 구현???�사?�한??
- 모델링과 ?�니메이?��? 개발?��? 직접 ?�작?�다.
- Codex???�의 ?�동, 공격 조건, ?��?지 처리, ?�수 ?�턴 로직 구현???�당?�다.
- ???�스?��? 공통 EnemyBase�?기�??�로 ?�장?�는 방향???�선 검?�한??
- ?��? 무한 ?�성보다 �?구역 ?�위 ?�투?�서 ?�해�??�량 ?�는 ?�이브로 ?�장?�는 방향???�선?�다.

## 3. ?�직 ?�의 중인 ?�용

- 최종 ??종류 개수
- �??�의 ?�확???�름
- ?�의 ?�계관/?�마
- ?�별 ?�확???�력�?- ?�리???�과 보스??구분
- 방별 ???�폰 ?�량/조합
- 방별 ?�이�???- ?�리?�방/보스�?구성
- ??보상 ?�롭 방식
- ?�별 ?�용 ?�니메이?�을 ?�작?��?, Manny ?�니메이?�을 리�?겟해???�용?��? ?��?
- 캐릭?�별 IK Rig / IK Retargeter 구성 방식

## 4. ???�???�보

| ???�??| ??�� | ?�징 | Codex 구현 ?�이??| ?�선?�위 | 비고 |
|---|---|---|---|---|---|
| 근거�?추격??| 가??기본 ??| ?�레?�어?�게 ?�근?�서 근거�?공격 | ?��? | MVP | ?�재 ?�스?�용 ?�모 ?�의 방향�?가??가깝다 |
| Little Demon | ?�형 ?�마 근거�???| 빠른 ?�도???�규 근거�????�보 | ?��? | MVP ?�장 | `/Game/Demons_Big_Pack/Little_Demon` ?�셋 기반 |
| 빠른 추격??| ?�박????| 체력?� ???�??�동?�도가 빠름 | ?��?~보통 | MVP | ?�레?�어 ?�동??강제�??�도?????�다 |
| ?�커??| 버티????| ?�동?�도???�리지�?체력???�음 | ?��? | MVP ?�는 1�??�장 | 기본 ?�력�?차이만으�?구현 가??|
| ?�거�??�격??| 거리 ?��? 공격 ??| ?�정 거리?�서 ?�사체로 공격 | 보통 | MVP ?�는 1�??�장 | ?�사체�? 거리 ?��? 로직 ?�요 |
| ?�폭??| ?�험 구역 ?�박 ??| ?�근 ????��?�여 범위 ?�해 | 보통 | 1�??�장 | ??�� 범위?� ?�망 처리 구분 ?�요 |
| ?�판??| 공간 ?�어 ??| 바닥???�험 구역 ?�성 | 보통 | 1�??�장 | 지???�해 ?�는 ?�동 ?�한 로직 ?�요 |
| ?�격형 | ?�거�?고위????| �?거리?�서 조�? ??강한 공격 | 보통~?�려?� | 2�??�장 | 조�??? 경고 ?�시, ?�피 ?�간 ?�요 |
| ?�환??| ?�장 복잡??증�? | ?�정 ?�간마다 ?�형 ???�성 | 보통~?�려?� | 2�??�장 | ?�환 ???�한�?관�?로직 ?�요 |
| 방어 버프??| 지?�형 ??| 주�? ?�에�?보호�??�는 방어 버프 | ?�려?� | 2�??�장 | 버프 ?�용/?�제?� ?�??관�??�요 |
| ?�리????| 강화 변??| 기존 ?�의 강화 버전 | 보통 | 1�??�장 ?�후 | ?�상, ?�기, ?�력�? ?�턴 변???�보 |

## 5. MVP ?�선 구현 ??
초기?�는 ?�래 4종을 ?�선 구현 ?�보�?본다.

1. 근거�?추격??2. 빠른 추격??3. ?�커??4. ?�거�??�격??
?�유:

- 구현 ?�이?��? 비교????��.
- `HealthComponent`?� 기본 AI만으�??�스??가?�하??
- ?�투??기본 ?�양?�을 만들 ???�다.
- ?�폭, ?�판, ?�환 같�? 복잡???�수 로직 ?�에 ?�정?�인 ??기반??만들 ???�다.

## 6. 1�??�장 ??
MVP ?�후 ?�래 ?�을 추�? ?�보�?본다.

1. ?�폭??2. ?�판??3. ?�리????
?�유:

- ?�투???�치 ?�정�??�피 ?�단??추�??�다.
- ?�순 추격/?�격�?반복?�는 ?�투�?보완?�다.
- 기존 EnemyBase 구조가 ?�정????구현?�기 ?�합?�다.

## 7. 2�??�장 ??
?�래 ?��? ?�중??검?�할 ?�보�??�다.

1. ?�격형
2. ?�환??3. 방어 버프??
?�유:

- 구현 ?�이?��? ?��??�으�??�다.
- 조�??? ?�환 관�? 버프 ?�용/?�제 ??추�? ?�스?�이 ?�요?�다.
- 초기 MVP 범위?�는 ?�함?��? ?�는??

## 8. 추천 ?�래??블루?�린??구조

?�래 구조???�보?�이?? ?�제 구현 ?�에???�재 ?�로?�트?�서 ?�용 중인 Blueprint?� C++ ?�결 ?�태�??�시 ?�인?�다.

- `BP_EnemyBase`
  - 공통 체력 처리
  - ?�망 처리
  - ?�동?�도
  - 공격??  - 공격 ?�거�?  - 공격 쿨�???  - ?�레?�어 ?��?

- `BP_Enemy_Melee`
  - 근거�?추격??
- `BP_Enemy_Fast`
  - 빠른 추격??
- `BP_Enemy_Tank`
  - ?�커??
- `BP_Enemy_Ranged`
  - ?�거�??�격??
- `BP_Enemy_Exploder`
  - ?�폭??
- `BP_Enemy_AreaHazard`
  - ?�판??
## 9. Enemy Data 기�?

?�재 구현??`EnemyStatsDataAsset` ??��:

| ??�� | ?�명 |
|---|---|
| EnemyID | ??고유 ID |
| EnemyName | ???�름 |
| MaxHealth | 최�? 체력 |
| MoveSpeed | ?�동?�도 |
| StopDistance | ?�레?�어?� ?��??�려??기본 ?��? 거리 |
| PersonalSpacePadding | ???�레?�어 캡슐??겹치지 ?�도�??�하???�유거리 |
| AttackDamage | 공격??|
| AttackRange | 공격 ?�거�?|
| AttackCooldown | 공격 쿨�???|

추후 ?�장 ?�보:

| ??�� | ?�명 |
|---|---|
| EnemyType | ???�??|
| DetectionRange | ?�레?�어 ?��? 범위 |
| RewardEXP | 처치 ??경험�?보상 |
| RewardScrap | 처치 ???�화 보상 |
| SpecialPattern | ?�수 ?�턴 |

## 10. 구현 ?�서 ?�보

1. EnemyBase ?�계
2. HealthComponent ?�결 ?�인
3. 근거�?추격??구현
4. Little Demon ??구조 ?�인
5. `RoomCombatActor` 기반 �??�위 ???�폰 구현
6. 빠른 추격??구현
7. ?�커??구현
8. ?�거�??�격??구현
9. ?�폭???�판??구현
10. ?�리?�방/보스�???구성 검??11. ?�격형/?�환??방어 버프??검??
## 11. Codex 구현 주의?�항

- 구현 ??`AGENTS.md`�??�른??
- 구현 ??`Docs/CurrentImplementation.md`�??�고 ?�재 ?�스????구조�??�인?�다.
- ?�제 ?�용 중인 ?�래??구조??`Docs/10_ProjectClassMap.md`가 ?�으�??�인?�다.
- ???�니메이?? Skeleton, IK Rig, IK Retargeter 관???�업 ?�에??`Docs/EnemyAnimationRetargeting.md`�??�인?�다.
- `HealthComponent`�??�구?�하지 말고 기존 기능???�사?�한??
- ?�망 ?�니메이??기능??기존 구현�?충돌?��? ?�도�??�다.
- ?�러 ???�?�을 ??번에 구현?��? ?�는??
- EnemyBase�?먼�? ?�정?�한 ???�?�별 ?�을 ?�나??추�??�다.
- 모델�??�니메이?��? 개발?��? 직접 ?�작?��?�?Codex??로직 구현??집중?�다.
- ?�용?��? ?�는 C++ ?�래?�에 ??로직??추�??��? ?�는??
- 구현 ???�정???�일, ?�성???�일, ?�결??Blueprint, ?�스??방법???�약?�다.
- ?�을 구현???�는 ??문서�?먼�? ?�고 ?�정???�용�??�보 ?�용??구분?�다.

## 12. EnemyBase ?�환 계획 ?�보

?�재 ?�스???��? `EnemySpawnSubsystem`, `EnemyBase`, `SurvivorSquareEnemy`�?중심?�로 ?�작?�다.
??구조???�투 ?�름 검증용 MVP?�며, 추후 ?�식 ?�테?��?/?�이�??�스?�과 ???�???�장?�로 분리???�보?�다.

?�재 ?�시 구조:

| ??�� | ?�재 구현 | 비고 |
|---|---|---|
| ??공통 기반 | `EnemyBase` | 체력, ?�동, 목표 ?�치, 근거�?공격, ?�망 ??Destroy ?�당 |
| ?�규 Little Demon ??| `BP_Enemy_LittleDemon` | Little Demon 메시, SingleNode/SimpleAnimation idle/run, rage ?�폰 ?�트�? attack1, DeathAnimation, StatsDataAsset ?�결 |
| ?�스?????�터 | `SurvivorSquareEnemy` | `EnemyBase`�??�속?�는 ?�모 Mesh 변??|
| ??체력 | `EnemyBase`??`HealthComponent` | ?�사???��? |
| ???�망 | `EnemyBase`???�망 ?�벤?�에??Destroy | 공통?�됨 |
| ???�폰 | `EnemySpawnSubsystem` | ?�역 ?�스???�폰/?�시 검증용, 추후 `RoomCombatActor` ?�는 Stage/WaveSystem ?�환 ?�보 |
| ?�의 ?�레?�어 공격 | `EnemyBase` | ?�후 복잡?��?�?EnemyAttackComponent 분리 ?�보 |

?�환 ?�보:

1. `EnemyBase` 기반 ?�스???�이 기존 ?�투 루프�??��??�는지 ?�인?�다.
2. ?�재 공격?? 공격 ?�거�? 공격 쿨�??��? `EnemyBase`???�다.
3. ?�역 ?�스?????�성?� `EnemySpawnSubsystem`???�당?�다.
4. ?�식 �??�투 ???�성?� `RoomCombatActor`?� `RoomSpawnPoint` ?�보 구조�???��??
5. ?�스???�을 근거�?추격??기�??�로 ?��??�다.
6. 빠른 ?�과 ?�커 ?��? `EnemyBase`??체력/?�동?�도 ?�치 차이�?먼�? ?�장?�다.
7. ?�거�??��? ?�사�?구조�??�계????추�??�다.

주의:

- EnemyBase ?�환�??�러 ???�??추�?�???번에 ?��? ?�는??
- 기존???�작 ?�인???�스???�투 루프�?깨�? ?�도�??��? ?�계�???��??
- ?�거�??��? ?�사�?구조가 ?�요?��?�?근거�?빠른/?�커보다 ?�에 ?�다.
