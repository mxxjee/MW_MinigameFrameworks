# 미니게임 프레임워크 개요

이 문서는 현재 체크아웃된 코드와 저장소에서 확인 가능한 Scene(Map)·UI 파일을 기준으로 작성한 온보딩 요약이다. Maker에서만 확인할 수 있는 세부 Entity hierarchy는 추측하지 않으며, 코드로 연결이 확인되지 않은 부분은 구현 예정 또는 미연결 상태로 구분한다.

## 1. 프레임워크 개요

현재 프레임워크는 `_MinigameRegistry`가 미니게임의 이름·ID·이동할 Scene(Map) 이름을 메타데이터로 등록하고, Lobby에서 선택한 플레이 모드를 `_GameLogic`이 보관한 뒤 `SelectScene`에서 그 메타데이터를 이용해 선택 UI를 구성하는 형태다.

선택된 게임을 실제로 실행할 공통 인터페이스인 `MinigameComponent`와 실행 생명주기를 담당할 `_MinigameManager`의 뼈대도 존재한다. 다만 현재 선택 UI는 좌우 탐색까지만 구현되어 있으며, 선택 확정부터 실제 미니게임 Component 연결까지는 아직 이어지지 않았다.

| 요소 | 현재 역할 |
| --- | --- |
| `_GameEnum` | `PlayMode.SINGLE = 0`, `PlayMode.MULTI = 1`을 클라이언트 시작 시 구성한다. |
| `_GameLogic` | 플레이 모드, 선택된 게임 ID, 점수, 제한 시간 등 상위 게임 상태와 선택·시작·종료 진입점을 관리한다. |
| `_MinigameRegistry` | `MinigameData`를 생성해 게임 ID를 Key로 등록하고 조회한다. 현재 `@Logic`이다. |
| `MinigameData` | 게임의 `Name`, `ID`, `MapName`을 보관하는 `@Struct`다. |
| `MinigameComponent` | 실제 미니게임 구현체가 공통으로 사용할 `Initialize`, `Update`, `Release` 인터페이스를 제공한다. |
| `_MinigameManager` | 현재 게임 Component, 경과 시간, 실행 여부를 기준으로 미니게임 생명주기를 관리하도록 설계되어 있다. 현재 실제 Component 연결은 미완성이다. |
| `MapEnterController` | `SelectScene` 진입을 감지해 선택 Controller를 초기화하고 선택 UI를 활성화한다. |
| `SelectSceneControllerComponent` | Registry 데이터로 선택 Widget을 생성하고 현재 선택 인덱스와 슬라이드 이동을 관리한다. |
| `SelectWidgetUIComponent` | 생성된 Widget에 게임 이름과 가로 배치 위치를 적용한다. |

큰 흐름은 다음과 같다.

```text
_MinigameRegistry
    └─ MinigameData(Name / ID / MapName) 등록
                    ↓
Lobby에서 PlayMode 선택 → _GameLogic에 저장
                    ↓
SelectScene 이동 → Registry 데이터로 선택 Widget 구성
                    ↓
[현재 미연결]
선택 확정 → 게임 Scene(Map) 이동 → 실제 MinigameComponent 연결
                    ↓
_MinigameManager가 Initialize / Update / Release 관리
```

## 2. 현재 Scene(Map) 구성

저장소에서 확인되는 미니게임 프레임워크 관련 Map 파일은 `LobbyScene.map`, `SelectScene.map`, `ButtonGameScene.map`이다.

### Lobby

`LobbyScene`은 플레이 방식을 고르는 시작 공간이다. `LobbyGroup.ui`의 실제 구성은 다음과 같다.

- `/ui/LobbyGroup/SingleButton`: `EnterBtnComponent`의 `Mode = 0`
- `/ui/LobbyGroup/MultiButton`: `EnterBtnComponent`의 `Mode = 1`
- 두 Entity 모두 `ButtonComponent`와 `EnterBtnComponent`를 사용한다.

`EnterBtnComponent.HandleButtonClickEvent()`는 `Mode`가 `_GameEnum.PlayMode.SINGLE`일 때 다음을 수행한다.

```text
SINGLE 클릭
    ↓
_GameLogic:SetPlayMode(0)
    ↓
LocalPlayer를 SelectScene의 (0, 0, 0)으로 Teleport
    ↓
Lobby UI 비활성화
```

`MULTI` 분기에는 현재 매칭을 시도한다는 주석만 있고, 플레이 모드 설정·매칭·Scene 이동은 구현되어 있지 않다. 또한 클릭 처리 마지막에 `/ui/SelectButtonGroup`을 비활성화하지만 현재 선택 화면에서 실제로 사용하는 경로는 `/ui/SelectGroup`이므로, 이 경로는 이후 정리가 필요한 상태다.

### SelectScene

`MapEnterController.OnMapEnter()`는 진입한 Map 이름이 `SelectScene`이면 같은 Entity의 `SelectSceneControllerComponent.Initialize()`를 호출하고 `/ui/SelectGroup`을 활성화한다.

`SelectSceneControllerComponent.Initialize()`는 `/ui/SelectGroup/GameList`를 찾아 위치와 선택 상태를 초기화한 뒤 `Refresh_Slots()`를 호출한다. `Refresh_Slots()`는 `_MinigameRegistry.MinigameDatas`를 순회해 `SelectWidgetModelId`의 Widget Model을 `GameList` 자식으로 동적 생성한다.

현재 `SelectGroup.ui`의 저장소 구성은 `SelectGroup` 아래 배경 `BG`와 Widget 부모 `GameList`가 있는 형태다. 생성된 Widget의 `SelectWidgetUIComponent.Initialize(gameName, pos)`가 표시 이름과 배치 위치를 설정한다.

### Minigame Scene(Map)

첫 번째 미니게임용 `ButtonGameScene.map` 파일은 존재하며, Registry의 `Button_Game` 데이터도 이 Map 이름을 가리킨다. 그러나 이 Scene(Map)에 연결될 `ButtonMinigameComponent`는 아직 존재하지 않고, `_MinigameManager`가 Scene의 실제 게임 Component를 얻는 연결도 구현되지 않았다.

나머지 세 Registry 항목은 현재 모두 `TestScene`을 가리키지만 저장소의 Map 목록에는 `TestScene.map`이 없다. 따라서 현재 실제 Map 파일과 일치하는 등록 항목은 `Button_Game → ButtonGameScene`이다.

## 3. 게임 데이터 저장과 관리

`_MinigameRegistry`는 실제 `MinigameComponent` 인스턴스를 저장하는 곳이 아니다. 게임 선택과 Scene(Map) 이동에 필요한 메타데이터를 `MinigameData`로 만들어 `MinigameDatas` 테이블에 보관한다.

`MinigameData`의 현재 필드는 다음 세 가지다.

| 필드 | 의미 |
| --- | --- |
| `Name` | 선택 Widget에 표시할 게임 이름 |
| `ID` | Registry 조회와 게임 선택에 사용할 고유 ID |
| `MapName` | 선택 후 이동할 MSW Scene(Map) 이름 |

등록 흐름은 다음과 같다.

```text
_MinigameRegistry.OnBeginPlay() [ClientOnly]
    ↓
RegisterMinigame(name, id, mapName)
    ↓
ValidateRegisterData(id)
    ├─ 빈 ID 거부
    └─ 중복 ID 거부
    ↓
MinigameData() 생성
    ↓
InitData(name, id, mapName)
    ↓
MinigameDatas[id] = gameData
```

현재 등록 데이터는 다음과 같다.

| Name | ID | MapName | 저장소 Map 존재 여부 |
| --- | --- | --- | --- |
| 버튼게임 | `Button_Game` | `ButtonGameScene` | 있음 |
| 민거니 먹빵 | `GunHee_Muckbang` | `TestScene` | 없음 |
| 민지꾸얌 | `Minji_Game` | `TestScene` | 없음 |
| 쥬신뿌수기 | `Jusin_Game` | `TestScene` | 없음 |

조회는 `_MinigameRegistry:GetMinigameData(id)`로 수행하며, 등록되지 않은 ID는 `nil`을 반환한다.

## 4. SINGLE 버튼 이후 전체 데이터·Scene 흐름

현재 코드에서 실제로 연결된 호출 순서는 다음과 같다.

```text
[플레이어]
/ui/LobbyGroup/SingleButton 클릭
    ↓
[EnterBtnComponent]
HandleButtonClickEvent()
    ↓
Mode가 _GameEnum.PlayMode.SINGLE인지 확인
    ↓
[_GameLogic]
SetPlayMode(self.Mode)
    └─ _GameLogic.PlayMode = SINGLE(0)
    ↓
[_TeleportService]
LocalPlayer를 SelectScene으로 이동
    ↓
[MapEnterController]
OnMapEnter(enteredMap)
    ↓ enteredMap.Name == "SelectScene"
[SelectSceneControllerComponent]
Initialize()
    ├─ /ui/SelectGroup/GameList 획득
    ├─ CurrentIndex와 이동 상태 초기화
    └─ Refresh_Slots()
            ↓
    _MinigameRegistry.MinigameDatas 순회
            ↓
    선택 Widget Model 생성 및 게임 이름 설정
```

화면 전환과 별개로 데이터는 다음과 같이 이어진다.

```text
SINGLE 선택값
    → _GameLogic.PlayMode에 저장

등록된 전체 게임 정보
    → _MinigameRegistry.MinigameDatas에 MinigameData로 저장
    → SelectSceneControllerComponent가 읽어 UI 구성

현재 선택 위치
    → SelectSceneControllerComponent.CurrentIndex에 저장
```

현재 흐름은 좌우 키로 `CurrentIndex`를 바꾸는 지점까지다. 선택을 확정해 `GameList[CurrentIndex]`의 게임 ID를 `_GameLogic:StartMinigame(gameId)`에 전달하는 호출은 아직 없다.

`_GameLogic:StartMinigame()` 자체는 `SelectMinigame()`으로 `CurrentGameId`를 저장하고 Registry에서 `MinigameData`를 조회한 뒤 `MapName`으로 이동하며 `_MinigameManager:StartGame()`을 호출하도록 작성되어 있다. 하지만 SelectScene UI에서 이 함수로 진입하는 연결이 없고, Manager의 실제 게임 Component 연결도 미완성이므로 현재 완성된 실행 흐름으로 보아서는 안 된다.

## 5. SelectScene 미니게임 선택 UI 구조

Widget 생성 흐름은 다음과 같다.

```text
_MinigameRegistry.MinigameDatas
    ↓ pairs로 순회
MinigameData를 GameList 테이블에 저장
    ↓
SelectWidgetModelId로 Widget Model 생성
    └─ 부모: /ui/SelectGroup/GameList
    ↓
GetComponent("script.SelectWidgetUIComponent")
    └─ local 정적 타입: SelectWidgetUIComponent
    ↓
Initialize(gameData.Name, pos)
    ├─ 자식 "Name"의 텍스트 설정
    └─ Widget의 anchoredPosition 설정
```

Widget은 `SlotSpacing = 300` 간격으로 `(0, 0)`, `(300, 0)`, `(600, 0)`처럼 처음부터 가로로 배치된다. 개별 Widget을 매번 움직이지 않고 부모 `GameList` 전체를 반대 방향으로 이동시켜 현재 Widget을 화면 중앙 `x = 0`에 맞춘다.

- `CurrentIndex`: 현재 선택 위치이며 1부터 시작한다.
- `MaxIndex`: 생성 대상 게임 수를 기준으로 좌우 이동 범위를 제한한다.
- `TargetPosition.x`: `-(CurrentIndex - 1) * SlotSpacing`으로 계산한다.
- `HandleKeyDownEvent()`: `LeftArrow`와 `RightArrow` 입력으로 인덱스와 목표 위치를 변경한다.
- `OnUpdate()`: `Vector2.Lerp`로 `GameList`를 목표 위치까지 부드럽게 이동한다.
- 이동 중 새 좌우 입력이 들어오면 `CurrentIndex`와 `TargetPosition`이 다시 계산되어 새 목표를 향해 계속 이동한다.

현재 Registry 순회가 `pairs()` 기반이므로 Widget 표시 순서는 고정 배열 순서로 보장되지 않는다.

## 6. 현재 실행 구조

### `_GameLogic`

프레임워크의 상위 상태와 실행 진입점을 담당한다.

- `PlayMode`: 현재 SINGLE/MULTI 모드
- `CurrentGameId`: 현재 선택한 게임 ID
- `CurrentScore`: 현재 점수
- `CurrentMaxTime`: 제한 시간
- `SelectMinigame()`: 게임 ID 저장, Registry 조회, 해당 Map으로 이동
- `StartMinigame()`: 선택 처리 후 `_MinigameManager:StartGame()` 요청
- `EndMinigame()`: `_MinigameManager:EndGame()` 요청
- `OnUpdate()`: Manager 갱신과 제한 시간 종료 판단

상대 플레이어 ID와 랭킹 데이터용 상태도 존재하지만, 현재 Lobby의 MULTI 흐름과는 연결되어 있지 않다.

### `_MinigameRegistry`

클라이언트 시작 시 미니게임 메타데이터를 등록하고 ID로 조회하는 전역 `@Logic`이다. 실제 게임 Component나 실행 상태는 저장하지 않는다.

### `MinigameData`

게임을 식별하고 선택 UI를 구성하며 이동할 Scene(Map)을 찾기 위한 `Name`, `ID`, `MapName` 데이터다.

### `MinigameComponent`

실제 미니게임 구현체가 공통으로 사용할 Base Component다. 현재 인터페이스는 다음과 같고 기본 구현은 비어 있다.

- `Initialize(MinigameData gameData)`
- `Update(number deltaTime)`
- `Release()`

### `_MinigameManager`

`CurrentGame: MinigameComponent`, `CurrentTime`, `IsPlaying`을 가지고 실행 생명주기를 관리하도록 작성되어 있다.

- `StartGame()`: Registry에서 데이터를 조회하고 시간·실행 상태를 초기화한 뒤 `CurrentGame:Initialize()` 호출 예정
- `Update()`: 실행 중이면 시간을 누적하고 `CurrentGame:Update()` 호출
- `EndGame()`: `CurrentGame:Release()` 호출 후 상태 초기화

현재 `StartGame()`에는 `CurrentGame`을 실제 Component로 설정하는 코드가 없고 관련 할당 부분이 주석 상태다. 그 상태에서 `CurrentGame:Initialize(gameData)`를 호출하므로, 실제 실행 연결은 완료되지 않았다.

## 7. 전체 실행 흐름 한눈에 보기

```text
[ LobbyScene ]
SingleButton (Mode = 0)
    ↓ ButtonClickEvent
EnterBtnComponent.HandleButtonClickEvent()
    ↓
_GameLogic:SetPlayMode(SINGLE)
    ↓
SelectScene으로 Teleport

==============================

[ SelectScene ]
MapEnterController.OnMapEnter()
    ├─ SelectSceneControllerComponent.Initialize()
    │       └─ GameList 초기화
    └─ /ui/SelectGroup 활성화
            ↓
_MinigameRegistry.MinigameDatas
            ↓
선택 Widget Model 동적 생성
            ↓
GameList에 가로 배치
            ↓
← / → 입력
            ↓
CurrentIndex와 TargetPosition 변경
            ↓
GameList 전체를 Lerp 이동
            ↓
[현재 구현은 여기까지]

==============================

[ 다음 연결 예정 ]
현재 선택 게임 확정
    ↓
선택된 GameId로 MinigameData 조회
    ↓
MinigameData.MapName의 Scene(Map)으로 이동
    ↓
Scene의 실제 ButtonMinigameComponent 획득
    ↓
_MinigameManager.CurrentGame에 연결
    ↓
Initialize → Update → Release
```

## 8. 현재 구현 완료 범위

아래는 런타임 Play Test 결과가 아니라, 현재 저장소 코드에서 구현이 확인된 범위다.

- `_GameEnum`의 SINGLE/MULTI 값 구성
- Lobby의 SINGLE/MULTI 버튼과 `EnterBtnComponent` 연결
- SINGLE 선택값을 `_GameLogic.PlayMode`에 저장
- SINGLE 버튼에서 `SelectScene` 이동 요청
- `SelectScene` 진입 감지와 `/ui/SelectGroup` 활성화
- `_MinigameRegistry`의 미니게임 메타데이터 등록·ID 조회
- `MinigameData`의 `Name` / `ID` / `MapName` 저장
- Registry 데이터 기반 선택 Widget 동적 생성
- Widget 게임 이름 설정과 일정 간격의 가로 배치
- 좌우 키에 따른 `CurrentIndex` 범위 제한 및 변경
- 부모 `GameList`의 Lerp 슬라이드 이동
- `MinigameComponent` 공통 생명주기 인터페이스
- `_MinigameManager`의 시간·실행 상태·생명주기 관리 뼈대

현재 미구현·미연결 항목은 다음과 같다.

- MULTI 모드의 설정, 매칭, 이동 흐름
- SelectScene에서 현재 Widget을 실제 게임으로 확정하는 입력·호출
- 선택 UI에서 `_GameLogic:StartMinigame(gameId)`로 이어지는 연결
- Scene(Map)에서 실제 `MinigameComponent` 구현체를 찾아 `_MinigameManager.CurrentGame`에 저장하는 처리
- `ButtonMinigameComponent` 구현
- `ButtonGameScene`의 실제 미니게임 시작·진행·종료 사이클
- Registry의 `TestScene` 항목과 실제 Map 파일 간 불일치 해소
- `/ui/SelectButtonGroup`과 실제 `/ui/SelectGroup` 경로 차이 정리

## 9. 다음 구현 예정: `ButtonMinigameComponent`

다음 단계에서는 첫 번째 실제 미니게임인 `ButtonMinigameComponent`를 구현하고, 현재 분리된 선택 흐름과 Manager 생명주기를 연결한다.

목표 흐름은 다음과 같다.

```text
SelectScene에서 현재 게임 선택 확정
    ↓
CurrentIndex에 해당하는 GameId 확인
    ↓
_MinigameRegistry:GetMinigameData(gameId)
    ↓
MinigameData.MapName으로 ButtonGameScene 이동
    ↓
ButtonGameScene의 ButtonMinigameComponent 연결
    ↓
_MinigameManager.CurrentGame에 MinigameComponent 타입 구현체로 연결
    ↓
Initialize(gameData)
    ↓
Update(deltaTime)
    ↓
Release()
```

즉, Base인 `MinigameComponent`와 `_MinigameManager`를 실제 구현체에 연결해 **선택 → Scene(Map) 이동 → 게임 시작 → 게임 진행 → 게임 종료**의 첫 완전한 사이클을 만드는 것이 다음 작업 목표다. 이 흐름과 `ButtonMinigameComponent`는 현재 구현 완료 상태가 아니라 다음 구현 예정이다.
