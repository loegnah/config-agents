# Quick-Win 로드 성능 최적화: 홈 & 피쳐노트

홈 대시보드의 중복 쿼리 제거 및 마운트 후 즉시 재요청(refetch) 방지, 그리고 피쳐노트 메인 페이지(`/feature`)의 클라이언트 3중 분산 RPC 호출을 SSR 로더 및 단일 BFF RPC로 통합하여 초기 로딩 성능과 불필요한 DB 조인 부하를 해소한다.

---

## 1. 홈 `latestRn` dead-fetch 제거 및 단건 통일

### 변경 내용
1. `src/modules/home/home.rpc.ts`:
   - `fetchHomeDashboardData`의 `Promise.all` 내 1번 쿼리(`db.query.rnTable.findFirst({ with: { target: true } })`)를 완전 제거.
   - 2번 쿼리(`db.query.rnTable.findMany({ limit: 1, with: { target: { with: { fns: ... } } } })`)를 `findFirst` 단건 쿼리로 변경하고 변수명을 `latestRn`으로 지정.
   - `Promise.all` 반환 튜플 및 최종 반환 객체에서 `latestRns` 필드를 삭제하고 단건 `latestRn`만 반환.
2. `src/modules/home/components/homeDashboard.tsx`:
   - 라인 61의 위젯 노드 등록부를 `{ id: "release", node: <ReleaseHomeWidget rns={data.latestRns} /> }`에서 `{ id: "release", node: <ReleaseHomeWidget rn={data.latestRn} /> }`로 변경.
3. `src/modules/home/components/widgets/releaseHomeWidget.tsx`:
   - `ReleaseHomeWidget`의 props 타입을 `{ rns }: { rns: HomeDashboardData["latestRns"] }`에서 `{ rn }: { rn: HomeDashboardData["latestRn"] }`로 변경.
   - 내부 렌더링 조건을 `rns && rns.length > 0 ? <ReleaseContent rn={rns[0]} />`에서 `rn ? <ReleaseContent rn={rn} />`로 변경.
   - `WidgetHeader`의 `to` 링크도 `rn ? "/release" : undefined`로 단순화.
   - `ReleaseContent`의 props 타입도 `rn: NonNullable<HomeDashboardData["latestRn"]>`으로 정리.

---

## 2. 홈 마운트 즉시 7개 쿼리 재호출 방지 (`staleTime`)

### 변경 내용
1. `src/modules/home/hooks/useMyActivity.ts`:
   - `useMyActivity` 훅의 `useQuery` 옵션에 `staleTime: 60 * 1000` (1분) 추가.
   - SSR 로더에서 전달된 `initialData`를 즉시 stale로 간주하여 마운트 직후 6개 DB 집계 쿼리가 재실행되는 현상 차단.
2. `src/modules/inbox/hook/useInbox.ts`:
   - `useInboxCounts` 훅의 `useQuery` 옵션에 `staleTime: 60 * 1000` (1분) 추가.
   - SSR 로더에서 전달된 `initialData`로 인해 마운트 직후 인박스 카운트 쿼리가 재실행되는 현상 차단 (인박스 메시지 변동 시 `qc.invalidateQueries({ queryKey: ["inbox", "counts"] })`가 이미 존재하므로 캐시 갱신 안전).

---

## 3. 피쳐노트 메인(`/feature`) 단일 BFF RPC & SSR 로더 통합 및 뱃지 경량화

### 서비스 레이어 분리 (`*.service.ts`)
1. `src/core/target/target.service.ts` 신규 생성:
   - `db.query.targetTable.findMany({ orderBy: { platform: "asc", majorVersion: "asc" } })`를 실행하는 `getTargetsService = createServerOnlyFn(...)` 작성.
2. `src/core/target/target.rpc.ts`:
   - 기존 `getTargets` RPC 핸들러가 `getTargetsService()`를 호출하도록 위임.
3. `src/modules/fn/fn.service.ts`:
   - `getAllPublishedFns = createServerOnlyFn(...)`: 발행된 릴리즈노트의 targetId에 속한 FN 목록(`fnTable.findMany({ where: { targetId: { in: targetIds } }, with: { category: true, assignee: true } })`)을 조회하는 서비스 함수 추가.
   - `getFnConfirmSummary = createServerOnlyFn(...)`: 파라미터 `{ assigneeId: number }`를 받아 `fnTable.findMany({ where: { assigneeId }, columns: { id: true, isConfirmed: true } })`로 조회 후 `{ hasUnconfirmed: boolean, totalAssignedCount: number }`만 반환하는 경량 요약 함수 추가 (3-way 조인 및 contents jsonb 로딩 배제).

### RPC 레이어 통합 (`*.rpc.ts`)
1. `src/modules/fn/fn.rpc.ts`:
   - `fetchFeatureIndexData = createServerFn().middleware([hasLoginMiddle]).handler(...)` 신규 정의:
     - `Promise.all`로 `getAllPublishedFns()`, `getTargetsService()`, `getFnConfirmSummary({ assigneeId: user.accountId })`를 병렬 호출.
     - 반환 객체: `{ allFns, targets, confirmSummary }`.
     - `export type FeatureIndexData = Awaited<ReturnType<typeof fetchFeatureIndexData>>;` 타입 내보내기.

### 훅 및 라우트 레이어 전환
1. `src/modules/fn/hooks/useFn.ts`:
   - `useFeatureIndexData(initialData?: FeatureIndexData)` 훅 추가 (`queryKey: ["feature", "indexData"]`, `queryFn: () => fetchFeatureIndexData()`, `initialData`, `staleTime: 60 * 1000`).
2. `src/routes/_authed/feature/index.tsx`:
   - Route 정의에 `loader: async () => fetchFeatureIndexData()` 추가.
   - `FnNotePage` 컴포넌트에서:
     - `const loaderData = Route.useLoaderData();`
     - `const { data } = useFeatureIndexData(loaderData);`
     - `const allFns = data?.allFns ?? loaderData.allFns;`
     - `const targets = data?.targets ?? loaderData.targets;`
     - `const confirmSummary = data?.confirmSummary ?? loaderData.confirmSummary;`
     - 기존 개별 훅 호출(`useAllFns({})`, `useConfirmFns(false, true)`, `useTargets()`) 삭제.
     - 뱃지 및 컨펌 버튼 판별식을 `confirmSummary` 기반으로 전환:
       - `const hasUnconfirmed = confirmSummary.hasUnconfirmed;`
       - `const showConfirmButton = isManager || confirmSummary.totalAssignedCount > 0;`
     - `isLoading`, `isConfirmLoading` 플래그 제거 (SSR 로더 데이터로 첫 렌더 즉시 완료, 스피너 대기 제거).
3. `src/modules/fn/components/fnConfirmBtn.tsx` & `src/modules/fn/components/fnRejectBtn.tsx`:
   - 컨펌 및 반려 성공 시 `queryClient.invalidateQueries({ queryKey: ["feature"] })` 무효화 호출 추가.

---

## 4. Critical Files & Anchors

- `src/modules/home/home.rpc.ts:114-135,173`: `latestRn`/`latestRns` 쿼리 단건화 및 반환 필드 정리
- `src/modules/home/components/widgets/releaseHomeWidget.tsx:34-60`: `rns` 배열 prop을 단건 `rn` 객체 prop으로 변환
- `src/modules/home/hooks/useMyActivity.ts:5-10`: `staleTime: 60 * 1000` 추가
- `src/modules/inbox/hook/useInbox.ts:35-41`: `staleTime: 60 * 1000` 추가
- `src/modules/fn/fn.service.ts:300-320`: `getAllPublishedFns`, `getFnConfirmSummary` 신규 서비스 작성
- `src/modules/fn/fn.rpc.ts:585-618`: `fetchFeatureIndexData` 복합 RPC 작성
- `src/routes/_authed/feature/index.tsx:54-75`: `loader` 등록 및 분산 훅 3개를 단일 `useFeatureIndexData`로 교체

---

## 5. Verification

1. **정적 검증**:
   - `bun run lint:write && bun run typecheck` 실행하여 타입 에러 0건 및 린트 통과 확인.
2. **동작 검증 (홈 `/`)**:
   - 브라우저로 `http://localhost:14005/` 접속 시 콘솔 에러 없음 확인.
   - 릴리즈노트 위젯이 최신 1건 및 롤링 피쳐노트 3건을 정상 렌더링하는지 확인.
   - 페이지 마운트 직후 네트워크 탭에서 `fetchMyActivity`, `fetchInboxCounts`가 중복 호출되지 않는지 확인.
3. **동작 검증 (피쳐노트 `/feature`)**:
   - `http://localhost:14005/feature` 접속 시 로딩 스피너 깜빡임 없이 즉시 목록과 필터가 렌더링되는지 확인.
   - 우측 상단 "피쳐노트 컨펌내역" 또는 "컨펌 요청이 있어요" 버튼이 사용자 권한/할당 상태에 맞게 정상 표시되는지 확인.
   - 네트워크 탭에서 `fetchConfirmFns(showConfirmed: true)` 풀조인 호출이 사라지고 `fetchFeatureIndexData` 1개로 통합되었는지 확인.
