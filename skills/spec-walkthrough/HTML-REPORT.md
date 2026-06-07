# HTML Walkthrough Map Format (spec-walkthrough)

A single self-contained HTML file written next to the comprehension checklist at `docs/raki/learning/<date>-<topic>-spec.html`. Tailwind and Mermaid via CDN — no build step. It is the **map a senior engineer sketches while walking a junior through the design**: it lays out the whole design across five dimensions and tracks comprehension.

**Two phases, mirrored in the file.** During the WALKTHROUGH the map is reference material you talk over — the checklist stays unchecked. Only in the VERIFY phase do checklist items get ticked. **Regenerate-on-tick:** the file is static; rewrite it when a comprehension item is confirmed. The only scripts are the Tailwind CDN and the Mermaid ESM import.

Mermaid handles graph-shaped diagrams (decision trees, spec→code maps, sequences); hand-built divs and inline SVG handle editorial visuals (alternatives tables, risk boards). Mix the two.

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
      .chosen   { stroke: #059669; stroke-width: 3px; }
      .rejected { stroke-dasharray: 4 4; opacity: 0.6; }
      .risk     { stroke: #d97706; stroke-width: 2px; }
      .done     { text-decoration: line-through; opacity: 0.55; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="comprehension">...</section>   <!-- checklist, ticked in VERIFY -->
      <section id="problem">...</section>          <!-- 1. problem & motivation -->
      <section id="rationale">...</section>        <!-- 2. solution & rationale -->
      <section id="end-to-end">...</section>       <!-- 3. how it works end-to-end -->
      <section id="codebase-fit">...</section>     <!-- 4. codebase & system fit -->
      <section id="risks-ops">...</section>        <!-- 5. risks & operations -->
    </main>
  </body>
</html>
```

## Header

Topic, date, the source (spec/design-doc/plan link or filename), and a compact comprehension meter — `N / M confirmed` plus a thin bar. No intro paragraph.

## Comprehension section

The checklist rendered visually, grouped by the five dimensions. Each row: a state marker (empty circle → emerald check when confirmed in VERIFY, apply `.done`), the dimension item, and an optional `verified ✓` tag. **Unchecked throughout the walkthrough; ticked only during verification.**

## The five dimensions

Each is one `<section>`. Diagrams carry the weight; prose is sparse and uses the design's own vocabulary (and `CONTEXT.md` terms if the project has them).

### 1. Problem & motivation

A short framing plus a **context graph** (Mermaid `flowchart`): the problem at the root, the forces/constraints that made it a problem, and what triggered the work now.

### 2. Solution & rationale

The centrepiece of the "why." An **alternatives view**: the chosen approach beside the rejected ones, each with the one-line reason it lost. Mermaid decision tree with the chosen edge `.chosen` and rejected branches `.rejected`, or a hand-built two-column "considered / chosen" table. Call out the load-bearing assumptions and the key tradeoffs (what each buys and costs).

```html
<pre class="mermaid">
  flowchart TD
    P[Goal] --> A[Option A: ...]:::rejected
    P --> B[Option B: chosen]:::chosen
    P --> C[Option C: ...]:::rejected
    classDef chosen stroke:#059669,stroke-width:3px;
    classDef rejected stroke-dasharray:4 4,opacity:0.6;
</pre>
```

### 3. How it works end-to-end

A **Mermaid sequence or flow diagram** of the design's behavior start to finish — the moving parts and how a request/operation flows through them. This is the "how," made concrete.

### 4. Codebase & system fit

A **Mermaid graph** mapping the design onto the real code: components/modules involved on one side, what the design adds/changes wired to them, edges showing interaction with current systems. Name real files (`font-mono`). Colour changed-existing-behavior red so the blast radius is obvious.

```html
<pre class="mermaid">
  flowchart LR
    N1[new: token service] --> M1["auth/tokens.ts (change)"]:::change
    N1 --> M2["auth/mailer.ts (exists)"]
    classDef change stroke:#dc2626,stroke-width:2px;
</pre>
```

### 5. Risks & operations

A **hand-built risk board**: cards in a grid, one per risk / edge case / failure mode / operational concern. Each card: what could go wrong, the expected production behavior, and what a future maintainer must know. Amber-tint (`.risk`) the ones that need a mitigation decision; tag any unresolved design gap `→ grill-me`.

## Diagram patterns

Pick what fits; mix them; don't make every diagram look the same.

- **Context graph** (Mermaid) — problem and the forces behind it. Dimension 1.
- **Alternatives tree / table** (Mermaid decision tree or hand-built columns) — chosen vs. rejected with reasons. Dimension 2, the "why" workhorse.
- **Sequence / flow** (Mermaid) — end-to-end behavior. Dimension 3.
- **Spec→code map** (Mermaid graph) — design wired to real components, changes highlighted. Dimension 4.
- **Risk board** (hand-built card grid) — risks, edge cases, ops concerns. Dimension 5.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. `font-serif` headings pair well with stone/slate.
- Colour sparingly: one accent (emerald or indigo), red for code that changes, amber for risks/open decisions, dashed/faded for rejected alternatives.
- Keep diagrams ~320px tall.
- Module/file labels `font-mono text-sm`; schematic labels inside diagrams `text-xs uppercase tracking-wider`.
- If a diagram needs a paragraph to be understood, redraw the diagram.

## Tone

Plain English, concise — the voice of the engineer who designed it. Use the design's own terms and the project's `CONTEXT.md` vocabulary. Every diagram should answer a *why*, *how*, *where-in-the-code*, or *what-could-go-wrong* the developer needs to own the design. No throat-clearing. If a sentence could be a bullet, make it a bullet.
