# Enhancement Opportunities Analysis — Agent Archon

**Analyst:** DreamBot  
**Date:** 2026-05-27  
**Agent path:** skills/agent-archon

---

## Agent Understanding

Archon is a senior enterprise architect persona for a Russian bank, designed to help product teams and architects align new initiatives with the bank's approved target architecture, regulatory requirements (152-FZ, 115-FZ, PCI DSS, GDPR), and an internal systems registry. The agent operates across five capability modes — consultation, adversarial review, compliance audit, reuse discovery, and artifact generation — and supports three headless invocation patterns for periodic automation. The primary user is assumed to be a bank architect or senior product manager who already understands architectural vocabulary and brings reasonably well-formed inputs.

---

## User Journeys

### 1. The First-Timer (Junior Developer Asked to "Check Architecture")

**Narrative:** A developer has been handed an ADR their team lead wrote. Their manager said "run it by Archon." They paste the document in and say "can you review this?"

**Friction points:**
- The agent offers five capability choices at startup. Without knowing the vocabulary ("adversarial review," "reuse discovery"), the user stalls at the menu. Nothing tells them "paste your document and say 'review it'" is sufficient.
- If they accidentally land in architectural-consultation instead of adversarial-review, they will get a different (less useful) output — and the routing is entirely intent-based with no confirmation.
- The greeting is one or two sentences "in character" — a whiteboard-architect tone that assumes peer-level understanding. A junior user may not feel invited to ask for explanation.

**Bright spots:**
- Degraded mode is genuinely thoughtful: missing knowledge files produce labeled gaps rather than hallucinated citations, which means even a zero-knowledge-file run produces honest output.

### 2. The Expert (Principal Architect Who Knows Exactly What They Want)

**Narrative:** The lead architect needs to generate an ARB package from an existing Solution Design before Thursday's board. They know the format. They want it done fast.

**Friction points:**
- The artifact-generation capability starts with a five-question discovery dialog, which the expert has already answered mentally. The only escape hatch is "если пользователь торопится" — an implicit affordance that requires them to know it exists and proactively push through it.
- The headless path (`--headless:arb-prep {path}`) is the right shortcut, but it is buried deep in artifact-generation.md. There is no quick-reference or summary of headless flags in SKILL.md's main routing section — only two sentences in the overview paragraph.
- Capability routing is described in a table in SKILL.md but contains no examples of natural-language triggers, so the expert must still phrase their request correctly.

**Bright spots:**
- The `--headless:arb-prep` flow is genuinely powerful: read artifact, read three knowledge sources, produce a complete ARB package. This is the expert's ideal path once discovered.
- The handoff chain (adversarial-review → compliance-check → save) maps cleanly to how experts actually sequence their work.

### 3. The Confused User (Invoked by Accident or Wrong Intent)

**Narrative:** A product manager heard "Archon" mentioned and opens it to ask about a vendor selection decision. They write: "We are choosing between Vendor A and Vendor B for our new loyalty platform."

**Friction points:**
- There is no explicit out-of-scope handling. Archon will try to absorb this into architectural-consultation because the input resembles an initiative — and may produce a plausible but misaligned output that reinforces the wrong framing (vendor selection vs. architecture decision).
- The Capture-Don't-Interrupt pattern is absent: if the user raises a question that is genuinely out of scope (e.g., "can you also write a business case?"), nothing tells the agent how to capture that without derailing.
- The persona's directness ("никаких 'просто сделай так'") may come across as dismissive to a user who does not share the vocabulary.

**Bright spots:**
- The capability routing table with menu-codes (AC, AR, CC, RD, AG) could be surfaced to the user as a quick self-selection tool, reducing mismatch.

### 4. The Edge-Case User (Multi-Initiative Input)

**Narrative:** An architect submits a 40-page Solution Design that contains three nested sub-decisions, each of which could be its own ADR. The document also includes a vendor contract appendix and a project plan.

**Friction points:**
- None of the capability prompts define a scope-limiting step. Adversarial-review will attempt all eight axes on the entire document, potentially producing an unwieldy report where the signal is buried in volume.
- The artifact-generation discovery dialog asks "what systems are involved" once, but a multi-initiative SD may involve fifteen systems — there is no guidance on how to handle partial coverage or progressive refinement.
- The output format for adversarial-review is a flat severity-sorted list. With a complex document, there is no provision for grouping findings by sub-decision or component, making it hard to assign ownership.

**Bright spots:**
- The eight-axis structure of adversarial-review gives natural grouping potential — findings could be grouped by axis AND sorted by severity within axis, which is not currently specified.

### 5. The Hostile Environment (Missing Files, Limited Context)

**Narrative:** The agent is installed in a new team's project. None of the bank knowledge files exist yet. The first user runs architectural-consultation.

**Friction points:**
- The Knowledge Disclosure block is shown, but the user receives no guidance on *how to remedy* the missing files — no reference to which config keys to set, no pointer to a setup guide, no offer to run in "general knowledge mode" explicitly acknowledged as limited.
- Reuse-discovery goes further with a helpful explicit refusal ("Реестр систем не загружен — reuse discovery недоступна") but then offers to fall back to architectural-consultation. That fallback also requires landscape_registry for its reuse-map step — so the user may bounce between two degraded modes.
- The activation Step 1 (resolve_customization.py) will silently fall back to manual TOML parsing if the script is missing. There is no user-facing message about which path was taken, making debugging slow.

**Bright spots:**
- Every capability has a Degraded Mode section, which is far better than most agents. The pattern is consistent.
- Greenfield Mode in reuse-discovery is a genuinely clever handling of "zero results" that avoids the dead-end of three empty tables.

### 6. The Automator (CI Pipeline or Scheduled Agent)

**Narrative:** A governance team sets up a nightly job: `--headless:compliance-audit` runs against the full registry. The output feeds a dashboard.

**Friction points:**
- The three headless flags are described, but there is no specification of exit codes, structured output format (JSON/YAML vs. Markdown), or machine-readable severity summaries. A CI pipeline consuming Markdown must parse natural language to determine if the audit found blockers.
- `--headless:compliance-audit` produces a file in `{agent.assessment_output_path}` and optionally runs `{agent.on_assessment_complete}`. There is no specification of what happens if the output path is not writable, or if `on_assessment_complete` fails — should the agent exit non-zero? Retry?
- The headless landscape-drift-scan has no schema for its report format. Two runs on two different days may produce differently structured output, making automated diffing fragile.
- There is no concept of a "headless dry-run" that returns findings without writing files, which is useful for pipeline testing.

**Bright spots:**
- The three headless modes cover the most valuable automation scenarios (drift detection, compliance audit, ARB prep). This is unusually forward-thinking.
- `{agent.on_assessment_complete}` as a configurable hook is the right extension point for CI integration.

---

## Headless Assessment

**Level: Easily Adaptable** (not yet headless-ready without additions)

The three defined headless modes show clear intent, but several gaps prevent clean CI/CD integration:

| Interaction point | Auto-resolvable? | What headless invocation needs |
|---|---|---|
| Capability selection | Yes — flag already selects mode | Nothing |
| Knowledge Disclosure | Yes — print to report header | Already specified |
| Discovery dialog questions | Partially — `--headless:arb-prep {path}` resolves this for artifact generation | Consultation and review modes have no headless equivalent |
| Output path | Yes — configurable via `assessment_output_path` | Needs writable-path error handling |
| `on_assessment_complete` hook | Yes — already configurable | Needs failure handling spec |
| Structured severity summary | No — output is Markdown prose | Needs a machine-readable format option (e.g., `--output-format json`) |
| Exit signal for blockers found | No — not specified | Needs exit-code contract |

---

## Key Findings

### [HIGH OPPORTUNITY] Missing first-use onboarding path

**Area:** On Activation / Step 7 routing  
**What was noticed:** When all three knowledge files are missing (common for a fresh install), the agent shows the Knowledge Disclosure block and then proceeds to offer capabilities. There is no "setup mode" that walks the user through providing the paths and populating the basic config. A new user with zero files gets a functional-looking interface that will produce degraded-quality output for every request without any clear path forward.  
**Concrete suggestion:** Add a conditional branch in Step 7: if all three key files are missing AND the session is interactive, offer a brief "let's get you set up" path that prompts for `target_architecture_path`, `architecture_principles_path`, and `landscape_registry_path`, explains what each file should contain, and offers to generate a minimal registry template. This is one paragraph of routing logic that could transform the first-use experience from "broken but silent" to genuinely welcoming.

---

### [HIGH OPPORTUNITY] No structured output contract for headless consumers

**Area:** Headless modes (all three)  
**What was noticed:** All headless output is Markdown. The automator archetype — a CI pipeline, a governance dashboard, another agent — cannot reliably parse severity counts, drift counts, or compliance statuses from prose Markdown without brittle regex. The `--headless:compliance-audit` mode produces a file, but the format of its summary section is not normalized between runs.  
**Concrete suggestion:** Define a YAML front-matter block at the top of every headless-generated report containing: `run_date`, `mode`, `knowledge_status`, and a `summary` object with named counts (`blockers`, `gaps`, `drift_items`, `compliant`, `needs_clarification`). This adds ten lines to the output spec and makes all three headless modes trivially consumable by downstream automation. Additionally, specify that the agent should signal failure (non-zero exit or error output) when any `blocker`-severity finding is present, enabling CI gates.

---

### [HIGH OPPORTUNITY] Capability routing has no confirmation step for ambiguous input

**Area:** Step 7 routing  
**What was noticed:** The routing from greeting to capability is purely intent-based. If a user says "I need to check my design," the agent must infer whether they want adversarial-review or compliance-check or architectural-consultation. There is no soft confirmation ("It sounds like you want an adversarial review of your design — shall I proceed with that?") before loading the capability prompt and committing to a mode. In practice this means the confused-user archetype will receive a full output in the wrong mode before realizing the mismatch.  
**Concrete suggestion:** Add a single-sentence routing confirmation before loading any capability prompt: state the inferred capability and menu-code, and ask for a brief confirmation or correction. This is a soft gate, not a blocker — "Sounds like Adversarial Review (AR). Confirm or pick another: AC / AR / CC / RD / AG." This adds at most one conversational turn and prevents entire wasted outputs.

---

### [MEDIUM OPPORTUNITY] No scope-limiting step for large or multi-initiative inputs

**Area:** adversarial-review, artifact-generation  
**What was noticed:** Neither capability asks the user to define the scope boundary before beginning. For a large Solution Design covering multiple domains, the adversarial-review will attempt all eight axes across the entire document, producing a report that may have thirty findings with no grouping by component or sub-decision. There is also no guidance on how to split a large input into multiple focused reviews.  
**Concrete suggestion:** Add a scope-definition step at the start of adversarial-review and artifact-generation: "What is the primary decision or component you want me to focus on? (I can review the full document, or focus on a specific section, integration pattern, or data domain.)" For artifact-generation, add: "Does this cover one decision or multiple? If multiple, I'll draft each as a separate ADR and reference them from a parent Solution Design." This prevents volume-driven signal loss.

---

### [MEDIUM OPPORTUNITY] Trade-off matrix in consultation lacks quantitative scaffolding

**Area:** architectural-consultation output format  
**What was noticed:** The trade-off axes (TTM, capex/opex, operational risk, compliance, target arch alignment, downstream impact) are listed but have no scoring guidance or scale. Different users will fill this matrix very differently — one architect writes "medium TTM" while another writes "three sprints." The output becomes incomparable across sessions and hard to present to a business stakeholder.  
**Concrete suggestion:** Add a light scoring convention to the consultation output spec: for each axis, use a three-point scale (low/medium/high for risk axes, short/medium/long for TTM, rough order of magnitude for cost). The scale does not need to be precise — it needs to be consistent. This also makes the matrix directly useful when the consultation output is handed to artifact-generation as input context.

---

### [MEDIUM OPPORTUNITY] Handoff chain between capabilities lacks state preservation

**Area:** Cross-capability transitions (e.g., consultation → adversarial-review → compliance-check)  
**What was noticed:** Each capability prompt says "proceed to X" at its completion criteria, but there is no specification of what context is carried forward. If the consultation produced a recommended option and the user says "now review it adversarially," the adversarial-review starts fresh from its own approach section. The artifact constructed in consultation — the option description, the trade-off matrix, the reuse candidates — is not explicitly handed to the next capability as structured input.  
**Concrete suggestion:** Define a lightweight "session context object" (informal, not a schema) that capabilities write to and read from: current initiative name/slug, identified systems, compliance flags raised so far, options generated. Each capability's opening step should check for this context and skip re-asking questions already answered. This transforms multi-capability sessions from a series of cold starts into a coherent workflow.

---

### [MEDIUM OPPORTUNITY] Persona hardness may create exit friction for confused users

**Area:** Communication style / On Activation greeting  
**What was noticed:** The persona is explicitly directive and high-confidence. The anti-patterns in adversarial-review explicitly forbid praise ("Хорошее решение, но..."). This is correct for the target expert user, but for a confused user or first-timer, the absence of any warmth or guidance signal may cause silent abandonment. There is no "not sure where to start?" affordance.  
**Concrete suggestion:** Add a single fallback phrase to the Step 7 greeting: if the user's opening message is a question about Archon's capabilities rather than an architectural input, respond with a brief plain-language description of what Archon can do and what to bring. "If you're not sure where to start, tell me what initiative you're working on or paste your document — I'll figure out the right approach." This does not soften the persona; it just prevents the door from being invisible.

---

### [LOW OPPORTUNITY] Greenfield Mode could offer a registry population workflow

**Area:** reuse-discovery / Greenfield Mode  
**What was noticed:** Greenfield Mode correctly avoids the empty-table dead end and offers a draft registry entry. But once the user has that draft entry, there is no path to actually add it to the registry file. The capability ends with "proceed to architectural-consultation" — useful, but the registry entry draft is left floating.  
**Concrete suggestion:** After providing the draft registry entry in Greenfield Mode, offer: "Would you like me to append this entry to your landscape registry file now? I can write it to `{agent.landscape_registry_path}` directly, or output it for your review first." This closes the loop and incrementally improves the knowledge base that makes future sessions more valuable.

---

### [LOW OPPORTUNITY] No version or revision tracking on generated artifacts

**Area:** artifact-generation output format  
**What was noticed:** ADR and Solution Design files include a Version field, but the artifact-generation capability does not update or track versions when a user returns to revise a previously generated document. If a user runs artifact-generation twice on the same initiative, they get two files with identical slugs and different dates — no diff, no change log, no supersession link.  
**Concrete suggestion:** Add a revision convention to the approach: "If a previous version of this artifact already exists in `{agent.assessment_output_path}`, I'll increment the version number and add a brief change summary at the top." This makes the output usable as a living document rather than a series of snapshots.

---

## Top Insights

**1. The missing knowledge files experience is the agent's most likely failure mode for new deployments.**  
The entire agent is anchored on three knowledge files that almost certainly do not exist when a team first installs it. The degraded modes are well-designed but not surfaced prominently enough to new users. A guided setup path — even just three prompts — would transform cold-start from "quietly broken" to actively useful. This is the single change with the highest ratio of impact to implementation effort.

**2. The headless modes are forward-thinking but not yet automation-grade.**  
The three headless flags represent a genuine architectural insight: compliance audits and drift detection should run on a schedule, not on demand. But the output contracts — Markdown only, no exit codes, no structured summaries — mean the automation value cannot be realized without additional glue. Adding a YAML front-matter summary block and a failure-exit contract would make these modes genuinely CI/CD-ready without changing any of the analytical logic.

**3. The agent is built for single-session, single-capability use. Multi-session and multi-capability workflows are the real world.**  
A real architect rarely uses just one capability in one session. They consult, then review adversarially, then generate an ARB package, then get a compliance check — across days or weeks, possibly with different team members involved at each step. The agent has no memory of what was decided in the consultation when the adversarial review starts. Building even a lightweight session context object — a few named fields carried forward — would make the agent feel like a continuous working relationship rather than a series of independent tool invocations.

---

## Facilitative Patterns Check

| Pattern | Status | Notes |
|---|---|---|
| Soft Gate Elicitation | Absent | No "anything else or shall we move on?" prompts between sections; completion criteria are declared but not conversationally confirmed |
| Intent-Before-Ingestion | Partially present | artifact-generation has discovery dialog; adversarial-review and compliance-check go straight to document analysis without asking why the user is bringing this artifact now |
| Capture-Don't-Interrupt | Absent | No mechanism for capturing out-of-scope requests without derailing the current capability flow |
| Dual-Output (LLM-optimized distillate) | Absent | All outputs are human-readable Markdown; no machine-readable companion output for downstream agents or tools |
| Parallel Review Lenses | Present | adversarial-review's eight-axis structure is an explicit parallel-lenses architecture; this is a strength |
| Three-Mode Architecture (Guided/Yolo/Autonomous) | Partially present | Headless flags approximate Autonomous mode; interactive sessions approximate Guided mode; there is no explicit "Yolo" fast-path for expert users who want to skip all confirmations |
| Graceful Degradation | Present and strong | Every capability has a Degraded Mode section; missing files produce labeled gaps rather than failures |

**Patterns that would add most value if added:**

1. **Intent-Before-Ingestion** in adversarial-review: asking "what outcome do you need from this review — ARB clearance, pre-implementation sanity check, or post-incident analysis?" changes what the review emphasizes without changing its structure.

2. **Capture-Don't-Interrupt**: a single instruction to note and acknowledge out-of-scope inputs (e.g., "I'll note that business case question — that's outside my scope but worth flagging to your PM") would prevent the confused-user persona from feeling dismissed or lost.

3. **Dual-Output**: adding a brief structured summary block (YAML front-matter with key counts and statuses) to all capability outputs would serve both human readers and the automator archetype without any loss of quality for the primary audience.
