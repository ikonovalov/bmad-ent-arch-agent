# Agent Cohesion Analysis: agent-archon (Archon)

**Generated:** 2026-05-27
**Agent path:** skills/agent-archon

---

## Assessment

Archon is a tightly conceived, purposeful agent. The persona of a "grey-haired" senior enterprise architect at a bank is authentic and consistently expressed across every capability — the communication style, principles, and output formats all feel like they come from the same person. The five capabilities form a coherent workflow that mirrors how real enterprise architecture work happens: discover what exists, consult on options, design artifacts, stress-test them, then verify compliance. The agent is one of the more cohesive examples of a domain-expert agent: it has a clear constituency (product teams and solution architects), a clear enemy (landscape drift), and a clear output standard (citable, source-anchored findings).

---

## Cohesion Dimensions

### 1. Persona–Capability Alignment

**Score: Strong**

The identity statement (20-year enterprise architect, TOGAF/ArchiMate/BIAN/Cloud-Native fluent, thinks in 3–5 year horizons) maps directly onto what each capability actually does. Architectural consultation requires exactly that strategic horizon. Adversarial review requires the "ARB reviewer with 20 years of experience" framing that the persona provides. Compliance check requires the regulatory knowledge listed. Reuse discovery requires landscape intimacy. Artifact generation requires standards fluency. The communication style — trade-offs over verdicts, mandatory citations, whiteboard voice — is consistently reinforced in every reference file's anti-patterns and approach sections. There are no capabilities that seem orphaned from the persona.

### 2. Capability Completeness

**Score: Strong with one notable gap**

The five capabilities form a nearly complete architectural engagement loop:
- `reuse-discovery` → understand what exists
- `architectural-consultation` → shape the initiative into options
- `artifact-generation` → crystallize the decision
- `adversarial-review` → stress-test it
- `compliance-check` → audit it formally

The loop is well-connected: each capability explicitly hands off to the next appropriate one. The gap is **landscape drift monitoring at the initiative level during consultation**. Archon can scan the entire landscape with `--headless:landscape-drift-scan`, and the adversarial review covers drift for a submitted artifact — but there is no lightweight "drift flag" during an active architectural consultation when the user is mid-decision. A user might receive a recommendation without realizing it creates drift, unless they then separately trigger adversarial review. This is a workflow gap, not a missing capability, and it is partially addressed by the consultation's completion criteria requiring a "Capability mapping (BIAN)" section.

### 3. Redundancy Detection

**Score: Moderate — one overlap worth watching**

There is genuine functional overlap between `architectural-consultation` and `artifact-generation`. Both:
- Ask discovery questions about the initiative
- Anchor to landscape registry, target architecture, and principles
- Produce compliance touchpoints
- Reference reuse candidates

The distinction is tone and output: consultation produces structured advice with trade-off matrices; artifact generation produces a formal document (ADR/SD/ARB package). This distinction is meaningful and the routing logic in SKILL.md is clear ("wants to understand how to fit it in" vs. "needs to create an ADR / Solution Design"). However, in practice a user finishing a consultation will often want to proceed immediately to artifact generation, and the handoff note at the bottom of `architectural-consultation.md` only mentions `adversarial-review` as a next step — it does not mention `artifact-generation`. This creates a silent gap in the user journey.

There is also light overlap between `adversarial-review` and `compliance-check`: adversarial review covers compliance-gap as one of its 8 axes, and compliance-check is its own capability. Both reference files handle this correctly — adversarial review explicitly says "if compliance aspect requires detailed audit, proceed to compliance-check" — so the boundary is clear. Not a problem.

### 4. External Skill Integration

**Score: Strong**

Archon is designed as a standalone domain agent, not a workflow orchestrator, and it handles this correctly. The three headless modes (`--headless:landscape-drift-scan`, `--headless:compliance-audit`, `--headless:arb-prep`) suggest clean integration points for external automation pipelines or CI/CD hooks. The `{agent.on_assessment_complete}` hook pattern throughout all capabilities provides a clean post-processing integration point. There are no referenced external BMad skills, which is appropriate: an enterprise architect agent should be self-contained rather than delegating to generic agents.

### 5. Capability Granularity

**Score: Strong**

Five capabilities is the right number. Each operates at a meaningfully different level of abstraction and serves a distinct user intent. None could reasonably be merged without losing important surface area, and none are so granular they should be split. The granularity within each capability is also appropriate: `compliance-check` is deliberately narrow (auditor mode, no trade-offs) while `architectural-consultation` is deliberately wide (strategist mode, full trade-off space). These two extremes existing at the same level in the menu is correct because they serve different user needs.

### 6. User Journey Coherence

**Score: Strong with one routing gap**

A user can accomplish meaningful work end-to-end. Common journeys are:
- **Greenfield:** reuse-discovery → architectural-consultation → artifact-generation → adversarial-review ✓
- **Brownfield review:** adversarial-review → compliance-check (if needed) ✓
- **ARB prep:** artifact-generation (headless arb-prep) ✓
- **Periodic governance:** headless landscape-drift-scan + compliance-audit ✓

The routing gap is in `architectural-consultation`'s completion criteria: it says "if user wants immediate stress-test, proceed to adversarial-review" but does not say "if user wants to formalize this into an ADR, proceed to artifact-generation." This is the most natural next step after a successful consultation and it is missing from the handoff text.

---

## Per-Capability Cohesion

**architectural-consultation** — Fits the persona perfectly. The "think out loud, don't dispense verdicts" approach matches the whiteboard-voice communication style. The minimum-two-options rule enforces the persona's trade-off principle. The output format with BIAN capability mapping is exactly what a BIAN-fluent architect would produce. Cohesion: high.

**adversarial-review** — The 8-axis review structure is admirably thorough and aligns with the persona's 20-year ARB experience. The "adversarial does not mean toxic" framing mirrors the persona's "correct but direct" communication style. The citability test ("if you can't cite a source, it's not a finding — it's an opinion") is a direct expression of the persona's citability principle. Cohesion: high.

**compliance-check** — Mode-switches the persona from strategist to auditor, and this is explicitly called out in the capability ("клинический" / clinical tone). This is a deliberate and appropriate shift that the persona can credibly make given its stated regulatory knowledge (152-ФЗ, 115-ФЗ, PCI DSS, GDPR). The per-row checklist format is the right output for an audit function. Cohesion: high.

**reuse-discovery** — The "reuse before new build" principle is listed as a core persona principle, so having a dedicated capability for it is structurally sound. The Greenfield Mode handling is particularly strong: rather than returning empty tables, it pivots to BIAN reference patterns and a first registry-entry draft, which is exactly what an experienced architect would do. Cohesion: high.

**artifact-generation** — Supports all three key artifact types (ADR, Solution Design, ARB Package). The discovery dialog approach ("don't start with a template") aligns with the persona's consultation style. The `[NEEDS INPUT: ...]` placeholder pattern instead of blank sections is a thoughtful UX decision that fits the persona's citability principle — it makes missing context explicit rather than hiding it. Cohesion: high.

---

## Key Findings

### Medium Severity

**Missing handoff: architectural-consultation → artifact-generation**
The `architectural-consultation` capability's completion criteria and final paragraph only suggest proceeding to `adversarial-review`. It does not mention `artifact-generation` as a natural next step after options are decided. In practice, the most common post-consultation action is to formalize the chosen option into an ADR or Solution Design. A user who has just completed a consultation may not realize this is available without re-reading the SKILL.md capability table.
*Recommendation:* Add to `architectural-consultation.md` completion criteria: "If the user wants to formalize the chosen option into an ADR or Solution Design, proceed to `artifact-generation`."

**Drift detection is reactive, not proactive during consultation**
The consultation capability produces a "Capability mapping (BIAN)" section that covers alignment — but this section appears at output time, not during the discovery dialog. A user might go through several rounds of option discussion without the agent flagging drift until the final recommendation. During the approach phase, there is no explicit step that says "check target_architecture for each option before presenting it."
*Recommendation:* Add an explicit sub-step in the Approach section of `architectural-consultation.md` to check each option against `{agent.target_architecture_path}` before finalizing the trade-off matrix, and surface drift findings inline in the trade-off row rather than deferring to the output format.

### Low Severity

**Headless landscape-drift-scan behavior is underspecified relative to compliance-audit**
The `compliance-check.md` reference file includes a detailed `## Headless: --headless:compliance-audit` section with specific output path and scope guidance. By contrast, `--headless:landscape-drift-scan` is only described in SKILL.md's routing block with a one-line description. If a team wants to implement scheduled drift scanning, there is no reference file to consult for output format, scope configuration, or prioritization logic.
*Recommendation:* Either add a `## Headless: --headless:landscape-drift-scan` section to an appropriate reference file (a new `references/landscape-drift.md` or appended to `architectural-consultation.md`), or add headless guidance directly to SKILL.md in more detail.

**No explicit capability for change impact analysis**
Archon has strong capabilities for new initiatives but no dedicated path for "I'm changing an existing system — what breaks?" This is a common enterprise architecture need: deprecating a service, upgrading a core banking module, migrating from ESB to Event Hub. Currently this would fall under `adversarial-review` (if there is a proposal document) or `architectural-consultation` (if not), but neither is optimized for impact analysis across the landscape registry.
*Recommendation:* Consider whether this is in scope. If the agent should serve teams making changes to existing systems, a dedicated capability or a named variant of `reuse-discovery` (e.g., a "dependency impact" mode) would improve routing clarity.

### Suggestion

**The activation greeting could surface available capabilities more actionably**
SKILL.md Step 7 says to "briefly greet the user (one-two sentences in character) and offer capabilities." Most persona agents list capabilities as a menu. Given that Archon has five distinct capabilities with meaningfully different routing conditions, the greeting could be more useful if it asked one qualifying question ("What brings you to architecture today — a new initiative, an artifact ready for review, or something else?") rather than listing all five. This would match the whiteboard-voice persona and reduce the user's cognitive load.

**Consider a light "state summary" at the start of each capability**
When Archon loads a capability mid-session (e.g., user moves from consultation to adversarial review), the capability prompt begins with the task description without recapping what was just learned. An explicit "Bringing into this review: [what we learned in consultation]" step would make the workflow feel more continuous and reduce repeated discovery questions. This matters most in the consultation → artifact-generation → adversarial-review chain.

---

## Strengths

The agent's **citability principle** is its single most valuable differentiator. Most agents hedge; Archon forces every finding to cite a source or be labeled as opinion. This is expressed consistently in every capability's anti-patterns and completion criteria — it is not just stated in identity, it is operationalized throughout.

The **degraded mode handling** is unusually thorough. Every capability specifies exactly what to do when each knowledge file is missing, rather than failing silently or hallucinating from general knowledge. The Knowledge Disclosure block that surfaces missing files at the start of each response is a strong trust-building mechanism.

The **Greenfield Mode** in `reuse-discovery` is a genuine quality-of-life improvement. Rather than returning empty tables, it pivots to BIAN reference patterns and a first registry-entry draft. This shows that the capability was designed for real usage scenarios, not just the happy path.

The **headless mode** support for three distinct automated scenarios (drift scan, compliance audit, ARB prep) shows forward-thinking design for governance automation use cases, not just interactive sessions.

The **inter-capability routing** is well-constructed. Each capability explicitly names one or two natural next capabilities with clear routing conditions, creating a navigable workflow graph rather than isolated functions.
