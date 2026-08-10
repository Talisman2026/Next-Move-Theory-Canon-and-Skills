# Producer contract — six cross-cutting behaviors (binding for all producer skills)

> The four producer skills (`nmt-market-research`, `nmt-craft-value-proposition`, `nmt-product-requirements`,
> `nmt-craft-go-to-market`) share six behaviors that came directly from user testing feedback.
> Specifying them once here keeps the four skills in sync. Each skill points to this file and wires the
> concrete hooks (intake questions, template blocks) into its own flow. The companion file
> `READABILITY-CONTRACT.md` governs the 3-layer output; this file governs intake + framing + integrity.

The six behaviors:

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

1. **Helicopter-view first** — orient the user before the first question.
2. **Output format choice** — Markdown (fast) or HTML (easier to read).
3. **Critical treatment of all user input** — everything the user provides is a hypothesis; surface the risks inside it in a dedicated block.
4. **Visible validation debt** — print how many unvalidated assumptions the artifact stands on; `GO` → `GO (to validation)`.
5. **Configurable output path** — default `Skills-Results/…`, but accept the host repo's convention.
6. **Deep-mode QA loop + Evidence Pack fallback** — Deep mode starts with direct Codex web access, must meet an evidence floor, and pauses for an attached external Evidence Pack when that floor cannot be met.

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

If the user picks **HTML**, the single output file (Rule 4 — still exactly one file per run) is a **self-contained `.html`** with the **same** content as the Markdown version — identical attribution, disclaimers, the three layers, every table, every link — rendered as:

- **Inline CSS only, no external dependencies** (opens offline, no network, no build step). A clean reading width (~720px), system font stack, comfortable line-height.
- **Working navigation** — the "How to read this" jump links and every `▸` drill-down link are real in-page `href="#id"` anchors to matching `id="…"` targets. A small sticky top bar with *Level 1 · Level 2 · Level 3* jumps.
- **Collapsible depth** — Layer 3 sections and every `▸ methodology trace` are `<details>` elements (Level 1–2 open by default; Level 3 collapsed) so the reader expands only what they want. This also answers a separate complaint about non-collapsible long content.
- **Source links** open in a new tab (`target="_blank" rel="noopener"`).
- Filename: same pattern as the Markdown file, with the `.html` extension. Do **not** also write the `.md` — one file per run.

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

**(c) Hand-off carries the debt.** When a skill hands off to the next in the chain (`$nmt-market-research` → `$nmt-craft-value-proposition` → `$nmt-product-requirements` → `$nmt-craft-go-to-market`), the next skill **opens by asking what from the prior artifact's validation debt has since been checked**, and re-tags anything still unvalidated. Debt travels down the chain; it is not silently dropped.

## 5. Configurable output path

From user testing: hard-coding `Skills-Results/{slug}/…` in the repo root breaks teams whose agent-ready repos keep research elsewhere (e.g., `*/docs/research`).

Add to intake (one line, default is the current behavior — no friction for the common case):

> **Where to save the result** — default `Skills-Results/{project}/{skill}/…` · or give a folder / path convention to match your repo (e.g., `docs/research/`).

If the user gives a path, write the single result file there, keeping the same `{YYYY-MM-DD_HH-MM}_{product-slug}-{skill}-result.{md|html}` filename. If they skip, use the default. Never write more than one file regardless of location (Rule 4).

## 6. Deep-mode QA loop + Evidence Pack fallback

Deep research in this Codex installation uses **direct Codex web access only**. MCP is not part of the execution path. The normal Deep workflow is unchanged whenever direct web research reaches the skill's evidence floor.

**(a) Direct web first; evidence floor is a hard gate.** Start every Deep-mode research leg with Codex's direct web tools and observe the leg's existing web caps. Treat the **lower bound as a floor**: a leg may return "done" only after it reaches the required minimum of distinct, relevant sources. "Did two queries and stopped" is a failure state, not completion. If the floor is reached, continue the ordinary Deep workflow.

**(b) Self-critic loop on each leg.** After a research leg returns, run a short critic pass asking: *enough distinct sources? load-bearing claims actually verified against a source? any methodology error (segment by demographics, Big-Job-as-segment, features-before-criteria, undersized market)? gaps left?* If it fails, re-run the direct-web leg with the gap named — up to 2 extra rounds. Don't ship a leg that failed its own critic.

**(c) Evidence Pack fallback when the floor cannot be reached.** If direct web access is fully or partly blocked and the required evidence floor remains unmet after the retry loop:

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

**(e) Quick after an explicit switch.** Quick uses one honest sizing calculation with every assumption named. It never simulates Deep's three-method sizing with model-generated inputs, and never states unverified competitor, review, price, market, or regulatory claims as facts. Mark them as hypotheses/unverified and provide a verification path.

---

## How each skill wires this in (integration checklist)

A producer skill satisfies this contract when:

- [ ] It prints the **helicopter-view** block before the first intake question (§1).
- [ ] Its intake batch includes the **output-format** question (§2) and the **output-path** question (§5).
- [ ] If HTML is chosen, it writes one self-contained `.html` with working anchors + `<details>` (§2).
- [ ] Its template carries the **"What you told me — and the risks I see in it"** block, and its intake/self-critic enforces the input-as-hypothesis gate (§3).
- [ ] Its Layer-1 template carries the **validation-debt line**, and every `GO` is **`GO (to validation)`** (§4).
- [ ] On hand-off, it asks what validation debt has been retired since the prior artifact (§4c).
- [ ] Deep mode uses **direct web first**, enforces the **evidence floor + self-critic loop**, and pauses for an attached **Evidence Pack** when the floor cannot be reached (§6).
