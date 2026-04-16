Here's the full process from auth to download:

---

## Step 1: Authenticate (one-time)

```bash
notebooklm login
```

Opens a Chromium browser. Log in with your Google account, then press **Enter** in the terminal to save the session to `storage_state.json`. You only need to do this once. [1](#2-0)

---

## Step 2: Find or create your notebook

If you already have a notebook, list it to get its ID:

```bash
notebooklm list
```

Or create a new one:

```bash
notebooklm create "My Notebook"
```

---

## Step 3: Set the active notebook

```bash
notebooklm use <notebook_id>
```

Partial IDs work (first 6+ chars). All subsequent commands will use this notebook by default. [2](#2-1)

---

## Step 4: Add sources (if not already present)

```bash
notebooklm source add "https://example.com/article"
# or a local file:
notebooklm source add "./my-document.pdf"
```

Wait for sources to be indexed before generating:

```bash
notebooklm source list
```

All sources should show `READY` status before proceeding. [3](#2-2)

---

## Step 5: Generate the quiz

```bash
notebooklm generate quiz
```

Optional flags:

- `--difficulty [easy|medium|hard]` (default: `medium`)
- `--quantity [fewer|standard|more]` (default: `standard`)
- `--wait` to block until generation completes
- A description argument for custom focus: `generate quiz "focus on vocabulary terms"`

Example with options:

```bash
notebooklm generate quiz "focus on key concepts" --difficulty hard --quantity more --wait
```

Without `--wait`, the command returns immediately with a `task_id`. Generation happens asynchronously. [4](#2-3) [5](#2-4)

---

## Step 6: (If no `--wait`) Check completion status

```bash
notebooklm artifact list --type quiz
```

Or poll a specific task:

```bash
notebooklm artifact poll <task_id>
``` [6](#2-5)

---

## Step 7: Download the quiz

```bash
# As JSON (default)
notebooklm download quiz quiz.json

# As Markdown
notebooklm download quiz --format markdown quiz.md

# As HTML
notebooklm download quiz --format html quiz.html
```

The JSON format preserves full API fields (`answerOptions`, `rationale`, `isCorrect`, `hint`). Markdown is human-readable with checkboxes. HTML is the raw format from NotebookLM. [7](#2-6) [8](#2-7)

---

## Condensed version (existing notebook, sources already ready)

```bash
notebooklm login                                          # once
notebooklm use <notebook_id>                              # set context
notebooklm generate quiz --difficulty hard --wait         # generate
notebooklm download quiz --format markdown ./quiz.md      # download
```

### Citations

**File:** docs/cli-reference.md (L105-123)

```markdown
### Source Commands (`notebooklm source <cmd>`)

Supported source types: URLs, YouTube videos, files (PDF, text, Markdown, Word, audio, video, images), Google Drive documents, and pasted text.

| Command | Arguments | Options | Example |
|---------|-----------|---------|---------|
| `list` | - | - | `source list` |
| `add <content>` | URL/file/text | - | `source add "https://..."` |
| `add-drive <id> <title>` | Drive file ID | - | `source add-drive abc123 "Doc"` |
| `add-research <query>` | Search query | `--mode [fast|deep]`, `--from [web|drive]`, `--import-all`, `--no-wait` | `source add-research "AI" --mode deep --no-wait` |
| `get <id>` | Source ID | - | `source get src123` |
| `fulltext <id>` | Source ID | `--json`, `-o FILE` | `source fulltext src123 -o content.txt` |
| `guide <id>` | Source ID | `--json` | `source guide src123` |
| `rename <id> <title>` | Source ID, new title | - | `source rename src123 "New Name"` |
| `refresh <id>` | Source ID | - | `source refresh src123` |
| `delete <id>` | Source ID | - | `source delete src123` |
| `delete-by-title <title>` | Exact source title | - | `source delete-by-title "My Source"` |
| `wait <id>` | Source ID | `--timeout`, `--interval` | `source wait src123` |

```

**File:** docs/cli-reference.md (L148-148)

```markdown
| `quiz [description]` | `--difficulty [easy\|medium\|hard]`, `--quantity [fewer\|standard\|more]`, `--wait` | `generate quiz --difficulty hard` |
```

**File:** docs/cli-reference.md (L155-166)

```markdown
### Artifact Commands (`notebooklm artifact <cmd>`)

| Command | Arguments | Options | Example |
|---------|-----------|---------|---------|
| `list` | - | `--type` | `artifact list --type audio` |
| `get <id>` | Artifact ID | - | `artifact get art123` |
| `rename <id> <title>` | Artifact ID, title | - | `artifact rename art123 "Title"` |
| `delete <id>` | Artifact ID | - | `artifact delete art123` |
| `export <id>` | Artifact ID | `--type [docs|sheets]`, `--title` | `artifact export art123 --type sheets` |
| `poll <task_id>` | Task ID | - | `artifact poll task123` |
| `wait <id>` | Artifact ID | `--timeout`, `--interval` | `artifact wait art123` |
| `suggestions` | - | `-s/--source`, `--json` | `artifact suggestions` |
```

**File:** docs/cli-reference.md (L266-287)

```markdown
### Session: `login`

Authenticate with Google NotebookLM via browser.

```bash
notebooklm login [OPTIONS]
```

Opens a Chromium browser with a persistent profile. Log in to your Google account, then press Enter in the terminal to save the session.

**Options:**

- `--storage PATH` - Where to save storage_state.json (default: `$NOTEBOOKLM_HOME/storage_state.json`)
- `--browser [chromium|msedge]` - Browser to use for login (default: `chromium`). Use `msedge` for Microsoft Edge.

**Examples:**

```bash
# Default (Chromium)
notebooklm login

# Use Microsoft Edge (for orgs that require Edge for SSO)
notebooklm login --browser msedge
```

```

**File:** docs/cli-reference.md (L289-300)
```markdown
### Session: `use`

Set the active notebook for subsequent commands.

```bash
notebooklm use <notebook_id>
```

Supports partial ID matching:

```bash
notebooklm use abc  # Matches abc123def456...
```

```

**File:** docs/cli-reference.md (L743-778)
```markdown
### Download: `quiz`, `flashcards`

Download quiz questions or flashcard decks in various formats.

```bash
notebooklm download quiz [OUTPUT_PATH] [OPTIONS]
notebooklm download flashcards [OUTPUT_PATH] [OPTIONS]
```

**Options:**

- `-n, --notebook ID` - Notebook ID (uses current context if not set)
- `--format FORMAT` - Output format: `json` (default), `markdown`, or `html`
- `-a, --artifact ID` - Select specific artifact by ID

**Output Formats:**

- **JSON** - Structured data preserving full API fields (answerOptions, rationale, isCorrect, hint)
- **Markdown** - Human-readable format with checkboxes for correct answers
- **HTML** - Raw HTML as returned from NotebookLM

**Examples:**

```bash
# Download quiz as JSON
notebooklm download quiz quiz.json

# Download quiz as markdown
notebooklm download quiz --format markdown quiz.md

# Download flashcards as JSON (normalizes f/b keys to front/back)
notebooklm download flashcards cards.json

# Download flashcards as markdown
notebooklm download flashcards --format markdown cards.md

# Download flashcards as raw HTML
notebooklm download flashcards --format html cards.html
```

```

**File:** src/notebooklm/cli/generate.py (L696-763)
```python
@generate.command("quiz")
@click.argument("description", default="", required=False)
@click.option(
    "-n",
    "--notebook",
    "notebook_id",
    default=None,
    help="Notebook ID (uses current if not set)",
)
@click.option("--quantity", type=click.Choice(["fewer", "standard", "more"]), default="standard")
@click.option("--difficulty", type=click.Choice(["easy", "medium", "hard"]), default="medium")
@click.option("--source", "-s", "source_ids", multiple=True, help="Limit to specific source IDs")
@click.option("--wait/--no-wait", default=False, help="Wait for completion (default: no-wait)")
@retry_option
@json_option
@with_client
def generate_quiz(
    ctx,
    description,
    notebook_id,
    quantity,
    difficulty,
    source_ids,
    wait,
    max_retries,
    json_output,
    client_auth,
):
    """Generate quiz.

    \b
    Use --json for machine-readable output.

    \b
    Example:
      notebooklm generate quiz "focus on vocabulary terms"
      notebooklm generate quiz "test key concepts" --difficulty hard --quantity more
    """
    nb_id = require_notebook(notebook_id)
    quantity_map = {
        "fewer": QuizQuantity.FEWER,
        "standard": QuizQuantity.STANDARD,
        "more": QuizQuantity.MORE,
    }
    difficulty_map = {
        "easy": QuizDifficulty.EASY,
        "medium": QuizDifficulty.MEDIUM,
        "hard": QuizDifficulty.HARD,
    }

    async def _run():
        async with NotebookLMClient(client_auth) as client:
            nb_id_resolved = await resolve_notebook_id(client, nb_id)
            sources = await resolve_source_ids(client, nb_id_resolved, source_ids)

            async def _generate():
                return await client.artifacts.generate_quiz(
                    nb_id_resolved,
                    source_ids=sources,
                    instructions=description or None,
                    quantity=quantity_map[quantity],
                    difficulty=difficulty_map[difficulty],
                )

            result = await generate_with_retry(_generate, max_retries, "quiz", json_output)
            await handle_generation_result(
                client, nb_id_resolved, result, "quiz", wait, json_output
            )
```

**File:** src/notebooklm/cli/download.py (L830-857)

```python
@download.command("quiz")
@click.argument("output_path", required=False, type=click.Path())
@click.option("-n", "--notebook", help="Notebook ID (uses current context if not set)")
@click.option(
    "--format",
    "output_format",
    type=click.Choice(["json", "markdown", "html"]),
    default="json",
    help="Output format",
)
@click.option("-a", "--artifact", "artifact_id", help="Select by artifact ID")
@click.pass_context
def download_quiz_cmd(ctx, output_path, notebook, output_format, artifact_id):
    """Download quiz questions.

    \b
    Examples:
      notebooklm download quiz quiz.json
      notebooklm download quiz --format markdown quiz.md
      notebooklm download quiz --format html quiz.html
    """
    try:
        result = run_async(
            _download_interactive(ctx, "quiz", output_path, notebook, output_format, artifact_id)
        )
        console.print(f"[green]Downloaded quiz to:[/green] {result}")
    except Exception as e:
        handle_error(e)
```
