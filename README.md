# claude-token-efficient-setup

> A tested setup guide and prompt workflow for pairing Claude Desktop with `code-review-graph` MCP and `filesystem` MCP on macOS, for token-efficient AI-assisted development on large codebases.

**Status**: Initial release based on a single production setup session. Tested on one Mac (macOS, Claude Desktop latest). Community contributions welcome — especially if you hit issues not covered in [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md).

---

## What this repo is

A documentation project. The files here are configuration templates, setup steps, prompt recipes, and troubleshooting notes — gathered while wiring up Claude Desktop to work efficiently against a large Next.js/TypeScript codebase.

## What this repo is NOT

- **Not a new tool.** The heavy lifting is done by [`code-review-graph`](https://pypi.org/project/code-review-graph/) (an existing MCP server) and Anthropic's `filesystem` MCP extension. Full credit in [`ATTRIBUTIONS.md`](ATTRIBUTIONS.md).
- **Not a fork or repackaging** of any of the above tools. Nothing here modifies or redistributes their code.
- **Not a shortcut** around learning MCP. If you want to understand the system, read the source docs linked in ATTRIBUTIONS. This repo is for developers who already want to use these tools and hit the same config gotchas I did.

---

## The problem this solves

When Claude reads a large codebase file-by-file, it burns through tokens fast. A 5,000-line React component plus its dependencies can eat through a context window before Claude does any real thinking.

Two tools solve this in complementary ways:

1. **`code-review-graph`** — indexes your codebase into a structural graph (functions, classes, imports, call relationships). Claude queries the graph instead of reading files blindly. Navigation queries that used to cost thousands of tokens now cost dozens.
2. **`filesystem` MCP** — gives Claude direct read/write access to your project files, so it can act on your code rather than just describe changes.

Together, these enable Claude Desktop to do real agentic development work — refactors, bug investigations, impact analysis — without drowning in context.

## Honest caveats about token savings

See [`docs/TOKEN_SAVINGS.md`](docs/TOKEN_SAVINGS.md) for the honest version. Short version: for **navigation and structural queries** on large codebases, the graph can reduce tokens dramatically (rough estimate: ~90% in our case). For **semantic queries** ("find me code that handles X"), savings are smaller unless you also build embeddings. We don't claim specific universal percentages because they depend heavily on your codebase and query style.

---

## Who this guide is for

- You're an engineer using Claude Desktop Mac app (not the CLI, not the browser)
- You work on a codebase large enough that file-by-file reading feels slow
- You're comfortable editing JSON config files and running terminal commands
- You want a worked, tested example rather than piecing things together from scratch

If any of these don't apply — check out the upstream tool docs directly.

---

## Quick start

1. Read [`SETUP.md`](SETUP.md) end-to-end before running commands. The order matters.
2. Back up any existing Claude Desktop config before touching it: `cp ~/Library/Application\ Support/Claude/claude_desktop_config.json ~/Desktop/claude_desktop_config.backup.json`
3. Follow the install + config steps. Expect 30–45 minutes the first time.
4. Try the [`prompts/session_starter.md`](prompts/session_starter.md) prompt on your first session to verify the setup.
5. If you hit problems, check [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) — several non-obvious issues are documented there with fixes.

---

## Repo structure

```
claude-token-efficient-setup/
├── README.md                                    # you're here
├── SETUP.md                                     # step-by-step install guide
├── LICENSE                                      # MIT
├── ATTRIBUTIONS.md                              # credit to upstream tools
├── .gitignore
├── templates/
│   ├── claude_desktop_config.example.json       # MCP config for code-review-graph
│   └── filesystem_extension_settings.example.json  # filesystem MCP allowed_directories
├── prompts/
│   ├── session_starter.md                       # first message of any session
│   ├── session_closer.md                        # last message before closing
│   ├── claude_code_starter.md                   # for Claude Code mode in Desktop
│   └── task_template.md                         # how to structure task prompts
└── docs/
    ├── TOKEN_SAVINGS.md                         # honest explanation with caveats
    ├── TROUBLESHOOTING.md                       # real problems + fixes
    └── WORKFLOW.md                              # the daily rhythm I use
```

---

## Quick demo

After setup, asking Claude Desktop something like:

> Where is `AddAircraftModal` used in this codebase? Show me every file and the functions that reference it.

Without the graph: Claude opens the largest candidate file, reads thousands of lines, may still miss references in other files. Expensive and slow.

With the graph: Claude calls one graph query, returns every reference with exact file + line numbers. Cheap and complete.

---

## Contributing

If you follow this guide and hit an issue that isn't in [`docs/TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md), please open an issue or PR. Real experience from different setups makes this guide more useful to everyone.

Explicitly welcome:
- Additional troubleshooting entries for failure modes I didn't hit
- Config variations for non-macOS systems (Linux, Windows)
- Prompt recipes that worked well for specific task types
- Corrections where my description of the underlying tools is wrong

Not appropriate:
- PRs that fork or modify `code-review-graph` or `filesystem` MCP code — those belong upstream
- PRs that add language-specific code to this repo (this is docs + templates only)

---

## License

MIT. See [`LICENSE`](LICENSE).

## Author

Andy Patel. The tools this guide uses have their own authors — see [`ATTRIBUTIONS.md`](ATTRIBUTIONS.md).
