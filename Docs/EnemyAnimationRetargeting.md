---
Status: ACTIVE
Scope: Enemy Animation
Last Updated: 2026-07-11
Source of Truth: Yes
---

# EnemyAnimationRetargeting.md

## 1. 문서 목적

이 문서는 현재 적 캐릭터의 메시, 스켈레톤, AnimBP, Montage, 리타겟 기준을 정리한다.

적 애니메이션의 공통 규칙은 `Docs/AnimationSystem.md`를 우선하고, 이 문서는 Little Demon, Fast, Tank처럼 적 개별 에셋 연결과 리타겟 기준을 다룬다.

이전 날짜별 기록과 과거 실험 내용은 아래 Archive에 보존한다.

- `Docs/Archive/EnemyAnimationRetargetingHistory_PreDocRestructure_2026-07-06.md`

## 2. 현재 기준

- 적의 공통 gameplay 로직은 `EnemyBase`가 담당한다.
- 적의 메시, 스켈레톤, AnimBP, Montage, 애니메이션 에셋은 적 Blueprint와 에셋 설정이 담당한다.
- C++ `EnemyBase`에 특정 적의 메시, 스켈레톤, AnimBP 경로를 하드코딩하지 않는다.
- 가능한 경우 각 몬스터의 원본 스켈레톤과 원본 애니메이션을 우선 사용한다.
- 다른 스켈레톤의 애니메이션을 공유해야 할 때만 IK Rig / IK Retargeter를 사용한다.
- 적 애니메이션 변경은 반드시 Unreal Editor에서 시각 확인이 필요하다.
- 공격 애니메이션이 재생된다고 해서 공격 판정 타이밍까지 애니메이션 기반이라는 뜻은 아니다. 현재 적 공격 데미지는 주로 `EnemyBase` 거리/쿨타임 로직 기준이다.

현재 적별 큰 방향:

| 적 | 현재 기준 |
|---|---|
| Little Demon | 안정성을 우선해 SingleNode / `EnemySimpleAnimationComponent` 경로 유지 |
| Fast / DemonHeavy | DemonHeavy 원본 스켈레톤과 dedicated AnimBP/Montage 경로 사용 |
| Tank / Butcher | Butcher 스켈레톤과 dedicated AnimBP/Montage 경로 사용 |
| Ranged Dummy / SkeletonMage | SkeletonMage 원본 Mesh와 `ThirdPerson_AnimBP` locomotion 사용. 전용 cast animation은 아직 없음 |

## 3. 현재 구현 상태

### Little Demon

현재 Little Demon은 안정 MVP 경로를 사용한다.

| 항목 | 현재 기준 |
|---|---|
| Blueprint | `BP_Enemy_LittleDemon` |
| 공통 기반 | `EnemyBase` |
| 애니메이션 방식 | `AnimationSingleNode` + `EnemySimpleAnimationComponent` |
| 공격 애니메이션 | `Anim_Little_Demon_attack1` |
| 스폰 인트로 | `Anim_Little_Demon_rage` |
| 인트로 후 대기 | `Anim_Little_Demon_idle2`, 약 1초 |
| 추격 전 회전 | 플레이어 방향으로 부드럽게 회전 |
| 현재 공격 기준 | attack1 단일 공격 |

현재 기준:

- Little Demon은 현재 단일 attack1 애니메이션을 사용한다.
- attack1~6 랜덤 재생은 현재 기준이 아니며, 관련 과거 기록은 Archive를 참고한다.
- `ABP_LittleDemon`은 전환 후보였지만 현재 `BP_Enemy_LittleDemon`의 안정 런타임 기준은 아니다.
- 스폰 인트로 중에는 추격과 공격을 잠근다.
- 스폰 인트로 높이 문제는 Blueprint의 `SpawnIntroMeshRelativeOffset` 같은 적별 설정으로 조정한다.

### Fast / DemonHeavy

현재 Fast는 DemonHeavy 에셋을 사용하는 빠른 압박형 적이다.

| 항목 | 현재 기준 |
|---|---|
| Blueprint | `BP_Enemy_Fast` |
| 메시 | `/Game/Demons_Big_Pack/DemonHeavy/Mesh/SK_DemonHeavy` |
| 애니메이션 방식 | dedicated AnimBP + Montage |
| AnimBP | `ABP_DemonHeavy_Fast` |
| 기본 이동 | DemonHeavy idle/run 계열 |
| 약공격 Montage | `AM_DemonHeavy_Fast_Attack1` |
| 강공격 Montage | `AM_DemonHeavy_Fast_Strong_Attack3` |
| 강공격 조건 | `StrongAttackEveryNthAttack = 3` |
| 강공격 성격 | 세 번째 성공 공격에서 더 강한 공격 후보 |

현재 기준:

- Fast는 DemonHeavy 원본 스켈레톤과 애니메이션을 사용한다.
- 현재 setup에서는 별도 리타겟이 필요하지 않다.
- Fast의 idle/run/attack 전환 품질은 `ABP_DemonHeavy_Fast`와 Montage Blend 값에서 조정한다.
- Fast 공격이 어색하면 먼저 AnimBP/Montage/Blend 설정을 확인하고, 공통 `EnemyBase` 로직을 먼저 바꾸지 않는다.

### Tank / Butcher

현재 Tank는 Butcher 에셋을 사용하는 느린 고체력 적이다.

| 항목 | 현재 기준 |
|---|---|
| Blueprint | `BP_Enemy_Tank` |
| 메시 | modified Butcher mesh using Butcher skeleton |
| 애니메이션 방식 | dedicated AnimBP + Montage |
| AnimBP | `/Game/Blueprints/Enemy/Tank/ABP_Butcher` |
| AnimBP 대상 | Butcher skeleton |
| 기본 이동 | `Anim_Butcher_idle1`, `Anim_Butcher_walk2` 기준 |
| 약공격 Montage | `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Weak_Attack1` |
| 강공격 Montage | `/Game/Blueprints/Enemy/Tank/AM_Butcher_Tank_Strong_Attack3` |
| 강공격 조건 | `StrongAttackEveryNthAttack = 2` |
| 공격 재생 속도 | 느리고 무거운 공격감을 우선 |
| 공격 후 회전 | 플레이어 방향으로 부드럽게 보정 |

현재 기준:

- Tank는 SingleNode fallback보다 dedicated AnimBP/Montage 경로를 우선한다.
- Butcher의 `run1` 계열은 과거 루프 끊김이 있었으므로 현재 이동은 검증된 walk 계열을 우선한다.
- Tank 공격 전환이 어색하면 `ABP_Butcher`, Montage BlendIn/BlendOut, 공격 lock duration, post-attack turn 값을 먼저 확인한다.
- 여러 Butcher 공격을 추가할 때는 무작위 재생보다 약공격/강공격 또는 Montage section 기반으로 검증한다.

### Ranged Dummy / SkeletonMage

현재 원거리 테스트 적은 SkeletonMage 에셋을 presentation으로 사용한다.

| 항목 | 현재 기준 |
|---|---|
| Blueprint | `/Game/Blueprints/Enemy/Ranged/BP_Enemy_RangedDummy` |
| 메시 | `/Game/SkeletonMage/Mesh/SK_SkeletonMage` |
| AnimBP | `/Game/SkeletonMage/Animations/ThirdPerson_AnimBP` |
| 지팡이 | `/Game/SkeletonMage/Mesh/SK_MagicStaff`, `hand_r` 연결 |
| 현재 애니메이션 범위 | idle / locomotion |
| 공격 presentation | 전용 캐스팅 애니메이션과 예고 VFX는 아직 미구현 |

PIE에서 메시 방향과 지면 정렬, 사거리 접근 후 정지/조준, 투사체 발사를 확인했다. 원거리 gameplay 수치는 `EnemyRangedDummy`가 담당하고 SkeletonMage 에셋 경로는 Blueprint에만 둔다.

### 공통 적 애니메이션 클래스

| 클래스 / 컴포넌트 | 역할 |
|---|---|
| `EnemyBase` | 공격 재생 요청, 스폰 인트로, 사망, 공통 상태 제어 |
| `EnemySimpleAnimationComponent` | 전용 AnimBP가 없는 적의 idle/run fallback |
| `EnemyAnimInstance` | 전용 AnimBP에서 GroundSpeed, 이동/공격 상태 같은 공통 변수 제공 |

## 4. 아직 미구현

- Little Demon 전용 AnimBP를 안정 런타임 기준으로 전환하는 작업.
- Little Demon attack1~6을 자연스럽게 섞는 검증된 Montage/section 구조.
- 적 공격 데미지 타이밍을 AnimNotify 기반으로 전환하는 구조.
- 적별 피격 반응 AnimBP/Montage 연결.
- 적별 사망 애니메이션 최종 품질 정리.
- MiniBoss/Boss 전용 애니메이션 구조.
- 원거리 적의 전용 캐스팅/예고 애니메이션 구조와 장판 적, 특수 적 전용 애니메이션 구조.
- 여러 몬스터 체형을 위한 공통 리타겟 파이프라인.

## 5. 수정 시 주의사항

- 특정 적의 메시/스켈레톤/AnimBP 경로를 C++에 직접 고정하지 않는다.
- Little Demon의 랜덤 공격 재생 실험 기록은 Archive에만 보존하고, 현재 기준으로 사용하지 않는다.
- Fast/Tank처럼 전용 AnimBP가 있는 적은 AnimBP/Montage/Blend 설정을 먼저 조정한다.
- 공격 모션이 자연스러운지, 추격 전환이 어색하지 않은지, 사망 후 포즈가 깨지지 않는지는 에디터 PIE에서 확인한다.
- 애니메이션 시퀀스 단독 미리보기에서 정상이어도 PIE에서는 AnimBP, Montage, root motion, slot, blend, collision, culling 영향으로 다르게 보일 수 있다.
- 리타겟 작업은 결과를 직접 눈으로 확인하기 전까지 현재 기준으로 확정하지 않는다.
- 현재 적 공격 데미지 타이밍은 애니메이션 Notify 기준이 아니므로, 공격 모션 수정과 데미지 판정 수정을 혼동하지 않는다.
- 과거 실험 로그는 ACTIVE 문서에 계속 쌓지 말고 Archive에 보존한다.

## 6. 관련 문서

- `Docs/AnimationSystem.md`: 공통 애니메이션 소유권, AnimNotify, Montage, HitReact 방향.
- `Docs/EnemySystem.md`: 적 gameplay와 현재 적 타입 기준.
- `Docs/EnemyArchitecture.md`: 적 구조와 책임 경계.
- `Docs/EnemySpawnSystem.md`: 적 스폰, 배치형 적, Encounter 기준.
- `Docs/VFXSystem.md`: 애니메이션 타이밍 기반 VFX 기준.
- `Docs/HealthSystem.md`: 데미지, 사망, HitReact 경계.
