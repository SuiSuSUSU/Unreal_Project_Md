---
Status: ACTIVE
Scope: Attack and Skill
Last Updated: 2026-07-06
Source of Truth: Yes
---

# AttackSkillSystem.md

## 1. 문서 목적

이 문서는 현재 게임의 공격, 스킬, 무기, 빌드 성장 방향을 정리한다.

`BasicAttackSystem.md`가 실제 좌클릭 1/2/3타 콤보 구현 기준을 담당한다면, 이 문서는 그 기본 공격 위에 어떤 스킬 축과 보상 축을 얹을지 정리하는 방향 문서다.

## 2. 현재 방향

- 현재 전투의 중심은 카타나 기반 좌클릭 1/2/3타 콤보다.
- 현재 정식 스킬 슬롯, 액티브 스킬 UI, 스킬 데이터베이스는 구현되어 있지 않다.
- 공격/스킬 성장은 경험치 레벨업보다 Stage Clear 보상 카드 중심으로 확장한다.
- Stage Clear 보상은 현재 Run 동안 전투 방식이 조금씩 바뀌는 느낌을 주는 것이 목표다.
- 첫 속성 빌드 축은 Katana + Lightning이다.
- 모든 속성, 모든 스킬, 모든 보상 등급을 한 번에 만들지 않는다.

## 3. 문서 책임 경계

| 영역 | 기준 문서 |
| --- | --- |
| 좌클릭 1/2/3타 콤보 구현 | `Docs/BasicAttackSystem.md` |
| WASD, 회피, 무기 소켓 조작 | `Docs/PlayerControlSystem.md` |
| 플레이어 공격/회피/체력 수치 | `Docs/PlayerStatsSystem.md` |
| Stage Clear 보상, 보상 타입, 보상 등급 | `Docs/RewardSystem.md` |
| 번개 보상, Prism 조건, 속성 시너지 | `Docs/ElementalRewardSystem.md` |
| 검 궤적, 스파크, 피격 VFX, 라이트닝 오라 | `Docs/VFXSystem.md` |
| 공격 애니메이션 Notify와 타이밍 | `Docs/AnimationSystem.md` |

이 문서는 위 문서들의 내용을 다시 구현 기준처럼 복제하지 않고, 공격/스킬 성장 방향을 연결하는 역할만 한다.

## 4. 현재 구현 상태

현재 구현된 공격/스킬 관련 기능:

| 기능 | 상태 | 기준 |
| --- | --- | --- |
| 좌클릭 1/2/3타 콤보 | 구현됨 | `PlayerMeleeAttackSubsystem` |
| 부채꼴 공격 판정 | 구현됨 | `PlayerMeleeAttackSubsystem` |
| 확정 히트 타이밍 | 구현됨 | `AnimNotify_PlayerMeleeHit`, fallback delay |
| 검 궤적 | 구현됨 | `WeaponTrailComponent`, `AnimNotifyState_WeaponTrail` |
| 지면 스파크 | 구현됨 | `AnimNotify_GroundImpactVfx` |
| 피격 피/Hit Distortion | 구현됨 | 현재는 `PlayerMeleeAttackSubsystem` 안의 임시 연결 |
| Lightning 런 상태 | 구현됨 | L 디버그 토글 또는 `run_lightning_current_slash` 보상, `PlayerStatsComponent` |
| Chain Lightning MVP | 구현됨 | Lightning 상태 + 확정 히트 + 최대 2번 전이 |
| Stage Clear 보상 타이밍 | 구현됨 | `StageFlowManager` MVP fallback |
| 정식 스킬 슬롯/스킬 UI | 미구현 | 후보 |
| RewardDataAsset | 미구현 | 후보 |
| Chain Lightning VFX arc | MVP 구현됨 | `NS_EletricLightning` 기반 임시 arc, 최종 튜닝은 후보 |

## 5. 기본 공격과 스킬의 관계

현재 기본 공격은 플레이어 전투의 중심이다.

기본 공격 기준:

- 좌클릭 수동 공격이다.
- 1/2/3타 콤보로 진행된다.
- 3타는 보상과 연출을 붙이기 좋은 콤보 payoff 지점이다.
- 실제 데미지와 판정은 `BasicAttackSystem.md` 기준을 따른다.

스킬 방향:

- 초기에는 별도의 복잡한 스킬 시스템보다 기본 공격을 강화하는 보상을 먼저 만든다.
- 이후 스킬은 보상으로 얻는 Run-long 전투 규칙, 액티브 스킬, 패시브 효과로 확장할 수 있다.
- 스킬이 생기더라도 기본 공격의 역할을 바로 대체하지 않는다.

## 6. 무기 방향

현재 무기 정체성은 카타나다.

현재 기준:

- 플레이어는 Otachi 계열 카타나를 사용한다.
- 기본 공격 모션과 검 궤적은 카타나 전투 감각을 기준으로 한다.
- `EquippedOtachi`, `EquippedOtachiSheath` 같은 실제 컴포넌트/소켓 조작은 `PlayerControlSystem.md` 기준을 따른다.
- 무기 교체 시스템과 `WeaponDataAsset`은 아직 구현하지 않는다.

장기 방향:

- 무기는 공격 리듬, 범위, 속도, 위험도, 콤보 형태를 바꾸는 큰 축이 될 수 있다.
- 속성은 같은 무기 위에 전투 효과와 시각 효과를 얹는 축으로 본다.
- 첫 검증은 Katana + Lightning으로 시작한다.

## 7. Stage Clear 보상과 공격 성장

공격/스킬 성장은 Stage Clear 보상을 중심으로 한다.

현재 기준:

- Room Clear마다 빌드 보상 카드를 주지 않는다.
- Stage 안의 Room들을 모두 클리어하면 Stage Clear 보상 카드를 제시한다.
- 보상 선택 후 그 효과가 현재 Run에 적용된 상태로 Stage Map으로 돌아간다.
- 현재 `RewardActor` / `RewardSelectionWidget` 흐름은 Stage Clear 보상 UI의 MVP 재사용 경로다.

공격 성장 후보:

- 기본 공격 데미지 증가
- 기본 공격 범위 증가
- 3타 공격 강화
- 3타 검기
- 회피 후 다음 공격 강화
- 적 처치 시 소량 회복
- 공격 적중 시 출혈
- 회피 후 잔상 폭발
- Lightning chain / Shock / dodge overcharge

## 8. 보상 등급과 기술 축

보상 등급은 카드의 희귀도와 효과 강도를 표현하는 방향이다.

현재 후보 등급:

| 등급 | 방향 |
| --- | --- |
| Bronze | 낮은 수치 보정 또는 단순한 시작 효과 |
| Silver | 전투 중 체감되는 보정이나 간단한 조건부 효과 |
| Gold | 특정 빌드 방향을 확실히 만드는 효과 |
| Prism | 매우 희귀하고 전투 방식을 크게 바꾸는 변형 효과 |

기술 축은 특정 전투 방식의 누적 성장 방향이다.

현재 후보 기술 축:

| 기술 축 | 방향 |
| --- | --- |
| 휘두르기 | 기본 공격, 공격 범위, 3타 강화, 검기 |
| 회피 | 회피 쿨타임, 회피 후 다음 공격 강화, 잔상 |
| 피/출혈 | 출혈, 처치 회복, 흡혈, 출혈 전파 |
| 번개 | Chain Lightning, Shock, dodge overcharge, Storm Sword |

주의:

- 기술 레벨 시스템은 아직 구현하지 않는다.
- Prism은 자주 나오면 안 되며, 실제 등장 규칙은 `ElementalRewardSystem.md`와 `RewardSystem.md` 기준을 따른다.
- Luck 스탯과 보상 확률 시스템은 아직 정식 구현 전이다.

## 9. Katana + Lightning 첫 구현 방향

첫 속성 빌드 축은 Katana + Lightning이다.

현재 구현된 MVP:

- L로 `Neutral` / `Lightning` 상태를 디버그 전환할 수 있다.
- Lightning 상태에서 플레이어 기본 공격이 실제 적중하면 Chain Lightning MVP가 실행된다.
- 현재 Chain Lightning은 원래 맞은 적을 제외하고 주변 활성 적에게 최대 2번 전이하며 약한 번개 데미지를 준다.
- 현재 Chain Lightning VFX arc는 임시 MVP로 구현되어 있으며, 최종 튜닝은 아직 남아 있다.

현재 기준:

- MVP Chain Lightning은 “모든 확정 근접 히트”에서 테스트한다.
- 이후 보상 설계에서는 3타, 검기, 회피 후 공격, Shock 같은 조건으로 세분화할 수 있다.
- 최종 번개 보상 목록과 Prism 조건은 `Docs/ElementalRewardSystem.md` 기준을 따른다.

우선 검증 순서:

1. Chain Lightning 데미지와 대상 선택이 재미있는지 확인한다.
2. Chain Lightning VFX arc의 위치, 길이, 폭, 색감, 지속시간을 튜닝한다.
3. Shock 같은 최소 상태 표식을 붙일지 결정한다.
4. 회피 후 번개 강화 보상을 검증한다.
5. Prism급 Storm Sword 후보를 나중에 검증한다.

## 10. 스킬 Trigger 후보

아직 정식 스킬 시스템은 없지만, 장기적으로 아래 Trigger를 후보로 둔다.

| Trigger | 의미 | 현재 판단 |
| --- | --- | --- |
| `Manual` | 플레이어가 키를 눌러 직접 발동 | 후보 |
| `AutoCooldown` | 쿨타임마다 자동 발동 | 후보 |
| `AutoInRange` | 적이 범위 안에 있으면 자동 발동 | 후보 |
| `Passive` | 보유 중 항상 적용 | 후보 |
| `OnHit` | 기본 공격 또는 스킬 적중 시 발동 | 우선 후보 |
| `OnKill` | 적 처치 시 발동 | 후보 |
| `AfterDodge` | 회피 후 다음 행동에 발동 | 우선 후보 |

초기에는 `OnHit`, `AfterDodge`, `Passive`처럼 현재 전투 흐름에 얹기 쉬운 Trigger를 우선 검토한다.

## 11. 스킬 Effect 후보

| Effect | 의미 | 현재 판단 |
| --- | --- | --- |
| `InstantDamage` | 즉시 데미지 | 후보 |
| `AreaDamage` | 범위 데미지 | 후보 |
| `Projectile` | 투사체 발사 | 후보 |
| `DamageOverTime` | 일정 시간 반복 데미지 | 후보 |
| `ChainDamage` | 주변 적에게 연쇄 피해 | Lightning 우선 후보 |
| `Buff` | 플레이어 능력치 강화 | 후보 |
| `Debuff` | 적 약화 또는 상태 부여 | 후보 |
| `Summon` | 소환물 생성 | 후순위 |

현재는 `ChainDamage`와 기본 공격 강화 계열을 먼저 검증한다.

## 12. 구현하지 않는 것

현재 단계에서 구현하지 않는 것:

- 전체 스킬 슬롯 UI
- 수십 개 스킬 데이터베이스
- 스킬 합성 시스템
- 무기 장비/교체 시스템
- 복잡한 속성 상성표
- 모든 속성 동시 구현
- Prism 보상 풀 전체 구현
- Luck 기반 보상 확률 시스템
- 저장/로드가 필요한 영구 스킬 강화
- 자동 공격 중심 뱀서류 전투로의 회귀

## 13. Codex 구현 주의

- 기본 공격 구현 변경은 먼저 `Docs/BasicAttackSystem.md`를 확인한다.
- 보상 타입, 등급, Stage Clear 보상은 `Docs/RewardSystem.md`를 확인한다.
- Lightning, Prism, 속성 시너지는 `Docs/ElementalRewardSystem.md`를 확인한다.
- VFX나 검 궤적은 `Docs/VFXSystem.md`와 `Docs/AnimationSystem.md`를 확인한다.
- `PlayerStatsDataAsset` 원본 값을 런타임 보상으로 직접 수정하지 않는다.
- 정식 스킬 시스템을 한 번에 만들지 말고, 현재 기본 공격에 붙는 작은 보상 효과부터 검증한다.
- 구현된 것과 기획 후보를 문서에서 명확히 구분한다.

## 14. 다음 후보

가까운 구현 후보:

- Chain Lightning VFX arc 최종 튜닝
- Chain Lightning 데미지/범위/대상 수 튜닝
- 3타 강화 보상 1개 구현
- 회피 후 다음 공격 강화 후보 검증
- `WeaponImpactVfxComponent`로 피격 VFX 분리

문서 정리 후보:

- `RewardSystem.md`와 `ElementalRewardSystem.md`의 보상 등급/Prism 중복 정리
- `EnemySpawnSystem.md` 인코딩 복구
- `AttackSkillSystem.md`와 `BasicAttackSystem.md` 사이의 중복 지속 점검

## 15. 관련 문서

- `Docs/BasicAttackSystem.md`
- `Docs/PlayerControlSystem.md`
- `Docs/PlayerStatsSystem.md`
- `Docs/RewardSystem.md`
- `Docs/ElementalRewardSystem.md`
- `Docs/VFXSystem.md`
- `Docs/AnimationSystem.md`
- `Docs/StageRunStructure.md`
- `Docs/CurrentImplementation.md`
- `Docs/10_ProjectClassMap.md`
