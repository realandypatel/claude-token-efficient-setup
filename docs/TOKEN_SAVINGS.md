# Token Savings: The Honest Version

**TL;DR**: For navigation and structural queries on large codebases, the graph can reduce tokens dramatically — in one specific setup I saw ~90% savings on common queries. For other query types, savings are smaller. Your mileage will vary a lot.

---

## Where the savings come from

Without the graph, when you ask Claude something like "find all callers of function X in this codebase," Claude's only option is to read files. For a large codebase that might mean:

- Opening the most likely file (thousands of tokens)
- Not finding what it needs
- Opening the next candidate (thousands more tokens)
- Eventually stitching together an incomplete picture

With the graph, the same query becomes a single structured call that returns every caller with exact file paths and line numbers — using perhaps 50–100 tokens of Claude's budget.

---

## Where the savings are BIG (real wins)

- **"Where is symbol X used?"** — graph lookup is essentially free vs. reading many files
- **"What does function Y call?"** — one query returns the call graph
- **"What will break if I change this function?"** — impact analysis in one call
- **"List all files in module Z"** — structural query, no file reads
- **"Which files import from module A?"** — dependency inversion, graph answers instantly
- **Onboarding/orientation** — a fresh Claude session can query structure without reading code

For these query types, 10× savings is realistic. 20× happens. The "~90% reduction" claim is roughly accurate for a day of navigation-heavy work on a large codebase.

---

## Where the savings are SMALL or ZERO

- **"Explain what this function does line by line"** — Claude still has to read the lines
- **"Rewrite this function to do Y instead of X"** — Claude has to see the function
- **"Find me code that handles authentication"** — semantic search; without embeddings, graph falls back to keyword matches
- **"Is this code buggy?"** — requires actually reading the code
- **"Add a new feature that does Z"** — writing new code isn't a graph query

If your Claude usage is mostly "generate new code" or "review specific code for quality," graphs help less.

---

## Why you shouldn't trust specific percentages

Any claim like "saves 98% of tokens" is probably marketing, not measurement. Real savings depend on:

- **Codebase size**: Graph wins bigger on bigger codebases. On a 50-file project, manual reading might be fine.
- **Query type**: See sections above. Navigation wins; explanation doesn't.
- **How Claude is prompted**: If your prompt says "read the whole file and tell me X," Claude reads the whole file regardless of graph availability. The prompt norms in this repo's `prompts/` folder are designed to actually leverage the graph.
- **Whether embeddings are built**: `code-review-graph` supports embeddings for semantic search. Without them, semantic queries fall back to keyword matching, which misses a lot.

I don't publish a specific "average percentage" because I'd be making it up. My rough direct experience on one production codebase: navigation queries went from "eat thousands of tokens" to "eat dozens." That's all I can say honestly.

---

## How to maximize savings (habit-building)

1. **Always use the session starter prompt** — it tells Claude the graph is available and to use it.
2. **Phrase questions structurally when possible**: "Show me callers of X" rather than "Tell me how X is used throughout the code."
3. **Don't dump full files into context** if a targeted read is enough. Ask Claude to use `view_range` / offset+limit when it does need file content.
4. **Update the graph after significant changes** so the answers it gives match reality.
5. **Build embeddings for semantic wins** — `code-review-graph` has a command for this. Takes one indexing pass.

---

## When the graph will cost you tokens (overhead)

Being fair: the graph isn't free.

- Initial build: parses every file once. On a 600-file project, this was a 1–2 minute one-time cost.
- Incremental builds: re-parses only changed files. Typically seconds.
- Query responses: small but non-zero context usage.

For tiny codebases or one-off questions, skip the graph. It's for recurring work on codebases large enough that file-by-file reading is the actual bottleneck.

---

## Anti-pattern: "rebuild graph on every message"

Don't. `code-review-graph build` is designed for when the code has actually changed. Running it on every message wastes time and produces no new information.

Run it:
- After a significant feature branch merge
- After a major refactor
- Once at the start of a session if you pulled changes from another machine
- As a git pre-commit hook if you want aggressive freshness

---

## Summary

The graph is a **force multiplier for navigation**, a **weak multiplier for semantic search without embeddings**, and a **non-factor for pure code generation**. Your own mix of task types determines whether you'll see big savings. Anyone quoting you a guaranteed percentage is selling, not measuring.
