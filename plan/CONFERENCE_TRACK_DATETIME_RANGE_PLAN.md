# Conference & Track 일시 timestamptz 및 기간(Range) 스키마 개선 계획

## Context
컨퍼런스와 트랙의 일시 필드를 단일 시점뿐만 아니라 기간(시작~종료 범위)을 저장할 수 있도록 확장하고, 컨퍼런스의 `heldAt`을 `text`에서 타임존 적용된 `timestamptz`로 정규화한다. 컨퍼런스 입력은 기존과 동일하게 연월(YYYY-MM) 형식을 유지하되 내부적으로 타임스탬프로 변환 저장하며, 트랙의 `order` 인덱스를 컨퍼런스별 정렬에 최적화된 복합 인덱스 `(conference_id, order)`로 교체한다.

## Approach

### 1. DB 스키마 및 인덱스 정의 변경
- **`src/db/schema/conference.schema.ts`**:
  - `heldAt`: `text("held_at")` → `timestamp("held_at", { withTimezone: true })` (시작 일시)
  - `heldEndAt`: `timestamp("held_end_at", { withTimezone: true })` 컬럼 추가 (종료 일시, nullable)
  - 인덱스: `index("conference_heldAt_idx").on(t.heldAt)` 유지 (타입만 timestamptz로 전환)
- **`src/db/schema/conference-track.schema.ts`**:
  - `heldEndAt`: `timestamp("held_end_at", { withTimezone: true })` 컬럼 추가 (종료 일시, nullable)
  - 기존 단독 인덱스 `index("conference_track_order_idx").on(t.order)` 삭제
  - 복합 인덱스 `index("conference_track_conferenceId_order_idx").on(t.conferenceId, t.order)` 추가

### 2. Drizzle 스키마 코드 변경 (마이그레이션 파일 미생성)
- 사용자 지침 및 컨벤션에 따라 `drizzle/` 디렉토리 내 마이그레이션 파일(`bun db:gen`, DDL 파일 등)은 일절 생성하거나 수정하지 않는다.
- `src/db/schema/` 내 Drizzle ORM 테이블 정의만 변경하여 애플리케이션 레벨 모델을 일치시킨다.
### 3. 타입 정의 및 날짜 변환/포맷 유틸리티 구현
- **`src/modules/conference/conference.type.ts`**:
  - `EditTrack`: `heldEndAt: Date | null` 필드 추가
  - `$conferenceTrackLangContent` 등 기존 타입 유지
- **`src/modules/conference/conference.util.ts`**:
  - `monthStringToStartTimestamp(monthStr: string): Date`: "YYYY-MM" 문자열을 해당 월 1일 00:00:00.000 UTC 시점 `Date`로 변환.
  - `monthStringToEndTimestamp(monthStr: string): Date`: "YYYY-MM" 문자열을 해당 월 말일 23:59:59.999 UTC 시점 `Date`로 변환.
  - `timestampToMonthString(date: Date | string | null): string`: `Date`를 "YYYY-MM" 문자열로 역변환.
  - `formatConferenceHeldAt(heldAt: Date | string | null, heldEndAt: Date | string | null, language: "ko" | "en"): string`:
    - `heldAt`만 존재: `YYYY년 M월` (ko) / `MMM YYYY` (en)
    - `heldEndAt`도 존재: 같은 연도면 `YYYY년 M월 ~ M월`, 다른 연도면 `YYYY년 M월 ~ YYYY년 M월`
  - `formatTrackHeldAt(heldAt: Date | string | null, heldEndAt: Date | string | null, language: "ko" | "en"): string`:
    - `heldAt`만 존재: 기존과 동일하게 `YYYY년 M월 D일 (ddd) HH:mm`
    - `heldEndAt`도 존재: 같은 날이면 `... HH:mm ~ HH:mm`, 다른 날이면 `... ~ YYYY년 M월 D일 (ddd) HH:mm`
  - `validateConferenceInfo`: `heldAt: string` 및 `heldEndAt?: string` 입력 검증 (시작월이 종료월보다 늦으면 에러 반환).
- **`src/modules/conference/conference.util.test.ts`**:
  - 날짜 변환 함수 및 포맷팅 함수의 단위 테스트 추가/갱신.

### 4. 서비스 및 RPC 계층 변경
- **`src/modules/conference/conference.service.ts`**:
  - `createConference`: 파라미터 `heldAt?: Date | null; heldEndAt?: Date | null;` 지원.
  - `updateConference`: 파라미터 `heldAt?: Date | null; heldEndAt?: Date | null;` 지원.
  - `getConferences`: 정렬 기준 `orderBy: { heldAt: "desc" }` 유지.
- **`src/modules/conference/conference.rpc.ts`**:
  - `createConferenceRpc` / `updateConferenceRpc`:
    - validator에서 `heldAt: z.string().optional()` ("YYYY-MM" 문자열) 및 `heldEndAt: z.string().nullable().optional()` ("YYYY-MM" 문자열) 수신.
    - handler에서 `monthStringToStartTimestamp(heldAt)` 및 `monthStringToEndTimestamp(heldEndAt)`로 `Date` 변환하여 서비스 함수로 전달.
- **`src/modules/conference/conferenceTrack.service.ts`**:
  - `createConferenceTrack`: 파라미터 `heldEndAt?: Date | null;` 추가 및 insert 값에 포함.
  - `updateConferenceTrack`: 파라미터 `heldEndAt?: Date | null;` 추가 및 update 값에 포함.
- **`src/modules/conference/conferenceTrack.rpc.ts`**:
  - `createConferenceTrackRpc`: validator에 `heldEndAt: z.string().datetime().optional()` 추가.
  - `updateConferenceTrackRpc`: validator에 `heldEndAt: z.string().datetime().nullable().optional()` 추가.
- **`src/modules/conference/hooks/useConferenceTrackSave.ts`**:
  - 저장 페이로드에 `heldEndAt: track.heldEndAt?.toISOString() ?? null` 전달.

### 5. 컨퍼런스 UI 컴포넌트 갱신
- **`src/modules/conference/components/edit/conferenceMonthPicker.tsx`**:
  - 기존 단일 월 선택 기능 유지하되, 필요 시 label/trigger 스타일을 공용화.
- **`src/modules/conference/components/edit/conferenceEditForm.tsx`**:
  - `heldAt`, `heldEndAt` 로컬 상태 관리 (`useState`).
  - "종료 연월 지정" 체크박스(토글) 추가:
    - 체크 시 두 번째 `ConferenceMonthPicker` 렌더링.
    - 체크 해제 시 `heldEndAt = ""`로 초기화.
  - 저장 시 `heldAt`, `heldEndAt: heldEndAt || null` 전송.
- **표시 컴포넌트들 (`conferenceCard.tsx`, `conferenceDetailHeader.tsx`, `conferenceFeedCard.tsx`, `conferenceDraftsDialog.tsx`)**:
  - 기존 `conference.heldAt` 직접 문자열 출력 부분을 `formatConferenceHeldAt(conference.heldAt, conference.heldEndAt, language)` 호출로 대체.

### 6. 트랙 UI 컴포넌트 갱신
- **`src/modules/conference/components/edit/conferenceTrackEditor.tsx`**:
  - `heldEndAt` 입력 필드 지원:
    - "종료 일시 지정" 체크박스/토글 제공.
    - 활성화 시 두 번째 `datetime-local` input 렌더링.
    - `onUpdate` 시 `heldAt`, `heldEndAt` 반영.
- **`src/modules/conference/components/conferenceTrackAddDialog.tsx`**:
  - 트랙 신규 생성 모달에 종료 일시(`newTrackHeldEndAt`) 입력 옵션 추가 (선택사항).
- **`src/modules/conference/components/conferenceTrackPanel.tsx`**:
  - 트랙 메타 바의 일시 출력을 `formatTrackHeldAt(track.heldAt, track.heldEndAt, language)`로 변경.

## Critical files & anchors
1. `src/db/schema/conference.schema.ts`: `conferenceTable` 정의 - `heldAt` timestamptz 변경 및 `heldEndAt` 추가.
2. `src/db/schema/conference-track.schema.ts`: `conferenceTrackTable` 정의 - `heldEndAt` 추가 및 복합 인덱스 변경.
3. `src/modules/conference/conference.util.ts`: 날짜 변환(`monthStringTo...`) 및 포맷(`formatConferenceHeldAt`, `formatTrackHeldAt`) 핵심 로직.
4. `src/modules/conference/components/edit/conferenceEditForm.tsx`: 컨퍼런스 기간(시작~종료월) 입력 UI 및 저장 핸들러.
5. `src/modules/conference/components/edit/conferenceTrackEditor.tsx`: 트랙 시작~종료 일시 입력 UI.

## Verification
- **단위 테스트**:
  - `bun test src/modules/conference/conference.util.test.ts` 실행:
    - 단일 월("2026-09") → 시작 타임스탬프 변환 검증
    - 기간("2026-09" ~ "2026-10") → 시작/종료 타임스탬프 및 포맷 출력 검증
    - 트랙 시작/종료 포맷팅 출력 검증
- **타입 및 린트 검증**:
  - `bun run typecheck`: 스키마 및 RPC/컴포넌트 변경 후 전체 타입 오류 0건 검증.
  - `bun run lint`: 코드 스타일 및 컨벤션 검증.
- **E2E / 런타임 수동 검증**:
  - 컨퍼런스 생성/수정 화면에서 기간(예: 2026-09 ~ 2026-11) 입력 후 저장 → 상세 화면 헤더 및 카드에 "2026년 9월 ~ 11월" 정상 렌더링 확인.
  - 트랙 편집 화면에서 시작/종료 일시 지정 후 저장 → 트랙 패널에 "YYYY년 M월 D일 HH:mm ~ HH:mm" 정상 표시 확인.

## Assumptions & contingencies
- **컨퍼런스 년월 타임스탬프 기준점**:
  - 시작월("2026-09")은 해당 월 1일 00:00:00 UTC로 저장, 종료월("2026-10")은 10월 31일 23:59:59.999 UTC로 저장. (서버 타임존이 필요할 경우 UTC 기준 일관 저장).
- **마이그레이션 파일**:
  - `drizzle/` 마이그레이션 파일 생성은 제외하며, 스키마 코드와 서비스/UI 코드 변경에 집중한다.
