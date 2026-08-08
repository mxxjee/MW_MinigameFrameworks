# 프로젝트 코드 구조 요약

> 이 문서는 프로젝트 코드의 현재 구조와 책임을 빠르게 파악하기 위한 기준 문서다.
>
> **유지보수 규칙:** 프로젝트 코드를 생성·수정·삭제할 때는 같은 작업에서 이 문서도 반드시 함께 갱신한다. 코드와 문서가 다르면 실제 코드를 기준으로 문서를 바로잡는다.

- 마지막 갱신일: 2026-08-08
- 현재 활성 요약 대상: `RootDesk/MyDesk/_GameLogic.mlua`, `RootDesk/MyDesk/Minigame/MinigameComponent.mlua`, `RootDesk/MyDesk/Minigame/MinigameData.mlua`, `RootDesk/MyDesk/Minigame/MinigameRegistry.mlua`, `RootDesk/MyDesk/Minigame/MinigameManager.mlua`
- 이전 경로: `RootDesk/MyDesk/Minigame/_GameLogic.mlua` (삭제 상태)
- 현재 구현 단계: 미니게임 최상위 공통 상태, 중앙 Registry, 로컬 실행 Manager 및 기반 Component 실행 흐름 연결

## 1. 구현 파일 현황

| 파일 | 타입 | 현재 상태 | 역할 |
|---|---|---|---|
| `RootDesk/MyDesk/_GameLogic.mlua` | `@Logic` | 존재, Git 추적 중 | 선택된 미니게임, 플레이 모드, 로컬 점수, 제한시간, 플레이어 및 랭킹 정보를 관리하는 전역 Logic |
| `RootDesk/MyDesk/Minigame/MinigameComponent.mlua` | `@Component` | 존재, 구현 전 | 개별 미니게임이 공통으로 사용할 초기화·갱신·해제 인터페이스를 정의하는 기반 Component |
| `RootDesk/MyDesk/Minigame/MinigameData.mlua` | `@Struct` | 구현 완료 | 미니게임 이름·ID와 실행 `Component` 참조를 한 단위로 보관하는 데이터 구조 |
| `RootDesk/MyDesk/Minigame/MinigameRegistry.mlua` | `@Component` | 구현 완료, 엔티티 부착 필요 | ID별 `MinigameData`를 중앙 등록·조회하고 `OnBeginPlay()`에서 Manager에 자기 Component를 정확한 타입으로 연결하는 Registry Component |
| `RootDesk/MyDesk/Minigame/MinigameManager.mlua` | `@Logic` | 신규 구현 | 현재 로컬 미니게임, 경과시간, 실행 여부를 관리하고 Component lifecycle을 호출하는 전역 Logic |

현재 활성 소스에는 `_GameLogic`, 기반 `MinigameComponent`, `MinigameData`, `MinigameRegistry`, `MinigameManager`가 존재한다. 공통 실행 흐름은 연결되었고 구체적인 미니게임 등록·구현은 아직 남아 있다.

## 2. `_GameLogic` 구조

> **현재 경로:** `RootDesk/MyDesk/_GameLogic.mlua`. 이전 `RootDesk/MyDesk/Minigame/_GameLogic.mlua` 경로에서는 삭제되었으며, 클래스 구조는 기존과 동일하다.

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
| `SelectMinigame(string gameId)` | 빈 ID면 warning 후 `false`를 반환하고 기존 값을 유지한다. 정상 ID면 `CurrentGameId`를 변경하고 `true`를 반환한다. |
| `SetPlayMode(string playMode)` | `SINGLE` 또는 `MULTI`만 허용한다. 다른 값은 warning 후 무시한다. |
| `SetCurrentScore(number score)` | `CurrentScore`를 전달받은 값으로 설정한다. |
| `AddScore(number score)` | `CurrentScore`에 전달받은 값을 더한다. 점수 산정 규칙은 포함하지 않는다. |
| `ResetScore()` | `CurrentScore`를 `0`으로 초기화한다. |
| `SetCurrentMaxTime(number maxTime)` | `0`보다 큰 값만 허용한다. 잘못된 값은 warning 후 무시한다. |
| `SetOpponentPlayer(string playerId)` | 빈 ID면 warning 후 기존 값을 유지하고, 정상 ID면 상대 ID를 설정한다. |
| `ClearOpponentPlayer()` | `OpponentPlayerId`를 빈 문자열로 초기화한다. |
| `StartMinigame(string gameId)` | `@ExecSpace("Client")`. `SelectMinigame()`이 실패하면 종료하고, 성공하면 `_MinigameManager:StartGame(CurrentGameId)`를 호출한다. 점수는 임의로 초기화하지 않는다. |
| `EndMinigame()` | `_MinigameManager:EndGame()`만 호출한다. `_GameLogic`의 ID·점수·모드 상태는 초기화하지 않는다. |
| `OnUpdate(number delta)` | `@ExecSpace("ClientOnly")`. Manager의 `Update(delta)`를 호출하고, 플레이 중 `Manager.CurrentTime >= CurrentMaxTime`이면 `EndMinigame()`을 호출한다. `_GameLogic` 자체는 시간을 누적하지 않는다. |

## 3. `MinigameComponent` 구조

```lua
@Component
script MinigameComponent extends Component
```

`MinigameComponent`는 구체적인 미니게임들이 공통으로 따라야 할 최소 실행 인터페이스를 정의한다. 현재는 Property가 없으며, 세 Method 모두 시그니처만 존재하는 빈 기반 구현이다.

### Property

현재 정의된 Property는 없다.

### Method

| 메서드 | 의도된 책임 | 현재 구현 상태 |
|---|---|---|
| `Initialize(MinigameData gameData)` | 미니게임 시작 전에 Registry가 조회한 미니게임 데이터와 상태를 초기화한다. | 빈 메서드 |
| `Update(number deltaTime)` | 실행 중인 미니게임의 프레임 단위 로직을 갱신한다. | 빈 메서드 |
| `Release()` | 미니게임 종료 시 사용한 리소스와 상태를 정리한다. | 빈 메서드 |

세 메서드에는 설명 주석이나 별도의 `@ExecSpace`가 지정되어 있지 않다. 향후 구체적인 미니게임 Component가 이 인터페이스를 확장하고, `_MinigameManager`가 현재 게임에 맞춰 호출하는 구조를 전제로 한다.

## 4. `MinigameData` 구조

```lua
@Struct
script MinigameData
```

`MinigameData`는 미니게임 하나의 식별 정보와 실행 Component 참조를 함께 보관하는 데이터 구조다.

### Property

| 이름 | 타입 | 기본값 | 책임 |
|---|---|---:|---|
| `Name` | `string` | `""` | 미니게임 표시 이름을 저장한다. |
| `ID` | `string` | `""` | 미니게임을 구분하는 ID를 저장한다. |
| `GameComponent` | `MinigameComponent` | `nil` | 실행 Component를 사용자 정의 정적 타입으로 직접 저장한다. |

### Method

| 메서드 | 현재 동작 |
|---|---|
| `InitData(string _name, string _id, MinigameComponent _GameComponent)` | 전달받은 이름·ID·Component 참조를 각각 저장한다. |

현재 `GameEntity` Property는 존재하지 않는다. `InitData()` 안에 `GameComponent.Entity`를 저장하려던 코드가 주석으로만 남아 있다.

Entity가 필요하면 별도 저장 없이 `gameData.GameComponent.Entity`로 접근한다.

## 5. `MinigameRegistry` 구조

```lua
@Component
script MinigameRegistry extends Component
```

`MinigameRegistry`는 모든 미니게임의 등록 정보를 ID 기준으로 중앙 관리하는 Component다. 게임 실행이나 lifecycle 처리는 하지 않고 등록·검증·조회만 담당한다.

전역 Logic singleton이 아니므로 사용하려면 모델 또는 맵의 엔티티에 이 Component를 부착해야 한다. `OnBeginPlay()` 실행과 Registry 상태의 수명은 부착된 엔티티의 생명주기를 따른다. 현재 작업에서는 모델·맵 부착까지 수행하지 않았다.

### Property

| 이름 | 타입 | 기본값 | 책임 |
|---|---|---:|---|
| `MinigameComponents` | `SyncTable<string, MinigameComponent>` | 엔진이 빈 컬렉션으로 초기화 | 에디터에서 미니게임 ID와 실제 Component 참조를 중앙 입력하는 용도다. `@Sync`는 사용하지 않는다. |
| `MinigameDatas` | `table` | `{}` | 실제 Registry 조회용 저장소다. `MinigameDatas[id] = MinigameData` 형태로 저장한다. |

실제 선언은 다음과 같다.

```lua
property SyncTable<string, MinigameComponent> MinigameComponents
property table MinigameDatas = {}
```

`MinigameComponents`는 에디터 Component Reference 입력용이고, `MinigameDatas`는 실행 중 ID 조회에 사용하는 Registry 저장소다. 두 곳의 `GameComponent`는 복제본이 아니라 동일한 Component 참조다.

### Lifecycle 및 Method

| 메서드 | 현재 동작 |
|---|---|
| `OnBeginPlay()` | `@ExecSpace("ClientOnly")`. 모든 게임의 중앙 등록 위치이며, 등록 처리 뒤 `_MinigameManager:SetRegistry(self)`로 정확한 `MinigameRegistry` 참조를 Manager에 연결한다. 현재 구체적인 등록 호출은 TODO다. |
| `RegisterMinigame(string name, string id, MinigameComponent gameComponent)` | 등록 데이터를 검증하고 `MinigameData`를 생성·초기화한 뒤 `MinigameDatas[id]`에 저장한다. 성공 시 `true`, 실패 시 `false`를 반환한다. |
| `ValidateRegisterData(string id, MinigameComponent gameComponent)` | 빈 ID, nil `GameComponent`, 중복 ID만 검사한다. 실패 원인을 warning으로 기록한다. |
| `GetMinigameData(string id)` | `MinigameDatas[id]`를 직접 조회해 `MinigameData` 또는 `nil`을 반환한다. 전체 table을 순회하지 않는다. |
| `GetMinigameComponent(string id)` | `GetMinigameData(id)` 결과에서 `GameComponent`를 반환하며, 등록되지 않은 ID면 `nil`을 반환한다. |

등록 흐름은 다음과 같다.

```text
OnBeginPlay
  → MinigameComponents에서 ID별 Component 참조 확인
  → RegisterMinigame(name, id, component)
  → ValidateRegisterData(id, component)
  → MinigameData 생성 및 InitData(...)
  → MinigameDatas[id]에 저장
  → _MinigameManager:SetRegistry(self)
```

현재 `OnBeginPlay()`에는 등록할 구체적인 미니게임이 없으므로 예시 ID나 가상 Component를 임의로 등록하지 않는다. 현재 `map01` 플레이 검증에서는 Registry Component가 부착된 활성 Entity가 없어 `MinigameRegistryComp`가 `nil`로 확인되었다. 실제 사용 전 대상 Entity 부착이 필요하다.

## 6. `MinigameManager` 구조

```lua
@Logic
script MinigameManager extends Logic
```

파일과 스크립트 이름은 `MinigameManager`이며 MSW 전역 Logic 접근자는 `_MinigameManager`다. `_MinigameManager.mlua` / `script _MinigameManager` 조합은 전역 `__MinigameManager`를 만들기 때문에 요청된 `_MinigameManager` 호출과 맞지 않아 사용하지 않는다.

이 Logic은 로컬 클라이언트의 현재 미니게임 실행 상태만 관리한다. 서버 권한 상태, 매칭, 네트워크, UI, 점수, 최대 제한시간, 플레이 모드, 게임 ID는 소유하지 않는다.

### Property

| 이름 | 타입 | 기본값 | 책임 |
|---|---|---:|---|
| `MinigameRegistryComp` | `MinigameRegistry` | `nil` | Registry Component를 사용자 정의 정적 타입으로 직접 보관한다. 외부 게임 상태가 아니다. |
| `CurrentGame` | `MinigameComponent` | `nil` | 현재 미니게임을 사용자 정의 정적 타입으로 직접 보관한다. |
| `CurrentTime` | `number` | `0` | 현재 로컬 미니게임의 누적 경과시간이다. |
| `IsPlaying` | `boolean` | `false` | 로컬 미니게임 실행 여부다. |

### Method

| 메서드 | 실행 공간 | 현재 동작 |
|---|---|---|
| `SetRegistry(MinigameRegistry Comp)` | `ClientOnly` | 정확한 `MinigameRegistry` 타입으로 전달받은 Component를 `MinigameRegistryComp`에 저장한다. |
| `StartGame(string gameId)` | `ClientOnly` | `MinigameRegistryComp`를 확인하고 `MinigameData`를 ID로 조회한다. `GameComponent`까지 유효하면 상태를 시작값으로 바꾸고 `Initialize(gameData)`를 호출한다. 실패 시 warning과 `false`, 성공 시 `true`를 반환한다. |
| `Update(number deltaTime)` | `ClientOnly` | 플레이 중이고 `CurrentGame`이 있을 때만 `CurrentTime`을 누적하고 `CurrentGame:Update(deltaTime)`를 호출한다. 엔진 lifecycle `OnUpdate()`가 아닌 일반 Method다. |
| `EndGame()` | `ClientOnly` | 현재 Component가 있으면 `Release()`를 호출한 뒤 `CurrentGame=nil`, `CurrentTime=0`, `IsPlaying=false`로 정리한다. |

`StartGame()`의 조회·시작 흐름은 다음과 같다.

```text
MinigameRegistryComp 유효성 확인
  → MinigameRegistryComp:GetMinigameData(gameId)
  → gameData.GameComponent 유효성 확인
  → CurrentGame 설정 / CurrentTime=0 / IsPlaying=true
  → CurrentGame:Initialize(gameData)
```

## 7. 책임 경계

### `_GameLogic`이 담당하는 것

- 현재 미니게임 ID 보관 및 변경
- `SINGLE` / `MULTI` 플레이 모드 보관 및 검증
- 로컬 플레이어 점수 보관·설정·가산·초기화
- 한 판의 최대 제한시간 보관 및 검증
- 내 플레이어 ID, 상대 플레이어 ID, 내 랭킹 데이터 보관
- 미니게임 시작·종료를 요청하는 최상위 인터페이스 제공
- `CurrentMaxTime`과 Manager의 `CurrentTime`을 비교한 제한시간 종료 판정

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
- Registry 조회와 실행 상태 보관

### `MinigameData`가 담당하는 것

- 미니게임 표시 이름 보관
- 미니게임 ID 보관
- ID에 대응하는 `MinigameComponent` 참조 보관
- `InitData()`를 통한 세 값의 일괄 초기화

### `MinigameData`가 담당하지 않는 것

- 미니게임 실행 및 lifecycle 호출
- 미니게임 검색·등록·중복 ID 검증
- `GameComponent` 또는 Entity 유효성 검사
- 점수, 진행시간 및 플레이 모드 관리

### `MinigameRegistry`가 담당하는 것

- 에디터에서 미니게임 ID별 `MinigameComponent` 참조 보관
- `OnBeginPlay()` 한 곳에서 모든 미니게임을 중앙 등록
- `MinigameData` 생성과 ID별 저장
- 빈 ID, nil Component, 중복 ID 등록 검증
- ID를 통한 `MinigameData` 및 `MinigameComponent` 조회

### `MinigameRegistry`가 담당하지 않는 것

- `MinigameComponent.Initialize()`, `Update()`, `Release()` 호출
- `CurrentTime`, `CurrentScore`, `CurrentMaxTime`, `PlayMode`, `IsPlaying` 관리
- 게임 시작·종료와 구체적인 미니게임 규칙
- UI, Network, Matching 처리

### `MinigameManager`가 담당하는 것

- 클라이언트 로컬의 현재 실행 Component 보관
- 정확한 `MinigameRegistry` Component 참조 보관과 Registry 조회
- `MinigameData`를 통한 `MinigameComponent` 선택
- 현재 실행시간 누적과 `Initialize()`, `Update()`, `Release()` 호출
- 실행 시작·종료 시 `CurrentGame`, `CurrentTime`, `IsPlaying` 정리

### `MinigameManager`가 담당하지 않는 것

- 서버 권한 상태와 서버 측 미니게임 lifecycle
- 미니게임 ID, 점수, 최대 제한시간, 플레이 모드 보관
- 제한시간 비교와 종료 여부 결정
- 매칭, 네트워크, 랭킹, UI 처리

점수 처리의 의도된 흐름은 다음과 같다.

```text
UI 입력
  → MinigameComponent가 게임 규칙과 획득 점수를 판단
  → _GameLogic.AddScore(score)
  → _GameLogic.CurrentScore에 결과 저장
```

## 8. 실행 흐름 및 향후 확장 방향

```text
_GameLogic
  → _MinigameManager
    → MinigameRegistry
      → MinigameData
        → MinigameComponent
```

- `_GameLogic.StartMinigame(gameId)`는 ID 선택 후 `_MinigameManager.StartGame()`에 로컬 실행 시작을 위임한다.
- `_MinigameManager.StartGame()`은 Registry에서 `MinigameData`를 찾고 해당 Component의 `Initialize(gameData)`를 호출한다.
- `_GameLogic.OnUpdate(delta)`는 `_MinigameManager.Update(delta)`를 호출하고 Manager의 경과시간과 `CurrentMaxTime`을 비교한다.
- 시간 제한에 도달하면 `_GameLogic.EndMinigame()`이 `_MinigameManager.EndGame()`을 호출하고 Manager가 `Release()`와 로컬 실행 상태 정리를 수행한다.
- `MinigameData`는 Registry가 미니게임 ID와 실행 Component를 연결하는 기본 데이터 단위다.
- `MinigameRegistry.OnBeginPlay()`는 중앙 등록 이후 자신(`MinigameRegistry`)을 Manager에 직접 연결하지만, 현재 구체적인 등록 목록과 실제 Entity 부착은 비어 있다.
- 다음 단계에서는 Registry Component를 활성 Entity에 부착하고 구체적인 미니게임 Component와 `RegisterMinigame()` 호출을 추가해야 한다.
- `_GameLogic`은 새 경로에서 활성화되어 있으며 현재 Git에서 정상 추적 중이다.

## 9. 현재 불변 조건 및 주의사항

- `PlayMode`는 문자열 `SINGLE` 또는 `MULTI`만 사용한다.
- `CurrentMaxTime`은 `0`보다 커야 하며 기본값은 `60`초다.
- `CurrentScore`는 `_GameLogic`이 저장하지만, 획득 점수 계산은 각 미니게임이 담당한다.
- `CurrentTime` Property는 `_GameLogic`에 존재하지 않는다.
- `StartMinigame()`과 `EndMinigame()`은 `_MinigameManager`에 실제 시작·종료를 위임한다.
- `MyRankingData`의 내부 스키마는 아직 확정하지 않는다.
- 현재 네트워크 동기화 및 서버 권한 처리는 구현하지 않았다.
- `MinigameComponent`에는 Property가 없으며 세 Method는 모두 빈 플레이스홀더다.
- `MinigameComponent.Initialize()`의 최종 시그니처는 `Initialize(MinigameData gameData)`다.
- `MinigameComponent.Update()`는 엔진 lifecycle `OnUpdate()`가 아니라 향후 Manager가 명시적으로 호출할 일반 Method다.
- `MinigameData.GameComponent`와 `MinigameManager.CurrentGame`의 선언 타입은 모두 `MinigameComponent`다. `any` 또는 기본 `Component`로 낮추지 않는다.
- `MinigameData.GameComponent`의 기본값은 `nil`이며 Manager 시작 시 유효성을 검사한다.
- `MinigameData.InitData()`는 값 저장만 수행하고 등록이나 실행은 담당하지 않는다.
- `MinigameRegistry.MinigameComponents`의 최종 타입은 `SyncTable<string, MinigameComponent>`이며 `@Sync`와 기본값 리터럴을 사용하지 않는다.
- `MinigameRegistry.MinigameDatas`만 실제 ID 조회용 Registry 저장소로 사용한다.
- `ValidateRegisterData()`는 빈 ID, nil Component, 중복 ID 외의 검증을 추가하지 않는다.
- 모든 실제 미니게임 등록은 `MinigameRegistry.OnBeginPlay()` 한 곳에 모은다.
- 현재 `OnBeginPlay()`에는 구체적인 미니게임 등록 코드가 없고 TODO만 존재한다.
- `MinigameRegistry`는 `@Component`이므로 전역 `_MinigameRegistry` singleton으로 접근하지 않는다.
- Registry를 사용하려면 대상 엔티티에 Component를 부착해야 하며, 활성화되면 `OnBeginPlay()`가 Manager에 `MinigameRegistry` Component 자체를 연결한다.
- `MinigameManager`의 파일명과 스크립트명은 `MinigameManager`이고 전역 접근자는 `_MinigameManager`다.
- `MinigameManager.MinigameRegistryComp`의 선언 타입은 `MinigameRegistry`이며 서버 상태가 아니다.
- `MinigameManager`의 모든 공개 Method는 `ClientOnly`이고 서버 상태·매칭·네트워크를 관리하지 않는다.
- 현재 `map01` 플레이 검증에서는 Registry Component가 부착되지 않아 `MinigameRegistryComp == nil`이다.
- `_GameLogic.mlua`의 활성 경로는 `RootDesk/MyDesk/_GameLogic.mlua`다.
- 이전 `RootDesk/MyDesk/Minigame/_GameLogic.mlua`는 삭제 상태이며, 새 경로의 파일은 현재 Git에서 정상 추적 중이다.

## 10. 정적 타입 및 Problems 검사 기준

- 현재 프로젝트의 실행 코드(`RootDesk/**/*.mlua`)에는 `GetComponent()` 호출이 0개다. 검색된 `GetComponent()`는 `Environment/NativeScripts`의 API 선언 3개뿐이며 실행 호출이 아니다.
- Registry 연결은 `GetComponent("script.MinigameRegistry")` 결과를 `Component`로 낮추는 방식이 아니라 `SetRegistry(MinigameRegistry Comp)`로 정확한 사용자 정의 Component를 직접 전달한다.
- `MinigameManager.MinigameRegistryComp`의 정적 타입은 `MinigameRegistry`이며 `GetMinigameData()`의 소유 타입과 일치한다.
- `MinigameManager.CurrentGame`과 `MinigameData.GameComponent`의 정적 타입은 `MinigameComponent`이며 `Initialize()`, `Update()`, `Release()`의 소유 타입과 일치한다.
- `MinigameData.GameComponent`를 `Component`로 선언했을 때 Manager 대입과 Registry 반환에서 각각 `'MinigameComponent' type is required, but 'Component' type was used.` 오류가 발생했다. 현재는 Property와 Method Parameter 모두 사용자 정의 타입을 선언부에 직접 명시한다.
- `gameData`는 `GetMinigameData()`의 `MinigameData` 반환 타입을 그대로 추론한다. 불필요한 `---@type MinigameData` local Annotation은 Maker LEA-1118을 발생시켜 제거했다.
- 향후 `GetComponent("script.X")`로 사용자 정의 Component를 획득하고 Custom Method를 호출할 때는 local에 `---@type X`를 지정한다. `---@type Component`로 낮춘 뒤 Custom Method를 호출하는 코드는 허용하지 않는다.
- 번들 mLua LSP `1.1.4`의 `diagnose` 명령으로 관련 5개 파일을 검사한 결과 diagnostic/error/warning이 모두 0건이었다.
- 직접 정적 타입으로 다시 변경한 이후 Maker Build Console은 이번 문서 갱신 작업에서 재검증하지 않았다. 기존 `Monster.OnBeginPlay`의 Entity 관련 Info 1건은 별도 문제다.
- Codex 환경에서는 사용자의 VS Code Problems 패널 자체를 직접 열어 읽지 못했다. 따라서 이 문서는 번들 mLua LSP 진단 결과를 기록하며, “VS Code Problems 패널 오류 0건”이라고 단정하지 않는다.

## 11. 코드 변경 시 문서 동기화 체크리스트

프로젝트 코드를 변경할 때 다음 항목을 확인하고 이 문서를 함께 수정한다.

- 새 파일이나 클래스를 추가·삭제했는가?
- Property의 이름, 타입, 기본값 또는 책임이 바뀌었는가?
- Method의 시그니처, 검증 조건 또는 상태 변경 방식이 바뀌었는가?
- `ExecSpace`, lifecycle 또는 서버·클라이언트 책임이 바뀌었는가?
- 클래스 사이의 호출 흐름이나 의존성이 추가되었는가?
- `GetComponent()` 반환 Annotation과 호출하는 사용자 정의 Method의 소유 타입이 일치하는가?
- mLua LSP 진단과 가능한 범위의 VS Code Problems 결과를 확인했는가?
- TODO가 구현되었거나 새로운 TODO가 생겼는가?
- 아직 구현하지 않은 기능과 책임 경계가 달라졌는가?
- 마지막 갱신일을 현재 작업일로 변경했는가?
