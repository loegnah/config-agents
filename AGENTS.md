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

<!-- graft:start -->
## Graft

Repo context graph in `graft/`. Check before grep/read.

- `graft ask "<query>" --source`: Find & understand code spans.
- `graft grep "<literal>"`: Exhaustive search across indexed symbols.
- `graft callers <symbol> [--direction out] [--depth N]`: Call hierarchy & blast radius.
- `graft skeleton <file>`: Signatures & line spans for a file.
- `graft map`: Repo orientation (clusters, hubs).
- `graft build`: Rebuild graph after large changes.
<!-- graft:end -->
