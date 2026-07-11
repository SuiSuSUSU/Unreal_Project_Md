---
Status: ACTIVE
Scope: Enemy Design
Last Updated: 2026-07-11
Source of Truth: Yes
---

# EnemySystem.md

## 1. 문서 목적

이 문서는 현재 프로젝트의 적 gameplay 기준을 정리한다.

EnemySystem은 적의 공통 행동, 현재 구현된 적 타입, 체력/공격/추격/사망/활성화 기준을 다룬다. 적 클래스 구조의 세부 책임은 `Docs/EnemyArchitecture.md`, 적 스폰과 방 배치는 `Docs/EnemySpawnSystem.md`와 `Docs/RoomCombatSystem.md`, 적 애니메이션 세부는 `Docs/AnimationSystem.md`와 `Docs/EnemyAnimationRetargeting.md`를 따른다.

## 2. 현재 방향

- 현재 적 시스템의 공통 기반은 `EnemyBase`다.
- Little Demon, Fast, Tank는 모두 `EnemyBase` 기반 Blueprint 적이다.
- 적은 무한 자동 생성보다 Room/Stage 전투 안에서 정해진 수량, 순차 스폰, 배치형 적으로 등장하는 방향이다.
- 전역 테스트 스폰은 `EnemySpawnSubsystem`에 남아 있지만 현재 게임의 주 스폰 구조가 아니다.
- 현재 주요 전투 기준은 Room 진입, 적 활성화/스폰, 적 전멸, Room Clear, Stage 진행이다.
- 적 외형, 메시, 스켈레톤, AnimBP, Montage, 개별 연출은 C++에 하드코딩하지 않고 Blueprint/Asset에서 설정한다.

## 3. 현재 구현 파일

| 구분 | 파일 | 역할 |
| --- | --- | --- |
| 적 공통 기반 | `Source/MyProject/Enemies/EnemyBase.h/.cpp` | 체력, 추격, 근접 공격, 활성화 상태, 분리 이동, 스폰 인트로, 사망 처리 |
| 원거리 적 더미 | `Source/MyProject/Enemies/EnemyRangedDummy.h/.cpp` | 사거리 접근, 정지/조준, 예고, 단발 직선 투사체 발사 |
| 적 직선 투사체 | `Source/MyProject/Enemies/EnemyStraightProjectile.h/.cpp` | 비유도 이동, 충돌, 플레이어 `HealthComponent` 피해, 교체 가능한 연출 컴포넌트 |
| 적 스탯 데이터 | `Source/MyProject/Enemies/EnemyStatsDataAsset.h/.cpp` | 적 ID, 이름, 체력, 이동, 분리, 공격 수치 |
| 적 AnimBP 변수 | `Source/MyProject/Enemies/EnemyAnimInstance.h/.cpp` | GroundSpeed, 이동 상태, 공격 상태, 사망 상태 |
| 전역 테스트 스폰 | `Source/MyProject/Enemies/EnemySpawnSubsystem.h/.cpp` | 비활성 기본값의 전역 테스트 스폰 경로 |
| 전역 테스트 스폰 데이터 | `Source/MyProject/Enemies/EnemySpawnDataAsset.h` | 테스트 스폰용 적 클래스/가중치/간격 데이터 |
| Room 전투 실행 | `Source/MyProject/Rooms/RoomCombatActor.h/.cpp` | 방 입장, 적 스폰/활성화, 생존 적 추적, Room Clear |
| Room 적 구성 데이터 | `Source/MyProject/Rooms/RoomEncounterDataAsset.h` | 방별 Initial/Wave 스폰, 배치형 적 활성화 정책, Clear 조건 |

## 4. 현재 적 Blueprint 기준

| 적 | 현재 상태 | 기준 |
| --- | --- | --- |
| `BP_Enemy_LittleDemon` | 구현된 MVP 적 | Little Demon 기반 소형 근접 적 |
| `BP_Enemy_Fast` | 구현된 MVP 적 | DemonHeavy 외형을 사용하는 빠른 압박형 적 |
| `BP_Enemy_Tank` | 구현된 MVP 적 | Butcher 외형을 사용하는 느린 고체력 적 |

`AEnemyRangedDummy`와 `AEnemyStraightProjectile`의 C++ gameplay 골격은 빌드 검증되었다. `/Game/Blueprints/Enemy/Ranged/BP_Enemy_RangedDummy`는 SkeletonMage Mesh, `ThirdPerson_AnimBP`, `hand_r`에 연결된 지팡이를 사용하고 `BP_Enemy_StraightProjectile`과 연결되어 있다. `TEST_Enemy_RangedDummy`는 PlayerStart에서 1000cm 떨어진 위치에 독립 배치되어 있다. PIE에서 약 650cm까지 접근, 정지/플레이어 방향 조준, 직선 투사체 발사, 플레이어 `HealthComponent`에 10 피해 적용을 확인했다. 전용 캐스팅 애니메이션, 예고 VFX, 최종 발사점, 투사체 Niagara/Sound, Dormant/Shock 시각 검증은 아직 남아 있다. Start Stage와 `RoomEncounterDataAsset`에는 추가하지 않는다.

현재 기준:

- 세 적 모두 `EnemyBase`, `HealthComponent`, `EnemyStatsDataAsset` 흐름을 사용한다.
- Fast와 Tank는 전용 AnimBP/Montage 경로를 사용한다.
- Little Demon은 현재 안정성을 우선해 SingleNode/SimpleAnimation 계열 MVP 경로를 유지한다.
- 새 적을 추가할 때는 먼저 `EnemyBase` 기반 Blueprint와 `EnemyStatsDataAsset` 값 차이로 검증한다.
- 새 적 전용 C++ 클래스를 바로 만들지 않는다.

## 5. EnemyBase 책임

`EnemyBase`는 적이 공통으로 가져야 하는 gameplay 로직을 담당한다.

현재 책임:

- `HealthComponent` 보유와 사망 이벤트 연결
- `EnemyStatsDataAsset` 값 적용
- 플레이어 탐색과 추격
- 하위 적 행동을 위한 `UpdateCombatBehavior()` 확장점. 기본 구현은 기존 추격/근접 공격을 그대로 사용
- 플레이어와의 목표 거리 유지
- 근접 공격 거리/쿨타임 판정
- 약공격/강공격 애니메이션 재생 요청
- 공격 성공 시 플레이어에게 데미지 적용
- 공격 후 플레이어 방향으로 부드럽게 회전하는 보정
- 적끼리 겹치지 않게 하는 soft separation
- Dormant/Active/Dead 활성화 상태 관리
- 스폰 인트로 애니메이션 동안 추격/공격 잠금
- 사망 애니메이션, 충돌 비활성화, Destroy 여부 처리

`EnemyBase`가 직접 담당하지 않는 것:

- 방별 적 조합과 웨이브 설계
- Stage 진행 순서
- 보상 생성/선택
- 최종 적 종류 기획
- 특정 적의 메시/스켈레톤/AnimBP 하드코딩
- 적별 VFX, 피격 이펙트, 보스 전용 연출
- 복잡한 HitReact/경직/넉백 정책
- 원거리 투사체, 장판, 소환 같은 특수 패턴의 적별 구현

## 6. 활성화 상태 기준

`EnemyBase`는 `EEnemyActivationState`를 사용한다.

| 상태 | 의미 |
| --- | --- |
| `Dormant` | 살아 있지만 추격/공격하지 않는 대기 상태 |
| `Active` | 플레이어를 추격하고 공격할 수 있는 상태 |
| `Dead` | 사망 상태 |

현재 기준:

- `ActivateEnemy()`는 Dormant 적을 Active 상태로 만든다.
- `DeactivateEnemy()`는 적을 Dormant 상태로 만들고 추격/공격을 멈춘다.
- `IsAlive()`는 Dead 여부와 Health 상태를 기준으로 적 추적에 사용한다.
- Room에 미리 배치된 적은 `RoomCombatActor.PlacedEnemyActors`로 등록할 수 있다.
- 배치형 적은 Room 시작 시 바로 활성화하거나, 플레이어 감지 거리 안에 들어왔을 때 활성화할 수 있다.

## 7. 이동과 분리 기준

현재 적 이동은 플레이어 추격과 soft separation을 함께 사용한다.

현재 기준:

- 적은 플레이어 위치를 목표로 이동한다.
- `StopDistance`와 캡슐 반경을 고려해 플레이어와 너무 겹치지 않도록 한다.
- 적끼리는 `SeparationRadius` 안의 주변 `EnemyBase`를 확인해 살짝 떨어지는 방향을 섞는다.
- `SeparationStrength`와 `MaxSeparationNeighbors`는 `EnemyStatsDataAsset`에서 조정한다.
- 공격 중에는 `AttackSeparationStrengthMultiplier`로 분리 강도를 줄인다.
- 분리 이동은 적이 겹쳐 한 덩어리처럼 보이는 문제를 줄이기 위한 것이다.

주의:

- 현재 separation은 소규모 Room 전투 기준으로 충분한 단순 구현이다.
- 대규모 웨이브나 수십 마리 전투에서는 별도 군중/회피 구조가 필요할 수 있다.
- 적이 플레이어를 물리적으로 밀어내거나 가두는 구조로 만들지 않는다.

## 8. 공격 기준

현재 적 공격은 근접 거리와 쿨타임 기반이다.

현재 기준:

- 적이 플레이어와 가까워지면 `AttackRange` 기준으로 공격을 시도한다.
- 공격 성공 시 플레이어 `HealthComponent::ApplyDamage`를 호출한다.
- 공격이 실제 데미지를 적용하면 플레이어 피격 반응을 요청할 수 있다.
- 일반 공격은 Light, 강공격은 Heavy 피격 반응을 요청한다.
- 적 공격 타이밍은 아직 공격 애니메이션 Notify 기반이 아니라 거리/쿨타임 로직 기반이다.
- 공격 애니메이션은 presentation이며, 데미지 판정 자체를 결정하지 않는다.

원거리 더미 예외 기준:

- `EnemyRangedDummy`는 `EnemyBase`의 체력/활성화/피격/사망 상태를 재사용하고 행동 함수만 재정의한다.
- 사거리 밖에서는 접근하고, 사거리 안에서는 정지해 플레이어를 바라본 뒤 짧게 예고하고 직선 비유도 투사체 한 발을 발사한다.
- 투사체는 플레이어 충돌 시 기존 `HealthComponent`에 피해를 적용한다.
- 후퇴, 좌우 이동, 유도, 다중 발사, 상태이상은 현재 범위가 아니다.
- 공격 예고와 발사 시점에는 Blueprint presentation 이벤트를 제공하며, 특정 외형/VFX/Sound 경로는 C++에 넣지 않는다.
- 공격 예고 중 Dormant 또는 Shock 상태가 시작되면 예고를 완전히 취소하며, 상태 해제 후 사거리 안에서 전체 예고 시간을 처음부터 다시 진행한다.

약공격/강공격 기준:

- `AttackAnimation`은 기본 약공격 애니메이션이다.
- `StrongAttackAnimation`은 강공격 후보 애니메이션이다.
- `StrongAttackEveryNthAttack` 값이 0보다 크면 N번째 성공 공격마다 강공격을 사용할 수 있다.
- `StrongAttackDamageMultiplier`와 `StrongAttackRangeMultiplier`로 강공격 gameplay 값을 다르게 줄 수 있다.

## 9. 현재 적별 공격/애니메이션 기준

| 적 | 현재 기준 |
| --- | --- |
| Little Demon | `Anim_Little_Demon_attack1` 단일 공격을 안정 MVP로 사용. 랜덤 attack1~6 재생은 제거됨. |
| Fast / DemonHeavy | 전용 `ABP_DemonHeavy_Fast`와 약/강 공격 Montage를 사용. 세 번째 성공 공격에서 강공격 후보를 사용. |
| Tank / Butcher | 전용 `ABP_Butcher`와 Butcher 약/강 공격 Montage를 사용. 무겁고 느린 공격감을 우선. |

주의:

- 여러 공격 애니메이션을 추가할 때는 먼저 한 적에서 자연스러운 Montage/Blend/Notify 구간을 검증한다.
- 애니메이션이 툭 끊기거나 회전하는 문제는 공통 `EnemyBase` 로직보다 개별 AnimBP/Montage/Blend 설정을 먼저 확인한다.
- Little Demon은 과거 랜덤 공격 재생에서 회전/전환 문제가 있었으므로 현재 단일 attack1 기준을 유지한다.

## 10. 스폰 인트로 기준

`EnemyBase`는 선택적으로 스폰 인트로 애니메이션을 재생할 수 있다.

현재 기준:

- `SpawnIntroAnimation`이 있으면 BeginPlay 또는 스폰 직후 인트로를 재생한다.
- 인트로 중에는 추격과 공격을 잠근다.
- `SpawnIntroFollowupAnimation`과 `SpawnIntroFollowupDuration`으로 짧은 후속 대기를 줄 수 있다.
- `SpawnIntroTurnToPlayerDuration`으로 추격 시작 전 플레이어 방향으로 부드럽게 회전할 수 있다.
- Little Demon은 rage 인트로와 idle follow-up 대기 흐름을 사용한다.

주의:

- 스폰 인트로는 적의 외형/연출 영역이므로 적 Blueprint와 Animation 문서를 함께 확인한다.
- 스폰 인트로 중 HitReact나 공격을 강제로 끼워 넣지 않는다.

## 11. 체력과 사망 기준

적 체력은 `EnemyBase`의 `HealthComponent`가 담당한다.

현재 기준:

- 적 최대 체력은 `EnemyStatsDataAsset.MaxHealth`가 있으면 그 값을 우선 사용한다.
- 사망 시 `EnemyBase::HandleDeath`가 호출된다.
- 사망하면 활성화 상태는 Dead로 간주한다.
- 설정에 따라 충돌을 끄고, 사망 애니메이션을 재생하고, Destroy 여부를 처리한다.
- `RoomCombatActor`는 적 사망 이벤트를 받아 `AliveEnemies`, `TrackedSpawnedEnemies`, `TrackedPlacedEnemies`를 갱신한다.
- 적 전멸 여부는 Room Clear 조건의 입력이다.

주의:

- 적 사망 이벤트는 Stage Clear를 직접 결정하지 않는다.
- Stage 진행은 `StageFlowManager`와 Room 진행 구조가 담당한다.
- 적별 사망 VFX나 보스 사망 연출은 아직 별도 설계 대상이다.

## 12. Electric Stack / Shock 기준

`EnemyBase`는 Lightning 빌드 확장을 위해 Electric Stack과 Shock 상태를 가진다.

현재 MVP 기준:

- Lightning 직접 타격을 받은 적은 `ElectricStack +2`를 받는다.
- Chain Lightning 전이 피해를 받은 적은 `ElectricStack +1`을 받는다.
- `ElectricStack`이 5 이상이 되면 Shock 상태가 발동한다.
- Shock 지속 시간은 2초다.
- Shock 발동 시 `ElectricStack`은 0으로 초기화한다.
- Shock 상태 중에는 추가 Electric Stack을 받지 않는다.
- 4초 동안 추가 전기 피해를 받지 않으면 `ElectricStack`은 0으로 사라진다.
- Shock 상태 중에는 적의 이동/공격 Tick을 멈추고 이동 컴포넌트를 정지시킨다.
- Shock 상태 중에는 적 Mesh 애니메이션을 일시 정지한다.
- Shock 상태 중에는 몸에 `/Game/SlashTrail_SoftTofu/Niagara/Lightning/NS_Sword_Lightning`을 attached한다.
- `IsShocked()`와 `GetElectricStack()`으로 보상/상태 로직이 현재 상태를 확인할 수 있다.

현재 Shock은 정지와 시각 표시만 담당한다.

- 추가 피해 없음
- 둔화 없음
- 폭발 없음
- 전파 없음

현재 Shock VFX는 임시로 `NS_Sword_Lightning`을 사용한다. 최종 전용 Shock Niagara가 정해지면 `EnemyBase`의 Shock VFX hook에서 교체한다.

## 13. EnemyStatsDataAsset 기준

`EnemyStatsDataAsset`은 적 한 종류의 gameplay 수치를 정의한다.

현재 필드:

| 값 | 의미 |
| --- | --- |
| `EnemyID` | 적 고유 ID |
| `EnemyName` | 표시용 적 이름 |
| `MaxHealth` | 최대 체력 |
| `MoveSpeed` | 이동 속도 |
| `StopDistance` | 플레이어와 유지할 최소 거리 |
| `PersonalSpacePadding` | 캡슐 겹침 방지를 위한 추가 거리 |
| `SeparationRadius` | 주변 적과 분리 계산을 시작할 반경 |
| `SeparationStrength` | 분리 방향을 섞는 강도 |
| `MaxSeparationNeighbors` | 분리 계산에 사용할 주변 적 최대 수 |
| `AttackDamage` | 공격 데미지 |
| `AttackRange` | 공격 거리 |
| `AttackCooldown` | 공격 쿨타임 |

주의:

- 스폰 수량, 방별 조합, 웨이브, 보상, VFX, 애니메이션 에셋은 `EnemyStatsDataAsset`에 넣지 않는다.
- `EnemyStatsDataAsset`은 “이 적이 얼마나 강하고 빠른가”만 담당한다.
- 방별 구성은 `RoomEncounterDataAsset` 또는 `RoomCombatActor` 설정을 따른다.

## 14. Room 전투와의 관계

현재 적은 Room 단위 전투에서 주로 사용된다.

현재 기준:

- Room 시작 시 `RoomCombatActor`가 적을 스폰하거나 배치형 적을 활성화한다.
- 스폰형 적은 `InitialSpawnEntries`, `WaveSpawnEntries`, `SpawnDelay` 흐름을 사용할 수 있다.
- 배치형 적은 `PlacedEnemyActors`로 등록해 Room 전투 대상에 포함할 수 있다.
- Room Clear 조건은 `AllTrackedEnemiesDefeated`, `SpawnedEnemiesDefeated`, `PlacedEnemiesDefeated` 중 하나가 될 수 있다.
- 적별 행동은 `EnemyBase`가 담당하고, 어떤 적이 몇 마리 나오는지는 Room/Encounter 쪽이 담당한다.

현재 유지할 중요한 구분:

- `EnemyBase`: 적 한 마리의 행동
- `EnemyStatsDataAsset`: 적 한 종류의 수치
- `RoomEncounterDataAsset`: 한 방의 적 구성 데이터
- `RoomCombatActor`: 한 방의 전투 실행과 적 추적
- `StageFlowManager`: Room 진행 순서와 Stage Clear 관리

## 14. HitReact와 피격 연출 기준

현재 적 HitReact는 정식 구현되어 있지 않다.

현재 기준:

- 공통 흰색 플래시 피격 연출은 제거되었다.
- `EnemyBase`가 모든 적에게 강제로 움찔/경직/넉백을 넣지 않는다.
- 피격 VFX, 피, Hit Distortion은 현재 플레이어 공격 확정 히트 경로와 VFX 문서 기준을 따른다.
- 향후 HitReact는 데미지, 공격 타입, 콤보 단계, 보상 보정, 적 저항값을 비교하는 구조가 우선 후보이다.
- Boss/MiniBoss는 일반 적과 다른 면역 또는 특수 반응 창을 가질 수 있다.

장기 방향 예시:

```text
HitReactScore = DamageAmount + AttackTypeBonus + ComboStepBonus + RewardBonus
if HitReactScore >= EnemyHitReactResistance:
    play allowed reaction
else:
    no reaction
```

## 15. 현재 사용하지 않는 기준

- `SurvivorSquareEnemy`와 `SurvivorSquareSpawner`는 오래된 템플릿/테스트 흐름으로 본다.
- `EnemySpawnSubsystem`은 현재 주 게임 스폰 구조가 아니라 전역 테스트 스폰 경로다.
- 무한 생존형 자동 스폰을 현재 적 gameplay 기준으로 삼지 않는다.
- Little Demon attack1~6 랜덤 재생은 현재 기준에서 제외한다.
- 공통 흰색 피격 플래시를 다시 기본 피격 피드백으로 사용하지 않는다.
- 적별 메시/애니메이션 경로를 C++ `EnemyBase`에 직접 하드코딩하지 않는다.

## 16. 다음 후보

아직 구현 또는 정리가 남은 후보:

- EnemySystem과 EnemyArchitecture의 중복 구조 추가 정리
- 적 HitReactComponent 또는 가벼운 피격 반응 요청 경로 설계
- Little Demon 전용 AnimBP 전환 여부 재검토
- Fast/Tank 공격 Notify 기반 데미지 타이밍 전환 검토
- 원거리 적, 장판 적, 폭발 적 후보 설계
- MiniBoss/Boss 적 구조 분리
- 대규모 전투용 군중 회피/성능 개선 후보 검토

## 17. 관련 문서

- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- `Docs/EnemyArchitecture.md`
- `Docs/EnemySpawnSystem.md`
- `Docs/RoomCombatSystem.md`
- `Docs/EnemyAnimationRetargeting.md`
- `Docs/AnimationSystem.md`
- `Docs/HealthSystem.md`
- `Docs/BasicAttackSystem.md`
- `Docs/VFXSystem.md`
