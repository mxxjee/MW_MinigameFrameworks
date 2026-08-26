# 2026-08-09 `Minji_Branch` 개발 요약

> 작성 기준: 2026-08-09 00:00~23:59(KST)에 현재 브랜치에 기록된 커밋과 실제 변경 내역  
> 작성 시점 HEAD: `8042a10` (`ADD: 페이드인아웃`)

## 한눈에 보기

| 항목 | 내용 |
|---|---|
| 브랜치 | `Minji_Branch` |
| 오늘 커밋 | 7개 |
| 변경 파일 | 19개 |
| 전체 변경량 | 2,096줄 추가 / 271줄 삭제 |
| 작업 트리 | 요약 문서 생성 전 기준, 커밋되지 않은 변경 없음 |
| 주요 작업 영역 | 미니게임 선택 UI, Registry 초기화, 싱글 플레이 룸 분리, 화면 전환 |

> 전체 변경량에는 에디터가 직렬화한 `.map`, `.ui`, 로그 파일의 변경이 포함되어 있어 실제 로직 코드량보다 크게 표시됩니다.

## 핵심 성과

### 1. 미니게임 선택 순서 고정

- `MinigameRegistry`에 `MinigameOrders`를 추가했습니다.
- 순서가 보장되지 않는 `MinigameDatas` 순회를 대신해 등록 순서대로 슬롯을 생성하도록 변경했습니다.
- 선택 화면에서 미니게임이 실행할 때마다 다른 순서로 보일 수 있던 문제를 해결했습니다.

### 2. 선택 화면 정보와 조작 개선

- `MinigameData`에 `Description` 필드를 추가하고 각 미니게임 설명을 등록했습니다.
- 선택 화면 진입 시 첫 번째 게임의 설명을 표시하고, 좌우 이동이 끝날 때 현재 게임의 설명으로 갱신합니다.
- 슬롯 이동 속도를 조정하고 Y축 위치를 유지한 채 X축만 이동하도록 보완했습니다.
- 선택 화면 관련 스크립트를 `RootDesk/MyDesk/SelectScene/` 폴더로 정리했습니다.

### 3. 게임 선택 확인 팝업 추가

- `Space` 키 입력 시 현재 게임 이름이 포함된 확인 팝업을 표시합니다.
- 확인/취소 버튼 이벤트를 선택 화면 컨트롤러에 연결했습니다.
- 팝업이 열린 동안 좌우 방향키 입력을 차단해, 확인 대상과 실제 선택 항목이 달라지는 문제를 방지했습니다.
- 취소하면 팝업 상태와 UI를 함께 해제합니다.

### 4. Registry 초기화 시점 안정화

- `MinigameRegistry.Initialize()`와 초기화 여부 플래그를 추가해 중복 등록을 방지했습니다.
- `SelectScene`에서 바로 플레이 테스트를 시작해도 Registry가 비어 있으면 초기화하도록 보완했습니다.
- 클라이언트 영역에서 데이터가 없는 경우 `GetMinigameData()`가 초기화를 보장하도록 처리했습니다.

### 5. 싱글 플레이어 전용 Instance Room 분리

- 기존의 단순 맵 텔레포트 방식에서 `RoomService` 기반 이동 방식으로 변경했습니다.
- 사용자 ID와 시간을 조합해 싱글 플레이 전용 Room Key를 생성합니다.
- 선택한 게임의 맵을 포함하는 Instance Room을 생성한 뒤 해당 사용자만 이동시킵니다.
- `SharedMemory`에 `CurrentGameId`를 저장하고, Room 진입 시 다시 읽어 룸 내부 `GameLogic` 상태를 복원합니다.
- 같은 미니게임을 플레이하더라도 싱글 플레이 사용자별 공간이 분리되도록 기반을 마련했습니다.

### 6. 화면 페이드 전환 추가

- 맵 진입 시 `ScreenTransitionService`의 페이드 인/아웃 기능을 활성화했습니다.
- 로비 입장 버튼은 클릭 직후가 아니라 Fade Out 완료 시 관련 UI를 숨기도록 변경했습니다.
- 게임 선택 화면은 Fade Out 시작 시 선택 UI와 확인 팝업을 숨기도록 연결했습니다.
- 중복 Fade Out 처리를 막기 위해 입장 버튼에 선택 여부 상태를 추가했습니다.

> 페이드 전환은 커밋 메시지에 명시된 대로 현재 임시 구현이며, 연출 시간과 세부 동작은 추가 조정이 필요합니다.

## 현재 플레이 흐름

```text
LobbyScene 진입
  -> 로비 UI 활성화 및 페이드 기능 활성화
  -> 싱글 플레이 입장 버튼 선택
  -> Fade Out 완료 후 로비 관련 UI 숨김
  -> SelectScene 진입
  -> Registry 데이터로 게임 슬롯 생성
  -> 좌우 키로 게임 이동 및 설명 갱신
  -> Space 키로 선택 확인 팝업 표시
  -> 확인 버튼 클릭
  -> 서버에서 사용자별 Solo Room Key 생성
  -> 선택 게임 ID를 SharedMemory에 저장
  -> 미니게임 맵을 포함한 Instance Room 생성
  -> 해당 사용자만 Instance Room으로 이동
  -> Room 진입 시 CurrentGameId 복원
```

## 커밋 타임라인

| 시간 | 커밋 | 내용 |
|---:|---|---|
| 13:03 | `c4626c8` | 미니게임 등록 순서를 별도 테이블로 관리해 선택 화면 순서 문제 해결 |
| 13:38 | `e0aa84d` | 게임 설명 데이터와 선택 슬롯 설명 UI 추가 |
| 15:06 | `cf6a2ac` | `SelectScene` 단독 테스트 시 Registry 초기화 타이밍 문제 해결 및 관련 파일 정리 |
| 15:18 | `7a16cb7` | 현재 게임명을 보여주는 선택 확인 팝업 추가 |
| 16:58 | `1bbd42b` | 팝업 표시 중 좌우 키 입력 차단 |
| 18:54 | `b53ad68` | 싱글 플레이 사용자별 Instance Room 생성·이동 구현 |
| 19:18 | `8042a10` | 맵/UI 전환에 임시 Fade In/Out 적용 |

## 주요 변경 파일

| 파일 | 역할 및 변경 내용 |
|---|---|
| `RootDesk/MyDesk/Logics/MinigameLogic/MinigameRegistry.mlua` | 게임 순서 관리, 설명 등록, 중복 방지 및 지연 초기화 |
| `RootDesk/MyDesk/Minigame/MinigameData.mlua` | 미니게임 설명 데이터 추가 |
| `RootDesk/MyDesk/SelectScene/SelectSceneControllerComponent.mlua` | 슬롯 생성·이동, 설명 갱신, 팝업, 입력 잠금, 싱글 룸 진입 요청, Fade Out 처리 |
| `RootDesk/MyDesk/Logics/DefaultLogic/GameLogic.mlua` | Instance Room 생성·이동, Solo Room Key, SharedMemory 상태 전달 |
| `RootDesk/MyDesk/MapEnterController.mlua` | 맵별 UI 활성화, Registry 초기화 보완, 페이드 활성화 |
| `RootDesk/MyDesk/UI/EnterBtnComponent.mlua` | 로비 입장 처리와 Fade Out 완료 시 UI 정리 |
| `map/ButtonGameScene.map` | Instance Room 기반 실행을 위한 버튼 게임 씬 구성 변경 |
| `map/LobbyScene.map`, `map/SelectScene.map` | 컨트롤러·UI·전환 이벤트 연결 변경 |
| `ui/SelectGroup.ui`, `ui/PopupGroup.ui` | 게임 설명 영역과 선택 확인 UI 구성 변경 |

## 후속 작업 메모

- 임시 Fade In/Out의 시간, UI 숨김 타이밍, 연출 디테일을 정리해야 합니다.
- 현재 룸 분리는 싱글 플레이 기준으로 완료된 상태이며, 멀티플레이 매칭/룸 진입 흐름은 별도 확장이 필요합니다.
- `StartMinigame()`의 기존 선택 호출부가 주석 처리되어 있으므로, Instance Room 진입 이후 실제 게임 시작 시점을 최종 흐름에 맞게 점검할 필요가 있습니다.
- 에디터에서 로비 → 선택 화면 → 확인/취소 → 게임 룸 진입까지 전체 흐름의 런타임 회귀 테스트가 필요합니다.

## 검증 범위

- Git 커밋 7개의 메시지, 변경 파일, 실제 `.mlua` diff를 기준으로 정리했습니다.
- 문서 작성 과정에서 게임 에디터 런타임 테스트는 실행하지 않았습니다.
