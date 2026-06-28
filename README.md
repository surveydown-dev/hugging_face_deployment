# surveydown-skill

A skill for working with [surveydown](https://surveydown.org) surveys — authoring
and deploying them.

👉 **Start with [`SKILL.md`](SKILL.md)**, which
routes to the right task doc.

## What it covers

| Task | Status |
|------|--------|
| Create a new survey | ✅ available |
| Connect a database to store responses | ✅ available |
| Deploy to Hugging Face Spaces | ✅ available |
| Deploy to Google Cloud Run | ✅ available |
| Deploy to Posit Connect Cloud | ✅ available |

Each section lives in its own folder with a `README.md` guide and its tooling.
Authoring starts with [`create-survey/`](create-survey/README.md) (scaffold from a
template or compose a custom survey) and, once a survey exists,
[`connect-database/`](connect-database/README.md) (wire it to PostgreSQL/Supabase
and switch to `mode: database`). Deployment is fully implemented for all three
hosts — [`deploy-hugging-face/`](deploy-hugging-face/README.md),
[`deploy-google-cloud/`](deploy-google-cloud/README.md), and
[`deploy-posit-cloud/`](deploy-posit-cloud/README.md).

## Install (Claude Code)

```bash
npx skills add surveydown-dev/surveydown-skill -a claude-code -g -y
```

This installs the `surveydown-skill` skill globally to `~/.claude/skills/`. Start a new
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
