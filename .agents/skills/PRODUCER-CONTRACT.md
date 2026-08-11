# Producer contract — Codex execution override

> **Binding base contract:** after applying the overrides in this file, read `./PRODUCER-CONTRACT-BASE.md` and use it for the complete producer rules. If this file conflicts with the base contract, **this file wins**.

This Codex fork is optimized for a non-technical browser user. The skills live in GitHub; **GitHub is not the default storage location for analytical results**.

## 0. Fixed defaults

- **Language is Russian.** `language_resolved = true` from invocation. Do not ask the user to choose a language and do not infer it from the current message; this fork simply works in Russian.
- **Result destination is the chat by default.** Do not ask for an output path during intake.
- Do not require a repo file, `Skills-Results/...`, commit, branch, or PR for a normal analytical run.
- Saving/exporting to a repository remains allowed when the user explicitly asks for it. Only then ask for a path if one is genuinely needed, or use the path the user supplied.

## 1. PRE-INTAKE HARD GATE

Before any skill-specific intake question, substantive analysis, canon loading beyond what is needed to orient the user, upstream-artifact read, subagent spawn, web/shell research, report drafting, result delivery, file write, commit, or PR action, resolve these states:

1. `orientation_emitted = true` — print the skill's helicopter view in Russian.
2. `mode_resolved = true` — Quick or Deep was explicitly chosen/supplied in this run.
3. `format_resolved = true` — Markdown or HTML was explicitly chosen/supplied in this run.

`language_resolved` is already PASS because Russian is fixed. There is no `output_path_resolved` pre-intake state.

If mode or format is unresolved, the **first assistant response** must begin with the helicopter view and then ask **only the missing mode / format controls**. Stop there. Do not ask intake depth, input route, product/business questions, materials questions, upstream path, validation-debt questions, or anything else until both controls PASS.

Local wording such as **“ask intake depth first”**, **“ask the business task first”**, **“ask input route first”**, **“Default English”**, or **“where to save?”** is subordinate to this gate and must not run before it.

Internal self-check before skill-specific intake:

```text
PRE-INTAKE
orientation: PASS/FAIL
language_russian_fixed: PASS
mode: PASS/FAIL
format: PASS/FAIL
```

Any FAIL blocks skill-specific intake and substantive work.

## 2. Default result delivery

The normal run ends by **delivering the result to the user in chat**.

- **Markdown selected:** deliver the finished substantive result as Markdown in chat.
- **HTML selected:** deliver the finished self-contained HTML to the user through the chat/artifact capability available in the host. Do not make a GitHub path a prerequisite for HTML.
- A repo file is optional, not part of the default workflow. Existing filename/path conventions in the base contract and individual skills apply **only when the user explicitly requests saving/exporting to the repository**.
- Do not create commits or PRs for analytical results unless the user explicitly asks for that GitHub action.

The one-result principle still applies: one analytical result per run. Optional post-approval semantic handoff is a separate transport step.

## 3. Approval and semantic handoff

The base contract's approved semantic Markdown handoff remains binding. Default transport is chat-first. Do not create a repo `.md` handoff unless the user explicitly asks to save it.

## 4. Deep-mode gates

All evidence floors, self-critics, validation-debt registry, methodology invariants, and Deep completion gates from the base contract remain binding. When the default result is chat-only, interpret “PRE-WRITE / PRE-COMMIT” as **PRE-FINAL-DELIVERY** as well: a failed mandatory Deep gate blocks presenting a complete Deep result in chat. File/commit checks activate only when the user explicitly requested those actions.
