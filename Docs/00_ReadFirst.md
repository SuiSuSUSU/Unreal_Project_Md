---
Status: ACTIVE
Scope: Document Router
Last Updated: 2026-07-06
Source of Truth: Yes
---

# Docs/00_ReadFirst.md

## 1. 문서 목적

이 문서는 Codex가 작업 전에 어떤 문서를 읽어야 하는지 안내하는 문서 라우팅 기준이다.

문서가 많아졌을 때 오래된 후보 문서, 참고 문서, 현재 구현 기준 문서를 혼동하지 않기 위해 사용한다.

## 2. 기본 읽기 순서

모든 작업 전 아래 순서로 문서를 확인한다.

1. `Docs/00_ReadFirst.md`: 이번 작업에 필요한 문서 경로를 정한다.
2. 공통 기준 문서: 현재 구현 상태, 클래스 책임, 작업 절차를 확인한다.
3. 작업 도메인 문서: 이번 작업과 직접 관련된 문서만 추가로 읽는다.
4. 필요 시 보조 도메인 문서: VFX, Animation, Reward처럼 연결된 영역만 추가 확인한다.

Archive 문서는 기본 읽기 대상이 아니다. `Docs/Archive/` 안의 문서는 사용자가 명시적으로 요청하거나 과거 히스토리 확인이 필요한 경우에만 참고한다.

## 3. 공통으로 먼저 확인할 문서

모든 작업 전 아래 문서를 먼저 확인한다.

- `Docs/05_DecisionLog.md`: 확정된 결정과 보류된 결정을 확인한다.
- `Docs/CurrentImplementation.md`: 현재 실제 구현 상태와 MVP 예외를 확인한다.
- `Docs/10_ProjectClassMap.md`: C++ 클래스, Blueprint, 주요 책임 경계를 확인한다.
- `Docs/04_CodexWorkflow.md`: Codex 작업 절차와 주의 기준을 확인한다.

## 4. 핵심 ACTIVE 문서 역할

아래 문서는 현재 구현 기준으로 자주 참조하는 핵심 문서다.

| 문서 | 역할 | 먼저 읽어야 하는 경우 |
|---|---|---|
| `Docs/CurrentImplementation.md` | 현재 실제 구현 스냅샷과 MVP 예외를 확인한다. | 모든 구현/문서 정리 전 공통 확인 |
| `Docs/10_ProjectClassMap.md` | 현재 클래스, Blueprint, 데이터, 책임 경계를 확인한다. | 클래스 책임 이동, 코드 변경, 구조 판단 전 |
| `Docs/RoomCombatSystem.md` | RoomCombatActor, StageFlowManager, Room Clear, Encounter 실행 기준을 확인한다. | Room, Stage 진행, Room Clear, 다음 방, Encounter 실행 작업 |
| `Docs/EnemySystem.md` | EnemyBase, 현재 적 타입, 적 이동/공격/체력/사망/활성화 기준을 확인한다. | 적 gameplay 작업 |
| `Docs/RewardSystem.md` | Stage Clear 보상, enemy drop item, reward apply type, 보상 책임 경계를 확인한다. | 보상 카드, 보상 타이밍, 보상 효과 작업 |
| `Docs/AnimationSystem.md` | 플레이어/적 공통 애니메이션, Montage, AnimNotify, HitReact 방향을 확인한다. | AnimBP, Montage, Notify, 공격/피격 타이밍 작업 |
| `Docs/EnemyAnimationRetargeting.md` | Little Demon, Fast, Tank의 메시/스켈레톤/AnimBP/Montage와 리타겟 기준을 확인한다. | 적 개별 애니메이션, 리타겟, 적 asset 연결 작업 |
| `Docs/VFXSystem.md` | WeaponTrail, hit VFX, ground spark, Lightning VFX, VFX 책임 경계를 확인한다. | 검 궤적, 피격 VFX, 스파크, 라이트닝, 보상 기반 VFX 작업 |

## 5. 도메인별 기준 문서

아래 문서는 현재 게임 구조를 도메인별로 나눈 기준 문서다.

| 도메인 | 우선 기준 문서 | 역할 |
|---|---|---|
| Project Direction | `Docs/00_ProjectOverview.md` | 프로젝트 전체 방향, 핵심 루프, 현재 보류/제외 범위 |
| Stage / Run | `Docs/StageRunStructure.md` | Run, Stage Map, Stage, Room 관계와 Stage Clear 흐름 |
| Room / Encounter | `Docs/RoomCombatSystem.md` | RoomCombatActor, Room clear, EncounterData, placed/spawned enemy 흐름 |
| Reward | `Docs/RewardSystem.md` | Stage Clear 보상, Instant/Run/Permanent, 보상 등급 방향 |
| Elemental Reward | `Docs/ElementalRewardSystem.md` | 속성 보상, 번개 빌드, Prism 해금 조건 |
| Enemy | `Docs/EnemySystem.md`, `Docs/EnemyArchitecture.md` | 적 공통 구조, EnemyBase, 적 타입, 적 책임 경계 |
| Enemy Spawn / Encounter | `Docs/EnemySpawnSystem.md` | RoomEncounterDataAsset, 스폰 큐, 배치형 적, EnemySpawnSubsystem 위치 |
| VFX | `Docs/VFXSystem.md` | WeaponTrail, Niagara, 보상 기반 VFX 성장, hit/death/spawn VFX 방향 |
| Animation | `Docs/AnimationSystem.md`, `Docs/EnemyAnimationRetargeting.md` | AnimBP, Montage, Retargeting, AnimNotify, 적 애니메이션 기준 |
| Player Control | `Docs/PlayerControlSystem.md` | WASD, 회피, 카메라, 인트로, 무기 소켓 |
| Basic Attack | `Docs/BasicAttackSystem.md` | 좌클릭 1/2/3타 콤보, 공격 판정, 기본 공격 기준 |
| Attack / Skill | `Docs/AttackSkillSystem.md` | 공격/스킬/무기/성장 보상 방향 |
| Health / Damage | `Docs/HealthSystem.md` | HealthComponent, 데미지, 회복, 사망, 클리어 이벤트 |
| Player Stats | `Docs/PlayerStatsSystem.md` | PlayerStatsDataAsset, PlayerStatsComponent, 런타임 스탯 |
| Architecture / Refactoring | `Docs/RefactoringCandidates.md` | 책임이 커진 클래스와 나중에 분리할 후보 |
| Development Constraints | `Docs/03_DevelopmentConstraints.md` | 개발 제약, 작업 범위, 1인 개발/AI 협업 기준 참고 |

## 6. 작업별 추가로 읽을 문서

### 프로젝트 방향 / 기획 정리 관련 작업

대상:

- 게임 장르 방향
- 핵심 루프
- MVP 범위
- 보류하거나 제외할 기능
- 큰 구조 변경 전 방향 확인

읽을 문서:

- `Docs/00_ProjectOverview.md`
- `Docs/05_DecisionLog.md`
- 관련 도메인 문서

### 구조 점검 / 리팩터링 관련 작업

대상:

- 클래스 책임 경계 점검
- 비시각적 리팩터링 후보 정리
- 기능 분리 순서 판단
- 기존 MVP 구조를 정식 구조로 옮기는 작업
- 개발 제약 또는 작업 범위 확인

읽을 문서:

- `Docs/RefactoringCandidates.md`
- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
- 필요 시 `Docs/03_DevelopmentConstraints.md`
- 관련 도메인 문서

### Stage / Run / Stage Map 관련 작업

대상:

- Run 구조
- Stage Map
- Stage Node
- Stage 선택
- Stage Clear
- 다음 Stage 이동
- Mid Boss Stage / Boss Stage / Run Clear

읽을 문서:

- `Docs/StageRunStructure.md`
- `Docs/RoomCombatSystem.md`
- `Docs/RewardSystem.md`
- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`

### Room / Encounter 관련 작업

대상:

- 방 입장
- 문 열림/닫힘
- 방 단위 적 스폰
- RoomEncounterDataAsset
- SpawnPoints
- PlacedEnemyActors
- Room Clear
- 다음 Room 열림

읽을 문서:

- `Docs/RoomCombatSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/EnemySpawnSystem.md`
- `Docs/EnemySystem.md`
- `Docs/HealthSystem.md`

### Reward 관련 작업

대상:

- 보상 카드 UI
- Stage Clear 보상
- Instant / Run / Permanent 보상
- 보상 등급
- Luck / 보상 확률
- 보상으로 인한 스탯 변화
- 보상으로 인한 VFX 변화

읽을 문서:

- `Docs/RewardSystem.md`
- `Docs/ElementalRewardSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/AttackSkillSystem.md`
- `Docs/VFXSystem.md`

### Enemy 관련 작업

대상:

- 적 이동
- 적 공격
- 적 체력
- 적 사망
- 적 능력치
- 적 타입
- 적 겹침/분리
- 배치형 적 활성화

읽을 문서:

- `Docs/EnemySystem.md`
- `Docs/EnemyArchitecture.md`
- `Docs/EnemySpawnSystem.md`
- `Docs/HealthSystem.md`
- 애니메이션 변경이 있으면 `Docs/AnimationSystem.md`, `Docs/EnemyAnimationRetargeting.md`
- VFX 변경이 있으면 `Docs/VFXSystem.md`

### Enemy Spawn / Encounter 관련 작업

대상:

- RoomEncounterDataAsset
- InitialSpawnEntries
- WaveSpawnEntries
- SpawnDelay
- SpawnPoints
- PlacedEnemyActors
- Dormant / Active enemy
- EnemySpawnSubsystem 위치 조정

읽을 문서:

- `Docs/EnemySpawnSystem.md`
- `Docs/RoomCombatSystem.md`
- `Docs/EnemySystem.md`
- `Docs/EnemyArchitecture.md`

### VFX 관련 작업

대상:

- 검 궤적
- Niagara
- WeaponTrailComponent
- AnimNotifyState_WeaponTrail
- 보상에 따른 공격 VFX 성장
- 적 피격/사망/스폰 VFX
- 스파크, 번개, 출혈, 잔상 효과

읽을 문서:

- `Docs/VFXSystem.md`
- `Docs/ElementalRewardSystem.md`
- `Docs/AnimationSystem.md`
- `Docs/BasicAttackSystem.md`
- `Docs/AttackSkillSystem.md`
- 적 VFX이면 `Docs/EnemySystem.md`, `Docs/EnemyAnimationRetargeting.md`

### Animation 관련 작업

대상:

- AnimBP
- Montage
- BlendSpace
- AnimNotify
- 공격 애니메이션
- 적 스폰 인트로
- 적 피격/사망 애니메이션
- Retargeting

읽을 문서:

- `Docs/AnimationSystem.md`
- `Docs/EnemyAnimationRetargeting.md`
- 적 관련이면 `Docs/EnemySystem.md`, `Docs/EnemyArchitecture.md`
- 플레이어 공격 관련이면 `Docs/BasicAttackSystem.md`, `Docs/PlayerControlSystem.md`
- VFX 타이밍 관련이면 `Docs/VFXSystem.md`

### Player Control 관련 작업

대상:

- WASD 이동
- 회피
- 카메라
- 시작 인트로
- 무기 소켓 전환

읽을 문서:

- `Docs/PlayerControlSystem.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/BasicAttackSystem.md`

### Basic Attack / Skill 관련 작업

대상:

- 좌클릭 공격
- 콤보
- 공격 범위
- 검기
- 스킬
- 무기
- 성장 보상

읽을 문서:

- `Docs/BasicAttackSystem.md`
- `Docs/AttackSkillSystem.md`
- `Docs/ElementalRewardSystem.md`
- `Docs/RewardSystem.md`
- `Docs/VFXSystem.md`
- `Docs/AnimationSystem.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/HealthSystem.md`

### Health / Damage 관련 작업

대상:

- 데미지 처리
- 회복
- 무적
- 플레이어 사망
- 적 사망
- 보스 체력
- 방 클리어 판정용 사망 이벤트

읽을 문서:

- `Docs/HealthSystem.md`
- `Docs/BasicAttackSystem.md`
- `Docs/EnemySystem.md`
- `Docs/PlayerStatsSystem.md`
- 보상 회복이면 `Docs/RewardSystem.md`

## 7. 문서 상태 규칙

각 문서 상단에는 가능하면 아래 메타 정보를 둔다.

- Status: ACTIVE / CANDIDATE / DEPRECATED / REFERENCE
- Scope:
- Last Updated:
- Source of Truth: Yes / No

상태 의미:

- ACTIVE: 현재 구현 또는 현재 기준으로 사용한다.
- CANDIDATE: 후보 아이디어다. 현재 구현 기준으로 단정하지 않는다.
- DEPRECATED: 더 이상 현재 기준이 아니다.
- REFERENCE: 참고용이다.
- Source of Truth: Yes인 문서를 우선 기준으로 삼는다.

## 8. 작업 전 확인 규칙

Codex는 구현 또는 문서 개편 전에 아래를 먼저 요약한다.

1. 읽은 문서 목록
2. 이번 작업에 적용할 기준
3. 수정할 파일 후보
4. 수정하지 않을 파일 또는 영역

## 9. Archive 규칙

`Docs/Archive/` 폴더가 생기면, 그 안의 문서는 사용자가 명시적으로 요청하지 않는 한 현재 구현 기준으로 사용하지 않는다.
