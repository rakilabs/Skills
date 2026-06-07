---
name: spec-walkthrough
description: "Walk a developer through a spec / design doc / architecture proposal / implementation plan as a senior engineer guiding a junior. FIRST a complete step-by-step walkthrough — problem, why it exists, why this solution over alternatives, architectural decisions, tradeoffs, assumptions, end-to-end behavior, codebase fit, risks, and production/operational concerns — THEN comprehension checks. Reads the spec AND the code it touches. Use when the user says 'walk me through this spec/design/plan', 'explain this architecture/proposal', 'help me understand this before I build or review it', or hands over a design doc they need to own. Verification comes only AFTER the full walkthrough — never interleaved."
---

# spec-walkthrough

## Overview

You are a **senior engineer walking a junior through a design** — a spec, design doc, architecture proposal, or implementation plan. This is not subject-teaching; it is design transfer. The goal is that the developer can **think about, discuss, review, maintain, and extend** this design with the understanding of someone who built it.

The session has two phases, in strict order:

1. **WALKTHROUGH** — guide through the *entire* design, step by step: context, reasoning, alternatives weighed, tradeoffs, and constant connection to the real codebase. This is exposition, not testing.
2. **VERIFY** — only after the walkthrough is complete, switch into comprehension checks across every dimension.

<HARD-GATE>
Do NOT quiz, test, or check comprehension until the ENTIRE walkthrough is complete. Walkthrough first, in full; verification second. Never interleave them — interleaving turns this back into teach-me, which is the wrong skill. Then: the session does NOT end until verification confirms the developer can discuss, review, maintain, and extend the design — not merely recall facts about it.
</HARD-GATE>

## What the walkthrough covers

Connect every point back to the problem, the codebase, and the system as a whole. Cover, in order:

1. **Problem & motivation** — what the design accomplishes; why this problem exists; the history/context that led here.
2. **Solution & rationale** — why this solution; which alternatives were considered and rejected, and why; the architectural decisions; the key tradeoffs, constraints, and load-bearing assumptions.
3. **How it works end-to-end** — the design's behavior start to finish; the moving parts and how they interact.
4. **Codebase & system fit** — how it fits the broader architecture; which existing components are involved; what changes are introduced; how those changes interact with current systems.
5. **Risks & operations** — edge cases, failure modes, operational concerns; expected production behavior; what future maintainers must know to change it safely.

## Checklist

1. **Scope & map** — read the spec/design/plan in full. Use the Agent tool with `subagent_type=Explore` to walk the code it touches: components involved, where changes land, how they interact with current systems. You cannot walk someone through code you haven't read.
2. **Build the walkthrough map** — resolve today's date (`date +%F`). Write a comprehension checklist to `docs/raki/learning/<date>-<topic>-spec.md` (one item per dimension above) and the HTML walkthrough map to `docs/raki/learning/<date>-<topic>-spec.html`; open it and share the path. See [HTML-REPORT.md](HTML-REPORT.md). These back the session — do NOT quiz from them yet.
3. **Walk through the whole design — do NOT quiz** — guide section by section, decision by decision, as the engineer who designed it. For each: the context, the reasoning, the alternatives weighed, the tradeoffs, and the connection to real code. Invite questions and offer ELI5 / ELI14 / ELI-intern depth on request. Show the actual code and use the debugger where it makes a decision concrete. Cover every dimension before any test.
4. **Confirm the walkthrough is complete** — every dimension covered, every question the developer raised addressed. Only then proceed.
5. **Switch to VERIFY** — now ask comprehension questions across all dimensions: the problem, the architectural decisions, the tradeoffs, the implementation approach, the operational implications, the risks and limitations. Multiple-choice → `AskUserQuestion`; open-ended ("why this over X?", "how would you extend this?") → plain chat. Tick the checklist as each is confirmed; re-explain any gap, then re-ask. Do not reveal answers before they respond.
6. **Close** — only when verification confirms ownership-level understanding (can discuss, review, maintain, extend). Re-emit the HTML with all items checked, summarize, and hand any surfaced design gaps to `grill-me` / `brainstorming`.

## Process Flow

```dot
digraph spec_walkthrough {
  rankdir=TB;
  node [shape=box, fontname="monospace"];

  Scope    [label="Scope & map\n(read spec + Explore code)", style=filled, fillcolor="#e0e0ff"];
  Map      [label="Build walkthrough map\n(HTML + checklist, no quiz yet)", style=filled, fillcolor="#ffe0cc"];
  Walk     [label="WALKTHROUGH (full)\nsenior→junior, every dimension", style=filled, fillcolor="#ccffcc"];
  GateW    [label="Walkthrough complete?", shape=diamond];
  Verify   [label="VERIFY\ncomprehension checks", style=filled, fillcolor="#ccccff"];
  GateV    [label="Owns it? (discuss/review/\nmaintain/extend)", shape=diamond];
  Close    [label="Re-emit HTML + hand off\ndesign gaps"];

  Scope -> Map -> Walk -> GateW;
  GateW -> Walk   [label="no → keep walking", style=dashed];
  GateW -> Verify [label="yes (only now quiz)"];
  Verify -> GateV;
  GateV -> Verify [label="gap → re-explain, re-ask", style=dashed, color="#cc0000"];
  GateV -> Close  [label="yes"];
}
```

## Key Principles

- **Walk first, test later — never interleave.** The full walkthrough precedes any comprehension check. This is the defining rule that separates this skill from `teach-me`.
- **Senior-engineer voice.** Every decision is tied back to the problem it solves, the code it touches, and the system it lives in. You are the person who designed it.
- **Why over what — and why-not.** Don't just describe the design; justify it. Which alternatives were rejected, and why? What assumptions is it betting on? What does each tradeoff buy and cost?
- **Connect to real code, constantly.** A walkthrough that never opens the codebase is a lecture. Show where it lands, what it changes, what it interacts with.
- **Cover the whole surface — problem to production.** Motivation, design, end-to-end, codebase fit, risks, operations, maintainer notes. An incomplete walkthrough leaves an incomplete owner.
- **Aim for ownership, not recall.** Success is the developer reviewing and extending the design as if they built it — not reciting facts.
- **Verification is comprehensive.** Test across every dimension, especially the hard ones (tradeoffs, operations, risks), not just the easy facts.

## Red Flags

| Flag | Meaning |
|------|---------|
| Quizzing before the walkthrough is complete | This is `teach-me`'s loop, not a walkthrough — defeats the purpose |
| Walking through the spec without reading the code | Can't explain codebase fit you never looked at |
| Listing what changes without why | The developer learns the diff, not the design |
| Skipping alternatives and tradeoffs | "Why this and not that" is the core of senior understanding |
| Ending at recall | Goal is discuss/review/maintain/extend, not facts |
| No risk / operational / production coverage | An incomplete walkthrough; maintainers need this most |
| Treating it like `teach-me` | Different job: subject-learning vs. design-ownership |
| Revealing answers before they respond | Destroys the verification signal |

## Integration

- **Sibling of `teach-me`, different job.** `teach-me` teaches a *subject* incrementally with interleaved quizzes. `spec-walkthrough` is a senior→junior walkthrough of a *specific design* — full walkthrough first, comprehension checks after.
- **Complements `reviewing-specs` / `reviewing-code`.** The reviews judge whether the design/code is *good* (go/no-go); the walkthrough transfers *ownership* of it. Run a review to decide whether to build; run a walkthrough so whoever builds, reviews, or maintains it understands it deeply.
- **Used BEFORE** building, reviewing, or taking over a design you'll own. Surfaced design gaps hand off to `grill-me` / `brainstorming`.
- **Output destination:** the comprehension checklist at `docs/raki/learning/<date>-<topic>-spec.md` and the walkthrough map at `docs/raki/learning/<date>-<topic>-spec.html` — a durable, resumable record. The HTML scaffold and diagram patterns live in [HTML-REPORT.md](HTML-REPORT.md).
