# Claude Code Skills

Custom skills for Claude Code that extend its capabilities with specialized workflows.

## Skills

### [`delegate`](delegate/SKILL.md)

Interactively gather codebase context and create self-contained Linear issues for Cyrus to work on independently.

**What it does:**
1. Captures user intent (what task to delegate)
2. Automatically gathers codebase context — relevant files, Snakemake pipeline dependencies, data manifests, config
3. Assembles a rich Linear issue with absolute paths, focused code snippets, and acceptance criteria
4. Creates the issue in Linear after user review

**Usage:** `/delegate Fix spike detection threshold` or just `/delegate` and follow the prompts.

## Installation

These skills are installed by symlinking (or copying) into `~/.claude/skills/`:

```bash
# Symlink approach (recommended — stays in sync with repo)
ln -s /home/jw3514/Work/GitRepos/skills/delegate ~/.claude/skills/delegate

# Or copy
cp -r delegate/ ~/.claude/skills/delegate/
```

## Adding New Skills

1. Create a new directory under the repo root (e.g., `my-skill/`)
2. Add a `SKILL.md` file following the [Claude Code skill format](https://docs.anthropic.com/en/docs/claude-code/skills)
3. Symlink or copy into `~/.claude/skills/`
