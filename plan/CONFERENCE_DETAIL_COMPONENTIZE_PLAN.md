# Conference Detail Page Componentization Plan

## Context
`src/routes/_authed/conference/$id.index.tsx` contains 852 lines of code combining routing, page orchestration, header metadata, navigation sidebar, main session grids, and four dialogs with multiple form states. This refactoring extracts the self-contained sections into three modular components under `src/modules/conference/components/` to encapsulate local dialog/form states and simplify the route file to ~140 lines.

## Approach

### 1. Extract Track Add Dialog (`conferenceTrackAddDialog.tsx`)
Create `src/modules/conference/components/conferenceTrackAddDialog.tsx` to encapsulate the track creation modal and its 4 form input states.
- **Props Interface**:
  ```ts
  interface ConferenceTrackAddDialogProps {
    open: boolean;
    onOpenChange: (open: boolean) => void;
    conferenceId: number;
    trackCount: number;
  }
  ```
- **Internal State & Logic**:
  - Manage `newTrackNameKo`, `newTrackNameEn`, `newTrackHeldAt`, `newTrackLocation`.
  - Reset form inputs on open/close and after submission.
  - Call `createConferenceTrackRpc({ data: { conferenceId, langContents, heldAt, location, order: trackCount } })`.
  - On creation success, close dialog, reset state, and navigate to `/conference/$id/track/$trackId/edit`.
- **UI**:
  - Wrap in `Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogFooter`, with Ko/En inputs, datetime-local input, and location input.

### 2. Extract Detail Header & Administrative Actions (`conferenceDetailHeader.tsx`)
Create `src/modules/conference/components/conferenceDetailHeader.tsx` to encapsulate conference metadata, cover image, breadcrumbs, and conference-level mutations (visibility toggling & conference deletion).
- **Props Interface**:
  ```ts
  interface ConferenceDetailHeaderProps {
    conference: ConferenceWithTracks;
    tracks: ConferenceTrackWithSessions[];
    canManage: boolean;
  }
  ```
- **Internal State & Logic**:
  - `visibilityDialogOpen` (boolean) and `isDeleteDialogOpen` (boolean).
  - Use `useLanguage()`, `useNavigate()`, `useRouter()`, and `useConferenceMutations(conference.id)`.
  - `handleToggleVisibility`: call `mutations.updateConference.mutateAsync`, invalidate router, toast success notification, close dialog.
  - `handleDelete`: call `mutations.deleteConference.mutateAsync`, navigate to `/conference`.
- **UI**:
  - Breadcrumb navigation link (`/conference`).
  - Cover image display with `imgSrc(conference.imageUrl, 1440)`.
  - Visibility badge, title (Ko/En fallback), description.
  - Held date, track count, total session count metadata badges.
  - Manager dropdown menu (Edit link, Publish/Private toggle button, Delete button).
  - `AlertDialog` for visibility toggle and `AlertDialog` for conference deletion.

### 3. Extract Navigation Sidebar & Track Management (`conferenceDetailSidebar.tsx`)
Create `src/modules/conference/components/conferenceDetailSidebar.tsx` to encapsulate the sidebar accordion, common sessions navigation list, track lists, track deletion, and track addition trigger.
- **Props Interface**:
  ```ts
  interface ConferenceDetailSidebarProps {
    conference: ConferenceWithTracks;
    tracks: ConferenceTrackWithSessions[];
    activeTrackId: number;
    canManage: boolean;
    onTrackSelect: (trackId: number) => void;
    onSessionOpen: (sessionId: number) => void;
    onSessionPrefetch: (sessionId: number) => void;
  }
  ```
- **Internal State & Logic**:
  - `expandedId` (number | null) for accordion state, initialized to `activeTrackId`.
  - `deleteTrackId` (number | null) for track deletion dialog.
  - `addTrackOpen` (boolean) for triggering `ConferenceTrackAddDialog`.
  - Use `useLanguage()`, `useNavigate()`, `useRouter()`, and `useConferenceMutations(conference.id)`.
  - `handleDeleteTrack`: call `mutations.deleteTrack.mutateAsync(deleteTrackId)`, invalidate router, reset `deleteTrackId`.
- **UI**:
  - Sticky aside navigation wrapper (`<aside className="hidden w-64 shrink-0 lg:block pt-6">`).
  - If `conference.useTracks`:
    - Common sessions section (if `directSessions.length > 0`) with session list and edit button.
    - Tracks header with "Add Common Session" button if no direct sessions exist.
    - Tracks accordion list with expand toggle, track selection, session count badge, and manager dropdown (Edit track / Delete track).
    - Sub-list of sorted sessions under expanded track.
    - "Add Track" button for managers.
  - If not `conference.useTracks`:
    - Sessions header and sorted session links with manager edit button.
  - Track delete `AlertDialog`.
  - `ConferenceTrackAddDialog` instance.

### 4. Refactor Route Page (`src/routes/_authed/conference/$id.index.tsx`)
Update `ConferenceDetailPage` to compose the newly extracted components:
- Remove obsolete states: `visibilityDialogOpen`, `addTrackOpen`, `newTrackNameKo`, `newTrackNameEn`, `newTrackHeldAt`, `newTrackLocation`, `deleteTrackId`, `isDeleteDialogOpen`, `expandedId`.
- Remove obsolete mutation calls and handlers: `handleToggleVisibility`, `handleConfirmAddTrack`, `handleDeleteTrack`, `handleDelete`, `toggleExpand`.
- Retain router-driven logic:
  - `prefetchSession` (via `queryClient`).
  - `handleTrackActivate`: navigates to `track: id`, clears `session`.
  - `handleSessionOpen`: navigates to `session: sessionId`.
  - `handleSessionClose`: navigates to `session: undefined`.
- Render hierarchy:
  ```tsx
  <ContentLayout>
    <div className="mx-auto w-full max-w-screen-2xl px-6 pb-16">
      <ConferenceDetailHeader conference={conference} tracks={tracks} canManage={canManage} />
      <div className="flex gap-8">
        <ConferenceDetailSidebar
          conference={conference}
          tracks={tracks}
          activeTrackId={activeTrackId}
          canManage={canManage}
          onTrackSelect={handleTrackActivate}
          onSessionOpen={handleSessionOpen}
          onSessionPrefetch={prefetchSession}
        />
        <main className="min-w-0 flex-1 pt-2">
          {/* Viewer & Main session grids */}
        </main>
      </div>
    </div>
    <ConferenceSessionDialog sessionId={session} tracks={tracks} onClose={handleSessionClose} />
  </ContentLayout>
  ```
- Remove unused imports (alert dialogs, dialog, dropdown, input, unused icons).

## Critical Files & Anchors
1. `src/routes/_authed/conference/$id.index.tsx`: Target route to simplify.
2. `src/modules/conference/components/conferenceDetailHeader.tsx`: New component for header, cover, and conference status/delete modals.
3. `src/modules/conference/components/conferenceDetailSidebar.tsx`: New component for sidebar navigation and track delete modal.
4. `src/modules/conference/components/conferenceTrackAddDialog.tsx`: New component for track creation form dialog.
5. `src/modules/conference/components/conferenceTrackPanel.tsx`: Existing component rendered in the main section (reference for props/patterns).

## Verification
1. **Typecheck & Linter**:
   - Run `bun run lint:write && bun run typecheck` to ensure no broken imports, unused variables, or type mismatches.
2. **Functional Smoke Check**:
   - Verify header renders cover, title, description, and metadata badges.
   - Verify manager dropdown opens visibility dialog and deletion dialog.
   - Verify sidebar track accordion toggles sessions, track selection navigates with `track` search param.
   - Verify session link hover prefetches session query, click opens `ConferenceSessionDialog` with `session` search param.
   - Verify track addition dialog opens, accepts form inputs, and redirects to track edit route.
   - Verify track deletion and conference deletion alerts function as before.
