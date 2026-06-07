# Raki Skills

A skills toolkit for agentic software development, in two families:

- **Review** — steel-man skills that destroy specs and plans to find their weaknesses, steelman them to find their strengths, then deliver structured go/no-go verdicts.
- **Learn** — `teach-me` teaches you a subject until you've *demonstrably* mastered it (quiz-gated loop); `spec-walkthrough` has a senior engineer walk you through a specific design until you can *own* it (full walkthrough, then comprehension checks). Both read the code they reference and produce a visual map.

Built for coding agents (Kimi CLI, Claude Code, Codex, Cursor) that use skills-based workflows.

## The Skills

### Review

| Skill | Use When | Output |
|-------|----------|--------|
| [`reviewing-specs`](skills/reviewing-specs/SKILL.md) | A design spec is complete and needs validation before implementation planning | Structured spec review with go/conditional-go/no-go verdict |
| [`reviewing-plans`](skills/reviewing-plans/SKILL.md) | An implementation plan is written and needs a gut-check before execution | Structured plan review with go/go-with-fixes/no-go verdict |
| [`reviewing-code`](skills/reviewing-code/SKILL.md) | A diff/branch/PR is ready for review before merge | Ranked findings (correctness first, then cleanup) with go/go-with-fixes/no-go verdict; optionally applies cleanup fixes |

### Learn

| Skill | Use When | Output |
|-------|----------|--------|
| [`teach-me`](skills/teach-me/SKILL.md) | You want to deeply understand a *completed* change, session, or unfamiliar code | A mastery checklist + HTML learning map; the session won't end until you've passed every quiz |
| [`spec-walkthrough`](skills/spec-walkthrough/SKILL.md) | You want a senior engineer to walk you through a spec/design/plan before you build, review, or own it | Full walkthrough (problem → rationale → end-to-end → codebase fit → risks/ops), *then* comprehension checks; HTML walkthrough map; design gaps handed to grilling/brainstorming |

## Philosophy

### Review: be ruthless, then fair

Most review frameworks are too polite. They hedge, qualify, and soften feedback until it becomes useless. Raki does the opposite:

1. **ATTACK** — Find every flaw. Assume the document is wrong until proven otherwise.
2. **STEELMAN** — Build the strongest possible case FOR it. Genuine, not sarcastic.
3. **VERDICT** — Honest go/no-go with actionable fixes.

All three review skills run this as a **multi-angle fan-out**: specialist finder subagents each get their own context window and attack one dimension, then an adversarial **verifier** steel-mans each candidate (CONFIRMED / PLAUSIBLE / REFUTED) to kill false positives before the verdict. LLM reviewers have a known leniency bias when a single agent both writes and judges; independent scoped finders plus a refute-capable verifier are the structural fix. An **effort dial** (`low` / `medium` / `high`) scales how many angles run — a two-line change doesn't need seven reviewers. Each skill reads a `.memory.md` of project-specific lessons on load and appends to it on close, so it stops re-flagging the same intentional patterns.

### Learn: mastery is demonstrated, not asserted

Most "explain this" prompts dump everything at once and accept "got it" as proof. Raki's learning skills don't:

1. **DIAGNOSE** — Have the learner restate first; teach into the *actual* gaps.
2. **TEACH** — One stage at a time, concrete (real code, the debugger), at the depth they ask for (ELI5/ELI14/ELI-intern).
3. **VERIFY** — A quiz per stage; the session is gated and won't end with unchecked items.

`teach-me` is the inverse of `grill-me`: where `grill-me` interviews you to extract a plan, `teach-me` teaches you until verified understanding. `spec-walkthrough` is a different beast — a senior engineer walking you through a *specific* design end to end (problem → rationale → architecture → codebase fit → risks/ops), and only *after* the full walkthrough checking that you can discuss, review, maintain, and extend it.

## How It Works

### Spec Review Pipeline

```
brainstorming (produces spec)
    -> reviewing-specs (destroys/validates spec)
        -> GO -> writing-plans (creates implementation plan)
        -> NO-GO -> back to brainstorming for revision
```

### Plan Review Pipeline

```
writing-plans (produces plan)
    -> reviewing-plans (destroys/validates plan)
        -> GO -> executing-plans / subagent-driven-development
        -> NO-GO -> back to writing-plans for revision
```

### Code Review Pipeline

```
implementation / executing-plans (produces a diff)
    -> reviewing-code (find -> verify -> sweep; correctness before cleanup)
        -> GO -> merge / open PR
        -> GO-WITH-FIXES -> apply ranked fixes (cleanup optionally auto-applied)
        -> NO-GO -> back to implementation
```

### Design Walkthrough Pipeline

```
spec / design doc / architecture proposal / plan
    -> spec-walkthrough (senior→junior: full walkthrough, THEN verify)
        -> owns it -> build / review / maintain with full understanding
        -> open design gaps -> grill-me / brainstorming for resolution
```

### Change Learning Pipeline

```
work completed (session / PR / refactor / executing-plans run)
    -> teach-me (teaches until mastered; quiz-gated)
        -> mastered -> durable record in docs/raki/learning/
```

## Installation

### Manual Install

Copy the skill folders into your agent's skills directory:

```bash
# For Kimi CLI
mkdir -p ~/.kimi/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans skills/reviewing-code skills/teach-me skills/spec-walkthrough ~/.kimi/skills/raki/

# For Claude Code
mkdir -p ~/.claude/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans skills/reviewing-code skills/teach-me skills/spec-walkthrough ~/.claude/skills/raki/

# For Codex
mkdir -p ~/.codex/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans skills/reviewing-code skills/teach-me skills/spec-walkthrough ~/.codex/skills/raki/
```

### Custom Agents (for the review skills)

All three review skills fan out to bundled subagents: `reviewing-specs` (7), `reviewing-plans` (6), and `reviewing-code` (4), each under the skill's `agents/` directory. They travel with the skill, but most hosts only auto-discover agents from a dedicated agents directory — copy them there so `subagent_type` resolves by name:

```bash
# Point AGENTS_DIR at your host's agents directory, then copy every skill's agents:
#   Claude Code → ~/.claude/agents   (or .claude/agents in a repo)
#   Kimi CLI    → ~/.kimi/agents
#   Codex       → ~/.codex/agents
AGENTS_DIR=~/.claude/agents   # change per host
mkdir -p "$AGENTS_DIR"
cp skills/reviewing-specs/agents/*.md "$AGENTS_DIR/"
cp skills/reviewing-plans/agents/*.md "$AGENTS_DIR/"
cp skills/reviewing-code/agents/*.md  "$AGENTS_DIR/"
```

If a host can't register named agents, the skills fall back to dispatching generic subagents using each agent file's body as the prompt — no registration required. The finder/verifier agents declare `tools: Read, Grep, Glob`, so they are read-only by construction.

### Cross-Session Use

These skills are designed for cross-session use:

- **Review skills** save their output to `docs/raki/reviews/`. Run them in a fresh session on a saved artifact and feed findings back into the originating session.
- **Learn skills** save a checklist (`.md`) and a visual map (`.html`) to `docs/raki/learning/` — a mastery checklist + learning map for `teach-me`, a comprehension checklist + walkthrough map for `spec-walkthrough`. The record is durable and resumable — pick a session back up where you left off.

## Directory Structure

Every skill is a lean orchestrator `SKILL.md` plus, where it fans out, an `agents/` directory of scoped read-only subagents. Each skill also carries a `.memory.md` for project-specific lessons that persist across sessions.

```
skills/
  reviewing-specs/
    SKILL.md                    # Skill definition (lean orchestrator + effort dial)
    .memory.md                  # Accumulated lessons (read on load, appended on close)
    agents/                     # 7 custom subagents the skill dispatches
      product-requirements-reviewer.md  # finder: stories, acceptance, metrics
      architecture-reviewer.md          # finder: components, boundaries, contracts
      edge-case-reviewer.md             # finder: boundaries, failures, concurrency
      scope-yagni-reviewer.md           # finder: scope creep, gold-plating
      security-privacy-reviewer.md      # finder: authn/authz, PII, secrets
      testability-reviewer.md           # finder: test strategy, observability
      spec-finding-verifier.md          # verifier: CONFIRMED/PLAUSIBLE/REFUTED
  reviewing-plans/
    SKILL.md                    # Skill definition (lean orchestrator + effort dial)
    .memory.md
    agents/                     # 6 custom subagents the skill dispatches
      dependency-order-reviewer.md      # finder: circular deps, unsafe ordering
      task-sizing-reviewer.md           # finder: oversized/vague tasks
      test-plan-reviewer.md             # finder: missing tests/verification
      rollback-migration-reviewer.md    # finder: rollback + migration safety
      integration-risk-reviewer.md      # finder: cross-system blast radius
      plan-verifier.md                  # verifier: CONFIRMED/PLAUSIBLE/REFUTED
  reviewing-code/
    SKILL.md                    # Skill definition
    .memory.md
    agents/                     # 4 custom subagents the skill dispatches
      correctness-reviewer.md   #   finder: line-by-line, removed-behavior, pitfalls
      integration-reviewer.md   #   finder: call sites, wrapper/proxy routing
      cleanup-reviewer.md       #   finder: reuse/simplify/efficiency/altitude (/simplify)
      finding-verifier.md       #   verifier: CONFIRMED/PLAUSIBLE/REFUTED
  teach-me/
    SKILL.md                    # Skill definition (+ effort dial)
    .memory.md                  # Per-learner lessons (what landed, what's mastered)
    HTML-REPORT.md              # HTML learning-map scaffold + diagram patterns
  spec-walkthrough/
    SKILL.md                    # Skill definition (+ effort dial)
    .memory.md                  # Per-developer / per-system lessons
    HTML-REPORT.md              # HTML learning-map scaffold + diagram patterns
```

## Integration with Superpowers

Raki skills slot into the Superpowers workflow at several points:

1. **After brainstorming** → `reviewing-specs` validates the spec before `writing-plans` begins.
2. **Before building/reviewing** → `spec-walkthrough` walks whoever will own the design through it (full walkthrough, *then* comprehension checks), surfacing gaps for `grill-me`/`brainstorming`.
3. **After writing-plans** → `reviewing-plans` validates the plan before `executing-plans` begins.
4. **Before merge** → `reviewing-code` validates the diff (correctness first, then cleanup) before the PR opens.
5. **After the work lands** → `teach-me` transfers understanding of the completed change.

Review skills write to `docs/raki/reviews/`; learning skills write to `docs/raki/learning/`. Both can feed back into the originating session.

## License

MIT License - see [LICENSE](LICENSE) file for details.
