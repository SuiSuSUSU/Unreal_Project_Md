---
Status: ACTIVE
Scope: Player Control
Last Updated: 2026-07-05
Source of Truth: Yes
---

# PlayerControlSystem.md

## 1. 문서 목적

이 문서는 플레이어 조작, 이동, 회피, 공격 중 이동 제한, 시작 인트로, 무기 소켓 전환, 디버그 입력 기준을 정리한다.

Codex가 입력 로직을 수정할 때 실제 사용 중인 `BP_PlayerCharacter` / `BP_PlayerController` 흐름을 혼동하지 않도록 기준을 제공한다.

## 2. 현재 실제 사용 구조

| 구분 | 현재 값 |
|---|---|
| Player Character | `BP_PlayerCharacter` |
| Player Controller | `BP_PlayerController` |
| 이동 / 회피 / 인트로 처리 | `PlayerKeyboardMovementSubsystem` |
| 공격 처리 | `PlayerMeleeAttackSubsystem` |
| 카메라 줌 | `MouseWheelCameraZoomSubsystem` |
| 플레이어 런타임 스탯 | `PlayerStatsComponent` |

주의:

- `MyProjectCharacter`와 `MyProjectPlayerController` 파일은 존재하지만, 현재 기본 플레이 흐름의 핵심 수정 위치로 보지 않는다.
- 입력 관련 작업 전에는 실제 맵의 GameMode, Pawn, Controller, Blueprint 부모 연결을 먼저 확인한다.

## 3. 현재 입력 배치

| 입력 | 현재 역할 |
|---|---|
| WASD | 이동 |
| Left Mouse Button | 좌클릭 근접 1/2/3타 콤보 공격 |
| Right Mouse Button | 현재 기본 역할 없음 |
| Space | 회피 |
| Mouse Wheel | 카메라 줌 인/아웃 |
| H | 디버그용 플레이어 체력 감소 |
| L | 디버그용 전투 속성 `None` / `Lightning` 토글 |
| F10 | 디버그용 슬로모션 0.1x / 1.0x 토글 |

## 4. 이동 시스템

- 기존 TopDown 클릭 이동은 사용하지 않는 방향이다.
- `PlayerKeyboardMovementSubsystem`이 기존 클릭 이동 Mapping Context를 제거한다.
- WASD 입력은 카메라 기준 방향으로 변환한 뒤 `AddMovementInput`으로 적용한다.
- 공격 중에는 이동 입력을 막는다.
- 회피 중에는 일반 이동을 하지 않는다.

## 5. 회피 시스템

현재 회피는 `Space`로 발동한다.

| 항목 | 현재 처리 |
|---|---|
| 입력 | `Space` |
| 방향 | WASD 입력 방향 우선, 없으면 캐릭터 전방 |
| 이동 방식 | 짧은 시간 동안 `AddActorWorldOffset` |
| 충돌 | Sweep 사용 |
| 무적 | `HealthComponent::SetInvulnerable` |
| 수치 관리 | `PlayerStatsComponent` / `PlayerStatsDataAsset` |

관련 Data Asset 값:

- `DodgeDistance`
- `DodgeDuration`
- `DodgeCooldown`
- `DodgeInvulnerabilityDuration`

## 6. 공격 중 캔슬 규칙

현재 전투 감각은 아래 방향으로 정리한다.

| 입력 | 공격 중 동작 |
|---|---|
| WASD | 공격 캔슬 불가, 이동 불가 |
| Left Mouse Button | 다음 콤보 입력 버퍼 |
| Space | 공격 캔슬 후 회피 |

구현 연결:

- 공격 중 이동 잠금은 `PlayerMeleeAttackSubsystem`이 처리한다.
- 회피 입력은 `PlayerKeyboardMovementSubsystem`에서 처리한다.
- 공격 중 Space 회피가 시작되면 `PlayerMeleeAttackSubsystem::CancelAttackForDodge`가 공격 진행, 이동 락, 예약된 타격을 정리한다.

## 7. 시작 인트로 애니메이션

게임 시작 시 `PlayerKeyboardMovementSubsystem`에서 인트로 애니메이션을 1회 재생한다.

현재 애니메이션:

```text
/Game/PowerfulSwordPack/Animations/Katana_Blade(SK_Mannequin)/1_Movements/0__Intro/Katana_Blade_Intro
```

인트로 중에는 이동 입력을 잠근다.

## 8. 무기 / 검집 소켓 전환

현재 `BP_PlayerCharacter`에는 아래 컴포넌트가 있다.

| 컴포넌트 | 역할 |
|---|---|
| `EquippedOtachi` | 손에 장착되는 검 |
| `Otachi_Handle` | 검 손잡이 후보 메시 |
| `EquippedOtachiSheath` | 등에 장착되는 검집 |

현재 사용하는 소켓:

| 소켓 | 역할 | 수정 위치 |
|---|---|---|
| `HandGrip_R` | 손에 든 검 위치 | `SKM_Manny_Simple` Skeleton Tree |
| `SheathSocket_Back` | 등에 찬 검집 / 초기 검 위치 | `SKM_Manny_Simple` Skeleton Tree |

주의:

- 검 방향과 위치는 우선 `HandGrip_R` 소켓에서 조정한다.
- 검집 방향과 위치는 우선 `SheathSocket_Back` 소켓에서 조정한다.
- 검/검집 크기는 `BP_PlayerCharacter`의 `EquippedOtachi`, `EquippedOtachiSheath` Scale에서 조정한다.
- 검/검집 컴포넌트는 적을 물리적으로 밀지 않도록 충돌과 Physics를 꺼둔다.

## 9. 카메라 줌

마우스 휠 줌은 `MouseWheelCameraZoomSubsystem`에서 처리한다.

- 현재 Pawn 내부의 `USpringArmComponent`를 찾아 거리 값을 조절한다.
- 특정 Character 상속 구조에 의존하지 않는 방향을 유지한다.

## 10. 방 전투 조작 기준

- 스테이지 클리어형 방 전투에서도 WASD 이동, 좌클릭 1/2/3타 콤보, Space 회피는 유지한다.
- 방 입장 Trigger, 문, 보상, 다음 방 이동 상호작용은 추후 `E`키 또는 근접 자동 상호작용 후보로 둔다.
- 초기 MVP에서는 새 조작을 늘리기보다 현재 전투 조작을 유지하고 방 흐름을 먼저 붙인다.
- 방 전투 중 문이 닫혀도 플레이어 이동/공격/회피 규칙은 현재 구조를 유지한다.

## 11. 디버그 입력

### L: Debug Element Toggle

- `L`은 플레이어의 런타임 전투 속성을 `None`과 `Lightning` 사이에서 토글한다.
- 이 상태는 `PlayerStatsComponent`의 `EPlayerCombatElement`로 관리한다.
- `L` 토글은 Output Log로만 기록하며, 화면 왼쪽 상단에 지속 표시되는 디버그 텍스트는 사용하지 않는다.
- 현재는 라이트닝 공격 개발을 위한 디버그 입력이다.
- `L` 자체는 보상 카드, Prism 해금, Shock 상태, Chain Lightning 데미지, 최종 Lightning VFX를 적용하지 않는다.
- Run 보상 초기화 시 전투 속성은 `None`으로 돌아간다.

### F10: Debug Slow Motion Toggle

- `F10`은 디버그 슬로모션을 토글한다.
- 첫 입력은 전역 시간 배율을 `0.1x`로 설정한다.
- 두 번째 입력은 전역 시간 배율을 `1.0x`로 복구한다.
- 이 기능은 애니메이션과 전투 타이밍 확인용이며 일반 게임 조작이 아니다.

## 12. Codex 구현 주의사항

- 플레이어 조작 관련 작업 전 이 문서를 먼저 읽는다.
- `BP_PlayerCharacter`가 실제 Pawn인지 확인한다.
- `MyProjectCharacter`에 조작 로직을 넣지 않는다.
- 공격 중 이동 락, 회피 캔슬, 예약된 공격 판정이 서로 충돌하지 않게 확인한다.
- 무기 위치 문제는 BP 컴포넌트 Transform보다 Skeleton 소켓을 먼저 확인한다.
- 디버그 입력은 일반 게임 조작과 섞이지 않게 문서와 코드에서 명확히 표시한다.
