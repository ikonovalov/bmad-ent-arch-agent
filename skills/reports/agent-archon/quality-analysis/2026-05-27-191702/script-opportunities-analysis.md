# Script Opportunities Analysis — agent-archon

**ScriptHunter Report**
Skill path: `skills/agent-archon`
Date: 2026-05-27

---

## Existing Scripts Inventory

No `scripts/` directory exists under `skills/agent-archon`. The skill delegates its only explicit script call to the shared infrastructure:

- `{project-root}/_bmad/scripts/resolve_customization.py` — referenced in SKILL.md Step 1 as an external shared script (not owned by this skill). Its job: merge `customize.toml` base → team → user layers per structural rules.

**Net result:** this skill ships zero scripts of its own. All deterministic work is either delegated to that shared script or handled by the LLM inline.

---

## Assessment

agent-archon is a high-judgment, domain-expert skill: every capability involves interpreting initiative context, weighing trade-offs, and anchoring output to bank-specific knowledge files the LLM must read. The core work is genuinely LLM territory. However, two categories of deterministic work are currently paid for in LLM tokens on every activation: configuration resolution (already partially scripted externally) and knowledge-status reporting. Additionally, the headless modes (`--headless:compliance-audit`, `--headless:landscape-drift-scan`) contain a structurally scriptable pre-pass layer — extracting structured fields from registry/architecture files — that the LLM currently performs inline, taxing every headless run with redundant parsing before any reasoning begins.

The overall intelligence placement is sound for the interactive capabilities. The penalty is concentrated at activation time and in the headless paths.

---

## Key Findings

---

### Finding 1 — Manual TOML Merge Fallback in Activation (Step 1)

**Severity:** High (LLM Tax: Heavy, 500+ tokens)
**File:** `SKILL.md` lines 48–52
**Affected path:** On Activation → Step 1

**What the LLM is currently doing:**
When `resolve_customization.py` is unavailable, the LLM is instructed to manually resolve the `agent` config block by reading three TOML files, understanding merge-rule semantics (scalars → override, tables → deep-merge, arrays-with-key → replace+append, other arrays → append), and producing a merged config. This is a pure data transformation with deterministic rules and no semantic judgment involved whatsoever.

**What a script would do instead:**
A Python script reads the three TOML files in order and applies the merge rules mechanically. Given identical inputs it always produces identical output. This is already recognized as scriptable — the external `resolve_customization.py` exists for exactly this reason. The fallback path just shouldn't exist as an LLM task.

**Script opportunity:** Make `resolve_customization.py` always available within the skill (or provide a minimal local fallback script `scripts/resolve_config.py`) so the LLM never executes the merge logic. If the external script is unavailable, the local script runs instead; if both fail, the agent surfaces a hard error rather than spending 500+ tokens on TOML merging.

**Estimated token savings:** 500–800 tokens per activation where the fallback triggers.
**Pre-pass potential:** Yes — this is pure pre-processing; its output (a flat resolved config dict) is direct LLM input for Steps 2–7.
**Reuse across skills:** High — the shared `resolve_customization.py` already serves this purpose across BMAD skills. A local fallback would be skill-specific insurance.

---

### Finding 2 — Knowledge Status Assembly (Step 5 + Step 7)

**Severity:** Medium (LLM Tax: Moderate, 100–200 tokens)
**File:** `SKILL.md` lines 58–103
**Affected path:** On Activation → Steps 3, 5, 7

**What the LLM is currently doing:**
Step 5 instructs the LLM to attempt loading up to five file paths (resolved from `{agent.*_path}` config keys), record a `loaded | missing` status for each, and then render a formatted Knowledge Disclosure block based on those statuses. This is file-existence checking and string templating — both fully deterministic given the config values.

**What a script would do instead:**
```python
# scripts/check_knowledge_status.py
# Input: resolved config dict (JSON from resolve_config.py)
# Output: JSON { "target_architecture": "loaded"|"missing", ... }
# Also outputs: file contents as separate per-key stdout blocks for LLM context injection
```
The script checks each configured path for existence (and optionally reads the content), outputs a machine-readable status JSON, and pre-renders the Knowledge Disclosure block as a string. The LLM receives the disclosure block ready-to-paste and the file contents pre-loaded — zero status-assembly tokens spent.

**Estimated token savings:** 100–200 tokens per activation (status assembly + disclosure formatting).
**Pre-pass potential:** Yes — strong. Script output feeds directly into Step 7 routing logic.
**Reuse:** Medium — specific to skills that use the same `persistent_facts` + knowledge-status pattern.

---

### Finding 3 — Glob Expansion for `persistent_facts` File Entries (Step 3)

**Severity:** Medium (LLM Tax: Moderate, 100–300 tokens depending on glob complexity)
**File:** `SKILL.md` lines 58–60
**Affected path:** On Activation → Step 3

**What the LLM is currently doing:**
`persistent_facts` entries prefixed with `file:` may be glob patterns. The LLM is instructed to "expand globs, load each file as a separate fact, missing files → warning". Glob expansion is a filesystem operation: given a pattern and a directory tree, the result is always the same list of matching paths. There is no judgment involved.

**What a script would do instead:**
A Python script (could be the same `check_knowledge_status.py` or a companion `expand_facts.py`) receives the `persistent_facts` list, expands any `file:` globs using `pathlib.Path.glob()`, emits the resolved file paths as JSON, and surfaces warnings for non-matching patterns. The LLM receives a flat list of concrete file paths to load — no glob mechanics, no expansion logic in the prompt.

**Estimated token savings:** 100–300 tokens per activation, scaling with number of glob entries.
**Pre-pass potential:** Yes — direct input to LLM context loading.
**Reuse:** High — any BMAD skill using glob-based `persistent_facts` benefits identically.

---

### Finding 4 — Landscape Registry Scan in Reuse Discovery (reuse-discovery.md, Approach Step 2)

**Severity:** Medium (LLM Tax: Moderate, 200–400 tokens per invocation)
**File:** `references/reuse-discovery.md` lines 52–54
**Affected path:** Reuse & Landscape Discovery → Approach

**What the LLM is currently doing:**
Step 2 instructs the LLM to scan the landscape registry "by relevant capability areas" across all system categories (Core Banking, CRM, Digital, Payments, ESB/Event Hub/API Gateway, DWH, Data Lake). In practice, if the registry is a structured file (TOML, JSON, YAML, or Markdown table), the LLM reads it, finds all entries with matching `capability_area` or `domain` fields, and returns a filtered list. Filtering structured data on field values is a textbook script task.

**What a script would do instead:**
```python
# scripts/scan_registry.py --capability-area "Payment Initiation" --registry path/to/registry.toml
# Output: JSON array of matching registry entries with name, owner, status, integration_contract
```
The LLM receives the pre-filtered candidate list as a JSON block. It then applies judgment — ranking, anti-recommendation analysis, greenfield detection — but skips the mechanical scan.

**Estimated token savings:** 200–400 tokens per reuse-discovery invocation (registry scan portion).
**Pre-pass potential:** Yes — the filtered candidate list is direct structured input for the LLM's ranking and anti-recommendation reasoning.
**Reuse:** High — the same scan is referenced in `architectural-consultation.md` (Approach, landscape traversal) and implicitly in `adversarial-review.md` (duplication axis). A single script serves three capabilities.

---

### Finding 5 — Compliance Checklist Scope Determination (compliance-check.md, Approach)

**Severity:** Low (LLM Tax: Light, 50–100 tokens)
**File:** `references/compliance-check.md` lines 35–36
**Affected path:** Compliance & Target-Architecture Check → Approach

**What the LLM is currently doing:**
The approach tells the LLM to read `{agent.compliance_corpus_path}` and "read only relevant documents (determined by initiative domain — cards → PCI DSS, individuals → 152-ФЗ, payments → 115-ФЗ)". The domain-to-regulation mapping is a lookup table: if domain contains indicator X, include regulation Y. This is a rule-based filter.

**What a script would do instead:**
A lightweight domain-classifier script takes a declared domain string (from the user's input or discovery) and returns the applicable regulation set as a list. This is a < 30-line Python function, not reasoning.

**Estimated token savings:** 50–100 tokens per compliance-check invocation.
**Pre-pass potential:** Modest — the output is a small list; the savings are real but not critical.
**Reuse:** Medium — applicable to any skill involving compliance scoping.
**Note:** This is the lowest-priority finding. The domain-to-regulation mapping is simple enough that the per-invocation tax is light. Worth including in a combined pre-pass script but not worth a standalone script.

---

### Finding 6 — ADR/SD Date Stamping and Filename Slug Generation (artifact-generation.md, Output Formats)

**Severity:** Low (LLM Tax: Light, < 50 tokens)
**File:** `references/artifact-generation.md` lines 107, 166, 250
**Affected path:** All three artifact output paths

**What the LLM is currently doing:**
Output file paths use `{YYYYMMDD}` date stamps and `{slug}` values derived from initiative names. The LLM generates both. `{YYYYMMDD}` is pure clock-reading (always the same given the same moment). Slug generation from a string (lowercase, replace spaces with hyphens, strip special characters) is a deterministic string transformation.

**What a script would do instead:**
Any pre-pass script already running at activation time (e.g., `check_knowledge_status.py`) emits `today_date: "20260527"` in its JSON output. Slug generation is a one-liner (`re.sub(r'[^a-z0-9]+', '-', name.lower()).strip('-')`). These values are injected as resolved variables before the LLM writes any output path.

**Estimated token savings:** < 50 tokens per artifact-generation invocation. Negligible in isolation.
**Pre-pass potential:** Yes — trivially bundled into any existing pre-pass.
**Note:** Not worth a standalone script. Bundle into the activation pre-pass if one is built.

---

## Aggregate Savings Estimate

| Finding | Per-invocation savings | Invocation frequency | Impact |
|---|---|---|---|
| F1: TOML merge fallback | 500–800 tokens | Low (only when external script unavailable) | High when triggered |
| F2: Knowledge status assembly | 100–200 tokens | Every activation | Medium |
| F3: Glob expansion | 100–300 tokens | Every activation with glob facts | Medium |
| F4: Registry scan pre-pass | 200–400 tokens | Every reuse-discovery / consultation | Medium–High |
| F5: Compliance scope filter | 50–100 tokens | Every compliance-check | Low |
| F6: Date/slug generation | <50 tokens | Every artifact save | Negligible |

**Baseline activation tax (F2 + F3, always):** 200–500 tokens saved per session.

**Capability-level tax (F4 per reuse/consultation, F5 per compliance-check):** 250–500 additional tokens saved per capability invocation.

**Realistic aggregate across a typical session** (1 activation + 1 reuse-discovery + 1 compliance-check): **450–1,000 tokens saved**, or roughly 15–30% of activation + routing overhead.

---

## Recommended Script Roadmap

**Priority 1 — Activation pre-pass bundle** (`scripts/activation_prepass.py`):
Combines F2 (knowledge status check + disclosure block rendering), F3 (glob expansion for persistent_facts), and F6 (date stamp + slug helper). Runs once at activation. Emits a JSON envelope the LLM reads as its first context block. Eliminates the most consistent per-session tax.

**Priority 2 — Registry scan pre-pass** (`scripts/scan_registry.py`):
Standalone script covering F4. Takes capability area(s) as arguments, scans the landscape registry file, returns filtered candidates as JSON. Called at the start of reuse-discovery and architectural-consultation capabilities. Single script serves three capabilities.

**Priority 3 — Config fallback hardening** (`scripts/resolve_config.py`):
Covers F1. A local minimal version of the shared `resolve_customization.py` scoped to the `agent` block, so the LLM fallback path in Step 1 can be removed entirely. Eliminates the heaviest per-incident token cost.

**Not recommended as script (keep as LLM):** F5 compliance scope determination — too lightweight to justify a separate script; bundle into activation pre-pass if desired. All judgment operations in every capability (trade-off analysis, finding severity, recommendation generation, citability checks) — these are the skill's core value and require LLM reasoning.
