---
Status: ACTIVE
Scope: Decision Log
Last Updated: 2026-07-01
Source of Truth: Yes
---

# 결정 로그

## 2026-07-01 Update: Run / Stage Map Structure Decisions

| 결정일 | 결정 내용 | 결정 이유 | 보류/대안 | 추후 재검토 여부 |
|---|---|---|---|---|
| 2026-07-01 | Run은 Stage Map에서 여러 Stage를 선택해 진행하고, Stage Map의 노드 하나는 Room이 아니라 Stage 하나로 정의한다. | Room과 Stage를 혼동하면 진행 구조, 보상 타이밍, 맵 UI 설계가 모두 흔들리기 때문이다. | Stage Node 하나를 Room 하나로 취급 | `Docs/StageRunStructure.md`를 기준 문서로 유지 |
| 2026-07-01 | 일반 Stage는 여러 Room으로 구성하며, 현재 기준은 7개 Room 전후를 baseline으로 하되 유동 가능하게 둔다. | Run 길이와 피로도를 조절하기 위해 모든 Stage를 항상 7개 Room 고정으로 박지 않기 위함이다. | 모든 Stage를 항상 7개 Room으로 고정 | Stage 타입과 플레이 테스트에 따라 Room 수 재조정 |
| 2026-07-01 | 장기 보상 타이밍은 Room마다가 아니라 Stage Clear 시점으로 둔다. | Room마다 보상을 주면 성장 속도가 과해지고 보상 가치가 희석될 수 있기 때문이다. | 모든 Room Clear마다 빌드 보상 지급 | 현재 Room-clear `RewardActor`는 MVP 테스트 파이프라인으로만 유지 |
| 2026-07-01 | Stage Clear 이후 순서는 보상 선택, 보상 적용, Stage Map 복귀, 다음 Stage 선택으로 둔다. | 플레이어가 보상으로 갱신된 빌드 상태를 보고 다음 목적지를 고르게 하기 위함이다. | 다음 Stage 선택 UI를 보상 선택보다 먼저 표시 | Stage Clear 보상과 Stage Map UI가 구현되기 전까지 현재 Room 보상과 재시작 오버레이는 MVP fallback으로 유지 |
| 2026-07-01 | Mid Boss Stage와 Final Boss Stage를 분리한다. | Run 중간 리듬 변화와 최종 목표를 분리해 진행감을 만들기 위함이다. | 모든 Stage 끝에 보스 배치 | Mid Boss는 Run 중간 특수 Stage, Final Boss는 가장 오른쪽 Boss Stage |
| 2026-07-01 | `StageFlowManager` 1차 MVP는 Room 순서 관리와 Stage Clear fallback까지만 담당한다. | 현재 `RoomCombatActor`가 너무 많은 진행 책임을 들고 있으나, Stage Map/보상/보스까지 한 번에 옮기면 위험하기 때문이다. | 처음부터 전체 Run/Stage Map/보상/보스 흐름을 모두 담당 | 1차 manager 경로가 에디터에서 안정화된 뒤 Stage Map과 Stage Clear 보상으로 확장 |
| 2026-07-01 | 다음 Room으로 가는 길은 현재 Room의 모든 몬스터를 처치하면 열린다. 보상 카드는 Room Clear가 아니라 Stage Clear 시점에 지급한다. | Room Clear의 의미와 Stage Clear 보상 타이밍을 분리해야 진행과 성장 구조가 섞이지 않기 때문이다. | Room마다 보상 카드 선택 후 다음 Room 열림 | 기존 Room Clear 보상 카드는 MVP 테스트 파이프라인으로만 유지하고, `StageFlowManager` 진행 이관 시 끄거나 Stage Clear 보상으로 옮김 |
| 2026-07-01 | `NextRoomActors`와 `bAutoFindNextRoomByOrder`는 아직 deprecated하지 않는다. | `StageFlowManager` 신호 브리지가 에디터 Output Log에서 확인되기 전까지 진행권을 옮기면 방 진행이 멈출 위험이 있기 때문이다. | 즉시 `StageFlowManager` 중심으로 전환 | manager가 다음 Room 활성화와 마지막 Room Stage Clear fallback을 검증한 뒤 fallback/테스트용으로 낮춤 |
| 2026-07-01 | `StageFlowManager` 다음 구현은 후보 계산, 기존 handoff 비교, 실제 활성화, Stage Clear fallback 이관 순서로 나눈다. | 바로 다음 방 활성화를 옮기면 신호/후보 계산 오류가 플레이 흐름 중단으로 이어질 수 있기 때문이다. | 후보 검증 없이 바로 다음 Room 활성화 이관 | Step A/B 로그 검증 후 Step C, Step C 검증 후 Step D 진행 |
| 2026-07-10 | 3개월 목표의 첫 구현 단위는 3 Room Start Stage 안정화로 둔다. | 새 시스템을 많이 늘리기보다 12~18분 완결 Run의 기초가 되는 전투 -> Stage Clear 보상 흐름을 먼저 안정화하기 위함이다. | 기존 7 Room 테스트 구성을 그대로 3개월 기준으로 사용 | 3 Room Start Stage를 5회 반복 테스트한 뒤 Stage Map 작업으로 이동 |
| 2026-07-10 | 현재 Start Stage 테스트 Encounter는 Room 0 Little Demon 중심, Room 1 Fast 중심, Room 2 Tank+Little Demon 조합으로 둔다. | Little Demon 기본 전투, Fast 회피 압박, Tank 마무리 전투를 짧게 검증하기 위함이다. | 적 조합 확정 전에 Stage Map부터 구현 | 안정화 후 유지할 Encounter는 `RoomEncounterDataAsset`으로 이전 |

이 문서는 앞으로 확정된 내용을 기록한다. 결정된 방향과 아직 미정인 항목을 분리해서 관리한다.

## 기록 형식

| 결정일 | 결정한 내용 | 결정 이유 | 보류한 대안 | 추후 재검토 여부 |
| --- | --- | --- | --- | --- |
| 2026-05-23 | Unreal Engine 5.7 사용 예정 | 현재 프로젝트가 UE 5.7 기반이며 개발 환경이 준비되어 있음 | 다른 엔진 또는 다른 UE 버전 | 필요 시 재검토 |
| 2026-05-23 | 1인 개발 목표 | 개발자의 현실적인 작업 방식에 맞춤 | 팀 개발 | 현재는 재검토 예정 없음 |
| 2026-05-23 | 개발자는 3D 모델러 | 개발자의 핵심 강점과 역할을 명확히 하기 위함 | 코딩 중심 개발자 역할 | 재검토 예정 없음 |
| 2026-05-23 | 코딩은 Codex/AI에 의존 예정 | 코딩 부담을 줄이고 작은 기능 단위로 구현하기 위함 | 직접 코딩 중심 개발 | 필요 시 일부 학습 병행 |
| 2026-05-23 | 기획 문서는 Markdown 기반으로 관리하는 방향 검토 | Codex가 읽고 수정하기 쉽고 변경 이력을 남기기 좋음 | 문서 없이 대화만으로 진행 | 문서 관리가 부담되면 형식 조정 |
| 2026-06-11 | 스테이지 클리어형 탑다운 액션 MVP로 방향 전환 | 현재 구현된 WASD 이동, 좌클릭 콤보, Space 회피, `EnemyBase`, Little Demon 적 구조를 유지하면서 작은 방 단위 전투로 확장하기 적합함 | 뱀파이어 서바이벌류 무한 생존 액션 | 뱀서류 요소는 제한 시간 생존 방, 이벤트 방, 특수 웨이브 방으로 재검토 가능 |
| 2026-06-11 | 초기 MVP는 고정된 방 2~3개를 같은 맵에 배치하는 방식 | 1인 개발과 Codex 의존 개발에서 랜덤 던전보다 구현/검증 부담이 낮음 | 랜덤 던전, 상점, 열쇠, 복잡한 아이템 시너지 | 핵심 방 전투가 안정된 뒤 재검토 |
| 2026-06-11 | 방 클리어형 핵심 루프 채택 | 방 진입, 문 닫힘, 정해진 적 스폰, 적 전멸, 방 클리어, 보상 획득, 다음 방 이동 구조가 명확함 | 무한 전역 스폰 루프 | 특수 방에서만 일부 재사용 가능 |
| 2026-06-26 | Stage 1 보상 방향은 등급별 카드 선택과 빌드 체감 중심으로 채택 | 짧은 전투방을 연속으로 돌파한 뒤 Stage Clear 보상으로 첫 스테이지부터 전투 방식 변화와 빌드의 맛을 주기 위함 | 단순 스탯 보상만 반복하는 구조 | 보상 수치, 등장 확률, 보상 풀, 희귀도 가중치는 구현 전 재검토 |
| 2026-06-27 | Stage는 여러 Room의 묶음으로 보고, Stage 클리어 후 다음 Stage 후보를 선택하는 방향을 채택 | Room 안에서는 액션 판단을, Stage Clear 후에는 성장 선택을, Stage 사이에서는 현재 빌드에 맞는 위험/보상 선택을 주기 위함 | 모든 Stage를 고정 순서로 자동 진행 | Stage 1 루프와 보스방이 안정된 뒤 구현 순서 재검토 |
| 2026-06-27 | 보상 등급은 보상 품질을, 기술 레벨은 성장 누적을 표현하는 방향으로 분리 | 같은 기술 강화라도 등급에 따라 효과량과 질이 달라지고, 누적 레벨로 장기 성장감을 줄 수 있음 | 등급 없이 단일 레벨업만 사용하는 구조 | Bronze/Silver/Gold/Prism 명칭과 정확한 등급 수는 추후 재검토 |

## 현재 확정된 게임 방향

- 장르 방향: 스테이지 클리어형 탑다운 액션 MVP.
- 전투 방향: 현재 좌클릭 1/2/3타 콤보, WASD 이동, Space 회피 구조 유지.
- 스테이지 방향: 작은 방/구역을 하나씩 클리어하는 방 클리어형 진행.
- 적 구조: `EnemyBase`, `EnemyStatsDataAsset`, Little Demon 적 구조 유지.
- 스폰 방향: 전역 무한 스폰보다 방 입장 시 SpawnPoint 기준 정해진 적 생성으로 전환.
- Stage 구조: 하나의 Stage는 여러 Room의 묶음이며, 7개 Room은 과거 테스트 baseline일 뿐 확정값이 아니다. 현재 3개월 첫 단계는 3 Room Start Stage 안정화로 둔다.
- Stage 1 방향: 짧은 전투방을 연속으로 클리어하고, 마지막 방 또는 별도 보스방에서 첫 보스를 처치하는 구조를 우선한다.
- 보상 방향: Stage Clear 시점에 후보 카드가 바뀌며, 등급형 보상 카드로 보상 강도를 나누는 구조를 채택한다. Bronze/Silver/Gold/Prism은 현재 후보 명칭이다.
- 성장 방향: 첫 스테이지부터 3타 강화, 회피 후 공격 강화, 처치 회복, 출혈, 잔상 폭발처럼 전투 방식이 체감되게 바뀌는 보상을 우선한다.
- Stage 선택 방향: Stage 클리어 후 다음 Stage 후보 2~3개를 보여주고, 현재 빌드와 체력 상태에 맞는 다음 목적지를 선택하게 한다.
- 운 스탯 방향: 희귀한 높은 등급 보상, 특히 Prism 계열 등장 확률에 영향을 주는 후보 스탯으로 검토한다.

## 아직 확정하지 않은 항목

- 테마와 세계관
- 보상 수치, 등장 확률, 보상 풀, 희귀도 가중치
- Bronze/Silver/Gold/Prism 명칭과 최종 등급 수
- 운 스탯의 정확한 계산 방식
- 플레이어 성장 방식의 최종 형태
- 엘리트방/보스방 구성
- 출시 플랫폼
- 최종 프로토타입 범위
