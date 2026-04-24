# Session Starter Prompt

Paste this as the **first message** of any new Claude Desktop chat. It loads context and sets working norms.

Replace `REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT` with your project path before first use. Save the filled-in version in a notes app for easy reuse.

---

```
Starting a new work session on [PROJECT_NAME]. Load context:

1. Read REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT/TOMORROW.md and confirm what you see. (This file is a personal pickup note from the previous session.)

2. Call list_graph_stats_tool from code-review-graph MCP and confirm the graph is current. I expect roughly the same file count as last session, with last_updated recent.

3. Confirm you have access to:
   - code-review-graph MCP (list_graph_stats_tool, search, and related tools)
   - filesystem MCP (read_text_file, list_directory)
   - any app connectors I have enabled (Supabase, Vercel, GitHub, etc.)

Working norms for this session:
- Never read node_modules/ or equivalent dependency folders unless I specifically ask.
- When reading large files (>1000 lines), use view_range / offset+limit — ask me which section if unclear.
- After each multi-step operation, give me a 3-line summary rather than dumping full output.
- Use code-review-graph MCP to locate symbols BEFORE reading files blindly.
- If you're about to read the same file twice, reuse what you already have in context.
- When uncertain what I want, ask before acting. No speculative implementation.

Once context is loaded, tell me what task from TOMORROW.md we should tackle and wait for my direction.
```
