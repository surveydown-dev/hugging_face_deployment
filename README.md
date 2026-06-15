# surveydown-skill

A skill for working with [surveydown](https://surveydown.org) surveys — authoring
and deploying them.

👉 **Start with [`SKILL.md`](SKILL.md)**, which
routes to the right task doc.

## What it covers

| Task | Status |
|------|--------|
| Create a new survey | 🚧 under construction |
| Connect a database to store responses | 🚧 under construction |
| Deploy to Hugging Face Spaces | ✅ available |
| Deploy to Posit Connect Cloud | 🚧 under construction |

The Hugging Face deployment is fully implemented — see
[`resources/hugging-face-deployment.md`](resources/hugging-face-deployment.md)
(tooling in [`hugging-face/`](hugging-face/)).
The other topics are stubbed and being filled in.

## Install (Claude Code)

```bash
npx skills add surveydown-dev/surveydown-skill -a claude-code -g -y
```

This installs the `surveydown` skill globally to `~/.claude/skills/`. Start a new
Claude Code session and it's available. ([`npx skills`](https://github.com/vercel-labs/skills)
is the open agent-skills installer.)

## Update

```bash
npx skills add surveydown-dev/surveydown-skill -a claude-code -g -y
```

Re-running `add` pulls the latest version.

## Uninstall (Claude Code)

```bash
npx skills remove surveydown-dev/surveydown-skill -g
```
