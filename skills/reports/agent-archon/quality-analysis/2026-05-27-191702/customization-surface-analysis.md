# Customization Surface Analysis — agent-archon

**Date:** 2026-05-27
**Analyzer:** Artisan
**Skill path:** `skills/agent-archon`

---

## Agent Archetype

**Stateless.** Confirmed via `customize.toml`: `agent_type = "stateless"`. This agent activates per-session, loads bank knowledge sources into context on each run, and produces artifacts without persisting state between sessions.

---

## Customization Posture

`customize.toml` is present. The metadata block is populated. The override surface goes well beyond metadata: it exposes five bank knowledge source paths, an output destination, two hook arrays, and a post-assessment command hook. This is a meaningfully opinionated surface for a stateless agent operating against org-specific document corpora.

---

## Metadata Findings

All six required fields are present and consistent:

| Field | customize.toml | SKILL.md frontmatter |
|---|---|---|
| `code` | `archon` | `agent-archon` (frontmatter `name`) |
| `name` | `Archon` | — (not in frontmatter as `name`) |
| `title` | `Корпоративный архитектор банка` | — |
| `icon` | `🏛️` | present in H1 heading |
| `description` | present | present |
| `agent_type` | `stateless` | — |

One note: the SKILL.md frontmatter `name` field holds `agent-archon` (the directory slug), which is the convention for skill identification, not the display name — this is not a conflict. The display name `Archon` lives in customize.toml and in the SKILL.md H1 heading. No drift detected.

`agent_type = "stateless"` is one of the three valid values. No issue.

---

## Opportunity Findings

### [medium-opportunity] No ADR/SD numbering sequence hint in `persistent_facts`

**Location:** `customize.toml` `persistent_facts`, `references/artifact-generation.md`

**Pattern:** The artifact-generation capability produces ADR files named `adr-{NNN}-...`. The sequence counter for `{NNN}` has no grounding source. If the team keeps an ADR index or a numbering registry (a common bank practice), the agent would benefit from knowing it.

**Concrete suggestion:** Add an optional entry to `persistent_facts` pointing at an ADR index or sequence file:
```toml
# persistent_facts = [
#   "file:{project-root}/**/project-context.md",
#   "file:{project-root}/docs/architecture/adr-index.md",  # optional: ADR sequence tracker
# ]
```
Or expose a dedicated scalar:
```toml
adr_index_path = ""   # e.g. "{project-root}/docs/architecture/adr-index.md"; empty = unset
```
**Why it matters:** Without a sequence source, the agent guesses `{NNN}`, which leads to numbering collisions on multi-author teams. This is an org-varying concern that fits cleanly in `customize.toml`.

---

### [medium-opportunity] ARB calendar / governance calendar not exposed

**Location:** `references/artifact-generation.md` ARB Package section, Step 7 headless routing

**Pattern:** The ARB Presentation Package format contains `**Дата ARB:** {YYYY-MM-DD или TBD}`. Many banks run ARB on a fixed cadence (e.g., bi-weekly). If the next ARB date were known, the agent could fill it rather than emit `TBD`. The headless `--headless:arb-prep` mode would also benefit: it could stamp the nearest upcoming session.

**Concrete suggestion:**
```toml
# Next ARB session date (ISO-8601) or leave empty for TBD.
arb_next_session_date = ""
```
Low-cost scalar, zero abuse surface (it is a string, not a toggle).

---

### [low-opportunity] `on_assessment_complete` accepts free-form command/prompt but has no documented examples

**Location:** `customize.toml` lines 77–79

**Pattern:** The field comment says "Например: уведомить ARB, открыть тикет в Jira, обновить дашборд" but gives no concrete shape. Teams integrating with Jira or a webhook will not know what format the agent expects (shell command? natural language prompt? URL?).

**Concrete suggestion:** Add a comment that documents one concrete example and the expected execution semantics:
```toml
# Shell command or prompt string executed after assessment is written.
# Examples:
#   Shell: "curl -s -X POST $WEBHOOK_URL -d @{output_file}"
#   Prompt: "Create a Jira ticket summarising the assessment at {output_file}."
# Leave empty to do nothing.
on_assessment_complete = ""
```
This is a documentation gap, not a structural one — hence low-opportunity.

---

### [low-opportunity] No `arb_template_path` for teams with a custom ARB format

**Location:** `references/artifact-generation.md` — ARB Presentation Package section

**Pattern:** The ARB Package output format is hardcoded in `artifact-generation.md`. Some banks mandate a specific slide/section structure for ARB submissions (often defined in a governance document). Teams with a non-default format must either fork the capability prompt or work around the output.

**Concrete suggestion:**
```toml
# Path to a custom ARB presentation template (Markdown). If set, artifact-generation
# uses this template instead of the built-in structure.
# arb_template_path = ""
```
This is low-priority because the built-in format is reasonable and many teams will use it as-is.

---

## Abuse Findings

### [low-abuse] `compliance_corpus_path` declared but capability prompts use it inconsistently

**Location:** `customize.toml` line 62; `references/compliance-check.md` line 35; `references/adversarial-review.md` line 25

**Pattern:** `compliance_corpus_path` is declared in `customize.toml`. `compliance-check.md` correctly references it as `{agent.compliance_corpus_path}`. However, `adversarial-review.md` names the compliance axis as "152-ФЗ, 115-ФЗ, PCI DSS, GDPR" without loading `{agent.compliance_corpus_path}` — it relies on the agent's built-in knowledge of public laws rather than the org's internal compliance documents. This means an override of `compliance_corpus_path` has no effect on adversarial review, even though that capability performs compliance checks.

This is not a structural abuse of the override surface, but it creates a false expectation: an operator who customizes `compliance_corpus_path` to point at their internal policy corpus will find that adversarial review ignores it. The contract implied by the TOML is not kept by all consumers.

**Concrete suggestion:** In `adversarial-review.md`, add an explicit Degraded Mode note for `compliance_corpus_path` and a load step analogous to the one in `compliance-check.md`, or add a comment in `customize.toml` clarifying that `compliance_corpus_path` is consumed only by the `compliance-check` capability.

---

### [low-abuse] `strategy_path` is declared but its load semantics are asymmetric across capabilities

**Location:** `customize.toml` line 65; SKILL.md Step 5; `references/architectural-consultation.md` line 33; `references/artifact-generation.md` line 186

**Pattern:** SKILL.md Step 5 says `strategy_path` is loaded "by capability prompts on demand." `architectural-consultation.md` references it as `{agent.strategy_path}` only in a "sверь с ... если релевантна" aside. `artifact-generation.md` references it in the Solution Design template under "Контекст и бизнес-цель." `adversarial-review.md` and `reuse-discovery.md` do not reference it at all. The field appears in TOML, suggesting it is a first-class input, but most capabilities treat it as optional background.

This is not broken, but it may surprise an operator who populates `strategy_path` expecting all five capabilities to anchor to it. The inconsistency should either be documented ("strategy_path is advisory and used only when capability prompts explicitly reference it") or the remaining capabilities should receive the reference where meaningful.

---

## Archetype-Fit Assessment

The customization surface is well-matched to the stateless archetype. The agent does not attempt to persist session state or user preferences via TOML, which would be inappropriate for stateless. All overridable knobs are either knowledge-source paths (org-varying by definition) or hook/output configuration (deployment-varying). There are no boolean toggle farms, no arrays-of-tables, and no persona parameters — all high-risk abuse patterns for stateless agents are absent.

The hook surface (`activation_steps_prepend`, `activation_steps_append`, `on_assessment_complete`) is well-designed: these are append-only arrays, so team and user overrides compose rather than clobber. This is the correct shape for a stateless agent that must work across multiple deployment contexts.

The `persistent_facts` default glob (`file:{project-root}/**/project-context.md`) satisfies the BMad convention and is present.

One area where the surface is slightly thin relative to the agent's operational complexity: it exposes five knowledge-source paths, an output path, and a hook, but gives no affordance for the ADR numbering sequence or ARB calendar — both are org-varying operational parameters that the agent currently handles by inference or `TBD` stubs.

---

## Top Insights

**1. The compliance corpus load gap is the highest-priority fix.**
`compliance_corpus_path` is declared as a first-class override but is silently ignored by `adversarial-review.md`, the second most commonly triggered capability. An operator who invests in populating internal compliance documents will not see that investment reflected in adversarial reviews. This is the only finding where a declared TOML knob fails to deliver its implied contract across all consumers.

**2. The ADR sequence and ARB date are the highest-value missing scalars.**
Both are simple strings. Both directly affect artifact quality in the most visible output the agent produces (ADR files and ARB packages). Adding `adr_index_path` and `arb_next_session_date` would eliminate two common `[NEEDS INPUT]` stubs without adding any complexity to the override surface.

**3. `strategy_path` load semantics need a single-sentence clarification.**
The field is declared, loaded on demand, and referenced in only two of five capabilities. This is fine architecturally, but it creates operator confusion. A one-line comment in `customize.toml` — "consumed by architectural-consultation and artifact-generation when explicitly referenced" — would close the expectation gap without any structural change.
