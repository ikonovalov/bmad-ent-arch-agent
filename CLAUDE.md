# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A BMad-platform skill package implementing **Archon** — an enterprise bank architect AI agent. The repo contains one skill (`skills/agent-archon/`) plus BMad infrastructure (`_bmad/`). There is no build step, no test runner invocation, and no server to start — the primary development artifact is prompt/config text.

## Activating the Agent

```
/agent-archon
```

Headless modes (no interaction, writes report to `_bmad-output/archon/`):

```
/agent-archon --headless:landscape-drift-scan
/agent-archon --headless:compliance-audit
/agent-archon --headless:arb-prep docs/decisions/adr-042.md
```

## Running the Customization Script

The only executable code is the TOML merge utility. Requires Python 3.11+.

```bash
# Resolve full merged config for agent-archon
python3 _bmad/scripts/resolve_customization.py --skill skills/agent-archon

# Resolve a single key (e.g., the agent block)
python3 _bmad/scripts/resolve_customization.py --skill skills/agent-archon --key agent

# Run tests
python3 -m pytest _bmad/scripts/tests/
```

## Architecture

### Three-Layer Customization System

Every configurable value merges from three TOML layers (lowest → highest priority):

1. `skills/agent-archon/customize.toml` — skill defaults (do **not** edit; overwritten on updates)
2. `_bmad/custom/agent-archon.toml` — team overrides (committed)
3. `_bmad/custom/agent-archon.user.toml` — personal overrides (gitignored)

Merge rules: scalars override, tables deep-merge, arrays of tables keyed by `code`/`id` do replace+append, all other arrays append. See `_bmad/scripts/resolve_customization.py` for the implementation.

Similarly, `_bmad/config.toml` (installer-managed, read-only) and `_bmad/custom/config.toml` / `_bmad/custom/config.user.toml` hold project-level settings (language, output folder, agent descriptors).

### Skill Structure (`skills/agent-archon/`)

- `SKILL.md` — agent identity, activation sequence, capability routing table. This is the agent's "brain".
- `customize.toml` — the customization surface (paths to bank knowledge files, hooks, output dir).
- `references/*.md` — capability prompts loaded on-demand: `architectural-consultation.md`, `adversarial-review.md`, `compliance-check.md`, `reuse-discovery.md`, `artifact-generation.md`, `landscape-drift.md`.
- `references/templates/` — output templates for ADR, Solution Design, ARB Package.

### Agent Activation Sequence (defined in `SKILL.md`)

1. Run `resolve_customization.py` to merge the `[agent]` block.
2. Execute `activation_steps_prepend`.
3. Load `persistent_facts` (literal strings or `file:` globs).
4. Load `_bmad/config.toml` and `_bmad/config.user.toml` for language/user settings.
5. Parallel-load the three bank knowledge files (target arch, principles, landscape registry). Record `loaded | missing` status.
6. Execute `activation_steps_append`.
7. Route: headless → execute + write report + exit; interactive → greet + offer capability menu.

### Bank Knowledge Files (`docs/` — not committed, user-supplied)

Archon reads four knowledge sources configured in `customize.toml`:

| Key | Default path |
|---|---|
| `landscape_registry_path` | `docs/landscape/registry.yaml` |
| `target_architecture_path` | `docs/architecture/target.md` |
| `architecture_principles_path` | `docs/architecture/principles.md` |
| `compliance_corpus_path` | `docs/compliance/` |
| `strategy_path` | `docs/strategy.md` |

Missing files produce warnings but do not abort the session. The agent discloses which files are missing before answering.

### Output

All reports land in `_bmad-output/archon/` (configurable via `assessment_output_path`). Headless reports include a YAML front-matter block for CI parsing. A blocker-severity finding causes a non-zero exit code (CI gate).

## Customization Workflow

To change agent behavior without touching skill defaults:

1. Edit `_bmad/custom/agent-archon.toml` (team) or `_bmad/custom/agent-archon.user.toml` (personal).
2. Verify the merge: `python3 _bmad/scripts/resolve_customization.py --skill skills/agent-archon --key agent`.

Use `bmad-customize` skill for guided customization (no TOML hand-authoring required).
