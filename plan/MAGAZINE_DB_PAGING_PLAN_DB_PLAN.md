# 매거진 목록 DB 페이징 및 TanStack Query 캐싱 최적화 계획

## Context
현재 매거진 목록(`/magazine`)은 DB 전체 행(`magazineTable.findMany`)을 조건 없이 가져와 JS 메모리에서 필터/정렬/슬라이스(`slice(offset, offset + limit)`)하며, loader가 파라미터 없이 호출하여 최대 50건만 조회되는 버그가 있다. 또한 본문 jsonb(`contents`) 및 미사용 작성자(`author`) 전체를 풀조인하여 오버페치를 유발하고, TanStack Query 캐시(`ensureQueryData`)를 사용하지 않는다.
블로그 목록(`getBlogPubList`, `blogListQueryOptions`)의 검증된 패턴을 수평 전개하여 DB 레벨 `LIMIT`/`OFFSET` 페이징, narrow projection, TanStack Router + Query 캐싱 구조로 전환한다.

## Approach

### 1. 매거진 목록 아이템 타입 정의 (`src/modules/magazine/magazine.type.ts`)
- `MagazinePubListItem` 타입 추가:
  ```ts
  export type MagazinePubListItem = Pick<
    Magazine,
    "id" | "title" | "description" | "coverImageUrl" | "issueDate" | "postNumber" | "publishedAt"
  >;
  ```
- 기존 `MagazinePub`은 타입 정의 유지하되, 목록 전용 narrow 타입으로 `MagazinePubListItem`을 명시.

### 2. DB 페이징 & Narrow Select 서비스 구현 (`src/modules/magazine/magazine.service.ts`)
- 기존 `getMagazinePubs`를 대체하는 `getMagazinePubList` 작성:
  ```ts
  export const getMagazinePubList = createServerOnlyFn(
    async (
      params: { limit?: number; offset?: number } = {},
    ): Promise<{ posts: MagazinePubListItem[]; total: number }> => {
      const { limit = 12, offset = 0 } = params;
      const where = isNotNull(magazineTable.postNumber);

      const [rows, total] = await Promise.all([
        db
          .select({
            id: magazineTable.id,
            title: magazineTable.title,
            description: magazineTable.description,
            coverImageUrl: magazineTable.coverImageUrl,
            issueDate: magazineTable.issueDate,
            postNumber: magazineTable.postNumber,
            publishedAt: magazineTable.publishedAt,
          })
          .from(magazineTable)
          .where(where)
          .orderBy(desc(magazineTable.issueDate), desc(magazineTable.id))
          .limit(limit)
          .offset(offset),
        db.$count(magazineTable, where),
      ]);

      return { posts: rows, total };
    },
  );
  ```
- `drizzle-orm`에서 `desc`, `isNotNull` 임포트 추가.
- `contents`(jsonb), `author`(account 조인), `draftData` 완전 제외.
- 기존 미사용 `getMagazinePubs` 삭제 (호출자 전무 확인 완료).

### 3. RPC 엔드포인트 갱신 (`src/modules/magazine/magazine.rpc.ts`)
- 기존 `fetchMagazinePubs`를 `fetchMagazinePubList`로 교체:
  ```ts
  export const fetchMagazinePubList = createServerFn()
    .validator(
      z
        .object({
          limit: z.number().int().positive().max(50).optional(),
          offset: z.number().int().nonnegative().optional(),
        })
        .optional(),
    )
    .handler(async ({ data }): Promise<{ posts: MagazinePubListItem[]; total: number }> => {
      return getMagazinePubList(data ?? {});
    });
  ```
- 기존 `getMagazinePubs` import 및 `fetchMagazinePubs` 삭제.

### 4. React Query 옵션 훅 신설 (`src/modules/magazine/hooks/useMagazine.ts`)
- 블로그 `useBlog.ts` 패턴과 동일하게 `useMagazine.ts` 생성:
  ```ts
  import { keepPreviousData, queryOptions } from "@tanstack/react-query";
  import { fetchMagazinePubList } from "@/modules/magazine/magazine.rpc";

  export const MAGAZINE_QUERY_KEY = ["magazine-list"] as const;

  export type MagazineListParams = {
    page: number;
    limit: number;
  };

  export const magazineListQueryKey = (params: MagazineListParams) =>
    [...MAGAZINE_QUERY_KEY, params] as const;

  export const magazineListQueryOptions = (params: MagazineListParams) =>
    queryOptions({
      queryKey: magazineListQueryKey(params),
      queryFn: () =>
        fetchMagazinePubList({
          data: {
            limit: params.limit,
            offset: (params.page - 1) * params.limit,
          },
        }),
      staleTime: 1000 * 30,
      placeholderData: keepPreviousData,
    });
  ```
- 참고: `useMagazineMutations.ts`의 `queryKey: ["magazine-list"]` 무효화와 프리픽스가 일치하여 생성/수정/삭제 시 자동 캐시 갱신됨.

### 5. 카드 컴포넌트 Props 타입 갱신 (`src/modules/magazine/components/magazineCard.tsx`)
- `MagazineCardProps.post` 타입을 `MagazinePub`에서 `MagazinePubListItem`으로 변경:
  ```ts
  import type { MagazinePubListItem } from "@/modules/magazine/magazine.type";

  type MagazineCardProps = {
    post: MagazinePubListItem;
    featured?: boolean;
  };
  ```

### 6. 라우트 Loader 및 페이지 컴포넌트 전환 (`src/routes/_authed/magazine/index.tsx`)
- **Loader**:
  - `loaderDeps: ({ search }) => ({ page: search.page ?? 1 })`
  - `await queryClient.ensureQueryData(magazineListQueryOptions({ page: deps.page, limit: PAGE_SIZE }))`
  - loader 리턴값에서 `posts` 제거 (`canWrite`만 반환).
- **Component**:
  - `const { data: listData, isLoading } = useQuery(magazineListQueryOptions({ page: currentPage, limit: PAGE_SIZE }))`
  - `const posts = listData?.posts ?? [];`
  - `const total = listData?.total ?? 0;`
  - `const totalPages = Math.ceil(total / PAGE_SIZE);`
  - 기존의 `allPosts.sort(...)`, `allPosts.slice(...)` 제거.
  - 인라인 50줄 커스텀 페이지네이션 UI를 기존 공용 `PaginationBar`(`src/components/nav/paginationBar.tsx`)로 교체:
    ```tsx
    <div className="pt-8">
      <PaginationBar
        currentPage={currentPage}
        totalPages={totalPages}
        onPageChange={setCurrentPage}
      />
    </div>
    ```

## Critical Files & Anchors
- `src/modules/magazine/magazine.service.ts`: `getMagazinePubs` (L32-42) 교체 대상.
- `src/modules/magazine/magazine.rpc.ts`: `fetchMagazinePubs` (L29-40) 교체 대상.
- `src/modules/magazine/hooks/useMagazine.ts`: 신설 파일 (React Query 옵션).
- `src/modules/magazine/components/magazineCard.tsx`: `MagazineCardProps` (L8-11) 타입 갱신.
- `src/routes/_authed/magazine/index.tsx`: loader (L24-32) 및 컴포넌트 내부 렌더링/페이징 (L34-217).

## Verification
1. **정적 검증**:
   - `bun run lint:write` 실행 후 오류 없음 확인.
   - `bun run typecheck` 실행 후 타입 에러 0건 확인.
2. **동작 검증 (throwaway node/bun 스크립트 또는 브라우저)**:
   - `getMagazinePubList({ limit: 12, offset: 0 })` 호출 시 `{ posts: [...], total: N }` 형식 반환 확인.
   - 반환 객체에 `contents`, `draftData`, `author` 필드가 없고 `id`, `title`, `coverImageUrl`, `issueDate`만 포함되는지 확인.
   - 브라우저로 `http://localhost:14005/magazine?page=1` 접속하여 목록 정상 렌더링, 페이지 변경(`?page=2`) 시 부드러운 페이지네이션 및 캐싱 동작 확인.

## Assumptions & Contingencies
- **페이지 크기 고정**: 매거진 목록은 그리드 레이아웃(1열 featured + 3열 그리드)에 맞춰 기존 `PAGE_SIZE = 12`를 유지한다.
- **정렬 기준**: 매거진 발행일(`issueDate DESC`)과 ID(`id DESC`) 복합 정렬을 기본으로 한다.
- **`PaginationBar` 전환**: 기존 인라인 수동 페이지네이션 버튼을 공용 컴포넌트(`PaginationBar`)로 대체하여 블로그/피드와 일관된 UX를 제공한다.
