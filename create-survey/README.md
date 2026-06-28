# Creating a surveydown survey

Scaffold a new surveydown survey — a directory holding a **`survey.qmd`** (pages,
questions, navigation) and an **`app.R`** (the Shiny app: DB connection, UI,
server). This is usually the **first** step of working with surveydown, so it can
be triggered on its own — you do not need an existing survey to start here. (Once a
survey exists, [`connect-database/`](../connect-database/README.md) and the deploy
sections build on it.)

The 17 official templates are open source under the **surveydown-dev** GitHub org —
**<https://github.com/surveydown-dev>** — one repo per template
(`template_<name>`). Rather than vendoring them into this skill, we fetch them on
demand: the package's own `sd_create_survey()` downloads
`https://github.com/surveydown-dev/template_<name>/archive/refs/heads/main.zip` and
copies the files into the target folder. Treat the live repos as the source of
truth; the catalog below is just a routing map.

## Agent workflow (follow this when creating a survey for a user)

### Step 1 — Ask *what survey* to create (do not list all 17 templates)

Open with a short, tiered menu. Most users want one of the first two; the third is
the net-new authoring path. Present these three choices:

1. **Minimum (default template)** — the bare starter: a two-page survey with one
   multiple-choice and one text question, plus a finish page. Best when the user
   wants a clean slate to write their own content into. → scaffold `default`.
2. **Themed showcase** — a richer survey demonstrating real features. Offer a few
   based on what the user is after (don't dump the whole list):
   - varied **question types** → `question_types` (questions in R) or
     `questions_yml` (questions defined in a `questions.yml` file).
   - **conditional logic** → `conditional_showing`, `conditional_skipping`, or
     `conditional_stopping`.
   - **randomization** → `random_options`, `random_options_predefined`,
     `option_shuffling`.
   - **dynamic / advanced** → `reactive_questions`, `reactive_drilldown`,
     `live_polling`, `external_redirect`, `conjoint_buttons`, `conjoint_tables`,
     `custom_plotly_chart`, `custom_leaflet_map`.
   See the **Template catalog** below for one-line descriptions, and only surface
   the handful relevant to the user's stated goal.
3. **Custom survey (define your own)** — the user describes the survey they want
   (topic, questions, flow) and you **compose** it from the template patterns. The
   result may differ from any single existing template. → see **Building a custom
   survey** below.

Ask **where** to create it too (a path / folder name). Every survey must live in
**its own separate project folder** — never scaffold on top of an unrelated
project.

### Step 2 — Scaffold

**For a known template (choices 1 & 2)**, prefer the package function — it pulls
the current template straight from GitHub:

```r
# from an R console, in the parent directory:
surveydown::sd_create_survey(template = "default", path = "my_survey")
# template = one of the catalog names below; path = the new survey's folder
```

`sd_create_survey()` is interactive by default (it confirms before using the
current directory and before overwriting files). When driving it non-interactively,
pass `ask = FALSE` and a fresh `path`:

```r
surveydown::sd_create_survey(template = "question_types", path = "my_survey", ask = FALSE)
```

If R isn't available in the environment, you can reproduce exactly what the function
does — download and unpack the template repo — without R:

```bash
mkdir -p my_survey && cd my_survey
curl -fsSL https://github.com/surveydown-dev/template_default/archive/refs/heads/main.zip -o t.zip
unzip -q t.zip && cp -R template_default-main/. . && rm -rf template_default-main t.zip
```

Either way you end up with `app.R` + `survey.qmd` (plus any template-specific files
like `questions.yml`, `images/`). Confirm the folder contents back to the user.

### Step 3 — Confirm settings and next steps

- The scaffolded survey ships in **`mode: preview`** (responses written to a local
  `preview_data.csv` — fine for testing, not durable). Leave it there for now;
  switching to real storage is the **[`connect-database/`](../connect-database/README.md)**
  step.
- Point the user at how to **preview locally**: open `app.R` in RStudio and click
  *Run App*, or run `shiny::runApp()` in the survey folder.
- Offer the obvious follow-ups: **connect a database** (durable responses) or
  **deploy** (Hugging Face / Posit Connect Cloud / Google Cloud Run).

## Template catalog (name → what it shows)

The `template` value is the catalog name; its repo is
`https://github.com/surveydown-dev/template_<name>`.

| Template (`template =`) | What it demonstrates |
|---|---|
| `default` | Minimum starter: 2 pages, one `mc` + one `text` question, finish page. |
| `question_types` | Every built-in question type, defined inline in R. |
| `questions_yml` | Every question type, defined in a `questions.yml` file. |
| `conditional_showing` | Show a question/page only if a condition holds (`sd_show_if()`). |
| `conditional_skipping` | Skip ahead to a page when a condition holds (`sd_skip_if()`). |
| `conditional_stopping` | End the survey early when a condition holds (`sd_stop_if()`). |
| `random_options` | Randomize the options shown for a question. |
| `random_options_predefined` | Randomize options drawn from predefined sets. |
| `option_shuffling` | Shuffle the order of a question's options. |
| `reactive_questions` | Questions that react to earlier answers. |
| `reactive_drilldown` | Later options depend on earlier selections (drill-down). |
| `live_polling` | Real-time response visualization (bar charts). |
| `external_redirect` | Reactivity + redirect links that accept URL parameters. |
| `conjoint_buttons` | Choice-based conjoint survey with options as buttons. |
| `conjoint_tables` | Choice-based conjoint survey with options in tables. |
| `custom_plotly_chart` | Custom Plotly chart question via `sd_question_custom()`. |
| `custom_leaflet_map` | Custom Leaflet map question via `sd_question_custom()`. |

These names are also the exact set `sd_create_survey()` accepts. If you're ever
unsure a name is current, the canonical list is the repo list at
<https://github.com/surveydown-dev> (filter for `template_`).

## What a survey looks like (so you can build/edit one)

A survey is two files. The **`app.R`** is nearly boilerplate — connect, build UI,
run server:

```r
library(surveydown)

db <- sd_db_connect()   # reads .env if present; NULL in preview mode (that's fine)

ui <- sd_ui()

server <- function(input, output, session) {
  sd_skip_if()   # conditional skip logic (optional)
  sd_show_if()   # conditional display logic (optional)
  sd_server(db = db)
}

shiny::shinyApp(ui = ui, server = server)
```

The **`survey.qmd`** holds a YAML header (theme + `survey-settings:` + system
messages) and the survey body. Pages are delimited by `--- pagename`; questions are
R chunks calling `sd_question()`; navigation is `sd_nav()` / `sd_close()`:

````markdown
---
theme-settings:
  theme: default
  barposition: top
survey-settings:
  mode: preview
  use-cookies: false
  all-required: false
---

```{r}
library(surveydown)
```

--- welcome

# Welcome

```{r}
sd_question(
  type   = 'mc',
  id     = 'penguins',
  label  = "What's your favorite penguin?",
  option = c('Adélie' = 'adelie', 'Chinstrap' = 'chinstrap', 'Gentoo' = 'gentoo')
)
```

--- end

```{r}
sd_close()
```
````

### Building a custom survey (choice 3)

When the user wants something no single template covers, compose one:

1. **Start from the closest template** — usually `default` for structure, or
   `questions_yml` if they have many questions (cleaner to edit in YAML than R).
2. **Gather the spec from the user** in the same Q&A style used elsewhere in this
   skill: the survey's topic, the list of questions (and their types), how many
   pages, and any conditional flow ("if they pick X, show/skip/stop ...").
3. **Write the questions.** Each `sd_question()` (or `questions.yml` entry) needs a
   `type`, a unique `id`, a `label`, and — for choice types — `option`/`options`.
   The full type list (with the `id`s referenced here) is in the `questions_yml`
   template and the docs: `text`, `textarea`, `numeric`, `mc`, `mc_buttons`,
   `mc_multiple`, `mc_multiple_buttons`, `mc_image`, `mc_multiple_image`, `select`,
   `slider`, `slider_numeric`, `date`, `daterange`, `matrix`, `matrix_multiple`.
4. **Lay out pages** with `--- pagename` delimiters; end with `sd_close()`.
5. **Wire conditional logic** in `app.R` if needed: `sd_show_if()` (show a question
   when a condition is true), `sd_skip_if()` (jump to a page), `sd_stop_if()` (end
   early). Copy the exact argument shape from the matching `conditional_*` template.
6. **Verify against the docs** before finalizing — fetch the relevant
   `*.llms.md` pages (see "Authoritative docs" in the top-level
   [`SKILL.md`](../SKILL.md)). The installed roxygen only covers signatures; the
   docs show how features are meant to be assembled.

Do **not** hand-write question/setting syntax from memory — confirm the option
shapes and setting names against a real template or the `.llms.md` docs first.

## After creating

The survey is now the source of truth. Next steps, each in its own section:

- **Durable responses** → [`connect-database/`](../connect-database/README.md)
  (Supabase / PostgreSQL, then flip `mode: database`).
- **Publish online** → [`deploy-hugging-face/`](../deploy-hugging-face/README.md),
  [`deploy-posit-cloud/`](../deploy-posit-cloud/README.md), or
  [`deploy-google-cloud/`](../deploy-google-cloud/README.md).
- **Record a walkthrough** → [`record-video/`](../record-video/README.md).
