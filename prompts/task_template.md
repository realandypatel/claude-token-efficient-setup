# Task Prompt Template

A structure for writing task prompts that produce consistent, reliable results. Use after you've done a session_starter.md to set context.

---

## The template

```
Task: [ONE SENTENCE DESCRIBING THE GOAL]

Context:
- [What area of the code this touches]
- [Any prior work or commits relevant]
- [Any constraints — tests must pass, no breaking API changes, etc.]

Success criteria:
- [Concrete, checkable outcome 1]
- [Concrete, checkable outcome 2]
- [Concrete, checkable outcome 3]

Non-goals (don't do these):
- [Scope guard 1]
- [Scope guard 2]

Approach:
- Use code-review-graph to [find X, list callers of Y, etc.] before touching files.
- [Any specific tools, patterns, or checks to apply]

Stop and ask me if:
- [Condition 1 — e.g. you find an unexpected pattern]
- [Condition 2 — e.g. the change would touch more than N files]

After completing:
- [What to verify]
- [Whether to commit automatically or wait]
```

---

## Why each section matters

**Task (one sentence)**: Forces you to know what you actually want. If you can't say it in one sentence, you're not ready to task it.

**Context**: Claude doesn't know your situation unless you tell it. Three bullets is usually enough.

**Success criteria**: These become Claude's self-check. It will aim at these specifically.

**Non-goals**: Prevents scope creep. Claude often wants to "also fix this other thing it noticed" — say no upfront.

**Approach**: Nudges Claude toward using the graph + being structural, rather than reading files randomly.

**Stop and ask if**: Safety valve. Forces Claude to pause before going off the rails.

**After completing**: Sets expectations for verification and commits so Claude doesn't surprise you.

---

## Filled-in example

```
Task: Extract the AddAircraftModal component out of the monolith MechanicPortal.tsx into its own file.

Context:
- MechanicPortal.tsx is ~5000 lines; we've already extracted InviteTeamMemberModal in a prior step.
- AddAircraftModal is currently inline; used by Aircraft, WorkOrders, and Squawks sections.
- The extraction should match the style of the previous InviteTeamMemberModal extraction (commit d00f80a).

Success criteria:
- New file: components/redesign/mechanicPortal/AddAircraftModal.tsx
- All 14 addAc* state fields and 4 handlers moved with it
- All 3 call sites updated to import from the new location
- tsc passes with no new errors
- Manual check: "Add Aircraft" flow from Aircraft tab still opens the modal

Non-goals:
- Do NOT refactor state management style while you're at it
- Do NOT remove unrelated dead code
- Do NOT touch anything outside MechanicPortal.tsx and the new file

Approach:
- Use code-review-graph to locate every caller of AddAircraftModal before extracting.
- Match the barrel-less pattern from the InviteTeamMemberModal extraction.

Stop and ask me if:
- You find more than 3 call sites (I was wrong about scope)
- Extraction would require touching shared state in a parent that you didn't expect
- tsc errors appear that were not there before

After completing:
- Show me git diff before committing
- Don't commit automatically — I'll review and commit
```

---

Save variants of this for the task types you hit repeatedly: refactor extraction, bug investigation, feature addition, dependency upgrade, etc.
