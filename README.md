# fresh-eyes-review

A compact agent skill for iterative document review. Fresh-context reviewer subagents audit a doc against a standard; the main agent triages findings, applies approved fixes, and loops until two clean rounds.

## Install (Claude Code)

```bash
cp -r skills/fresh-eyes-review ~/.claude/skills/
```

Then run `/fresh-eyes-review` and give it a file.

## License

Apache-2.0. Derived from [google/elephant-goldfish-model](https://github.com/google/elephant-goldfish-model) (`skills/egm-review-loop`); see [NOTICE](NOTICE).
