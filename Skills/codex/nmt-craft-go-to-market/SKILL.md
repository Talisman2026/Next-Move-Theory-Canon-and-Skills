---
name: nmt-craft-go-to-market
description: >-
  Write go-to-market communication for a chosen segment using Ivan Zamesin's AJTBD /
  Next Move Theory methodology. Russian-language Codex wrapper; Quick or Deep;
  results are delivered to the user in chat by default.
user-invocable: true
---

# Craft Go-To-Market — Codex entrypoint

> **Binding base workflow:** after the PRE-INTAKE gate below passes, read `./SKILL-BASE.md` and execute its complete workflow. Read `../PRODUCER-CONTRACT.md` first. If this wrapper conflicts with `SKILL-BASE.md`, **this wrapper wins**.

## PRE-INTAKE HARD GATE

This fork works **in Russian by default and without a language question**.

The first assistant response after `$nmt-craft-go-to-market` must:
1. print the base skill's helicopter-view in Russian;
2. ask only for any unresolved **Mode: Quick / Deep** and **Format: Markdown / HTML**;
3. stop until both are resolved.

Before both controls PASS, do **not** ask intake depth, input route, upstream artifact path, requested GTM assets, product/segment/value questions, materials, validation-debt questions, or any other skill-specific question. Do not load canon beyond what is necessary to orient the user; do not analyze, draft copy, research, write, commit, or open a PR.

After mode + format are resolved, continue with `SKILL-BASE.md`. Any base instruction such as “Intake depth — ask this first”, “Default English”, or “where to save the result” is subordinate to this wrapper and the shared producer contract.

## Result delivery override

Normal output is delivered **to the user in chat**. Do not ask for an output path and do not require `Skills-Results/...`, a repo file, commit, or PR. Existing path/file rules in the base skill apply only if the user explicitly asks to save/export the result to the repository.

All upstream-debt semantics, copy guardrails, validation-debt registry, Deep research/evidence rules, readability rules, approval, and semantic-handoff behavior from the base workflow remain binding.
