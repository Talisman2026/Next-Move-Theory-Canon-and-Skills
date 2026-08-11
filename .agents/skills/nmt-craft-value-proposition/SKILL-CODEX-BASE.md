---
name: nmt-craft-value-proposition
description: >-
  Generate the strongest possible Value Proposition for a chosen segment using
  Ivan Zamesin's AJTBD / Next Move Theory methodology. Supports three first-class
  inputs: a $nmt-market-research result, a $nmt-analyze-interviews result, or a
  manual segment+Jobs description. Two modes — Quick (default) and Deep.
  Defaults to English.
user-invocable: true
---

# Craft Value Proposition v2 — Codex entrypoint

> **Binding base workflow:** after the pre-intake gate below passes, read `./SKILL-BASE.md` and execute its complete S0→S6 workflow. `SKILL-BASE.md` contains the full methodology, generation/filtering algorithm, output template, Deep-mode research rules, and QA gates. The rules in this entrypoint are a Codex execution override: if they conflict with `SKILL-BASE.md`, **this file wins**.

## 0. PRE-INTAKE HARD GATE — run before anything else

The first assistant response after `$nmt-craft-value-proposition` is invoked must begin with the helicopter-view from `SKILL-BASE.md` S0. Before any skill-specific question, upstream-file read, canon loading beyond what is necessary to orient the user, analysis, research, file write, git commit, or PR action, resolve all five states:

1. `orientation_emitted = true` — helicopter-view printed in chat.
2. `language_resolved = true` — language explicitly chosen/supplied in this run.
3. `mode_resolved = true` — Quick or Deep explicitly chosen/supplied.
4. `format_resolved = true` — Markdown or HTML explicitly chosen/supplied.
5. `output_path_resolved = true` — default path explicitly accepted or custom path supplied.

Do **not** infer language from the user's message or silently apply a default. If any state is unresolved, ask only the missing cross-cutting controls and stop. **Do not ask intake depth, input route, product questions, materials questions, or hand-off-debt questions until all five states PASS.**

After orientation, the compact control prompt is:

- **Language** — English / user's language.
- **Mode** — Quick / Deep.
- **Format** — Markdown / HTML.
- **Save path** — accept default `Skills-Results/{project}/craft-value-proposition/…` / custom path.

If some values were already explicitly supplied, ask only the missing ones. Once all five PASS, continue with intake depth and routing from the base workflow.

## 1. `$nmt-analyze-interviews` is a first-class upstream route

If the user supplies a `$nmt-analyze-interviews` result path, **do not route them through manual segment intake and do not make them re-describe the segment**. Preserve the path during pre-intake; read it only after the hard gate passes. Treat this as **Path D** alongside the base workflow's market-research Path A.

For Path D, parse and carry forward:

- selected/recommended Job-based segment and causal criteria;
- confidence label, support weights, supporting interview IDs, coherence/saturation caveats;
- Core Jobs, success criteria, and their **priority order**;
- current Solutions/DIY and `Job → Solution → Problem` evidence;
- Consideration Set / named alternatives and fears actually present in interviews;
- Big Jobs only at the confidence level supported upstream;
- explicit gaps, next-interview plan, and validation debt.

If the invocation already names the target segment or business goal, do not re-ask it. Ask only genuinely missing load-bearing information.

### Evidence semantics for Path D

Do not flatten an interview-analysis artifact into a generic user hunch. Preserve provenance:

- respondent quotes and concrete past behavior keep the quality assigned by `$nmt-analyze-interviews`;
- support weights / interview counts stay visible and constrain confidence;
- synthesized Jobs, segment labels, Big Jobs, thresholds, and value implications remain hypotheses unless independently validated;
- missing evidence remains missing — never fill it from model memory in Quick mode.

The value-proposition run may refine the **value hypothesis**, but it must not silently inflate the upstream **segment confidence**.

## 2. Hand-off debt is mandatory for Path A and Path D

This question is mandatory even when intake depth = **Just the essentials**.

- **Path A (`$nmt-market-research`):** ask what unvalidated assumptions from the research have since been checked in interviews, sales, or tests and what was learned.
- **Path D (`$nmt-analyze-interviews`):** ask: *“That interview analysis left explicit confidence limits and validation gaps. Since that report, which of them have you checked with additional interviews, sales, operational data, or a test — and what did you learn?”*

Anything genuinely confirmed becomes evidence with the validation method recorded. Anything not checked remains explicit validation debt and flows into S5 RAT cards and the Layer-1 debt count. For Path D, “not checked since” does **not** erase interview evidence already present; preserve its original weight/quality.

## 3. Amendments to the base S0→S6 workflow

Read `SKILL-BASE.md` now and execute it fully, with these binding amendments:

- In base **Intake depth**, “Just the essentials” may defer non-load-bearing materials/claims-ledger questions, but **may not defer upstream hand-off debt**.
- In base **Determine input path**, recognize Path D automatically when a `$nmt-analyze-interviews` result is supplied. If no route is supplied, the upstream-result option means either market-research or analyze-interviews; classify after loading.
- In base **Input-as-hypothesis**, distinguish customer/interview evidence from team-authored claims and model synthesis; preserve upstream provenance rather than labeling every artifact statement equally as a hunch.
- In **S4 / RICE Confidence**, ground confidence in upstream evidence (`$nmt-market-research` sources or `$nmt-analyze-interviews` interview support/quality). Do not award high confidence because a generated value idea merely sounds coherent.
- In **S5**, Path D starts from the interview report's confidence limits, explicit gaps, weak/unvalidated parts, and next-interview plan. Convert remaining gaps into falsifiable RAT assumptions, then add value/unit-economics/channel risks. Do not rebuild Segment+Jobs risk from model memory.
- In the final artifact, Path D's confidence note must name the source interview-analysis file, selected segment support/confidence, and unresolved gaps. Layer 2 must distinguish inherited interview evidence from new synthesis.
- In the execution/self-validation checklist, treat hand-off from `$nmt-analyze-interviews` exactly like a binding upstream hand-off: debt transferred, evidence quality preserved, confidence not inflated.

## 4. Fail-closed checks before S1 and before writing

Before S1, confirm in working context:

```text
PRE-INTAKE
orientation: PASS
language: PASS
mode: PASS
format: PASS
output_path: PASS

UPSTREAM (if Path D)
artifact_loaded: PASS
segment_selected: PASS
confidence_and_support_preserved: PASS
validation_debt_asked: PASS
```

Any FAIL means stop and resolve it; **no S1 analysis and no artifact write/commit**.

Before writing the final file, the base workflow's full gates still apply. Additionally, for Path D fail the run if the report:

- presents a thin/interview-hypothesis segment as confirmed/Solid without new evidence;
- invents competitor capabilities in Quick mode from mere product mentions;
- drops the upstream validation debt;
- treats respondent evidence and model synthesis as the same evidence class.

Then continue with the complete `SKILL-BASE.md` workflow and output contract.
