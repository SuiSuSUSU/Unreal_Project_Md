---
Status: ACTIVE
Scope: Health System
Last Updated: 2026-07-06
Source of Truth: Yes
---

# HealthSystem.md

## 1. 문서 목적

이 문서는 현재 프로젝트의 체력, 데미지, 회복, 사망 이벤트 기준을 정리한다.

HealthSystem은 "체력 수치와 사망 이벤트"의 공통 기반만 담당한다. 공격 판정, 보상 선택, VFX, 애니메이션 반응, 방 클리어 흐름은 각 전용 시스템에서 처리한다.

## 2. 현재 방향

- 플레이어와 적은 공통 `HealthComponent`를 기준으로 체력 처리를 한다.
- `HealthComponent`는 데미지, 회복, 최대 체력, 무적, 사망 이벤트만 담당한다.
- 플레이어 체력 HUD, 플레이어 사망 처리, 플레이어 피격 애니메이션은 `PlayerHealthSubsystem`이 담당한다.
- 적의 사망 처리와 방 클리어 추적은 `EnemyBase`와 `RoomCombatActor`가 담당한다.
- 보상으로 인한 회복이나 최대 체력 증가는 `RewardSelectionWidget`과 `PlayerStatsComponent`를 거쳐 적용한다.

## 3. 현재 구현 파일

| 구분 | 파일 | 역할 |
| --- | --- | --- |
| 공통 체력 | `Source/MyProject/Combat/HealthComponent.h/.cpp` | 체력, 데미지, 회복, 최대 체력, 무적, 사망 이벤트 |
| 플레이어 체력 흐름 | `Source/MyProject/Combat/PlayerHealthSubsystem.h/.cpp` | 플레이어 체력 컴포넌트 보장, 체력 UI, 사망 처리, 피격 반응 요청 |
| 플레이어 체력 UI | `Source/MyProject/UI/PlayerHealthOrbWidget.h/.cpp` | 원형 체력 오브 HUD |
| 플레이어 스탯 | `Source/MyProject/Combat/PlayerStatsComponent.h/.cpp` | 최대 체력 기준값과 Run 보상 최대 체력 보정 |
| 임시 보상 UI | `Source/MyProject/UI/RewardSelectionWidget.h/.cpp` | 회복, 최대 체력 Run 보상 적용 |
| 적 공통 체력 | `Source/MyProject/Enemies/EnemyBase.h/.cpp` | 적 `HealthComponent` 보유, 사망 처리 |
| 방 클리어 추적 | `Source/MyProject/Rooms/RoomCombatActor.h/.cpp` | 적 `OnDeath`를 받아 생존 적 수와 클리어 판정 갱신 |

## 4. HealthComponent 기준

`HealthComponent`는 플레이어와 적이 공유하는 최소 체력 컴포넌트다.

현재 제공 기능:

- `ApplyDamage(float DamageAmount)`
- `Heal(float HealAmount)`
- `SetMaxHealth(float NewMaxHealth, bool bFillHealth)`
- `Revive(float ReviveHealth)`
- `GetCurrentHealth()`
- `GetMaxHealth()`
- `GetHealthPercent()`
- `IsDead()`
- `SetInvulnerable(bool bNewInvulnerable)`
- `IsInvulnerable()`
- `OnHealthChanged`
- `OnDeath`

처리 규칙:

- 데미지나 회복 결과는 실제 적용된 수치만 반환한다.
- 이미 죽은 대상은 추가 데미지와 회복을 받지 않는다.
- 무적 상태에서는 데미지를 받지 않는다.
- 체력이 0 이하가 되면 `OnDeath`가 한 번 발생한다.
- `OnHealthChanged`는 UI, 로그, 외부 시스템이 구독할 수 있는 이벤트로 사용한다.

`HealthComponent`가 하지 않는 일:

- 공격 판정 계산
- 보상 카드 선택
- 적 피격 VFX 선택
- 적 HitReact나 경직 판정
- 플레이어 카메라 흔들림
- 방 클리어 판정
- Stage Clear 판정
- 저장/로드 기반 영구 성장

## 5. 플레이어 체력 흐름

플레이어 체력은 `PlayerHealthSubsystem`이 런타임에서 보장한다.

현재 흐름:

1. 현재 플레이어 Pawn을 찾는다.
2. `HealthComponent`가 없으면 런타임에 붙인다.
3. `PlayerStatsComponent`를 확인하거나 보장한다.
4. `PlayerStatsComponent::GetConfiguredMaxHealth()`를 기준으로 최대 체력을 설정한다.
5. `HealthComponent.OnHealthChanged`와 `OnDeath`에 바인딩한다.
6. `PlayerHealthOrbWidget`을 생성하고 체력 변화에 맞춰 갱신한다.
7. 사망 시 플레이어 입력/이동을 막고 사망 처리를 한다.
8. 사망 또는 재시작 흐름에서 Run 보상 스탯을 초기화한다.

디버그 기준:

- 플레이어 스탯의 `DebugDamage`는 체력 테스트용 데미지 값이다.
- 체력 디버그 표시는 `bShowHealthDebug` 기준을 따른다.
- 디버그 기능은 실제 게임 입력 설계의 기준으로 삼지 않는다.

## 6. 플레이어 사망 기준

플레이어 사망은 `PlayerHealthSubsystem`에서 처리한다.

현재 기준:

- `HealthComponent.OnDeath`가 발생하면 사망 처리가 시작된다.
- 중복 사망 처리는 `bHasHandledDeath`로 막는다.
- 플레이어 이동과 입력을 비활성화한다.
- Run 보상으로 누적된 런타임 스탯 보정은 초기화한다.
- `WeaponTrailComponent`의 런타임 보상 기반 시각 상태도 초기화한다.

주의:

- 사망은 일반 피격 반응보다 우선한다.
- 사망 상태에서 일반 HitReact나 공격 애니메이션을 재생하지 않는다.
- 최종 게임 오버 UI와 Run Clear/Restart UI는 별도 문서 기준을 따른다.

## 7. 플레이어 피격 반응 경계

플레이어 피격 애니메이션은 현재 `PlayerHealthSubsystem`에 있는 작은 MVP 경로다.

현재 타입:

- `EPlayerHitReactType::None`
- `EPlayerHitReactType::Light`
- `EPlayerHitReactType::Heavy`

현재 기준:

- 적 공격이 실제로 데미지를 적용한 뒤 `RequestPlayerHitReaction`을 호출한다.
- 약한 공격은 Light, 강한 공격은 Heavy 반응을 요청한다.
- 피격 반응은 체력 수치를 바꾸지 않는다.
- 체력 감소는 이미 `HealthComponent::ApplyDamage`에서 처리된 뒤다.

장기 방향:

- HitReact 판단 규칙은 `HealthComponent` 안에 넣지 않는다.
- 향후 적/플레이어 공통 HitReact 구조가 생기면 `AnimationSystem.md` 기준을 따른다.

## 8. 적 체력과 사망 기준

적은 `EnemyBase`가 `HealthComponent`를 보유한다.

현재 기준:

- 적 최대 체력은 `EnemyStatsDataAsset` 또는 `EnemyBase` 기본값을 따른다.
- `EnemyBase`는 `HealthComponent.OnDeath`를 받아 적 사망 처리를 한다.
- 사망 시 충돌 비활성화, 사망 애니메이션, Destroy 여부는 `EnemyBase` 설정을 따른다.
- `RoomCombatActor`는 적의 `OnDeath`를 받아 `AliveEnemies`를 갱신한다.
- 방 안의 추적 대상 적이 모두 죽으면 Room Clear 조건으로 이어진다.

주의:

- 적 사망 이벤트는 방 클리어 판단의 입력일 뿐이다.
- Stage Clear 판단은 `StageFlowManager`와 Room 진행 구조가 담당한다.
- 적 피격 반응, 피격 VFX, 피 튀김, 히트 디스토션은 HealthSystem이 아니라 Combat/VFX/Animation 문서 기준을 따른다.

## 9. 회복과 최대 체력 보상 기준

현재 구현된 보상 관련 체력 효과:

| 보상 | 타입 | 현재 처리 |
| --- | --- | --- |
| `instant_heal_20` | Instant | 선택 즉시 `HealthComponent::Heal` 호출 |
| `run_max_health_10` | Run | `PlayerStatsComponent::AddRunMaxHealthBonus` 적용 후 `HealthComponent::SetMaxHealth` 호출 |

현재 기준:

- `Heal +20`은 즉시 회복이다.
- `Max Health +10`은 `PlayerStatsDataAsset` 원본을 수정하지 않고 런타임 보정으로만 적용한다.
- Run 보상 최대 체력은 사망 또는 재시작 흐름에서 초기화된다.
- 영구 최대 체력 강화는 아직 구현하지 않는다.

장기 방향:

- 즉시 회복 아이템은 적 드랍 아이템 쪽으로 옮기는 방향이다.
- Stage Clear 보상은 현재 Run 동안 유지되는 빌드 보상 중심으로 정리한다.
- 영구 강화는 저장/로드, 로비 성장 UI, 재화 시스템이 생긴 뒤 별도 구현한다.

## 10. 무적과 회피 기준

현재 회피 중 무적은 `PlayerKeyboardMovementSubsystem`이 `HealthComponent::SetInvulnerable`을 호출하는 방식이다.

기준:

- 무적 여부는 `HealthComponent`가 보관한다.
- 무적을 언제 켜고 끌지는 회피/입력 시스템이 결정한다.
- HealthSystem은 회피 거리, 회피 쿨타임, 회피 애니메이션을 결정하지 않는다.

## 11. 제거되었거나 사용하지 않는 기준

- 적 피격 순간 공통 흰색 플래시 경로는 제거되었다.
- `HealthComponent.OnHealthChanged`에서 적 피격 연출을 직접 고르지 않는다.
- HealthSystem 문서에서 오래된 뱀서류 무한 스폰 기준을 체력 기준처럼 다루지 않는다.
- Room Clear마다 보상 카드를 주는 구조는 과거 MVP 테스트 흐름이며, 현재 방향은 Stage Clear 보상이다.

## 12. 다음 후보

아직 구현 또는 정리가 남은 후보:

- 보스 체력 UI
- 보스 사망과 Boss Stage Clear 조건
- 적 HitReactComponent
- 적별 피격/사망 애니메이션 정책
- 적 드랍 회복 아이템
- 영구 체력 강화 저장/로드
- 플레이어 사망 후 정식 Game Over UI

## 13. 관련 문서

- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/RewardSystem.md`
- `Docs/AnimationSystem.md`
- `Docs/VFXSystem.md`
- `Docs/EnemySystem.md`
- `Docs/RoomCombatSystem.md`
