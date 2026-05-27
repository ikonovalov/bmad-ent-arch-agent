# Prompt Craft Analysis — agent-archon

**Analyzer:** PromptCraftBot  
**Date:** 2026-05-27  
**Skill path:** `skills/agent-archon`  
**Model:** Stateless agent (is_memory_agent: false)

---

## Assessment

**Skill type:** Multi-capability enterprise domain agent (5 capabilities). Not a memory agent — all standard stateless checks apply.

**Overview quality:** The Overview at SKILL.md:10 is 5 lines (pre-pass confirmed: `overview_lines: 5`). It is dense and functional: names all five modes, mentions headless variants, states the mission directly. It reads like a mission brief, not a marketing paragraph. The mission sentence (`**Your Mission:**`) is an effective persona anchor. No philosophical disconnection. For a five-capability agent, 5 lines of overview is lean-but-sufficient, not thin.

**Persona context quality:** Identity (SKILL.md:16), Communication Style (SKILL.md:18–28), and Principles (SKILL.md:30–36) together form a coherent and load-bearing persona block. The communication style section includes three voiced examples that are distinctly architectural in register — these are not decoration. The "citability" principle echoed both in SKILL.md and in each capability prompt is a genuine behavioral constraint, not repetition: each capability applies it in its own domain context. Identity is precise (20 years experience, named frameworks, named landscape components). No philosophical drift, no self-explanation to the model.

**Progressive disclosure:** All 5 capability prompts are loaded on demand from `references/`. The SKILL.md Capabilities table routes cleanly to each. Pre-pass confirms all 5 prompts have config headers and progression conditions. Token-heavy templates (artifact-generation.md at 320 lines / ~2,972 tokens) are kept in references, not inlined. This is correct progressive disclosure design.

**Synthesis:** This is a well-crafted domain agent for a regulated banking context. The prompts are outcome-focused, persona-consistent, and structurally sound with zero waste patterns detected by pre-pass. The primary craft issues are: (1) a thin SKILL.md Overview that under-sells the agent's theory of mind and operational context to a bootstrapping executor; (2) the On Activation section in SKILL.md is a detailed seven-step procedure that could benefit from clearer failure-mode guidance in Step 3; (3) `artifact-generation.md` is the largest prompt (~2,972 tokens, 320 lines) and embeds full document templates inline rather than separating procedural guidance from output scaffolding.

---

## Prompt Health Summary

| Metric | Value |
|---|---|
| Total capability prompts | 5 |
| Prompts with config header `{communication_language}` | 5 / 5 |
| Prompts with progression conditions | 5 / 5 |
| Waste patterns detected | 0 |
| Back-references to SKILL.md | 0 |
| Wall-of-text blocks | 0 |
| Suggestive loading | 0 |
| Total token estimate (all files) | ~10,850 |

All prompts are self-contained: each states its purpose in the first paragraph without requiring SKILL.md in context. Config header (`{communication_language}` / `{document_output_language}`) appears on line 9 of every prompt — consistent and correctly placed before any substantive instruction.

---

## Per-Capability Craft

### adversarial-review.md (~1,225 tokens, 98 lines)

**Outcome focus:** Excellent. "What Success Looks Like" is structured as eight numbered axes with explicit priority order — the agent knows exactly what to look for and in what sequence. Completion criteria are concrete and binary: either each axis is covered and every finding is sourced, or the review is not done.

**Voice alignment:** Fully aligned. "Adversarial — не значит токсичный" is persona-consistent character guidance, not padding. The Anti-Patterns section mirrors the Communication Style principle of citability.

**Progression:** Degraded Mode handles all three missing-knowledge states. Progression to `compliance-check` and `architectural-consultation` is explicit in Completion Criteria.

**Minor note:** The Output Format template at lines 69–97 is a fenced block of ~27 lines. This is appropriate for a structured report format. No issue.

---

### architectural-consultation.md (~1,306 tokens, 95 lines)

**Outcome focus:** Strong. "What Success Looks Like" lists five concrete deliverables the user walks away with — including the reuse map, options with trade-off matrix, and a sourced recommendation. The approach section is well-sequenced (understand → landscape → form options → recommend), avoiding the anti-pattern of jumping to recommendation.

**Voice alignment:** The example anti-pattern at line 48 ("«Используй Kafka» — не консультация...") is an ideal persona illustration — it teaches both what the agent should produce and the register it should use.

**Progression:** Correctly degrades on all three missing knowledge states. Handoff to `adversarial-review` is present.

**Minor note:** The output template (lines 68–94, fenced block of ~25 lines) is structural scaffolding — proportionate and non-wasteful.

---

### artifact-generation.md (~2,972 tokens, 320 lines)

**Outcome focus:** Strong in procedural guidance. Discovery Dialog, Anchoring, and Draft & Iterate sub-sections of Approach are well-differentiated. The `[NEEDS INPUT: {точный вопрос}]` convention is a distinctive and practical quality signal.

**Voice alignment:** Consistent with persona. The anti-pattern "Шаблон-в-лоб" directly enforces the Discovery-first principle established in SKILL.md.

**Size concern (Medium):** This is the largest prompt at 320 lines / ~2,972 tokens — more than twice any other capability prompt. The bulk is three complete document templates embedded as fenced code blocks (ADR: lines 109–160, Solution Design: lines 166–244, ARB Package: lines 252–311). These templates (~183 fenced lines, pre-pass confirmed) are genuinely necessary as output scaffolds, but they make the procedural guidance harder to scan. The templates could be extracted to separate template files (e.g., `references/templates/adr.md`) and referenced by path, reducing this prompt to ~130–140 lines while preserving output quality.

**Headless section:** Lines 315–319 are a concise, well-placed headless mode instruction. Correct placement at end of file.

---

### compliance-check.md (~1,377 tokens, 111 lines)

**Outcome focus:** Excellent. "Тон: клинический" is the tightest persona-mode shift in the entire skill — this prompt deliberately suppresses the consultative voice for an auditor mode. This is sophisticated and correct design: same agent, different register, explicitly signaled.

**Voice alignment:** The auditor persona is consistently maintained throughout. Anti-patterns section correctly prohibits architectural options discussion ("Это в `adversarial-review`"), enforcing capability separation.

**Headless support:** The `--headless:compliance-audit` section at lines 108–111 is appropriately brief — bank-wide audit scope is correctly distinguished from single-initiative scope.

**Strength:** The four-axis structure (Регуляторика / Целевая архитектура / Принципы банка / Стандарты данных) maps directly to the output template's four checklist tables. This structural coherence between Approach and Output Format is exemplary.

---

### reuse-discovery.md (~1,858 tokens, 148 lines)

**Outcome focus:** Strong. Greenfield Mode (lines 32–48) is a standout feature: the prompt anticipates the failure case where the registry scan returns zero results and provides a full recovery procedure — including BIAN reference patterns, a registry-entry draft template, and explicit citability guidance. This prevents the agent from returning empty tables or hallucinating registry entries.

**Voice alignment:** Consistent. Anti-pattern "«Можно использовать X» без проверки" enforces the SKILL.md principle of reuse-before-build with the same citability constraint.

**Dual output format:** The presence of two output templates (standard and Greenfield variant, lines 86–147) is intentional and non-redundant. The Greenfield template is structurally different from the standard one — not a copy. This is correct design.

**Minor note:** At 148 lines this is the second-largest prompt but substantially shorter than artifact-generation.md. The size is justified by the two distinct operational modes (normal vs. Greenfield).

---

## Key Findings

### Finding 1 — Medium

**File:** `SKILL.md:8–12` (Overview section)  
**Issue:** The Overview is 5 lines / functionally adequate, but it is written as a capability enumeration rather than a theory-of-mind framing. A bootstrapping agent executor reading only the SKILL.md (without context from the persona sections) would know *what* Archon does but not *why* it matters or *how* its judgment is calibrated. The Identity section at line 16 carries the deeper framing but is a separate section — the Overview does not bridge to it.  
**Why it matters:** For stateless agents, the Overview is the first thing loaded and establishes the interpretive frame for everything that follows. A thin Overview means capability-routing decisions and persona initialization are less grounded.  
**Fix:** Expand the Overview by 3–4 sentences to include: (a) the core judgment model ("drift from target architecture is the primary signal"), (b) the citability constraint as a behavioral anchor, and (c) the bank-domain context (regulated environment, multi-layer knowledge dependency). Target ~8–9 sentences. This is persona investment, not bloat.

---

### Finding 2 — Medium

**File:** `references/artifact-generation.md:103–313` (Output Formats — embedded templates)  
**Issue:** Three complete document templates (ADR, Solution Design, ARB Package) totaling ~183 lines of fenced blocks are embedded directly in the procedural guidance prompt. This makes the prompt 320 lines — more than double all other capability prompts. The templates are correct and necessary, but their placement inside the capability prompt conflates "how to conduct discovery" with "what the output looks like."  
**Why it matters:** When an agent executes the Discovery Dialog and Approach sections, it must scroll past the entire template set to reach the Headless section. More importantly, future edits to output formats risk unintentional changes to procedural guidance and vice versa. Template maintenance becomes coupled to prompt maintenance.  
**Fix:** Extract the three templates to separate files: `references/templates/adr-template.md`, `references/templates/solution-design-template.md`, `references/templates/arb-package-template.md`. Replace the inline blocks with: `Output template: load \`references/templates/adr-template.md\`` (etc.). The procedural prompt drops to ~130 lines; templates are independently maintainable. This does not affect output quality — the agent loads the template on demand.

---

### Finding 3 — Low

**File:** `SKILL.md:60` (Step 3: Load Persistent Facts)  
**Issue:** The instruction "Записи с префиксом `file:` — пути/глобы: разверни глобы, загрузи содержимое каждого файла как отдельный факт, отсутствующие файлы — warning, не fail" is correct in intent but does not specify what the warning should look like or where it should surface. An agent that silently processes glob expansion failure is indistinguishable from one that succeeds — both continue to Step 4. The warning disposition is ambiguous relative to Step 7's Knowledge Disclosure block (which handles knowledge-file misses explicitly).  
**Why it matters:** Persistent facts could include critical architectural context. A silent miss could cause the agent to operate without facts it believes are loaded, without surfacing this in the Knowledge Disclosure block.  
**Fix:** Add: "Warning при отсутствующем файле фиксируй в `knowledge_status` под ключом `persistent_facts_warnings` — и включай в блок Knowledge Disclosure, если хотя бы один файл не загружен."

---

### Finding 4 — Low

**File:** `references/compliance-check.md:35–43` (Approach section)  
**Issue:** The Approach section correctly instructs loading `{agent.compliance_corpus_path}` and reading only relevant documents. However, it does not define "relevant" — the parenthetical "(определяется доменом инициативы — карты → PCI DSS, физлица → 152-ФЗ, payments → 115-ФЗ)" provides examples but not a decision rule. For a compliance audit, selecting the wrong subset of documents is a meaningful error.  
**Why it matters:** If the compliance corpus contains many documents, an agent might load too few (missing applicable requirements) or too many (inefficient). The current guidance is illustrative, not deterministic.  
**Fix:** Add a brief decision rule: "Определяй периметр по домену данных из инициативы: если в scope ПДн — 152-ФЗ обязателен; карточные данные — PCI DSS обязателен; EU-клиенты — GDPR обязателен; операции с клиентами-ФЛ — проверяй 115-ФЗ. Остальное — по контексту." This converts an illustrative hint into an actionable decision procedure.

---

### Finding 5 — Note

**File:** `SKILL.md:113–123` (Capabilities table)  
**Observation:** The Capabilities routing table uses pipe-delimited Markdown and is 7 lines (pre-pass confirmed: `table_lines: 7`). The "Когда" column functions as a capability-selection trigger. These triggers are concise but somewhat user-intent-dependent (e.g., "Пользователь приносит инициативу/идею и хочет понять, как её вписать" vs. "Пользователь приносит готовый ADR / Solution Design / proposal на разбор"). The distinction between AC and AR is clear in the Capabilities table and correctly differentiated. No issue — noting for completeness.

---

### Finding 6 — Note

**File:** All capability prompts, `Completion Criteria` sections  
**Observation:** Every capability prompt includes explicit cross-capability routing at the end of Completion Criteria (e.g., "proceed to `compliance-check`", "proceed to `adversarial-review`"). These are consistent, correct, and non-redundant — each prompt routes to a different next capability based on what was discovered. This is a strength of the design, noted here because it is an uncommon pattern worth preserving.

---

## Strengths

**Citability as a structural invariant.** The citability principle is not just stated in SKILL.md — it is operationalized differently in each capability prompt. In adversarial-review it becomes the "тест на цитируемость" with an explicit severity downgrade rule. In compliance-check it becomes the "размытость источника" anti-pattern with a negative example. In artifact-generation it becomes the anti-pattern "ссылки понарошку." This is excellent prompt engineering: a single principle expressed through domain-appropriate enforcement mechanisms.

**Degraded Mode across all capabilities.** All five prompts define what the agent should do when one or more knowledge files are missing — and they do so differently based on the capability's dependency profile. adversarial-review degrades gracefully by axis; reuse-discovery has a full Greenfield Mode rather than just a degraded mode. This is robust design that anticipates real operational conditions.

**Tone modulation by capability.** The agent explicitly shifts register between capabilities: consultative in architectural-consultation, adversarial-but-collegial in adversarial-review, clinical/auditor in compliance-check. These are not implicit expectations — they are stated directly in each prompt. This level of intra-skill voice management is sophisticated and well-executed.

**Greenfield Mode in reuse-discovery.** The explicit handling of zero-result registry scans (lines 32–48) with BIAN reference fallback, registry-entry draft, and citability labeling (`[BIAN baseline]`) is the strongest single section in the skill. It transforms a potential failure mode into a constructive output.

**Self-contained capability prompts.** Every capability prompt states its purpose in the first two lines without back-references to SKILL.md. An agent receiving only the capability prompt as context can execute the task correctly. This is correct design for progressive disclosure architectures.

**Consistent output naming convention.** All five capability prompts specify output file paths with the pattern `{agent.assessment_output_path}/{type}-{slug}-{YYYYMMDD}.md`. This is a small but important operational consistency that reduces ambiguity in multi-session or headless contexts.
