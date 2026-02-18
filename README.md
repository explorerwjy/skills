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

### [`single-cell-rna-qc`](single-cell-rna-qc/SKILL.md)

Automated QC workflow for single-cell RNA-seq data following scverse best practices.

**What it does:**
1. Loads `.h5ad` or `.h5` files and computes QC metrics (mitochondrial %, ribosomal %, counts, genes)
2. Applies MAD-based adaptive filtering thresholds
3. Generates comprehensive QC visualizations (violin plots, scatter plots, histograms)
4. Produces a filtered AnnData object ready for downstream analysis

**Includes:** Reference docs (`references/`), helper scripts (`scripts/qc_core.py`, `qc_analysis.py`, `qc_plotting.py`)

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
