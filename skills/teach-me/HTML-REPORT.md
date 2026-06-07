# HTML Learning Map Format

The learning map is a single self-contained HTML file written next to the markdown checklist at `docs/raki/learning/<date>-<topic>.html`. Tailwind and Mermaid both come from CDNs — no build step. It is a **visual companion** the learner keeps open while being taught: it maps the material and shows live mastery progress.

**Regenerate-on-tick.** The file is static — there is no live state. Whenever a checklist item is mastered, rewrite the whole file so the progress reflects reality. The only scripts allowed are the Tailwind CDN and the Mermaid ESM import. No app code, no interactivity beyond Mermaid's own rendering.

Mermaid handles graph-shaped diagrams reliably (decision branches, blast radius, call flow); hand-built divs and inline SVG handle the more editorial visuals (before/after of the change). Mix the two — don't lean on Mermaid for everything, it starts to look generic.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Learning map — {{topic}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .branch { stroke-dasharray: 4 4; }
      .chosen { stroke: #059669; stroke-width: 2px; }
      .done   { text-decoration: line-through; opacity: 0.55; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="progress">...</section>       <!-- mastery checklist -->
      <section id="problem">...</section>         <!-- pillar 1 -->
      <section id="solution">...</section>        <!-- pillar 2 -->
      <section id="context">...</section>         <!-- pillar 3 -->
    </main>
  </body>
</html>
```

## Header

Topic, date, and a compact progress meter — `N / M mastered` plus a thin bar. No introduction paragraph; the learner already knows what they're learning.

## Progress section (the mastery checklist)

The checklist rendered visually, grouped by the three pillars. Each item is a row:

- A state marker — empty circle (unmastered) or filled emerald check (mastered, apply `.done`).
- The concept, one short line.
- Optional: a tiny quiz tag (`quizzed ✓`) once verified.

This section is the visual twin of the `.md` checklist — they must always agree. Re-emit on every tick.

## The three pillars

Each pillar is one `<section>`. Diagrams carry the weight; prose is sparse and plain.

### 1. Problem — what it was, why it existed, the branches

The centrepiece is a **Mermaid decision graph**: the problem at the root, the branches/alternatives that were considered, and which one was chosen (style the chosen edge with `classDef chosen stroke:#059669`). This makes the "why this and not that" visible.

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart TD
      P[Problem: stale cache on write] --> A[Option A: TTL only]
      P --> B[Option B: write-through invalidation]
      P --> C[Option C: event bus]
      classDef chosen stroke:#059669,stroke-width:3px;
      class B chosen
  </pre>
</div>
```

### 2. Solution — before/after of the change

A **hand-built before/after**, two columns side by side. Left: the old shape. Right: the new shape. Modules as bordered `<div>`s, arrows as inline SVG over a relative container. Reach for hand-built here (not Mermaid) so the "after" can carry visual weight — a thick-bordered new path, the removed pieces greyed out. Annotate the edge cases right on the diagram where they bite.

### 3. Context — why it matters, the blast radius

A **Mermaid graph** of what the change touches: the changed module at the centre, edges out to every caller / downstream system / test affected. This is the "what will this impact" pillar made concrete. Colour anything that needs follow-up attention amber.

## Diagram patterns

Pick what fits; mix them; don't make every diagram look the same.

- **Decision graph** (Mermaid `flowchart`) — problem + branches considered, chosen path highlighted. The workhorse for pillar 1.
- **Before/after boxes** (hand-built divs + SVG) — the change itself. Pillar 2.
- **Blast-radius graph** (Mermaid `graph`) — changed module → everything it touches. Pillar 3.
- **Sequence diagram** (Mermaid) — when the lesson is "before: 6 round-trips; after: 1," or any ordering/timing concept.
- **Cross-section** (stacked `h-12 border-l-4` bands) — for layered flow a call passes through.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. `font-serif` headings pair well with stone/slate.
- Colour sparingly: one accent (emerald or indigo), red for anything broken/edge-case, amber for follow-ups.
- Keep diagrams ~320px tall so before/after sits side by side without scrolling.
- Module labels inside diagrams: `text-xs uppercase tracking-wider` — schematic, not UI.
- If a diagram needs a paragraph to be understood, redraw the diagram.

## Tone

Plain English, concise. The HTML is a teaching aid, not a report to impress — every diagram should answer a *why*, *what*, or *how* the learner needs. No throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it.
