# Raki Skills

A skills toolkit for agentic software development, in two families:

- **Review** — steel-man skills that destroy specs and plans to find their weaknesses, steelman them to find their strengths, then deliver structured go/no-go verdicts.
- **Learn** — teaching skills that read a change or a spec (and the code it touches) and teach you until you've *demonstrably* mastered it — with a quiz-gated loop and a visual learning map.

Built for coding agents (Kimi CLI, Claude Code, Codex, Cursor) that use skills-based workflows.

## The Skills

### Review

| Skill | Use When | Output |
|-------|----------|--------|
| [`reviewing-specs`](skills/reviewing-specs/SKILL.md) | A design spec is complete and needs validation before implementation planning | Structured spec review with go/conditional-go/no-go verdict |
| [`reviewing-plans`](skills/reviewing-plans/SKILL.md) | An implementation plan is written and needs a gut-check before execution | Structured plan review with go/go-with-fixes/no-go verdict |

### Learn

| Skill | Use When | Output |
|-------|----------|--------|
| [`teach-me`](skills/teach-me/SKILL.md) | You want to deeply understand a *completed* change, session, or unfamiliar code | A mastery checklist + HTML learning map; the session won't end until you've passed every quiz |
| [`spec-walkthrough`](skills/spec-walkthrough/SKILL.md) | You want to understand a spec *before* building it, mapped onto the code it touches | A mastery checklist + HTML learning map; surfaced gaps handed off to grilling/brainstorming |

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

`teach-me` is the inverse of `grill-me`: where `grill-me` interviews you to extract a plan, `teach-me` teaches you until verified understanding. `spec-walkthrough` is its spec-flavored sibling.

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

### Spec Learning Pipeline

```
spec exists (PRD / ticket / design doc)
    -> spec-walkthrough (reads spec + the code it touches; teaches until mastered)
        -> mastered -> writing-plans / implementation
        -> open gaps -> grill-me / brainstorming for resolution
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
cp -r skills/reviewing-specs skills/reviewing-plans skills/teach-me skills/spec-walkthrough ~/.kimi/skills/raki/

# For Claude Code
mkdir -p ~/.claude/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans skills/teach-me skills/spec-walkthrough ~/.claude/skills/raki/

# For Codex
mkdir -p ~/.codex/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans skills/teach-me skills/spec-walkthrough ~/.codex/skills/raki/
```

### Cross-Session Use

These skills are designed for cross-session use:

- **Review skills** save their output to `docs/raki/reviews/`. Run them in a fresh session on a saved artifact and feed findings back into the originating session.
- **Learn skills** save a mastery checklist (`.md`) and a visual learning map (`.html`) to `docs/raki/learning/`. The record is durable and resumable — pick a session back up where you left off.

## Directory Structure

```
skills/
  reviewing-specs/
    SKILL.md                    # Skill definition
    spec-reviewer-prompt.md     # Subagent prompt template
  reviewing-plans/
    SKILL.md                    # Skill definition
    plan-reviewer-prompt.md     # Subagent prompt template
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
2. **Before building** → `spec-walkthrough` makes sure whoever implements the spec actually understands it (and surfaces gaps for `grill-me`/`brainstorming`).
3. **After writing-plans** → `reviewing-plans` validates the plan before `executing-plans` begins.
4. **After the work lands** → `teach-me` transfers understanding of the completed change.

Review skills write to `docs/raki/reviews/`; learning skills write to `docs/raki/learning/`. Both can feed back into the originating session.

## License

MIT License - see [LICENSE](LICENSE) file for details.
