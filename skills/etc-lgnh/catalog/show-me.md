# Show Me (Visual Explainer)

Help the user understand the current topic or code visually with concise diagrams, code-shape sketches, and focused HTML artifacts. Skip the preamble and keep prose brief. Pick the smallest view that makes the key point clear.

## Language

Respond in the same language the user is using. Match the user's language for all explanations and labels.

## Arguments

- `topic` (optional): The concept, architecture, code flow, or file changes to visualize. If omitted, visualize the active topic or recent changes in the current session.

## Core Visual Patterns

Choose the smallest view that makes the key point clear:

1. **Logic / Algorithm**: Concise pseudocode

   ```text
   on(save)
     if content is unchanged
       return cached result
     write new content
     return fresh result
   ```

2. **Runtime Control Flow**: Hierarchical call tree

   ```text
   submitForm
     createSession
       persistPrompt
       launchAgent
     navigateToSession
   ```

3. **UI Structure**: Component tree with boundaries and relevant state

   ```tsx
   <SessionPage> (apps/example/src/routes/session.tsx)
     useSessionEvents()
     <SessionToolbar>
       <RunSkillButton> (packages/ui)
   ```

4. **File Architecture / Refactor**: Shallow file tree

   ```text
   src/
   ├── commands/       # parses user actions
   ├── sessions/       # owns session state
   └── transport/      # sends API requests
   ```

5. **Interaction / Flow**: Mermaid diagram

   ```mermaid
   sequenceDiagram
       participant User
       participant UI
       participant Daemon
       User->>UI: choose command
       UI->>Daemon: send expanded prompt
       Daemon-->>UI: stream result
   ```

6. **Changes / Transitions**: Targeted diff matching the topic shape
   - Component change:
     ```diff
      <SessionPage>
        useSessionEvents()
        <SessionToolbar>
     +    <RunSkillButton />
        <SessionTimeline>
     +    <SkillResultCard />
     ```
   - File layout change:
     ```diff
      src/
      ├── commands/
     +│   └── show-me.ts
      ├── sessions/
     -└── transport.ts
     +└── transport/
     +    ├── client.ts
     +    └── stream.ts
     ```
   - Call tree change:
     ```diff
      submitForm
        createSession
          persistPrompt
     +    expandSkillMention
          launchAgent
     -  navigateToSession
     +  navigateToSession
     +    subscribeToEvents
     ```
   - State / Control flow change:
     ```diff
      on(save)
     -  write content
     +  if content is unchanged
     +    return cached result
     +  write new content
     +  invalidate cache
     ```

7. **Target Code**: Whole block when mostly new or context/order matters

   ```ts
   function expandSkill(command: string): string {
     const skillName = command.slice(1);
     return `use the ${skillName} skill`;
   }
   ```

8. **Dense / UI Concepts**: Self-contained HTML artifact
   - When a concept is too dense for Mermaid or requires visual UI/layout/infographic preview.
   - Use responsive HTML5, embedded CSS, and inline SVGs (no external asset dependencies).
   - Write to a local HTML file or render directly as an artifact.

## Guidance

- Place each visual next to the short text it supports.
- Keep only the calls, files, props, states, and boundaries needed to answer the question.
- Use one or a few views; do not overwhelm the user with every possible format.

## Execution Steps

1. **Clarify Topic**
   - Extract topic from arguments. If omitted, use current conversation context or active file changes.
2. **Select Representation**
   - Pick 1–2 most appropriate visual patterns from above (pseudocode, call tree, component tree, file tree, mermaid, diff, code block, or HTML artifact).
3. **Draft Visualization**
   - Generate clean, focused visuals with minimal surrounding prose.
4. **Deliver**
   - Present the visual directly to the user with a 1–2 sentence summary.
