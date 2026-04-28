# Claude Code Workflow

A set of files for running structured, multi-session Claude Code projects using **Technical Project Plans (TPPs)**.

## What's in this repo

| File | Purpose |
|------|---------|
| `smartclaude` | Shell wrapper that starts Claude with a TPP-aware system prompt |
| `TPP-GUIDE.md` | Guide to the TPP format, naming conventions, and workflow |
| `handoff-skill.md` | Claude Code skill (`/handoff`) — updates a TPP before ending a session |
| `pickup-skill.md` | Claude Code skill (`/pickup`) — resumes work from a TPP in a new session |
| `SIMPLE-DESIGN.md` | Design principles used to evaluate code during implementation |

## The core idea

Claude Code sessions are ephemeral — context is lost when a session ends. **TPPs** are markdown files in `docs/` that carry research, design decisions, and task progress forward across sessions. `smartclaude` injects TPP awareness into every session's system prompt so Claude knows to use them.

## Setup

1. Copy `smartclaude` somewhere on your `$PATH` and make it executable:
   ```bash
   cp smartclaude ~/bin/smartclaude
   chmod +x ~/bin/smartclaude
   ```

2. Install the skills. Choose **project-level** (available only in this project) or **global** (available in every project):

   **Project-level:**
   ```bash
   mkdir -p .claude/commands
   cp handoff-skill.md .claude/commands/handoff.md
   cp pickup-skill.md .claude/commands/pickup.md
   ```

   **Global:**
   ```bash
   mkdir -p ~/.claude/skills/handoff ~/.claude/skills/pickup
   cp handoff-skill.md ~/.claude/skills/handoff/SKILL.md
   cp pickup-skill.md ~/.claude/skills/pickup/SKILL.md
   ```

3. Copy `TPP-GUIDE.md` and `SIMPLE-DESIGN.md` into your project's `docs/` directory:
   ```bash
   mkdir -p docs
   cp TPP-GUIDE.md docs/TPP-GUIDE.md
   cp SIMPLE-DESIGN.md docs/SIMPLE-DESIGN.md
   ```

## Workflow

```
# Start a session
smartclaude

# In Claude: enter plan mode, pick up an existing TPP
/pickup docs/20260427-my-feature.md

# Work happens...

# Before ending the session, hand off
/handoff
```

On the next session, `/pickup` re-reads the TPP and continues where work left off.

## TPP lifecycle

1. Create a TPP in `docs/` named `YYYYMMDD-feature-name.md`
2. `/pickup` reads it, validates assumptions, and works through the phases
3. `/handoff` updates progress, documents discoveries, and flags blockers
4. When complete, move the TPP to `docs/_done/`

See `TPP-GUIDE.md` for the full template and phase checklist.
