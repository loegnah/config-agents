# Stage 3: Build Plan (plan.md)

Construct a concrete, file-level implementation plan based on `spec.md` in Plan Mode before writing any application code.

## Language

Respond in the same language the user is using (e.g., Korean if the user communicates in Korean). Match the user's language for artifact content and discussions.

## Arguments

- `target` (optional): Path to target story directory (e.g. `docs/story/2026-09-06-feature/`).

## Constraints

- **Strict Plan Mode (No code modifications)**: Do NOT modify any application or source files during this step. Focus purely on writing `plan.md`.
- Detail the exact file paths and work order so that any third-party engineer could implement it identically.
- Must include changing files, sequential steps, risk mitigations, and automated proof criteria.

## Execution Steps

1. **Locate Target Directory & `spec.md`**
   - If argument provided, resolve `<target_dir>`.
   - If omitted:
     - Search `docs/story/` for the most recently modified directory containing `spec.md`.
   - Read `<target_dir>/spec.md` and `<target_dir>/intent.md`. If `spec.md` is missing, instruct user to run `/sdlc-lgnh spec` first.

2. **Blast Radius & Codebase Investigation**
   - Investigate exact files that will need to be created, modified, or tested.
   - Run LSP / grep / file reading tools to check symbols, callsites, dependencies, and potential regressions.
   - Clarify 3 core planning questions:
     1. What is the most dangerous functionality that could break with this change?
     2. What alternatives were considered and why were they rejected?
     3. What is the minimal automated proof (test/check) required to verify success?

3. **Write `plan.md`**
   - Save to `<target_dir>/plan.md` using the standard template:
     ```markdown
     # 구현 계획: [기능 명칭] (기반: spec.md YYYY-MM-DD)

     - 담당 엔지니어: [이름]
     - 상태: [계획중(planning) | 승인됨(approved) | 구현완료(completed)]

     ## 1. 변경 대상 파일 (Files that change)

     - [생성] `경로/파일명`: 생성 목적 및 역할
     - [수정] `경로/파일명`: 변경 상세
     - [테스트] `경로/파일명`: 단위/통합 테스트 범위

     ## 2. 작업 순서 (Order of Work)

     1. 작업 1단계 세부 내용
     2. 작업 2단계 세부 내용
     3. 작업 3단계 세부 내용

     ## 3. 잠재 위험 및 대응 방안 (Risks & Mitigations)

     - 위험: 예상되는 위험 요소
     - 대응: 사전 방지 및 폴백 방안

     ## 4. 검증 및 완료 증거 (Proof)

     - 자동화 테스트: 실행할 테스트 명령어 (예: `pytest tests/...`, `bun test ...`)
     - 린트/빌드: 실행할 린트/체크 명령어 (예: `bun run check`)
     - 수동/시각적 확인: UI 목업 대조, 콘솔 확인 등
     ```

4. **Report & Guide Next Step**
   - Output `plan.md` summary, highlighting risk mitigations and proof criteria.
   - Guide next step: Inform the user to review/approve the plan, then run `/sdlc-lgnh run <target_dir>` (or `/sdlc-lgnh run`) to begin implementation.
