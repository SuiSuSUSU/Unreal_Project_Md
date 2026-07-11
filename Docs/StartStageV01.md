---
Status: CANDIDATE
Scope: Start Stage v0.1 Editor Setup
Last Updated: 2026-07-10
Source of Truth: No
---

# Start Stage v0.1

## 1. 문서 목적

이 문서는 Start Stage의 첫 3-Room 플레이 테스트 구성을 기록한다.

DataAsset 생성과 Room 인스턴스 연결은 2026-07-10에 완료했다. SpawnPoint 배치 확인과 PIE 검증이 남아 있으므로 아직 구현 완료 기준으로 승격하지 않는다.

현재 에디터 설정 상태:

- `/Game/Data/Rooms`에 Encounter DataAsset 3개 생성 및 저장 완료
- `RoomOrder 0~2`에 `StageId=Stage1`과 각 `EncounterData` 연결 완료
- 세 Room의 `PlacedEnemyActors` 비움
- `StageFlowManager.ActiveStageRoomCount=3`
- `StageFlowManager` 런타임 Encounter override 비활성화
- PIE 타이밍 및 5회 softlock 검증 대기

| 구분 | 현재 상태 |
|---|---|
| C++ 코드 | 전체 Editor 빌드 및 정적 검토 완료 |
| Encounter DataAsset | 3개 생성 및 입력값 검증 완료 |
| 맵 인스턴스 연결 | Room 0~2, StageId, RoomOrder, EncounterData와 StageFlowManager 설정 검증 완료 |
| PIE | SpawnPoint, Initial/Wave 타이밍, 중간 Room 진행, 마지막 보상, 5회 softlock 검증 대기 |

현재 맵에는 `TEST_Enemy_RangedDummy`가 PlayerStart 근처에 별도 테스트 Actor로 배치되어 있다. 이 Actor는 Room의 `PlacedEnemyActors`와 세 Start Stage Encounter에 포함되지 않으며 Room Clear 대상이 아니다. Start Stage 자체의 5회 안정화 테스트에서는 전투 간섭을 피하기 위해 필요 시 에디터에서 일시 비활성화한다.

## 2. Stage 기준

- `StageId`: `Stage1`
- 활성 Room: `RoomOrder 0~2`
- 중간 Room Clear: 다음 Room만 활성화
- 마지막 Room Clear: Stage Clear 보상 표시
- `PlacedEnemyActors`: 사용하지 않음
- `EnemySpawnSubsystem`: 비활성 유지
- Clear 조건: 모든 Room에서 `AllTrackedEnemiesDefeated`

## 3. Encounter 후보

| RoomOrder | DataAsset | InitialSpawnEntries | WaveSpawnEntries | SpawnDelay |
|---|---|---|---|---:|
| 0 | `DA_Start_Room01_BasicMelee` | `BP_Enemy_LittleDemon x2` | `BP_Enemy_LittleDemon x1` | 5.0초 |
| 1 | `DA_Start_Room02_FastPriority` | `BP_Enemy_Fast x1`, `BP_Enemy_LittleDemon x1` | `BP_Enemy_LittleDemon x1` | 6.0초 |
| 2 | `DA_Start_Room03_TankMixed` | `BP_Enemy_Tank x1`, `BP_Enemy_LittleDemon x1` | `BP_Enemy_Fast x1` | 8.0초 |

`InitialSpawnEntries`는 Room 전투 시작 시 함께 생성된다. `WaveSpawnEntries`의 첫 후속 적은 Initial 적의 생존 여부와 관계없이 Room 전투 시작 시점부터 `SpawnDelay`가 지난 뒤 합류한다. 예약된 Wave와 생성된 Wave 적이 모두 처리되기 전에는 Room Clear가 발생하지 않는다.

## 4. Stage Clear 고정 보상 후보

Stage Clear 카드 수는 3장으로 두고 아래 순서를 사용한다.

1. `run_attack_damage_5`
2. `run_lightning_current_slash`
3. `run_dodge_cooldown_015`

현재 `RewardActor`의 기존 효과 적용 경로를 재사용한다. 랜덤 보상 풀이나 `RewardDataAsset`은 만들지 않는다.

## 5. 구현 완료 판정

아래 조건을 모두 통과하기 전에는 이 구성을 현재 구현 완료로 승격하지 않는다.

- 세 DataAsset이 Unreal Editor에서 생성되고 저장됨
- 세 `BP_RoomCombatActor` 인스턴스에 올바른 `EncounterData`가 연결됨
- `StageFlowManager`의 런타임 Encounter override가 꺼져 있음
- Room 0과 Room 1에서 보상 UI가 표시되지 않음
- Room 2 Clear 후 고정 보상 3장만 한 번 표시됨
- 각 Room의 Initial/Wave 순서와 SpawnDelay가 PIE에서 확인됨
- 같은 흐름을 5회 반복해 softlock가 없음

## 6. 관련 문서

- `Docs/RoomCombatSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/EnemySpawnSystem.md`
- `Docs/RewardSystem.md`
- `Docs/CurrentImplementation.md`
