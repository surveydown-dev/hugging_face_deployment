# Connecting a database to a surveydown survey

Switch a survey from throwaway local storage to a **durable PostgreSQL database**
so responses survive restarts and are collected for real. This is what makes
`mode: database` work.

**This step always builds on an existing survey.** A database connection has no
meaning without a `survey.qmd` + `app.R` to attach it to. So before anything else,
make sure there is a survey — and if there isn't, create one first.

## Agent workflow (follow this when connecting a database for a user)

### Step 0 — Locate the survey (or create one)

You need a concrete survey directory (one containing both `app.R` and
`survey.qmd`) before connecting a database.

- If the user named or pointed at a survey, use it.
- If not, **ask which survey** they want to connect — and/or search the working
  area for survey folders:
  ```bash
  # find candidate surveys (dirs containing both app.R and survey.qmd)
  find . -name survey.qmd -maxdepth 4 -printf '%h\n' 2>/dev/null | sort -u
  ```
- **If there is no survey**, say so and route the user into
  **[`create-survey/`](../create-survey/README.md)** first. Do not try to connect a
  database to nothing. Once the survey exists, come back here.

Confirm the chosen survey directory back to the user before editing it.

### Step 1 — Ask: do you already have database credentials?

Ask whether the user already has a **Supabase** (recommended) or other PostgreSQL
database they can use.

- **Yes, I have credentials** → go to **Step 3**.
- **No / not sure** → walk them through creating a free Supabase database in
  **Step 2**, then continue.

surveydown stores data on **any** PostgreSQL database. The docs recommend
[Supabase](https://supabase.com) as a free, easy option, so that's the default
path below — but if the user already has another PostgreSQL host (RDS, Neon,
self-hosted, …), skip Step 2 and use those credentials in Step 3.

### Step 2 — Create a Supabase database (only if they have none)

Guide the user through this in their browser (you can't do it for them — it needs
their account):

1. Create a free account at <https://supabase.com> and create a **New project**.
2. Set a **project name**, a strong **database password** (have them save it — it's
   the `SD_PASSWORD`), and a **region** near their respondents.
3. Wait for the project to finish provisioning, then click **Connect** (top of the
   dashboard).
4. Choose the **Transaction pooler** connection (works from ephemeral hosts and
   IPv4). Copy the **connection string** — it looks like:
   ```
   postgresql://postgres.abcdefghijklmnop:[YOUR-PASSWORD]@aws-0-us-east-1.pooler.supabase.com:6543/postgres
   ```

That single URL contains every field you need (see **Credential reference** for how
it maps). The user supplies it; you turn it into a `.env` in Step 3.

### Step 3 — Write the credentials into a git-ignored `.env`

Credentials live in a **`.env`** file next to `app.R`, holding six `SD_*`
variables. **Never** commit it, never paste credentials into chat, never put them
in a dotfile — the `.env` is git-ignored and stays local.

**Preferred — `sd_db_config()` (it also git-ignores `.env` for you):** run it in an
R console **from the survey directory**. With no arguments it prompts interactively
for each field; you can also pass them explicitly:

```r
# interactive prompts (Host / Port / Database name / User / Password / Table):
surveydown::sd_db_config()

# or non-interactively, one field per argument:
surveydown::sd_db_config(
  host     = "aws-0-us-east-1.pooler.supabase.com",
  port     = "6543",
  dbname   = "postgres",
  user     = "postgres.abcdefghijklmnop",
  password = "YOUR-PASSWORD",
  table    = "responses"          # any name; created automatically on first write
)
```

`sd_db_config()` writes the `.env` **and appends `.env` to `.gitignore`**, so the
secret can't be committed.

> **Note — no `url =` shortcut:** `sd_db_config()` takes the **six fields** above;
> it has **no `url =` argument** (older docs implied one). So **parse the Supabase
> connection URL yourself** into the six fields (see Credential reference) and pass
> them, or enter them at the interactive prompts.

**Fallback — hand-write `.env`** (e.g. no R handy). Create `.env` in the survey
folder:

```
# Database connection settings for surveydown
SD_HOST=aws-0-us-east-1.pooler.supabase.com
SD_PORT=6543
SD_DBNAME=postgres
SD_USER=postgres.abcdefghijklmnop
SD_TABLE=responses
SD_PASSWORD=YOUR-PASSWORD
```

Then make sure `.env` is git-ignored:

```bash
grep -qxF '.env' .gitignore 2>/dev/null || echo '.env' >> .gitignore
```

### Step 4 — Make sure `app.R` connects

The scaffolded templates already do this — `app.R` has, in its global scope:

```r
db <- sd_db_connect()        # reads .env
...
  sd_server(db = db)         # inside server()
```

`sd_db_connect()` reads the six `SD_*` values from `.env` (falling back to existing
environment variables) and returns a connection pool. If `app.R` is missing this
line (e.g. a hand-built survey), add it. No credentials are written in `app.R` —
only the call.

### Step 5 — Flip the survey into `database` mode

The **mode** decides where responses go, and it lives in `survey.qmd`, not in the
connection call. Edit the `survey-settings:` block of the survey's `survey.qmd`:

```yaml
survey-settings:
  mode: database      # was: preview
```

The three modes:
- **`preview`** — responses go to a local `preview_data.csv`; any DB connection is
  **ignored**. Default for new templates; good for testing.
- **`local`** — responses go to a local `local_data.csv`; no database.
- **`database`** — responses go to the connected PostgreSQL database. The only
  **durable** option on ephemeral hosts.

Only `mode: database` actually writes to the database — connecting without flipping
the mode still saves to the CSV.

### Step 6 — Verify the connection

- **Locally:** open `app.R` and *Run App* (or `shiny::runApp()` in the folder).
  - A successful `sd_db_connect()` prints **"Successfully connected to the
    database."** in the R console.
  - If credentials are missing/wrong, the app still launches but shows a
    **"DATABASE NOT CONNECTED — responses are not being saved"** banner, and the
    console reports which `SD_*` values are missing. Fix the `.env` and rerun.
  - If connection details look correct but it still won't connect, try
    `sd_db_connect(gssencmode = "disable")` (some networks block GSSAPI; `"auto"`
    already retries this, but confirm).
- **Check stored data:** submit a test response, then in R:
  ```r
  db <- surveydown::sd_db_connect()
  surveydown::sd_get_data(db)     # pulls the table back as a data frame
  ```
  The `table` (`SD_TABLE`, default `responses`) is created automatically on the
  first write — you don't pre-create it.

## Credential reference

The six variables `sd_db_connect()` reads (and `sd_db_config()` writes):

| `.env` variable | Meaning | From a Supabase pooler URL |
|---|---|---|
| `SD_HOST` | database host | the part after `@`, before `:` (`aws-0-…pooler.supabase.com`) |
| `SD_PORT` | port | after the host's `:` (`6543` for the transaction pooler) |
| `SD_DBNAME` | database name | after the final `/` (`postgres`) |
| `SD_USER` | database user | between `//` and `:` (`postgres.abcdefghijklmnop`) |
| `SD_PASSWORD` | database password | the `[YOUR-PASSWORD]` slot (the password you set) |
| `SD_TABLE` | table for responses | **not in the URL** — your choice (default `responses`); auto-created |

Connection string anatomy:

```
postgresql://<SD_USER>:<SD_PASSWORD>@<SD_HOST>:<SD_PORT>/<SD_DBNAME>
             └── user ──┘ └ password ┘ └─ host ─┘ └port┘  └ dbname ┘
```

## Secrets safety (for both user and assistant)

- **`.env` is local and git-ignored.** Never commit it; never paste DB credentials
  into the conversation, a script, or a dotfile (`.zshrc`/`.Renviron` checked into
  git, etc.). `sd_db_config()` git-ignores `.env` automatically; the hand-written
  fallback above does it explicitly.
- **Assistant rule:** drive credentials through `sd_db_config()` prompts or the
  `.env` file — don't ask the user to type a password into chat. If you only need
  to confirm a connection works, run the app / `sd_get_data()`; don't echo the
  password.
- **On a host, credentials are host secrets, not files.** When you later deploy a
  `mode: database` survey, the same six `SD_*` values must exist as the host's
  **secrets** (e.g. Hugging Face Space Secrets), and the deploy tooling pushes them
  from `.env` for you — the `.env` itself is never uploaded. See the deploy
  sections for specifics:
  [`deploy-hugging-face/`](../deploy-hugging-face/README.md) (auto-syncs `.env` →
  Space Secrets), [`deploy-google-cloud/`](../deploy-google-cloud/README.md),
  [`deploy-posit-cloud/`](../deploy-posit-cloud/README.md).

## When unsure

Fetch the authoritative data-storage doc (it regenerates on every site build):
**<https://www.surveydown.org/docs/storing-data.llms.md>**. Prefer it over memory —
and where it disagrees with the installed package (e.g. the `url =` argument above),
trust the **real package** behavior. See "Authoritative docs" in the top-level
[`SKILL.md`](../SKILL.md).
