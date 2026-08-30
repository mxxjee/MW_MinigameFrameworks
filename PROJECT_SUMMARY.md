# 프로젝트 코드 구조 및 현재 구현 상태

> 이 문서는 현재 프로젝트의 실제 경로, 클래스 책임, 리소스 연결, 실행 흐름과 미완성 지점을 빠르게 파악하기 위한 기준 문서다.
>
> **유지보수 규칙:** mLua 코드나 관련 UI·맵 구성을 생성·수정·삭제·이동할 때는 같은 작업에서 이 문서도 반드시 갱신한다. 문서와 코드가 다르면 실제 코드를 기준으로 바로잡는다.

- 마지막 점검일: 2026-08-18
- 확인 브랜치: `Minji_Branch`
- 확인 HEAD: `d630769` (`ADD: Spirte`)
- 분석 기준: 현재 작업 트리의 mLua, Map, UI, Model, Sprite 리소스
- 현재 단계: **로비 → 게임 선택 → 확인 팝업 → 사용자별 싱글 Instance Room 생성·입장** 구현
- 아직 미완성: 실제 `MinigameComponent` 선택·실행, `MULTI` 매칭, 미구현 게임 맵 3개

## 1. 현재 활성 구조

```text
RootDesk/MyDesk/
├─ Assets/Img/
│  ├─ Loby.sprite
│  ├─ SelectScene_BG.sprite
│  ├─ button1_default.sprite
│  ├─ button1_Hover.sprite
│  ├─ button2_default.sprite
│  └─ button2_hover.sprite
├─ ETC/
│  ├─ Monster.mlua
│  ├─ PlayerAttack.mlua
│  ├─ PlayerHit.mlua
│  ├─ UIPopup.mlua
│  └─ UIToast.mlua
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
├─ SelectScene/
│  ├─ SelectSceneControllerComponent.mlua
│  └─ SelectWidgetUIComponent.mlua
├─ UI/
│  └─ EnterBtnComponent.mlua
├─ MapEnterController.mlua
├─ SelectSceneControllerComponent.mlua
├─ SelectWidgetUIComponent.mlua
└─ Default/
   ├─ Monster.mlua
   ├─ PlayerAttack.mlua
   ├─ PlayerHit.mlua
   ├─ UIPopup.mlua
   └─ UIToast.mlua
```

### 경로 이동 상태

| 이전 경로 | 현재 경로 |
|---|---|
| `RootDesk/MyDesk/Logics/GameLogic.mlua` | `RootDesk/MyDesk/Logics/DefaultLogic/GameLogic.mlua` |
| `RootDesk/MyDesk/Logics/GameEnum.mlua` | `RootDesk/MyDesk/Logics/DefaultLogic/GameEnum.mlua` |
| `RootDesk/MyDesk/Logics/GameHelper.mlua` | `RootDesk/MyDesk/Logics/DefaultLogic/GameHelper.mlua` |
| `RootDesk/MyDesk/Minigame/MinigameManager.mlua` | `RootDesk/MyDesk/Logics/MinigameLogic/MinigameManager.mlua` |
| `RootDesk/MyDesk/Minigame/MinigameRegistry.mlua` | `RootDesk/MyDesk/Logics/MinigameLogic/MinigameRegistry.mlua` |
| `RootDesk/MyDesk/SelectSceneControllerComponent.mlua` | `RootDesk/MyDesk/SelectScene/SelectSceneControllerComponent.mlua` |
| `RootDesk/MyDesk/SelectWidgetUIComponent.mlua` | `RootDesk/MyDesk/SelectScene/SelectWidgetUIComponent.mlua` |
| `RootDesk/MyDesk/Default/*.mlua` | `RootDesk/MyDesk/ETC/*.mlua` |

`GameLogic`은 `RootDesk/MyDesk/Logics/GameLogic.mlua`에 유지된다. `MinigameComponent`와 `MinigameData`는 `RootDesk/MyDesk/Minigame/`에 유지된다.

| 파일 | 선언 | 전역 접근/부착 위치 | 현재 책임 |
|---|---|---|---|
| `Logics/GameLogic.mlua` | `@Logic script GameLogic` | `_GameLogic` | 플레이 모드·점수·제한시간·선택 게임 상태와 시작/종료 진입점 |
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

## 3. 전역 Logic

### 3.1 `GameEnum`

ClientOnly `OnBeginPlay()`에서 다음 값을 만든다.

| 이름 | 값 |
|---|---:|
| `SINGLE` | `0` |
| `MULTI` | `1` |

`LobbyGroup`의 Single 버튼은 `Mode = 0`, Multi 버튼은 `Mode = 1`로 직렬화되어 있다.

### 3.2 `GameHelper`

`EnableUIGroup(string path, boolean bEnable)`은 `_EntityService:GetEntityByPath(path)`로 Entity를 찾고 유효할 때만 `Enable`을 변경한다.

현재 코드에서 사용하는 경로:

- `/ui/LobbyGroup`
- `/ui/SelectGroup`
- `/ui/PopupGroup`
- `/ui/SelectButtonGroup` — 현재 실제 UI 파일에는 없는 과거 경로

### 3.3 `GameLogic`

#### Property

| 이름 | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `CurrentGameId` | `string` | `""` | 현재 룸에서 선택된 게임 ID |
| `PlayMode` | `integer` | `0` | `SINGLE` 또는 `MULTI` |
| `CurrentScore` | `number` | `0` | 현재 점수 |
| `CurrentMaxTime` | `number` | `60` | 제한시간 |
| `MyPlayerId` | `integer` | `0` | 로컬 플레이어 ID 저장 자리 |
| `OpponentPlayerId` | `string` | `""` | 상대 플레이어 ID |
| `MyRankingData` | `table` | `{}` | 랭킹 데이터 저장 자리 |

#### 룸 관련 Method

| 메서드 | ExecSpace | 현재 동작 |
|---|---|---|
| `SelectMinigame(string gameID, table userIds)` | `ServerOnly` | 현재 Instance Room을 유지하며 지정 사용자들을 선택 게임 맵으로 이동한다. 현재 선택 UI에서는 직접 호출하지 않는다. |
| `EnterMiniGameRoom(table gameIds, string roomKey, table userIds)` | `ServerOnly` | SharedMemory에 첫 게임 ID를 저장하고, 게임 맵 목록으로 Instance Room을 만든 뒤 첫 맵으로 사용자를 이동한다. |
| `MakeSoloRoomKey(string userId)` | `ServerOnly` | `Solo_{userId}_{UtcNow.Elapsed}` 형식의 키를 만든다. |
| `HandleRoomBeginEvent(RoomBeginEvent)` | Server Event | 현재 Room Key의 SharedMemory에서 `CurrentGameId`를 읽어 서버 영역의 Property를 복원한다. |

#### 상태·생명주기 Method

| 메서드 | 현재 동작 |
|---|---|
| `SetPlayMode(integer playMode)` | `_GameEnum.PlayMode.SINGLE/MULTI`만 허용한다. |
| `SetCurrentScore/AddScore/ResetScore` | 점수를 설정·가산·초기화한다. |
| `SetCurrentMaxTime(number maxTime)` | 0보다 큰 제한시간만 허용한다. |
| `SetOpponentPlayer/ClearOpponentPlayer` | 상대 ID를 설정·초기화한다. |
| `StartMinigame(string gameId)` | Client 실행 Method. 현재 `gameId` 인자를 사용하지 않고 `_MinigameManager:StartGame(self.CurrentGameId)`만 호출한다. |
| `EndMinigame()` | `_MinigameManager:EndGame()`에 종료를 위임한다. |
| `OnUpdate(number delta)` | ClientOnly에서 Manager를 갱신하고 제한시간 도달 시 종료한다. |

현재 구현에서 `CurrentGameId`는 RoomBegin 시 **서버 영역**에서 갱신된다. Client의 `StartMinigame()`으로 값을 전달하거나 동기화하는 연결은 아직 없다.

### 3.4 `MinigameRegistry`

Registry는 Entity에 부착하는 Component가 아니라 전역 `@Logic`이며 `_MinigameRegistry`로 접근한다.

#### Property

| 이름 | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `MinigameDatas` | `table` | `{}` | `MinigameDatas[id] = MinigameData` 저장소 |
| `MinigameOrders` | `table` | `{}` | 등록된 게임 ID의 표시 순서 |
| `bInitailized` | `boolean` | `false` | 중복 초기화 방지 플래그. 현재 이름에 오탈자가 있으나 모든 참조가 동일하다. |

#### 등록된 미니게임

| 순서 | 표시 이름 | ID | MapName | 설명 | 실제 Map |
|---:|---|---|---|---|---|
| 1 | 버튼게임 | `Button_Game` | `ButtonGameScene` | 버튼 연타 안내 | 있음 |
| 2 | 민거니 먹빵 | `GunHee_Muckbang` | `TestScene` | 민거니 소개 | **없음** |
| 3 | 민지꾸얌 | `Minji_Game` | `TestScene` | 생일 안내 | **없음** |
| 4 | 쥬신뿌수기 | `Jusin_Game` | `TestScene` | 쥬신 뿌수기 안내 | **없음** |

#### Method

| 메서드 | 현재 동작 |
|---|---|
| `OnBeginPlay()` | `Initialize()`를 호출한다. |
| `Initialize()` | 이미 초기화했으면 종료하고, 네 게임을 등록한 뒤 플래그를 `true`로 만든다. |
| `RegisterMinigame(name, id, mapName, desc)` | 검증 후 `MinigameData`를 만들고 Dictionary와 순서 Table에 저장한다. |
| `ValidateRegisterData(string id)` | 빈 ID와 중복 ID를 거부한다. |
| `GetMinigameData(string id)` | 데이터가 없다고 판단하면 초기화를 시도한 뒤 `MinigameData` 또는 `nil`을 반환한다. |

`MinigameDatas`는 문자열 키 Dictionary이므로 `#self.MinigameDatas`는 데이터 수 확인 용도로 신뢰할 수 없다. 현재 `GetMinigameData()`의 `#self.MinigameDatas == 0` 검사는 매번 참이 될 수 있지만, `bInitailized`가 실제 중복 등록은 막고 있다.

### 3.5 `MinigameManager`

| Property | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `CurrentGame` | `MinigameComponent` | `nil` | 현재 실행할 구체 미니게임 Component |
| `CurrentTime` | `number` | `0` | 로컬 경과시간 |
| `IsPlaying` | `boolean` | `false` | 실행 여부 |

| 메서드 | 현재 동작 |
|---|---|
| `StartGame(string gameId)` | Registry 조회 후 시간과 상태를 초기화하고 `CurrentGame:Initialize(gameData)`를 호출한다. |
| `Update(number deltaTime)` | 실행 중이고 `CurrentGame`이 있을 때 시간을 누적하고 `Update()`를 호출한다. |
| `EndGame()` | `Release()` 호출 후 Component·시간·상태를 초기화한다. |

`StartGame()` 내부의 실제 `CurrentGame` 할당 코드는 주석 상태다. 따라서 외부에서 Component를 먼저 넣지 않으면 `CurrentGame:Initialize()`에서 nil 접근이 발생하는 미완성 구조다.

## 4. 미니게임 데이터와 기반 Component

### 4.1 `MinigameData`

| 이름 | 타입 | 기본값 |
|---|---|---:|
| `Name` | `string` | `""` |
| `ID` | `string` | `""` |
| `MapName` | `string` | `""` |
| `Description` | `string` | `""` |

`InitData(name, id, mapName, Desc)`가 네 값을 저장한다. 실제 실행 Component 참조는 보관하지 않는다.

### 4.2 `MinigameComponent`

| 메서드 | 시그니처 | 현재 구현 |
|---|---|---|
| 초기화 | `Initialize(MinigameData gameData)` | 빈 기반 Method |
| 갱신 | `Update(number deltaTime)` | 빈 기반 Method |
| 해제 | `Release()` | 빈 기반 Method |

구체 미니게임이 이 Component를 상속하거나 동일 인터페이스를 구현하고 Manager에 연결하는 단계가 남아 있다.

## 5. 로비·선택 화면·룸 입장 흐름

### 5.1 `EnterBtnComponent`

- `SingleButton`은 `Mode = 0`, `MultiButton`은 `Mode = 1`이다.
- Single 클릭 시 플레이 모드를 설정하고 로컬 플레이어를 `SelectScene`으로 Teleport한다.
- 클릭 후 `bSelected = true`로 설정한다.
- Fade Out 완료 이벤트에서 `bSelected`가 true인 경우 `/ui/LobbyGroup`과 `/ui/SelectButtonGroup`을 숨긴다.
- Multi 분기는 매칭 주석만 있으며 실제 서버 요청은 없다. 현재도 마지막에 `bSelected = true`는 설정된다.

### 5.2 `MapEnterController`

ClientOnly `OnMapEnter()`의 현재 동작:

1. 모든 대상 맵에서 `_ScreenTransitionService:SetFadeInOutEnable(true)` 호출
2. Registry가 초기화되지 않았으면 직접 `Initialize()` 호출
3. `LobbyScene`이면 `/ui/LobbyGroup` 활성화
4. `SelectScene`이면 `SelectSceneControllerComponent.Initialize()` 호출 후 `/ui/SelectGroup` 활성화

### 5.3 `SelectSceneControllerComponent`

#### 핵심 Property

| Property | 기본값/역할 |
|---|---|
| `GameListEntity` | `/ui/SelectGroup/GameList` Entity |
| `BtnOK`, `BtnCancel` | PopupGroup의 확인/취소 버튼 참조 |
| `PopupMessage` | 팝업 메시지 `TextComponent` 참조 |
| `SelectWidgetModelId` | `model://b9a29910-7633-405a-b5c4-1d7a4c7df643` |
| `SlotSpacing` | `300` |
| `CurrentIndex`, `MaxIndex` | 현재 선택 위치와 최대 슬롯 수 |
| `GameList` | 게임 ID 목록 |
| `WidgetList` | Spawn된 위젯 Entity 목록 |
| `MoveSpeed` | `30` |
| `DescUI` | `/ui/SelectGroup/Description` Entity 참조 |
| `bShowPopup` | 팝업 표시 중 입력 잠금 상태 |

#### 초기화와 슬롯 생성

`Initialize()`는 확인/취소 버튼 이벤트를 연결하고 선택 상태를 초기화한 뒤 `Refresh_Slots()`와 `SetDescription()`을 호출한다.

`Refresh_Slots()`는 `MinigameOrders`를 순회해 다음을 수행한다.

1. 게임 ID를 `GameList`에 저장
2. 선택 위젯 Model을 `/ui/SelectGroup/GameList` 아래에 Spawn
3. `SelectWidgetUIComponent.Initialize(gameData.Name, pos)` 호출
4. 슬롯을 `SlotSpacing` 간격으로 배치

현재 순회가 `ipairs()`가 아니라 `pairs()`이므로 배열의 순서를 언어 차원에서 보장하지는 않는다.

#### 입력·팝업·룸 진입

- 좌우 방향키로 `CurrentIndex`를 변경하고 Lerp 이동을 시작한다.
- 이동 완료 시 위치를 보정하고 현재 게임 설명을 갱신한다.
- 팝업이 열린 동안 좌우 방향키를 차단한다.
- Space 키로 현재 게임명이 포함된 확인 팝업을 표시한다.
- 취소 버튼은 팝업을 닫고 `bShowPopup = false`로 되돌린다.
- 확인 버튼은 Server 실행 `EnterSoloMinigame(gameID)`을 호출한다.
- Fade Out 시작 이벤트에서 PopupGroup과 SelectGroup을 숨긴다.

`EnterSoloMinigame()`은 `senderUserId`로 Solo Room Key를 만들고 `_GameLogic:EnterMiniGameRoom({gameID}, roomKey, {userId})`을 호출한다.

### 5.4 `SelectWidgetUIComponent`

`Initialize(string gameName, Vector2 pos)`는 자식 `Name`의 `TextGUIRendererComponent.Text`와 자신의 `anchoredPosition`을 설정한다.

현재 `@Sync property MinigameComponent NewValue1 = nil`이 존재하지만 어떤 코드에서도 사용하지 않는다. 에디터에서 임시로 생성된 Property인지 확인이 필요하다.

### 5.5 전체 실행 흐름

```text
LobbyScene 진입
  → MapEnterController: Fade 활성화, Registry 초기화, LobbyGroup 활성화
  → SingleButton 클릭
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

## 7. Default 지원 코드

| 파일 | 역할 |
|---|---|
| `Default/Monster.mlua` | 동기화 HP, 피격, 사망, 숨김/삭제, 선택적 Respawn |
| `Default/PlayerAttack.mlua` | BoxShape 기반 일반 공격, 고정 피해·치명타 계산 |
| `Default/PlayerHit.mlua` | 피격 면역 쿨다운 판정 |
| `Default/UIPopup.mlua` | 확인/취소 Callback과 Tween 기반 Popup 표시 |
| `Default/UIToast.mlua` | 시간·Alpha Tween 기반 Toast 표시 |

이 파일들은 현재 경로 이동 대상이 아니며 `RootDesk/MyDesk/Default/`에 유지된다.

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
- Registry 등록 게임·순서·설명·`MapName`이 변경됐는가?
- Map에 등록된 Custom Component가 변경됐는가?
- UI Entity 경로와 스크립트 Property 참조가 변경됐는가?
- Lobby → Select → Room → Game 실행 흐름이 변경됐는가?
- 구현된 연결과 아직 TODO인 연결을 구분했는가?
- 알려진 런타임 위험과 Play Test 결과를 갱신했는가?
- 마지막 점검일, 브랜치, HEAD를 갱신했는가?
