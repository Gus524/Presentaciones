---
trigger: always_on
---

# User Feedback Alignment & Dynamic Memory Protocol (Slidev Presentation Deck)

## Core Mandate
You operate under a strict human-in-the-loop paradigm. Your primary goal is aligning your execution plans with the user's stylistic choices, presentation structure, visual layout, and Slidev conventions (Markdown, Vue 3 components, UnoCSS/Tailwind, and Frontmatter configurations). You MUST prioritize structural alignment and visual intent validation over raw execution speed.

## Protocols

### 1. Retrospective Initialization (The Memory Check)
- **Mandatory First Step:** At the start of any new session, task initialization, or slide editing step, check the dynamic memory file located at `.agents/memory/lessons-learned.md` (if it exists).
- Analyze past presentation corrections, rejected styling anti-patterns (e.g., bad layouts, inconsistent transitions, incorrect UnoCSS classes, invalid frontmatter), and direct instructions recorded there to avoid repeating past mistakes.

### 2. The Planning Gate (Slide Layout Blueprint & Execution Mode)
- **Direct Interactive Mode:** When receiving a request to create or refactor slides, presentation sections, custom Vue components, or thematic styles, present a conceptual blueprint (Slide Structure, Layout types, Content hierarchy, Component integration, Animations/Clicks) and prompt the user for validation before modifying files (`slides.md`, `components/*.vue`, `layouts/*.vue`, `styles/*.css`).
- **Delegated & Automated Mode (Pre-Approved Tasks):** When executing a batch or design plan already approved by the user, proceed directly with code/markdown generation adhering strictly to Slidev standards and the established theme.

### 3. Dynamic Learning Protocol (Lessons Learned Updates)
- Whenever the user issues a visual correction, layout adjustment, or structural critique on your Slidev setup or slides:
- **Action:** Append a new entry to `.agents/memory/lessons-learned.md`.
- **Entry Structure:**
  1. **Context/Slide:** (e.g., Title Slide, Code Demo Layout, Vue Component, UnoCSS utility).
  2. **Rejected Anti-Pattern:** (e.g., Overcrowding text on single slide, missing frontmatter layout, hardcoded pixel styles).
  3. **Corrected Behavior:** (e.g., Split into multiple slides via `---`, use grid layout classes, integrate `v-click` for progressive disclosure).
- Acknowledge this update to the user: *"I have recorded this lesson in `.agents/memory/lessons-learned.md` to avoid repeating this mistake."*

### 4. Slidev Code Generation Standards
When generating or modifying presentation assets, enforce the following core principles:
- **Markdown & Frontmatter (`slides.md`):** Use native Slidev YAML frontmatter delimiters (`---`) correctly. Ensure layout names (`cover`, `intro`, `two-cols`, `center`, `default`) match available layouts.
- **Interactivity & Progress:** Use Slidev built-in directives like `v-click`, `v-clicks`, and `v-after` for pacing.
- **Custom Components (`components/`):** Write clean, single-file Vue 3 components (`<script setup>`, `<template>`). Rely on Slidev auto-importing.
- **Styling (`styles/`, UnoCSS/Tailwind):** Prioritize utility classes for responsive spacing, text sizing, and color contrast suitable for presentations. Avoid rigid absolute positioning unless required by specific graphic overlays.
- **Code Snippets:** Use Slidev code blocks with line highlighting and transitions syntax (e.g., ````ts {all|2-3|5}````) where applicable.
