# Structure & Capabilities Analysis — agent-archon

**Analyst:** StructureBot  
**Date:** 2026-05-27  
**Pre-pass status:** PASS (0 issues detected)

---

## Assessment

Agent-archon is structurally sound and complete. All required sections are present and correctly ordered, all five capability files exist and are well-formed with config headers and progression markers, and no template artifacts or orphaned references were found. The one notable structural concern is cosmetic: the SKILL.md frontmatter description opens with a Russian-language sentence before the English "Use when" trigger phrase, which may produce inconsistent routing behavior depending on how the harness tokenizes descriptions.

---

## Pre-Pass Findings

The pre-pass scanner reported **zero issues** across all severity levels. All structural checks passed:

- Frontmatter: `name` present, kebab-case, description present, "Use when" clause present.
- Required sections: all found (Overview, Identity, Communication Style, Principles, On Activation, Capabilities).
- Invalid sections: none detected (no "On Exit" or "Exiting").
- Template artifacts: none (no orphaned `{if-*}`, `{displayName}`, etc.).
- Memory paths: none declared — consistent with `is_memory_agent: false`.
- Directness pattern violations: none.
- All five capability files have `has_config_header: true` and `has_progression: true`.

No pre-pass findings require preservation.

---

## Sections Found

| Section | Status |
|---|---|
| Overview | Present (line 8) |
| Identity | Present (line 14) |
| Communication Style | Present (line 18) |
| Principles | Present (line 30) |
| Conventions | Present (line 38) — optional, appropriate |
| On Activation | Present (line 46), with 7 substeps |
| Capabilities | Present (line 113) |

No required sections are missing. The optional "Conventions" section is well-scoped and non-redundant.

---

## Capabilities Inventory

| Capability | Route | Config Header | Progression | Notes |
|---|---|---|---|---|
| Архитектурная консультация | `references/architectural-consultation.md` | Yes | Yes | — |
| Adversarial review | `references/adversarial-review.md` | Yes | Yes | — |
| Compliance & target check | `references/compliance-check.md` | Yes | Yes | — |
| Reuse & landscape discovery | `references/reuse-discovery.md` | Yes | Yes | — |
| Draft architectural artifact | `references/artifact-generation.md` | Yes | Yes | New file (untracked), fully formed |

All five capabilities declared in the SKILL.md routing table have corresponding files under `references/`. File names match routes exactly. No orphaned capability files, no phantom routes.

---

## Key Findings

### MEDIUM — Frontmatter description language mixing

**File:** `SKILL.md:3`  
**What:** The description field opens with a Russian sentence ("Корпоративный архитектор банка, который вписывает инициативы в утверждённую целевую архитектуру.") followed by the English "Use when" trigger phrase. The "Use when" clause is what the harness uses for routing — it is present and correct — but the Russian prefix may cause unpredictable behavior if the harness performs prefix-based matching or truncates the description.  
**Fix:** Consider placing the "Use when" clause first, or separating the Russian display name into the Overview section: `Use when the user asks to talk to Archon, ... | Корпоративный архитектор банка...`

### LOW — Identity section is a role description, not a one-sentence persona prime

**File:** `SKILL.md:16`  
**What:** The Identity section is a dense multi-clause paragraph listing expertise domains (TOGAF, ArchiMate, BIAN, IT4IT, DDD, Cloud-Native, etc.) rather than a concise one-sentence behavioral prime. While the content is accurate and domain-specific, the format is closer to a CV than an identity anchor.  
**Fix:** Consider leading with a single priming sentence — e.g., "You are Archon, a Chief Enterprise Architect with twenty years at the intersection of bank business and systems, thinking in 3–5-year horizons." — followed by the expertise list as supporting context.

### LOW — Communication Style variables not resolved

**File:** `SKILL.md:28`  
**What:** The Communication Style section references `{communication_language}` and `{document_output_language}` inline as behavior instructions ("Использует `{communication_language}` для общения"). These are config tokens resolved at session start (Step 4), not structural template artifacts — so they are intentional and correct. However, they appear in a section that primes the agent's behavior before activation steps run, which means the agent's communication style section contains unresolved tokens at the time the section is first read.  
**Fix:** Minor — this is an architectural pattern choice. If the harness resolves config before applying Identity/Communication Style, no action needed. If sections are applied sequentially before config resolution, consider rephrasing as: "Use the configured communication language for conversation and the configured document language for artifacts."

### LOW — Headless modes declared in Overview but partially documented in capability files

**File:** `SKILL.md:10`, `references/compliance-check.md:108`, `references/artifact-generation.md:315`  
**What:** SKILL.md Step 7 (Routing) documents three headless modes: `--headless:landscape-drift-scan`, `--headless:compliance-audit`, and `--headless:arb-prep`. The `--headless:compliance-audit` behavior is documented in `compliance-check.md` (line 108), and `--headless:arb-prep` in `artifact-generation.md` (line 315). However, `--headless:landscape-drift-scan` has no dedicated documentation in any capability file — its behavior is described inline in SKILL.md Step 7 only.  
**Fix:** Add a "Headless: `--headless:landscape-drift-scan`" subsection to a capability file (likely `architectural-consultation.md` or a dedicated `landscape-drift-scan.md` reference) consistent with the pattern established by the other two headless modes.

---

## Strengths

**Activation sequence design.** The seven-step On Activation sequence is logically ordered and operationally complete: customization resolution precedes prepend hooks, knowledge loading precedes routing, and the Knowledge Disclosure block is correctly placed as a gate before any user-visible output. The degraded-mode handling in each capability file mirrors the same knowledge_status flags established in Step 5, creating a coherent fallback chain across the entire agent.

**Capability file completeness.** Each of the five capability files includes a config header, a "What Success Looks Like" outcome definition, a "Degraded Mode" section tied to specific knowledge_status flags, an "Anti-Patterns" section, "Completion Criteria", and a structured Output Format. This is unusually thorough — the agent has clear definitions of done for every capability, which prevents the agent from producing partial outputs silently.

**Principles are domain-specific and decision-guiding.** All five principles ("Целевая архитектура — красная нить", "Trade-off, а не вердикт", "Reuse прежде новой стройки", "Compliance — не приложение, а вход", "Цитируемость") are concrete, create clear decision frameworks, and directly reflect the agent's operating context. None are generic platitudes.

**Communication Style examples are concrete and persona-consistent.** The three example phrases ("Допустим, идём через ESB — тогда…", "Со стороны Risk это выглядит иначе, потому что…", "Trade-off здесь — TTM против target arch alignment.") are specific to architectural conversations, not generic professional communication. They prime the right cognitive register.

**Capability cross-routing.** Capabilities explicitly reference each other when proceeding to the next logical step (architectural-consultation → adversarial-review, adversarial-review → compliance-check, reuse-discovery → architectural-consultation, artifact-generation → adversarial-review). This creates a coherent workflow graph without over-specifying transitions.

**Greenfield Mode in reuse-discovery.** The explicit Greenfield Mode in `reuse-discovery.md` (lines 34–48) prevents the agent from returning empty tables or hallucinating registry entries when no candidates exist — a common failure mode in discovery capabilities. This is architecturally appropriate.

**No over-specification.** Capability files do not repeat Identity or Communication Style from SKILL.md. The persona is established once and referenced through behavior, not repeated in each capability.

---

## Memory & Headless Status

- **Memory agent:** No (`is_memory_agent: false`). No memory paths declared. Standard stateless agent checks apply — all pass.
- **Headless mode:** Declared and partially documented. Three headless variants are supported. Two have dedicated documentation sections in capability files (`--headless:compliance-audit` in `compliance-check.md`, `--headless:arb-prep` in `artifact-generation.md`). One (`--headless:landscape-drift-scan`) is documented only inline in SKILL.md Step 7. See LOW finding above.

---

## Summary

**4 findings total: 0 critical, 0 high, 1 medium, 3 low.** The agent is structurally ready for use. The medium finding (description language mixing) is the only item worth addressing before production deployment; the three low findings are refinement opportunities that do not affect runtime reliability.
