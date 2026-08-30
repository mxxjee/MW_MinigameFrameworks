# 프로젝트 코드 구조 및 현재 구현 상태

> 이 문서는 현재 프로젝트의 실제 경로, 클래스 책임, 리소스 연결, 실행 흐름과 미완성 지점을 빠르게 파악하기 위한 기준 문서다.
>
> **유지보수 규칙:** mLua 코드나 관련 UI·맵 구성을 생성·수정·삭제·이동할 때는 같은 작업에서 이 문서도 반드시 갱신한다. 문서와 코드가 다르면 실제 코드를 기준으로 바로잡는다.

- 마지막 점검일: 2026-08-30
- 확인 브랜치: `Minji_Branch`
- 확인 HEAD: `fd391f5` (`Fix: 머지 완료`) + 현재 작업 트리
- 분석 기준: 현재 작업 트리의 mLua, Map, UI, Model, Sprite 리소스
- 현재 단계: **로비/선택/대기실 → Instance Room 입장 → 인트로 준비 → 공통 서버 시각 기반 실제 게임 시작·종료 → 남은 시간 UI 표시** 흐름 연결 중
- 아직 미완성: 준비 신호의 사용자별 중복 방지·라운드 검증, 구체 미니게임 본체, 등록 게임의 실제 로고 RUID

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
│     ├─ MinigameEndLogic.mlua
│     ├─ MinigameIntroLogic.mlua
│     ├─ MinigameManager.mlua
│     └─ MinigameRegistry.mlua
├─ Minigame/
│  ├─ DropGameComponent.mlua
│  ├─ MinigameComponent.mlua
│  └─ MinigameData.mlua
├─ SelectScene/
│  ├─ SelectSceneControllerComponent.mlua
│  └─ SelectWidgetUIComponent.mlua
├─ UI/
│  ├─ EnterBtnComponent.mlua
│  ├─ MinigameIntroUIComponent.mlua
│  └─ WaitingRoomMyAvatarComponent.mlua
├─ Events/
│  ├─ MinigameIntroReadyEvent.mlua
│  ├─ MinigameLogoEndEvent.mlua
│  └─ MinigameCountdownEndEvent.mlua
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

`GameLogic`은 `RootDesk/MyDesk/Logics/DefaultLogic/GameLogic.mlua`에 있다. `MinigameComponent`와 `MinigameData`는 `RootDesk/MyDesk/Minigame/`에 유지된다.

| 파일 | 선언 | 전역 접근/부착 위치 | 현재 책임 |
|---|---|---|---|
| `Logics/DefaultLogic/GameLogic.mlua` | `@Logic script GameLogic` | `_GameLogic` | 플레이 모드·점수·제한시간·준비 인원, 공통 서버 시작/종료 시각과 남은 시간 관리 |
| `Logics/DefaultLogic/GameEnum.mlua` | `@Logic script GameEnum` | `_GameEnum` | 플레이 모드 enum 테이블 초기화 |
| `Logics/DefaultLogic/GameHelper.mlua` | `@Logic script GameHelper` | `_GameHelper` | 경로로 UI Group을 찾아 활성화/비활성화 |
| `Logics/MinigameLogic/MinigameRegistry.mlua` | `@Logic script MinigameRegistry` | `_MinigameRegistry` | 미니게임 이름·ID·맵 경로의 전역 등록 및 조회 |
| `Logics/MinigameLogic/MinigameManager.mlua` | `@Logic script MinigameManager` | `_MinigameManager` | 로컬 미니게임 Component와 서버 시작 시각 기준 경과시간·실행 여부 관리 |
| `Logics/MinigameLogic/MinigameIntroLogic.mlua` | `@Logic script MinigameIntroLogic` | `_MinigameIntroLogic` | 로컬 인트로 상태 관리, UI Component 획득, 로고/카운트다운 이벤트 연결 |
| `Logics/MinigameLogic/MinigameEndLogic.mlua` | `@Logic script MinigameEndLogic` | `_MinigameEndLogic` | Finish UI 시작·종료 상태 관리와 클라이언트별 애니메이션 완료 서버 보고 |
| `Minigame/MinigameData.mlua` | `@Struct script MinigameData` | `MinigameData()` | 미니게임 이름·ID·MapName 저장 |
| `Minigame/MinigameComponent.mlua` | `@Component script MinigameComponent` | 엔티티 부착/상속 기반 | 미니게임 초기화·갱신·해제 인터페이스 |
| `UI/MinigameIntroUIComponent.mlua` | `@Component` | `/ui/GameIntroGroup` | 로고·카운트다운·START·Finish Sprite 연출과 애니메이션 종료 처리 |
| `Events/MinigameLogoEndEvent.mlua` | `@Event` | `GameIntroGroup` Entity Event | 로고 Fade Out 완료 알림 |
| `Events/MinigameCountdownEndEvent.mlua` | `@Event` | `GameIntroGroup` Entity Event | START 이미지 출력 완료 알림 |
| `Events/MinigameIntroReadyEvent.mlua` | `@Event` | `_GameLogic` Self Event | Room 동기화와 미니게임 Component 준비 상태 재검사 트리거 |
| `ETC/UIToast.mlua` | `@Logic script UIToast` | `_UIToast` | 기존 Toast 표시와 게임 중 지속형 남은 시간 텍스트 표시 |
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
| `PlayMode` | `string` | `"NONE"` | `NONE`, `SINGLE`, `MULTI` |
| `CurrentScore` | `number` | `0` | 현재 점수 |
| `CurrentMaxTime` | `number` | `60` | 제한시간 |
| `MyPlayerId` | `integer` | `0` | 로컬 플레이어 ID 저장 자리 |
| `OpponentPlayerId` | `string` | `""` | 상대 플레이어 ID |
| `MyRankingData` | `table` | `{}` | 랭킹 데이터 저장 자리 |
| `introReadyCnt` | `integer` | `0` | 서버에서 받은 인트로 준비 신호 수 |
| `GameReadyCnt` | `integer` | `0` | 서버에서 받은 실제 게임 준비 신호 수 |
| `EndUIFinishedCnt` | `integer` | `0` | Finish Sprite 재생 완료를 보고한 클라이언트 수 |
| `GameEndTimerId` | `integer` | `0` | 서버의 공통 종료 신호 Timer ID |
| `ReserveServerTime` | `number` | `0` | 클라이언트가 실제 게임을 시작할 공통 서버 시각 |
| `GameEndServerTime` | `number` | `0` | `ReserveServerTime + CurrentMaxTime`으로 계산한 공통 종료 시각 |
| `RemainingTime` | `number` | `0` | `GameEndServerTime - ServerElapsedSeconds`로 계산한 남은 시간 |
| `LastDisplayedRemainingTime` | `number` | `-1` | UI에 마지막으로 표시한 정수 초 |
| `IsReservedGamePlay` | `boolean` | `false` | 공통 서버 시작 시각 대기 여부 |
| `IsGameEnding` | `boolean` | `false` | 클라이언트가 공통 종료 단계에 진입했는지 여부 |
| `IsFinishGameProcessed` | `boolean` | `false` | 서버 게임 종료 처리를 한 번만 수행하기 위한 상태 |

#### 룸 관련 Method

| 메서드 | ExecSpace | 현재 동작 |
|---|---|---|
| `SelectMinigame(string gameID, table userIds)` | `Server` | 현재 Instance Room을 유지하며 지정 사용자들을 선택 게임 맵으로 이동한다. |
| `EnterMiniGameRoom(table gameIds, string roomKey, table userIds, string PlayMode)` | `Server` | SharedMemory에 첫 게임 ID와 PlayMode를 저장하고 Instance Room을 만든 뒤 첫 맵으로 이동한다. |
| `MakeSoloRoomKey(string userId)` | `Server` | `Solo_{userId}_{UtcNow.Elapsed}` 형식의 키를 만든다. |
| `HandleRoomBeginEvent(RoomBeginEvent)` | Server Event | SharedMemory의 `CurrentGameId`와 `PlayMode`를 복원하고 클라이언트 PlayMode 설정과 인트로 준비 검사를 요청한다. |

#### 상태·생명주기 Method

| 메서드 | 현재 동작 |
|---|---|
| `Set_PlayMode(string playMode)` | Client 실행 Method. 문자열 `SINGLE/MULTI`만 허용한다. |
| `SetCurrentScore/AddScore/ResetScore` | 점수를 설정·가산·초기화한다. |
| `SetCurrentMaxTime(number maxTime)` | 0보다 큰 제한시간만 허용한다. |
| `SetOpponentPlayer/ClearOpponentPlayer` | 상대 ID를 설정·초기화한다. |
| `Request_ReadyIntro()` | Client에서 `MinigameIntroReadyEvent`를 발생시킨다. |
| `ReportIntroReady_Server()` | 서버 인트로 준비 수가 모드별 필요 인원에 도달하면 룸 클라이언트에 `PlayIntro()`를 호출한다. |
| `PlayIntro(string gameId)` | Client 실행 Method. `_MinigameIntroLogic:BeginIntro(gameId)`에 인트로를 위임한다. |
| `ReportGameReady_Server()` | START 애니메이션 완료 준비 수가 모이면 Registry의 `MaxTime`을 `CurrentMaxTime`에 연결하고 시작·종료 서버 시각을 계산한다. |
| `ReserveGameStart_Client(number startServerTime, number endServerTime, number maxTime)` | 룸 클라이언트에 동일한 시작·종료 시각과 제한시간을 저장한다. |
| `StartGamePlay(string gameId, number startServerTime)` | 공통 시작 시각과 함께 `_MinigameManager:StartGame(gameId, startServerTime)`을 호출한다. |
| `EndMinigame()` | Manager를 해제하고 예약/시간 UI 상태를 초기화한다. |
| `BeginGameEnd_Client()` | 공통 종료 시각에 로컬 게임을 정지하고 `_MinigameEndLogic:BeginEndUI()`로 Finish UI를 시작한다. |
| `FinishGame_Server()` | 공통 종료 Timer가 호출하는 서버 종료 진입점. 라운드 승패·점수 확정은 추후 이 위치에 추가한다. |
| `ReportEndUIFinished_Server()` | Finish Sprite 종료 보고를 SINGLE 1개/MULTI 2개까지 모은 뒤 `ResolveNextGame_Server()`를 호출한다. |
| `ResolveNextGame_Server()` | PlayMode에 따라 SINGLE/MULTI 다음 게임 판정 함수로 분기한다. |
| `ResolveSingleNextGame_Server()` | SINGLE 다음 게임 진행 여부와 이동 대상을 판정할 자리다. 현재는 로그만 남긴다. |
| `ResolveMultiNextGame_Server()` | MULTI 라운드 결과·다음 게임·최종 결과 이동 여부를 판정할 자리다. 현재는 로그만 남긴다. |
| `MoveRoomUsersToScene_Server(string nextMapName, table userIds)` | 현재 RoomKey로 `MoveUsersToInstanceRoom()`을 호출해 Instance Room을 유지하며 지정 Scene으로 이동한다. |
| `OnUpdate(number delta)` | ClientOnly에서 공통 시작 시각을 기다리고, 종료 시각으로 남은 시간을 계산하며 Manager와 남은 시간 UI를 갱신한다. |

서버 `CurrentGameId`는 RoomBegin 또는 미니게임 선택 시 갱신되고, 각 클라이언트의 `CurrentGameId`는 해당 맵의 `MinigameComponent.OnBeginPlay()`가 자신의 `MinigameID`로 설정한다.

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
| 1 | 똥피하기 | `Drop_Game` | `ButtonGameScene` | 버튼 연타 안내 | 있음 |
| 2 | 민거니 먹빵 | `GunHee_Muckbang` | `TestScene` | 민거니 소개 | **없음** |
| 3 | 민지꾸얌 | `Minji_Game` | `TestScene` | 생일 안내 | **없음** |
| 4 | 쥬신뿌수기 | `Jusin_Game` | `TestScene` | 쥬신 뿌수기 안내 | **없음** |

#### Method

| 메서드 | 현재 동작 |
|---|---|
| `OnBeginPlay()` | `Initialize()`를 호출한다. |
| `Initialize()` | 이미 초기화했으면 종료하고, 네 게임을 등록한 뒤 플래그를 `true`로 만든다. |
| `RegisterMinigame(name, id, mapName, desc, logoGUID, endType, maxTime)` | 검증 후 `MinigameData`를 만들고 Dictionary와 순서 Table에 저장한다. |
| `ValidateRegisterData(string id)` | 빈 ID와 중복 ID를 거부한다. |
| `GetMinigameData(string id)` | 데이터가 없다고 판단하면 초기화를 시도한 뒤 `MinigameData` 또는 `nil`을 반환한다. |

`MinigameDatas`는 문자열 키 Dictionary이므로 `#self.MinigameDatas`는 데이터 수 확인 용도로 신뢰할 수 없다. 현재 `GetMinigameData()`의 `#self.MinigameDatas == 0` 검사는 매번 참이 될 수 있지만, `bInitailized`가 실제 중복 등록은 막고 있다.

### 3.5 `MinigameManager`

| Property | 타입 | 기본값 | 역할 |
|---|---|---:|---|
| `CurrentGame` | `MinigameComponent` | `nil` | 현재 실행할 구체 미니게임 Component |
| `CurrentTime` | `number` | `0` | 공통 시작 서버 시각 기준 경과시간 |
| `GameStartServerTime` | `number` | `0` | 서버가 모든 룸 클라이언트에 전달한 실제 게임 시작 시각 |
| `IsPlaying` | `boolean` | `false` | 실행 여부 |

| 메서드 | 현재 동작 |
|---|---|
| `StartGame(string gameId, number startServerTime)` | 시작 서버 시각을 저장하고 현재 서버 시각과의 차이로 경과시간을 초기화한 뒤 `CurrentGame:Initialize(gameData)`를 호출한다. |
| `Update(number deltaTime)` | `ServerElapsedSeconds - GameStartServerTime`으로 경과시간을 재계산하고 이전 값과의 차이를 `CurrentGame:Update()`에 전달한다. |
| `EndGame()` | `Release()` 호출 후 Component·경과시간·시작 서버 시각·실행 상태를 초기화한다. |

현재는 `MinigameComponent.OnBeginPlay()`가 ClientOnly에서 자신을 `CurrentGame`으로 등록한다. `MinigameManager.StartGame()`도 Component가 등록되지 않은 경우 `false`를 반환하도록 방어한다.

### 3.6 `MinigameIntroLogic`

`MinigameIntroLogic`은 GameLogic과 UI 구현을 분리하는 Client 인트로 조정 계층이다.

| Property | 타입 | 기본값 | 역할 |
|---|---|---|---|
| `IntroUI` | `MinigameIntroUIComponent` | `nil` | 현재 클라이언트의 구체 인트로 UI Component |
| `CurrentGameId` | `string` | `""` | 현재 인트로가 진행 중인 게임 ID |
| `IntroState` | `string` | `"IDLE"` | `IDLE → LOGO → COUNTDOWN → COMPLETED` 단계 상태 |

`BeginIntro(gameId)`는 Registry에서 `MinigameData`를 구체 타입으로 조회하고 `/ui/GameIntroGroup`에서 `MinigameIntroUIComponent`를 구체 타입으로 얻는다. UI를 초기화한 뒤 `PlayLogo(gameData.LogoGUID)`를 호출한다. 진행 중 중복 호출은 거부해 Timer와 실제 게임 시작이 두 번 실행되지 않도록 한다.

`GameIntroGroup`은 기본 비활성 UI라 첫 활성화 시 `MinigameIntroUIComponent.OnBeginPlay()`가 인트로 호출보다 늦게 실행될 수 있다. 이때 로고를 다시 끄는 초기화 경합을 막기 위해 `OnBeginPlay()`에서는 자식 UI 상태를 변경하지 않고 카운트다운 애니메이션 종료 이벤트만 연결한다. `BeginIntro() → Reset() → PlayLogo()`가 상태를 전담하며, `PlayLogo()`와 `PlayCountdown()`은 대상 Entity의 `Visible`과 `Enable`을 함께 켜고 단계 종료와 `Reset()`에서는 둘을 함께 끈다.

Registry의 `LogoGUID`가 빈 문자열이면 `PlayLogo()`는 UI Editor에 설정된 기본 `ImageRUID`를 유지한다. 게임별 `LogoGUID`가 등록된 경우에만 해당 RUID로 교체한다.

카운트다운은 더 이상 `3`, `2`, `1` RUID를 Timer로 하나씩 교체하지 않는다. `CountDown` Entity에 설정된 단일 `3 → 2 → 1` 애니메이션 클립을 `SpriteAnimClipPlayType.Onetime`으로 첫 프레임부터 재생하고, `SpriteGUIAnimPlayerEndEvent`를 실제로 받은 시점에 `CountdownStartRUID`로 교체해 START 이미지를 표시한다. START 표시 시간(`StartImageDuration`)이 끝나면 `MinigameCountdownEndEvent`를 전송한다. `CountdownStartRUID`는 `GameIntroGroup`의 `MinigameIntroUIComponent` Inspector에서 실제 START 이미지 RUID를 연결해야 한다.

이벤트 수신 대상은 `GameIntroGroup` Entity ID `8f88597a-523d-471b-a665-86636c31ff2e`다.

```text
_GameLogic:StartGame(gameId)
  → _GameLogic:PlayIntro(gameId)
  → _MinigameIntroLogic:BeginIntro(gameId)
  → MinigameIntroUIComponent:PlayLogo(LogoGUID)
  → 로고 Fade Out 완료
  → MinigameLogoEndEvent
  → MinigameIntroLogic: HandleMinigameLogoEndEvent
  → MinigameIntroUIComponent:PlayCountdown()
  → 단일 3 → 2 → 1 Sprite Animation 재생
  → SpriteGUIAnimPlayerEndEvent
  → START 이미지
  → MinigameCountdownEndEvent
  → MinigameIntroLogic: HandleMinigameCountdownEndEvent
  → _GameLogic:ReportGameReady_Server()
  → 모드별 필요 인원 GameReady 확인
  → startServerTime = ServerElapsedSeconds + 0.5
  → endServerTime = startServerTime + CurrentMaxTime
  → _GameLogic:ReserveGameStart_Client(startServerTime, endServerTime, maxTime)
  → 각 클라이언트 OnUpdate가 공통 startServerTime 도달 확인
  → _GameLogic:StartGamePlay(gameId, startServerTime)
  → _MinigameManager:StartGame(gameId, startServerTime)
  → RemainingTime = endServerTime - ServerElapsedSeconds
  → _UIToast:ShowRemainingTime(정수 초)
  → 공통 종료 시각에 로컬 종료 및 서버 EndMinigame_Client 신호
```

SINGLE과 MULTI 모두 같은 준비 흐름을 사용한다. 서버는 SINGLE 1명, MULTI 2명의 인트로 준비와 실제 게임 준비 신호를 기다린다. 실제 게임 준비가 완료되면 현재 Room 전체에 동일한 시작·종료 서버 시각을 전달한다. 현재 준비 수는 단순 정수 카운트이므로 사용자별 중복 방지와 라운드 ID 검증은 후속 보완이 필요하다.

### 3.7 `MinigameEndLogic`

`MinigameEndLogic`은 `/ui/GameIntroGroup`의 기존 `MinigameIntroUIComponent`를 구체 타입으로 얻어 `PlayFinish()`를 호출한다. 실제 Finish Sprite Entity 경로는 `/ui/GameIntroGroup/Finish`다.

각 클라이언트의 Finish Sprite에서 `SpriteGUIAnimPlayerEndEvent`가 발생하면 `MinigameIntroUIComponent.OnFinishAnimationEnd()`가 `_MinigameEndLogic:NotifyFinishAnimationEnd()`를 호출한다. End Logic은 UI Group을 숨긴 뒤 `_GameLogic:ReportEndUIFinished_Server()`로 로컬 애니메이션 완료를 보고한다.

```text
공통 GameEndServerTime 도달
  → _GameLogic:BeginGameEnd_Client()
  → _MinigameManager:EndGame()
  → _MinigameEndLogic:BeginEndUI()
  → /ui/GameIntroGroup/Finish 단발 재생
  → SpriteGUIAnimPlayerEndEvent
  → _MinigameEndLogic:NotifyFinishAnimationEnd()
  → _GameLogic:ReportEndUIFinished_Server()
  → SINGLE 1개 / MULTI 2개 완료 보고 확인
  → _GameLogic:ResolveNextGame_Server()
  → ResolveSingleNextGame_Server() 또는 ResolveMultiNextGame_Server()
  → 필요할 때 MoveRoomUsersToScene_Server(nextMapName, userIds)
```

Scene 이동은 Client Teleport가 아니라 서버 `RoomService.MoveUsersToInstanceRoom()`을 사용하므로 현재 Instance Room을 유지한다. 실제 다음 Scene 호출은 SINGLE/MULTI 판정 구현 전까지 자동으로 수행하지 않는다.

## 4. 미니게임 데이터와 기반 Component

### 4.1 `MinigameData`

| 이름 | 타입 | 기본값 |
|---|---|---:|
| `Name` | `string` | `""` |
| `ID` | `string` | `""` |
| `MapName` | `string` | `""` |
| `Description` | `string` | `""` |
| `LogoGUID` | `string` | `""` |
| `EndType` | `integer` | `1` |
| `MaxTime` | `number` | `0` |

`InitData(name, id, mapName, Desc, logoGUID, endType, maxTime)`가 메타데이터와 종료 유형·제한시간을 저장한다. 실제 실행 Component 참조는 보관하지 않는다.

### 4.2 `MinigameComponent`

| 메서드 | 시그니처 | 현재 구현 |
|---|---|---|
| 초기화 | `Initialize(MinigameData gameData)` | 빈 기반 Method |
| 갱신 | `Update(number deltaTime)` | 빈 기반 Method |
| 해제 | `Release()` | 빈 기반 Method |
| 로컬 등록/인트로 준비 | `OnBeginPlay()` | ClientOnly에서 Manager에 자신을 등록하고 `CurrentGameId`를 설정한 뒤 `_GameLogic:Request_ReadyIntro()`를 호출한다. |

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
| `ETC/UIToast.mlua` | 시간·Alpha Tween Toast 표시와 ClientOnly 지속형 남은 시간 표시·숨김 |

이 파일들은 현재 경로 이동 대상이 아니며 `RootDesk/MyDesk/Default/`에 유지된다.

## 8. 현재 구현 상태와 남은 연결

- Registry는 전역 `@Logic`이며 별도 Entity/Component 연결이 필요하지 않다.
- Registry는 네 개의 미니게임 메타데이터를 등록한다.
- `MinigameData`는 이름·ID·맵·설명·로고와 `EndType`, `MaxTime`을 저장한다.
- 선택 화면은 Registry 데이터로 위젯을 생성하고 이름을 표시한다.
- 선택 위젯의 클릭·게임 ID 전달·`StartMinigame()` 연결은 아직 없다.
- `MinigameComponent.OnBeginPlay()`가 로컬 `MinigameManager.CurrentGame`을 설정한다.
- 인트로는 `MinigameLogoEndEvent → PlayCountdown → MinigameCountdownEndEvent → ReportGameReady_Server` 순서로 연결됐다.
- 카운트다운은 단일 `3 → 2 → 1` Sprite Animation의 `SpriteGUIAnimPlayerEndEvent`가 도착한 뒤 START 이미지로 전환한다.
- SINGLE 1명/MULTI 2명의 실제 게임 준비가 완료되면 서버가 동일한 시작·종료 시각을 룸 클라이언트에 전달한다.
- 클라이언트 경과시간은 `ServerElapsedSeconds - GameStartServerTime`, 남은 시간은 `GameEndServerTime - ServerElapsedSeconds`로 계산한다.
- 남은 정수 초는 기존 `ToastGroup`의 `TextComponent`에 `남은 시간 : N` 형식으로 표시하고 종료 시 숨긴다.
- 공통 종료 시각에는 `/ui/GameIntroGroup/Finish` Sprite를 각 클라이언트에서 단발 재생한다.
- Finish Sprite가 끝난 클라이언트가 서버에 완료를 보고하고, SINGLE 1명/MULTI 2명 완료 후 `ResolveNextGame_Server()`가 호출된다.
- 다음 Scene 이동은 `MoveRoomUsersToScene_Server()`로 현재 Instance Room을 유지한다. 자동 이동 대상 판정은 아직 구현하지 않았다.
- Registry에 등록된 현재 `LogoGUID`가 모두 빈 문자열이므로 실제 로고 리소스 연결이 필요하다.
- `MinigameIntroUIComponent.CountdownStartRUID`가 빈 문자열이면 START 이미지로 교체할 리소스가 없으므로 UI Editor Inspector에서 실제 START 이미지 RUID를 지정해야 한다.
- `GameLogic.SelectMinigame()`은 Registry 조회 결과가 `nil`인지 확인하기 전에 `GameData.MapName`을 사용한다.
- MULTI 플레이의 서버 매칭·상대 연결은 TODO다.
- 실제 점수 산정 규칙과 랭킹 데이터 스키마는 아직 없다.

## 9. 정적 타입 검사 상태

현재 프레임워크에서 확인한 구체 타입:

- `GameLogic.SelectMinigame()`의 `GameData`: `MinigameData`
- `MinigameRegistry.RegisterMinigame()`의 `gameData`: `MinigameData`
- `MinigameRegistry.GetMinigameData()` 반환: `MinigameData`
- `MinigameManager.CurrentGame`: `MinigameComponent`
- `MinigameManager.GameStartServerTime`: `number`
- `MinigameEndLogic.EndUI`: `MinigameIntroUIComponent`
- `GameLogic.CurrentMaxTime`, `ReserveServerTime`, `GameEndServerTime`, `RemainingTime`: `number`
- `GameLogic.GameEndTimerId`: `integer`
- `MinigameComponent.Initialize()` 매개변수: `MinigameData`
- `SelectSceneControllerComponent.widgetComp`: `SelectWidgetUIComponent`

현재 실행 코드의 `GetComponent()` 호출은 두 곳이다.

- `SelectSceneControllerComponent.widgetComp`: `SelectWidgetUIComponent`
- `MinigameIntroLogic.introUI`: `MinigameIntroUIComponent`

둘 다 획득한 사용자 정의 Component의 Custom Method 소유 타입과 local Annotation이 일치하며, `Component` 또는 `any`로 낮추지 않았다. `MinigameIntroLogic.IntroUI` Property도 저장 단계부터 `MinigameIntroUIComponent` 구체 타입이다.

mLua Extension `1.1.7`의 진단 엔진으로 `GameLogic`, `MinigameIntroLogic`, `MinigameManager`, `MinigameRegistry`, `MinigameComponent`, `DropGameComponent`, `MinigameData`, `MinigameIntroUIComponent`, `MinigameLogoEndEvent`, `MinigameCountdownEndEvent`를 분석했으며 diagnostic 0건, provider error 0건이었다.

로고 표시 수정 후 `GameLogic`, `MinigameIntroLogic`, `MinigameIntroUIComponent`를 다시 분석했으며 diagnostic 0건, provider error 0건이었다.

단일 카운트다운 Sprite Animation 종료 이벤트 방식으로 변경한 뒤에도 같은 세 파일을 mLua Extension `1.1.7` 진단 엔진으로 다시 분석했으며 diagnostic 0건, provider error 0건이었다.

Codex 환경에서는 사용자의 VS Code Problems 패널 자체를 직접 읽지 못했다. 따라서 “VS Code Problems 오류 없음”이라고 단정하지 않으며, 위 결과는 번들 mLua LSP 진단 기준이다.

2026-08-30 서버 시각 기반 시작·종료 및 남은 시간 UI 변경에서는 코드 검색과 `git diff --check`로 호출 시그니처·명시 타입·공백 오류를 검사했다. 현재 환경에서는 VS Code mLua Extension Problems와 Maker 런타임을 직접 실행하지 못했으므로 이 변경의 실제 진단/런타임 결과는 확인되지 않았다.

2026-08-30 Finish UI와 다음 게임 판정 흐름 추가 후 `GameLogic`, `MinigameEndLogic`, `MinigameIntroUIComponent`의 사용자 정의 Method 호출 타입을 정적으로 확인했다. `MinigameEndLogic.EndUI`와 `GetComponent("script.MinigameIntroUIComponent")` local은 모두 `MinigameIntroUIComponent` 구체 타입이며 `any` 또는 기본 `Component`로 낮추지 않았다. VS Code Problems와 Maker Sprite 종료 이벤트는 직접 확인하지 못했다.

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
