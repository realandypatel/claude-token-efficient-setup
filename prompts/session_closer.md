# Session Closer Prompt

Paste this as the **last message** before closing a Claude Desktop chat. Ensures state is captured in `TOMORROW.md` and no background processes are left running.

Replace `REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT` with your project path before first use.

---

```
Winding down this session. Before we stop:

1. Update REPLACE_WITH_ABSOLUTE_PATH_TO_YOUR_PROJECT/TOMORROW.md:
   - Add any new bugs discovered today
   - Add any new features/product work identified
   - Update refactor/migration/task progress (what got committed, what's still pending)
   - Remove items that are now fully complete
   - Keep any CRITICAL section at the top — only remove items once genuinely resolved

2. Check for anything uncommitted that shouldn't be lost:
   - Run: git status --short
   - Flag anything that looks like real work I might want to commit vs. ephemeral noise

3. Kill any background processes you started this session (dev servers, watchers, monitors, tsc). List what's running first, kill only after I confirm.

4. Give me a 5-line summary of what we accomplished this session — I want to read it before I close.

Do NOT commit TOMORROW.md (it should be in .gitignore).
Do NOT touch production environments.
Do NOT auto-fix things we haven't agreed on.
```

---

## Why this pattern works

- Captures state externally (in `TOMORROW.md`) so next session's context loads quickly
- Prevents leaving dev servers or watchers running overnight
- Forces you to review what was done — catches mistakes before they carry forward
- The 5-line summary is short enough to actually read when tired
