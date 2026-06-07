# HTML Learning Map Format (spec-walkthrough)

The learning map is a single self-contained HTML file written next to the markdown checklist at `docs/raki/learning/<date>-<topic>-spec.html`. Tailwind and Mermaid both come from CDNs — no build step. It is a **visual companion** the developer keeps open while being taught the spec: it maps the spec onto the code and shows live mastery progress.

**Regenerate-on-tick.** The file is static — there is no live state. Whenever a checklist item is mastered, rewrite the whole file so the progress reflects reality. The only scripts allowed are the Tailwind CDN and the Mermaid ESM import. No app code, no interactivity beyond Mermaid's own rendering.

Mermaid handles graph-shaped diagrams reliably (requirement trees, spec→code mappings, dependency graphs); hand-built divs and inline SVG handle the more editorial visuals (gap/decision boards). Mix the two — don't lean on Mermaid for everything, it starts to look generic.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Spec walkthrough — {{topic}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .seam   { stroke-dasharray: 4 4; }
      .gap    { stroke: #d97706; stroke-width: 2px; }
      .done   { text-decoration: line-through; opacity: 0.55; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="progress">...</section>     <!-- mastery checklist -->
      <section id="intent">...</section>        <!-- pillar 1 -->
      <section id="codebase-map">...</section>  <!-- pillar 2 -->
      <section id="gaps">...</section>          <!-- pillar 3 -->
    </main>
  </body>
</html>
```

## Header

Topic, date, the spec's source (PRD/ticket link or filename), and a compact progress meter — `N / M mastered` plus a thin bar. No introduction paragraph.

## Progress section (the mastery checklist)

The checklist rendered visually, grouped by the three pillars. Each item is a row:

- A state marker — empty circle (unmastered) or filled emerald check (mastered, apply `.done`).
- The concept, one short line.
- Optional: a tiny quiz tag (`quizzed ✓`) once verified.

This section is the visual twin of the `.md` checklist — they must always agree. Re-emit on every tick.

## The three pillars

Each pillar is one `<section>`. Diagrams carry the weight; prose is sparse and uses the spec's own vocabulary (and `CONTEXT.md` terms if the project has them).

### 1. Intent — what the spec asks & why

A **requirements tree** (Mermaid `flowchart` / `mindmap`): the spec's goal at the root, branching into its requirements and the constraints/acceptance criteria under each. This makes scope and the "why" legible at a glance. Tag any acceptance criterion that's testable.

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart TD
      G[Goal: users can reset password] --> R1[Req: email reset link]
      G --> R2[Req: link expires in 1h]
      G --> R3[Req: rate-limit requests]
      R2 --> C1[Acceptance: expired link → 410]
      classDef gap stroke:#d97706,stroke-width:2px;
  </pre>
</div>
```

### 2. Codebase map — where the spec lands

The centrepiece. A **Mermaid graph** mapping spec requirements onto the real code: requirement nodes on one side, the modules/files/seams they touch on the other, edges connecting them. Colour edges that touch existing behavior (a change, not an addition) so the blast radius is obvious. This is the pillar the `Explore` pass feeds — name real files (`font-mono`).

```html
<pre class="mermaid">
  flowchart LR
    R1[email reset link] --> M1["auth/mailer.ts"]
    R3[rate-limit] --> M2["middleware/rateLimit.ts (exists)"]
    R2[1h expiry] --> M3["auth/tokens.ts (change)"]
    classDef change stroke:#dc2626,stroke-width:2px;
    class M3 change
</pre>
```

### 3. Gaps — open decisions & edge cases

A **hand-built gap board**: cards in a grid, one per ambiguity / open decision / edge case the spec leaves unresolved. Each card: the question, why it matters, and where it bites in the code. Amber-tint (`.gap`) the ones that block implementation. These are surfaced for hand-off, not resolved here — make that visible (e.g. a "→ grill-me" tag).

## Diagram patterns

Pick what fits; mix them; don't make every diagram look the same.

- **Requirements tree** (Mermaid `flowchart`/`mindmap`) — goal → requirements → acceptance criteria. Pillar 1.
- **Spec→code map** (Mermaid `graph`) — requirements wired to the modules they touch, changes highlighted. Pillar 2, the workhorse.
- **Gap board** (hand-built div grid) — open decisions and edge cases as cards. Pillar 3.
- **Sequence diagram** (Mermaid) — when a requirement is about ordering/flow ("link issued → clicked → validated → expired").
- **Cross-section** (stacked `h-12 border-l-4` bands) — for a request path the spec changes layer by layer.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. `font-serif` headings pair well with stone/slate.
- Colour sparingly: one accent (emerald or indigo), red for code that changes, amber for gaps/open decisions.
- Keep diagrams ~320px tall so they sit comfortably without scrolling.
- Module/file labels: `font-mono text-sm`; schematic labels inside diagrams: `text-xs uppercase tracking-wider`.
- If a diagram needs a paragraph to be understood, redraw the diagram.

## Tone

Plain English, concise. Use the spec's own terms and the project's `CONTEXT.md` vocabulary — don't invent synonyms. Every diagram should answer a *why*, *what*, or *where-in-the-code* the developer needs. No throat-clearing. If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it.
