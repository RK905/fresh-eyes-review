---
name: fresh-eyes-review
license: Apache-2.0
disable-model-invocation: true
description: >-
  Iterative multi-agent document review: reviewers with fresh context audit a
  document against a standard; the main agent triages findings and applies
  approved fixes until two clean rounds. Use for design docs, specs, READMEs,
  implementation plans.
---

# Fresh Eyes Review

**Reviewers** are subagents with fresh context (no conversation history), so they catch unstated assumptions. **You** keep the context, judge their findings, and edit. No subagent tool → stop and tell the user; a same-context review defeats the point.

## Inputs (ask only for missing ones, then confirm the plan once)

- `target` – file path.
- `scope` – stable locator (e.g. "section 'Rollout plan'") or `full file`.
- `reviewers` – list of lenses, each with a standard (a file path or a one-line rubric). Defaults: spec → *design critic* + *structure*; implementation plan → *implementation readiness* + *codebase accuracy*.
- `context` – files/URLs reviewers should read first (empty is valid). Save URLs to local files; pass absolute paths.

## Each round

1. **Locate**: re-read `target`; resolve `scope` to current line range (edits move it).
2. **Review**: spawn all reviewers in parallel. Give each: its lens, standard, `target` + line range, `context`, and this output format: findings with severity 🔴 Blocker / 🟡 Major / 🟢 Nit, lines, issue, rationale. Do not pass your opinions or prior findings.
   - Failed/empty reviewer → retry once, else mark **not run** (never "zero findings").
   - Merge and dedupe findings; id `<reviewer>-<round>-<n>`.
3. **Triage**: for each finding decide **contest** (with reason) or **propose fix**. Write `review_report.md` (findings, decisions, status per reviewer). Ask the user: accept all fixes / pick individually / skip. Apply approved edits.
4. **Stop and wait** for the user before another round.

## Convergence

- **Settled** = contested or accepted-risk; never reopen. **Fixed** ≠ settled; same issue at same scope reappearing → reopen.
- **Clean round** = no unsettled Blocker/Major and no not-run reviewer (judge before applying fixes).
- **Done** after 2 consecutive clean rounds. **Cap** 5 rounds. **Churn** = a new finding that reverting a prior fix would resolve.
- On cap/churn: list open findings by severity; ask: accept, continue, or back to drafting.

## Finish

Report rounds run, outcome (converged / accepted at cap / halted), report path; ask about next step.
