# 프로젝트 코드 구조 요약

> 이 문서는 프로젝트 코드의 현재 구조와 책임을 빠르게 파악하기 위한 기준 문서다.
>
> **유지보수 규칙:** 프로젝트 코드를 생성·수정·삭제할 때는 같은 작업에서 이 문서도 반드시 함께 갱신한다. 코드와 문서가 다르면 실제 코드를 기준으로 문서를 바로잡는다.

- 마지막 갱신일: 2026-08-07
- 현재 요약 대상: `RootDesk/MyDesk/Minigame/_GameLogic.mlua`, `RootDesk/MyDesk/Minigame/MinigameComponent.mlua`
- 현재 구현 단계: 미니게임 프레임워크의 최상위 공통 상태와 미니게임 기반 인터페이스 정의

## 1. 구현 파일 현황

| 파일 | 타입 | 역할 |
|---|---|---|
| `RootDesk/MyDesk/Minigame/_GameLogic.mlua` | `@Logic` | 선택된 미니게임, 플레이 모드, 로컬 점수, 제한시간, 플레이어 및 랭킹 정보를 관리하는 전역 Logic |
| `RootDesk/MyDesk/Minigame/MinigameComponent.mlua` | `@Component` | 개별 미니게임이 공통으로 사용할 초기화·갱신·해제 인터페이스를 정의하는 기반 Component |

현재 미니게임 프레임워크에는 `_GameLogic`과 기반 `MinigameComponent`가 존재한다. 두 클래스 사이를 연결할 `_MinigameManager`와 `MinigameRegistry`는 아직 구현되지 않았다.

## 2. `_GameLogic` 구조

```lua
@Logic
script _GameLogic extends Logic
```

`_GameLogic`은 특정 맵이나 엔티티에 종속되지 않고 게임 세션 전반에서 공유할 미니게임 상태를 관리한다. 구체적인 미니게임 규칙을 처리하기보다는 상위 상태와 시작·종료 진입점을 제공한다.

### Property

| 이름 | 타입 | 기본값 | 책임 |
|---|---|---:|---|
| `CurrentGameId` | `string` | `""` | 현재 선택된 미니게임 ID. 빈 문자열은 선택되지 않은 상태를 뜻한다. |
| `PlayMode` | `string` | `"SINGLE"` | 현재 플레이 모드. 유효한 값은 `SINGLE`, `MULTI`다. |
| `CurrentScore` | `number` | `0` | 현재 로컬 플레이어의 미니게임 점수다. |
| `CurrentMaxTime` | `number` | `60` | 현재 미니게임 한 판의 최대 제한시간(초)이다. |
| `MyPlayerId` | `string` | `""` | 현재 로컬 플레이어 ID다. |
| `OpponentPlayerId` | `string` | `""` | 멀티플레이 상대 ID다. 싱글플레이나 매칭 전에는 빈 문자열이다. |
| `MyRankingData` | `table` | `{}` | 내 랭킹 정보를 저장할 테이블이다. 내부 스키마는 아직 정의하지 않았다. |

### Method

| 메서드 | 현재 동작 |
|---|---|
| `SelectMinigame(string gameId)` | 빈 ID면 warning 후 기존 값을 유지한다. 정상 ID면 `CurrentGameId`를 변경한다. |
| `SetPlayMode(string playMode)` | `SINGLE` 또는 `MULTI`만 허용한다. 다른 값은 warning 후 무시한다. |
| `SetCurrentScore(number score)` | `CurrentScore`를 전달받은 값으로 설정한다. |
| `AddScore(number score)` | `CurrentScore`에 전달받은 값을 더한다. 점수 산정 규칙은 포함하지 않는다. |
| `ResetScore()` | `CurrentScore`를 `0`으로 초기화한다. |
| `SetCurrentMaxTime(number maxTime)` | `0`보다 큰 값만 허용한다. 잘못된 값은 warning 후 무시한다. |
| `SetOpponentPlayer(string playerId)` | 빈 ID면 warning 후 기존 값을 유지하고, 정상 ID면 상대 ID를 설정한다. |
| `ClearOpponentPlayer()` | `OpponentPlayerId`를 빈 문자열로 초기화한다. |
| `StartMinigame()` | 선택된 미니게임이 없으면 warning 후 종료한다. 정상 상태에서는 향후 `_MinigameManager`를 호출할 TODO만 존재한다. 점수는 임의로 초기화하지 않는다. |
| `EndMinigame()` | 향후 `_MinigameManager`에 종료를 요청할 TODO만 존재한다. 현재 상태값은 임의로 초기화하지 않는다. |
| `OnUpdate(number delta)` | `@ExecSpace("ClientOnly")` lifecycle 메서드다. 현재는 실행 코드 없이 향후 Manager 갱신 및 제한시간 비교를 위한 주석만 존재한다. |

## 3. `MinigameComponent` 구조

```lua
@Component
script MinigameComponent extends Component
```

`MinigameComponent`는 구체적인 미니게임들이 공통으로 따라야 할 최소 실행 인터페이스를 정의한다. 현재는 Property가 없으며, 세 Method 모두 시그니처와 설명만 존재하는 빈 기반 구현이다.

### Property

현재 정의된 Property는 없다.

### Method

| 메서드 | 의도된 책임 | 현재 구현 상태 |
|---|---|---|
| `Initialize(table gameData)` | 미니게임 시작 전에 필요한 데이터와 상태를 초기화한다. | 빈 메서드 |
| `Update(number deltaTime)` | 실행 중인 미니게임의 프레임 단위 로직을 갱신한다. | 빈 메서드 |
| `Release()` | 미니게임 종료 시 사용한 리소스와 상태를 정리한다. | 빈 메서드 |

세 메서드에는 별도의 `@ExecSpace`가 지정되어 있지 않다. 향후 구체적인 미니게임 Component가 이 인터페이스를 확장하고, `_MinigameManager`가 현재 게임에 맞춰 호출하는 구조를 전제로 한다.

## 4. 책임 경계

### `_GameLogic`이 담당하는 것

- 현재 미니게임 ID 보관 및 변경
- `SINGLE` / `MULTI` 플레이 모드 보관 및 검증
- 로컬 플레이어 점수 보관·설정·가산·초기화
- 한 판의 최대 제한시간 보관 및 검증
- 내 플레이어 ID, 상대 플레이어 ID, 내 랭킹 데이터 보관
- 미니게임 시작·종료를 요청하는 최상위 인터페이스 제공

### `_GameLogic`이 담당하지 않는 것

- 게임별 점수 발생 조건과 점수 계산
- 현재 한 판의 진행시간 `CurrentTime` 보관
- 구체적인 미니게임 초기화·갱신·해제
- Registry 조회 및 MinigameComponent 생명주기 관리
- UI 입력 처리와 화면 표시
- 서버 통신, 점수 동기화, 매칭 및 랭킹 계산

### `MinigameComponent`가 담당하는 것

- 미니게임 시작 전 초기화 진입점 제공
- 실행 중 프레임 갱신 진입점 제공
- 미니게임 종료 시 정리 진입점 제공
- 향후 구체적인 미니게임이 구현해야 할 공통 실행 형태 정의

### 현재 `MinigameComponent`가 담당하지 않는 것

- 실제 미니게임 규칙과 점수 계산
- 현재 실행 중인 미니게임 선택 및 교체
- `CurrentTime` 누적과 제한시간 종료 판정
- `_GameLogic` 또는 `_MinigameManager`와의 실제 연결

점수 처리의 의도된 흐름은 다음과 같다.

```text
UI 입력
  → MinigameComponent가 게임 규칙과 획득 점수를 판단
  → _GameLogic.AddScore(score)
  → _GameLogic.CurrentScore에 결과 저장
```

## 5. 향후 확장 방향

```text
_GameLogic
  → _MinigameManager
    → MinigameRegistry
      → MinigameComponent
```

- `_MinigameManager`는 현재 실행 중인 미니게임과 `CurrentTime`을 관리할 예정이다.
- `_GameLogic.OnUpdate(delta)`는 Manager의 갱신을 호출하고 `Manager.CurrentTime`과 `CurrentMaxTime`을 비교하는 역할로 확장될 예정이다.
- 시간 제한에 도달하면 `_GameLogic.EndMinigame()`을 호출하는 흐름을 구성할 예정이다.
- `MinigameComponent` 기반 인터페이스는 생성되었지만, `_MinigameManager`, Registry 및 구체적인 미니게임과의 연결은 구현되지 않았다.

## 6. 현재 불변 조건 및 주의사항

- `PlayMode`는 문자열 `SINGLE` 또는 `MULTI`만 사용한다.
- `CurrentMaxTime`은 `0`보다 커야 하며 기본값은 `60`초다.
- `CurrentScore`는 `_GameLogic`이 저장하지만, 획득 점수 계산은 각 미니게임이 담당한다.
- `CurrentTime` Property는 `_GameLogic`에 존재하지 않는다.
- `StartMinigame()`과 `EndMinigame()`은 아직 Manager를 실제로 호출하지 않는다.
- `MyRankingData`의 내부 스키마는 아직 확정하지 않는다.
- 현재 네트워크 동기화 및 서버 권한 처리는 구현하지 않았다.
- `MinigameComponent`에는 Property가 없으며 세 Method는 모두 빈 플레이스홀더다.
- `MinigameComponent.Update()`는 엔진 lifecycle `OnUpdate()`가 아니라 향후 Manager가 명시적으로 호출할 일반 Method다.

## 7. 코드 변경 시 문서 동기화 체크리스트

프로젝트 코드를 변경할 때 다음 항목을 확인하고 이 문서를 함께 수정한다.

- 새 파일이나 클래스를 추가·삭제했는가?
- Property의 이름, 타입, 기본값 또는 책임이 바뀌었는가?
- Method의 시그니처, 검증 조건 또는 상태 변경 방식이 바뀌었는가?
- `ExecSpace`, lifecycle 또는 서버·클라이언트 책임이 바뀌었는가?
- 클래스 사이의 호출 흐름이나 의존성이 추가되었는가?
- TODO가 구현되었거나 새로운 TODO가 생겼는가?
- 아직 구현하지 않은 기능과 책임 경계가 달라졌는가?
- 마지막 갱신일을 현재 작업일로 변경했는가?
