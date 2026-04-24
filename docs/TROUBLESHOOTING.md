# Troubleshooting

Real problems encountered during setup, with the fixes that worked. If you hit something not listed here, please open an issue or PR — adding to this file is the most valuable kind of contribution.

---

## Symptom: Claude Desktop shows "Connectors ⚠ 2" or similar warning, and my MCP servers aren't in the list

**Likely cause**: The "Connectors" panel often shows only Anthropic's cloud connectors (Figma, Gmail, Slack, etc.), NOT local MCP servers you added via `claude_desktop_config.json`. Your local MCP servers may be working fine and just not appearing in that list.

**Fix / diagnosis**:

1. Don't trust the UI — test directly. In a new chat, paste:
   ```
   Call list_graph_stats_tool from code-review-graph MCP. Report the result.
   ```
   If Claude returns graph stats, the MCP is working. The UI is just showing a different list.

2. The `⚠ 2` badge is about those cloud connectors needing reauth, not about your local MCPs.

3. If the test above fails, check the MCP logs:
   ```bash
   ls -lat ~/Library/Logs/Claude/ | head -10
   tail -80 ~/Library/Logs/Claude/mcp-server-code-review-graph.log
   ```

---

## Symptom: MCP log shows "Server started and connected successfully" but Claude says the tool isn't available

**Likely cause**: Request timeout, not a startup failure. MCP servers can go into timeout if idle for ~60 seconds, then reinitialize on next call. If Claude reported "no tool" right after a long gap, it may have been during a reconnection window.

**Fix**: Just retry the prompt. If it still fails, restart Claude Desktop (Cmd+Q, relaunch). A clean restart almost always clears transient MCP issues.

---

## Symptom: filesystem MCP says "Access denied - path outside allowed directories"

**Likely cause #1**: You configured the filesystem MCP via `claude_desktop_config.json` at the project path, but you ALSO have the filesystem Extension installed, and the Extension's settings only allow a different directory (often iCloud Drive).

**Fix**:

1. Open the Extension settings file:
   ```bash
   cat "/Users/YOURNAME/Library/Application Support/Claude/Claude Extensions Settings/ant.dir.ant.anthropic.filesystem.json"
   ```
2. If you see `allowed_directories` with only iCloud Drive or a different path, that's the problem.
3. Fix by either:
   - **(preferred)** Editing that file to add your project path to `allowed_directories`, and removing the filesystem entry from `claude_desktop_config.json`
   - **(or)** Disabling the Extension in Claude Desktop → Settings → Extensions, and keeping the `claude_desktop_config.json` filesystem entry

Don't have both — they conflict and one will win silently.

**Likely cause #2**: The path in your config has an odd character (space, special char) that isn't quoted properly.

**Fix**: Ensure the path is quoted in the config file's `args` entry: `'/Users/you/my project/'` with single quotes inside the JSON string.

---

## Symptom: I edited claude_desktop_config.json and now Claude Desktop won't work

**Likely cause**: JSON syntax error. The most common one is TextEdit converting straight quotes (`"`) to curly quotes (`"` `"`) which break JSON.

**Fix**:

1. Restore your backup:
   ```bash
   cp ~/Desktop/claude_desktop_config.backup.json "/Users/YOURNAME/Library/Application Support/Claude/claude_desktop_config.json"
   ```
2. Validate the JSON before launching:
   ```bash
   python3 -m json.tool "/Users/YOURNAME/Library/Application Support/Claude/claude_desktop_config.json" > /dev/null && echo "VALID" || echo "BROKEN"
   ```
3. Re-edit in a real code editor (VS Code, Sublime, BBEdit, nano) — not TextEdit.

**Prevention**: Always validate with the `python3 -m json.tool` command before launching Claude Desktop.

---

## Symptom: `code-review-graph` command not found after `pipx install`

**Likely cause**: `pipx ensurepath` was run but Terminal wasn't restarted, so the updated PATH hasn't been picked up.

**Fix**: Close ALL Terminal windows and open a fresh one. Verify:

```bash
echo $PATH | tr ':' '\n' | grep -i local
which code-review-graph
```

If still not found:

```bash
ls ~/.local/bin/ | grep code-review
```

If the binary is there, your PATH doesn't include `~/.local/bin`. Add it:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

---

## Symptom: `npx` fails inside Claude Desktop's MCP spawn, logs show "node not found"

**Likely cause**: Claude Desktop spawns MCP servers with a minimal PATH that doesn't include your Node.js install location. This matters if Node is in a non-standard location (e.g., nvm, fnm, or `~/.local/bin`).

**Fix**: Use a shell wrapper in `claude_desktop_config.json` that explicitly sets PATH before running the command:

```json
"filesystem": {
  "command": "/bin/sh",
  "args": [
    "-c",
    "HOME=/Users/YOURNAME PATH=/Users/YOURNAME/.local/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin exec npx -y @modelcontextprotocol/server-filesystem '/absolute/path/to/project'"
  ]
}
```

The same pattern works for `code-review-graph` — see the template in `templates/claude_desktop_config.example.json`.

---

## Symptom: The graph returns stale data — doesn't reflect recent code changes

**Likely cause**: Graph build hasn't run since the changes.

**Fix**:

```bash
cd "/path/to/your/project"
code-review-graph build     # incremental update
code-review-graph stats     # check last_updated
```

To automate: add a git pre-commit hook that runs `code-review-graph build` after staging changes.

---

## Symptom: Graph build is slow on a large project

**Context**: First build on a ~600-file TypeScript project: ~1–2 minutes. Incremental builds after small changes: seconds.

**If your first build is much slower** (e.g. 10+ minutes), check:

- Is the tool trying to parse `node_modules/`? It shouldn't, but check. If so, there may be a `.gitignore` issue — the tool respects gitignore patterns.
- Are you on a particularly large monorepo? Consider pointing the graph at a specific package root instead of the repo root.

---

## Symptom: I want the graph to understand semantic queries ("find auth code") not just structural ones

**Current state**: Without embeddings, semantic queries fall back to keyword matching.

**Fix**: `code-review-graph` supports embedding the graph for semantic search:

```bash
code-review-graph embed
```

This is a one-time indexing pass (takes longer than `build`). After it completes, semantic queries get much better.

Note: embeddings cost some disk space and may require API credentials if using a remote embedding model — check the tool's own docs for current options.

---

## Symptom: My secrets got committed to git and I'm panicking

This isn't about MCP setup but it's a rite of passage for most developers using `.env` files. Quick triage:

1. **Do NOT delete the commit and force-push.** If the secrets are already on a remote, they're already in the clone history of anyone who pulled.
2. **Rotate the secrets immediately.** Go to the provider (Supabase, Vercel, AWS, etc.) and rotate any keys that were exposed. Rotation first, cleanup second.
3. **Update your local .gitignore** so it can't happen again: add `.env`, `.env.local`, `.env.*.local`, `.env.deploy`, etc.
4. **Run `git rm --cached <file>`** on the committed file to stop tracking it (without deleting the local copy).
5. **Commit the .gitignore change** so other branches/collaborators pick it up.

For a proper history cleanup, tools like `git-filter-repo` or BFG Repo-Cleaner can scrub old commits — but note that secrets already on a remote are already compromised regardless of history rewriting.

This scenario is the reason `TOMORROW.md` has a recommended "security posture" section template — see `prompts/session_closer.md`.

---

## Something else broken?

Open an issue with:

1. What you were trying to do
2. Exact command(s) that failed
3. Exact error message
4. Your macOS version, Claude Desktop version, Node version, Python version
5. Relevant content of `~/Library/Logs/Claude/mcp-server-*.log`

If you figure it out before getting help — even better, open a PR adding your fix to this file.
