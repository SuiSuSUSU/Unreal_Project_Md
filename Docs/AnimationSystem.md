---
Status: ACTIVE
Scope: Animation System
Last Updated: 2026-07-06
Source of Truth: Yes
---

# AnimationSystem.md

## 1. 문서 목적

이 문서는 플레이어와 적 애니메이션의 현재 공통 기준을 정리한다.

AnimBP, Montage, BlendSpace, Slot, AnimNotify, 공격 타이밍, 피격 반응, 스폰 인트로, 애니메이션 기반 VFX를 수정하기 전에 이 문서를 확인한다.

이전 날짜별 기록과 과거 실험 내용은 아래 Archive에 보존한다.

- `Docs/Archive/AnimationSystemHistory_PreDocRestructure_2026-07-06.md`

## 2. 현재 기준

- 애니메이션 품질은 에셋별로 다르며 Unreal Editor에서 직접 확인해야 한다.
- C++는 애니메이션 재생 요청과 gameplay 상태를 전달할 수 있지만, 자연스러운 전환은 주로 AnimBP, Montage, BlendSpace, Slot, Notify 설정이 담당한다.
- 모든 몬스터를 하나의 스켈레톤이나 하나의 AnimBP 구조로 강제 통일하지 않는다.
- 플레이어 공격 타이밍, 검 궤적, 지면 스파크처럼 프레임 정확도가 필요한 연출은 AnimNotify 또는 AnimNotifyState를 우선한다.
- 적 공격 데미지 타이밍은 현재 주로 `EnemyBase`의 거리/쿨타임 로직 기준이며, 아직 적 공격 Notify 기반이 아니다.
- 과거 실험 로그는 ACTIVE 문서에 누적하지 않고 Archive로 보존한다.

현재 큰 방향:

| 영역 | 현재 기준 |
|---|---|
| 플레이어 기본 공격 | 1/2/3타 콤보, Montage/Slot 재생, `AnimNotify_PlayerMeleeHit` 우선 |
| 플레이어 검 궤적 | `AnimNotifyState_WeaponTrail`이 정확한 trail window 담당 |
| 플레이어 지면 스파크 | Ground Impact / Contact / Scrape Notify 계열 담당 |
| 플레이어 피격 반응 | `PlayerHealthSubsystem`의 Light/Heavy MVP |
| Little Demon | SingleNode / `EnemySimpleAnimationComponent` 안정 경로 |
| Fast | `ABP_DemonHeavy_Fast` + weak/strong Montage |
| Tank | `ABP_Butcher` + weak/strong Montage |
| 적 HitReact | 아직 정식 구현 전, 설계 후보 |

## 3. 현재 구현 상태

### 플레이어 기본 공격 애니메이션

현재 플레이어 기본 공격은 1/2/3타 카타나 콤보다.

| 항목 | 현재 기준 |
|---|---|
| 소유자 | `PlayerMeleeAttackSubsystem` |
| 슬롯 | `DefaultSlot` 또는 `PlayerStatsDataAsset.BasicAttackAnimationSlotName` |
| 공격 애니메이션 | `Katana_Blade_Attack_3Combo_1_Inplace`, `2`, `3` |
| 히트 타이밍 | `AnimNotify_PlayerMeleeHit` 우선 |
| Fallback | Notify가 없을 때 combo hit delay |
| 이동 잠금 | 공격 애니메이션 길이 기준 |

현재 공격 흐름:

```text
좌클릭
-> 콤보 단계 결정
-> 공격 애니메이션 재생
-> PendingMeleeHit 저장
-> AnimNotify_PlayerMeleeHit 또는 fallback delay
-> DamageEnemiesInAttackCone
-> 확정 히트 VFX / Chain Lightning MVP
```

`AnimNotify_PlayerMeleeHit`은 검이 실제로 닿는 프레임에 데미지, 피, Hit Distortion이 함께 실행되도록 맞추는 현재 우선 경로다.

### 플레이어 피격 반응

플레이어 비사망 피격 반응은 작은 MVP로 구현되어 있다.

| 항목 | 현재 기준 |
|---|---|
| 소유자 | `PlayerHealthSubsystem` |
| 타입 | `EPlayerHitReactType::None`, `Light`, `Heavy` |
| Light 요청 | 적 약공격 |
| Heavy 요청 | 적 강공격 |
| 재생 방식 | `PlaySlotAnimationAsDynamicMontage` |
| 슬롯 | `DefaultSlot` |
| 현재 성격 | presentation-only |

현재 피격 반응은 이동 잠금, 공격 취소, 무적, 데미지 규칙 변경을 담당하지 않는다.

### 적 애니메이션

| 적 | 현재 기준 |
|---|---|
| Little Demon | SingleNode / `EnemySimpleAnimationComponent`, attack1 단일 공격, rage 스폰 인트로 |
| Fast / DemonHeavy | dedicated `ABP_DemonHeavy_Fast`, 약/강 Montage, 세 번째 성공 공격에서 강공격 |
| Tank / Butcher | dedicated `ABP_Butcher`, 약/강 Montage, 두 번째 성공 공격에서 강공격 |

공통 기준:

- `EnemyBase`는 공격 재생 요청, 스폰 인트로, 사망, 공통 상태 제어를 담당한다.
- 메시, 스켈레톤, AnimBP, Montage, 애니메이션 에셋은 적 Blueprint/Asset에서 관리한다.
- 랜덤 공격 시퀀스 교체는 현재 기본 방식이 아니다.
- 여러 공격을 쓰려면 검증된 Montage 또는 weak/strong 구조를 우선한다.

### AnimNotify / NotifyState

현재 구현된 Notify 계열:

| Notify | 역할 |
|---|---|
| `AnimNotify_PlayerMeleeHit` | 플레이어 기본 공격의 queued hit 실행 |
| `AnimNotifyState_WeaponTrail` | 플레이어 검 궤적 시작/종료 window |
| `AnimNotify_GroundImpactVfx` | 한 프레임 지면 충돌 스파크 |
| `AnimNotifyState_GroundContactImpactVfx` | 짧은 contact window 동안 바닥 접촉 감지 후 스파크 |
| `AnimNotifyState_GroundScrapeVfx` | 검이 바닥을 긁는 동안 반복 스파크 |

Notify 기준:

- 정확한 타이밍이 필요한 연출은 고정 delay보다 Notify를 우선한다.
- Ground spark 계열은 `WeaponVfx_GroundContact` SceneComponent를 trace point로 사용한다.
- 지면 스파크 Niagara는 월드 기준으로 재생되고 자연 소멸해야 한다.
- Static Mesh 에셋에 자동으로 socket을 추가하는 방식은 사용하지 않는다.

### HitReact 설계 후보

적 HitReact는 아직 정식 구현되지 않았다.

현재 방향:

```text
HitReactScore = DamageAmount + AttackTypeBonus + ComboStepBonus + RewardBonus

if HitReactScore >= EnemyHitReactResistance:
    request allowed reaction
else:
    no reaction
```

HitReact는 데미지 적용, 공격 판정, 사망 처리, VFX 재생을 한 곳에 섞지 않고 별도 요청/판정/연출로 나누는 방향이다.

## 4. 아직 미구현

- 적 공격 데미지 타이밍을 AnimNotify 기반으로 전환.
- Little Demon 전용 AnimBP의 안정 런타임 전환.
- 적별 최종 피격 반응 시스템.
- 적별 사망 애니메이션 최종 품질 정리.
- Boss / MiniBoss 전용 애니메이션 상태 구조.
- 모든 적에 대한 최종 BlendSpace 기반 locomotion 정리.
- 플레이어 전체 locomotion overhaul.
- 플레이어 공격 cancel, stagger, invulnerability 같은 gameplay 피격 반응 확장.
- Chain Lightning arc 같은 속성 VFX와 애니메이션 타이밍 연동.

## 5. 수정 시 주의사항

- 애니메이션 파일 단독 미리보기와 PIE 결과가 다를 수 있다.
- 깜빡임, 툭 끊김, 회전 튐은 코드보다 AnimBP/Montage/Slot/Blend/Root Motion/Bounds 문제일 수 있다.
- Little Demon의 랜덤 공격 재생 실험 기록은 Archive에만 보존하고, 현재 기준으로 사용하지 않는다.
- 적 메시, 스켈레톤, AnimBP 경로를 C++에 하드코딩하지 않는다.
- Notify는 타이밍을 담당하고, 데미지/상태/VFX 책임을 무리하게 모두 소유하지 않는다.
- 지면 스파크는 캐릭터나 무기에 계속 붙어 있는 VFX가 아니라 월드 기준 이펙트로 처리한다.
- `WeaponVfx_GroundContact`가 없으면 Ground spark 위치가 부정확해질 수 있으므로 Blueprint setup을 먼저 확인한다.
- 적 공격 애니메이션을 바꿨다고 해서 데미지 판정 타이밍이 바뀐 것으로 보지 않는다.
- Archive 문서는 현재 구현 기준으로 사용하지 않는다.

## 6. 관련 문서

- `Docs/EnemyAnimationRetargeting.md`: 적별 메시, 스켈레톤, AnimBP, Montage 기준.
- `Docs/EnemySystem.md`: 적 gameplay 행동과 공격 기준.
- `Docs/EnemyArchitecture.md`: 적 구조와 책임 경계.
- `Docs/BasicAttackSystem.md`: 플레이어 기본 공격과 히트 타이밍 기준.
- `Docs/VFXSystem.md`: 검 궤적, 피격 VFX, 지면 스파크, 라이트닝 VFX 기준.
- `Docs/HealthSystem.md`: 체력, 데미지, 사망, HitReact 경계.
- `Docs/PlayerControlSystem.md`: 이동, 회피, 공격 중 입력 기준.
