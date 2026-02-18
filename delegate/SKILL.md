---
name: delegate
description: Interactively gather codebase context and create a self-contained Linear issue for Cyrus to work on independently. Use when you want to hand off a task to Cyrus with all the context it needs.
---

# Delegate to Cyrus

Create rich, self-contained Linear issues that Cyrus can act on independently — no vague descriptions, no missing context.

## Phase 1: Intent Capture

1. If the user passed arguments (e.g., `/delegate Fix spike detection threshold`), use that as the intent. Otherwise ask: **"What do you want Cyrus to do?"**
2. Ask **at most one** clarifying question if the intent is ambiguous (e.g., "Is this a bug fix or a new feature?"). Do not over-interrogate.
3. Determine the task type: `code`, `bug-fix`, `refactor`, `documentation`, or `research`.

## Phase 2: Context Gathering

### 2a. Repo Detection

1. Read `~/.cyrus/config.json` to get all configured repositories.
2. Get the current working directory (`pwd`).
3. Match cwd against each repository's `repositoryPath` — check if cwd is equal to or is a subdirectory of any `repositoryPath`.
4. If a match is found, that is the target repo. Extract:
   - `repositoryPath` (the absolute root — **all paths are resolved against this**)
   - `name` (repo name)
   - `projectKeys` (Linear project for routing)
5. If **no match**, present the list of repos to the user via `AskUserQuestion` and let them pick.

### 2b. Read Project Context

1. Read `<repositoryPath>/CLAUDE.md` if it exists — this contains project-specific guidance Cyrus will also see.
2. Read `<repositoryPath>/DATA_MANIFEST.yaml` if it exists — this documents data files.
3. Read `<repositoryPath>/config/config.yaml` or similar config files if they exist.

### 2c. Search for Relevant Files

Based on the user's intent, search the repo intelligently:

1. **Grep** for keywords from the intent across `Snakefile*`, `*.py`, `*.yaml`, `*.md` files.
2. **Glob** for structural files: `Snakefile`, `scripts/*.py`, `notebooks/*.py`, `config/*.yaml`, `src/**/*.py`.
3. Read matched files and **extract only the relevant portions** — the specific function, Snakemake rule, or config section. Never dump entire files.
4. For **Snakemake projects**: trace the rule dependency chain.
   - Find the rule most relevant to the intent.
   - Read its `input:` and `output:` to identify upstream/downstream rules.
   - Follow one level up and one level down to map the local pipeline context.
5. For **data files**: identify input/output formats, key columns, and file paths from the code or DATA_MANIFEST.yaml.

### 2d. Path Resolution

**CRITICAL: Every file path in the issue must be absolute.**

- Use `repositoryPath` from config as the base for all paths.
- Convert any relative path found in code/config to absolute: `<repositoryPath>/<relative_path>`.
- For cross-repo references (e.g., code that reads from `../ASD_Circuits/dat/`), look up the other repo's `repositoryPath` in config and resolve against that.
- For wildcard paths in Snakemake (e.g., `dat/raw/{sample}.h5ad`), preserve the wildcard but make the base absolute: `/home/user/.cyrus/repos/RepoName/dat/raw/{sample}.h5ad`.

### 2e. Confirm with User

Present a summary of what was gathered:

```
I found these relevant files and context:
- [list of key files with one-line descriptions]
- [pipeline dependencies if applicable]
- [data files if applicable]

Anything else I should include?
```

If the user adds more context, gather it. Then proceed to assembly.

## Phase 3: Issue Assembly

Build the issue using the template below. **Include sections conditionally** — only add sections that are relevant to the task.

### Title

Write a concise, actionable title (under 80 chars). Examples:
- "Fix spike detection threshold in HH model simulator"
- "Add pairwise feature correlation rule to Snakemake pipeline"
- "Investigate cell-type bias distribution across psychiatric disorders"

### Description Template

```markdown
## Task

[Clear, actionable statement of what to do. 2-4 sentences max.]

## Classification Hint

Type: [code | bug-fix | refactor | documentation | research]

## Relevant Files

| File | Purpose |
|------|---------|
| `/abs/path/to/file.py` | What this file does in context of the task |

## Key Code Context

[Paste ONLY the relevant function/rule/section from each file, with file path and line numbers. Keep each snippet focused — typically 10-40 lines. Format as fenced code blocks with language hint.]

## Pipeline Context

[Include ONLY for Snakemake/pipeline projects where the task touches a pipeline step.]

- **Upstream:** `rule rule_name` -> produces `/abs/path/to/output`
- **This step:** `rule rule_name` -> reads X, produces Y
- **Downstream:** `rule rule_name` -> consumes Y

## Data Context

[Include ONLY when the task involves reading/writing/transforming data files.]

- **Input:** `/abs/path/to/input.parquet` — [format, key columns/fields, row count if known]
- **Output:** `/abs/path/to/output.pkl` — [expected format]
- **Config:** `/abs/path/to/config.yaml` — [list key parameters and their current values]

## Results Reporting

[Include ONLY for research/analysis tasks.]

When complete, generate a summary report including:
- Key findings in a concise narrative (2-3 paragraphs)
- Summary tables (e.g., comparison across conditions, statistical tests)
- Figures saved as PNG to `/abs/path/results/figures/` with transparent backgrounds
- Post the report as a Linear comment with embedded tables and figure references

## Acceptance Criteria

- [ ] [Specific, testable criterion]
- [ ] [Another criterion]
- [ ] Pipeline runs without error / Tests pass
```

### Assembly Rules

- **All paths absolute** — never `scripts/foo.py`, always `/home/user/.cyrus/repos/Repo/scripts/foo.py`
- **Code snippets are focused** — only the relevant function/rule, not the whole file
- **Include line numbers** — so Cyrus can navigate directly to the code
- **Add a routing tag** at the very end of the description: `[repo=<repo_name>]` — this ensures Cyrus routes to the correct repo even without projectKeys config

### Review

Show the user the complete title + description and ask: **"Does this look right? Any changes before I create the issue?"**

Apply any requested changes. Do not proceed until the user approves.

## Phase 4: Create & Route

1. Determine the Linear team. Read `~/.cyrus/config.json` — use the team associated with the workspace. If unsure, use `mcp linear list_teams` to find the correct team.
2. Determine the Linear project from the repo's `projectKeys[0]`.
3. Create the issue:

```
mcp linear create_issue:
  title: <the title>
  description: <the assembled description>
  team: <team name>
  project: <project name from projectKeys>
```

4. Report back: **"Issue created: [URL]. Cyrus will pick it up when assigned."**

## Hard Rules

1. **NEVER use relative paths.** All file paths in the issue description must be absolute. Cyrus runs in worktrees at `~/.cyrus/worktrees/ISSUE-ID/`, not in the original repo.
2. **NEVER create the issue without user review.** Always show the full description and get explicit approval.
3. **NEVER dump entire files.** Extract only the relevant portions with line numbers.
4. **NEVER skip the `[repo=<name>]` tag.** Always include it at the end of the description as a routing fallback.
5. **NEVER over-interrogate.** Phase 1 gets at most one clarifying question. The skill's value is in automatic context gathering, not in asking the user 10 questions.
6. **Resolve cross-repo paths.** If code references files in another repo (e.g., Neural-P reading ASD_Circuits data), look up that repo's `repositoryPath` in config and use the absolute path.
