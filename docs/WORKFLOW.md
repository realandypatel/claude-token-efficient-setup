# Daily Workflow

How to actually use this setup day-to-day. This is the rhythm I landed on after a few false starts.

---

## The three surfaces and when to use each

You have three Claude surfaces available on macOS. They share MCP servers but have different strengths.

### Claude Desktop — Chat mode

**Best for**: strategic planning, MCP-heavy work (Supabase/Vercel queries), iterative exploration where you want manual checkpoints between steps.

**How it feels**: you ask, Claude proposes, you approve, Claude acts, you review, repeat. More deliberate.

### Claude Desktop — Claude Code mode (separate sidebar tab)

**Best for**: agentic multi-step tasks (refactors, multi-file edits, scripted cleanups, running tests). Claude Code mode has shell execution and can chain many operations.

**How it feels**: you describe the goal, Claude proposes a sequence of actions, you approve in batches, Claude works through them with less back-and-forth.

### Claude Code CLI (Terminal)

**Best for**: scripted/automated work, pre-commit checks, CI-style integration, or when you want Claude's output in a scriptable pipeline.

**How it feels**: command-line. Same capabilities as Claude Code in Desktop, just without the GUI.

---

## The daily cycle

### Start of day (or start of any session)

1. **Open Claude Desktop** (Chat or Claude Code mode, depending on what you plan to do)
2. **Start a new chat** — don't continue yesterday's chat (it has irrelevant context that will confuse Claude and burn tokens)
3. **Paste `prompts/session_starter.md`** — loads your TOMORROW.md pickup note, verifies MCPs, sets working norms
4. **Wait for Claude to confirm** context before giving it work
5. **State the day's goal** clearly — one main task, maybe a backup task

### During the day (the work rhythm)

1. **Write a task prompt** using the structure in `prompts/task_template.md`:
   - One-sentence goal
   - Context (what area of the code)
   - Success criteria (concrete, checkable)
   - Non-goals (prevent scope creep)
   - Approach (nudge toward using the graph)
   - Stop-and-ask conditions
2. **Let Claude propose** the plan before acting. Approve or adjust.
3. **Watch for scope creep** — if Claude wants to fix additional things, say "not now, note it for TOMORROW.md."
4. **Commit often, in small logical chunks.** Bigger commits are harder to review and harder to roll back.
5. **When you hit a stopping point, update TOMORROW.md** with any new findings — don't trust that you'll remember.

### End of day (session close)

1. **Paste `prompts/session_closer.md`**
2. **Claude updates TOMORROW.md**, runs `git status`, lists running processes, gives a 5-line summary
3. **Review the 5-line summary** — this is your chance to catch misunderstandings
4. **Close the chat** — don't leave it open for tomorrow

---

## Rules I wish I'd made earlier

### Rule 1: One task at a time, fully done

Start a task, finish it (or decide to park it with a clear note), then start the next. Don't juggle 3 half-done things — both you and Claude lose track of what's in each.

### Rule 2: Commit before starting anything risky

If you're about to run a big refactor, a dependency upgrade, or anything that touches the database — commit your current clean state first. That way rollback is `git reset`.

### Rule 3: Never rotate production secrets while tired

Security work is the worst thing to do at midnight. It feels urgent but the actual threat window rarely moves in the next 8 hours. Sleep first, rotate with a clear head.

### Rule 4: Pre-launch projects don't need the same caution as live ones

If you have zero real users, a 10-minute production outage is a non-issue. Don't over-engineer your ops rituals for a pre-launch project. Post-launch, upgrade.

### Rule 5: TOMORROW.md is sacred

It's the single source of truth for "what was in my head when I stopped." Keep it:
- Gitignored (never commit — it's personal state)
- Updated at every session close
- Read at every session start
- Short enough to actually read — prune completed items aggressively

### Rule 6: Don't switch tools mid-task

If you started a refactor in Claude Desktop Chat mode, finish it there. Switching to Claude Code mode or Terminal mid-task means losing context and re-explaining. Finish, then switch for the next task if appropriate.

### Rule 7: Prompts aren't one-and-done

The prompts in `prompts/` are templates to iterate on. After a month of real use, you'll notice patterns — add them to your versions of these prompts. Keep yours in a notes app you trust.

---

## A TOMORROW.md template

Your project's `TOMORROW.md` should live at the project root, be gitignored, and have roughly this structure:

```markdown
# Tomorrow's pickup

## CRITICAL
(only for genuinely urgent/blocking items — keep this section empty most days)

## Known bugs (need investigation)
- [bug 1 — where, what, what you tried]
- [bug 2]

## Missing features (product work)
- [feature 1]
- [feature 2]

## Refactor / migration progress
- Step 1 complete: commit [hash]
- Step 2 pending: [what]
- ...

## Small cleanups pending
- [low-priority item 1]

## Session workflow rules
- [any personal norms you've added — e.g. "never touch X without checking Y"]

## Switching surfaces
[notes about which Claude surface you're using for which task]
```

First few days, this file feels awkward. After a week, it feels like second nature and saves you a lot of re-explaining.

---

## Warning signs you're doing it wrong

- **Sessions feel confused or scattered** → you probably skipped the session_starter prompt
- **Claude keeps reading the same file over and over** → you're not leveraging the graph; check that the session_starter actually loaded
- **You keep discovering "oh I forgot about X"** → your TOMORROW.md is too sparse; write more at session close
- **Context feels bloated and slow after an hour** → close the chat and start a new one. Bloated context is a tax, not an asset.
- **You're making decisions at midnight** → stop. Nothing in software is actually urgent enough to justify that.

---

## Signs it's working

- New sessions load context in under a minute
- You don't re-explain what you're working on each time
- Claude reaches for graph queries instead of blind file reads
- `git log` shows steady small commits rather than weekend mega-commits
- TOMORROW.md is a short, accurate, useful document — not a dumping ground
