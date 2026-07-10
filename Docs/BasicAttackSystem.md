---
Status: ACTIVE
Scope: Basic Attack
Last Updated: 2026-07-06
Source of Truth: Yes
---

# BasicAttackSystem.md

## 1. 문서 목적

이 문서는 현재 플레이어 기본 공격의 실제 구현 기준을 정리한다.

현재 기본 공격은 좌클릭으로 실행하는 1/2/3타 카타나 근접 콤보다. 이 문서는 공격 입력, 콤보 진행, 공격 방향, 공격 판정, 데미지 적용, 확정 히트 타이밍의 기준을 다룬다.

검 궤적, 피격 VFX, 스파크, 라이트닝 이펙트의 세부 연출은 `Docs/VFXSystem.md`와 `Docs/AnimationSystem.md`를 따른다. 보상으로 공격이 성장하는 방향은 `Docs/RewardSystem.md`와 `Docs/ElementalRewardSystem.md`를 따른다.

## 2. 현재 방향

- 기본 공격은 자동 공격이 아니라 플레이어가 좌클릭으로 직접 사용하는 수동 공격이다.
- 공격은 1타, 2타, 3타 콤보로 진행된다.
- 공격 방향은 마우스 커서 방향을 우선 사용한다.
- 공격 판정은 검 메시 충돌이 아니라 플레이어 전방 부채꼴 범위 판정이다.
- 데미지는 `HealthComponent::ApplyDamage`로 적용한다.
- 공격 데미지 타이밍은 `AnimNotify_PlayerMeleeHit`이 우선이며, fallback delay가 남아 있다.
- 방 단위 전투, Stage 진행, 보상 구조가 바뀌어도 현재 좌클릭 콤보는 기본 전투의 중심으로 유지한다.

## 3. 현재 구현 파일

| 구분 | 파일 | 역할 |
| --- | --- | --- |
| 기본 공격 소유자 | `Source/MyProject/Combat/PlayerMeleeAttackSubsystem.h/.cpp` | 좌클릭 입력, 콤보, 공격 방향, 판정, 데미지, 확정 히트 후 임시 VFX, Chain Lightning MVP |
| 공격 스탯 | `Source/MyProject/Combat/PlayerStatsComponent.h/.cpp` | 공격력, 범위, 각도, 쿨타임, 콤보 값, 디버그 플래그 제공 |
| 공격 기본값 | `Source/MyProject/Combat/PlayerStatsDataAsset.h` | 에디터에서 조정할 플레이어 공격 기준값 |
| 체력/데미지 적용 | `Source/MyProject/Combat/HealthComponent.h/.cpp` | 실제 데미지 적용과 사망 이벤트 |
| 적 대상 | `Source/MyProject/Enemies/EnemyBase.h/.cpp` | 기본 공격의 현재 대상 타입 |
| 검 궤적 | `Source/MyProject/Combat/WeaponTrailComponent.h/.cpp` | 콤보별 Trail 슬롯, AnimNotify 기반 Trail 재생, 라이트닝 검 오라 |
| 검 궤적 Notify | `Source/MyProject/Combat/AnimNotifyState_WeaponTrail.h/.cpp` | 공격 애니메이션의 검 궤적 시작/종료 구간 |
| 히트 타이밍 Notify | `Source/MyProject/Combat/AnimNotify_PlayerMeleeHit.h/.cpp` | 애니메이션의 실제 타격 프레임에서 queued hit 실행 |
| 지면 스파크 Notify | `Source/MyProject/Combat/AnimNotify_GroundImpactVfx.h/.cpp` | 검이 지면에 닿는 연출용 VFX 트리거 |

## 4. 현재 플레이 흐름

1. 플레이어가 좌클릭한다.
2. `PlayerMeleeAttackSubsystem`이 현재 플레이어 Pawn과 `PlayerStatsComponent`를 확인한다.
3. 사망 상태이거나 공격이 불가능한 상태면 중단한다.
4. 마우스 커서 기준으로 공격 방향을 계산한다.
5. 플레이어를 공격 방향으로 회전시킨다.
6. 현재 콤보 단계를 1/2/3 중 하나로 진행한다.
7. 콤보 단계에 맞는 전진 이동 값을 적용한다.
8. `WeaponTrailComponent::PlayAttackTrail(ComboStep)`을 호출한다.
9. 콤보 애니메이션을 재생한다.
10. 공격 판정 데이터를 `PendingMeleeHit`으로 저장한다.
11. `AnimNotify_PlayerMeleeHit` 또는 fallback delay가 실제 히트 실행을 호출한다.
12. 부채꼴 범위 안의 살아 있는 `EnemyBase`를 찾는다.
13. 각 적의 `HealthComponent::ApplyDamage`를 호출한다.
14. 실제 적용 데미지가 0보다 크면 확정 히트로 취급한다.
15. 확정 히트 후 현재 임시 VFX와 Chain Lightning MVP가 이어진다.

## 5. 콤보 규칙

현재 콤보는 최대 3타다.

| 항목 | 현재 기준 |
| --- | --- |
| 입력 | 좌클릭 |
| 최대 단계 | 3타 |
| 콤보 입력 방식 | 공격 쿨타임 중 좌클릭하면 다음 콤보 입력으로 버퍼링 |
| 콤보 리셋 | `ComboResetWindow` 이후 초기화 |
| 콤보 전환 | 애니메이션 길이와 `ComboTransitionRatio` 기준 |
| 마지막 타 이후 | 다음 공격은 다시 1타부터 시작 |

현재 공격 애니메이션:

| 콤보 | 애니메이션 |
| --- | --- |
| 1타 | `Katana_Blade_Attack_3Combo_1_Inplace` |
| 2타 | `Katana_Blade_Attack_3Combo_2_Inplace` |
| 3타 | `Katana_Blade_Attack_3Combo_3_Inplace` |

애니메이션 경로:

```text
/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/2_Attacks/0__3Combos
```

## 6. 공격 판정 기준

현재 기본 공격은 실제 검 메시 충돌이 아니라 부채꼴 범위 판정이다.

| 항목 | 현재 기준 |
| --- | --- |
| 판정 모양 | 전방 부채꼴 |
| 기준 위치 | 플레이어 위치 |
| 기준 방향 | 마우스 커서 방향, 실패 시 캐릭터 전방 |
| 대상 | 살아 있는 `EnemyBase` |
| 거리 | `BasicAttackRange` |
| 각도 | `BasicAttackAngleDegrees` |
| 데미지 | `BasicAttackDamage` |
| 데미지 적용 | `HealthComponent::ApplyDamage` |
| 다중 타격 | 범위 안의 여러 적에게 적용 가능 |
| Debug 표시 | `bShowBasicAttackDebug` 기준 |

주의:

- 현재 판정은 카타나 궤적의 실제 모양과 1:1로 일치하지 않는다.
- 검 궤적 VFX는 판정 기준이 아니라 시각 연출이다.
- 적이 사망했거나 `HealthComponent`가 없으면 데미지를 적용하지 않는다.
- `ApplyDamage` 반환값이 0보다 클 때만 확정 히트로 본다.

## 7. 히트 타이밍 기준

현재 구조는 공격 시작과 실제 데미지 적용 시점을 분리한다.

우선 기준:

- 공격 애니메이션에 `AnimNotify_PlayerMeleeHit`을 배치한다.
- Notify가 실행되는 프레임에서 `ExecutePendingMeleeHitForPawn`이 호출된다.
- 이 방식이 데미지, 피, 히트 디스토션을 검이 실제로 닿는 프레임에 맞추는 정식 방향이다.

Fallback 기준:

- Notify가 없거나 실행되지 않아도 공격이 완전히 죽지 않도록 fallback delay가 남아 있다.
- fallback delay는 `ComboHitDelayRatios`, `MinComboHitDelay`, `MaxComboHitDelay`를 기준으로 계산된다.
- fallback은 안전장치이며 최종 타이밍 튜닝 기준은 아니다.

## 8. 입력과 이동 잠금 기준

현재 공격 중에는 이동 입력을 잠그고, 공격 전진 이동은 공격 시스템이 직접 관리한다.

| 상황 | 현재 동작 |
| --- | --- |
| 공격 중 WASD | 이동 잠금 |
| 공격 중 좌클릭 | 다음 콤보 입력으로 버퍼링 가능 |
| 공격 중 Space 회피 | 공격 취소 후 회피 우선 |
| 사망 상태 | 공격 불가 |

주의:

- 회피 캔슬은 `PlayerKeyboardMovementSubsystem`과 함께 봐야 한다.
- 공격 전진 이동은 적을 밀어내기 위한 물리 충돌이 아니라 공격 모션의 전진감을 위한 값이다.
- 공격 입력/이동 규칙을 바꾸면 `Docs/PlayerControlSystem.md`도 함께 확인한다.

## 9. PlayerStats 기준값

기본 공격 관련 값은 `PlayerStatsComponent`와 `PlayerStatsDataAsset` 기준을 따른다.

| 값 | 의미 |
| --- | --- |
| `BasicAttackDamage` | 기본 공격 데미지 |
| `BasicAttackRange` | 부채꼴 공격 거리 |
| `BasicAttackAngleDegrees` | 부채꼴 공격 각도 |
| `BasicAttackCooldown` | 공격 입력 최소 쿨타임 |
| `ComboResetWindow` | 콤보 리셋 시간 |
| `MaxComboStep` | 최대 콤보 단계, 현재 1~3으로 제한 |
| `Combo1ForwardStepDistance` | 1타 전진 거리 |
| `Combo2ForwardStepDistance` | 2타 전진 거리 |
| `Combo3ForwardStepDistance` | 3타 전진 거리 |
| `ComboForwardStepDuration` | 공격 전진 이동 시간 |
| `ComboTransitionRatio` | 다음 콤보로 넘어갈 수 있는 타이밍 비율 |
| `BasicAttackAnimationSlotName` | 공격 애니메이션 재생 슬롯 |
| `bShowComboDebug` | 콤보 디버그 표시 여부 |
| `bShowBasicAttackDebug` | 공격 범위 디버그 표시 여부 |
| `bShowChainLightningDebug` | Chain Lightning 의도 경로 초록선 표시 여부 |

Run 보상으로 `BasicAttackDamage` 계열 값이 바뀔 수 있지만, 원본 `PlayerStatsDataAsset`을 런타임에 직접 수정하지 않는다.

## 10. 확정 히트 후 연결되는 현재 기능

`PlayerMeleeAttackSubsystem`은 현재 확정 히트 후 일부 임시 연출과 라이트닝 MVP를 직접 호출한다.

현재 연결:

| 기능 | 현재 상태 | 기준 |
| --- | --- | --- |
| 피 VFX | 구현됨 | 적용 데미지 0 초과 시 무작위 Blood Niagara 재생 |
| Hit Distortion VFX | 구현됨 | 적용 데미지 0 초과 시 `NS_Hit_Distortion` 재생 |
| Chain Lightning MVP | 구현됨 | Lightning 상태, 확정 히트, 주변 추가 대상 1명, 임시 chain VFX |
| Sword Trail | 구현됨 | `WeaponTrailComponent`와 `AnimNotifyState_WeaponTrail` 기준 |
| Ground Spark | 구현됨 | `AnimNotify_GroundImpactVfx` 기준 |

주의:

- 피 VFX와 Hit Distortion은 현재 BasicAttack 코드 안에 있지만 장기적으로는 전용 `WeaponImpactVfxComponent` 후보로 옮기는 방향이다.
- Chain Lightning은 현재 데미지/로그와 임시 Source -> Target arc VFX를 가진 MVP다.
- Ground Spark는 기본 공격 판정이 아니라 애니메이션/VFX 연출이다.

## 11. Chain Lightning MVP 기준

현재 라이트닝 상태에서 기본 공격이 실제 적중하면 Chain Lightning MVP가 실행된다.

현재 규칙:

- `PlayerStatsComponent::IsLightningElementActive()`가 true일 때만 발동한다.
- 플레이어 공격이 실제로 적에게 데미지를 적용한 뒤에만 발동한다.
- 원래 맞은 적은 체인 대상에서 제외한다.
- 직접 맞은 적이 해당 타격으로 사망해도, 확정 히트 순간의 저장된 위치를 시작점으로 Chain Lightning을 시작할 수 있다.
- 한 번의 공격 판정당 체인 시퀀스는 1회만 발생한다.
- 현재는 최대 2개의 전이 구간을 만든다.
- 각 전이는 0.03~0.06초의 짧은 지연 후 순차적으로 적용된다.
- 대상은 살아 있고 활성화된 `EnemyBase`여야 한다.
- 체인 반경은 현재 코드 기준 650이다.
- 체인 데미지는 기본 공격 데미지의 35%이며 최소 1이다.
- Lightning 직접 타격은 대상 적에게 `ElectricStack +2`를 부여한다.
- Chain Lightning 전이 피해는 대상 적에게 `ElectricStack +1`을 부여한다.
- `ElectricStack`이 3 이상이면 `EnemyBase`의 Shock 상태가 발동한다.
- Chain Lightning Beam은 월드 독립 VFX로 생성되고, 전이 대상 피격 전기 VFX는 대상 Mesh 또는 Root에 짧게 attached된다.
- Chain Lightning 초록 디버그 선은 `bShowChainLightningDebug`가 켜져 있을 때만 표시한다.
- 최종 Chain Lightning VFX 튜닝은 아직 완료하지 않았고, 현재는 임시 Beam/target hit VFX를 사용한다.

장기 확장은 `Docs/ElementalRewardSystem.md` 기준을 따른다.

## 12. 검 궤적과 공격 VFX 경계

BasicAttackSystem은 공격 판정과 데미지 타이밍의 기준 문서다.

검 궤적 기준:

- `WeaponTrailComponent`가 콤보별 Trail 슬롯을 가진다.
- `AnimNotifyState_WeaponTrail`이 실제 검을 휘두르는 구간을 열고 닫는다.
- `WeaponTrailComponent`의 fallback delay/duration은 Notify 타이밍을 쓰지 않을 때의 안전장치다.

공격 VFX 기준:

- 피, Hit Distortion, 스파크, 라이트닝 오라, Chain Lightning arc는 VFX 문서 기준을 따른다.
- VFX는 공격이 맞았는지 결정하지 않는다.
- VFX는 데미지, 사망, 경직, 보상 효과를 직접 결정하지 않는다.

## 13. 보상과 성장 경계

기본 공격은 보상의 주요 성장 축이다.

현재 방향:

- 공격력 증가, 공격 범위 증가, 3타 강화, 검기, 출혈, 흡혈, 라이트닝 체인 같은 보상은 기본 공격에 붙을 수 있다.
- 그러나 보상 데이터, 등급, 등장 확률, Prism 조건은 BasicAttackSystem이 아니라 `RewardSystem.md`와 `ElementalRewardSystem.md`가 담당한다.
- BasicAttackSystem은 보상이 적용될 때 사용할 안정적인 연결점만 제공한다.

현재 유지할 연결점:

- `BasicAttackDamage`
- `BasicAttackRange`
- `ComboStep`
- 확정 히트 결과
- 직접 맞은 적 목록
- Lightning 상태 여부

## 14. 현재 사용하지 않는 기준

- 무한 생존 자동 공격 구조는 현재 기본 공격 기준이 아니다.
- 기존 검기/Slash VFX 직접 생성 경로는 폐기되었고, 현재는 `WeaponTrailComponent`와 전용 VFX 경로를 사용한다.
- 기본 공격에서 적 흰색 플래시를 직접 호출하지 않는다.
- 기본 공격 문서에서 전체 스킬 시스템, 무기 장비 시스템, 보상 풀 테이블을 정의하지 않는다.

## 15. 다음 후보

아직 구현 또는 정리가 남은 후보:

- `PlayerMeleeAttackSubsystem`의 피격 VFX 호출을 미래 `WeaponImpactVfxComponent`로 분리
- Chain Lightning VFX arc 최종 튜닝
- 3타 강화 보상 1개를 실제 기능으로 검증
- 적 HitReact와 공격 강도 연결
- 검 궤적/Hit VFX의 최종 Niagara 튜닝
- 무기 교체가 필요해지는 시점의 `WeaponDataAsset` 후보 설계

## 16. 관련 문서

- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/HealthSystem.md`
- `Docs/PlayerControlSystem.md`
- `Docs/VFXSystem.md`
- `Docs/AnimationSystem.md`
- `Docs/RewardSystem.md`
- `Docs/ElementalRewardSystem.md`
- `Docs/AttackSkillSystem.md`
## Chain Lightning 현재 적용 기준

- 현재 Chain Lightning MVP는 Lightning 상태에서 확정 근접 히트가 발생했을 때만 실행된다.
- 원래 직접 맞은 적은 체인 대상으로 제외한다.
- 한 번의 공격 판정에 체인 시퀀스는 1회만 발생한다.
- 현재 테스트 기준으로 최대 2번 전이된다.
- 각 전이는 0.03~0.06초 짧은 지연 후 순차적으로 실행된다.
- 대상은 살아 있고 활성화된 `EnemyBase`여야 한다.
- 체인 반경은 현재 코드 기준 650이다.
- 체인 데미지는 기본 공격 데미지의 35%이며 최소 1이다.
- Chain Lightning VFX는 `/Game/EnergyBeam/NS/NS_ChainLightning_Controlled`를 임시 arc VFX로 사용한다.
