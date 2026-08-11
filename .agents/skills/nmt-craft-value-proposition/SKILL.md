---
name: nmt-craft-value-proposition
description: >-
  Generate a Value Proposition for a chosen segment using Ivan Zamesin's AJTBD /
  Next Move Theory methodology. Russian-language Codex wrapper; Quick or Deep;
  results are delivered to the user in chat by default.
user-invocable: true
---

# Craft Value Proposition — Codex entrypoint

> **Binding Codex workflow:** after the PRE-INTAKE gate below passes, read `./SKILL-CODEX-BASE.md` and execute it fully. That file preserves the existing interview-analysis handoff, evidence-semantics, validation-debt, and fail-closed overrides and then loads `SKILL-BASE.md`. Read `../PRODUCER-CONTRACT.md` first. If this wrapper conflicts with `SKILL-CODEX-BASE.md`, **this wrapper wins**.

## PRE-INTAKE HARD GATE

This fork works **in Russian by default and without a language question**.

The first assistant response after `$nmt-craft-value-proposition` must:
1. print the existing helicopter-view in Russian;
2. ask only for any unresolved **Mode: Quick / Deep** and **Format: Markdown / HTML**;
3. stop until both are resolved.

Before both controls PASS, do **not** ask intake depth, input route, upstream artifact path, target segment/business-goal questions, materials, hand-off debt, or any other skill-specific question. Do not load canon beyond what is necessary to orient the user; do not analyze, research, write, commit, or open a PR.

After mode + format are resolved, continue with `SKILL-CODEX-BASE.md`. Its older requirements to explicitly choose language or output path are overridden by this wrapper and the shared producer contract.

## Result delivery override

Normal output is delivered **to the user in chat**. Do not ask for an output path and do not require `Skills-Results/...`, a repo file, commit, or PR. Existing path/file rules apply only if the user explicitly asks to save/export the result to the repository.

All existing Path A/Path D handoff rules, evidence semantics, confidence controls, validation debt, RAT logic, Deep QA, readability rules, approval, and semantic-handoff behavior remain binding through `SKILL-CODEX-BASE.md` + `SKILL-BASE.md`.
