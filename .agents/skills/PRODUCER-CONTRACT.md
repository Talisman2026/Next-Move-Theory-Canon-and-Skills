# Producer contract — seven cross-cutting behaviors (binding for all producer skills)

> Producer skills that bind this contract share seven behaviors that came directly from user testing feedback.
> Specifying them once here keeps the skills in sync. Each skill points to this file and wires the
> concrete hooks (intake questions, template blocks) into its own flow. The companion file
> `READABILITY-CONTRACT.md` governs the 3-layer output; this file governs intake + framing + integrity.

The seven behaviors:

## Codex interactive intake compatibility

This section is a Codex-only override in the `Skills/codex` / installed `.agents/skills` copy. It does not apply to the Claude `Skills/claude` / `.claude/skills` copy.

- Use `request_user_input` only for structured-choice questions.
- The Codex tool contract supports 1-3 short questions per call and 2-3 mutually exclusive options per question. Put the recommended option first and mark it `(Recommended)` when there is a default.
- If one logical question lists more than 3 choices, ask that entire question directly in chat. Do not split one logical question across several `request_user_input` calls. Do not include an explicit Other option.
- Ask free-text prompts directly in chat and wait for the reply. This includes product descriptions, paths, URLs, corrections, custom output paths, multi-select answers, and cases where a custom answer is needed.
- Split listed intake batches only across separate logical questions. Do not drop required intake questions just because they no longer fit in one call.
- If `request_user_input` is unavailable in the current Codex mode, ask the same questions directly in chat and wait for the user. Do not silently skip intake.
- The root Codex agent asks interactive questions before spawning subagents. Subagents should not call root-only interactive tools.
- Treat references to tool names such as Read, WebFetch, WebSearch, Write, and Agent as capability requirements, not literal tool names. Use equivalent available Codex tools. If subagents are unavailable, perform the same work sequentially.

### PRE-INTAKE HARD GATE (Codex execution gate)

This gate is binding for every skill that points to this contract. It is an execution barrier, not a suggestion.

Before **any** substantive analysis, canon loading beyond what is needed to render the orientation, subagent spawn, web research, shell research, report drafting, file write, git commit, or PR action, the root agent must have the following five states resolved:

1. `orientation_emitted = true` — the helicopter-view has been printed in chat.
2. `language_resolved = true` — the document language has been explicitly chosen or explicitly supplied by the user.
3. `mode_resolved = true` — Quick or Deep has been explicitly chosen or explicitly supplied by the user.
4. `format_resolved = true` — Markdown or HTML has been explicitly chosen or explicitly supplied by the user.
5. `output_path_resolved = true` — the default path has been explicitly accepted, or a custom path has been supplied.

If **any** state is unresolved, stop and ask only the missing intake question(s). Do not infer the missing choice from convenience, from the language of the user's message, from a recommended default, or from prior runs unless the user explicitly supplied that choice in the current run.

The **first assistant response** after a producer-skill invocation must therefore begin with the helicopter-view. It may then ask the unresolved intake questions. It must not contain a verdict, segment synthesis, research findings, artifact path, file write, commit, or PR result.

If the user's opening message already supplies one or more of the five choices, mark only those supplied choices resolved; still emit the helicopter-view before doing any substantive work. If all five choices are already supplied, the gate may pass immediately after the helicopter-view.

After the five cross-cutting states are resolved, continue the skill-specific intake. A skill-specific required question can impose an additional gate; this central gate does not authorize skipping it.

**Pre-intake self-check before work starts:**

```text
PRE-INTAKE GATE
orientation_emitted: PASS/FAIL
language_resolved: PASS/FAIL
mode_resolved: PASS/FAIL
format_resolved: PASS/FAIL
output_path_resolved: PASS/FAIL
```

Keep this check in working context; it does not need to be printed to the user unless a gate fails unexpectedly. Any `FAIL` means **no analysis / no write / no commit**.

1. **Helicopter-view first** — orient the user before the first question.
2. **Output format choice** — Markdown (fast) or HTML (easier to read).
3. **Critical treatment of all user input** — everything the user provides is a hypothesis; surface the risks inside it in a dedicated block.
4. **Visible validation debt** — print how many unvalidated assumptions the artifact stands on; `GO` → `GO (to validation)`.
5. **Configurable output path** — default `Skills-Results/…`, but accept the host repo's convention.
6. **Deep-mode QA loop + Evidence Pack fallback** — Deep mode follows the native-web → retry/self-critic → read-only shell-HTTP ladder, must meet an evidence floor, and pauses for an attached external Evidence Pack only when that ladder cannot meet the floor.
7. **Approved semantic Markdown handoff** — after the user explicitly approves the final artifact, offer a copy-ready semantic Markdown handoff for the next skill; never hand off a draft or silently create a stale second artifact.

---

## 1. Helicopter-view first (before the first intake question)

The very first thing the skill prints — before STAGE 0 / the first `request_user_input` — is a short orientation block, in plain language, in the user's chosen document language. From user testing — *"I always ask Claude to first explain schematically what we'll do, then go deep. The skill jumps straight to 'describe your idea.'"*

Keep it to ~8–12 lines:

- **What you'll get** — the one deliverable, in one line.
- **The steps** — 3–6 numbered phases, one line each.
- **Where AI works vs. where you decide / validate** — name explicitly that AI does the analysis; the user picks direction and runs the field validation (interviews, sales, tests). AI cannot validate for them.
- **The two modes** — *Quick (default): no internet, ~3–5 min, reasoning only — good for a first cut, hypotheses, "did I miss something."* · *Deep (opt-in): subagents + web research, longer — real competitor/market/review data.* (From user testing: people didn't know Quick vs Deep existed, or what Quick is for.)
- **Rough cost** — a ballpark of time and token usage so the user can choose a model (Quick: light; Deep: heavy — best on a top model with direct web access).
- **One honest caveat** — *"This speeds up the thinking, not the proving. The numbers and segments are hypotheses until you check them in the field."*

End with: *"Ready? First, a few questions."* → proceed to intake. Don't make the user read a wall — this is a map, not a manual.

## 2. Output format choice — Markdown or HTML

Ask once, in the intake batch (alongside mode). From user testing — *"reading walls of markdown is painful; give me HTML I can open in a browser."*

> **Output format** — **Markdown** (default; faster to generate; opens anywhere) · **HTML** (a bit slower to generate; easier to read — collapsible sections and working in-page navigation).

**Both formats keep every link clickable** — source links (Rule 2) and the Layer-1→2→3 drill-down `▸` links. That's the answer to *"what about the links?"*: HTML doesn't lose them, it makes them nicer (real in-page anchors + sources opening in a new tab).

If the user picks **HTML**, the single output file (Rule 4 — still exactly one analytical result file per run) is a **self-contained `.html`** with the **same** content as the Markdown version — identical attribution, disclaimers, the three layers, every table, every link — rendered as:

- **Inline CSS only, no external dependencies** (opens offline, no network, no build step). A clean reading width (~720px), system font stack, comfortable line-height.
- **Working navigation** — the "How to read this" jump links and every `▸` drill-down link are real in-page `href="#id"` anchors to matching `id="…"` targets. A small sticky top bar with *Level 1 · Level 2 · Level 3* jumps.
- **Collapsible depth** — Layer 3 sections and every `▸ methodology trace` are `<details>` elements (Level 1–2 open by default; Level 3 collapsed) so the reader expands only what they want. This also answers a separate complaint about non-collapsible long content.
- **Source links** open in a new tab (`target="_blank" rel="noopener"`).
- Filename: same pattern as the Markdown file, with the `.html` extension. Do **not** also write the `.md` during the analytical run. The optional post-approval handoff in §7 is a separate export step and is never created before explicit approval.

Build the HTML by rendering the finished content you would have written as Markdown — same layers, same anchors. Don't water it down for HTML; it is the same report in a more readable shell.

## 3. Critical treatment of all user input (everything is a hypothesis)

From user testing: on a project that came in with a deck + landing + codebase, the skill *took the descriptions as truth, baked them into the wedge, and never proposed checking them* — substituting the team's imagined Job for the customer's real one. The canon forbids skipping field validation (Phase II).

**Rule: treat every input the user provides — free-text claims, uploaded decks, landing pages, codebases, past research, "everyone wants X" — as a hypothesis, never as established fact.** This extends the existing user-claims ledger to *all* materials, not just spoken claims.

Two concrete obligations:

**(a) Actively hunt for risks inside the provided material.** Don't just record claims — interrogate them. For each load-bearing input, ask:
- Is this a customer-validated fact, or the team's belief about the customer? (A landing page is the team's hypothesis about value, not proof customers want it.)
- Does the stated Job / segment look like the customer's real Job, or the team's projection? (The most expensive error — seen in real runs.)
- Are there internal contradictions, or numbers presented as data that are actually guesses?
- What would have to be true for this to hold — and is that checked?

**(b) Emit a dedicated, visible block in the output** — *"What you gave me — and the risks I see in it."* This is Ivan's explicit ask: *"when you gave me this information, I see these risks in it,"* marked separately. Place it in **Layer 2** (where trust is built), as its own subsection, and surface the single worst one in **Layer 1** as (or alongside) the make-or-break risk. Format:

```markdown
## What you told me — and the risks I see in it
*Everything below is taken as a hypothesis, not as fact. These are the inputs the analysis leans on, and what I'd check before trusting each.*

| What you provided / claimed | How I treated it | The risk I see in it | How to check it fast |
|---|---|---|---|
| {claim / material, tagged data / observation / hunch} | {used as hypothesis in {where}} | {the specific risk — e.g., "this is your team's stated value, not customer-validated; the real Job may differ"} | {the cheapest falsifying test} |
```

**Hard gate:** no verdict, target-segment pick, wedge, value proposition, or PRD scope may rest *primarily* on an unvalidated user input without the output saying so explicitly and pointing a RAT row at it. If the wedge is built on a Job taken from the user's materials and not confirmed by customer evidence, that is named as the single most expensive risk.

## 4. Visible validation debt + "GO (to validation)"

From user testing — *"A PRD built in 20 minutes on guesses looks as convincing as one after 8 interviews. Print the debt."* And: *"`GO` reads to a founder as 'build it — 3 months'; by your own algorithm GO means 'go validate' — 8 interviews and a fake door."*

Two changes:

**(a) Validation-debt line in Layer 1** — one line, near the top of the answer:

> **Validation debt:** this stands on **{N}** unvalidated assumptions — **{M}** of them fatal (would sink it if wrong). The fatal ones are the first things to check. [see them ▸](#l2-risks)

N = count of risky assumptions in the RAT / risk table. M = those tagged "kills it if wrong." Count honestly; a Quick run on thin input has high debt — say so. This makes a fast artifact legibly fast, not falsely authoritative.

**(b) Verdict wording** — wherever a skill emits a `GO` verdict, write **`GO (to validation)`**, never bare `GO`. Keep `NARROW` and `PIVOT` as-is (they already read as "not yet building"). In Layer 1 add a half-line gloss the first time: *"GO (to validation) — the idea is worth the next step, which is checking it in the field, not building it yet."*

**(c) Hand-off carries the debt.** When a skill hands off to the next in the chain (`$nmt-market-research` / `$nmt-analyze-interviews` → `$nmt-craft-value-proposition` → `$nmt-product-requirements` → `$nmt-craft-go-to-market`), the next skill **opens by asking what from the prior artifact's validation debt has since been checked**, and re-tags anything still unvalidated. Debt travels down the chain; it is not silently dropped.

## 5. Configurable output path

From user testing: hard-coding `Skills-Results/{slug}/…` in the repo root breaks teams whose agent-ready repos keep research elsewhere (e.g., `*/docs/research`).

Add to intake (one line, default is the current behavior — no friction for the common case):

> **Where to save the result** — default `Skills-Results/{project}/{skill}/…` · or give a folder / path convention to match your repo (e.g., `docs/research/`).

If the user gives a path, write the single analytical result file there, keeping the same `{YYYY-MM-DD_HH-MM}_{product-slug}-{skill}-result.{md|html}` filename. If they skip, use the default. Never write more than one analytical result file regardless of location (Rule 4). The optional §7 handoff is generated only after approval and, by default, is emitted in chat rather than written to the repo.

## 6. Deep-mode QA loop + Evidence Pack fallback

Deep research uses the available native Codex web/search/open capabilities first, then safe read-only direct HTTP from the shell when native access is insufficient. MCP and external research connectors are not part of the execution path.

**(a) Internet-access ladder; evidence floor is a hard gate.** Start every Deep-mode research leg with available native/direct Codex web/search/open capabilities and observe the leg's existing caps. A blocked, unauthorized, thin, or below-floor native result is **not** evidence that internet access is unavailable. Treat the **lower bound as a floor**: a leg may return "done" only after it reaches the required minimum of distinct, relevant sources. "Did two queries and stopped" is a failure state, not completion.

**(b) Self-critic and mandatory read-only shell fallback.** After a research leg returns, run a short critic pass asking: *enough distinct sources? load-bearing claims actually verified against a source? any methodology error (segment by demographics, Big-Job-as-segment, features-before-criteria, undersized market)? gaps left?* If it fails, retry with the gap named — up to 2 extra rounds. If native access is blocked, unauthorized, thin, or still below the floor, then, **before requesting an Evidence Pack**, try read-only direct HTTP from the shell: `curl` GET/HEAD, Python `urllib.request`, or an equivalent safe read-only GET/HEAD mechanism available in the environment. Public GET search/index pages may be used for discovery when the environment permits. Do not use POST, PUT, PATCH, or DELETE anywhere in this research fallback.

Do not bypass authentication, paywalls, explicit access restrictions, CAPTCHA, or site blocks. If a source returns 401/403, do not circumvent it; find another public source. Never recommend MCP, Firecrawl, or Exa.

**(c) Evidence Pack fallback only after the ladder fails.** Request an Evidence Pack only when the required floor remains unmet after **native web attempt → retry/self-critic → read-only shell HTTP attempt**:

1. **Stop the external-research portion.** Do not recommend or attempt MCP, Firecrawl, Exa, or another connector. Do not fill missing market facts from model memory, proceed on thin coverage, or publish the artifact as a complete Deep result.
2. **Return a precise evidence-gap request in chat:**
   - the evidence missing from each blocked leg and why it is required;
   - the facts and source types/pages an external research tool must investigate;
   - the required return format shown below.
3. Tell the user they may attach or paste the resulting **Evidence Pack directly into this Codex chat**. Never require it to be saved in GitHub or committed to the repository.
4. When the Evidence Pack arrives, **resume the same Deep workflow at the blocked leg**. Treat as externally verified only claims supported by the pack or by sources Codex successfully verified itself; label every other claim `hypothesis` or `unverified`.
5. If the user does not want to obtain an Evidence Pack, offer an **explicit switch to Quick**. Do not switch modes silently.

Required Evidence Pack format:

```markdown
# Evidence Pack
## {research leg / missing evidence category}
- Claim or fact:
- Value / finding:
- Geography and date / period:
- Source title and publisher:
- Direct URL:
- Publication or access date:
- Relevant excerpt or table row:
- Notes on method / limitations:
```

Attach source files, screenshots, or exports where a direct URL is inaccessible. A URL without the relevant finding is a lead, not evidence.

**(d) Mode-integrity rule.** Never label or present an output as full/complete **Deep** unless every mandatory evidence floor has been reached with sources Codex verified or evidence in the attached Evidence Pack. A paused Deep run is an evidence request, not a deliverable.

**(e) PRE-WRITE / PRE-COMMIT Deep completion gate.** Before first saving a final Deep artifact, updating an existing artifact as complete Deep, or committing a final Deep artifact, the orchestrator must explicitly check and record against the evidence actually collected and stages actually run:

- every mandatory evidence floor — **PASS / FAIL**;
- every research-leg self-critic — **PASS / FAIL**;
- the load-bearing-source audit — **PASS / FAIL**;
- skill-specific methodological invariants — **PASS / FAIL**;
- required output/readability-contract gates — **PASS / FAIL**.

A textual PASS without the supporting evidence or completed stage does not count. If **any** mandatory gate is FAIL: do not label the result full/complete Deep; do not write a new final Deep artifact; do not overwrite an existing Deep artifact as corrected/complete; do not commit the final Deep artifact. Stop that part of the workflow, tell the user the exact failed gate and reason, and continue research/retry or request an Evidence Pack under §6(c). A paused Deep remains an evidence/research request, not a deliverable.

**(f) Quick after an explicit switch.** Quick uses one honest sizing calculation with every assumption named. It never simulates Deep sizing with model-generated inputs, and never states unverified competitor, review, price, market, or regulatory claims as facts. Mark them as hypotheses/unverified and provide a verification path.

## 7. Approved semantic Markdown handoff (post-approval only)

The human-readable result and the machine handoff have different jobs. The result file is for review and discussion; the handoff is a transport snapshot for the next skill. **Never create the handoff from a draft.**

### 7.1 Approval gate

After the result file is delivered, let the user review it, ask questions, and request revisions. Treat the artifact as mutable until the user explicitly approves the current version.

Track these working states:

```text
HANDOFF STATE
artifact_approved: PASS/FAIL
handoff_requested: PASS/FAIL
handoff_fresh: PASS/FAIL
```

- `artifact_approved = PASS` only after an explicit approval of the current result (for example: "approved", "согласовано", "утверждаю", or an equally unambiguous answer to an approval question). Do not infer approval from silence, from the user asking a follow-up question, or merely from moving the discussion forward.
- Before approval, do not generate, write, commit, or present a semantic handoff as final.
- After explicit approval, **offer** the handoff for the normal next skill. Do not force it. If the same user message both approves the artifact and explicitly asks to move it to the next skill, that counts as `handoff_requested = PASS`; no redundant confirmation is needed.
- Any substantive revision after approval immediately resets `artifact_approved = FAIL` and `handoff_fresh = FAIL`. Any previously emitted handoff is stale and must not be reused. Re-approval is required before regenerating it.

Default next-skill routes:

```text
$nmt-market-research      → $nmt-craft-value-proposition
$nmt-analyze-interviews   → $nmt-craft-value-proposition
$nmt-craft-value-proposition → $nmt-product-requirements
$nmt-product-requirements → $nmt-craft-go-to-market
$nmt-craft-go-to-market   → terminal producer step unless the user names another workflow
```

When offering, use plain language, e.g.: *"If this result is approved and you're ready to continue, I can prepare the semantic Markdown handoff for `$nmt-product-requirements`."*

### 7.2 What the handoff contains

The handoff is **not a summary** and not a newly reasoned artifact. It is the final approved semantic content in Markdown, prepared for another skill to consume.

Preserve all load-bearing information from the approved result, including where present:

- selected segment / Jobs / criteria and their priority order;
- user choices and challenge-the-build decisions;
- evidence provenance, respondent/source support, confidence, coherence/saturation, and uncertainty labels;
- validation debt, fatal assumptions, RATs, and what remains unverified;
- selected value direction / offer / build and rejected or excluded alternatives that constrain the next step;
- Critical Chain, Aha Moment, requirements or other skill-specific outputs needed downstream;
- risks, out-of-scope boundaries, forbidden claims/actions, and next-step constraints;
- named unknowns: missing information stays missing.

Do **not** shorten for token savings, collapse evidence classes, promote hypotheses to facts, invent missing values, re-run analysis, or silently "improve" the approved decisions. The purpose is faithful transport, not optimization.

If the approved result is HTML, generate the handoff from the **same final semantic Markdown content/state that the HTML represents**, not by mechanically scraping or stripping tags from HTML. Presentation-only material — CSS, HTML tags, navigation widgets, `<details>` scaffolding, decorative classes — is omitted. If the approved result is already Markdown, reuse its approved semantic content rather than rewriting it.

### 7.3 Copy-ready transport format

Because browser Codex may not accept Markdown/PDF document attachments, the default transport is **chat-first**. Emit the full handoff directly in chat between explicit boundaries so the user can copy it into a new task:

```text
NMT SEMANTIC HANDOFF
source_skill: {skill}
target_skill: {next skill}
source_artifact: {final approved artifact path}
approval_status: APPROVED

--- BEGIN UPSTREAM ARTIFACT ---

{full approved semantic Markdown}

--- END UPSTREAM ARTIFACT ---
```

The boundary markers are part of the transport contract. The downstream task should be able to distinguish its new instructions from the upstream artifact.

By default, **do not create a second repo file** just to transport the handoff. If the user explicitly asks to save the handoff as a `.md` file, treat that as a separate post-approval export action and write exactly the same content that was emitted in chat; do not create a divergent file version. Suggested filename: `{approved-result-basename}-handoff-to-{target-skill}.md`.

### 7.4 Downstream consumption

When a producer skill receives an `NMT SEMANTIC HANDOFF` block, treat the content between the boundaries as an upstream artifact after the PRE-INTAKE gate passes. Do not ask the user to re-describe information that is already present, do not re-derive upstream decisions, and do not inflate confidence. Ask only genuinely missing load-bearing questions plus the mandatory validation-debt question from §4(c): what has been checked since the approved upstream artifact, and what changed.

The handoff is valid only while it matches the last approved upstream state.

---

## How each skill wires this in (integration checklist)

A producer skill satisfies this contract when:

- [ ] It enforces the **PRE-INTAKE HARD GATE** before any substantive work: orientation, language, mode, format, and output path are all resolved.
- [ ] It prints the **helicopter-view** block before the first intake question (§1).
- [ ] Its intake batch includes the **output-format** question (§2) and the **output-path** question (§5).
- [ ] If HTML is chosen, it writes one self-contained `.html` with working anchors + `<details>` (§2).
- [ ] Its template carries the **"What you told me — and the risks I see in it"** block, and its intake/self-critic enforces the input-as-hypothesis gate (§3).
- [ ] Its Layer-1 template carries the **validation-debt line**, and every `GO` is **`GO (to validation)`** (§4).
- [ ] On hand-off, it asks what validation debt has been retired since the prior artifact (§4c).
- [ ] It never creates a final semantic handoff before explicit approval; after approval it offers the normal next-skill handoff, invalidates it after later revisions, and emits the approved semantic Markdown in the copy-ready §7 format.
- [ ] Deep mode follows the **native web → retry/self-critic → read-only shell HTTP** ladder, enforces every evidence floor, runs the evidence-backed PRE-WRITE / PRE-COMMIT gate, and pauses for an attached **Evidence Pack** only when the full ladder cannot reach the floor (§6).
