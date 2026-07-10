# AGENTS.md

이 문서는 Codex가 `D:\Unreal_Pro\MyProject`에서 작업하기 전에 확인할 최소 지침이다.

## 문서 읽기 규칙

- 모든 작업 전 `Docs/00_ReadFirst.md`를 먼저 확인한다.
- 공통 기준 문서도 확인한다.
  - `Docs/05_DecisionLog.md`
  - `Docs/CurrentImplementation.md`
  - `Docs/10_ProjectClassMap.md`
  - `Docs/04_CodexWorkflow.md`
- 작업 종류에 따라 `Docs/00_ReadFirst.md`가 안내하는 세부 문서만 추가로 읽는다.
- `CANDIDATE`, `DEPRECATED`, `ARCHIVE` 상태의 문서는 현재 구현 기준으로 사용하지 않는다.
- `Source of Truth: Yes`인 문서를 우선 기준으로 삼는다.

## 작업 전 요약 규칙

코드나 문서를 수정하기 전에는 아래를 먼저 요약한다.

1. 읽은 문서 목록
2. 이번 작업에 적용할 기준
3. 수정할 파일 후보
4. 수정하지 않을 파일 또는 영역

## 수정 제한

- C++ 코드, Blueprint, uasset 파일은 사용자가 명시적으로 요청하지 않으면 수정하지 않는다.
- Blueprint/uasset 변경은 가능하면 Unreal Editor 안에서 처리한다.
- 사용자가 만든 변경이나 에디터에서 저장한 에셋을 임의로 되돌리지 않는다.
- `Config`, `Content`, `Source`를 수정할 때는 현재 실제 맵과 Blueprint 연결을 먼저 확인한다.

## 현재 주요 기준

- 기본 맵은 현재 `/Game/TopDown/Lvl_TopDown`이다.
- 현재 플레이 흐름은 대략 아래 구조를 기준으로 본다.

```text
/Game/TopDown/Lvl_TopDown
-> /Game/Blueprints/Core/BP_CombatGameMode
-> /Game/Blueprints/Player/BP_PlayerCharacter
-> /Game/Blueprints/Player/BP_PlayerController
```

- 실제 구현 상태는 항상 `Docs/CurrentImplementation.md`를 우선 확인한다.
- 클래스 책임과 현재 사용 여부는 `Docs/10_ProjectClassMap.md`를 확인한다.
- Room/Stage/Reward/Enemy/VFX/Animation 관련 작업은 `Docs/00_ReadFirst.md`의 도메인별 라우팅을 따른다.

## Unreal 작업 주의

- C++ 타입, UPROPERTY, UFUNCTION, Build.cs 변경은 에디터 종료 후 전체 빌드를 우선한다.
- Live Coding 중에 C++ 클래스 추가/삭제를 하지 않는다.
- 빌드 기본 명령은 다음과 같다.

```powershell
& 'C:\Program Files\Epic Games\UE_5.7\Engine\Build\BatchFiles\Build.bat' MyProjectEditor Win64 Development 'D:\Unreal_Pro\MyProject\MyProject.uproject' -WaitMutex
```

- 사용자가 정상 구현 시 에디터 실행을 원한다고 했으므로, C++ 변경 빌드 성공 후 가능하면 Unreal Editor를 다시 실행한다.
