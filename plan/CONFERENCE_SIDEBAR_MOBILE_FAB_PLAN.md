# 컨퍼런스 상세 모바일 목차(FAB + Dialog) 구현 계획

## Context
컨퍼런스 상세 페이지(`src/routes/_authed/conference/$id.index.tsx`)의 사이드바(`ConferenceDetailSidebar`)는 데스크톱(`lg` 이상)에서만 표시되고 모바일/태블릿(`< lg`) 화면에서는 `hidden` 처리되어 세션/트랙 목차를 탐색할 수 없는 문제가 있습니다.
모바일 사용자가 본문 스크롤 위치와 상관없이 언제든 목차를 열어 원하는 트랙이나 공통 세션으로 이동할 수 있도록, 모바일 전용 플로팅 버튼(FAB)과 목차 모달(Dialog)을 구현합니다.

## Approach

### 1. 사이드바 목록 렌더링 로직 재사용화 (`src/modules/conference/components/conferenceDetailSidebar.tsx`)
- `ConferenceDetailSidebar` 내부의 트랙 및 세션 목록 렌더링 JSX(L140-L391)를 인라인 렌더 함수 `renderSidebarContent(onSelect?: () => void)`로 분리합니다.
- 트랙 타이틀 클릭(`handleTrackClick(track.id)`) 시 `onSelect?.()`를 호출하여 모바일 모달이 자동으로 닫히고 선택된 트랙 본문으로 시선이 전환되도록 합니다 (아코디언 토글 화살표 버튼 클릭 시에는 `onSelect`를 호출하지 않고 아코디언만 토글).
- 세션 클릭(`onSessionOpen(session.id)`) 시 `onSelect?.()`를 호출하여 목차 모달을 닫고 `ConferenceSessionDialog`가 단독으로 노출되도록 합니다.
- 데스크톱 사이드바에서는 `renderSidebarContent()` 형태로 인자 없이 호출하여 기존 동작을 100% 유지합니다.

### 2. 모바일 FAB 및 목차 Dialog 구현 (`src/modules/conference/components/conferenceDetailSidebar.tsx`)
- `useState(false)`로 `mobileOpen` 상태를 선언합니다.
- `lucide-react`에서 `List` 아이콘을 추가 import합니다.
- `@/components/base/dialog`에서 `Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`을 import합니다.
- **모바일 플로팅 액션 버튼 (FAB)**:
  - 위치: `fixed bottom-20 right-6 z-40 lg:hidden` (기존 `ScrollToTopBtn` `bottom-6 right-6 z-50`의 바로 위 16px 간격으로 안정적 스택 배치, 겹침 완전 방지).
  - 스타일: `<Button variant="default" size="icon" className="size-11 rounded-full shadow-lg" onClick={() => setMobileOpen(true)} aria-label={isKo ? "목록 열기" : "Open index"}><List className="size-5" /></Button>`.
- **모바일 목차 Dialog**:
  - `<Dialog open={mobileOpen} onOpenChange={setMobileOpen}>` 구성.
  - `<DialogContent className="max-h-[80vh] overflow-y-auto sm:max-w-md p-5">` 내부에:
    - `<DialogHeader className="pb-3 border-b border-border"><DialogTitle>{isKo ? "목록" : "Index"}</DialogTitle></DialogHeader>`
    - 본문 컨테이너에 `{renderSidebarContent(() => setMobileOpen(false))}` 삽입.
  - 다이얼로그의 닫기(`XIcon`) 버튼은 `DialogContent` 내장 기능을 사용합니다.

## Critical files & anchors
1. `src/modules/conference/components/conferenceDetailSidebar.tsx`
   - L1-L24: `List` 아이콘 및 `@/components/base/dialog` 컴포넌트 import.
   - L64-L92: `mobileOpen` 상태 선언 및 `renderSidebarContent` 헬퍼 구성.
   - L94-L425: 데스크톱 `<aside>` 렌더링 유지 + 루트 fragment에 모바일 FAB 및 모바일 `Dialog` 배치.

## Verification
1. **정적 검증**:
   - `bun run typecheck` 실행하여 타입 에러 0건 확인.
   - `bun run lint:write` 실행하여 린트 및 포맷팅 검증.
2. **동작 및 시각적 검증**:
   - 데스크톱 화면(`lg` 이상, >=1024px):
     - 플로팅 버튼(FAB)이 화면에 전혀 나타나지 않는지 확인 (`lg:hidden`).
     - 기존 우측 사이드바(펼침/접힘 기능)가 정상 동작하는지 확인.
   - 모바일/태블릿 화면(`< lg`, <1024px):
     - 우측 하단(`bottom-20 right-6`)에 원형 플로팅 목록 버튼이 표시되는지 확인.
     - 스크롤을 내려도 `ScrollToTopBtn`(`bottom-6`)과 겹치지 않고 그 위에 깔끔하게 위치하는지 확인.
     - FAB 클릭 시 모바일 목차 모달(Dialog)이 열리고 공통 세션 및 트랙 목록이 정상 렌더링되는지 확인.
     - 트랙 선택 시 모달이 닫히고 해당 트랙이 활성화되는지 확인.
     - 세션 선택 시 모달이 닫히고 해당 세션 상세 다이얼로그가 열리는지 확인.
     - 트랙 아코디언 토글 클릭 시 모달이 닫히지 않고 내부에서 정상적으로 열리고 닫히는지 확인.

## Assumptions & contingencies
- 모바일 FAB의 z-index는 `z-40`으로 설정하여 `ScrollToTopBtn`(`z-50`) 및 모달 오버레이(`z-50`) 아래에 정상 배치되도록 합니다.
- 모바일 환경에서도 컨퍼런스 관리자(`canManage === true`)의 경우 세션 추가/관리, 트랙 추가/삭제 등의 액션 버튼이 모바일 모달 내에 동일하게 제공됩니다.
