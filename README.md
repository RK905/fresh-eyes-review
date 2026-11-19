# fresh-eyes-review

An agent skill that reviews a document the way a good team does: several reviewers who have **never seen the conversation** read it cold, flag problems, and the author fixes them. It repeats until two rounds in a row come back clean.

Works with Claude Code and any agent harness that can spawn subagents.

## Why fresh eyes?

When you write a doc with an AI agent, the agent "knows" everything you discussed, so it can't see what the doc leaves unsaid. Reviewers that start from a clean context only see what's on the page. They catch:

- unstated assumptions and missing context
- vague or untestable requirements
- gaps between the doc and the actual codebase
- structure and clarity problems

## How it works

```
             ┌──────────────────────────────────────────┐
             │  Main agent (keeps full context)         │
             │                                          │
  round N →  │ 1. Locate the target section             │
             │ 2. Spawn reviewers in parallel ──────────┼──► Reviewer A (fresh context)
             │                                 ◄────────┼─── Reviewer B (fresh context)
             │ 3. Triage findings: contest or fix       │    Reviewer C ...
             │ 4. You approve fixes → edits applied     │
             │ 5. Wait for you → next round             │
             └──────────────────────────────────────────┘
```

Each round:

1. **Locate**: re-read the file and find the target section (line numbers move after edits).
2. **Review**: each reviewer gets one lens (e.g. "implementation readiness"), a standard to judge against, and the doc. Nothing else. They report findings as 🔴 Blocker, 🟡 Major or 🟢 Nit.
3. **Triage**: the main agent decides, per finding, whether to **contest** it (with a reason) or **propose a fix**, and writes `review_report.md`.
4. **Approve**: you accept all fixes, pick individually, or skip.
5. **Pause**: the agent waits for you before starting the next round.

### When it stops

| Outcome | Rule |
|---|---|
| ✅ Converged | Two clean rounds in a row (no open Blocker/Major, every reviewer ran) |
| ⏹ Cap | 5 rounds reached |
| 🔁 Churn | A new finding would be fixed by reverting an earlier fix |

On cap or churn, you get the open findings and choose: accept, keep going, or send the doc back to drafting.

Contested or accepted-risk findings stay closed. A fixed issue that comes back is reopened.

## Install

**Claude Code (personal, all projects):**

```bash
git clone https://github.com/RK905/fresh-eyes-review /tmp/fer
cp -r /tmp/fer/skills/fresh-eyes-review ~/.claude/skills/
```

**One project only:** copy `skills/fresh-eyes-review` into `<project>/.claude/skills/`.

**Other harnesses:** point your agent at `skills/fresh-eyes-review/SKILL.md`. It needs a tool that spawns subagents with fresh context.

## Usage

```
/fresh-eyes-review docs/design.md
```

The skill asks only for what you didn't provide:

| Input | What it is | Example |
|---|---|---|
| `target` | File to review | `docs/payments-spec.md` |
| `scope` | Section or `full file` | `section "Rollout plan"` |
| `reviewers` | Lenses + standard for each | `security: no secrets, authz on every endpoint` |
| `context` | Files/URLs reviewers read first | `src/payments/`, `docs/architecture.md` |

Default reviewers if you don't pick:

- **Feature spec**: design critic + structure
- **Implementation plan**: implementation readiness + codebase accuracy

### Example

```
/fresh-eyes-review docs/migration-plan.md
  scope: full file
  reviewers:
    - implementation readiness: could an engineer start tomorrow without questions?
    - codebase accuracy: every file, table and function named actually exists
    - risk: rollback path and failure modes are explicit
  context: src/db/, migrations/
```

A standard can be a one-line rubric (as above) or a path to a longer checklist file.

## Tips

- **Write your own standards.** A sharp rubric gives sharp findings. Keep reusable ones in a `standards/` folder and pass the path.
- **2–3 reviewers is plenty.** More lenses means more noise per round.
- **Give code as context** for implementation plans, so "codebase accuracy" can actually check.
- **Contest freely.** Fresh reviewers lack context by design; some findings will be wrong. Contested findings don't come back.

## Credits

Derived from `egm-review-loop` in [google/elephant-goldfish-model](https://github.com/google/elephant-goldfish-model), condensed and made self-contained. See [NOTICE](NOTICE).

## License

[Apache-2.0](LICENSE)
