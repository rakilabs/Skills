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

### Custom Agents (for `reviewing-code`)

`reviewing-code` dispatches 4 bundled subagents (`skills/reviewing-code/agents/`). They travel with the skill, but most hosts only auto-discover agents from a dedicated agents directory — copy them there so `subagent_type` resolves by name:

```bash
# Point AGENTS_DIR at your host's agents directory, then copy:
#   Claude Code → ~/.claude/agents   (or .claude/agents in a repo)
#   Kimi CLI    → ~/.kimi/agents
#   Codex       → ~/.codex/agents
AGENTS_DIR=~/.claude/agents   # change per host
mkdir -p "$AGENTS_DIR"
cp skills/reviewing-code/agents/*.md "$AGENTS_DIR/"
```

If a host can't register named agents, the skill falls back to dispatching generic subagents using each agent file's body as the prompt — no registration required.

### Cross-Session Use

These skills are designed for cross-session use:

- **Review skills** save their output to `docs/raki/reviews/`. Run them in a fresh session on a saved artifact and feed findings back into the originating session.
- **Learn skills** save a checklist (`.md`) and a visual map (`.html`) to `docs/raki/learning/` — a mastery checklist + learning map for `teach-me`, a comprehension checklist + walkthrough map for `spec-walkthrough`. The record is durable and resumable — pick a session back up where you left off.

## Directory Structure

```
skills/
  reviewing-specs/
    SKILL.md                    # Skill definition
    spec-reviewer-prompt.md     # Subagent prompt template
  reviewing-plans/
    SKILL.md                    # Skill definition
    plan-reviewer-prompt.md     # Subagent prompt template
  reviewing-code/
    SKILL.md                    # Skill definition
    agents/                     # 4 custom subagents the skill dispatches
      correctness-reviewer.md   #   finder: line-by-line, removed-behavior, pitfalls
      integration-reviewer.md   #   finder: call sites, wrapper/proxy routing
      cleanup-reviewer.md       #   finder: reuse/simplify/efficiency/altitude (/simplify)
      finding-verifier.md       #   verifier: CONFIRMED/PLAUSIBLE/REFUTED
  teach-me/
    SKILL.md                    # Skill definition
    HTML-REPORT.md              # HTML learning-map scaffold + diagram patterns
  spec-walkthrough/
    SKILL.md                    # Skill definition
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
