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
├─ MingiBabo.mlua
└─ Model_MinigameSelectWidget.model

map/
├─ LobbyScene.map
├─ SelectScene.map
└─ ButtonGameScene.map

ui/
├─ DefaultGroup.ui
├─ LobbyGroup.ui
├─ SelectGroup.ui
├─ PopupGroup.ui
└─ ToastGroup.ui
```

모든 Custom mLua 파일에는 같은 경로에 대응하는 `.codeblock` 파일이 있다. `.directory` 파일은 Maker 내부 폴더 구조를 보존한다.

### 주요 경로 변경 이력

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

## 2. 핵심 클래스 현황

| 파일 | 선언 | 전역 접근/부착 위치 | 현재 책임 |
|---|---|---|---|
| `Logics/DefaultLogic/GameLogic.mlua` | `@Logic script GameLogic` | `_GameLogic` | 플레이 상태, 싱글 룸 생성·이동, 룸 간 선택 게임 ID 전달 |
| `Logics/DefaultLogic/GameEnum.mlua` | `@Logic script GameEnum` | `_GameEnum` | `SINGLE`, `MULTI` 플레이 모드 값 제공 |
| `Logics/DefaultLogic/GameHelper.mlua` | `@Logic script GameHelper` | `_GameHelper` | 경로로 UI Group 활성화/비활성화 |
| `Logics/MinigameLogic/MinigameRegistry.mlua` | `@Logic script MinigameRegistry` | `_MinigameRegistry` | 미니게임 메타데이터 등록, 순서 저장, 조회 |
| `Logics/MinigameLogic/MinigameManager.mlua` | `@Logic script MinigameManager` | `_MinigameManager` | 로컬 미니게임 생명주기와 시간 관리 뼈대 |
| `Minigame/MinigameData.mlua` | `@Struct script MinigameData` | `MinigameData()` | 이름, ID, 맵, 설명 데이터 |
| `Minigame/MinigameComponent.mlua` | `@Component` | 구체 미니게임 Component의 기반 | `Initialize/Update/Release` 인터페이스 |
| `UI/EnterBtnComponent.mlua` | `@Component` | Lobby의 Single/Multi 버튼 | 플레이 모드 분기, SelectScene 이동, Fade 종료 처리 |
| `MapEnterController.mlua` | `@Component` | LobbyScene, SelectScene | 맵 진입 시 페이드·Registry·UI 초기화 |
| `SelectScene/SelectSceneControllerComponent.mlua` | `@Component` | SelectScene Controller Entity | 슬롯 생성·이동, 설명, 확인 팝업, 싱글 룸 진입 요청 |
| `SelectScene/SelectWidgetUIComponent.mlua` | `@Component` | 선택 슬롯 Model | 게임 이름 표시와 슬롯 위치 설정 |

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
  → SelectScene으로 Teleport
  → Fade Out 완료 시 LobbyGroup 숨김

SelectScene 진입
  → SelectSceneController.Initialize()
  → Registry 순서 Table로 선택 슬롯 Spawn
  → 첫 게임 설명 표시
  → 좌우 키로 슬롯 이동 및 설명 갱신
  → Space 키로 확인 팝업 표시
  → 확인 클릭

Server EnterSoloMinigame(gameID)
  → Solo Room Key 생성
  → SharedMemory에 CurrentGameId 저장
  → 선택 게임 Map을 포함한 Instance Room 생성
  → 해당 사용자만 첫 게임 Map으로 이동
  → RoomBegin 이벤트에서 서버 CurrentGameId 복원
```

여기까지가 현재 연결된 흐름이다. Room 입장 이후 `StartMinigame()` 또는 구체 `MinigameComponent`를 시작하는 호출은 없다.

## 6. Map·UI·Model 연결 상태

### 6.1 Map

| Map | EntryKey | 주요 연결 | 상태 |
|---|---|---|---|
| `LobbyScene.map` | `map://lobbyscene` | `/maps/LobbyScene/MapEnter`에 `MapEnterController` | 로비 진입점 |
| `SelectScene.map` | `map://selectscene` | `MapEnterController`, `SelectSceneControllerComponent` | 선택 UI와 Popup 참조 연결 |
| `ButtonGameScene.map` | `map://buttongamescene` | Custom `script.*` Component 없음 | Instance Room 대상 Map 껍데기 |

Registry가 참조하는 `TestScene.map`은 현재 저장소에 없다.

### 6.2 UI

| UI | 주요 Entity/역할 |
|---|---|
| `LobbyGroup.ui` | 배경, `SingleButton`, `MultiButton`; 두 버튼에 `EnterBtnComponent` 부착 |
| `SelectGroup.ui` | `BG`, `GameList`, `Description/Text`, 안내 Sprite |
| `PopupGroup.ui` | `PopupMessage`, `PopupBtnOK`, `PopupBtnCancel` |
| `ToastGroup.ui` | `Toast_message` |
| `DefaultGroup.ui` | Attack, Jump, Joystick, Chat 기본 UI |

각 UI Group은 리소스상 `Enable = true`이며 런타임 코드가 필요한 시점에 활성/비활성 상태를 조정한다.

### 6.3 선택 위젯 Model과 Sprite

- `Model_MinigameSelectWidget.model`
  - EntryKey: `model://b9a29910-7633-405a-b5c4-1d7a4c7df643`
  - `SelectWidgetUIComponent` 부착
  - 자식 `Thumbnail`, `Name` 포함
- 로비와 선택 화면 배경 Sprite가 Assets에 등록되어 있다.
- `button1_*`, `button2_*` Sprite는 로비 버튼 기본/호버 상태에 사용된다.

## 7. ETC 지원 코드

| 파일 | 선언/상속 | 역할 |
|---|---|---|
| `ETC/Monster.mlua` | `Component` | 동기화 HP, 피격, 사망, 숨김/삭제, 선택적 Respawn |
| `ETC/PlayerAttack.mlua` | `AttackComponent` | BoxShape 일반 공격, 고정 피해·치명타 계산 |
| `ETC/PlayerHit.mlua` | `HitComponent` | 피격 면역 쿨다운 판정 |
| `ETC/UIPopup.mlua` | `Logic` | 확인/취소 Callback과 Tween 기반 범용 Popup |
| `ETC/UIToast.mlua` | `Logic` | 메시지 표시와 Tween 기반 Toast |
| `MingiBabo.mlua` | `Component` | 동기화 문자열 하나를 가진 테스트성 Component |

현재 선택 확인 흐름은 범용 `_UIPopup` Logic 대신 `SelectSceneControllerComponent`가 PopupGroup을 직접 제어한다.

## 8. 구현 완료도

| 영역 | 상태 | 비고 |
|---|---|---|
| 로비 Single/Multi 버튼 배치 | 완료 | Single만 실제 이동 동작 연결 |
| 로비 → SelectScene 이동 | 완료 | Fade 이벤트와 UI 정리 포함 |
| 미니게임 Registry | 완료 | 이름, ID, 맵, 설명, 표시 순서 저장 |
| 선택 슬롯 생성·좌우 이동 | 완료 | `pairs()` 순서 보장 문제는 남음 |
| 선택 게임 설명 표시 | 완료 | 초기 진입 및 이동 완료 시 갱신 |
| 선택 확인 팝업 | 완료 | 팝업 중 좌우 입력 차단 |
| 싱글 Instance Room 분리 | 1차 완료 | SharedMemory/이동 결과 오류 처리 보강 필요 |
| 실제 미니게임 Map | 부분 | `ButtonGameScene`만 존재 |
| Room 입장 후 게임 시작 | 미완성 | `StartMinigame()` 호출 연결 없음 |
| 구체 `MinigameComponent` 연결 | 미완성 | `CurrentGame` 할당 경로 없음 |
| MULTI 매칭·룸 입장 | 미완성 | 주석만 존재 |
| 점수·랭킹·결과 화면 | 미완성 | 데이터 자리만 존재 |

## 9. 알려진 문제와 병합 전 확인사항

### 우선순위 높음

1. Registry의 세 게임이 존재하지 않는 `TestScene`을 가리킨다. 해당 게임 선택 시 Instance Room 생성 또는 이동이 실패할 수 있다.
2. `EnterMiniGameRoom()`이 `GetSharedMemory()` 반환 코드와 `memory` 유효성을 확인하기 전에 `SetVariableAndWait()`를 호출한다.
3. `SetVariableAndWait()` 결과와 `room:MoveUsers()` 반환값을 검사하지 않아 실패를 성공으로 처리할 수 있다.
4. `MinigameManager.CurrentGame` 할당 없이 `CurrentGame:Initialize()`를 호출하는 구조다.

### 구조·정리 필요

1. `MinigameOrders` 순회는 순서를 보장하도록 `pairs()` 대신 `ipairs()` 사용을 검토해야 한다.
2. 문자열 키 Dictionary인 `MinigameDatas`에 길이 연산자 `#`를 사용하지 않는 초기화 판정이 필요하다.
3. `/ui/SelectButtonGroup`은 실제 리소스에 없는 경로다.
4. `SelectWidgetUIComponent.NewValue1`은 사용되지 않는 동기화 Property다.
5. 선택 화면 재진입 시 버튼 이벤트가 중복 연결되거나 기존 Spawn Widget이 남는지 Maker Play Test가 필요하다.
6. 확인 버튼을 연속 클릭할 때 여러 Room 생성 요청을 막는 상태값이 없다.
7. `StartMinigame(gameId)`는 현재 인자를 사용하지 않으며 서버의 `CurrentGameId`를 Client에 전달하는 경로도 없다.
8. Multi 버튼은 실제 동작 없이 `bSelected = true`만 설정한다.

## 10. 정적 검증 상태

HEAD `d630769` 기준 확인 결과:

- mLua 확장 `1.1.7`로 프로젝트 mLua 638개 분석
- 변경된 핵심 스크립트 7개 전체 진단: diagnostic 0건
- `RootDesk` 사용자 스크립트 17개 구문 진단: 0건
- Map, UI, Sprite, Model 등 JSON 리소스 54개 파싱 성공
- 중복 EntryKey 0건
- Custom mLua와 `.codeblock` 대응 17/17
- 로컬 `main` 기준 텍스트 병합 충돌 없음

저장소에는 실행 가능한 CLI 빌드·테스트 진입점이 없다. 따라서 위 결과는 정적 진단과 리소스 무결성 기준이며, MapleStory Worlds Maker의 실제 빌드 및 다음 Play Test가 별도로 필요하다.

### 필수 Play Test 시나리오

1. LobbyScene 진입 시 로비 UI와 Fade 상태 확인
2. Single 버튼 → SelectScene 이동 확인
3. 슬롯 4개의 순서, 이동, 설명 갱신 확인
4. 팝업 표시 중 좌우 입력 차단 및 취소 상태 복구 확인
5. `Button_Game` 확인 → 사용자별 Instance Room 입장 확인
6. 없는 `TestScene`을 참조하는 게임 선택 시 실패 로그 확인
7. SelectScene 재진입 시 Widget·버튼 이벤트 중복 여부 확인
8. Room 입장 후 `CurrentGameId` 서버 복원 여부 확인

## 11. 코드 변경 시 문서 동기화 체크리스트

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
