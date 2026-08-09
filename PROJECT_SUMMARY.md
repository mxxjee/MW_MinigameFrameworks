# 프로젝트 코드 구조 요약

> 이 문서는 현재 프로젝트 코드의 실제 경로, 클래스 책임, 호출 흐름을 빠르게 파악하기 위한 기준 문서다.
>
> **유지보수 규칙:** mLua 코드나 관련 UI·맵 구성을 생성·수정·삭제·이동할 때는 같은 작업에서 이 문서도 반드시 갱신한다. 문서와 코드가 다르면 실제 코드를 기준으로 바로잡는다.

- 마지막 갱신일: 2026-08-08
- 확인 브랜치: `Framework_Test`
- 분석 기준: 현재 작업 트리에 실제로 존재하는 파일
- 현재 단계: 로비→선택 화면→Registry 조회→미니게임 맵 이동 흐름 구현 중

## 1. 현재 활성 경로

```text
RootDesk/MyDesk/
├─ Logics/
│  ├─ DefaultLogic/
│  │  ├─ GameEnum.mlua
│  │  ├─ GameHelper.mlua
│  │  └─ GameLogic.mlua
│  └─ MinigameLogic/
│     ├─ MinigameManager.mlua
│     └─ MinigameRegistry.mlua
├─ Minigame/
│  ├─ MinigameComponent.mlua
│  └─ MinigameData.mlua
├─ UI/
│  └─ EnterBtnComponent.mlua
├─ MapEnterController.mlua
├─ SelectSceneControllerComponent.mlua
├─ SelectWidgetUIComponent.mlua
└─ ETC/
   ├─ Monster.mlua
   ├─ PlayerAttack.mlua
   ├─ PlayerHit.mlua
   ├─ UIPopup.mlua
   └─ UIToast.mlua
```

### 경로 이동 상태

| 이전 경로 | 현재 경로 |
|---|---|
| `RootDesk/MyDesk/Logics/GameEnum.mlua` | `RootDesk/MyDesk/Logics/DefaultLogic/GameEnum.mlua` |
| `RootDesk/MyDesk/Logics/GameHelper.mlua` | `RootDesk/MyDesk/Logics/DefaultLogic/GameHelper.mlua` |
| `RootDesk/MyDesk/Minigame/MinigameManager.mlua` | `RootDesk/MyDesk/Logics/MinigameLogic/MinigameManager.mlua` |
| `RootDesk/MyDesk/Minigame/MinigameRegistry.mlua` | `RootDesk/MyDesk/Logics/MinigameLogic/MinigameRegistry.mlua` |

`GameLogic`은 현재 `RootDesk/MyDesk/Logics/DefaultLogic/GameLogic.mlua`에 있다. `MinigameComponent`와 `MinigameData`는 `RootDesk/MyDesk/Minigame/`에 유지된다.

## 2. 클래스 현황

| 파일 | 선언 | 전역 접근자 또는 사용 형태 | 책임 |
|---|---|---|---|
| `Logics/DefaultLogic/GameLogic.mlua` | `@Logic script GameLogic` | `_GameLogic` | 플레이 모드·점수·제한시간·선택 게임 상태와 시작/종료 진입점 |
| `Logics/DefaultLogic/GameEnum.mlua` | `@Logic script GameEnum` | `_GameEnum` | 플레이 모드 enum 테이블 초기화 |
| `Logics/DefaultLogic/GameHelper.mlua` | `@Logic script GameHelper` | `_GameHelper` | 경로로 UI Group을 찾아 활성화/비활성화 |
| `Logics/MinigameLogic/MinigameRegistry.mlua` | `@Logic script MinigameRegistry` | `_MinigameRegistry` | 미니게임 이름·ID·맵 경로의 전역 등록 및 조회 |
| `Logics/MinigameLogic/MinigameManager.mlua` | `@Logic script MinigameManager` | `_MinigameManager` | 로컬 미니게임 Component와 경과시간·실행 여부 관리 |
| `Minigame/MinigameData.mlua` | `@Struct script MinigameData` | `MinigameData()` | 미니게임 이름·ID·MapName 저장 |
| `Minigame/MinigameComponent.mlua` | `@Component script MinigameComponent` | 엔티티 부착/상속 기반 | 미니게임 초기화·갱신·해제 인터페이스 |
| `UI/EnterBtnComponent.mlua` | `@Component` | UI 버튼 부착 | 싱글/멀티 진입 분기와 선택 화면 이동 |
| `MapEnterController.mlua` | `@Component` | 맵 진입 엔티티 부착 | SelectScene 진입 시 선택 화면 초기화 |
| `SelectSceneControllerComponent.mlua` | `@Component` | 선택 화면 Controller | Registry 기반 선택 위젯 생성 |
| `SelectWidgetUIComponent.mlua` | `@Component` | 선택 위젯 모델 부착 | 게임 이름 표시와 위젯 위치 설정 |

## 3. 전역 Logic 구조

### 3.1 `GameEnum`

`OnBeginPlay()`은 ClientOnly에서 다음 플레이 모드 테이블을 만든다.

| 이름 | 값 |
|---|---:|
| `SINGLE` | `0` |
| `MULTI` | `1` |

`GameLogic.PlayMode`와 `EnterBtnComponent.Mode`는 정수 값을 사용하고 `_GameEnum.PlayMode`와 비교한다.

### 3.2 `GameHelper`

`EnableUIGroup(string path, boolean bEnable)`은 `_EntityService:GetEntityByPath(path)`로 UI Entity를 찾고, 유효할 때 `Enable`을 변경한다.

현재 호출 경로:

- `/ui/SelectButtonGroup`
- `/ui/SelectGroup`

### 3.3 `GameLogic`

#### Property

| 이름 | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `CurrentGameId` | `string` | `""` | 현재 선택한 미니게임 ID |
| `PlayMode` | `integer` | `0` | `SINGLE` 또는 `MULTI` |
| `CurrentScore` | `number` | `0` | 현재 점수 |
| `CurrentMaxTime` | `number` | `60` | 제한시간 |
| `MyPlayerId` | `integer` | `0` | 로컬 플레이어 ID 저장 자리 |
| `OpponentPlayerId` | `string` | `""` | 상대 플레이어 ID |
| `MyRankingData` | `table` | `{}` | 랭킹 데이터 저장 자리 |

#### 주요 Method

| 메서드 | 현재 동작 |
|---|---|
| `SelectMinigame(string gameId)` | 빈 ID를 거부하고 `CurrentGameId`를 저장한 뒤 Registry에서 `MinigameData`를 찾아 `MapName`으로 로컬 플레이어를 Teleport한다. |
| `SetPlayMode(integer playMode)` | `_GameEnum.PlayMode.SINGLE/MULTI`만 허용한다. |
| `SetCurrentScore/AddScore/ResetScore` | 점수를 설정·가산·초기화한다. |
| `SetCurrentMaxTime(number maxTime)` | 0보다 큰 제한시간만 허용한다. |
| `SetOpponentPlayer/ClearOpponentPlayer` | 상대 ID를 설정·초기화한다. |
| `StartMinigame(string gameId)` | Client 실행 Method. 선택과 맵 이동이 성공하면 `_MinigameManager:StartGame(CurrentGameId)`을 호출한다. |
| `EndMinigame()` | `_MinigameManager:EndGame()`에 종료를 위임한다. |
| `OnUpdate(number delta)` | ClientOnly에서 Manager를 갱신하고 제한시간에 도달하면 종료한다. |

### 3.4 `MinigameRegistry`

`MinigameRegistry`는 현재 `@Component`가 아니라 전역 `@Logic`이다. Entity 부착이나 Registry Entity 연결 없이 `_MinigameRegistry`로 접근한다.

#### Property

| 이름 | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `MinigameDatas` | `table` | `{}` | `MinigameDatas[id] = MinigameData` 저장소 |

#### 등록된 미니게임

| 표시 이름 | ID | MapName |
|---|---|---|
| 버튼게임 | `Button_Game` | `ButtonGameScene` |
| 민거니 먹빵 | `GunHee_Muckbang` | `TestScene` |
| 민지꾸얌 | `Minji_Game` | `TestScene` |
| 쥬신뿌수기 | `Jusin_Game` | `TestScene` |

#### Method

| 메서드 | 현재 동작 |
|---|---|
| `OnBeginPlay()` | ClientOnly에서 위 네 미니게임을 중앙 등록한다. |
| `RegisterMinigame(string name, string id, string _mapName)` | 검증 후 `MinigameData`를 만들어 ID로 저장한다. |
| `ValidateRegisterData(string id)` | 빈 ID와 중복 ID를 거부한다. |
| `GetMinigameData(string id)` | `MinigameData` 또는 `nil`을 반환한다. |

### 3.5 `MinigameManager`

#### Property

| 이름 | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `CurrentGame` | `MinigameComponent` | `nil` | 현재 실행 중인 구체 미니게임 Component |
| `CurrentTime` | `number` | `0` | 로컬 경과시간 |
| `IsPlaying` | `boolean` | `false` | 실행 여부 |

#### Method

| 메서드 | 현재 동작 |
|---|---|
| `StartGame(string gameId)` | ClientOnly에서 Registry의 `MinigameData`를 조회하고 시간/실행 상태를 초기화한 뒤 `CurrentGame:Initialize(gameData)`를 호출한다. |
| `Update(number deltaTime)` | 실행 중이고 `CurrentGame`이 있을 때 시간 누적 후 `Update()`를 호출한다. |
| `EndGame()` | `Release()` 호출 후 Component·시간·실행 상태를 초기화한다. |

현재 `StartGame()` 안의 `CurrentGame` 할당 코드는 주석 상태다. 또한 `MinigameData`는 더 이상 GameComponent를 보관하지 않는다. 따라서 현재 코드만 기준으로는 `CurrentGame`이 외부에서 먼저 설정되지 않으면 `Initialize()` 호출 대상이 `nil`인 연결 미완성 상태다.

## 4. 미니게임 데이터와 기반 Component

### 4.1 `MinigameData`

| 이름 | 타입 | 기본값 |
|---|---|---:|
| `Name` | `string` | `""` |
| `ID` | `string` | `""` |
| `MapName` | `string` | `""` |

`InitData(string _name, string _id, string _mapName)`는 세 값을 그대로 저장한다.

현재 `GameComponent` Property와 Component 참조는 존재하지 않는다. Registry의 책임은 실행 Component 등록이 아니라 미니게임 메타데이터와 이동할 맵의 연결이다.

### 4.2 `MinigameComponent`

| 메서드 | 시그니처 | 현재 구현 |
|---|---|---|
| 초기화 | `Initialize(MinigameData gameData)` | 빈 기반 Method |
| 갱신 | `Update(number deltaTime)` | 빈 기반 Method |
| 해제 | `Release()` | 빈 기반 Method |

`Initialize()`는 `table`이나 `any`가 아니라 구체 타입 `MinigameData`를 사용한다.

## 5. 선택 화면과 UI 흐름

### 5.1 `EnterBtnComponent`

- `Mode: integer`를 가진다.
- 버튼 클릭 시 `SINGLE`이면 `_GameLogic:SetPlayMode(Mode)`를 호출한다.
- 로컬 플레이어를 `SelectScene`의 `Vector3(0, 0, 0)`으로 이동한다.
- 처리 후 `/ui/SelectButtonGroup`을 비활성화한다.
- `MULTI` 분기는 서버 매칭 TODO만 존재한다.

### 5.2 `MapEnterController`

ClientOnly `OnMapEnter(Entity enteredMap)`에서 맵 이름이 `SelectScene`이면:

1. `self.Entity.SelectSceneControllerComponent:Initialize()`
2. `_GameHelper:EnableUIGroup("/ui/SelectGroup", true)`

를 호출한다.

### 5.3 `SelectSceneControllerComponent`

| Property | 타입 | 기본값/역할 |
|---|---|---|
| `SelectWidgetModelId` | `string` | 선택 위젯 모델 ID |
| `SlotSpacing` | `number` | `300`, 가로 간격 |
| `GameList` | `table` | 현재 Registry 데이터 목록 |
| `WidgetList` | `table` | 생성된 위젯 Entity 목록 |

`Refresh_Slots()`는 `_MinigameRegistry.MinigameDatas`를 순회하고 각 게임마다:

1. `/ui/SelectGroup/GameList` 아래에 위젯 모델을 Spawn한다.
2. `---@type SelectWidgetUIComponent`로 정확한 타입을 지정해 `GetComponent("script.SelectWidgetUIComponent")` 결과를 받는다.
3. `Initialize(gameData.Name, pos)`를 호출한다.
4. 0, 300, 600, 900 순으로 가로 배치한다.

현재 `HandleKeyDownEvent`는 생성만 되어 있고 실행 로직은 없다.

### 5.4 `SelectWidgetUIComponent`

`Initialize(string gameName, Vector2 pos)`는:

- 자식 `Name` Entity의 `TextGUIRendererComponent.Text`에 게임 이름을 설정한다.
- 위젯 `UITransformComponent.anchoredPosition`을 전달받은 위치로 설정한다.

현재 위젯은 표시와 배치만 담당한다. 게임 ID 저장, 선택 상태, 클릭 처리, `_GameLogic:StartMinigame(gameId)` 연결은 아직 없다.

## 6. 현재 호출 흐름

### 로비에서 선택 화면 진입

```text
EnterBtnComponent.HandleButtonClickEvent
  → _GameLogic:SetPlayMode(SINGLE)
  → _TeleportService:TeleportToMapPosition(..., "SelectScene")
  → _GameHelper:EnableUIGroup("/ui/SelectButtonGroup", false)
```

### 선택 화면 구성

```text
MapEnterController.OnMapEnter("SelectScene")
  → SelectSceneControllerComponent.Initialize()
  → Refresh_Slots()
  → _MinigameRegistry.MinigameDatas 순회
  → SelectWidget 모델 Spawn
  → SelectWidgetUIComponent.Initialize(gameData.Name, pos)
  → /ui/SelectGroup 활성화
```

### 미니게임 시작 요청

```text
_GameLogic:StartMinigame(gameId)
  → SelectMinigame(gameId)
  → _MinigameRegistry:GetMinigameData(gameId)
  → GameData.MapName으로 Teleport
  → _MinigameManager:StartGame(gameId)
  → _MinigameRegistry:GetMinigameData(gameId)
  → CurrentGame:Initialize(gameData)
```

마지막 `CurrentGame` 선택·할당 단계는 현재 구현되지 않았다.

## 7. ETC 지원 코드

| 파일 | 역할 |
|---|---|
| `ETC/Monster.mlua` | 동기화 HP, 피격, 사망, 숨김/삭제, 선택적 Respawn |
| `ETC/PlayerAttack.mlua` | BoxShape 기반 일반 공격, 고정 피해·치명타 계산 |
| `ETC/PlayerHit.mlua` | 피격 면역 쿨다운 판정 |
| `ETC/UIPopup.mlua` | 확인/취소 Callback과 Tween 기반 Popup 표시 |
| `ETC/UIToast.mlua` | 시간·Alpha Tween 기반 Toast 표시 |

이 파일들은 현재 `RootDesk/MyDesk/ETC/`에 유지된다. 예전 `Default/` 기준 문서가 남아 있으면 현재 구조에 맞춰 `ETC/` 기준으로 해석해야 한다.

## 8. 현재 구현 상태와 남은 연결

- Registry는 전역 `@Logic`이며 별도 Entity/Component 연결이 필요하지 않다.
- Registry는 네 개의 미니게임 메타데이터를 등록한다.
- `MinigameData`는 `Name`, `ID`, `MapName`만 저장한다.
- 선택 화면은 Registry 데이터로 위젯을 생성하고 이름을 표시한다.
- 선택 위젯의 클릭·게임 ID 전달·`StartMinigame()` 연결은 아직 없다.
- `MinigameManager.CurrentGame`을 실제 `MinigameComponent`로 설정하는 경로가 아직 없다.
- `StartGame()`의 현재 `CurrentGame:Initialize(gameData)` 호출은 Component 할당 전에는 사용할 수 없다.
- `GameLogic.SelectMinigame()`은 Registry 조회 결과가 `nil`인지 확인하기 전에 `GameData.MapName`을 사용한다.
- MULTI 플레이의 서버 매칭·상대 연결은 TODO다.
- 실제 점수 산정 규칙과 랭킹 데이터 스키마는 아직 없다.

## 9. 정적 타입 검사 상태

현재 프레임워크에서 확인한 구체 타입:

- `GameLogic.SelectMinigame()`의 `GameData`: `MinigameData`
- `MinigameRegistry.RegisterMinigame()`의 `gameData`: `MinigameData`
- `MinigameRegistry.GetMinigameData()` 반환: `MinigameData`
- `MinigameManager.CurrentGame`: `MinigameComponent`
- `MinigameComponent.Initialize()` 매개변수: `MinigameData`
- `SelectSceneControllerComponent.widgetComp`: `SelectWidgetUIComponent`

현재 실행 코드의 `GetComponent()` 호출은 `SelectSceneControllerComponent.mlua` 한 곳이다. 반환값에는 실제 Custom Method `Initialize()`의 소유 타입인 `SelectWidgetUIComponent` Annotation이 지정되어 있다.

번들 mLua LSP `1.1.4`로 다음 11개 활성 프레임워크 파일을 검사했으며 diagnostic/error/warning은 모두 0건이었다.

- `GameLogic`, `GameEnum`, `GameHelper`
- `MinigameManager`, `MinigameRegistry`
- `MinigameComponent`, `MinigameData`
- `MapEnterController`, `SelectSceneControllerComponent`, `SelectWidgetUIComponent`, `EnterBtnComponent`

Codex 환경에서는 사용자의 VS Code Problems 패널 자체를 직접 읽지 못했다. 따라서 “VS Code Problems 오류 없음”이라고 단정하지 않으며, 위 결과는 번들 mLua LSP 진단 기준이다.

## 10. 코드 변경 시 문서 동기화 체크리스트

- 클래스 파일의 실제 경로가 이동했는가?
- `@Logic`, `@Component`, `@Struct` 책임이 변경됐는가?
- Property 이름·타입·기본값이 변경됐는가?
- Method 시그니처와 ExecSpace가 변경됐는가?
- Registry 등록 게임과 `MapName`이 변경됐는가?
- 선택 화면·UI·맵 이동 흐름이 변경됐는가?
- `GetComponent()` 반환 Annotation과 Custom Method 소유 타입이 일치하는가?
- 구체 타입을 `Component`, `table`, `any`로 불필요하게 낮추지 않았는가?
- 구현된 연결과 아직 TODO인 연결을 구분했는가?
- 마지막 갱신일을 현재 작업일로 갱신했는가?
