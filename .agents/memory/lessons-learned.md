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

## Entry 6: Cover Slide Background Contrast & Text Avoidance
- **Context/Slide:** Cover slide background image (`/cover-bg.jpg`).
- **Rejected Anti-Pattern:** Using background images with baked-in text/typography or light/busy graphics that obscure white title text.
- **Corrected Behavior:** Generate or select dark navy/slate geometric textures completely free of text or words, providing maximum contrast and legibility for white presentation typography.
