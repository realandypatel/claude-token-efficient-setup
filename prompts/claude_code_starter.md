# Claude Code Mode Session Starter

Use this prompt when you open **Claude Code** in Claude Desktop's sidebar (a different surface than regular Chat). Claude Code mode is more agentic — it can run multi-step tasks, execute shell commands, and chain operations without as much back-and-forth.

Replace `REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT` with your project path before first use.

---

```
New Claude Code session on [PROJECT_NAME]. Load context:

1. Read REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT/TOMORROW.md and confirm what you see.

2. Call list_graph_stats_tool from code-review-graph MCP.

3. Confirm you have:
   - Filesystem access (you should have direct read/write, not just MCP)
   - Shell/bash execution
   - code-review-graph MCP
   - Any other connectors I have enabled

Working norms for this session:
- You have full filesystem + shell access in this mode. Use it.
- Never read node_modules/ or equivalent dependency folders.
- For large files (>1000 lines), read in chunks with offset/limit.
- After multi-step operations, give me 3-line summaries. Don't dump full output.
- Use code-review-graph to locate symbols before reading files blindly.
- When uncertain, ask before acting — no speculative implementation.
- Propose shell commands before running them; run after I approve.

After context loads, propose the first task from TOMORROW.md. Wait for my approval before starting any real work.
```

---

## When to use Claude Code mode vs Chat mode

**Claude Code mode is better when:**
- The task has many small steps (refactor touching multiple files, scripted cleanup, multi-file edits)
- You want Claude to verify its own work (run tsc, run tests, check git diff)
- You're okay approving actions rather than reviewing every word

**Chat mode is better when:**
- You want to think through a problem with Claude before acting
- The task is exploratory or research
- You want manual checkpoints between every small step
- You're doing work where a mistake would be expensive (e.g. production secrets)
