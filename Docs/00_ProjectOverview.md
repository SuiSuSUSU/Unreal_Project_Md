---
Status: ACTIVE
Scope: Project Overview
Last Updated: 2026-07-06
Source of Truth: Yes
---

# 프로젝트 개요

## 문서 목적

이 문서는 Unreal Engine 5.7 기반 1인 개발 프로젝트의 현재 게임 방향과 핵심 기획 요약을 정리한다.

Codex는 큰 방향을 확인할 때 이 문서를 먼저 보고, 세부 구조는 `Docs/StageRunStructure.md`, 구현 상태는 `Docs/CurrentImplementation.md`, 결정 이력은 `Docs/05_DecisionLog.md`를 확인한다.

## 현재 게임 방향

현재 게임 방향은 스테이지 클리어형 탑다운 액션 로그라이트다.

하나의 `Run`은 왼쪽에서 오른쪽으로 진행되는 `Stage Map`에서 여러 `Stage` 노드를 선택하며 진행된다. `Stage Map`의 노드 하나는 전투방 하나가 아니라 여러 `Room`을 포함하는 하나의 `Stage`를 의미한다.

일반 `Stage`는 여러 `Room`으로 구성된다. 현재 테스트 기준으로는 `Room 0~6`, 총 7개 Room 구성을 사용해보지만, 이는 최종 확정값이 아니다. Room 개수는 Stage 타입, 난이도, 플레이 피로도, 플레이 테스트 결과에 따라 조정할 수 있다.

장기 보상 흐름은 `Stage Clear -> 보상 카드 선택 -> 보상 적용 -> Stage Map 복귀 -> 다음 Stage 선택`이다. 현재 Room Clear 보상 흐름은 MVP 테스트 파이프라인이며 최종 보상 타이밍으로 보지 않는다.

Stage 선택은 현재 빌드, 체력, 위험도, 보상 경향을 보고 고르는 전략 선택이어야 한다. 다음 Stage 후보는 Stage Clear 보상을 선택하고 적용한 뒤 표시하는 방향을 기준으로 한다.

Mid Boss Stage는 Run 중간 리듬 변화를 담당하고, 가장 오른쪽의 Boss Stage는 Final Boss와 Run Clear를 담당한다. Boss Stage는 일반 Stage와 별도 구조이며, 일반 Stage마다 보스가 있는 구조도 아니다.

## 현재 상위 루프

```text
Run Start
-> Stage Map
-> Choose connected Stage
-> Clear Rooms inside selected Stage
-> Stage Clear reward card selection
-> Apply selected reward
-> Return to Stage Map
-> Show connected next Stage candidates
-> Choose next connected Stage
-> Reach Boss Stage
-> Defeat Final Boss
-> Run Clear
```

## 확정된 내용

- 개발자는 3D 모델러다.
- 1인 개발을 목표로 한다.
- 코딩은 AI/Codex에 크게 의존할 예정이다.
- 엔진은 Unreal Engine 5.7을 사용할 예정이다.
- 기획 문서는 Markdown 파일로 관리할 가능성이 높다.
- Codex가 프로젝트 문서와 코드를 읽고 참고할 수 있는 개발 문서를 만들고자 한다.
- 현재 게임 방향은 스테이지 클리어형 탑다운 액션 MVP다.
- 진행 구조는 작은 방/구역을 하나씩 클리어하는 방 클리어형 구조를 우선한다.
- 현재 구현된 WASD 이동, 좌클릭 1/2/3타 콤보, Space 회피, `HealthComponent`, `EnemyBase`, `EnemyStatsDataAsset`, Little Demon 적 구조, 기본 전투 구조는 유지한다.

## 현재 핵심 루프

Room 단위 루프:

1. 방 진입
2. 전투 시작
3. 문 닫힘
4. 정해진 적 스폰
5. 적 처치
6. 방 클리어
7. 현재 MVP에서는 보상 테스트 또는 다음 방 이동
8. 다음 방 이동

Stage 단위 루프:

1. Stage 시작
2. 여러 개의 Room을 순서대로 클리어
3. 현재 테스트 기준의 마지막 Room 클리어
4. Stage Clear
5. Stage Clear 보상 카드 선택
6. 선택한 보상 적용
7. Stage Map 복귀
8. 다음 Stage 후보 선택
9. 선택한 다음 Stage 진입

Stage 1 현재 테스트 기준:

- Stage 1은 현재 테스트 베이스라인으로 `Room 0~6`, 총 7개 Room 구성을 사용해본다.
- 7개 Room은 최종 확정 규칙이 아니다.
- Stage 1의 정확한 Room 개수는 레벨 디자인 테스트와 플레이 피로도 확인 후 조정할 수 있다.
- 일반 Stage마다 보스가 있는 구조는 아니다.

## 채택된 Stage 1 보상 방향

- Stage 1은 짧은 전투방을 연속으로 돌파하면서 보상과 전투 흐름을 검증하는 구조를 우선한다.
- 방 하나하나는 짧게 끝나야 하며, 장기 기획에서는 Stage Clear 시점에 보상 후보 카드가 바뀌어야 한다.
- 첫 스테이지부터 단순 수치 상승만이 아니라 전투 방식이 조금씩 바뀌는 빌드 체감을 제공한다.
- 보상 카드는 등급 구조를 가진다. Bronze, Silver, Gold, Prism은 현재 후보 명칭이며 최종 등급 수와 이름은 미정이다.
- 높은 등급 보상일수록 3타 공격, 회피 후 공격, 처치 회복, 출혈, 잔상 폭발처럼 플레이 방식 변화가 큰 효과를 우선한다.
- 같은 기술 강화라도 등급에 따라 효과량과 질이 달라질 수 있다. 예를 들어 Bronze는 작은 수치 증가, Prism은 공격 방식 변화 같은 큰 효과를 줄 수 있다.
- 높은 등급 보상은 희귀하게 등장하며, 추후 운 스탯이 등장 확률에 영향을 주는 방향을 검토한다.

## Stage 선택 방향

Stage를 클리어하면 바로 다음 Stage로 자동 이동하는 방식만 쓰지 않고, 다음 Stage 후보 2~3개 중 하나를 선택하는 구조를 장기 방향으로 둔다.

목적:

- 현재 빌드와 체력 상태를 보고 다음 목적지를 고르게 한다.
- 다음 Stage의 적 구성과 보상 경향을 미리 예상하게 한다.
- Room 안에서는 액션 판단을, Stage 사이에서는 전략 판단을 하게 한다.

Stage 후보 예시:

| Stage 후보 | 특징 |
|---|---|
| 오염된 하수구 | 다수 적 중심, 처치 보상과 범위 공격이 유리 |
| 버려진 병기고 | 강적 중심, 3타 강화와 출혈 보상이 유리 |
| 붉은 숲 | 빠른 적 중심, 회피 강화와 반격 보상이 유리 |

빌드와 Stage 연결 예시:

| 현재 빌드 | 잘 맞는 Stage |
|---|---|
| 3타 검기 / 공격 범위 증가 | 다수 적 스테이지 |
| 출혈 / 3타 강화 / 흡혈 | 강적 스테이지 |
| 회피 강화 / 반격 / 이동속도 | 빠른 적 스테이지 |
| 체력이 낮은 상태 | 회복 스테이지 |
| 빌드가 강하게 완성된 상태 | 고위험 스테이지 |

현재 구현 우선순위에서는 Stage 1의 Room 루프와 Stage Clear 보상 흐름을 먼저 안정화하고, Stage 선택 구조와 Boss Stage는 그 다음 확장으로 둔다.

## 진행 용어 기준

| 용어 | 의미 |
|---|---|
| `Stage` | 하나의 큰 구역, 챕터, 테마 |
| `Room` | `Stage` 안에서 전투가 벌어지는 개별 방 |
| `Encounter` | `Room` 안에서 실제로 나오는 적 조합 |
| `Wave` | `Encounter` 안에서 순차적으로 나오는 적 묶음 |
| `Run` | 게임 시작부터 죽거나 클리어할 때까지의 전체 플레이 |

기본 계층은 `Run > Stage > Room > Encounter > Wave`로 본다.

## 미정인 내용

- 세계관과 분위기
- 플레이어 캐릭터 콘셉트
- 적, 보스, 오브젝트 최종 구성
- Stage Clear 보상 수치, 등장 확률, 보상 풀, 희귀도 가중치
- Stage 1의 최종 Room 개수
- Stage 후보 개수와 Stage별 위험/보상 표시 방식
- 운 스탯의 최종 역할과 계산 방식
- 플레이어 성장 방식의 최종 형태
- 출시 플랫폼
- 상용화 범위와 목표 완성도

## 보류 또는 제외한 내용

- 무한히 적이 계속 나오는 뱀파이어 서바이벌류 구조는 현재 메인 방향에서 보류한다.
- 랜덤 던전 생성은 초기 MVP에서 제외한다.
- 상점, 열쇠, 복잡한 아이템 시너지는 초기 MVP에서 제외한다.
- 뱀파이어 서바이벌류 요소는 나중에 제한 시간 생존 방, 이벤트 방, 특수 웨이브 방으로 재활용할 수 있다.

## 앞으로 결정해야 할 내용

- Stage 1 기준 최소 플레이 가능 버전 범위
- 방별 적 구성과 Encounter/Wave 구성
- Stage별 적 구성, 보상 후보, 등급 등장 비율, 보상 경향
- 문/보상/다음 방 이동 상호작용 방식
- Codex가 안정적으로 구현할 수 있는 기능 단위
- 코드와 블루프린트 역할 분담

## 문서 관리 목적

- 확정된 내용과 논의 중인 아이디어를 분리한다.
- Codex가 구현 전 참고할 기준 문서를 제공한다.
- 1인 개발자가 감당 가능한 범위로 기획을 유지한다.
- 결정 사항을 추적해서 기획 변경으로 인한 혼선을 줄인다.
