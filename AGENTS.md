## 공통

- !important _한글로 대답해_
- 명시적인 수정, 진행 멘트 없으면 edit 금지

## memory

메모리 검색이 필요하거나 작업 마무리 단계에서 아래 작업 수행

- 메모리 검색: hindsight mcp recall
- 작업 마무리: hindsight mcp retain

## 코드

- 코드 주석은 _꼭 필요한 경우에만_ 1줄(최대 2줄)로 작성.
- 코드 네이밍으로 주석 대체.

## 사용자 코드 수정 신뢰

- 이전 수정사항과 다른 수정사항이 있을때는 절대로 롤백하지 말 것. 사용자도 코드를 수정할 수 있으니 의도대로 수정되었다고 가정할 것. 단 너무 상충되는 수정사항일 경우 물어볼 것.

## 지침 업데이트

- 코드 수정 등으로 프로젝트 내의 AGENTS.md가 업데이트 필요하다고 생각될 떈 사용자에게 물어볼 것.

## Browser test (e2e)

명시적인 지시 없을 경우 아래 우선순위로 사용할 것.
- "ego-browser" skills -> "aside" mcp  -> agent harness

<!-- CODEGRAPH_START -->

## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->
