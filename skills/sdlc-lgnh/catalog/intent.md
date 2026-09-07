# Stage 1: Intent Capture (intent.md)

Capture business and user intent into `intent.md` as the first version-controlled artifact of the AI-Native SDLC.

## Language

Respond in the same language the user is using (e.g., Korean if the user communicates in Korean). Match the user's language for artifact content and discussions.

## Arguments

- `slug` or `feature-name` (optional): Short hyphen-separated slug for directory naming (e.g., `user-auth`, `PAY-1042-refund`).

## Constraints

- Focus on **"Why & What is the problem, and what is the desired outcome"**, rather than "How to implement".
- Use business and user vocabulary rather than technical implementation jargon.
- Explicitly identify and document constraints and open questions.

## Execution Steps

1. **Resolve Target Directory**
   - Base path convention: `docs/story/YYYY-MM-DD-<slug>/` (or `docs/story/<ticket>-<slug>/`).
   - If argument provided, format `<slug>` appropriately.
   - If argument omitted:
     - Check git branch name (e.g. `feature/user-auth` -> `user-auth`).
     - If no branch hint, prompt the user for a short feature slug.
   - Ensure the directory exists (`mkdir -p <target_dir>`).

2. **Gather Requirements (Interview / Analysis)**
   - Analyze user prompt. If information is missing, clarify:
     - **Problem Definition**: What problem is occurring and who is impacted?
     - **Proposed Outcome**: What will users be able to do, and how is success measured?
     - **Affected Users & Systems**: Target user groups and related systems/components.
     - **Constraints**: Mandatory technical and business boundaries (e.g., security policies, auth mechanisms, latency).
     - **Open Questions**: Unresolved questions to be answered during technical specification.

3. **Write `intent.md`**
   - Save to `<target_dir>/intent.md` using the standard template:
     ```markdown
     # 의도: [기능 또는 변경 명칭]

     - 작성자: [작성자 이름 / 소속]
     - 작성일: YYYY-MM-DD
     - 상태: [초안(draft) | 검토요청(in-review) | 승인됨(approved)]

     ## 1. 문제 정의 (Problem)

     - 현재 어떤 문제가 발생하고 있는가?
     - 누가 불편을 겪고 있으며, 정량적/정성적 영향은 어느 정도인가?

     ## 2. 제안 결과 (Proposed Outcome)

     - 이 작업이 완료되면 사용자는 무엇을 할 수 있게 되는가?
     - 성공 여부를 어떻게 측정할 것인가?

     ## 3. 영향 받는 대상 (Affected Users & Systems)

     - 대상 사용자군
     - 관련 시스템 및 컴포넌트

     ## 4. 제약조건 (Constraints)

     - 반드시 지켜야 할 기술적/비즈니스적 한계

     ## 5. 미해결 질문 (Open Questions)

     - 기획 시점에 아직 결정되지 않아 설계 단계에서 확인해야 할 사항
     ```

4. **Report & Guide Next Step**
   - Output created file path and a concise summary.
   - Guide next step: Inform the user to review/approve the intent artifact, then run `/sdlc-lgnh spec <target_dir>` (or `/sdlc-lgnh spec`) to proceed with Stage 2 Design.
