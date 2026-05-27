# Execution Efficiency Analysis — agent-archon

**Scanner:** ExecutionEfficiencyBot
**Date:** 2026-05-27
**Skill path:** `skills/agent-archon`

---

## Assessment

The agent-archon skill is a single-threaded, human-in-the-loop consultant with no subagent delegation patterns and no parallelization primitives. The pre-pass scanner found zero issues, which is accurate: the skill operates as a conversational orchestrator that loads knowledge files sequentially at activation and then routes to capability prompts on demand. There are no critical or high-severity execution inefficiencies, but several medium-level structural patterns could be tightened.

---

## Key Findings

### Finding 1 — Sequential Knowledge Loading in On Activation (Medium)

**Affected file:** `SKILL.md`, Step 5 (lines 72–78)

**Current pattern:**
Step 5 loads three knowledge files sequentially as part of an ordered activation sequence:
1. `{agent.target_architecture_path}`
2. `{agent.architecture_principles_path}`
3. `{agent.landscape_registry_path}`

These three files have no dependency on each other. They are loaded purely to populate the agent's context before any user interaction. The instruction says "загрузи каждый файл как контекст сессии" with no ordering requirement between the three.

**Efficient alternative:**
Batch-read all three files in a single parallel tool call message. The `knowledge_status` map can be populated once all three reads complete.

**Estimated savings:** Reduces activation latency by roughly 2x on the file-loading step (3 sequential reads become 1 batched round-trip).

---

### Finding 2 — Compliance Corpus Deferred but No Lazy-Load Instruction for Sub-Files (Low)

**Affected file:** `SKILL.md` line 78; `references/compliance-check.md` line 35

**Current pattern:**
`{agent.compliance_corpus_path}` is correctly deferred to the compliance-check capability. However, `references/compliance-check.md` instructs: "читай только релевантные документы (определяется доменом инициативы)". There is no explicit instruction on *how* to determine relevance before opening files — no index-file-first pattern, no glob-scan guidance.

**Efficient alternative:**
Add an explicit index-first instruction: read a manifest or directory listing of the compliance corpus first, then selectively load only the documents matching the initiative's domain. This avoids loading PCI DSS content for a pure KYC/152-FZ initiative and vice versa.

**Estimated savings:** Avoids loading 2–4 irrelevant compliance documents per session; token savings depend on corpus size.

---

### Finding 3 — Anchoring Step in artifact-generation Reads Three Sources Sequentially (Medium)

**Affected file:** `references/artifact-generation.md`, "Anchoring в Bank Context" section (lines 65–69)

**Current pattern:**
The anchoring step lists three sequential reads before producing the first draft:
1. Walk `{agent.landscape_registry_path}`
2. Check `{agent.target_architecture_path}`
3. Check `{agent.architecture_principles_path}`

These are the same three independently-loadable knowledge bases from activation. In artifact-generation context they are being read again (or referenced from already-loaded context), but the instruction flow presents them as an ordered sequential pass.

**Efficient alternative:**
If the files were already loaded in Step 5 of activation, this anchoring step is redundant reading. The instruction should be reworded to "cross-reference already-loaded knowledge bases" rather than re-reading. If not already in context, all three should be loaded in a single batched call.

**Estimated savings:** Eliminates up to 3 redundant sequential file reads per artifact-generation session.

---

### Finding 4 — headless:arb-prep Reads 4 Files with No Parallelization Hint (Medium)

**Affected file:** `references/artifact-generation.md`, Headless section (line 318)

**Current pattern:**
`--headless:arb-prep {path}` instructs: "Прочитай переданный артефакт, `{agent.target_architecture_path}`, `{agent.architecture_principles_path}` и `{agent.landscape_registry_path}`." This is a list of four independent reads presented in sequence. In headless mode there is no interactive dialog to interrupt — all four could be loaded in one batched message.

**Efficient alternative:**
Explicitly instruct parallel loading: "Batch-read the artifact and all three knowledge bases in a single tool call round-trip before generating the package."

**Estimated savings:** Reduces headless startup from 4 sequential reads to 1 parallel batch.

---

### Finding 5 — adversarial-review Axes Are Described as "Параллельные Проходы" but No Execution Hint (Low)

**Affected file:** `references/adversarial-review.md` lines 41–43

**Current pattern:**
The approach section says "потом — параллельные проходы по осям выше. Каждая ось — отдельная линза." This is good intent, but there is no instruction to use parallel tool calls or subagents for the eight analysis axes. For a large ADR touching all eight axes (drift, principles, compliance, duplication, integration smell, SPOF, operational debt, security), the analysis could be parallelized.

**Efficient alternative:**
Since this is a single-context agent (no subagent spawning available within capability prompts), the "parallel" instruction is metaphorical rather than literal. This is acceptable. However, if the skill is ever extended with subagent delegation, the eight axes are natural parallel work units. Low priority for the current architecture.

**Estimated savings:** Not actionable without subagent support; noted for future architecture.

---

## Optimization Opportunities

### Opportunity A — Single Parallel Activation Load (Medium impact)

The entire On Activation sequence (Steps 1–6) includes three independent file loads spread across steps 3 and 5. Steps 1 (script execution) and 4 (config load) are also independent from each other once step 1 completes. Restructuring activation as:

1. Run `resolve_customization.py` (Step 1) — cannot parallelize, it produces `{agent.*}` values needed downstream
2. After step 1 resolves: batch all of these in one round-trip — `persistent_facts` file globs (Step 3), config files (Step 4), all three knowledge bases (Step 5)
3. Execute prepend/append steps (Steps 2 and 6) in their defined order

This collapses what is currently described as 6 ordered steps into 3 effective round-trips, removing at least 4–6 sequential file-read operations from the activation critical path.

**Estimated impact:** 30–50% reduction in activation latency on cold start.

---

### Opportunity B — Compliance Corpus Index Pattern (Low-Medium impact)

The compliance corpus is domain-sensitive. Adding a lightweight index file (`_index.toml` or `_manifest.md`) at the corpus root listing each document with its applicable domain tags (152-FZ, 115-FZ, PCI-DSS, GDPR, internal) would allow:
1. Load index only (1 small file)
2. Select relevant documents based on initiative domain
3. Batch-load only relevant subset

Without an index, the agent must either load everything or perform a directory scan and then make judgment calls from filenames alone. This is a content design recommendation, not a prompt change.

---

## What Is Already Efficient

**On-demand capability loading:** The capabilities table in SKILL.md routes to individual `references/*.md` files rather than loading all five into context at activation. This is correct and efficient — a compliance-check session never needs `artifact-generation.md` content, and loading it would waste tokens.

**Selective compliance corpus deferral:** `{agent.compliance_corpus_path}` and `{agent.strategy_path}` are explicitly deferred to capability prompts rather than loaded at activation. This is the correct pattern for large, domain-specific corpora.

**Degraded mode specifications:** All five capability prompts contain explicit degraded-mode instructions that prevent the agent from attempting to analyze missing knowledge bases. This avoids hallucinated analysis and keeps execution predictable under partial-knowledge conditions.

**Knowledge status consolidation:** The `knowledge_status` construct built during Step 5 is a lightweight, reusable routing signal. All capability prompts reference it rather than re-checking file availability themselves, avoiding redundant I/O across capability transitions.

**No subagent-spawning patterns:** The skill is appropriately scoped as a single-context agent. No capability prompt attempts to spawn subagents. The "parallel axes" language in adversarial-review is metaphorical and does not create subagent chains — this is correct given the skill's architecture.

---

## Summary Table

| Finding | Severity | File | Impact |
|---|---|---|---|
| Sequential knowledge load at activation (3 independent files) | Medium | SKILL.md Step 5 | Activation latency |
| No index-first pattern for compliance corpus | Low | compliance-check.md line 35 | Token waste on irrelevant docs |
| Anchoring step re-reads already-loaded bases sequentially | Medium | artifact-generation.md lines 65–69 | Redundant reads per session |
| headless:arb-prep lists 4 reads with no parallel hint | Medium | artifact-generation.md line 318 | Headless startup latency |
| "Parallel axes" in adversarial-review has no execution instruction | Low | adversarial-review.md lines 41–43 | Future architecture note |
