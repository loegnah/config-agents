# Wiki List Performance Optimization Plan (Priority #2)

## Context

`docs/plan/perf-audit-6-areas.md`의 우선순위 2번 항목으로, 위키 목록 조회에서 발생하는 중복 전량 스캔(P0-1)과 필터 적용 시 DB 페이징 무력화 및 본문 overfetch(P0-2)를 해소한다.
현재 위키 목록(`useWikiPubInfinite`)과 건수(`useWikiPubCount`)는 검색/태그/날짜 필터 적용 시 DB `limit`/`offset`을 무력화하고 전체 행(무거운 Plate.js 에디터 `contents` jsonb 포함)을 전량 fetch한 뒤 메모리 JS에서 필터링 및 `.length`를 산출하여 동일 전량 스캔이 2회 중복 발생하고 있다.
이 계획은 (1) DB SQL COUNT 전용 서비스 분리, (2) 검색/태그/날짜 필터의 DB WHERE 절 이전 및 DB 페이징 복원, (3) 목록 결과에 `stripLangContents`를 적용하여 거대한 에디터 본문 트리를 제외한 경량 프로젝션을 반환하는 최적화를 구현한다.

---

## Approach

### 1. `stripLangContents` 공통 적용 및 본문 제외 (P0-2 Overfetch 해소)

- **위치**: `src/modules/wiki/wiki.service.ts`
- **변경 사항**:
  - 기존 파일 내부에 있던 `stripLangContents(lc: WikiLangContents | null | undefined): WikiLangContents` 헬퍼(title, description만 남기고 `contents: []`로 초기화)를 최상단 유틸 영역으로 이동 또는 export.
  - `fetchWikiPubRows`의 반환 매핑 단계에서 `pubVersion`에 `stripLangContents(row.pubVersion?.langContents)`를 적용.
  - 이를 통해 `getWikiPubs`(일반 목록), `getWikiPubsByAuthor`(작성자 목록), `getWikiPubsByIds`(검색엔진 결과 ID 목록) 등 모든 목록성 쿼리에서 무거운 Plate.js JSONB 본문(`contents`)이 제거되어 SSR 및 클라이언트 페이로드가 극적으로 축소됨.
  - 단건 상세 조회(`getWikiPub`)는 `getWikiByStatus`를 거치므로 본문 전체가 그대로 유지됨(영향 없음).

### 2. DB WHERE 절 필터 빌더 구현 (`buildWikiPubWhere`)

- **위치**: `src/modules/wiki/wiki.service.ts`
- **구현 내용**:
  ```ts
  function buildWikiPubWhere(options: {
    accountId?: number;
    bookmarkedWikiIds?: number[];
    mineOnly?: boolean;
    verifiedOnly?: boolean;
    search?: string;
    tags?: string[];
    createdFrom?: Date;
    updatedFrom?: Date;
    idIn?: number[];
  }): SQL | undefined
  ```
  - 기본 조건: `status = WIKI_STATUS.PUBLISHED`, `pubVersionId IS NOT NULL`.
  - `mineOnly`: `options.mineOnly && options.accountId != null`일 때 `eq(wikiTable.authorId, options.accountId)`.
  - `verifiedOnly`: `options.verifiedOnly`일 때 `eq(wikiTable.isVerified, true)`.
  - `idIn` / `bookmarkedWikiIds`: `inArray(wikiTable.id, ids)`.
  - `createdFrom`: `gte(wikiTable.createdAt, options.createdFrom)`.
  - `updatedFrom`: `gte(wikiTable.updatedAt, options.updatedFrom)`.
  - `tags`: Drizzle ORM의 `arrayOverlaps(wikiVersionTable.tags, options.tags)`를 사용한 `EXISTS` 조건:
    ```ts
    sql`EXISTS (
      SELECT 1 FROM ${wikiVersionTable}
      WHERE ${wikiVersionTable.id} = ${wikiTable.pubVersionId}
        AND ${arrayOverlaps(wikiVersionTable.tags, options.tags)}
    )`
    ```
  - `search`: 검색엔진 미사용 또는 폴백 시 DB 레벨 ILIKE 조건(블로그 패턴과 동일):
    ```ts
    const escaped = search.replace(/[\\%_]/g, "\\$&");
    const pattern = `%${escaped}%`;
    sql`EXISTS (
      SELECT 1 FROM ${wikiVersionTable}
      WHERE ${wikiVersionTable.id} = ${wikiTable.pubVersionId}
        AND (
          ${wikiVersionTable.langContents}->'ko'->>'title' ILIKE ${pattern} ESCAPE '\\' OR
          ${wikiVersionTable.langContents}->'en'->>'title' ILIKE ${pattern} ESCAPE '\\' OR
          ${wikiVersionTable.langContents}->'ko'->>'description' ILIKE ${pattern} ESCAPE '\\' OR
          ${wikiVersionTable.langContents}->'en'->>'description' ILIKE ${pattern} ESCAPE '\\' OR
          EXISTS (
            SELECT 1 FROM unnest(${wikiVersionTable.tags}) AS t(name)
            WHERE t.name ILIKE ${pattern} ESCAPE '\\'
          )
        )
    )`
    ```

### 3. DB 전용 단일 COUNT 서비스 함수 추가 (`getWikiPubsCount`)

- **위치**: `src/modules/wiki/wiki.service.ts`
- **시그니처**:
  ```ts
  export const getWikiPubsCount = createServerOnlyFn(
    async (params: {
      accountId?: number;
      bookmarkedOnly?: boolean;
      mineOnly?: boolean;
      verifiedOnly?: boolean;
      search?: string;
      tags?: string[];
      createdFrom?: Date;
      updatedFrom?: Date;
    }): Promise<number>
  )
  ```
- **구현**:
  - `mineOnly`인데 `accountId == null`이면 0 즉시 반환.
  - `bookmarkedOnly`이면 `getUserBookmarkedWikiIds(accountId)` 조회 후 비어있으면 0 반환, 있으면 IDs를 `buildWikiPubWhere`에 전달.
  - `where` 조건 빌드 후 `db.select({ count: sql<number>\`count(*)::int\` }).from(wikiTable).where(where)` 단일 집계 쿼리 실행 및 반환.
  - 기존 수백~수천 개 행 풀스캔 + 메모리 `.length` 패턴 완전 제거.

### 4. `getWikiPubs` 및 `fetchWikiPubRows`의 DB 페이징 복원

- **위치**: `src/modules/wiki/wiki.service.ts`
- **시그니처 확장**:
  - `getWikiPubs` 파라미터에 `search?: string`, `tags?: string[]`, `createdFrom?: Date`, `updatedFrom?: Date` 추가.
  - `fetchWikiPubRows`에 `whereCondition?: SQL` 추가.
- **로직 변경**:
  - `buildWikiPubWhere`로 생성된 `whereCondition`을 `fetchWikiPubRows`에 전달.
  - `fetchWikiPubRows`의 `db.query.wikiTable.findMany` 호출 시 `where: whereCondition`을 적용하여 DB 레벨에서 필터링과 `limit`/`offset` 페이징이 항상 함께 동작하도록 보장.
  - 북마크 순 정렬(`sort === WIKI_SORT.BOOKMARKED`) 시:
    - 필터가 있는 경우: `bookmarkWikiIds`를 조건에 포함한 `whereCondition`으로 필터링된 행을 조회하고 `bookmarkWikiIds` 순서로 정렬 후 `slice(start, start + limit)` 적용.
    - 필터가 없는 경우: 기존의 `bookmarkWikiIds.slice(start, start + limit)` 후 ID 조회 최적화 유지.

### 5. `wiki.rpc.ts` 핸들러 리팩터링

- **위치**: `src/modules/wiki/wiki.rpc.ts`
- **`getWikiPubsRpc`**:
  - `hasFilters ? undefined : limit` 및 `filterWikiPubs` 제거.
  - 검색엔진 분기(`search && isSearchEnabled()`): 기존 유지 (성공 시 IDs로 `getWikiPubsByIds` 조회).
  - 일반 / 폴백 분기:
    ```ts
    return getWikiPubs({
      limit,
      offset,
      sort,
      accountId: user.accountId,
      bookmarkedOnly,
      mineOnly,
      verifiedOnly,
      search,
      tags,
      createdFrom,
      updatedFrom,
    });
    ```
- **`getWikiPubsCountRpc`**:
  - `const all = await getWikiPubs(...); filterWikiPubs(all, ...).length` 제거.
  - 검색엔진 분기(`search && isSearchEnabled()`): `searchWikiIds({ ..., limit: 1 })` 호출 후 `result.total` 반환.
  - 일반 / 폴백 분기:
    ```ts
    return getWikiPubsCount({
      accountId: user.accountId,
      bookmarkedOnly,
      mineOnly,
      verifiedOnly,
      search,
      tags,
      createdFrom,
      updatedFrom,
    });
    ```

---

## Critical files & anchors

1. `src/modules/wiki/wiki.service.ts`:
   - `buildWikiPubWhere` 헬퍼 함수 신설
   - `getWikiPubsCount` 서비스 함수 신설 (DB SQL COUNT)
   - `fetchWikiPubRows`: `whereCondition` 지원 및 `stripLangContents` 매핑 적용
   - `getWikiPubs`: 검색/태그/날짜 파라미터 수용 및 DB 페이징 호출

2. `src/modules/wiki/wiki.rpc.ts`:
   - `getWikiPubsRpc`: `hasFilters` 기반 limit 무력화 및 JS `filterWikiPubs` 제거, `getWikiPubs`로 직결
   - `getWikiPubsCountRpc`: `getWikiPubs(전량)` + `filterWikiPubs.length` 제거, `getWikiPubsCount`로 직결

3. `src/modules/wiki/wiki.util.ts`:
   - `filterWikiPubs` 함수 자체는 기존 단위 테스트 및 호환성을 위해 유지

---

## Verification

1. **단위 및 기존 테스트 검증**:
   - `bun test src/modules/wiki/wiki.util.test.ts` 실행하여 기존 유틸 테스트 통과 확인.
2. **타입 및 린트 검증**:
   - `bun run typecheck` 실행하여 RPC, Service, 훅 간 타입 불일치 없음 확인.
   - `bun run lint:write` 실행하여 코드 스타일 일치 확인.
3. **동작 검증 (Throwaway 스크립트 또는 Vitest)**:
   - 태그 필터(`tags: ["태그명"]`), 날짜 필터(`updatedFrom`), 검색어 필터(`search`)를 전달했을 때:
     - `getWikiPubsCount`가 전체 행 전송 없이 정확한 숫자를 반환하는지 확인.
     - `getWikiPubs` 호출 결과로 반환된 각 항목의 `pubVersion.langContents.ko.contents`가 빈 배열(`[]`)로 축소되어 본문이 제외되었는지 확인.
     - `limit: 5, offset: 0` 전달 시 정확히 최대 5건만 반환되는지 확인.

---

## Assumptions & contingencies

- **`search` DB 폴백 ILIKE 범위**: 검색엔진 비활성화 시 제목(`title`), 설명(`description`), 태그(`tags`)를 대상으로 대소문자 무시 ILIKE 검색을 수행한다.
- **`stripLangContents`의 `contents: []`**: `WikiPub` 타입(`WikiVersion.langContents`)과의 하위 호환성을 완벽히 유지하기 위해 언어별 `contents` 필드를 빈 배열로 설정한다. `WikiExploreItem`의 `bodyText`는 빈 문자열이 되며 카드는 `title`과 `description`, `tags`를 정상 렌더링한다.
- **`filterWikiPubs` 보존**: 클라이언트 및 RPC에서 제거되더라도 `wiki.util.ts` 내 함수는 삭제하지 않고 보존하여 기존 단위 테스트를 온전히 유지한다.
