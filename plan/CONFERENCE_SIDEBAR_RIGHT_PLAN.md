# 컨퍼런스 상세 사이드바 우측 이동 및 접기 기능 구현 계획

## Context
컨퍼런스 상세 페이지(`src/routes/_authed/conference/$id.index.tsx`)에서 좌측에 위치하던 `ConferenceDetailSidebar`를 본문 우측으로 이동하고, 사용자가 사이드바를 접고 펼 수 있도록 개선합니다.
접힘/펼침 상태는 새로고침 시 초기화(기본 펼침 유지)되어야 하며, 토글 버튼이 본문 영역(`main`) 내부에 위치하여 본문의 세션 카드 그리드나 콘텐츠 뷰어의 수직 위치/레이아웃을 변형시키지 않도록 사이드바 전용 영역에 독립 배치합니다. 사이드바가 접히면 본문 영역(`main`)은 가용 우측 공간으로 자연스럽게 확장됩니다.

## Approach

### 1. 라우트 컴포넌트(`src/routes/_authed/conference/$id.index.tsx`) 레이아웃 재배치 및 상태 제어
- `useState(true)`를 사용하여 새로고침 시 항상 펼쳐져 있는 `sidebarOpen` 로컬 상태를 선언합니다 (스토리지/쿼리 파라미터 영속화 배제).
- `<div className="flex gap-8">` 컨테이너 내부의 자식 순서를 변경합니다:
  - `<main className="min-w-0 flex-1 pt-2">`를 첫 번째 자식으로 배치 (좌측 본문 영역).
  - `<ConferenceDetailSidebar>`를 두 번째 자식으로 배치 (우측 사이드바 영역).
- `ConferenceDetailSidebar`에 `isOpen={sidebarOpen}`, `onToggleOpen={() => setSidebarOpen((prev) => !prev)}` props를 전달합니다.
- 본문 영역(`main`)은 `flex-1`을 유지하므로, 사이드바가 접혔을 때 가용 공간만큼 자연스럽게 넓어집니다.

### 2. 사이드바 컴포넌트(`src/modules/conference/components/conferenceDetailSidebar.tsx`) 접기/펼치기 UI 구현
- `ConferenceDetailSidebarProps` 인터페이스에 `isOpen?: boolean`, `onToggleOpen?: () => void`를 추가합니다 (기본값 `isOpen = true`).
- `lucide-react`에서 `PanelRightClose`, `PanelRightOpen` 아이콘을 import합니다.
- **펼침 상태 (`isOpen === true`)**:
  - `<aside className="hidden w-64 shrink-0 lg:block pt-6">` 구조 유지.
  - sticky 컨테이너(`sticky top-27.25 max-h-[calc(100vh-4rem-45px)] overflow-y-auto pb-8 pl-2 pr-1`) 최상단에 헤더 영역 추가:
    - 타이틀: 한국어 `"목록"`, 영어 `"Index"`.
    - 접기 버튼: `Tooltip` 내부에 `<Button variant="ghost" size="icon-xs" onClick={onToggleOpen}><PanelRightClose className="size-4" /></Button>`, 툴팁 텍스트는 `"사이드바 접기"` / `"Collapse sidebar"`.
  - 기존의 공통 세션 및 트랙 목록 렌더링 유지.
- **접힘 상태 (`isOpen === false`)**:
  - `<aside className="hidden shrink-0 lg:block pt-6">` 렌더링.
  - `sticky top-27.25` 컨테이너에 펼치기 버튼 배치:
    - `Tooltip` 내부에 `<Button variant="outline" size="icon-sm" onClick={onToggleOpen}><PanelRightOpen className="size-4" /></Button>`, 툴팁 텍스트는 `"사이드바 펼치기"` / `"Expand sidebar"`.
- **다이얼로그 유지**:
  - `AlertDialog`(트랙 삭제 확인)와 `ConferenceTrackAddDialog`(트랙 추가)는 `isOpen` 상태와 무관하게 루트 fragment 레벨에서 항상 렌더링되어 다이얼로그 상태 및 애니메이션이 유실되지 않도록 보장합니다.

## Critical files & anchors
1. `src/routes/_authed/conference/$id.index.tsx`
   - L114-L126: `<div className="flex gap-8">` 내의 `main`과 `ConferenceDetailSidebar` 배치 순서 교체 및 `sidebarOpen` 상태 연동.
2. `src/modules/conference/components/conferenceDetailSidebar.tsx`
   - L33-L41: `ConferenceDetailSidebarProps` 인터페이스 확장 (`isOpen`, `onToggleOpen`).
   - L83-L336: `<aside>` 렌더링을 펼침(`w-64` + 상단 접기 헤더)과 접힘(`shrink-0` + sticky 펼치기 버튼)으로 분기.

## Verification
1. **정적 검증**:
   - `bun run typecheck` 실행하여 타입 에러 0건 확인.
   - `bun run lint:write` 실행하여 린트 및 포맷팅 검증.
2. **동작 검증 (브라우저 또는 시나리오 테스트)**:
   - 컨퍼런스 상세 페이지(`/conference/1` 등) 접속 시 사이드바가 본문 우측에 `w-64`로 기본 펼쳐져 표시되는지 확인.
   - 상단 우측 사이드바 접기 버튼(`PanelRightClose`) 클릭 시 사이드바가 닫히고 본문(`main`)이 우측으로 자연스럽게 확장되는지 확인.
   - 사이드바가 닫힌 상태에서 우측 상단 sticky 펼치기 버튼(`PanelRightOpen`)이 본문 콘텐츠를 가리거나 본문 내부로 침범하지 않는지 확인.
   - 펼치기 버튼 클릭 시 사이드바가 다시 우측에 나타나고 본문 너비가 원래 크기로 복원되는지 확인.
   - 페이지 새로고침 시 접힌 상태가 유지되지 않고 항상 펼쳐진 상태로 초기화되는지 확인.
   - 본문 영역(`main`) 내부의 에디터 뷰어, 공통 세션 카드, 트랙 메타바 등에 버튼으로 인한 추가 여백이나 높이 변경이 발생하지 않는지 확인.

## Assumptions & contingencies
- 모바일/태블릿 화면(`< lg`)에서는 기존과 동일하게 사이드바가 `hidden` 처리되며, 데스크톱(`lg` 이상)에서만 우측 사이드바 및 접기/펼치기 토글이 활성화됩니다.
- 본문 영역 레이아웃 변경 방지 원칙에 따라 토글 버튼은 100% `<aside>` 컨테이너 내부에만 위치하며 `<main>` 엘리먼트 내부로는 일체 주입되지 않습니다.
