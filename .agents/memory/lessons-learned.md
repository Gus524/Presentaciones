# Lessons Learned & User Architectural Preferences

This file records user feedback, rejected anti-patterns, and architectural preferences to maintain long-term alignment across sessions.

## Entry 1: Slidev Visual Styling & Diagram Scaling
- **Context/Slide:** Global slide design, typography, spacing, and Mermaid diagrams (Context Map).
- **Rejected Anti-Pattern:** Coarse text layout ("textos toscos"), bad element distribution, overflowing diagrams cut off without scrollability or proper scaling.
- **Corrected Behavior:**
  1. Add a global custom stylesheet (`styles/index.css` or slide-scoped CSS) with refined modern typography (e.g. Inter/Fira Code), clean line heights, elegant padding, and backdrop blur cards.
  2. Implement proper visual hierarchy: readable code font size (`text-[0.75rem]`), compact line spacing, clean margins.
  3. Scale Mermaid diagrams properly (`transform: scale(...)`, responsive svg styling, max-height bounds) or break large diagrams into multi-column/focused views so they never cut off.
  4. Use Slidev layout options and utility classes (`overflow-y-auto`, custom scrollbar, clean grids) to ensure content is fully visible and scrollable if needed.

## Entry 2: Two-Column Slide Alignment & Header Spanning
- **Context/Slide:** Two-column slides (e.g. Slide 1, Slide 4, Slide 14).
- **Rejected Anti-Pattern:** Using standard `layout: two-cols` with titles placed before `<template #left>`, causing headings to collapse into the left column width instead of stretching across the full slide width.
- **Corrected Behavior:** Use `layout: two-cols-header` with `::left::` and `::right::` (or explicit top slot) so headings (`# Title` & `## Subtitle`) span the full width of the slide at the top, and the two columns sit neatly side-by-side underneath.

## Entry 3: Flexbox Scroll Containment & Top Alignment for Diagrams
- **Context/Slide:** Scrollable containers with tall diagrams or flowcharts (Slide 3 Context Map).
- **Rejected Anti-Pattern:** Using `align-items: center` on scrollable flex containers (`overflow: auto`), which centers tall SVGs vertically and pushes top elements (like `IAM BC`) into negative scroll space where browsers cannot scroll up to see them.
- **Corrected Behavior:**
  1. Use `align-items: flex-start !important;` on `.mermaid-container` and `.mermaid` so content always starts aligned at `top=0`.
  2. Use `{scale: 0.75}` or `{scale: 0.7}` on Slidev Mermaid code blocks so large diagrams scale responsively to fit on-screen.

## Entry 4: Mermaid Diagram Arrow Layout & Layering
- **Context/Slide:** Bounded Context Map (Slide 3).
- **Rejected Anti-Pattern:** Overlapping arrow text labels caused by multiple parallel connections from a single node to many individual sub-nodes with long label strings.
- **Corrected Behavior:**
  1. Organize flowcharts into explicit architectural layers (Layer 1: Generic, Layer 2: Supporting Providers, Layer 3: Core Domains, Layer 4: Accounting).
  2. Use `direction LR` on inner subgraphs so nodes within the same domain line up side-by-side.
  3. Simplify connections to entry-point nodes or group targets to avoid line collisions.
  4. Add custom CSS classes (`classDef core`, `classDef support`, `classDef generic`) for instant visual distinction between domain types.

## Entry 5: Horizontal Scrollable Diagram Container & Font Enlargement (No Bold, Emoji Removal)
- **Context/Slide:** Horizontal LR Flowchart diagrams (Slide 3 Context Map).
- **Rejected Anti-Pattern:** Forcing heavy bold fonts (`font-weight: 800`) or placing inline emojis inside large-font node labels, causing text clipping and overlap inside node boxes.
- **Corrected Behavior:**
  1. Set large font sizes strictly for Mermaid (`30px` nodes, `24px` edge labels, `26px` titles) with normal font weight (`font-weight: 400` / `500`).
  2. Remove inline emojis from node labels when font size is >= 24px so text renders cleanly with generous internal node padding (`12px 24px`).
  3. Maintain `.mermaid-scrollable` horizontal scroll container to prevent compression while preserving clean typography.

## Entry 7: Card Column Width Sizing & Vertical Scroll Containment
- **Context/Slide:** Two-column slides (e.g., slides 2, 3, 5, 14, 17) and tall content slides.
- **Rejected Anti-Pattern:** Cards overflowing horizontally so the second column is hidden or cut off off-screen; cards with tall content getting truncated at the bottom.
- **Corrected Behavior:**
  1. Add `w-full overflow-hidden` or explicit max-width/flex containment (`w-full flex-1`) on two-column slots (`::left::` and `::right::`).
  2. Implement vertical scrollable containers (`max-h-[420px] overflow-y-auto pr-1` or `.scrollable-card`).

## Entry 8: Vertical Stacked Layout for Code Comparison Slides
- **Context/Slide:** Code comparison slides (Slide 8 Backend CQRS, Slide 9 Domain Rich Model, Slide 10 Angular Signals).
- **Rejected Anti-Pattern:** Forcing side-by-side two-column layouts for code comparisons with different prompts.
- **Corrected Behavior:** Use single-column layout with full-width cards stacked vertically.

## Entry 9: Slide-Level Vertical Scrolling & Generous Grid Gaps
- **Context/Slide:** Global slide layouts (`styles/index.css` and all slides in `slides.md`).
- **Rejected Anti-Pattern:** Cramped horizontal gaps between side-by-side cards.
- **Corrected Behavior:** Enable page-level scrolling and use generous gaps.

## Entry 10: Two-Column Header Margin & Explicit Column Gap
- **Context/Slide:** Two-column horizontal slides (`layout: two-cols-header`).
- **Rejected Anti-Pattern:** Subtitle touching the top border of cards (`margin-bottom: 0`); left and right cards touching border-to-border (`gap: 0`).
- **Corrected Behavior:** Use explicit 2-column grid layout with 32px gap and header margin.

## Entry 11: Avoid Fragile Positional Selectors in Slidev Layouts
- **Context/Slide:** `styles/index.css` for Slidev `two-cols-header` layouts.
- **Rejected Anti-Pattern:** Using positional child selectors like `> div:nth-child(2)` on `.two-cols-header`.
- **Corrected Behavior:** Define `.slidev-layout.two-cols-header` as a clean 2-column grid.

## Entry 12: Native Mermaid Code Blocks & Clean HTML Parsing in Slidev
- **Context/Slide:** Mermaid diagrams (Slide 20) and HTML list structures across slides.
- **Rejected Anti-Pattern:** Using raw `<div class="mermaid">` HTML tags instead of fenced ` ```mermaid ` code blocks.
- **Corrected Behavior:** Use standard fenced ` ```mermaid ` code blocks for all diagrams.

## Entry 13: Left-to-Right Mermaid Pipeline Flow & QA Error Feedback Loop
- **Context/Slide:** SDD Pipeline Diagram (Slide 20).
- **Rejected Anti-Pattern:** Vertical `graph TD` flow causing phases to stack out of numerical order; omitting the QA failure feedback loop.
- **Corrected Behavior:**
  1. Use `graph LR` (Left-to-Right) and link phase subgraphs sequentially (`P1 --> P2 --> P3 --> P4 --> P5`) to force an ordered horizontal progression.
  2. Include the QA FAIL feedback loop (`DQTE -.->|QA FAIL| HITL_QA -.->|Fix Task| P4`) showing the error classification path back to tactical agents or design.

## Entry 14: Central Orchestrator Dispatch & Claim-Check Response Flow in Mermaid
- **Context/Slide:** SDD Pipeline Diagram (Slide 20).
- **Rejected Anti-Pattern:** Showing sub-agents operating autonomously without visual links to the central orchestrator.
- **Corrected Behavior:** Show `global-architect-orchestrator` dispatching sub-agents and receiving payloads.

## Entry 15: Simplified Human-Centric Pipeline Diagram & Dual HITL Decision Gates
- **Context/Slide:** SDD Pipeline Diagram (Slide 20).
- **Rejected Anti-Pattern:** Over-saturating diagrams with repetitive orchestrator arrows in every subgraph.
- **Corrected Behavior:**
  1. Simplify pipeline flow sequentially (`Phase 1 ➔ Phase 2 ➔ Phase 3 ➔ Phase 4 ➔ Phase 5`).
  2. Prominently highlight the 2 Human-in-the-Loop decision gates: `👤 HITL Gate 1` (Branch A vs B selection after Scout) and `👤 HITL Gate 2` (Error classification & retry routing after QA failure).
  3. Keep orchestrator context implicit or backgrounded for maximum visual clarity (`scale: 0.7`).

## Entry 16: Vue Directive Wrapper Syntax & HTML List Formatting in Slidev
- **Context/Slide:** Slide 11 (`v-click` animation cards) and Slide 17 (Agent catalog list).
- **Rejected Anti-Pattern:** Placing `v-click` directly as an inline attribute on raw HTML `<div v-click class="...">`, which breaks HTML element parsing and outputs `<p class="...">` as plain text; using indented Markdown lists inside HTML `<div>` blocks without explicit `<ul>`/`<li>` tags.
- **Corrected Behavior:**
  1. Wrap interactive elements in explicit Slidev `<v-click><div class="...">...</div></v-click>` tags.
  2. Use clean HTML `<ul class="list-disc pl-4 space-y-2"><li>...</li></ul>` structures inside HTML cards for 100% reliable DOM rendering.

## Entry 17: Replace `<p>` tags with `<div>` inside HTML cards in Slidev
- **Context/Slide:** Slide 10 and Slide 11 (`Evaluación Crítica`).
- **Rejected Anti-Pattern:** Using `<p class="...">` tags inside custom HTML card containers, which causes Markdown-it in Slidev to render literal `<p class="...">` raw text on screen.
- **Corrected Behavior:** Use `<div class="...">` tags instead of `<p>` inside HTML containers for clean, raw-text-free rendering.

## Entry 18: Do NOT Wrap Fenced Code Blocks (` ``` `) Inside HTML `<div>` Tags in Slidev
- **Context/Slide:** Slides de "Casos Prácticos" (Slide 8 Backend CQRS, Slide 9 Dominio, Slide 10 Frontend Signals).
- **Rejected Anti-Pattern:** Wrapping fenced code blocks (` ```csharp ` / ` ```ts `) inside HTML `<div class="card-container">` blocks, which causes Markdown-it to abort HTML container parsing and dump literal `<div class="card-container">` raw text on screen.
- **Corrected Behavior:** Place fenced code blocks (` ```csharp ` / ` ```ts `) directly in Markdown, with header badges/prompts in clean HTML above/below the code blocks outside any enclosing `<div>`.























