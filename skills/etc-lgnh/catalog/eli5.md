# ELI5 (Explain Like I'm 5)

Explain any complex concept, system, or technology to a complete beginner using intuitive real-world metaphors, big visual diagrams, and minimal text via a self-contained HTML artifact.

## Language

Respond in the same language the user is using. Match the user's language for all explanations and text inside the artifact.

## Arguments

- `topic`: The topic, technology, or question to explain (e.g. `how DNS works`, `kubernetes`, `vector databases`). If omitted, ask the user for the topic before proceeding.

## Core Principles

- **Big Pictures, Few Words**: Emphasize visuals, flow diagrams, and iconography over paragraphs. Keep text bite-sized and punchy.
- **Relatable Metaphor**: Anchor the explanation in a familiar, everyday analogy (e.g. DNS = phone contacts list, API = restaurant waiter, cache = desk drawer vs. archive warehouse).
- **No Unexplained Jargon**: Never use technical buzzwords without immediate everyday translation.
- **Self-Contained HTML Artifact**: Deliver a modern, standalone HTML document with embedded CSS and inline SVGs—zero external font/image dependencies.

## Execution Steps

1. **Clarify Topic & Choose Core Metaphor**
   - Extract the topic from user arguments. If empty, ask what they would like explained.
   - Select one clear, relatable real-world metaphor that maps directly to how the system works.

2. **Structure the Explainer (3 to 4 sections)**
   - **Hero / One-Liner**: What is it in one simple sentence.
   - **The Real-World Metaphor**: Side-by-side comparison (Real Life vs. Tech).
   - **How It Works Step-by-Step**: 3–4 visual stages with numbered steps, simple arrows, and inline SVG illustrations.
   - **Takeaway / Why We Need It**: 1–2 key benefits in plain language.

3. **Build the HTML Artifact**
   - Use clean, modern semantic HTML5 and embedded CSS:
     - Cheerful, friendly color palette (soft pastel backgrounds, clear contrast, rounded cards).
     - System font stack (e.g. `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`).
     - Responsive grid/flex layout that looks great on both mobile and desktop.
     - Inline `<svg>` graphics for diagrams, icons, and flow arrows (no broken external assets).
   - Present the HTML as a complete, copyable, and previewable artifact.

4. **Present to User**
   - Provide the rendered HTML artifact and a 1-2 sentence friendly summary in the user's language.
