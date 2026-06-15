# surveydown — Hugging Face deployment

Tooling to deploy the [surveydown](https://surveydown.org) survey templates to
**Hugging Face Spaces** (Docker SDK).

Each template is its own GitHub repo and remains the **single source of truth**.
This repo does not duplicate the surveys — it *generates* each Hugging Face Space
from its template at deploy time, so you only ever edit the template.

## How it works

For each template, `deploy.sh`:

1. Clones the template from `github.com/<GITHUB_ORG>/template_<name>` (tracked files only).
2. Assembles the Space content: the template's `app.R`, `survey.qmd`, and any
   `images/`, `data/`, etc., **plus** the shared `assets/Dockerfile`, a generated
   `README.md` (with Hugging Face frontmatter), and a generated `packages.txt`.
3. Pushes that content to the matching Hugging Face Space, which auto-rebuilds.

Naming is derived automatically: `template_question_types` → Space
`question-types` → URL `https://<HF_OWNER>-question-types.hf.space`.

`_survey/` is **not** shipped — the container renders the survey at startup
(Quarto is in the image). This also keeps the Space repos free of binary files,
which Hugging Face rejects in plain git.

## One shared Dockerfile

`assets/Dockerfile` is identical for every Space. Per-template R packages are
supplied via a generated `packages.txt` (derived from each template's
`library()`/`require()` calls), so there is exactly one Dockerfile to maintain.
surveydown itself is installed from GitHub (dev v1.3.0; CRAN only has 1.0.1).

## Prerequisites

- `git` and `tar`.
- Each target Space must already exist on Hugging Face (Docker SDK). Create one at
  <https://huggingface.co/new-space>, or with the HF CLI:
  ```bash
  hf repo create <HF_OWNER>/<space> --repo-type space --space-sdk docker
  ```
- Git must be able to push to `huggingface.co`. Run `hf auth login`, or you'll be
  prompted for your username and a **Write** token on the first push.

## Usage

```bash
# Build only (no push) — inspect the assembled Space folder under /tmp
./deploy.sh --no-push question-types

# Deploy one or more templates (any name form works)
./deploy.sh question-types
./deploy.sh question_types template_default

# Deploy everything listed in templates.txt
./deploy.sh --all
```

Override the GitHub org or Hugging Face owner via env vars:

```bash
GITHUB_ORG=surveydown-dev HF_OWNER=surveydown ./deploy.sh --all
```

## Files

| File | Purpose |
|------|---------|
| `deploy.sh` | Build + push script |
| `templates.txt` | List of template repos to deploy with `--all` |
| `assets/Dockerfile` | Shared Dockerfile used by every Space |
| `assets/dockerignore` | Copied into each Space as `.dockerignore` |
| `assets/space-readme.template.md` | README template (HF frontmatter) for each Space |

## Adding a template

Add its repo name (`template_<name>`) to `templates.txt`, create the Space, and
run `./deploy.sh <name>`. No other changes needed.

## Notes

- Spaces run in `mode: preview` (responses to an ephemeral `preview_data.csv`).
  For real data collection, switch the template to `mode: database` with an
  external PostgreSQL database: <https://surveydown.org/docs/storing-data>.
- Free Spaces sleep after inactivity and wake on the next visit (cold start).
