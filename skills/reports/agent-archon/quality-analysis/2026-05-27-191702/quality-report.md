# BMad Method · Quality Analysis: agent-archon

**🏛️ Archon** — Корпоративный архитектор банка
**Analyzed:** 2026-05-27T19:17:02Z | **Path:** skills/agent-archon
**Interactive report:** quality-report.html

---

## Agent Portrait

Archon is a grey-haired Chief Enterprise Architect who has spent twenty years at the intersection of bank business and systems — fluent in TOGAF, ArchiMate, BIAN, DDD, and every compliance framework from 152-FZ to PCI DSS. He thinks in 3–5 year horizons and operates at the whiteboard: laying out trade-offs rather than dispensing verdicts, ensuring every recommendation is citable to a specific principle, regulation, or landscape entry. His singular mission is to catch when an initiative drifts from the bank's approved target architecture — even when nobody else is paying attention.

---

## Capabilities

| Capability | Status | Observations |
| ---------- | ------ | ------------ |
| Архитектурная консультация | Good | Strong outcome focus and trade-off structure; missing handoff to artifact-generation at completion |
| Adversarial Review | Good | 8-axis structure is exemplary; compliance_corpus_path silently not loaded despite being relevant |
| Compliance & Target Check | Good | Clinical tone shift is sophisticated design; compliance scope filter could be made deterministic |
| Reuse & Landscape Discovery | Good | Greenfield Mode is a standout feature; registry scan is an LLM-tax candidate for scripting |
| Draft Architectural Artifact | Needs attention | Largest prompt at 320 lines; three full templates embedded inline; anchoring step re-reads already-loaded knowledge bases sequentially |

---

## Assessment

**Good** — Archon is an unusually well-engineered domain agent: zero structural defects, five coherent capabilities with full degraded-mode coverage, and a citability principle operationalized consistently across every output format. The primary opportunity is a cluster of efficiency and workflow-continuity improvements — the agent works well in a single-capability session but loses context across capability transitions, and its headless modes lack the structured output contracts needed for CI/CD integration.

---

## What's Broken

No critical or high-severity issues were found. The agent is production-ready.

---

## Opportunities

### 1. Sequential I/O on Hot Paths (medium — 7 observations)

Multiple activation and capability steps instruct the LLM to perform independent file reads sequentially: the three knowledge bases in Step 5, the anchoring pass in artifact-generation, and four reads in `--headless:arb-prep`. These reads have no dependency on each other and could all be issued as a single batched round-trip. Additionally, knowledge-status assembly, glob expansion for persistent_facts, and date/slug generation are fully deterministic operations currently paid for in LLM tokens on every session.

**Impact:** Eliminating these sequential reads would reduce activation latency by an estimated 30–50% on cold start and save 450–1,000 tokens per typical session (activation + one reuse/consultation + one compliance check).

**Fix:** Restructure Step 5 to batch all three knowledge-base reads in one parallel call. Reword artifact-generation's anchoring step to "cross-reference already-loaded knowledge bases" rather than re-reading. In `--headless:arb-prep`, explicitly instruct a single batched read of all four sources. Build an `activation_prepass.py` script that handles knowledge-status assembly, glob expansion, date stamping, and slug generation — eliminating the LLM-as-file-checker pattern across every activation.

*Constituent findings: Execution Efficiency F1, F3, F4; Script Opportunities F2, F3, F4, F6.*

---

### 2. Headless Modes Lack Automation Contracts (medium — 5 observations)

The three headless flags (`--headless:landscape-drift-scan`, `--headless:compliance-audit`, `--headless:arb-prep`) represent forward-thinking governance automation design, but none specify structured output formats, exit-code behavior for blocker findings, or error handling for unwritable output paths. Downstream consumers (CI pipelines, dashboards, other agents) must parse natural-language Markdown to determine severity counts or whether the run found blockers. Additionally, `--headless:landscape-drift-scan` has no dedicated reference section — its behavior lives only in a one-line description in SKILL.md Step 7.

**Impact:** Without structured output contracts, the automation value of headless modes cannot be realized. CI gates and governance dashboards require machine-readable summaries, not prose.

**Fix:** Add a YAML front-matter block to every headless report specifying `run_date`, `mode`, `knowledge_status`, and a `summary` object with named counts (`blockers`, `gaps`, `drift_items`, `compliant`). Define an exit-failure contract for when any blocker-severity finding is present. Add a `## Headless: --headless:landscape-drift-scan` section to `architectural-consultation.md` (or a new `references/landscape-drift.md`) consistent with the pattern in `compliance-check.md` lines 108–111.

*Constituent findings: Enhancement Opportunities HIGH×2; Agent Cohesion LOW; Structure Analysis LOW; Execution Efficiency F4.*

---

### 3. Workflow Continuity Across Capability Transitions (medium — 4 observations)

When a user moves from architectural-consultation to adversarial-review to artifact-generation, each capability starts cold: the options explored, systems identified, compliance flags raised, and trade-off decisions made in the prior capability are not explicitly carried forward. The consultation's completion criteria also omit artifact-generation as a natural next step, creating a routing gap for the most common post-consultation action (formalizing a chosen option into an ADR or Solution Design).

**Impact:** Multi-capability sessions — the real-world pattern for enterprise architecture work — feel like a series of disconnected tool invocations rather than a continuous working relationship. Users re-answer questions they already answered.

**Fix:** Define a lightweight session context object (initiative name/slug, identified systems, compliance flags raised, options generated) that each capability writes to and reads from at the start. Add to `architectural-consultation.md` completion criteria: "If the user wants to formalize the chosen option into an ADR or Solution Design, proceed to `artifact-generation`." Add an explicit "Bringing into this review: [context from prior step]" opening step in adversarial-review and artifact-generation.

*Constituent findings: Agent Cohesion MEDIUM×2; Enhancement Opportunities MEDIUM; Prompt Craft MEDIUM (Finding 1).*

---

### 4. Customization Surface Gaps for Org-Specific Operational Parameters (low — 4 observations)

`compliance_corpus_path` is declared as a first-class override in `customize.toml` but is silently ignored by `adversarial-review.md` — the second most commonly triggered capability. An operator who invests in populating internal compliance documents will not see that investment reflected in adversarial reviews. Additionally, the ADR numbering sequence and ARB calendar date are org-varying parameters the agent currently resolves by inference or `TBD` stubs, and `strategy_path` load semantics are inconsistent across capabilities without documentation.

**Impact:** Operators who invest in customizing knowledge sources receive incomplete coverage. ADR numbering collisions are likely on multi-author teams.

**Fix:** In `adversarial-review.md`, add an explicit load step for `{agent.compliance_corpus_path}` (mirroring compliance-check.md line 35) or document in `customize.toml` that `compliance_corpus_path` is consumed only by compliance-check. Add `adr_index_path = ""` and `arb_next_session_date = ""` scalars to `customize.toml`. Add a one-line comment to `strategy_path`: "consumed by architectural-consultation and artifact-generation when explicitly referenced."

*Constituent findings: Customization Surface MEDIUM×2, LOW×2.*

---

### 5. First-Use Experience and Routing UX (low — 3 observations)

The frontmatter description opens with a Russian sentence before the English "Use when" trigger phrase, which may cause unpredictable prefix-based routing. New users with no knowledge files loaded receive a Knowledge Disclosure block but no path forward — no setup guidance, no pointer to which config keys to set. Capability routing is purely intent-based with no soft confirmation before committing to a mode, meaning confused users receive a full output in the wrong capability before realizing the mismatch.

**Impact:** Cold-start experience for new deployments is "quietly broken" — functional-looking but producing degraded output with no clear remedy. Expert users can power through; first-timers and confused users may not.

**Fix:** Move the "Use when" clause to the start of the frontmatter description. Add a conditional branch in Step 7: if all three key files are missing AND session is interactive, offer a brief setup path prompting for the three knowledge-base paths. Add a single-sentence routing confirmation before loading any capability: "Sounds like Adversarial Review (AR). Confirm or pick another: AC / AR / CC / RD / AG."

*Constituent findings: Structure Analysis MEDIUM; Enhancement Opportunities HIGH; Enhancement Opportunities MEDIUM.*

---

## Strengths

**Citability as a structural invariant.** The citability principle is not just stated in SKILL.md — it is operationalized in a capability-appropriate way in all five prompts. In adversarial-review it becomes a severity downgrade rule. In compliance-check it becomes the "размытость источника" anti-pattern. In artifact-generation it becomes the `[NEEDS INPUT: ...]` convention. A single principle, five domain-specific enforcement mechanisms.

**Degraded mode coverage across every capability.** All five capability prompts define exactly what to do when each knowledge file is missing — and they do so differently based on each capability's dependency profile. The Knowledge Disclosure block that surfaces missing files before any response is a strong trust-building mechanism that is architecturally consistent across the entire agent.

**Greenfield Mode in reuse-discovery.** Rather than returning empty tables or hallucinating registry entries when no candidates exist, the capability pivots to BIAN reference patterns, a registry-entry draft template, and explicit citability labeling. This is the strongest single section in the skill — designed for real operational conditions, not just the happy path.

**Tone modulation by capability.** The agent explicitly shifts register between capabilities: consultative in architectural-consultation, adversarial-but-collegial in adversarial-review, clinical/auditor in compliance-check. These are stated directly in each prompt, not left as implicit expectations — sophisticated and well-executed intra-skill voice management.

**Activation sequence design.** The seven-step On Activation sequence is logically ordered and operationally complete: customization resolution precedes prepend hooks, knowledge loading precedes routing, and Knowledge Disclosure is correctly placed as a gate before any user-visible output. The degraded-mode handling in each capability file mirrors the knowledge_status flags established in Step 5, creating a coherent fallback chain.

**On-demand capability loading with zero waste patterns.** All five capability prompts are loaded only when needed; no capability bleeds into another's context. Pre-pass confirmed zero waste patterns, zero back-references, zero wall-of-text blocks across all 10,850 tokens.

**Inter-capability routing graph.** Each capability explicitly names natural next capabilities with clear routing conditions, creating a navigable workflow graph. The five-capability loop (reuse-discovery → architectural-consultation → artifact-generation → adversarial-review → compliance-check) mirrors how real enterprise architecture work unfolds.

---

## Detailed Analysis

### Structure & Capabilities

All required sections are present, correctly ordered, and well-scoped. The five capability files each include config headers, "What Success Looks Like," Degraded Mode, Anti-Patterns, Completion Criteria, and structured Output Format — unusually thorough by any standard. Zero issues detected by pre-pass. One medium finding: the frontmatter description opens with Russian before the English "Use when" trigger, which may affect harness routing. Three low findings: Identity section reads as a CV rather than a priming sentence; config-token variables appear in Communication Style before activation resolves them (architectural choice, not a bug); `--headless:landscape-drift-scan` is documented only inline in SKILL.md without a dedicated capability-file section.

### Persona & Voice

Overview quality is lean-but-sufficient at 5 lines for a 5-capability agent, written as a mission brief with a direct "Your Mission" anchor. The persona block (Identity + Communication Style + Principles) is coherent and load-bearing — the three voiced communication examples are distinctly architectural in register, not decoration. All five capability prompts are self-contained: each states its purpose without back-references to SKILL.md, and each applies the citability principle through its own domain-appropriate mechanism. One medium finding: `artifact-generation.md` at 320 lines is more than double all other capability prompts due to three full document templates embedded inline, coupling template maintenance to procedural guidance maintenance.

### Identity Cohesion

Persona-capability alignment is strong across all five capabilities: the 20-year architect identity, TOGAF/BIAN fluency, and citability principle map directly onto what each capability requires. The five capabilities form a complete architectural engagement loop. One medium finding: the consultation → artifact-generation handoff is missing from completion criteria, creating a routing gap for the most natural post-consultation action. One low finding: drift detection during consultation is reactive (appears at output time) rather than proactive (checked during options discussion). The headless modes demonstrate forward-thinking design for governance automation and are correctly scoped to the three most valuable periodic tasks.

### Execution Efficiency

Pre-pass found zero dependency-cycle or sequential-pattern issues. The agent correctly defers compliance corpus and strategy path to capability prompts, uses on-demand capability loading, and maintains a lightweight `knowledge_status` construct that all capability prompts share without redundant re-checking. Four findings: three medium (sequential knowledge load at activation, sequential anchoring reads in artifact-generation, four sequential reads in `--headless:arb-prep`) and one low (compliance corpus has no index-first pattern for selective loading). The "parallel axes" language in adversarial-review is metaphorical and correctly so — no subagent spawning, appropriate for the single-context architecture.

### Conversation Experience

The agent is well-suited to its primary user archetype: a principal architect or senior product manager who brings well-formed inputs and knows what they want. Expert and automation journeys are well-served. Friction exists for first-timers (capability menu without natural-language examples), confused users (no out-of-scope handling), edge-case users with large multi-initiative inputs (no scope-limiting step before 8-axis adversarial review), and automators (Markdown-only output with no structured severity summary or exit-code contract). Headless modes are classified as "easily adaptable" — the intent is right, the output contracts are not yet CI/CD-grade.

### Script Opportunities

Zero scripts exist in the skill. Six script opportunities identified: F1 (TOML merge fallback, 500–800 tokens/incident, high severity), F2 (knowledge status assembly, 100–200 tokens/activation, medium), F3 (glob expansion, 100–300 tokens/activation, medium), F4 (registry scan pre-pass, 200–400 tokens/capability invocation, medium), F5 (compliance scope filter, 50–100 tokens, low), F6 (date/slug generation, <50 tokens, negligible). Recommended roadmap: Priority 1 — `activation_prepass.py` bundle (F2+F3+F6); Priority 2 — `scan_registry.py` (F4, serves three capabilities); Priority 3 — `resolve_config.py` local fallback (F1). Estimated realistic aggregate: 450–1,000 tokens saved per typical session.

### Customization Surface

`customize.toml` is present with all six required metadata fields populated and consistent with SKILL.md. The override surface is well-matched to the stateless archetype: five knowledge-source paths, an output path, two hook arrays, and a post-assessment command. No boolean toggle farms or persona parameters — all high-risk abuse patterns are absent. The hook surface is append-only, correctly composing across base/team/user layers. Two medium opportunities: ADR numbering sequence and ARB calendar date are org-varying parameters with no dedicated TOML scalar. Two low findings: `compliance_corpus_path` contract is not honored by adversarial-review; `strategy_path` load semantics are inconsistent across capabilities without documentation.

---

## Recommendations

1. **Add parallel file loading to SKILL.md Step 5 and artifact-generation anchoring step** — consolidates three medium efficiency findings into a one-paragraph change with 30–50% activation latency reduction. Low effort, high frequency impact.

2. **Build `activation_prepass.py` bundle script** — eliminates knowledge-status assembly, glob expansion, and date/slug generation LLM tax on every session. Medium effort, persistent per-session savings of 200–500 tokens.

3. **Add YAML front-matter summary block and exit-code contract to all three headless modes** — makes automation-grade CI/CD integration possible without changing any analytical logic. Low effort, unlocks the full value of the headless architecture.

4. **Add `artifact-generation` to architectural-consultation completion criteria and define a lightweight session context object** — two related changes that transform multi-capability sessions from cold-start sequences into continuous workflows. Medium effort, resolves the most common expert-user friction point.

5. **Fix `compliance_corpus_path` load gap in adversarial-review** — highest-priority customization surface fix; one load step addition ensures operators' internal compliance documents are reflected in the second most common capability. Low effort.

6. **Add `--headless:landscape-drift-scan` section to a capability reference file** — completes the headless documentation pattern established by compliance-check and artifact-generation. Low effort.

7. **Add `adr_index_path` and `arb_next_session_date` scalars to `customize.toml`** — eliminates ADR numbering collisions and `TBD` stubs in the most visible agent outputs. Low effort.

8. **Move "Use when" clause to start of frontmatter description** — resolves the medium routing risk from Russian-prefix in SKILL.md frontmatter. Trivial effort.
