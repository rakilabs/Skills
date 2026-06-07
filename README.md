# Raki Skills

A steel-man review framework for agentic software development. Two skills that destroy specs and plans to find their weaknesses, steelman them to find their strengths, then deliver structured go/no-go verdicts.

Built for coding agents (Kimi CLI, Claude Code, Codex, Cursor) that use skills-based workflows.

## The Skills

| Skill | Use When | Output |
|-------|----------|--------|
| [`reviewing-specs`](skills/reviewing-specs/SKILL.md) | A design spec is complete and needs validation before implementation planning | Structured spec review with go/conditional-go/no-go verdict |
| [`reviewing-plans`](skills/reviewing-plans/SKILL.md) | An implementation plan is written and needs a gut-check before execution | Structured plan review with go/go-with-fixes/no-go verdict |

## Philosophy

Most review frameworks are too polite. They hedge, qualify, and soften feedback until it becomes useless. Raki does the opposite:

1. **ATTACK** - Find every flaw. Assume the document is wrong until proven otherwise.
2. **STEELMAN** - Build the strongest possible case FOR it. Genuine, not sarcastic.
3. **VERDICT** - Honest go/no-go with actionable fixes.

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

## Installation

### Manual Install

Copy the skill folders into your agent's skills directory:

```bash
# For Kimi CLI
mkdir -p ~/.kimi/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans ~/.kimi/skills/raki/

# For Claude Code
mkdir -p ~/.claude/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans ~/.claude/skills/raki/

# For Codex
mkdir -p ~/.codex/skills/raki
cp -r skills/reviewing-specs skills/reviewing-plans ~/.codex/skills/raki/
```

### Cross-Session Review

These skills are designed for cross-session use:

1. **Session A**: Brainstorm + write spec/plan
2. **Between sessions**: Review artifacts are saved to `docs/raki/reviews/`
3. **Session B**: Run review skill on saved artifact, feed findings back to Session A's context

## Directory Structure

```
skills/
  reviewing-specs/
    SKILL.md                    # Skill definition
    spec-reviewer-prompt.md     # Subagent prompt template
  reviewing-plans/
    SKILL.md                    # Skill definition
    plan-reviewer-prompt.md     # Subagent prompt template
```

## Integration with Superpowers

Raki skills slot into the Superpowers workflow at two points:

1. **After brainstorming** → `reviewing-specs` validates the spec before `writing-plans` begins
2. **After writing-plans** → `reviewing-plans` validates the plan before `executing-plans` begins

Both skills write their review output to `docs/raki/reviews/` and can feed findings back into the originating session.

## License

MIT License - see [LICENSE](LICENSE) file for details.
