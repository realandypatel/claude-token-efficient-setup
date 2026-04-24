# Setup Guide

Step-by-step install for Claude Desktop + `code-review-graph` + `filesystem` MCP on macOS.

Budget 30–45 minutes the first time. Subsequent projects on the same machine take ~5 minutes.

---

## Prerequisites

You need:

- **macOS** (tested on Apple Silicon; Intel should work identically)
- **Claude Desktop app** installed and signed in
- **Homebrew** installed ([brew.sh](https://brew.sh))
- **Python 3.11+** (the system Python on recent macOS works)
- **Node.js 18+** (for the `filesystem` MCP, which runs via `npx`)
- **A project directory** you want to index — any language works, Python/TypeScript/JavaScript/Go/Rust are well-supported by the graph

Verify prerequisites in Terminal:

```bash
python3 --version         # should be 3.11+
which brew                # should print /opt/homebrew/bin/brew (Apple Silicon) or /usr/local/bin/brew (Intel)
node --version            # should be 18+
```

If any are missing, install them first and come back.

---

## Step 1 — Install `pipx` and `code-review-graph`

`pipx` runs Python CLIs in isolated environments, which is what `code-review-graph` wants.

```bash
brew install pipx
pipx ensurepath
```

Close and reopen your Terminal after `ensurepath` so the PATH update takes effect.

Now install the graph tool:

```bash
pipx install code-review-graph
```

Verify:

```bash
which code-review-graph
code-review-graph --help
```

You should get a path like `/Users/yourname/.local/bin/code-review-graph` and a usage message.

---

## Step 2 — Build the graph for your project

Navigate to your project and initialize + build:

```bash
cd "/path/to/your/project"
code-review-graph init
code-review-graph build
```

`init` creates a `.code-review-graph/` folder in your project (add this to `.gitignore` — see below).
`build` parses your code and populates the graph. First run takes 30 seconds to a few minutes depending on project size.

Verify:

```bash
code-review-graph stats
```

Should print file count, node count, edge count, and a `last_updated` timestamp.

Add the graph's database folder to your project's `.gitignore`:

```bash
echo ".code-review-graph/" >> .gitignore
```

---

## Step 3 — Install the `filesystem` MCP

The `filesystem` MCP is distributed as a Claude Desktop **Extension**, which is the modern, easier path.

In Claude Desktop:

1. Go to **Settings** → **Extensions** (or **Connectors**, depending on version)
2. Search for **"filesystem"**
3. Install the official Anthropic **Filesystem** extension
4. In the extension settings, configure **allowed_directories** to include your project path:
   ```
   /path/to/your/project
   ```

Alternative: if the Extension is not available in your Claude Desktop version, the filesystem MCP can also be configured via `claude_desktop_config.json`. Instructions below in Step 4b.

---

## Step 4 — Configure `code-review-graph` MCP in Claude Desktop

Unlike `filesystem`, `code-review-graph` is configured via the `claude_desktop_config.json` file, not as an Extension.

### 4a — Back up the existing config

```bash
cp ~/Library/Application\ Support/Claude/claude_desktop_config.json ~/Desktop/claude_desktop_config.backup.json
```

If the file doesn't exist yet, that's fine — the backup will just fail silently and you'll create a fresh file.

### 4b — Edit the config

Open the file:

```bash
open -a TextEdit ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

**Important**: TextEdit can silently convert straight quotes (`"`) to curly quotes which break JSON. If you have a code editor installed (VS Code, Sublime, BBEdit), use that instead:

```bash
code ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

Paste the template from [`templates/claude_desktop_config.example.json`](templates/claude_desktop_config.example.json), then **replace** these placeholders:

- `/REPLACE/WITH/PATH/TO/YOUR/PROJECT` — the absolute path to your project
- `/Users/YOURNAME/.local/bin/code-review-graph` — your pipx install path (use `which code-review-graph` to get this)
- `/Users/YOURNAME` — your home directory

### 4c — Validate the JSON

```bash
python3 -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json > /dev/null && echo "VALID" || echo "BROKEN"
```

If this prints `BROKEN`, the most common cause is smart quotes from TextEdit. Open the file and look for `"` vs `"` characters — replace all curly quotes with straight ones.

### 4d — Optional: add `filesystem` here instead of the Extension

If you couldn't install the Extension in Step 3, use the template at [`templates/claude_desktop_config.example.json`](templates/claude_desktop_config.example.json) which includes both `code-review-graph` and `filesystem` as MCP entries. This works but is slightly more fragile than the Extension approach.

Important gotcha: you can have **either** the Extension **or** the config-file entry for filesystem, but **not both** — they'll conflict and one will win (usually the Extension). See [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md).

---

## Step 5 — Restart Claude Desktop and verify

Fully quit Claude Desktop: `Cmd+Q`. Not just closing the window — actually quit.

Relaunch Claude Desktop. Open a new chat.

Paste this test prompt:

```
Please do two things:
1. Call list_graph_stats_tool from code-review-graph MCP and tell me file count, node count, edge count, and last_updated timestamp.
2. Use filesystem MCP to read the first 10 lines of README.md in my project at /path/to/your/project and show me.
```

**Three possible outcomes:**

1. **Both work** → setup is successful. ✓
2. **Graph works, filesystem denies access** → filesystem MCP is installed but `allowed_directories` doesn't include your path. Fix in Settings → Extensions → Filesystem.
3. **Neither works** → check [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md).

---

## Step 6 — Establish a session workflow

Copy the prompts in [`prompts/`](prompts/) into a notes app you trust (Apple Notes, a Markdown file, etc.). Use them as follows:

- **Start of every session**: paste [`prompts/session_starter.md`](prompts/session_starter.md) as the first message
- **End of every session**: paste [`prompts/session_closer.md`](prompts/session_closer.md) before closing
- **For agentic multi-step work**: use Claude Code in Desktop (separate sidebar tab) with [`prompts/claude_code_starter.md`](prompts/claude_code_starter.md)

See [`docs/WORKFLOW.md`](docs/WORKFLOW.md) for the daily rhythm.

---

## Step 7 — Keep the graph fresh

After significant code changes, update the graph:

```bash
cd "/path/to/your/project"
code-review-graph build         # incremental, only re-parses changed files
```

Many people run this as a git pre-commit hook or after any substantial feature branch merge. Not required, but keeps Claude's structural picture accurate.

---

## Multiple projects?

Same setup, once per project. Build a graph in each project folder. Add entries for each to `claude_desktop_config.json` using distinct names like `code-review-graph-projectA` and `code-review-graph-projectB`.

Or, cleaner: switch which project `claude_desktop_config.json` points at when you switch focus, and restart Claude Desktop.

---

## What's next

- Read [`docs/WORKFLOW.md`](docs/WORKFLOW.md) for how to actually use the setup day-to-day
- Read [`docs/TOKEN_SAVINGS.md`](docs/TOKEN_SAVINGS.md) for honest caveats on what to expect
- If stuck, read [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) before opening an issue
