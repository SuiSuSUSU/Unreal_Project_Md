---
Status: REFERENCE
Scope: Overnight Visual QA Report
Last Updated: 2026-07-11
Source of Truth: No
---

# Overnight Visual Bug Report - 2026-07-11

## 1. 실행 상태

- 작업 브랜치: `codex/overnight-visual-qa-2026-07-11`
- Unreal Editor 프로세스와 `MyProject - Unreal Editor` 창은 확인했다.
- Computer Use로 Unreal Editor 창 활성화와 화면 캡처를 요청했으나 앱 제어 승인이 만료되었다.
- 따라서 PIE 화면 관찰, 2회 재현, 수정 후 5회 반복 검증은 수행하지 못했다.
- 재현되지 않은 추측성 문제는 수정하지 않는다는 기준에 따라 gameplay 코드와 에셋은 수정하지 않았다.
- `Codex_VisualQA_Sleep_0800` 절전 예약도 명시적 승인 없이 생성하지 않았다.

## 2. 발견한 버그

현재 실제 PIE 화면으로 확인된 시각 버그는 없다.

정적 점검에서 아래 위험 후보를 확인했지만, 런타임 재현 전에는 버그로 확정하지 않는다.

1. `Lvl_TopDown`의 Room 7개 모두 `SpawnPoints` 배열이 비어 있다.
2. fallback 스폰은 `RoomCombatActor` 위치에 X 오프셋만 더하고 지면 투영을 하지 않는다.
3. Room 0과 Room 1의 Actor Z는 약 `169.28`, Room 2 이후 일부 Room은 Z가 `0`으로 서로 다르다.
4. 적 Capsule Half Height는 공통적으로 약 `96`이므로 fallback 스폰 높이에 따라 초기 지면 간격이 달라질 가능성이 있다.

## 3. 재현 방법

승인 후 다음 순서로 우선 재현한다.

1. `/Game/TopDown/Lvl_TopDown`에서 PIE를 시작한다.
2. Room 0에 진입해 Little Demon 초기 스폰을 관찰한다.
3. Actor 위치, Capsule 하단, Mesh Bounds 하단, 지면 Trace 높이를 기록한다.
4. 동일 진입을 최소 2회 반복한다.
5. Room 1과 Z=0인 Room에서도 같은 비교를 수행한다.
6. Spawn Intro 시작 전, 진행 중, 종료 후의 높이 변화를 각각 확인한다.

## 4. 확인된 근본 원인

런타임 재현이 없으므로 확인된 근본 원인은 없다.

정적 후보는 `RoomCombatActor::GetSpawnTransform`의 fallback 경로가 지면 Trace 또는 NavMesh 투영 없이 Room Actor 위치를 그대로 사용한다는 점이다. 이 후보는 실제 화면과 좌표 계측으로 검증하기 전에는 수정하지 않는다.

## 5. 적용한 수정

- gameplay 코드 수정 없음
- Blueprint/AnimBP 수정 없음
- uasset/umap 수정 없음
- Config 수정 없음
- QA 보고서만 생성

## 6. 빌드 및 반복 검증 결과

- 이번 QA 세션에서 새 코드 변경이 없어 별도 빌드는 실행하지 않았다.
- PIE 기준 실행 확인: `BLOCKED`
- 동일 현상 2회 재현: `BLOCKED`
- 수정 후 5회 반복 검증: 해당 없음
- 간헐 문제 10회 검증: 해당 없음

## 7. 변경 파일과 에셋

- 생성: `Docs/Overnight_Visual_Bug_Report_2026-07-11.md`
- 변경한 C++/Blueprint/AnimBP/uasset/umap: 없음

## 8. 로컬 커밋

- 없음. 검증을 통과한 버그 수정이 없으므로 체크포인트 커밋을 만들지 않았다.

## 9. BLOCKED

### Unreal Editor Computer Use 승인

- 상태: `BLOCKED`
- 증상: Unreal Editor 창은 감지됐지만 활성화/캡처 단계의 앱 제어 승인이 시간 초과됐다.
- 영향: PIE 실행, 실제 화면 관찰, 스크린샷 기록, 반복 재현 및 검증을 진행할 수 없다.
- 해제 조건: 사용자가 Codex의 Unreal Editor 제어 승인 창을 허용한다.

### 오전 8시 절전 예약

- 상태: `BLOCKED`
- 작업 이름: `Codex_VisualQA_Sleep_0800`
- 영향: 2026-07-11 08:00 KST 일회성 절전 예약이 생성되지 않았다.
- 해제 조건: 사용자가 Windows 작업 예약 생성 권한을 명시적으로 승인한다.

## 10. NEEDS_HUMAN_REVIEW

- 없음. 미적 취향에 관한 수정 판단 단계까지 도달하지 않았다.

## 11. 다음 사용자 확인 항목

1. Codex의 Unreal Editor 제어 승인 창을 허용한다.
2. Room 0 스폰 순간에 적이 공중에서 떨어지거나 지면 아래에서 보이는지 확인한다.
3. Spawn Intro 전후 Little Demon의 발 위치가 달라지는지 확인한다.
4. Room 1과 Room 2의 스폰 높이 차이가 화면에서 드러나는지 확인한다.
5. 절전 예약 생성을 계속 원하면 해당 시스템 작업을 명시적으로 승인한다.
