# surveydown-skill

A skill for working with [surveydown](https://surveydown.org) surveys — authoring
and deploying them.

👉 **Start with [`SKILL.md`](SKILL.md)**, which routes to the right task doc.

Implemented:
- **Deploy templates to Hugging Face Spaces** — see
  [`resources/hugging-face-deployment.md`](resources/hugging-face-deployment.md)
  (tooling in [`hugging-face/`](hugging-face/)).

Planned: creating a survey from scratch, and deploying to Posit Connect Cloud.

## Install (Claude Code)

Clone the repo, then run the installer. It symlinks the skill into
`~/.claude/skills/surveydown`, so `git pull` later updates it instantly.

```bash
git clone https://github.com/surveydown-dev/surveydown-skill.git
cd surveydown-skill
./install.sh
```

Start a new Claude Code session and the `surveydown` skill is available.

Prefer one line and no scripts? Clone straight into the skills directory:

```bash
git clone https://github.com/surveydown-dev/surveydown-skill.git ~/.claude/skills/surveydown
```

## Update

```bash
git -C ~/.claude/skills/surveydown pull   # if cloned into the skills dir
# or, if you used install.sh, pull in your cloned repo:
git pull
```

## Uninstall (Claude Code)

```bash
./uninstall.sh            # if you installed with install.sh
# or, if you cloned into the skills dir:
rm -rf ~/.claude/skills/surveydown
```

`uninstall.sh` only removes the `~/.claude/skills/surveydown` symlink; your cloned
repo is left untouched.
